# SpringBoot限流
为了避免服务被高频访问或恶意访问，保证服务的高用性，必须采用限流，以保证服务的不会死机
## 单机模式
### guava 令牌桶限流
guava自带了限流模块，配置缓存就可以实现ip加方法限流
#### 1.导入包
```xml
<dependency>
	<groupId>com.google.common</groupId>
	<artifactId>guava</artifactId>
	<version>33.5.0-jre</version>
</dependency>

<dependency>
    <groupId>com.github.ben-manes.caffeine</groupId>
    <artifactId>caffeine</artifactId>
    <version>3.2.3</version>、
</dependency>

```

#### 2.AOP
1. 先写注解
```java
@Document
@Target(ElementType.METHOD) 
@Retention(RetentionPolicy.RUNTIME) 
public @interface RateLimit { 
	double qps() default 10.0; // 默认每秒 10 个令牌 
}
```
2. 编写 AOP 切面
```java
@Aspect 
@Component 
public class RateLimitAspect {
	private static final Cache<String, RateLimiter> cache = Caffeine.newBuilder() .expireAfterAccess(Duration.ofMinutes(10)) .maximumSize(10000) .build();
	
	@Around("@annotation(rateLimit)") 
	public Object around(ProceedingJoinPoint joinPoint, RateLimit rateLimit) throws Throwable { 
		ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes(); 
		HttpServletRequest request = attributes.getRequest();
		String ip = getClientIp(request); 
		String methodName = joinPoint.getSignature().toShortString(); 
		String limitKey = methodName + ":" + ip;
		RateLimiter limiter = limiters.get(key, k -> RateLimiter.create(rateLimit.qps())); 
		if (limiter != null && !limiter.tryAcquire()){
			 throw new RuntimeException("请求过于频繁"); 
		} 
		return joinPoint.proceed();
	
	}

	private String getClientIp(HttpServletRequest request) { 
		String ip = request.getHeader("X-Forwarded-For"); 
		if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) { 
			ip = request.getRemoteAddr();
		} 
		return ip.split(",")[0];
	 }
}
```

## 分布式模型
### redis限流
#### 1.导入包
```xml
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson-spring-boot-starter</artifactId>
    <version>3.27.0</version>
</dependency>
```
### 2.AOP
1. 先写注解
```java
@Document
@Target(ElementType.METHOD) 
@Retention(RetentionPolicy.RUNTIME) 
public @interface RateLimit { 
	long qps() default 10; // 默认每秒 10 个令牌 

}
```
2. 编写AOP切面
```java
@Aspect
@Component
public class RateLimitAspect {

    @Autowired
    private RedissonClient redissonClient;

    @Around("@annotation(rateLimit)")
    public Object around(ProceedingJoinPoint joinPoint, RateLimit rateLimit) throws Throwable {
        HttpServletRequest request = ((ServletRequestAttributes) RequestContextHolder.currentRequestAttributes()).getRequest();
        String ip = getClientIp(request);
        String methodName = joinPoint.getSignature().toShortString();
        
        String limitKey = "rate_limit:" + methodName + ":" + ip;

        RRateLimiter limiter = redissonClient.getRateLimiter(limitKey);

        limiter.trySetRate(RateType.OVERALL, rateLimit.qps(), 1, RateIntervalUnit.SECONDS);
        
        limiter.expire(Duration.ofMinutes(10));

        if (!limiter.tryAcquire()) {
            throw new RuntimeException("访问太频繁，请稍后再试");
        }

        return joinPoint.proceed();
    }
    
    private String getClientIp(HttpServletRequest request) { 
		String ip = request.getHeader("X-Forwarded-For"); 
		if (ip == null || ip.isEmpty() || "unknown".equalsIgnoreCase(ip)) { 
			ip = request.getRemoteAddr();
		} 
		return ip.split(",")[0];
	 }
}

```
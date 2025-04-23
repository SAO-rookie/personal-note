[Spring-framework 测试文档](https://docs.spring.io/spring-framework/reference/testing.html)
[Spring Boot的Testing文档](https://docs.spring.io/spring-boot/reference/testing/index.html)
[Mockito 系列 |贝尔东 --- Mockito Series | Baeldung](https://www.baeldung.com/mockito-series)
[JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)
# springboot测试
## web测试
1.开启测试
需要开启web接口测试，需要在类头上使用以下注解
```java
@EnableWebMvc  
@AutoConfigureMockMvc  
@SpringBootTest
public testControllerTest {
	@Autowired  
	private MockMvc mockMvc;
}
```
或者 用以下方式
```java
@SpringBootTest
public testControllerTest {
	private MockMvc mockMvc;
	
	@BeforeEach 
	public void setup() { 
		this.mockMvc = MockMvcBuilders.standaloneSetup(new AccountController()).build();
	}
}
```
2.
## 业务测试



# Mockito的使用






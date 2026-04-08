[Spring-framework 测试文档](https://docs.spring.io/spring-framework/reference/testing.html)
[Spring Boot的Testing文档](https://docs.spring.io/spring-boot/reference/testing/index.html)
[Mockito 系列 |贝尔东 --- Mockito Series | Baeldung](https://www.baeldung.com/mockito-series)
[JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/#writing-tests-annotations)
[JsonPath: 文档](https://github.com/json-path/JsonPath)
[Matchers (Hamcrest 3.0 API)](https://hamcrest.org/JavaHamcrest/javadoc/3.0/org/hamcrest/Matchers.html)
# springboot测试
## web测试
[参考文档-MockMvc :: Spring Framework](https://docs.spring.io/spring-framework/reference/testing/mockmvc.html)
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
或者 用以下方式,使用指定控制器
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
2.使用MockMvc测试接口
当使用MockMVC，请静态导入以下类
- `MockMvcBuilders.*`
- `MockMvcRequestBuilders.*`
- `MockMvcResultMatchers.*`
- `MockMvcResultHandlers.*`
```java
@Test
void whenBuildingDoNotExistPageCraftDocumentReturnNull() throws Exception {
	mockMvc.perform(get("/craft/registration/page/1/10"))  
	        .andExpect(status().isOk())  
	        .andExpect(jsonPath("$.code").value(0))  
	        .andExpect(jsonPath("$.data.total").value(0));
}
```
## 业务测试
1.开启测试
```java
@SpringBootTest
public testControllerTest {
	@MockBean  
	private FileInfoService fileInfoService;
}
```
2.测试
```java
@Test  
void test(){
	when(fileInfoService.checkFileInfoExistsByMd5(anyString())).thenReturn(null)
    Assertions.assertNull(fileInfoService.checkFileInfoExistsByMd5("4ec2ca9723cba50d8843b042836b951b"));  
}
```








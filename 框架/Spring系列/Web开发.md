

# WEB开发

## 模板引擎

Spring Boot不建议我们使用jsp，建议使用模板引擎；同样也他给我们提供了模板引擎的自动配置。

提供了默认配置的模板引擎有以下几种：

- Thymeleaf
- FreeMarker
- Velocity
- Groovy
- Mustache

通过使用模板引擎来避免使用jsp。

## 使用Thymeleaf

首先第一步，引坐标

```xml
<!-- 模板引擎-->
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>
```

默认情况下，Thymeleaf模板引擎的文件会放在templates文件夹下面，和静态资源一样，映射也已经配置好了。

默认的配置属性如下：

```properties
spring.thymeleaf.prefix=classpath:/templates/
spring.thymeleaf.suffix=.html
```

有需要的可以根据自己的实际情况去配置。

### 代码

将我们需要进行展示的网页放在templates文件夹下面，另外就是Thymeleaf的语法自己去网上看：[传送门](https://www.thymeleaf.org/doc/tutorials/3.0/usingthymeleaf.html#what-kind-of-templates-can-thymeleaf-process)

下面这个是我的代码，然后作用就是，访问的时候，后端会传递一个值给前端，然后显示出来。

```java
@Controller
public class WelcomeController {

    @RequestMapping("/")
    public String index(ModelMap modelMap) {
        
        modelMap.addAttribute("msg", "hello, how are you?");

        return "/index";
    }
}
```

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<h1 th:text="${msg}">123</h1>
</body>
</html>
```

### 启动

启动应用之后，直接访问127.0.0.1:8080就能够看到结果。



## RESTful

啥是RESTful风格：我的理解就是，请求的地址比较简洁，使用GET、POST、PUT、DELETE这些不同的请求对应CRUD这些不同的方法

| 请求类型 |  请求地址  |      功能说明      |
| :------: | :--------: | :----------------: |
|   GET    |   /user    | 查询所有的用户信息 |
|   GET    | /user/{id} |  查询单个用户信息  |
|   POST   |   /user    |    添加用户信息    |
|   PUT    | /user/{id} |    修改用户信息    |
|  DELETE  | /user/{id} |    删除用户信息    |

### 代码

下面的这个是一个Controller，然后进行各种操作之后的返回值为json数据

```java
@RestController
@RequestMapping("/user")
public class UserController {
    /**
     * 使用Map模拟数据库
     */
    private static Map<Integer, User> userMap = new HashMap<>();

    static {
        userMap.put(0, new User(0, "张三", 22));
        userMap.put(1, new User(1, "李四", 32));
        userMap.put(2, new User(2, "王五", 43));
        userMap.put(3, new User(3, "刘六", 17));
    }
    /**
     * 获取用户列表，访问路径：/user/
     * @return 用户列表
     */
    @ApiOperation(value = "获取用户列表，访问路径")
    @GetMapping("")
    public List<User> getUserList() {
        ArrayList<User> users = new ArrayList<>(userMap.values());

        return users;
    }

    /**
     * 获取单个用户信息，访问路径：/user/{id}
     * @param id 用户Id
     * @return 用户信息
     */
    @ApiOperation("获取单个用户信息")
    @ApiImplicitParam(name = "id", value = "用户Id")
    @GetMapping("{id}")
    public User getUser(@PathVariable Integer id) {
        return userMap.get(id);
    }

    /**
     * 添加一个用户，访问路径：/user/
     * @param user 要增加的用户信息
     * @return 结果
     */
    @PostMapping("")
    public String addUser(User user) {
        userMap.put(user.getId(), user);
        System.out.println(user);
        return "success";
    }

    /**
     * 通过id更新用户信息，访问路径：/user/{id}
     * @param id 用户id
     * @param user 用户信息
     * @return 结果
     */
    @PutMapping("{id}")
    public String updateUser(@PathVariable Integer id, User user) {
        userMap.put(id, user);
        System.out.println(userMap);
        return "success";
    }

    /**
     * 删除一个用户，访问路径：/user/{id}
     * @param id 删除的用户的id
     * @return 结果
     */
    @DeleteMapping("{id}")
    public String deleteUser(@PathVariable Integer id) {
        userMap.remove(id);
        System.out.println(userMap);
        return "success";
    }
}
```

### 测试

省事没有写视图，直接使用了Spring Boot集成的测试

```java
@SpringBootTest
@AutoConfigureMockMvc
public class UserControllerTest {
    @Autowired
    private MockMvc mockMvc;

    @Test
    public void getUserList() throws Exception {
        mockMvc.perform(get("/user"))
                .andDo(print())
                .andExpect(status().isOk());
    }

    @Test
    public void getUser() throws Exception {
        mockMvc.perform(get("/user/0"))
                .andDo(print())
                .andExpect(status().isOk());
    }

    @Test
    public void addUser() throws Exception {
        RequestBuilder requestBuilder = post("/user").param("id", "4")
                .param("name", "xiaoming")
                .param("age", "23");

        mockMvc.perform(requestBuilder)
                .andDo(print())
                .andExpect(status().isOk());
    }

    @Test
    public void updateUser() throws Exception {
        RequestBuilder requestBuilder = put("/user/0").param("id", "0")
                .param("name", "xiaoming")
                .param("age", "23");

        mockMvc.perform(requestBuilder)
                .andDo(print())
                .andExpect(status().isOk());

    }java

    @Test
    public void deleteUser() throws Exception {

        mockMvc.perform(delete("/user/1"))
                .andDo(print())
                .andExpect(status().isOk());
    }
}
```
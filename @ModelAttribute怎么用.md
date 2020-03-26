# 引导
在项目中，controller中经常看到@ModelAttribute或者@ModelAttribute("someName")这种用法，这到底是个什么意思呢？本文带你一探究竟。

# 作用
@ModelAttribute，即org.springframework.web.bind.annotation.ModelAttribute，是spring的一个注解，可以用在方法或者属性上。被该注解标注的方法会在该controller的请求方法执行之前执行，比如你定义一个TestController,其中一个方法test()标注@RequestMapping("/api/test"),然后发起一个请求/api/test,那么在test执行前，该类中所有@ModelAttribute标注的方法会先执行，执行之后的结果会放在Model中，后续test方法可以通过Model获得一些全局属性。那么问题来了，**Model是什么**，这个暂时可以理解为该类的一个全局属性，是一个map形式。
**简单理解就是**：

- 该注解标注的方法会在handler具体方法执行之前，作为全局属性存放在Model，供后续handler具体方法执行的时候调用


在知道了ModelAttribute的执行时间和作用，我们来看看这个注解怎么用。
# 
# 使用方法
该注解标注的方法可以带有和请求方法一样的参数，比如请求方法带有一个String userId，那么该方法也可以带有，两个方法对应的值是一样的，但要注意的是：  

- 能自动获取的值是可以用@RequestParam拿到的值，即get请求的?后接的参数或者post请求的body里的参数，且如果是body里的参数，则此post请求对应的Content-Type为application/x-www-form-urlencoded，如果为application/json则无法用@RequestParam获取到，@ModelAttribute方法的参数也就获取不到，参数为null。

```java
private static final String USER_ID = "userId";
@ModelAttribute(USER_ID)
    public String getUserId(String userId) {
        System.out.println(userId);
        return "my" + userId;
    }
```
如上面的代码，就是自动获取的方式，我的测试请求可以用如下格式：
```
Get /api/test/model?userId=123
```
```
Post /api/test
Content-Type: application/x-www-form-urlencoded

userId=234

```
---
另一种方式是在方法的参数前标明获取方式：
```java
private static final String USER_ID = "userId";
@ModelAttribute(USER_ID)
    public String getUserId(@PathVariable("userId") String userId) {
        System.out.println(userId);
        return "my" + userId;
    }
```
如上，这种情况下对应的请求需要在路径中包含userId,且需要注意的是：

**如果用了这种方式，那么这个类中所有的请求方法对应的uri都路径中都需要包含userId的占位符**
比如我请求了如下接口
```
Get /api/test/model/tt
//会报错"Missing URI template variable 'userId' for method parameter of type String"
```
同理，还可以用@RequestParam @RequestBody等获取参数

使用的话我们可以直接用如下的方式
```java
public Object getSomeThing1(@ModelAttribute(USER_ID) String userID) {
        
        return userID;
}
```

---
不加参数怎么用呢？
举个例子：
```java
@ModelAttribute
    public UserData getuserData() {
        UserData userData = new UserData();
        userData.setUserId("id");
        Map<String, String> map = new HashMap<>();
        map.put("testKey", "testValue");
        userData.setData(map);
        System.out.println(userData);
        return userData;
    }
```
我们定义了一个getuserData方法，返回了一个userData实例，由于我们没有像上面定义getUserId方法一样直接指定@ModelAttribute的value值，返回后对应的key默认为返回类型且首字母小写，即userData。通过定义这个方法，我们可以在model中获得一个key为userData的value，值为该方法的返回值。

如下为方法执行后model中的值。
```json
{
    "userData": {
        "userId": "id",
        "data": {
            "testKey": "testValue"
        }
    }
}
```
那么定义好了，怎么在方法中用呢?
```java
@RequestMapping(value = "/api/test/model/post", method = RequestMethod.POST)
    public Object postUserData(@ModelAttribute UserData data) {
        return data;
    }
```
这样我们就可以获得一个UserData实例了，及时我的请求中没有任何参数。  
**注意**：上面的例子我们获取了一个userData实例，如果对不同的请求我想得到不同的实例怎么办呢？  

- 首先可以通过类似于上面getUserId(String UserId)这种方法实现，也就是你的请求参数中添加一个userId,在@ModelAttribute修饰的方法中构造自己的UserData实例，这样就可以

- 其次我们可以用下面的getuserData()的方式，只传递一个userId参数，然后代码不变，由于ModelAttribute会自动获取@RequestParam可以获取到的值，会用请求中的值替换model中预设的值。

例子：
```
Post /api/test/model/post?userId=popo
```
最后返回的值有原来的
```json
{
  "userId": "id",
  "data": {
    "testKey": "testValue"
  }
}
```
变成了
```json
{
  "userId": "popo",
  "data": {
    "testKey": "testValue"
  }
}
```
这个变化只是因为我的请求加了参数，代码未变。

---

# 总结几点

- @ModelAttribute适合给一个controller增加一些全局的变量

- 如果返回一个String值，可以用上面例子中的@ModelAttribute(USER_ID) String userID 这种方式，也可以在请求方法中直接引入model或者modelMap进行get，但不可以直接@ModelAttribute String userID。

- 如果返回的是一个对象，一个实例，那么可以直接@ModelAttribute  UserData data使用，也可以@ModelAttribute("userData") UserData data使用，还可以用Model或者ModelMap进行get.



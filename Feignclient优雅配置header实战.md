@[TOC]
# 综述
在使用声明式feignclient时，有些情况下会用到要添加header，比如token、cookie之类的信息，本人根据使用场景将header分为两类 ：  
- 静态header，比如Content-Type之类的，请求一个接口的所有请求，它们的Content-Type都是一样的  
- 动态header，比如每个请求对应着不同的token信息，服务端需要通过不同的token进行校验，得到具体的用户信息  

# 静态header的处理方式
对于该类型的header，可以直接添加在注解的headers属性上，可以对每个接口指定不同的值。
```java
    @RequestMapping(method = RequestMethod.POST, value = "/something",headers = {"Content-Type=application/x-www-form-urlencoded"})
    Integer myMethod(@RequestBody String s);
```
# 动态header的处理方式
对于动态header是无法使用静态的方式的，每个请求对应的header值不一样，以下以token为例，总结两种解决方法

## 添加拦截器的配置
```java
public class AuthorityConfig implements RequestInterceptor{
    @Override
    public void apply(RequestTemplate template) {
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        //添加token
        template.header(Constants.TOKEN, request.getHeader(token.getToken()));
    }
}
```
使用这种方式，在feignclient声明处指定configuration属性即可，如下：  
```java
@FeignClient(value = "client", url = domain, configuration = AuthorityConfig.class)
```
## 使用自定义FeignClient的方式
```java
public MyFeignClient getClient(Client client, String token) {
        return Feign.builder().client(client)
                .encoder(new Encoder.Default())
                .decoder(new Decoder.Default())
                .requestInterceptor(template -> template.header("token", token))
                .target(MyFeignClient.class, "http://domain");
    }
```
如上，在配置类中添加该方法，在service中注入配置类，调用该方法即可产生自定义的client实例，进行http的调用。**使用该方法不需要使用声明式client**         

---
在添加动态header时，不管是方式一还是方式二，都有一些细节问题，推荐查看[feign踩坑](https://blog.csdn.net/cuiyaocool/article/details/104808742)

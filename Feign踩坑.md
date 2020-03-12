本文踩坑有二：
  1. not annotated with HTTP method type (ex. GET, POST)
  2. FeignException$BadRequest: status 400 

# not annotated with HTTP method
报错内容如下：
```java
java.lang.IllegalStateException:
 Method XXX not annotated with HTTP method type (ex. GET, POST)
```
Feign可以使用自带注解@RequestLine以及spring的注解@RequestMapping、@GetMapping等
## 几个需要注意的点：  

 - 使用spring的注解@RequestMapping需要指定method属性以及value（路径信息）
 - 看了很多其他博客说不可以使用@GetMapping等这类注解，本人亲测，可以使用
 - **第三点比较重要的**：
如果使用Feign自带注解@RequestLine，需要修改默认配置。spring cloud netflix默认为feign提供的默认bean如下：

```java
Decoder feignDecoder：ResponseEntityDecoder（其中包含SpringDecoder）

Encoder feignEncoder：SpringEncoder

Logger feignLogger：Slf4jLogger

Contract feignContract：SpringMvcContract

Feign.Builder feignBuilder：HystrixFeign.Builder

Client feignClient：如果Ribbon启用，则为LoadBalancerFeignClient，否则将使用默认的feign客户端。
```
看上方的Contract feignContract：SpringMvcContract，说明feign默认使用springmvc的协议或者约定，使用@RequestMapping的注解，若需要使用@RequestLine，需要修改约束配置，如下
```java
@Configuration
public class Configuration {
    @Bean
    public Contract feignContract() {
        return new feign.Contract.Default();
    }
}
```
在feignclient注解里配置

```java
@FeignClient(value = "client", url = "${XXX}", configuration = Configuration .class)
```
# FeignException
报错内容如下：

```java
FeignException$BadRequest: status 400 reading xxService#xxmethod(String, Interger)
```
看错误码400，http错误码400主要有两种形式：
1、bad request 意思是 "错误的请求"；
2、invalid hostname 意思是 "不存在的域名"。
400 Bad Request 是由于明显的客户端错误（例如，格式错误的请求语法，太大的大小，无效的请求消息或欺骗性路由请求），服务器不能或不会处理该请求。
## 原因
查找其他博客得到结果有一下几类：

 - 传递的参数为空
 - 参数长度过长
 - header过长
 - RequestMapping没有指定method属性
 - 本人遇到的情况，header参数错误（下面讲解错误原因）
## 解决
对于第一种传参为空，可以指定参数require=false，如下

```java
@RequestParam(value = "param",required = false) String param
```
还要注意的是如果使用rest风格的请求，路径有占位符，一定要使用@PathVariable注解，这也是很多人踩过的坑。
对于第二种问题，是由于springboot内嵌tomcat对参数长度限制是8K，可以修改配置进行解决

```java
server.max-http-header-size=20480
```
第三种情况与第二种情况一致。
第四种情况直接指定方法就好。
## 本人踩坑复现
本人遇到的问题是header设置的问题。由于需要使用token进行验证，而且不同的请求对应不同的token，导致无法使用静态header进行指定，于是使用修改配置的方式进行，直接修改restTemplate.

```java
@Configuration
public class AuthorityConfig implements RequestInterceptor{
    @Override
    public void apply(RequestTemplate template) {
        SecurityContext securityCxt = SecurityContextHolder.getContext();
        XXX token = XXX securityCxt.getAuthentication();
        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
        HttpServletRequest request = attributes.getRequest();
        //添加token
        template.header(Constants.TOKEN, request.getHeader(token.getToken()));
        //template.headers().remove("Content-Type");
//        Map<String, Collection<String>> headers = new HashMap<>();
//        ArrayList<String> headerToken = new ArrayList<>();
//        headerToken.add(request.getHeader(token.getToken()));
//        headers.put(Constants.TOKEN, headerToken);
//        headers.put("Content-Type", new ArrayList(){{add("application/x-www-form-urlencoded");}});
        template.headers(new HashMap<>());
        template.header("Content-Type", "application/x-www-form-urlencoded");
        template.header(Constants.TOKEN_TEACHER, token.getToken());
        System.out.println("headers is ： "+template.headers());
    }
}
```
上方代码看倒数第四行，我设置了一个空的map，之后才进行的其他header的添加。这是踩坑之后才发现得这么做的。
看我注释掉的代码发现，template.header（）需要一个string和一个集合，也就是说，template会把key相同的属性放到一个集合里。这似乎也很正常，坑的是它本身有一些默认的header，如果你添加了同名的header后只是加入到了一个集合而不是覆盖，导致我注释的代码添加Content-Type后，该key对应的header是

```java
[application/json,application/x-www-form-urlencoded]
```
这就很尴尬，服务器因为这个拒绝了我。查看其源码实现

```java
public RequestTemplate headers(Map<String, Collection<String>> headers) {
    if (headers != null && !headers.isEmpty()) {
      headers.forEach(this::header);
    } else {
      this.headers.clear();
    }
    return this;
  }
```
发现传入null或者空可以实现clear()，所以我传了空先进行清除后再逐个添加，这样解决了400的问题。


以上是这两天的踩坑集锦，有误之处请指正，不胜感激~  
[更多文章](https://blog.csdn.net/cuiyaocool)
1. 后端不接受前端传递过来的数据格式
```java
org.springframework.web.HttpMediaTypeNotSupportedException: Content type 'application/json;charset=UTF-8' not supported
```
如何解决:


2. 引入*mvc:annotation-driven*时发生没有可用的*cacheManager*的bean

描述：
```xml
<mvc:annotation-driven />
```

错误信息：
```java
org.springframework.beans.factory.BeanCreationException: Error creating bean with name 'org.springframework.cache.interceptor.CacheInterceptor#0': Cannot resolve reference to bean 'cacheManager' while setting bean property 'cacheManager'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No bean named 'cacheManager' available
```

原因是：
引入了错误的命名空间，例如：
```xml
xmlns:mvc="http://www.springframework.org/schema/cache"
```
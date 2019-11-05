## 使用*AspectJ*注解开发*Spring AOP*
### 选择切点
使用动态代理的理论来讲，就是需要拦截哪个方法，这里以jdk动态代理为例：
jdk动态代理为*RoleServiceImpl*(被代理对象)创建一个代理对象，其中需要借助*RoleService*接口来实现，关键的代码就是：
```java
Proxy.newProxyInstance(被代理对象.getClass().getClassLoader(), 被代理对象.getClass().getInterface(), 拦截器对象);
```
```java
public interface RoleService {
    void printRole(Role role);
}

@Component
public class RoleServiceImpl implements RoleService {

    @Override
    public void printRole(Role role) {
        System.out.println(role);
    }
    
}
```
### 创建切面
类比动态代理，就是创建一个拦截器
```java
@Aspect
public class RoleAspect {
    
    @Pointcut("execution(* ch11.aop.service.impl.RoleServiceImpl.printRole(..))")
    public void print(){}
    
    @Before("print()")
    public void before(){
        System.out.println("before ...");
    }
    
    @After("print()")
    public void after(){
        System.out.println("after ...");
    }
    
    @AfterThrowing("execution(* ch11.aop.service.impl.RoleServiceImpl.printRole(..))")
    public void afterThrowing(){
        System.out.println("afterThrowing ...");
    }
    
    @AfterReturning("execution(* ch11.aop.service.impl.RoleServiceImpl.printRole(..))")
    public void afterReturn(){
        System.out.println("afterReturning ...");
    }
    
}
```
### 配置Spring，让Spring找到对应的Bean和产生动态代理对象
```java
@Configuration
@EnableAspectJAutoProxy // 启用AspectJ框架的自动代理，Spring才会生成动态代理对象，进而可以使用AOP
@ComponentScan("ch11.aop")
public class AopConfig {

    @Bean
    public RoleAspect getRoleAspect(){
        return new RoleAspect();
    }

}
```

### 测试
```java
// 通过注解来生成ApplicationContext对象
ApplicationContext act = new AnnotationConfigApplicationContext(AopConfig.class);
RoleService roleService = (RoleService) act.getBean(RoleService.class);
Role role = new Role(1L, "role_name_1", "note_1");
roleService.printRole(role);
System.out.println("++++++++++++++++ 测试异常情况 +++++++++++++++");
role = null;
roleService.printRole(role);
```

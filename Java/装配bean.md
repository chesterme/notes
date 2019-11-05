## 依赖注入的三种方式
+ 构造器注入
+ setter注入
+ 接口注入

### 构造器注入
1. 定义一个pojo
```java
public class Role {

    private Long id;
    private String roleName;
    private String note;

    public Role(Long id, String roleName, String note) {
        this.id = id;
        this.roleName = roleName;
        this.note = note;
    }

    public Role(){}

    //getter and setter
}
```
2. 定义一个spring bean文件
```xml
<bean id="role1" class="ch10.pojo.Role">
    <constructor-arg index="0" value="1"/>
    <constructor-arg index="1" value="总经理"/>
    <constructor-arg index="2" value="公司管理者"/>
</bean>
```

3. 生成Role对象
```java
 ApplicationContext ctx = new ClassPathXmlApplicationContext("ch10/config/Beans.xml");
        Role role = (Role) ctx.getBean("role1");
        System.out.println(role);
```

### 使用setter注入
1. 定义一个pojo
2. 定义一个Spring bean文件
```xml
<bean id="role2" class="ch10.pojo.Role">
    <property name="roleName" value="高级工程师"/>
    <property name="note" value="重要人员"/>
</bean>
```

### 使用接口注入


## 通过xml配置装配bean
```xml
<bean id="user1" class="ch10.pojo.User">
    <property name="name" value="张三"/>
    <property name="role" ref="role1"/>
</bean>

<bean id="complexAssembly" class="ch10.pojo.ComplexAssembly">
    <property name="id" value="1"/>
    <property name="list">
        <list>
            <value>value-list-1</value>
            <value>value-list-2</value>
            <value>value-list-3</value>
        </list>
    </property>
    <property name="map">
        <map>
            <entry key="key1" value="value-key-1"/>
            <entry key="key2" value="value-key-2"/>
            <entry key="key3" value="value-key-3"/>
        </map>
    </property>
    <property name="prop">
        <props>
            <prop key="prop1">value-prop-1</prop>
            <prop key="prop2">value-prop-2</prop>
            <prop key="prop3">value-prop-3</prop>
        </props>
    </property>
    <property name="array">
        <array>
            <value>value-array-1</value>
            <value>value-array-2</value>
            <value>value-array-3</value>
        </array>
    </property>
    <property name="set">
        <set>
            <value>value-set-1</value>
            <value>value-set-2</value>
            <value>value-set-3</value>
        </set>
    </property>
</bean>
```

## 通过命名空间装配bean
+ 引入对应的命名空间和XML模式文件

## 通过*Component*注解装配Bean
1. 使用*Component*注解标识该类可以被Spring容器进行扫描作为装配的bean
```java
@Component(value = "book")
public class Book {

    @Value("1")
    private Long id;
    @Value("数据库系统概念")
    private String bookName;
    @Value("Abraham Silberschata, Henry F.Korth, S.Sudarshan")
    private String author;
    @Value("机械工业出版社")
    private String publish;

    // setter and getter
```
2. 使用*ComponentScan*注解设置扫描的位置
```java
/**
 * 通知Spring IoC扫描组件
 */

//@ComponentScan(basePackageClasses = {Role.class, RoleService.class})
@ComponentScan(basePackages = {"ch10.pojo", "ch10.service"})
public class PojoConfig {
}
```
3. 使用*AnnotationConfigApplicationContext*构造器生成*ApplicationContent*
```java
ApplicationContext act = new AnnotationConfigApplicationContext(PojoConfig.class);
Book book = act.getBean(Book.class);
System.out.println(book);
Role role = act.getBean(Role.class);
RoleService roleService = act.getBean(RoleService.class);
roleService.printRoleInfo(role);
```

## 使用*Autowired*自动装配
注入对象
```java
@Component("RoleService2")
public class RoleServiceImpl2 implements RoleService2 {

    @Autowired
    private Role role = null;

    @Override
    public void printRoleInfo() {
        System.out.println(role);
    }

    // getter and setter
}
```

### 二义性问题
```error
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'roleController': Unsatisfied dependency expressed through field 'roleService'; nested exception is org.springframework.beans.factory.NoUniqueBeanDefinitionException: No qualifying bean of type 'ch10.service.RoleService' available: expected single matching bean but found 2: roleServiceImpl,roleServiceImpl3
```

### 解决二义性问题
1. 使用注解*Primary*，优先注入某类
2. 使用注解*Qualifier*，改变Spring寻找依赖注入的方式(按照类型匹配)，按类型查找
```java
@Component
public class RoleController {

    @Autowired
    @Qualifier("roleService3")
    private RoleService roleService = null;

    public void printRole(Role role){
        roleService.printRoleInfo(role);
    }

}
```
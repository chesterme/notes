## 通过Spring提供的JNDI机制获取tomcat上配置的数据源
tomcat上配置的数据源
```xml
<!--
name：JNDI名称，
url：数据库jdbc连接，
username：用户名，
password：数据库密码
-->
<Resource name="jdbc/ssm"
        auth="Container"
        type="javax.sql.DataSource"
        driverClassName="com.mysql.cj.jdbc.Driver"
        url="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8&amp;serverTimezone=GMT%2B8"
        username="test"
        password="123456"/>
```
spring中配置bean
```xml
<bean id="dataSource" class="org.springframework.jndi.JndiObjectFactoryBean">
    <property name="jndiName" value="java:comp/env/jdbc/ssm"/>
</bean>
```

## 使用第三方的数据库连接池
```xml
<bean id="dataSource" class="org.apache.commons.dbcp2.BasicDataSource">
    <property name="driverClassName" value="com.mysql.cj.jdbc.Driver"/>
    <property name="url" value="jdbc:mysql://localhost:3306/ssm?characterEncoding=utf-8&amp;serverTimezone=GMT%2B8"/>
    <property name="username" value="test"/>
    <property name="password" value="123456"/>
    <!-- 连接池最大数据库连接数 -->
    <property name="maxOpenPreparedStatements" value="255"/>
    <!-- 最大等待连接中的数量 -->
    <property name="maxIdle" value="5"/>
    <!-- 最大等待毫秒数 -->
    <property name="maxWaitMillis" value="10000"/>
</bean>
```

## 使用JdbcTemplate
### 配置bean
```xml
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"/>
</bean>
```
### 使用
```java
ApplicationContext act = new ClassPathXmlApplicationContext("ch12/beans/bean.xml");
JdbcTemplate jdbcTemplate = (JdbcTemplate) act.getBean("jdbcTemplate");
Long id = 1L;
String sql = "select id, role_name, note from t_role where id = " + id;
Role role = jdbcTemplate.queryForObject(sql, new RowMapper<Role>() {
    @Override
    public Role mapRow(ResultSet resultSet, int i) throws SQLException {
        Role role = new Role();
        role.setId(resultSet.getLong(1));
        role.setRoleName(resultSet.getString(2));
        role.setNote(resultSet.getString(3));
        return role;
    }
});
System.out.println(role);
```

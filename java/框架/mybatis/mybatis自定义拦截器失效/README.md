# :confused: 一、问题描述
新搭建的一个`springboot+mybatis`，使用`@Intercepts`进行分页拦截，不同的是这次项目是多数据源，以前一直都这么这么用，然后这次新项目居然没有实现分页。
![](./pic/mybatis1.gif)

# 🧐 二、面向搜索编程
在网上找了一圈，大致说法有几种：
- 使用了`WebMvcConfigurationSupport`替代了`WebMvcConfigurer`要重写`WebMvcConfigurer`的`addInterceptors`方法，把分页拦截器加进去
   
   > 我项目中并没有干这样的事情。
- 还有说`swagger`的配置实现了`WebMvcConfigurer`然后重写了`addInterceptors`，没有添加自定义的拦截器。
   
   > 说得有道理，然后我检查了`swagger`配置发现并没有实现`WebMvcConfigurer`，为了彻底排除不是`swagger`的问题，直接先删掉swagger
- 既然`swagger`没有，那会不会是其它的配置
  > 优秀的程序员的学会举一反三，然后事实就是都没有
  

上面三个方向，在本项目中好像这些都没有关系
![](./pic/mybatis2.jpg)

 # :smirk: 三、终极大招
到这一步实在是没办法，只有把代码回退了到刚搭建好的零状态，改成单数据源，简单写个查询。发现分页又可以了。离谱到家了
![](./pic/mybatis3.png)

**🤨提出疑问：难道是多数据源导致的？**

继续面向搜索编程，终于发现了一个东西叫做`SqlSessionFactory`，`SqlSessionFactoryBean`在初始化执行`buildSqlSessionFactory`方法创建`SqlSessionFactory`的时候会有下面这几行代码，说明默认初始化的`SqlSessionFactory`会把这些拦截器都自动装载进去。
```java
if (!isEmpty(this.plugins)) {
      for (Interceptor plugin : this.plugins) {
        configuration.addInterceptor(plugin);
        if (LOGGER.isDebugEnabled()) {
          LOGGER.debug("Registered plugin: '" + plugin + "'");
        }
      }
    }
```
![](./pic/mybatis4.jpg)
我在初始化`SqlSessionFactory`的时候，完全没有考虑到拦截器的问题，下面是我初始化`SqlSessionFactory`的配置

```java
@Primary
@Bean(name = "primarySqlSessionFactory")
public SqlSessionFactory primarySqlSessionFactory(@Qualifier("primaryDataSource") DataSource primaryDataSource) throws Exception {
    final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    sessionFactory.setDataSource(primaryDataSource);
    sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
            .getResources(LocalDataSourceConfig.MAPPER_LOCATION));
    return sessionFactory.getObject();
}
```

在上面的代码创建`SqlSessionFactory`做一些修改
```java
@Primary
@Bean(name = "primarySqlSessionFactory")
public SqlSessionFactory primarySqlSessionFactory(@Qualifier("primaryDataSource") DataSource primaryDataSource) throws Exception {
    final SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    sessionFactory.setDataSource(primaryDataSource);
    // 添加拦截器
    sessionFactory.setPlugins(new Interceptor[]{new PageInterceptor()});
    sessionFactory.setMapperLocations(new PathMatchingResourcePatternResolver()
            .getResources(LocalDataSourceConfig.MAPPER_LOCATION));

    return sessionFactory.getObject();
}
```
然后再试一下，发现分页可以了。

![](./pic/mybatis5.jpg)
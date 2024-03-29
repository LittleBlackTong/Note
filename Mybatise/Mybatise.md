# Mybatise 源码

一、 mybatise 如果获取数据源

1. 通过SqlSessionFactoryBuilder.build() 将config-propertis.xml 转换成 stream 作为参数
2. 通过XMLConfigBuilder.parse()对读取配置进行转换.
3. 通过parseConfiguration()方法将数据转换
4. enviromentsElement 方法将读取到的数据转换成enviroments
5. 通过 dataSourceElement 将 xml 中的配置文件转化为datasource对象.
6. 将 enviroment 设置到Configuration当中

二、 读取 mapper 文件的四种方式以及读取级别

* resource
* url
* class
* package
 
> 在读取 配置文件 xml mapper 方式转换成 String, 分别判断 那种类型不为 null 便使用那种读取方式 优先级按照上边顺序.

三、Executor type

* SIMPLE
* REUSE
* BATCH

> 默认使用的 SMIPLE

四、Mybatise 的一级缓存

1. 在通过 executor 执行 sql 将 MappingStatement id,offset,limit,sql 生成 key
> mybatise 默认是开启 一级缓存的

六、Mybatise 如果操作数据库的

1. 通过 DefaultSqlSessionFactory.openSession() 开启session
2. 通过 Configuration 类来创建 Simplexecutor
3. 通过 simpleExecutor 分别调用 selectOne() selectList()
4. 然后调用CachingExecutor 执行 query 方法将结果缓存下来

七、Mybatise 处理流程

(一) 初始化运行环境

1. SqlSessionFactoryBuilder.build() 

> sqlSessionFactory 创建SqlSession

2. new XMLConfigBuilder(inputStream,enviroment,properties)

> 将 config.xml 通过 stream 转化成 XMLConfigBuilder

3. parser.parse() 

> 将 XMLConfigBuilder 转化将读取的config 转化成XNode(可以理解成上下文,包含了config配置文件的内容)

4. parseConfiguration(parser.evaNode)

> 通过获得的 Xnode 会包含 mybitise-config 中的所有内容.分别调用
> * propertiesElement()
>  获取 resource 和 url 设置到 configuration 和 Xparser 中
> * environmentsElement() 
> 读取配置文件当中的 environment 节点进行设置,配置数据源 dataSource 和 transactionManager 将配置设置到 configuration 中
> * mapperElement()
> 读取 配置文件中的 mapper 节点根据,根据 mapper 节点的属性 **package** **resource** **url** **class** 分别读取 mapper 配置中的内容

5.加载配置完成后获得 sqlSessionFactory

(二)开启 session

1. .openSessionFactory()来开启 session
2. .openSessionFormaDataSource(ExecutorType   execType,TransactionIsolationLeval leval,boolean autoCommit)

> 分别调用 getEnviroment()
> getTransactionFactoryFromEnviroment() 
> new Executor()
> 初始化 session
> * 其中 newExecutor 会初始化执行器 也就是 Executor 对象
> * Executor 执行器默认有三个值类型 
> **BATCH** 批量
> **REUSE** 重用
> **SIMPLE** 默认 三种执行器类型

3. 初始化环境和执行器之后返回 DefaultSqlSession

(三) 执行查询

1. session.selectOne()
> configuration 获取 mappedStatement
> 通过 executor 调用 query 获取进行查询,
> 调用 createCatchKey 方法创建一级缓存的 key 值
> 由 mapperStatement 进行获取的 id 
> 分页 roubounds 获取 limit 和 offset
> 以及 执行的 sql 生成一级缓存的 key 值


6. Mybatis 二级缓存

mybatis 通过二级缓存来提升数据检索效率，来避免每一次数据查询都查询数据库，一级缓存是sql session 级别的缓存，每一个
用户查询的时候使用 sqlsession 的本地缓存里面数据，而不是从数据库里面读取，如果想要实现跨 sqlsession 级别的缓存，一级缓存无法实现，
当多个用户查询数据的时候，就可以把数据放在二级缓存里面。

一级缓存，sqlsession 对象中有个 excutor excutor 会先到 一级缓存（localcache）里面区查，查到了直接返回，没有查到再去数据库中查询。
二级缓存则是引入了 cachingExecutor 装饰器， 在进行查询的时候 会先用区 mapper namespace 二级缓存里面先查询。全局缓存跨 session。






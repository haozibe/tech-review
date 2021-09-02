## mybatis

## mybatis原理

1. 通过getResourceAsStream方法读取核心配置文件config,返回inputStream。
2. 根据返回的InputStream解析出Configuration对象，然后创建sqlSessionFactory。new SqlSessionFactory.build(input stream)
3. 根据一系列属性从SqlSessionFactory中创建SqlSession对象。 openSessionFromDataSource方法
4. 从SqlSession中调用Executor执行数据库操作和生成具体sql指令。
5. 对执行结果进行二次封装。
6. 提交和执行事务。

## 缓存机制

一级缓存是SqlSession级别的缓存，同一个SqlSession下面如果存在缓存直接返回。

![img](https://images2017.cnblogs.com/blog/1254583/201710/1254583-20171026214546023-1354746770.png)

二级缓存是mapper级别缓存，

![img](https://images2017.cnblogs.com/blog/1254583/201710/1254583-20171029185910164-1823278112.png)

### 和hibernate对比

1.缓存对比：hibernate一级缓存是session缓存，需要对Session进行严格管理。二级缓存是sessionFactory缓存，分为内置和外置

## 面试问题

- mybatis级联查询。一对一通过association标签，一对多通过collection标签。
- mybatis 美元符号和井号区别。
- mybatis原理
- 字段名不一样，通过sql别名或者resultMap中指定对应关系。
- 通过selectLastInsertId获取自增主键。
- mybatis和hibernate对比。
- 怎么传递参数。用#{1}占位符传递，用@param传递。用Map集合作为参数装载。
- mybatis用rowBounds对象进行分页，是针对ResultSet结果集执行内存分页，而非物理分页，可以在sql内直接书写带有物理分页的参数完成物理分页。分页插件是通过拦截sql后改写sql，添加物理分页的语句和参数实现物理分页的。
- mybatis如何实现事务。




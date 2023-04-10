# MyBatis 架构简介

MyBatis 分为三层架构，分别是基础支撑层、核心处理层和接口层，如下图所示：

<div align="center"> <img src="https://technotes.oss-cn-shenzhen.aliyuncs.com/2022/202206152125788.png" width="800px"/> </div>

## 一、基础支撑层

MyBatis 基础支撑层可以划分为上图所示的九个基础模块。

- 资源加载模块
- 类型转换模块：实现了 MyBatis 中 JDBC 类型与 Java 类型之间的相互转换。
- 日志模块：集成 Java 生态中的第三方日志框架。
- 反射工具模块：在 Java 反射的基础之上进行的一层封装。
- Binding 模块：我们无须编写 Mapper 接口的具体实现，而是利用 Binding 模块自动生成 Mapper 接口的动态代理对象。
- 数据源模块：MyBatis 的数据源模块提供了一套数据源实现，也提供了与第三方数据源集成的相关接口。
- 缓存模块：MyBatis 提供了一级缓存和二级缓存。

<div align="center"> <img src="https://technotes.oss-cn-shenzhen.aliyuncs.com/2022/202206152138111.png" width="400px"/> </div>

- 解析器模块：MyBatis 中有 mybatis-config.xml，Mapper.xml 两部分配置文件需要进行解析。
- 事务管理模块：MyBatis 对数据库中的事务进行了一层简单的抽象，提供了简单易用的事务接口和实现。

## 二、核心处理层

核心处理层是 MyBatis 核心实现所在，其中涉及 MyBatis 的初始化以及执行一条 SQL 语句的全流程。

### 配置解析

MyBatis 有三处可以添加配置信息的地方，分别是：mybatis-config.xml 配置文件、Mapper.xml 配置文件以及 Mapper 接口中的注解信息。在 MyBatis 初始化过程中，会加载这些配置信息，并将解析之后得到的配置对象保存到 Configuration 对象中。

### SQL 解析与 scripting 模块

MyBatis 中的 scripting 模块就是负责动态生成 SQL 的核心模块。它会根据运行时用户传入的实参，解析动态 SQL 中的标签，并形成 SQL 模板，然后处理 SQL 模板中的占位符，用运行时的实参填充占位符，得到数据库真正可执行的 SQL 语句。

### SQL 执行

在 MyBatis 中，要执行一条 SQL 语句，会涉及非常多的组件，比较核心的有：Executor、StatementHandler、ParameterHandler 和 ResultSetHandler。下图展示了 MyBatis 执行一条 SQL 语句的核心过程：

<div align="center"> <img src="https://technotes.oss-cn-shenzhen.aliyuncs.com/2022/202206152128946.png" width="800px"/> </div>

Executor 会调用事务管理模块实现事务的相关控制，同时会通过缓存模块管理一级缓存和二级缓存。

SQL 语句的真正执行将会由 StatementHandler 实现。StatementHandler 会先依赖 ParameterHandler 进行 SQL 模板的实参绑定，然后由 java.sql.Statement 对象将 SQL 语句以及绑定好的实参传到数据库执行，从数据库中拿到 ResultSet，最后，由 ResultSetHandler 将 ResultSet 映射成 Java 对象返回给调用方，这就是 SQL 执行模块的核心。

### 插件

很多成熟的开源框架，都会以各种方式提供扩展能力。当框架原生能力不能满足某些场景的时候，就可以针对这些场景实现一些插件来满足需求，这样的框架才能有足够的生命力。这也是 MyBatis 插件接口存在的意义。

## 三、接口层

接口层是 MyBatis 暴露给调用的接口集合，这些接口都是使用 MyBatis 时最常用的一些接口，例如，SqlSession 接口、SqlSessionFactory 接口等。

其中，最核心的是 SqlSession 接口，你可以通过它实现很多功能，例如，获取 Mapper 代理、执行 SQL 语句、控制事务开关等。
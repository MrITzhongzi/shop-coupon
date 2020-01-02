## 商城优惠券项目


### 第3集    手把手带你进行数据库设计

**简介：小D商场优惠券功能数据库设计，优惠券表、用户优惠券表设计全过程**

------

- 商场功能模块数据库设计，一般企业中自己负责设计所开发功能模块的数据表

- 商场优惠券表设计

  * 优惠券表设计

    ```sql
    CREATE TABLE `t_coupon`
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `code` varchar(128) NOT NULL DEFAULT '' COMMENT '优惠券码',
    `pic_url`varchar(255) NOT NULL DEFAULT '' COMMENT '优惠券图',
    `achieve_amount` int(11) NOT NULL DEFAULT 0 COMMENT '达到满减资格金额',
    `reduce_amount` int(11) NOT NULL DEFAULT 0 COMMENT '所减金额',
    `stock` int(11) NOT NULL DEFAULT 0 COMMENT '库存，当库存为0不可领取',
    `title`varchar(64) NOT NULL DEFAULT '' COMMENT '优惠券名称',
    `status` int(1) NOT NULL DEFAULT 0 COMMENT '状态为0表示可用，1为不可用',
    `create_time` datetime DEFAULT NULL,
    PRIMARY KEY(`id`)
    )ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8 COMMENT '优惠券定义表';
    ```

    

  * 用户优惠券表设计

    ```
    CREATE TABLE `t_user_coupon`(
    `id` int(11) NOT NULL AUTO_INCREMENT,
    `user_coupon_code` varchar(128) NOT NULL DEFAULT '' COMMENT '用户优惠券码',
    `pic_url`varchar(255) NOT NULL DEFAULT '' COMMENT '优惠券图',
    `coupon_id` int(11) NOT NULL DEFAULT 0 COMMENT 't_coupon表外键ID',
    `user_id` int(11) NOT NULL DEFAULT 0 COMMENT '所领取用户id',
    `status` int(1) NOT NULL DEFAULT 0 COMMENT '状态为0表示未使用，1为已使用',
    `order_id` int(11) NOT NULL DEFAULT 0 COMMENT '对应t_order表外键',
    `create_time` datetime DEFAULT NULL,
    PRIMARY KEY(`id`)
    )ENGINE=InnoDB AUTO_INCREMENT=0 DEFAULT CHARSET=utf8 COMMENT '用户优惠券';
    ```

    

​             



### 第4集    springboot整合数据库连接池durid插件

**简介：小D商场优惠券功能梳理，每个模块功能梳理**

------

- 在可视化界面生成数据库表

- springboot整合durid功能实战

- durid后台功能讲解，配置过程

  * ```yml
    server:
       port: 8089
    druid:
       allow: #允许登陆的IP地址
          ip: 127.0.0.1
       login: #登陆的账户密码
          user_name: root
          password: root
    spring:
         datasource:
          driverClassName: com.mysql.jdbc.Driver
          url: jdbc:mysql://localhost:3306/shop?autoReconnect=true&useUnicode=true&characterEncoding=UTF-8&useSSL=false
          username: root
          password: daniel
          type: com.alibaba.druid.pool.DruidDataSource
          # 连接池的配置信息
          # 初始化大小，最小等待连接数量，最大等待连接数量，最大连接数
          initialSize: 1
          minIdle: 1
          maxIdle: 5
          maxActive: 20
          # 配置获取连接等待超时的时间
          maxWait: 60000
          # 配置间隔多久才进行一次检测，检测需要关闭的空闲连接，单位是毫秒
          timeBetweenEvictionRunsMillis: 60000
          # 配置一个连接在池中最小生存的时间，单位是毫秒
          minEvictableIdleTimeMillis: 300000
          validationQuery: SELECT 1 FROM DUAL
          testWhileIdle: true
          testOnBorrow: true
          testOnReturn: false
          # 打开PSCache，并且指定每个连接上PSCache的大小
          poolPreparedStatements: false
          maxPoolPreparedStatementPerConnectionSize: 20
          # 配置监控统计拦截的filters，去掉后监控界面sql无法统计，'wall'用于防火墙
          filters: stat,wall
          # 通过connectProperties属性来打开mergeSql功能；慢SQL记录
          connectionProperties: druid.stat.mergeSql=true;druid.stat.slowSqlMillis=5000
          # 合并多个DruidDataSource的监控数据
          #spring.datasource.useGlobalDataSourceStat=true
    ```

    ```java
        <dependency>
    			<groupId>com.alibaba</groupId>
    			<artifactId>druid</artifactId>
    			<version>1.1.1</version>
    		</dependency>
    		<dependency>
    			<groupId>mysql</groupId>
    			<artifactId>mysql-connector-java</artifactId>
    			<optional>true</optional>
    		</dependency>
    ```

    ```java
    package com.xdclass.jvm.config;
    
    import com.alibaba.druid.pool.DruidDataSource;
    import com.alibaba.druid.support.http.StatViewServlet;
    import com.alibaba.druid.support.http.WebStatFilter;
    import org.springframework.beans.factory.annotation.Value;
    import org.springframework.boot.context.properties.ConfigurationProperties;
    import org.springframework.boot.web.servlet.FilterRegistrationBean;
    import org.springframework.boot.web.servlet.ServletRegistrationBean;
    import org.springframework.context.annotation.Bean;
    import org.springframework.context.annotation.Configuration;
    
    import javax.sql.DataSource;
    import java.util.HashMap;
    import java.util.Map;
    
    @Configuration
    public class DruidConfig {
    
    
        @Value("${druid.login.user_name}")
        private String userName;
    
        @Value("${druid.login.password}")
        private String password;
    
        /**
         * 必须配置数据源，不然无法获取到sql监控，与sql防火墙监控
         */
        @Bean(name = "default_databaseSource")
        @ConfigurationProperties(prefix = "spring.datasource")
        public DataSource druidDataSource() {
            return new DruidDataSource();
        }
    
        @Bean
        public ServletRegistrationBean druidServlet() {
            ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean();
            servletRegistrationBean.setServlet(new StatViewServlet());
            servletRegistrationBean.addUrlMappings("/druid/*");
            Map<String, String> initParameters = new HashMap<>();
            initParameters.put("loginUsername", userName);// 用户名
            initParameters.put("loginPassword", password);// 密码
            initParameters.put("resetEnable", "false");// 禁用HTML页面上的“Reset All”功能
            servletRegistrationBean.setInitParameters(initParameters);
            return servletRegistrationBean;
        }
    
        @Bean
        public FilterRegistrationBean filterRegistrationBean() {
            FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
            filterRegistrationBean.setFilter(new WebStatFilter());
            filterRegistrationBean.addUrlPatterns("/*");
            filterRegistrationBean.addInitParameter("exclusions", "*.js,*.gif,*.jpg,*.png,*.css,*.ico,/druid/*");
            return filterRegistrationBean;
        }
    }
    ```

    

* 效果图              

![image-20190704205228566](/Users/daniel/Library/Application Support/typora-user-images/image-20190704205228566.png)





### 第5集    mybatis逆向工程使用

**简介：小D商场优惠券功能梳理，每个模块功能梳理**

------

- mybatis逆向生成工具讲解
- mybatis逆向生成工具实战生成代码
- 逆向工程mybatis-generator配置
  * 引入依赖
  * 引入generator工具类
  * 引入配置文件generator.xml

```powershell
<dependency>
      <groupId>org.mybatis.spring.boot</groupId>
      <artifactId>mybatis-spring-boot-starter</artifactId>
      <version>1.3.0</version>
</dependency>
<dependency>
			<groupId>org.mybatis.generator</groupId>
			<artifactId>mybatis-generator-core</artifactId>
			<scope>test</scope>
			<version>1.3.2</version>
			<optional>true</optional>
</dependency>
```









### 第6集    关于功能测试Junit的使用

**简介：功能测试Junit**

------

- 什么是Junit
  *  JUnit是一个Java语言的单元测试框架，可以大大缩短你的测试时间和准确度。多数Java的开发环境都已经集成了JUnit作为单元测试的工具
- 带你进入Junit与Springboot的整合实战
- 引入pom文件
- 代码规范之测试类编写中应该注意哪些点
  * Test类命名规范
  * Springboot场景下禁止使用Main方法进行测试







### 第7集    新入职第一件事—跑增删改查

**简介：基于mybatis的增删改查代码**

------

- 持久层框架很多，如：Mybatis、hibernate、JPA
- 基于mybatis的增删改查代码
- 增删改查实战
- 引入mybatis依赖
- 配置@MapperScan进行包扫描





![image-20190708214527197](/Users/daniel/Library/Application Support/typora-user-images/image-20190708214527197.png)













### 第8集    基于durid查看查询效果

**简介：durid查询效果查看**

------

- 基于durid查看查询效果
  * 对比有无下面的查询结果
  * 插入数据得出结论，时间越短，UUID重复的概率越高，因为UUID生成规则和机器码（MAC地址）有关
- 对比有无索引的请求差异化  
  * 10万条数据，根据code查询数据记录
    * 没加索引15次查询里面最高时间102ms
    * 加了索引15次查询里面最高时间4ms



![image-20190708213818145](/Users/daniel/Library/Application Support/typora-user-images/image-20190708213818145.png)











### 第9集    springboot分层架构mapper与example讲解

**简介：小D商场优惠券功能梳理，每个模块功能梳理**

------

- 数据库查询example是什么？
  * example是mybatis-generator生成的工具包，用于构建查询语句
- 关于example
  * Example类可以用来生成一个几乎无限的where子句.
  * Example类包含一个内部静态类 Criteria 包含一个用 anded 组合在where子句中的条件列表. Example类包含一个 List 属性,所有内部类Criteria中的子句会用 ored组合在一起. 使用不同属性的 Criteria 类允许您生成无限类型的where子句.
  * 创建 Criteria 对象 可以使用Example类中的 createCriteria() 或者 or() . 如果 Criteria 对象是用 createCriteria() 创建的，它会自动为 List 属性添加一个 Criteria 对象 - 这使得它更容易写一个简单的where子句， 如果您不需要 or 或者其他几个子句组合的话. 用 or(Criteria criteria) 方法创建 Criteria 对象, 方法里的 criteria 对象会被添加进 Criteria 对象的列表中.
- 实战example的日常使用





### 第10集    打好基本功才能稳定前进-代码分层规范

**简介：springboot代码分层规范**

------

- 关于代码规范，idea里面安装Alibaba Java Coding Guidelines

![image-20190708224525837](/Users/daniel/Library/Application Support/typora-user-images/image-20190708224525837.png)



- 代码分层的重要性

  * 代码除了实现功能，另外一个重点就是让人看得懂，方便接手
  * 业务分层对于代码规范是比较重要，决定着以后的代码是否可复用，是否职责清晰，边界清晰

  

![image-20190704212445818](/Users/daniel/Library/Application Support/typora-user-images/image-20190704212445818.png)

* Web 层:主要是对访问控制进行转发，各类基本参数校验，或者不复用的业务简单处理等。
* Service 层:相对具体的业务逻辑服务层
* DAO 层:数据访问层，与底层 MySQL、Oracle、Hbase 进行数据交互



 

**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第五章   分布式系统RPC远程调用

### 

### 第1集   互联网不可不知的RPC远程调用协议？

**简介：RPC远程过程调用知识普及，互联网行业程序员不可不知的RPC**

* RPC是什么？
  * RPC是指远程过程调用，也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据

* RPC怎么做
  * 连接：通过在客户端和服务器之间建立TCP连接，远程过程调用的所有交换的数据都在这个连接里传输
  * 寻址：A服务器上的应用怎么告诉底层的RPC框架，如何连接到B服务器（如主机或IP地址）以及特定的端口
  * 编码：网络协议是基于二进制的，内存中的参数的值要序列化成二进制的形式，也就是序列化（Serialize）或编组（marshal），通过寻址和传输将序列化的二进制发送给B服务器
  * 解码：B服务器收到请求后，需要对参数进行反序列化（序列化的逆操作），恢复为内存中的表达方式，然后找到对应的方法（寻址的一部分）进行本地调用，然后得到返回值
  * 处理返回值：返回值还要发送回服务器A上的应用，也要经过序列化的方式发送，服务器A接到后，再反序列化，恢复为内存中的表达方式，交给A服务器上的应用

![image-20190706183215902](/Users/daniel/Library/Application Support/typora-user-images/image-20190706183215902.png)

* RPC的协议有很多，比如最早的CORBA，Java RMI，Web Service的RPC风格，Hessian，Thrift，甚至Rest API



### 第2集   Dubbo是什么？为什么我们要用Dubbo

**简介：是什么？为什么？怎么做？从学习三要素出发带你重新分析Dubbo**

* Dubbo是什么？

    Apache Dubbo 是一个高性能，轻量级，基于Java的RPC框架。Dubbo提供三个关键功能，包括基于接口的远程调用，容错和负载平衡以及自动服务注册和发现。

* Dubbo功能分析

  ![image-20190709162325361](/Users/daniel/Library/Application Support/typora-user-images/image-20190709162325361.png)

  

* 调用功能指责

  * 服务容器负责启动，加载，运行服务提供者
  * 服务提供者在启动时，向注册中心注册自己提供的服务。
  * 服务消费者在启动时，向注册中心订阅自己所需的服务。
  * 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
  * 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
    * 软负载均衡区分于硬负载均衡（例如F5硬负均衡器），是一种基于软件实现的负载均衡(例如:nginx负载均衡)
  * 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心

    * map(调用id，times)  map(调用id,int)，防止于集装箱进行汇总再上报到注册中心

### 第3集   ZooKeeper是什么？

**简介：是什么？为什么？怎么做？从学习三要素出发带你重新认识ZooKeeper**

* Zookeeper是什么？

  * ZooKeeper是一种集中式服务，用于维护配置信息，命名，提供分布式同步和提供组服务

* Zookeeper在Dubbo中的主要功能

  * 当提供者出现断电等异常停机时，注册中心能自动删除提供者信息。
  * 当注册中心重启时，能自动恢复注册数据，以及订阅请求。
  * 当会话过期时，能自动恢复注册数据，以及订阅请求。
  * 当设置<dubbo:registry check="false" />时，记录失败注册和订阅请求，后台定时重试。

  

  ![image-20190709172941241](/Users/daniel/Library/Application Support/typora-user-images/image-20190709172941241.png)





### 第4集   ZooKeeper的搭建

**简介：手把手进行ZooKeeper在linux环境下的搭建过程**

* linux用wget

* ZooKeeper下载

  * ```powershell
    下载地址：http://www.apache.org/dyn/closer.cgi/zookeeper
    ```

* ZooKeeper安装

    * ```powershell
       tar -zxvf zookeeper-3.4.8.tar.gz
       ```

* 配置在“conf”目录下，新建一个名为“zoo.cfg”的文件，其中内容如下：

  * ```powershell
    tickTime=2000  
    dataDir= /usr/local/zookeeper/data  (填写自己的data目录)  
    dataLogDir=/usr/local/zookeeper/logs  
    clientPort=2181
    ```

* ZooKeeper启动

  * sudo ./bin/zkServer.sh start
* 检查启动端口

  * netstat -an|grep 2181

* 参数解析

  * tickTime：以毫秒为单位，这个时间作为 Zookeeper 服务器之间或客户端之间维持心跳的时间间隔
  * dataDir：存储内存中数据库快照的位置，顾名思义就是 Zookeeper 保存数据的目录，默认情况下，Zookeeper 将写数据的日志文件也保存到这个目录里

  - clientPort：这个端口就是客户端连接 Zookeeper 服务器的端口，Zookeeper 会监听这个端口，接受客户
  - 端的访问请求





### 第5集   微服务用户系统搭建

**简介：基于xdclass-coupon进行用户系统的搭建**

* 实战企业级新建项目全过程

  * 考呗项目，项目项目名和module名称

    ![image-20190712125222835](/Users/daniel/Library/Application Support/typora-user-images/image-20190712125222835.png)

  

  * 全局搜索替换pom.xml里面的coupon关键字为user，coupon==》user

  

  ![image-20190712125514589](/Users/daniel/Library/Application Support/typora-user-images/image-20190712125514589.png)

  

  * 修改启动类型名称CouponAppApplication ==》UserAppApplicaiton

  ![image-20190712125641344](/Users/daniel/Library/Application Support/typora-user-images/image-20190712125641344.png)

  

  * 更新依赖，重新刷新项目以及install

* 从实战出发解决企业级项目搭建过程

* 遇到问题，解决问题，优秀的程序员不需要慌







### 第6集   微服务用户系统之实战获取用户信息(上)

**简介：基于xdclass-coupon进行用户系统的开发**

- 生成用户表SQL，企业级单表操作，互联网服务几乎不会用到物理外键

  

```sql
CREATE TABLE `t_user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `account` varchar(255) NOT NULL DEFAULT '' COMMENT '账号',
  `address` varchar(50) NOT NULL DEFAULT '' COMMENT '地址',
  `password` varchar(64) NOT NULL DEFAULT  COMMENT '密码',
  `phone` char(11) DEFAULT NULL  COMMENT '电话号码',
  `point` int(11) DEFAULT NULL COMMENT '积分值',
  `remark` varchar(50) NOT NULL DEFAULT '',
  `tel_phone` char(11) NOT NULL DEFAULT '' COMMENT '备份电话',
  `username` varchar(15)NOT NULL DEFAULT '' COMMENT '用户昵称',
  `zip_code` varchar(6) NOT NULL DEFAULT '' COMMENT '邮政编码',
  `balance` bigint(20) DEFAULT NULL COMMENT '账户金额',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=5 DEFAULT CHARSET=utf8 COMMENT '用户基础信息表';
```



- 通过user表生成user项目反向工程

  



### 第7集   微服务用户系统之实战获取用户信息(下)

**简介：基于xdclass-coupon进行用户系统的开发**

- 验证反向工程代码
- 新建UserController、UserService对外提供getUserById接口服务
- 阿里开发者规范注释
  * 类注释需要加@author标签和日期
  * 方法注释需要加方法用途(自动生成的方法除外)





 

**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第六章   微服务远程调用系统实战




### 第1集   user与coupon基于dubbo协议调用梳理

**简介：基于xdclass-user进行Dubbo Provider的搭建**

* coupon服务对user服务的RPC调用梳理
* user-service-api提供公共jar包
  * user-app实现jar包里面的接口
  * coupon-app调用jar包的接口不关心jar包实现
* user-service-api打包过程







![coup与user rpc调用数据流向图](/Users/daniel/Desktop/material/优惠券原图/coup与user rpc调用数据流向图.jpg)





### 第2集   Springboot整合Dubbo Provider的配置

**简介：基于xdclass-user进行Dubbo Provider的搭建**

* 引入pom依赖

```powershell
       <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.7.2</version>
            <exclusions>
                <exclusion>
                    <artifactId>spring</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo -->

        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.10</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.0.1</version>
        </dependency>
```

* 修改配置文件

```yml
# dubbo config
# 应用定义了提供方应用信息，用于计算依赖关系；在 dubbo-admin 或 dubbo-monitor 会显示这个名字，方便辨识
dubbo:
  application:
    name: user-app

  # 使用 zookeeper 注册中心暴露服务，注意要先开启 zookeeper
  # 注册中心id
  registry:
    id: zookeeper-registry
    # 注册中心协议
    protocol: zookeeper
    # 注册中心地址
    address: 127.0.0.1:2181

  # dubbo协议在20880端口暴露服务
  # 协议名称
  protocol:
    name: dubbo
    # 协议端口
    port: 20880
    # 协议访问log
    accesslog: dubbo-access.log
  # 重试次数
  provider:
    retries: 0
    # 超时时间
    timeout: 3000
  # 注册监控中心
  monitor:
    protocol: registry
```

* 开启注解

```java
  @EnableDubboConfig
  @DubboComponentScan("com.xdclass.xdclassdubboproviders.service")
```

* 进行@Service注解扫描  @org.apache.dubbo.config.annotation.Service
* 序列化传输对象   
* 对比加入dubbo依赖和没加依赖前后的区别   
  * dubbo跑起来了
  * ZK连接上了
  * dubbo端口和http端口同时跑起来了





### 第3集   Springboot整合Dubbo Comsumer的配置以及注意项

**简介：基于xdclass-coupon进行Dubbo Comsumer的搭建**

* 引入user-service-api的jar包
* 引入pom依赖

```powershell
        <!-- 新增 dubbo 依赖 -->
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo</artifactId>
            <version>2.7.2</version>
            <exclusions>
                <exclusion>
                    <artifactId>spring</artifactId>
                    <groupId>org.springframework</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>org.apache.zookeeper</groupId>
            <artifactId>zookeeper</artifactId>
            <version>3.4.10</version>
            <exclusions>
                <exclusion>
                    <artifactId>slf4j-log4j12</artifactId>
                    <groupId>org.slf4j</groupId>
                </exclusion>
            </exclusions>
        </dependency>
        <dependency>
            <groupId>com.github.sgroschupf</groupId>
            <artifactId>zkclient</artifactId>
            <version>0.1</version>
        </dependency>
        <dependency>
            <groupId>org.apache.curator</groupId>
            <artifactId>curator-recipes</artifactId>
            <version>4.0.1</version>
        </dependency>
```

- 修改配置文件

```yml
dubbo:
  application:
    name: user-coupon
  # 使用 zookeeper 注册中心暴露服务，注意要先开启 zookeeper
  # 注册中心id
  registry:
    id: zookeeper-registry
    # 注册中心协议
    protocol: zookeeper
    # 注册中心地址
    address: 127.0.0.1:2181

  # dubbo协议在20880端口暴露服务
  # 协议名称
  protocol:
    name: dubbo
    # 协议端口
    port: 20881
    # 协议访问log
    accesslog: dubbo-access.log
  # 重试次数
  provider:
    retries: 0
    # 超时时间
    timeout: 3000
  # 注册监控中心
  monitor:
    protocol: registry
```

- 开启注解

```java
@EnableDubboConfig
@DubboComponentScan("com.xdclass.userapp.service.dubbo")
```

- 在调用的地方加入注解@Reference
  * com.alibaba.dubbo.config.annotation.Reference






**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第七章   Dubbo可视化后台实战课题



### 第1集   Dubbo调用可视化

**简介：dubbo调用可视化后台dubbo-admin搭建**

- Dubbo服务启动情况、存活节点情况问题

  * 如果某个节点挂了之后我们怎么知道
  * dubbo节点地址有哪些
  * 如果要对某个节点关闭外部rpc调用怎么做？

- Dubbo-admin搭建过程

  * 下载dubbo-admin 

    ```shell
    git clone https://github.com/apache/dubbo-admin.git
    ```

    

  * build  

    ```powershell
    mvn clean package
    ```

  * Start

    ```powershell
    cd dubbo-admin-distribution/target
    nohup java -jar dubbo-admin-0.1.jar &
    ```

  * Visit `http://localhost:8080`

   






### 第2集   Dubbo Admin元数据上报

**简介：dubbo admin的使用**

* 修改注册中心地址和启动地址为7000

  * Specify registry address in `dubbo-admin-server/src/main/resources/application.properties`

* kill 8080端口，重新用nohup启动

* Visit `http://localhost:7000`

* 元数据上报配置，添加bean MetadataReportConfig

  

```java
    @Bean
    public MetadataReportConfig metadataReportConfig() {
        MetadataReportConfig metadataReportConfig = new MetadataReportConfig();
        metadataReportConfig.setAddress("zookeeper://127.0.0.1:2181");
        return metadataReportConfig;
    }
```





### 第3集   微服务单机实现集群部署

**简介：集群部署的使用**

* 什么是集群？

  * 单机处理到达瓶颈的时候，你就把单机复制几份，这样就构成了一个“集群”。集群中每台服务器就叫做这个集群的一个“节点”，所有节点构成了一个集群

  ![preview](https://pic2.zhimg.com/v2-e628e972ac34b597ba2c1f7f0d326705_r.jpg)

* 单机如何搭建集群

  * 服务对外提供的是端口，端口状态是监听状态（listen），每个服务器的对外端口可以非常多，通过反向代理进行端口映射
* 手把手实战多coupon和user dubbo consumer和provider的调用







### 第4集   Dubbo Admin的日常使用

**简介：dubbo admin的使用**

- 使用dubbo admin查看消费组和提供组节点
- 使用dubbo admin完成后台接口测试
- 使用dubbo admin设置请求权重
- 使用dubbo admin添加黑名单
- dubbo负载均衡策略
  - 轮询调度算法Round Robin Scheduling
    - 轮询调度算法的原理是每一次把来自用户的请求轮流分配给内部中的服务器，从1开始，直到N(内部服务器个数)，然后重新开始循环。算法的优点是其简洁性，它无需记录当前所有连接的状态，所以它是一种无状态调度。  
  - 最少活跃调用数 LeastActive LoadBalance
    - 相同活跃数的随机，活跃数指调用前后计数差。
    - 使慢的提供者收到更少请求，因为越慢的提供者的调用前后计数差会越大





### 第5集   基于JVM的ShutdownHook优雅关闭

**简介：shutdownHook优雅关闭讲解**

- 服务启动端口冲突解决方案   Address already in use

  - 修改启动端口指定server.port
  - lsof -i:8080/netstat -anp|grep  8080  ==>找到启动端口对应的服务进程pid==》 kill -9执行进程pid强杀
  - kill -15配合ShutdownHook实现优雅关闭

- kill命令  Linux 中 kill 指令负责杀死进程，其后可以紧跟一个数字，代表**信号编号**(Signal)

    

  ```shell
  kill -l                                                                     
  HUP INT QUIT ILL TRAP ABRT BUS FPE KILL USR1 SEGV USR2 PIPE ALRM TERM STKFLT CHLD CONT STOP TSTP TTIN TTOU URG XCPU XFSZ VTALRM PROF WINCH POLL PWR SYS
  ```

  

- JAVA进程优雅关闭的意义

  - 关闭 socket 链接
  - 清理临时文件
  - 发送消息通知给订阅方，告知自己下线
  - 各种资源的释放   jerkins钩子 ，往git提交代码==》执行了自动化构建==》服务重启 

- 实战spring实战 implements DisposableBean 优雅关闭



### 第6集   互联网经典面试题分析之为什么有了Http还要用dubbo

**简介：还原蚂蚁金服Dubbo相关面试题**

* 问题分析流程，面试官所问的这个问题包含的哪些知识点？
  * dubbo的定义(定位)
  * rpc相对于http的优势

  ![image-20190717183603564](/Users/daniel/Library/Application Support/typora-user-images/image-20190717183603564.png)

* 这个问题应该从哪些方面着手去回答

  * 通用定义的http1.1协议的tcp报文包含无用信息，一个POST协议的格式大致如下(数据大小)

    ```shell
    HTTP/1.0 200 OK 
    Content-Type: text/plain
    Content-Length: 137521
    Expires: Thu, 05 Dec 2019 16:00:00 GMT
    Last-Modified: Wed, 5 August 2019 15:55:28 GMT
    Server: Apache 0.84
    
    <html>
      <body>Hello xdclass</body>
    </html>
    
    ```

  * RPC封装了“服务发现”，"负载均衡"，“熔断降级”一类面向服务的高级特性，这些是http做不到的(RPC特色)

  * 从个人使用经验来讲RPC调用还拥有传输安全的优势，防止了Http调用的数据包篡改和流量劫持(个人经验)

* 技巧总结==》不经意之间测漏自己是有相关技术实战经验的开发人员





**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第八章   优惠券列表实战开发

### 第1集    小D商城功能优惠券列表开发

**简介：小D商场优惠券功能梳理，每个模块功能梳理**

------

- 优惠券列表功能梳理
- 设计先行，提供dubbo API接口
- 对于用户而已==》读优惠券
- 对于运营人员而已==》写优惠券

![优惠券列表逻辑梳理](/Users/daniel/Desktop/material/优惠券原图/优惠券列表逻辑梳理.jpg)



### 第2集    优惠券列表开发

**简介：优惠券列表功能开发**

------

- 优惠券列表开发
- 获取优惠券系统接口
- 接口DTO与VO在开发中的规范讲解

* * DO（ Data Object）：与数据库表结构一一对应，通过DAO层向上传输数据源对象。

  * DTO（ Data Transfer Object）：数据传输对象，Service或Manager向外传输的对象。
  * VO（ View Object）：显示层对象，通常是Web向模板渲染引擎层传输的对象。





### 第3集   高级程序员必会--JMH基准测试

**简介：带你从0到1认识基准测试JMH**

------

- 基准测试的意义
  * 对业务模型中的重要业务做单独的测试，获取单用户运行时的各项性能指标，为多用户**并发测试**和**综合场景测试**等**性能分析**提供参考依据
  * 举个例子 list.contain ==>set.contain ==> 布隆过滤器

![带你认识JMH](/Users/daniel/Desktop/material/优惠券原图/带你认识JMH.jpg)  

* JMH是什么？
  - JMH，即Java Microbenchmark Harness，这是专门用于进行代码的微基准测试的一套工具API，JMH 由 OpenJDK/Oracle 里面那群开发了 Java 编译器的大牛们所开发

- JMH典型使用场景
  * 已经找出了热点函数，而需要对热点函数进行进一步的优化时，就可以使用 JMH 对优化的效果进行定量的分析。
  * 想定量地知道某个函数需要执行多长时间，以及执行时间和输入 n 的相关性
  * 一个函数有两种不同实现（例如JSON序列化/反序列化有Jackson和Gson实现），不知道哪种实现性能更好

  

  

### 第4集   JMH基准测试HelloWorld

**简介：带你手把手进行springboot和JMH的整合**

  

* 基准测试引入pom依赖

  

  ```powershell
      <dependency>
           <groupId>org.openjdk.jmh</groupId>
           <artifactId>jmh-core</artifactId>
           <version>1.21</version>
      </dependency>
      <dependency>
           <groupId>org.openjdk.jmh</groupId>
           <artifactId>jmh-generator-annprocess</artifactId>
           <version>1.21</version>
           <scope>provided</scope>
      </dependency>
  
  ```

  

* 基准测试JMH helloWorld编写

  - 类似于mybatis-generator的配置方式
  - 第一步配置pom依赖
  - 第二步复制工具类
  - @Benchmark方法所注释的类为我们所要进行测试的类







### 第5集   Springboot整合JMH基准测试

**简介：带你手把手进行springboot和JMH的整合**

------



- 关键参数分析

  * `warmupIterations(10)`的意思是预热做10轮，

  * `measurementIterations(10)`代表正式计量测试做10轮，而每次都是先执行完预热再执行正式计量，内容都是调用标注了`@Benchmark`的代码

  * `forks(3)`指的是做3轮测试，因为一次测试无法有效的代表结果，所以通过3轮测试较为全面的测试，而每一轮都是先预热，再正式计量

    

- 实战springboot整合JMH过程

- 对比springboot整合JMH和Main方法直接跑JMH的区别在哪里

  * springboot容器只被初始化一次
  * 如果多次初始化springboot容器会造成端口被占用







### 第6集   数字化分析执行性能

**简介：数字化分析单机执行数据**

------



- JMH结果值参看
- 通过druid对照接口，查看sql统计
- <http://localhost:8088/druid/sql.html>







### 第7集   系统性能调优指南

**简介：系统性能调优指南**

------

- <http://localhost:8088/druid/sql.html>

- 代码性能差分析思路

  



![接口性能优化指南](/Users/daniel/Desktop/material/优惠券原图/接口性能优化指南.jpg)



 

**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第九章   互联网高级必会之进程缓存



### 第1集    分析Mysql执行过程

**简介：mysql执行过程分析**

------

- mysql传统关系型数据库

  - 概念

    关系型数据库的一个常见用法是存储长期的报告数据，并将这些报告数据用作固定时间范围内的聚合数据。收集聚合数据的常见做法是：先将各个行插入到一个报告表里面， 之后再通过扫描这些行来收集聚合数据， 并更新聚合表中巳有的那些行。

  - 图解剖析mysql的执行过程



![mysql执行细节](/Users/daniel/Desktop/data/redis第一季课堂资料/第2章/第1节/mysql执行细节.png)





### 第2集        良药苦口缓存的收益和成本



- 缓存是什么？

  * 起源于CPU，原始意义是指访问速度比一般[随机存取存储器](https://baike.baidu.com/item/%E9%9A%8F%E6%9C%BA%E5%AD%98%E5%8F%96%E5%AD%98%E5%82%A8%E5%99%A8)（RAM）快的一种RAM，通常它不像系统主存那样使用[DRAM](https://baike.baidu.com/item/DRAM)技术，而使用昂贵但较快速的[SRAM](https://baike.baidu.com/item/SRAM)技术

- 缓存类别

  * CPU L1/L2/L3 Cache、Linux page Cache加速硬盘读写、浏览器缓存、堆内缓存、分布式缓存

- 缓存带来的回报

  - 高速读写
    - 缓存加速读写速度
  - 降低后端负载
    - 后端服务器通过前端缓存降低负载： 业务端使用Redis降低后端MySQL负载等  

- 缓存带来的代价

  - 数据不一致  

    - 缓存层和数据层有时间窗口不一致，和更新策略有关      

  - 代码维护成本

    - 原本只需要读写MySQL就能实现功能，但加入了缓存之后就要去维护缓存的数据，增加了代码复杂度。

  - 堆内缓存可能带来内存溢出的风险影响用户进程，如ehCache、loadingCache 

    xmx

- 堆内缓存一般性能更好，远程缓存需要套接字传输
  - 用户级别缓存尽量采用远程缓存
  - 大数据量尽量采用远程缓存，服务节点化原则





### 第3集   缓存怎么做？

**简介：缓存怎么做，怎么加到优惠券列表里面**

------

- 优惠券列表基于缓存的优化

- 缓存的使用，先从缓存查数据，如果查不到数据再从数据库刷数据到缓存

- 缓存使用流程

    

![s缓存接入流程](/Users/daniel/Desktop/material/优惠券原图/缓存接入流程.jpg)









### 第4集    Google GauvaCache的使用

**简介：优惠券列表堆内缓存优化**

------

- 优惠券列表基于loadingCache优化

- loadingCache接入

  ```powershell
     <!-- https://mvnrepository.com/artifact/com.google.guava/guava -->
     <dependency>
        <groupId>com.google.guava</groupId>
        <artifactId>guava</artifactId>
        <version>28.0-jre</version>
     </dependency>
  ```

  





### 第5集    Google GuavaCache常用api

**简介：Google GauvaCache常用api有哪些**

------

- loadingCache常用api讲解





### 第6集    Google GuavaCache实战批量优惠券ID接口

**简介： Google GauvaCache实战批量优惠券ID接口**

------

- 批量获取优惠券列表怎么实现？
- 这个常用的api常见怎么用guava缓存



![GuavaCache通过批量ID数据接口](/Users/daniel/Desktop/material/优惠券原图/GuavaCache通过批量ID数据接口.jpg)





 

**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十章   实战探索Springboot定时任务机制



## 第1集   LoadingCache与Map的对比

**简介：深度探索map与loadingCache区别**

------

- 深入底层探索，为什么要用LoadingCache，不可以用Map来存放数据吗?

 ![Map与LoadingCache区别](/Users/daniel/Desktop/material/优惠券原图/Map与LoadingCache区别.jpg)

* 为什么要选择ConcurrentHashMap而不是HashMap

  * HashMap存储底层

    ![2](/Users/daniel/Desktop/material/优惠券原图/2.jpeg)

  * ConcurrentHashMap存储底层

  ​                  ![1](/Users/daniel/Desktop/material/优惠券原图/1.jpeg)  

​         





### 第2集   使用springboot定时任务

**简介：springboot框架整合定时任务处理数据异步更新**

------

- 基于springboot的定时任务怎么做

- 实战定时任务整合

- 在启动类CouponAppApplication加入注解@EnableScheduling

  ![image-20190728172415713](/Users/daniel/Library/Application Support/typora-user-images/image-20190728172415713.png)

- 在所有执行任务的方法加上注解@Scheduled(cron = "0/5 * * * * ?")

  ![image-20190728172505996](/Users/daniel/Library/Application Support/typora-user-images/image-20190728172505996.png)









### **第3集   基于springboot的数据更新**

**简介：springboot实战优惠券列表数据异步更新**

------

- 基于springboot的实战优惠券列表异步更新

​      

![image-20190728215459703](/Users/daniel/Library/Application Support/typora-user-images/image-20190728215459703.png)



![image-20190728215547245](/Users/daniel/Library/Application Support/typora-user-images/image-20190728215547245.png)





### 第4集   使用springboot集成logback配置

**简介：springboot框架整合定时任务处理数据异步更新**

------

- logback日志框架接入实战
- 接入流程
  * 引入xml
  * 加入yml配置









### 第5集   logback日志打印规范

**简介：logback日志打印规范**

------

- logback日志规范
  * 关键字原则 ，方便通过grep查询
  * 限制大小原则
  * 数据脱敏原则







### 第6集   实战springboot整合JSON

**简介：springboot整合json实战**

------

- springboot整合json实战演练
- 引入pom依赖
- 进行日志打印



```powershell
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.58</version>
</dependency>
```







### 第7集   基于JMH探索缓存的影响

**简介：JMH实验对照加缓存前后的变化**

------

- JMH实验对照加缓存前后的变化
- 实战JMH
- 分析JMH数字化结果



![image-20190729230242453](/Users/daniel/Library/Application Support/typora-user-images/image-20190729230242453.png)







 

**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十一章   内存淘汰算法读写性能原理



### 第1集   内存缓存淘汰算法

**简介：内存缓存淘汰算法的作用**

------

- 内存缓存淘汰算法是什么？
- 为什么要用淘汰算法
- 有哪些淘汰算法
  * FIFO
  * LRU
  * LFU
  * W-Tiny-LFU

* FIFO:先进先出

  * 在这种淘汰算法中，先进入缓存的会先被淘汰

  * 命中率很低













### 第2集   缓存淘汰算法LRU算法

**简介：精雕细刻缓存淘汰算法LRU算法**

------

- LRU算法是什么？
  * Least recently used，最近最少使用get
- 为什么要用LRU算法
  * 根据数据的历史访问记录来进行淘汰数据，其核心思想是“如果数据最近被访问过，那么将来被访问的几率也更高”
- LRU算法原理剖析   

![image-20190727153717127](/Users/daniel/Library/Application Support/typora-user-images/image-20190727153717127.png)







### 第3集    缓存淘汰算法LFU算法

**简介：缓存淘汰算法LRU算法实战**

------

- LFU算法是什么？
  * Least Frequently Used
- 为什么要用LFU算法
  * 算法根据数据的历史访问频率来淘汰数据，其核心思想是“如果数据过去被访问多次，那么将来被访问的频率也更高”
- LFU算法原理剖析
  *  新加入数据插入到队列尾部（因为引用计数为1）
  * 队列中的数据被访问后，引用计数增加，队列重新排序；
  * 当需要淘汰数据时，将已经排序的列表最后的数据块删除。

![image-20190730230257056](/Users/daniel/Library/Application Support/typora-user-images/image-20190730230257056.png)

* LFU的缺点
  * 复杂度
  * 存储成本
  * 尾部容易被淘汰





### 第4集  来自未来的缓存Caffine介绍

**简介：来自未来的缓存Caffine**

------

- Caffine是什么?
  * Caffeine是一个基于Java 8的[高性能](https://github.com/ben-manes/caffeine/wiki/Benchmarks)，[接近最佳的](https://github.com/ben-manes/caffeine/wiki/Efficiency)缓存库
- Caffine有什么特色？
  - [自动将条目加载](https://github.com/ben-manes/caffeine/wiki/Population)到缓存中，可选择异步[加载](https://github.com/ben-manes/caffeine/wiki/Population)
  - 基于[频率和新近度](https://github.com/ben-manes/caffeine/wiki/Efficiency)超过最大值时的[基于大小的驱逐](https://github.com/ben-manes/caffeine/wiki/Eviction#size-based)
  - 自上次访问或上次写入以来测量的条目[的基于时间的到期](https://github.com/ben-manes/caffeine/wiki/Eviction#time-based)
  - 当第一个条目的陈旧请求发生时，[异步刷新](https://github.com/ben-manes/caffeine/wiki/Refresh)
  - 密钥自动包含在[弱引用中](https://github.com/ben-manes/caffeine/wiki/Eviction#reference-based)
  - 值自动包含在[弱引用或软引用中](https://github.com/ben-manes/caffeine/wiki/Eviction#reference-based)
  - 被驱逐（或以其他方式删除）条目的[通知](https://github.com/ben-manes/caffeine/wiki/Removal)
  - [写入传播](https://github.com/ben-manes/caffeine/wiki/Writer)到外部资源
  - 累积缓存访问[统计信息](https://github.com/ben-manes/caffeine/wiki/Statistics)
- W-Tiny-LFU实现原理分析
- Count-Min Sketch。基于滑动窗口的时间衰减设计机制，借助于一种简易的reset操作：每次添加一条记录到Sketch的时候，都会给一个计数器上加1，当计数器达到一个尺寸W的时候，把所有记录的Sketch数值都除以2，该reset操作可以起到衰减的作用

![image-20190731083848914](/Users/daniel/Library/Application Support/typora-user-images/image-20190731083848914.png)







![布隆过滤器实现原理](/Users/daniel/Desktop/material/优惠券原图/布隆过滤器实现原理.jpg)















### 第5集  Caffine缓存的使用

**简介：来自未来的缓存Caffine**

------

- Caffine怎么用
- Caffine实战环节







### 第6集  内存缓存对比

**简介：内存缓存产品对比**

------

- 内存缓存分类

- 从淘汰算法和并发性分析内存缓存的利弊

  

![进程缓存对比](/Users/daniel/Desktop/material/优惠券原图/进程缓存对比.png)















### 第7集   手把手阿里面试题LRU算法实现

**简介：缓存淘汰算法LRU算法实战**

------

- LRU算法实战过程








**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十二章   用户领取优惠券功能开发



### 第1集   用户领取优惠券功能梳理

**简介：用户领取优惠券功能梳理**

------

- 用户领券功能需求分析

  

   

![领取优惠券功能](/Users/daniel/Desktop/material/优惠券原图/领取优惠券功能.jpg)

### 第2集   雪花算法介绍

**简介：雪花算法介绍**

------

- 什么是雪花算法

  * 分布式唯一ID生成策略
- 为什么要用雪花算法
  * 同一业务场景要全局唯一
  * 生成速度快性能好  CPU密集型
  * 实现简单没有依赖

- 雪花算法生成原则和原理

  * 使用41bit作为毫秒数，10bit作为机器的ID（5个bit是数据中心，5个bit的机器ID），12bit作为毫秒内的流水号（意味着每个节点在每毫秒可以产生 4096 个 ID），最后还有一个符号位，永远是0

  

  ![image-20190803105240595](/Users/daniel/Library/Application Support/typora-user-images/image-20190803105240595.png)

* 类比法



### 第3集   用户优惠券领券开发(上)

**简介：基于雪花算法,用户优惠券领券开发**

------

- 基于雪花算法,用户优惠券领券开发

  

### 第4集   用户优惠券领券开发(下)

**简介：基于雪花算法,用户优惠券领券开发**

------

- 基于雪花算法,用户优惠券领券开发

  

### 第5集   优惠券系统对外提供Dubbo封装

**简介：优惠券对外提供dubbo服务开发**

------

- 优惠券对外提供dubbo服务开发

  

### 第6集   shop商城调用coupon实战

**简介：shop商城调用coupon封装**

------

- shop商城调用coupon封装

- dubbo引入步骤如果忘记请看第六章第3集

  

 

**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十三章   用户优惠券列表功能开发



### 第1集   用户获取自己的coupon列表实战

**简介： 用户获取自己的coupon列表实战**

------

-  用户获取自己的coupon列表实战

  



### 第2集   用户获取coupon列表信息组装

**简介：用户获取coupon列表信息组装实战**

------

-  用户获取coupon列表信息组装

  



### 第3集   RPC远程调用项目调试

**简介：debug调试解决用户coupon列表问题**

------

- 实战debug调试解决用户coupon列表问题

  



### 第4集   shop调coupon完成用户优惠券列表功能

**简介：shop调coupon完成用户优惠券列表功能**

------

- 实战shop调coupon完成用户优惠券列表功能





**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十四章   优惠券公告栏需求开发



### 第1集   coupon公告栏功能梳理

**简介： coupon广告栏功能梳理**

* 需求：公告栏展示最近10条用户核销优惠券功能
* 收到需求，设计先行
* 公告栏展示用户用券消费
* 通过用户用券消费促进其他用户购买
* 性能调优
  * 分析数据是否落盘入库
  * 数据缓存存储方案
    * 选择什么缓存
    * 怎么存

![coupon公告](/Users/daniel/Desktop/material/优惠券原图/coupon公告.jpg)

### 第2集   使用redis实现coupon公告栏分析

**简介： 使用redis实现coupon公告栏分析**

- 需求：公告栏展示最近10条用户核销优惠券功能
- redis：string、hash、list、sortset     key，value：json        user:{"id":{},"name":{}}    key:  value list []   lpop rpush     sortedSet:排行榜   获取前N条 LRU
- 场景分析：读多写少  只保留前N条数据，当达到N+1条进行删除  10条   1条   zrange
- 每次插入数据时删除最后一条数据    zadd ==>zrem

![image-20190809231448712](/Users/daniel/Library/Application Support/typora-user-images/image-20190809231448712.png)











### 第3集   分布式Redis缓存安装

**简介： redis缓存安装实战**

- Redis安装过程  wget   apt get   

- wget http://download.redis.io/releases/redis-4.0.6.tar.gz 

  ```shell
       tar -zxvf redis-4.0.6.tar.gz
  ```

  ```powershell
       yum install gcc
  ```

  ```shell
       跳转到redis解压目录下 cd redis-4.0.6
  ```

  ```shell
       编译安装 make MALLOC=libc　　
  ```

  ```shell
       测试是否安装成功 
  
          cd src  ./redis-server
  ```

* telnet redis默认端口6379看能否通
* 远程连接redis记得关闭防火墙
* bind 是否已经改成了0.0.0.0









### 第4集   Redis sortedSet功能api讲解

**简介：sortedSet功能api讲解**

- ./redis-cli -h 127.0.0.1 -p  6379
- zadd ——在key对应的zset中添加一个元素
- zrange——获取key对应的zset中指定范围的元素，-1表示获取所有元素
- zrem——删除key对应的zset中的一个元素
- zrangebyscore——返回有序集key中，指定分数范围的元素列表,排行榜中运用
- zrank——返回key对应的zset中指定member的排名。其中member按score值递增(从小到大）；
         排名以0为底，也就是说，score值最小的成员排名为0,排行榜中运用      
- zadd ==》zrem
-    set是通过hashmap存储，key对应set的元素，value是空对象
     sortset是怎么存储并实现排序的呢，hashmap存储，还加了一层跳跃表
     跳跃表：相当于双向链表，在其基础上添加前往比当前元素大的跳转链接 











### 第5集   手把手SpringBoot整合Redis

**简介：springboot整合redis实战**

- 引入bean redisTemplate的使用，类型于：monogoTemplate、jdbcTemplate数据库连接工具

- 配置步骤:

     * 引入pom依赖

- 
  ```powershell
     <dependency>
          <groupId>org.springframework.boot</groupId>
          <artifactId>spring-boot-starter-data-redis</artifactId>
     </dependency>      
  ```

  * 编写redisTemplate类，设置redisConnectFactory

  * 引入配置文件







### 第6集   SpringBoot整合sortedSet api讲解               

**简介： springboot整合sortedSet实现数据操作**

* 本地连不上远程阿里云redis服务问题定位步骤
  * 在redis.conf修改配置参数
    * bind 改为 0.0.0.0   默认只能本机访问 （127.0.0.1）
    * daemonize yes     默认非守护方式启动(no)
    * protected-mode no  默认为保护模式(yes)
  * 启动方式为./redis-server redis.conf 
  * 关闭防火墙
* sortedSet springboot数据操作类ZSetOperations
* 基于ZSetOperations实现redis zrange、zadd、zrem方法实战





### 第7集  Coupon落地方案分析

**简介：Coupon落地方案分析**

- SortedSet是什么？
  * Redis 有序集合和集合一样也是string类型元素的集合,且不允许重复的成员
  * 不同的是每个元素都会关联一个double类型的分数。redis正是通过分数来为集合中的成员进行从小到大的排序 
- 为甚么要用SortedSet?
  * 有序
  * key唯一
  * 性能高
    * 集合是通过哈希表实现的，所以添加，删除，查找的复杂度都是O(1)
- coupon公告栏功能以及Api分析



![coupon公告栏功能](/Users/daniel/Desktop/material/优惠券原图/coupon公告栏功能.JPEG)

* 主要命令Zrem、Zadd、Zrange、Zrevrange   zadd myset score(timestamp) key
* key：需求：展示某某使用了XXX优惠券   userId_couponId   1_1    1_2  1_3   2_3   1_10086  1_10087







### 第8集  手把手Coupon实现公告栏数据展示

**简介： coupon广告栏功能数据展示**

- 手把手Coupon实现公告栏数据展示 ZRange    Timestamp  Score

- 手把手Coupon实现公告栏数据维护   增、删     ZAdd、ZRem

  



### 第9集  手把手Coupon实现公告栏数据维护

**简介： coupon广告栏功能数据维护**

- 手把手Coupon实现公告栏数据维护   增、删     ZAdd、ZRem









### 第10集   实战组装Coupon信息提供Dubbo接口

**简介：实战组装Coupon信息提供Dubbo接口**

- coupon公告栏信息组装
- 通过couponId组装coupon信息





### 第11集   实战Shop整合Coupon实现公告栏

**简介：实战Shop整合coupon实现公告栏**

- 测试coupon提供的dubbo接口是否连通

- shop调用coupon提供的接口数据

  







**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十五章   剖析Jmeter系统测试功能



### 第1集   性能测试功能Jmeter的概述

**简介： 带你了解jmeter**

- 带你了解jmeter测试工具
- Jmeter是什么？
  * Apache JMeter可用于测试静态和动态资源，Web动态应用程序的性能。
    它可用于模拟服务器，服务器组，网络或对象上的重负载，以测试其强度或分析不同负载类型下的整体性能

* 同类型测试工具对比

  ![互联网测试工具对比](/Users/daniel/Desktop/material/优惠券原图/互联网测试工具对比.jpeg) 

* ab -n1000 -c10 http://localhost:8080/getByCache?id=2 

* jmeter下载

```powershell
https://jmeter.apache.org/download_jmeter.cgi   
```

![image-20190818003649060](/Users/daniel/Library/Application Support/typora-user-images/image-20190818003649060.png)

* 点击启动

```java
windows/mac进入jmeter的下载目录的bin文件，找到jmeter启动文件，点击启动
linux ./jmeter/bin/jmeter.sh
```

* 需要配置jdk环境





### 第2集  Jmeter基础功能组件介绍线程组和Sampler

**简介： Jmeter基础功能组件介绍**

- 添加->threads->线程组（控制总体并发）    [请求发起者]

  ```
  	线程数：虚拟用户数。一个虚拟用户占用一个进程或线程
  		
  	准备时长（Ramp-Up Period(in seconds)）：全部线程启动的时长，比如100个线程，20秒，则表示20秒内	   100个线程都要启动完成，每秒启动5个线程
  		
  	循环次数：每个线程发送的次数，假如值为5，100个线程，则会发送500次请求，可以勾选永远循环
  ```

* 添加-> Sampler(采样器)        [请求接受者]

   

  ```
  		名称：采样器名称
  
  		注释：对这个采样器的描述
  		
  		web服务器：
  			默认协议是http
  			默认端口是80
  			服务器名称或IP ：请求的目标服务器名称或IP地址
  
  		路径：服务器URL
  ```

  

* 查看测试结果        [请求结果]

  ```
  		线程组->添加->监听器->察看结果树
  ```

  



### 第3集  Jmeter验证Shop公告栏接口

**简介： Jmeter验证Shop公告栏接口**

- 关于服务端渲染和客户端渲染    移动互联网
- 填写shop公告栏接口
- 创建线程组进行压测请求
- 谷歌扩展程序插件jsonViewer







### 第4集 Jmeter性能压测聚合报告分析

**简介：Jmeter性能压测聚合报告分析**

- 新增聚合报告：线程组->添加->监听器->聚合报告（Aggregate Report）

  ```powershell
      lable: sampler的名称
  		Samples: 一共发出去多少请求,例如10个用户，循环10次，则是 100
  		Average: 平均响应时间
  		Median: 中位数，也就是 50％ 用户的响应时间
  
  		90% Line : 90％ 用户的响应不会超过该时间 （90% of the samples took no more than this time. 		 The remaining samples at least as long as this）
  		95% Line : 95％ 用户的响应不会超过该时间
  		99% Line : 99％ 用户的响应不会超过该时间
  		min : 最小响应时间
  		max : 最大响应时间
  		
  		Error%：错误的请求的数量/请求的总数
  		Throughput： 吞吐量——默认情况下表示每秒完成的请求数（Request per Second) 可类比为qps
  		KB/Sec: 每秒接收数据量
  
  ```

  








### 第5集 蚂蚁金服面试题之服务器性能指标

**简介：Linux服务性能指标讲解**

* 为什么要观察服务器性能指标



![ Linux服务负载](/Users/daniel/Desktop/material/优惠券原图/Linux服务负载.jpeg)

* load average
  * 为什么会有三个数字呢？它们的意思分别是1分钟、5分钟、15分钟内系统的平均负荷。
  * 当CPU完全空闲的时候，平均负荷为0,值越低系统负荷越低
  * 值跟CPU核数相关，比如8核CPU，最高负载为8.0
  * grep -c 'model name' /proc/cpuinfo"命令，直接返回CPU的总核心数

* mem
  * 展示了当前内存的状态，total是总的内存大小，userd是已使用的，free是剩余的，buffers是目录缓存
* Task
  * 展示了目前的进程总数及所处状态，要注意zombie，表示僵尸进程，不为0则表示有进程出现问题
* Cpu
* * 展示了当前CPU的状态，us表示用户进程占用CPU比例，sy表示内核进程占用CPU比例，id表示空闲CPU百分比，wa表示IO等待所占用的CPU时间的百分比。**wa占用超过30%则表示IO压力很大**。











**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十六章   深入理解coupon优惠券核销功能



### 第1集  coupon优惠券核心功能梳理

**简介： coupon优惠券核心功能梳理**

- coupon优惠券核心功能梳理

- 业务步骤： 用户触发订单支付  ==》订单支付成功  ==》coupon用户券核销 ==》

  更新优惠券使用状态 ==》更新公告栏最新优惠流水

* 流程梳理  



  ![优惠券核销功能分析1](/Users/daniel/Desktop/material/优惠券原图/优惠券核销功能分析1.jpeg)

  





### 第2集  手把手coupon优惠券核销功能实战

**简介：coupon优惠券核销功能实战**

- coupon优惠券核销功能
- 对于coupon系统来说核销功能的开发功能点
  * 接入Rocketmq，梳理MQ接入消息体 
  * 处理coupon数据库更新状态为已核销状态
  * 维护公告栏数据

  

### 第3集   JMS消息服务以及运用模型

**简介： JMS消息服务与消息中间件概念普及**

- JMS消息服务是什么？
  *  Java消息服务（Java Message Service),提供创建，发送和读取消息的工具。它提供松耦合，可靠和异步通信。

* 为什么要用JMS
  * 1）**异步：**要接收消息，客户端不需要发送请求。消息将自动到达客户端
  * 2）**可靠：**它保证了消息的传递     shop @Async  ==>  (出现网络异常)coupon
* 运用模型

  * 点对点模型

    ![image-20190819125348207](/Users/daniel/Library/Application Support/typora-user-images/image-20190819125348207.png)

  * 订阅模型

     ![image-20190819125409536](/Users/daniel/Library/Application Support/typora-user-images/image-20190819125409536.png)

​                

### 第4集   JMS常见概念以及消息中间件

**简介：  JMS常见概念以及消息中间件**

* 常见概念
  - JMS提供者：连接面向消息中间件的，JMS接口的一个实现，RocketMQ,ActiveMQ,Kafka等等
  - JMS生产者(Message Producer)：生产消息的服务
  - JMS消费者(Message Consumer)：消费消息的服务
  - JMS消息：数据对象
  - JMS队列：存储待消费消息的区域
  - JMS主题：一种支持发送消息给多个订阅者的机制
  - JMS消息通常有两种类型：点对点（Point-to-Point)、发布/订阅（Publish/Subscribe）
* 基础编程模型
  - MQ中需要用的一些类
  - ConnectionFactory ：连接工厂，JMS 用它创建连接
  - Connection ：JMS 客户端到JMS Provider 的连接
  - Session： 一个发送或接收消息的线程
  - Destination ：消息的目的地;消息发送给谁.
  - MessageConsumer / MessageProducer： 消息消费者，消息生产者
* JMS编程模型
  * ![image-20190819125659964](/Users/daniel/Library/Application Support/typora-user-images/image-20190819125659964.png)

* 常见消息中间件
  * Kafka   
    * 是由Apache软件基金会开发的一个开源流处理平台，由Scala和Java编写。Kafka是一种高吞吐量的分布式发布订阅消息系统，它可以处理大规模的网站中的所有动作流数据(网页浏览，搜索和其他用户的行动)，副本集机制，实现数据冗余，保障数据尽量不丢失；支持多个生产者和消费者
  * RocketMQ
    * 阿里开源的一款的消息中间件, 纯Java开发，具有高吞吐量、高可用性、适合大规模分布式系统应用的特点, 性能强劲(零拷贝技术)，支持海量堆积, 支持指定次数和时间间隔的失败消息重发,支持consumer端tag过滤、延迟消息等，在阿里内部进行大规模使用，适合在电商，互联网金融等领域使用
  * RabbitMQ
    * 在国内一些互联网公司近几年用rabbitmq也比较多一些   但是问题也是显而易见的，RabbitMQ确实吞吐量会低一些，这是因为他做的实现机制比较重
* 3种消息中间件各有优劣，在国内使用比重相差不大，但同类型产品一般只需深刻掌握一种，并了解不同产品的区别



### 第5集 RocketMQ4.x本地源码部署

**简介:RocketMQ4.x本地快速部署**

* RocketMQ核心组件

    * Broker：MQ程序，接收生产的消息，提供给消费者消费的程序

    * Name Server：给生产和消费者提供路由信息，提供轻量级的服务发现、路由、元数据信息，可以多个部署，互相独立（比zookeeper更轻量）

      

* 安装前提条件(推荐) 64bit OS, Linux/Unix/Mac (Windows不兼容) 64bit JDK 1.8+;

* 快速开始 <http://rocketmq.apache.org/docs/quick-start/> 下载安装包：<http://mirror.bit.edu.cn/apache/rocketmq/4.4.0/rocketmq-all-4.4.0-source-release.zip>

  ```
  unzip rocketmq-all-4.4.0-source-release.zip
  cd rocketmq-all-4.4.0/
  mvn -Prelease-all -DskipTests clean install -U
  cd distribution/target/apache-rocketmq
  ```

  

* 最新版本部署存在问题：

  * Please set the JAVA_HOME variable in your environment, We need java(x64)
  * 解决：本地需要配置 JAVA_HOME 使用命令 vim ~/.bash_profile

  ```
  JAVA_HOME="/Library/Java/JavaVirtualMachines/jdk1.8.0_171.jdk/Con
  export JAVA_HOME
  CLASS_PATH="$JAVA_HOME/lib"
  PATH=".$PATH:$JAVA_HOME/bin"
  ```

​    

* 启动nameServer

- ```
  nohup sh bin/mqnamesrv &
  ```

  

- 查看日志 tail -f nohup.out (结尾：The Name Server boot success. serializeType=JSON 表示启动成功)

- 启动broker (-n指定nameserver地址，nameserver服务端口为9876, broker默认端口 10911)

  ```
  nohup sh bin/mqbroker -n localhost:9876 &
  ```

  

- 关闭nameserver broker执行的命令

  ```
  sh bin/mqshutdown broker
  ```

  ```
  sh bin/mqshutdown namesrv
  ```

  

- 使用 jps查看进程

​       





**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十七章   coupon优惠券核销功能实战



### 第1集  手把手shop接入rocketmq producer

**简介：手把手shop接入rocketmq producer实战**

- 接入消息配置

- 接入pom依赖

  ```powershell
  <dependency>
       <groupId>org.apache.rocketmq</groupId>
       <artifactId>rocketmq-client</artifactId>
       <version>4.3.0</version>
  </dependency>
  ```

* 配置properties

  ```powershell
  rocketmq.producer.groupName=shop
  rocketmq.producer.namesrvAddr=127.0.0.1:9876
  rocketmq.producer.default=true
  ```

  

* 引入bean

  ```powershell
  /**
       * 创建普通消息发送者实例
       *
       * @return
       * @throws MQClientException
       */
      @Bean
      public DefaultMQProducer defaultProducer() throws MQClientException {
          log.info(producerConfigure.toString());
          log.info("defaultProducer 正在创建---------------------------------------");
          DefaultMQProducer producer = new DefaultMQProducer(producerConfigure.getGroupName());
          producer.setNamesrvAddr(producerConfigure.getNamesrvAddr());
          producer.setVipChannelEnabled(false);
          producer.setRetryTimesWhenSendAsyncFailed(10);
          producer.start();
          log.info("rocketmq producer server开启成功---------------------------------.");
          return producer;
      }
  ```

  

### 第2集  手把手coupon接入rocketmq consumer

**简介：手把手shop接入rocketmq consumer实战**

- 接入消息配置

- 接入pom依赖

  ```powershell
  <dependency>
       <groupId>org.apache.rocketmq</groupId>
       <artifactId>rocketmq-client</artifactId>
       <version>4.3.0</version>
  </dependency>
  ```

- 配置properties

  ```powershell
  rocketmq.producer.groupName=shop
  rocketmq.producer.namesrvAddr=127.0.0.1:9876
  rocketmq.producer.default=true
  ```

  

* 配置consumer消费者

   

  ```java
  import org.apache.rocketmq.client.consumer.DefaultMQPushConsumer;
  import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyContext;
  import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
  import org.apache.rocketmq.client.consumer.listener.MessageListenerConcurrently;
  import org.apache.rocketmq.client.exception.MQClientException;
  import org.apache.rocketmq.common.message.MessageExt;
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  import org.springframework.beans.factory.annotation.Value;
  import org.springframework.context.annotation.Configuration;
  import java.util.List;
  
  
  @Configuration
  public abstract class DefaultConsumerConfigure {
  
  
      private static final Logger log= LoggerFactory.getLogger(DefaultConsumerConfigure.class);
  
      @Value("${rocketmq.consumer.groupName}")
      private String groupName;
      @Value("${rocketmq.consumer.namesrvAddr}")
      private String namesrvAddr;
  
      // 开启消费者监听服务
      public void consumer(String topic, String tag) throws MQClientException {
          log.info("开启" + topic + ":" + tag + "消费者-------------------");
  
          DefaultMQPushConsumer consumer = new DefaultMQPushConsumer(groupName);
  
          consumer.setNamesrvAddr(namesrvAddr);
  
          consumer.subscribe(topic, tag);
  
          // 开启内部类实现监听
          consumer.registerMessageListener(new MessageListenerConcurrently() {
              @Override
              public ConsumeConcurrentlyStatus consumeMessage(List<MessageExt> msgs, ConsumeConcurrentlyContext context) {
                  return DefaultConsumerConfigure.this.dealBody(msgs);
              }
          });
  
          consumer.start();
  
          log.info("rocketmq启动成功---------------------------------------");
  
      }
  
      // 处理body的业务
      public abstract ConsumeConcurrentlyStatus dealBody(List<MessageExt> msgs);
  
  }
  ```

  

* 注册监听器监听消息

  ```java
  import org.apache.rocketmq.client.consumer.listener.ConsumeConcurrentlyStatus;
  import org.apache.rocketmq.client.exception.MQClientException;
  import org.apache.rocketmq.common.message.MessageExt;
  import org.slf4j.Logger;
  import org.slf4j.LoggerFactory;
  import org.springframework.context.ApplicationListener;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.event.ContextRefreshedEvent;
  
  import java.io.UnsupportedEncodingException;
  import java.util.List;
  
  @Service
  public class TestListener extends DefaultConsumerConfigure implements ApplicationListener<ContextRefreshedEvent> {
  
      private static final Logger log= LoggerFactory.getLogger(DefaultConsumerConfigure.class);
  
  
      @Override
      public void onApplicationEvent(ContextRefreshedEvent arg0) {
          try {
              super.consumer("TopicTest", "Tag1");
          } catch (MQClientException e) {
              log.error("消费者监听器启动失败", e);
          }
      }
  
      @Override
      public ConsumeConcurrentlyStatus dealBody(List<MessageExt> msgs)  {
          int num = 1;
          log.info("进入");
          for(MessageExt msg : msgs) {
              log.info("第" + num + "次消息");
              try {
                  String msgStr = new String(msg.getBody(), "utf-8");
                  log.info(msgStr);
              } catch (UnsupportedEncodingException e) {
                  log.error("body转字符串解析失败");
              }
          }
          return ConsumeConcurrentlyStatus.CONSUME_SUCCESS;
      }
  }
  ```







### 第3集  梳理用户下单流程coupon数据更新

**简介：用户下单逻辑梳理**

- 接入消息配置

![用户下单支付coupon数据更新](/Users/daniel/Desktop/material/优惠券原图/用户下单支付coupon数据更新.jpeg)        

- 定义shop传递给coupon的消息结构体

  ```powershell
  orderId
  couponCode
  userId
  ```

- 接口协议是一种规则，不管是http接口，dubbo接口，还是mq，都是系统间的互调，只要是系统间的互调就要协议，规范开发流程步骤可以减少出错概率
- 市面上的协议文档
  * wiki文档
  * apidoc自动文档
  * swagger自动生成文档







### 第4集  实战用户下单跟新coupon和order关联关系

**简介：实战用户下单跟新coupon和order关联关系**

- 用户下单跟新coupon和order关联关系开发

- 创建订单的时候处理发送mq给coupon告诉coupon和订单发送生关联

- 取消订单的时候发送消息解除coupon和订单关联

- 发送关联实战

  ```java
  private String couponCode;
  private Integer orderId;
  private Integer userId;
  ```







### 第5集  实战用户支付shop发送消息

**简介：实战用户支付shop发送消息**

- 用户支付行为调用shop发送mq通知

- 支付发送mq实战

  ```java
  private String couponCode;
  private Integer orderId;
  private Integer userId;
  //默认0代表用户支付成功，1代表退款
  private Integer status;
  ```







### 第6集  实战用户下单支付coupon跟新状态

**简介：实战用户下单支付coupon跟新状态**

- 用户下单==》coupon新增couponCode和order关联关系
- 用户支付==》coupon更新userCoupon为订单已支付状态







**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十八章   高级工程师进阶课Nginx反向代理



### 第1集  从分布式集群到Nginx反向代理

**简介：从分布式集群到Nginx反向代理**

- 集群是什么？

  * 服务器集群就是指将很多服务器集中起来一起进行同一种服务，在客户端看来就像是只有一个服务器

- 为什么要集群部署？

  * 集群可以利用多个计算机进行并行计算从而获得很高的计算速度
  * 有效减少单点故障问题      冷备份

- 集群部署怎么部署？

  * 通过服务通过域名映射到多台机器
  * 通过nginx反向代理


* nginx是什么
  * 是一个高性能的HTTP和反向代理服务器
* nginx的特点
  * Nginx 专为性能优化而开发，性能是其最重要的考量
  * 能经受高负载的考验,有报告表明能支持高达 50,000 个并发连接数





### 第2集  正向代理与反向代理

**简介：正向代理与反向代理**

- 正向代理

  * 正向代理服务器位于客户端和服务器之间，为了向服务器获取数据，客户端要向代理服务器发送一个请求，并指定目标服务器，代理服务器将目标服务器返回的数据转交给客户端
  * 正向代理的案例：VPN
    * VPN 通俗的讲就是一种中转服务，当我们电脑接入 VPN 后，我们对外 IP 地址就会变成 VPN 服务器的 公网 IP，我们请求或接受任何数据都会通过这个VPN 服务器然后传入到我们本机

  ![正向代理](/Users/daniel/Desktop/material/优惠券原图/正向代理.jpeg)

- 反向代理

  * 反向代理，其实客户端对代理是无感知的，因为客户端不需要任何配置就可以访问，我们只需要将请求发送到反向代理服务器，由反向代理服务器去选择目标服务器获取数据后，在返回给客户端，此时反向代理服务器和目标服务器对外就是一个服务器，暴露的是代理服务器地址，隐藏了真实服务器IP地址

     ![反向代理](/Users/daniel/Desktop/material/优惠券原图/反向代理.jpeg)

* 反向代理和正向代理的区别就是：**正向代理代理客户端，反向代理代理服务器**









### 第3集  手把手反向代理服务器Nginx安装(基于Linux服务器)

**简介：反向代理服务器Nginx安装**

* 安装编译工具及库文件

  ```she&amp;#39;l
  yum -y install make zlib zlib-devel gcc-c++ libtool  openssl openssl-devel
  ```

* 安装 PCRE

  ```shell
  [root@daniel src]# cd /usr/local/src/
  [root@daniel src]# wget http://downloads.sourceforge.net/project/pcre/pcre/8.35/pcre-8.35.tar.gz
  [root@daniel src]# tar zxvf pcre-8.35.tar.gz
  [root@daniel src]# cd pcre-8.35
  [root@bogon pcre-8.35]# ./configure
  [root@bogon pcre-8.35]# make && make install
  [root@bogon pcre-8.35]# pcre-config --version
  ```

* 下载 Nginx，下载地址：<http://nginx.org/download/nginx-1.6.2.tar.gz

  ```shell
  [root@daniel src]# cd /usr/local/src/
  [root@daniel src]# wget http://nginx.org/download/nginx-1.6.2.tar.gz
  [root@daniel src]# tar zxvf nginx-1.6.2.tar.gz
  [root@daniel src]# cd nginx-1.6.2
  [root@daniel nginx-1.6.2]# ./configure --prefix=/usr/local/webserver/nginx --with-http_stub_status_module --with-http_ssl_module --with-pcre=/usr/local/src/pcre-8.35
  [root@daniel nginx-1.6.2]# make
  [root@daniel nginx-1.6.2]# make install
  [root@daniel nginx-1.6.2]# /usr/local/webserver/nginx/sbin/nginx -v
  ```

* Nginx配置

  * 创建 Nginx 运行使用的用户 www：

    ```shell
    [root@daniel conf]# /usr/sbin/groupadd www 
    [root@daniel conf]# /usr/sbin/useradd -g www www
    ```

  * 配置nginx.conf ，将/usr/local/webserver/nginx/conf/nginx.conf替换为以下内容

     

    ```shell
    [root@daniel conf]#  cat /usr/local/webserver/nginx/conf/nginx.conf
    user www www;     
    worker_processes auto; #设置值和CPU核心数一致
    error_log /usr/local/webserver/nginx/logs/nginx_error.log crit; #日志位置和日志级别
    pid /usr/local/webserver/nginx/nginx.pid;
    #Specifies the value for maximum file descriptors that can be opened by this process.
    worker_rlimit_nofile 65535;
    events
    {
      use epoll;
      worker_connections 65535;
    }
    http
    {
      include mime.types;
      default_type application/octet-stream;
      log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
                   '$status $body_bytes_sent "$http_referer" '
                   '"$http_user_agent" $http_x_forwarded_for';
      
    #charset gb2312;
         
      server_names_hash_bucket_size 128;
      client_header_buffer_size 32k;
      large_client_header_buffers 4 32k;
      client_max_body_size 8m;
         
      sendfile on;
      tcp_nopush on;
      keepalive_timeout 60;
      tcp_nodelay on;
      fastcgi_connect_timeout 300;
      fastcgi_send_timeout 300;
      fastcgi_read_timeout 300;
      fastcgi_buffer_size 64k;
      fastcgi_buffers 4 64k;
      fastcgi_busy_buffers_size 128k;
      fastcgi_temp_file_write_size 128k;
      gzip on; 
      gzip_min_length 1k;
      gzip_buffers 4 16k;
      gzip_http_version 1.0;
      gzip_comp_level 2;
      gzip_types text/plain application/x-javascript text/css application/xml;
      gzip_vary on;
     
      #limit_zone crawler $binary_remote_addr 10m;
     #下面是server虚拟主机的配置
     server
      {
        listen 80;#监听端口
        server_name localhost;#域名
        index index.html index.htm index.php;
        root /usr/local/webserver/nginx/html;#站点目录
          location ~ .*\.(php|php5)?$
        {
          #fastcgi_pass unix:/tmp/php-cgi.sock;
          fastcgi_pass 127.0.0.1:9000;
          fastcgi_index index.php;
          include fastcgi.conf;
        }
        location ~ .*\.(gif|jpg|jpeg|png|bmp|swf|ico)$
        {
          expires 30d;
      # access_log off;
        }
        location ~ .*\.(js|css)?$
        {
          expires 15d;
       # access_log off;
        }
        access_log off;
      }
    
    }
    ```

  * 检查配置文件nginx.conf的正确性命令：

    ```shell
    [root@daniel conf]# /usr/local/webserver/nginx/sbin/nginx -t
    ```

  * 启动nginx

    ```shell
    [root@daniel conf]# /usr/local/webserver/nginx/sbin/nginx
    ```

  * 从浏览器访问我们配置的站点ip

  * 几个常见命令

  * ```
    /usr/local/webserver/nginx/sbin/nginx -s reload            # 重新载入配置文件
    /usr/local/webserver/nginx/sbin/nginx -s reopen            # 重启 Nginx
    /usr/local/webserver/nginx/sbin/nginx -s stop              # 停止 Nginx
    ```

* linux vi编辑器清空文件内容 

    

  ```shell
  :1,$d
  ```

  



### 第4集  Nginx企业级常用配置讲解

**简介：Nginx企业级常用配置讲解**

- nginx核心配置讲解

  * worker_connections设置可由一个worker进程同时打开的最大连接数

    ```shell
    events {
    worker_connections 2048;
    multi_accept on;
    use epoll;
    }
    ```

  * upstream负载均衡列表

    * weigth表示权重，权重越大分配比例越大
    * upstream www.abc.com {
          server 192.168.16.3:8080 weight=1;
          server 192.168.26.3:8080  weight=2;
          server 192.168.36.3:8080  weight=3;
         }

  * server设置访问的负载均衡处理的域名

    * server_name代表域名，listen代表端口

  * location配置域名下的路径地址

    * 可以配置一下静态文件的处理，就可以不用tomcat处理静态文件了，这个地方支持正则表达式

* 实战coupon项目搭建

   

  ```shell
  upstream shop {
      server 127.0.0.1:8088;
  }
  
  server {
      listen  80;
      server_name shop.xdclass.com;
      access_log logs/shop.xdclass.com.access.log;
      error_log  logs/shop.xdclass.com.error.log;
  
      charset utf-8;
  
      location / {
          proxy_pass  http://shop;
  
          #Proxy Settings
          proxy_redirect     off;
          proxy_set_header   Host             $host;
          proxy_set_header   X-Real-IP        $remote_addr;
          proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
          proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
          proxy_max_temp_file_size 0;
          proxy_connect_timeout      90;
          proxy_send_timeout         90;
          proxy_read_timeout         90;
          proxy_buffer_size          4k;
          proxy_buffers              4 32k;
          proxy_busy_buffers_size    64k;
          proxy_temp_file_write_size 64k;
          expires -1;
          add_header Cache-Control no-store;
     }
  }
  }
  ```

* 80端口代表服务器默认端口

    http://172.16.244.151:80  = http://172.16.244.151





### 第5集  配置shop域名反向代理

**简介：从域名访问到最终定位到机器IP**

- 从域名访问到最终定位到机器IP经历了什么
- 什么是DNS解析？
  * 用于将用户提供的主机名解析为ip地址

* DNS解析过程
  具体过程如下：
  ①用户主机上运行着DNS的客户端，就是我们的PC机或者手机客户端运行着DNS客户端了
  ②浏览器将接收到的url中抽取出域名字段，就是访问的主机名，比如

  ```text
  http://shop.xdclass.com/ 
  ```


  ③DNS客户机端向DNS服务器端发送一份查询报文，报文中包含着要访问的主机名字段（中间包括一些列缓存查询以及分布式DNS集群的工作）
  ④该DNS客户机最终会收到一份回答报文，其中包含有该主机名对应的IP地址
  ⑤一旦该浏览器收到来自DNS的IP地址，就可以向该IP地址定位的HTTP服务器发起TCP连接

* DNS解析过程

   本地浏览器缓存==》hosts文件==》本地DNS服务器 ==》 ISP商DNS服务器 ==》顶级域名服务器 ==》跟域名服务器

* 得出结论，DNS解析的过程第一步会走本地缓存，那么我们可以配置一个不存在的域名进行内网/本地访问

* 配置shop.xdclass.com这个域名的hosts文件

  *  添加配置

    ```powershell
    172.16.144.152 shop.xdclass.com
    ```

  * windows：

  ```powershell
  首先，打开计算机后，点击进入C盘，也就是系统盘符，找到windows文件夹
  在windows文件中找到System32-->drivers-->etc,进入到etc文件夹中就能看到hosts文件
  在hosts文件上右键操作，弹出菜单用记事本或者其他文本工具打开都可以是，这里是有UE打开
  打开后，按照配置要求，前面配置IP 后面配置对应的域名或者网站
  ```

  * mac/linux:

  ```
  vi /etc/hosts
  ```

  ```
  user www www;     
  worker_processes auto; #设置值和CPU核心数一致
  error_log /usr/local/webserver/nginx/logs/nginx_error.log crit; #日志位置和日志级别
  pid /usr/local/webserver/nginx/nginx.pid;
  #Specifies the value for maximum file descriptors that can be opened by this process.
  worker_rlimit_nofile 65535;
  events
  {
    use epoll;
    worker_connections 65535;
  }
  http
  {
    include mime.types;
    default_type application/octet-stream;
    log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
                 '$status $body_bytes_sent "$http_referer" '
                 '"$http_user_agent" $http_x_forwarded_for';
    
  #charset gb2312;
       
    server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 8m;
       
    sendfile on;
    tcp_nopush on;
    keepalive_timeout 60;
    tcp_nodelay on;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    gzip on; 
    gzip_min_length 1k;
    gzip_buffers 4 16k;
    gzip_http_version 1.0;
    gzip_comp_level 2;
    gzip_types text/plain application/x-javascript text/css application/xml;
    gzip_vary on;
   
    #limit_zone crawler $binary_remote_addr 10m;
   #下面是server虚拟主机的配置
   
   upstream shop {
      server 127.0.0.1:8097;
  }
  
  server {
      listen  80;
      server_name shop.xdclass.com;
      access_log logs/shop.xdclass.com.access.log htpmLogFormat;
      error_log  logs/shop.xdclass.com.error.log;
  
      charset utf-8;
  
      location / {
          proxy_pass  http://shop;
  
          #Proxy Settings
          proxy_redirect     off;
          proxy_set_header   Host             $host;
          proxy_set_header   X-Real-IP        $remote_addr;
          proxy_set_header   X-Forwarded-For  $proxy_add_x_forwarded_for;
          proxy_next_upstream error timeout invalid_header http_500 http_502 http_503 http_504;
          proxy_max_temp_file_size 0;
          proxy_connect_timeout      90;
          proxy_send_timeout         90;
          proxy_read_timeout         90;
          proxy_buffer_size          4k;
          proxy_buffers              4 32k;
          proxy_busy_buffers_size    64k;
          proxy_temp_file_write_size 64k;
          expires -1;
          add_header Cache-Control no-store;
     }
  }
  
  }
  ```

  



### 第6集  实战coupon服务在Linux环境部署

**简介：coupon服务在linux环境部署**

- 服务器部署过程

  * 从git下载服务器最新代码 git clone 
  * 确定代码构建分支，如test分支表示测试环境、master分支常表示线上
  * 构建代码，如 mvn install
  * 启动服务nohup java -jar 报名 &

- 从本地上传构建好的jar包到服务器直接启动

  * scp 本地jar包 到远程服务器    scp ==>ssh copy

  ```shell
  scp /Users/daniel/Desktop/coupon-app-0.0.1-SNAPSHOT.jar 172.16.244.151:/usr/local/jar_file
  输入密码为使用ssh链接远程服务器的密码(虚拟机)
  ```

  





**![å°Dè¯¾å ](data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAGoAAAAiCAMAAACA9LykAAAAY1BMVEUAAAAjJikjJiojJiokJigjJikkJigjJikjJikjJikjJikkJigjJiojJiokJigjJiomJiYmJibWABPWABMmJiYmJiYmJibWABN9Ex99Ex/WABMmJiYjJiqgoKB3d3dcXFxBQUEZhcyGAAAAG3RSTlMAQIC/MGAQz9+fj+8gr3BQv4C/gCCvUN+/gGCwJXa4AAACP0lEQVRIx72W2ZLbIBBFe2EVQrLHnslCZ/n/rwwoOCDH5CWyzxO2VJy63TQlGGAQFbwEhagpOseE8BpmzRL8Cq9h9iJOw2uYWYRneBb67LtTsYiEZwXDlE47s4g8x2VsVplXuJRLGbtzUXap4zOFYvr54x16rIiFo1lS4fu3sIuFzyjhe6qcocftYukJRrQn7c0Bp6a679bcfvHDLpMCEB/jVB8bWonB4EBlq8kDjCvYVF8vlwtpA4UoMauQGeU2kZYYfBhMb9oIV7gjq6YHqrdPb28sQUEGGUsq5/ymQvLi2MUwkX5Y6lRdCgpq6VT8SHUphYphLi8z9KnIol+w4HnUqpZLJ2/+UsWpqBQvVVVcjrYiFwE5RwSZWQHlJauyesA53bAA14RQ6VWxqKIIZD5/gYLnqkJEaxGxpRccj9UNDR+2XfBZFaFgRJUt1uAhw1T1N9Wy9KrAPFSppnJwPu2uQaqT0vWqqpToTaUAJsc82b6rQ0Jz4dJU9s8tSGavQkQKXAdiXmVhBuuriNgRjWL5pprQ9mPlunPKuykIWZ+Zg1ZuBea88oNUgwpalfpQ1G0/ja6lGYoKVJYbzHiLmRke4prLJFN3KKG6+xfhHygFGwYrQ9XUVB8O65nIIBxNf9zPJ9z+ka18/816Hw7vVMZLxsMBTOvwXPikAJ1kJjgEYuzr1/XKnRSLHPlthtZq08rXYSVz7BendmIJi+7aPJJ+ixCOZY3btuw3ScMjHI9ZvZUdzq8GngYiVTQeq/kFPvQz03kWbEEAAAAASUVORK5CYII=)   愿景："让编程不在难学，让技术与生活更加有趣"          		**

------

## 第十九章  微服务实战总结与思考



### 第1集  互联网线上故障问题定位小工具

* 线上定位问题能力的重要性

  * 作为互联网公司，经常会出现线上故障，如：cpu占用过高、线程数占满、服务质量下降，掌握出现异常的时候定位问题， 解决问题的能力显得尤为重要
* 掌握定位问题和解决问题的思路

  ![互联网线上故障定位处理工具](/Users/daniel/Desktop/material/优惠券原图/互联网线上故障定位处理工具.jpeg)

* CPU占满出现在计算密集型程序  kill -15
* 做练习多实操才能信手拈来







### 第2集  互联网架构优惠券系统实战

- 互联网架构优惠券系统实战总结

  

### 第3集  互联网架构学习秘笈分享

- 学习秘笈分享


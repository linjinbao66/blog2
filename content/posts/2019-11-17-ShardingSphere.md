---
title: MySQL 分库分表-ShardingSphere使用
tags:
  - Spring
  - Mysql
date: 2019-11-17
---

## MySQL 分库分表-ShardingSphere使用

## 分库和分表的实现-java工程版

1. 依赖项

```markup
<dependencies>
        <!-- 主要 -->
        <dependency>
            <groupId>org.apache.shardingsphere</groupId>
            <artifactId>sharding-jdbc-core</artifactId>
            <version>4.0.0-RC2</version>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
        </dependency>
        <dependency>
            <groupId>commons-dbcp</groupId>
            <artifactId>commons-dbcp</artifactId>
            <version>1.4</version>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.41</version>
        </dependency>
        <dependency>
            <groupId>io.netty</groupId>
            <artifactId>netty</artifactId>
            <version>3.7.0.Final</version>
        </dependency>

    </dependencies>
```

1. 示例代码

```java
public class ShardingSphereDemo {

    @Test
    public void test01() throws SQLException {
        // 配置真实数据源
        Map<String, DataSource> dataSourceMap = new HashMap<String, DataSource>();

        // 配置第一个数据源
        BasicDataSource dataSource1 = new BasicDataSource();
        dataSource1.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource1.setUrl("jdbc:mysql://localhost:3306/test");
        dataSource1.setUsername("root");
        dataSource1.setPassword("Gepoint");
        dataSourceMap.put("test", dataSource1);

        // 配置表规则
        TableRuleConfiguration orderTableRuleConfig = new TableRuleConfiguration(
                "user", "test.user_${1..2}");

        orderTableRuleConfig.setTableShardingStrategyConfig(
                new InlineShardingStrategyConfiguration("id", "user_${id % 2}"));
        orderTableRuleConfig.setKeyGeneratorConfig(new KeyGeneratorConfiguration("SNOWFLAKE", "id"));

        // 配置分片规则
        ShardingRuleConfiguration shardingRuleConfig = new ShardingRuleConfiguration();
        shardingRuleConfig.getTableRuleConfigs().add(orderTableRuleConfig);

        // 省略配置order_item表规则...
        // ...

        // 获取数据源对象
        DataSource dataSource = ShardingDataSourceFactory.createDataSource(dataSourceMap, shardingRuleConfig, new Properties());


        Connection connection = dataSource.getConnection();
        System.out.println(connection);

        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery("select * from  user");
        while (resultSet.next()) {
            String username = resultSet.getString("username");
            String passwd = resultSet.getString("passwd");
            System.out.println("username=" + username + "\t" + "passwd=" + passwd);
        }

        statement.execute("insert  into  user(username, passwd) values ('林金保2', 'Gepoint')");

    }
}
```

## 实现-Spring 容器管理代码实现

1. xml配置

   \`\`\`xml &lt;?xml version="1.0" encoding="UTF-8"?&gt;

```text
2. ShardingDemo.java
```java
public class ShardingDemo {

    ApplicationContext context = new ClassPathXmlApplicationContext("applicationcontext.xml");

    @Test
    public void test01() throws SQLException {
        Map<String, DataSource> dataSourceMap = new HashMap<String, DataSource>();
        DataSource dataSource1 = context.getBean("comboPooledDataSource",ComboPooledDataSource.class);
        dataSourceMap.put("test", dataSource1);
        //容器中取出
        TableRuleConfiguration tableRuleConfiguration =  context.getBean(TableRuleConfiguration.class);
        ShardingRuleConfiguration shardingRuleConfiguration = context.getBean(ShardingRuleConfiguration.class);
        shardingRuleConfiguration.getTableRuleConfigs().add(tableRuleConfiguration);

        DataSource dataSource = ShardingDataSourceFactory.createDataSource(dataSourceMap, shardingRuleConfiguration, new Properties());
        Connection connection = dataSource.getConnection();
        System.out.println(connection);

        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery("select * from  user");
        while (resultSet.next()) {
            String username = resultSet.getString("username");
            String passwd = resultSet.getString("passwd");
            System.out.println("username=" + username + "\t" + "passwd=" + passwd);
        }
        statement.execute("insert  into  user(username, passwd) values ('林金保5', 'Gepoint')");
    }

}
```

## 步骤解释

1. 配置真实数据源dataSourceMap，在此处允许配置多个数据库，实现分库
2. 配置库规则和表规则，ShardingSphere会根据此规则解析，生成真实sql语句
3. 注意此处有虚拟表名的概念，即使用`user`表加上上面的表规则表示`user_1`和`user_2`两张表，查询语句只需要写`user`即可
4. `insert`数据时，需要数据库`user_1`和`user_2`满足**主键不重复**原则，此处使用Shard ingSphere提供的`SNOWFLAKE`实现

## 总结

核心思想：在数据库和java代码之间多了一层，用户写的sql是逻辑sql，由shardingsphere根据配置方式，生成真实sql，并操作数据库连接执行，隔绝了程序员与真实数据库之间的直接连接

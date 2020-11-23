# 概念

Java DataBase Connectivity --Java数据库连接

本质：官方定义的一套操作所有**关系型数据库**的规则（接口），需要JDBC驱动（实现类）才能连接。JDBC驱动是由各个数据库厂商提供的jar包。我们使用的是接口中的方法，真正执行的是JDBC驱动。

# 使用

- 下载JDBC驱动并导入。
- 注册驱动。
- 获取数据库连接对象 Connection。
- 定义一个SQL语句。
- 获取执行SQL语句的对象 Statement。
- 执行SQL，接收返回结果。
- 处理结果。
- 释放资源，关闭连接。

# 详解各个对象

- `DriverManager`：驱动管理对象

  - 注册驱动，此类源码中有`static`包裹的静态代码块：`java.sql.DriverManager.registerDriver(new Driver());`，调用`DriverManager`的注册驱动的方法。
  - 注意：mysql5之后的jar包可以省略加载jar包的步骤，`DriverManager`类的源码中有`static`包裹的静态代码块`loadInitialDrivers();`，该方法会自动加载jdbc驱动。
  - 获取数据库连接，如果连接的是本机mysql服务器，则url可以简写为 **jdbc:mysql:///db** 

- `Connection`：数据库连接对象

  - 获取执行SQL的对象`Statement`或者`PreparedStatement`
  - 管理事务：
    - 开启事务 `void setAutoCommit(boolean autoCommit);`调用该方法，设置参数为false即是开启事务，关闭自动提交
    - 提交事务 `void commit();` 提交事务
    - 回滚事务 `void rollback();` 回滚

- `Statement`：用于执行静态SQL并返回结果的对象

  - `boolean execute(String sql);` 执行任意SQL
  - `int executeUpdate(String sql);` 执行DML(insert、update、delete)语句，DDL(creat、alter、drop)语句，返回值是受影响的行数。可以通过受影响的行数老判断是否执行成功。
  - `ResultSet executeQuery(String sql);` 执行DQL(select)语句，返回的是一个结果集对象。

- `ResultSet`：结果集对象，封装查询结果

  - `boolean next()`：游标向下移动一行，判断移动之后当前行还有没有数据，返回布尔值。
  - `getXxx()`：获取数据
    - Xxx代表数据类型如 `int getInt()`，`String getString()`
    - 参数：
      - int：列的编号，从1开始，比如`getString(1)`
      - String：列的名称，如`getDouble("price")`

- `PreparedStatement`：执行SQL对象，比`Statement`更强大。

  - SQL注入问题：在拼接SQL时，有一些SQL的特殊关键字参与字符串拼接，会造成一些安全性问题。

  - 解决：使用`PreparedStatement`

    - 预编译SQL：使用?作为占位符

  - 使用步骤：

    - 下载JDBC驱动并导入。

    - 注册驱动。

    - 获取数据库连接对象 Connection。

    - 定义一个SQL语句。sql的参数使用 ? 作为占位符，如

      ```sql
      select * from user where username = ? and psasswoed = ?
      ```

    - 获取执行SQL语句的对象 `PreparedStatement`。`PreparedStatement Connection.prepareStatement(String sql)`

    - 给 ? 赋值 使用setXxx(参数1，参数2)。参数1，？的位置，从1开始；参数2，值。

    - 执行SQL，接收返回结果。不需要传递SQL语句。

    - 处理结果。

    - 释放资源，关闭连接。

# JDBC控制事务

事务：一个包含多个数据库操作的业务操作。如果这个业务操作被事务管理，则这多个步骤要么同时成功，要么同时失败。

使用Connection对象来管理事务：

- 开启事务 `void setAutoCommit(boolean autoCommit);`调用该方法，设置参数为false即是开启事务，关闭自动提交
- 提交事务 `void commit();` 提交事务
- 回滚事务 `void rollback();` 回滚
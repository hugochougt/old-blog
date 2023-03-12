---
layout: post
title: "在 Ubuntu 上安装 MySQL 及 JDBC 连接测试程序"
date: 2013-07-01 19:40
tags: [MySQL, JDBC, Java]
---

## 安装 MySQL
在能联网的情况下，在 Ubuntu 上安装软件可以说是毫无技术含量可言。要安装最新版的 MySQL，在终端里输入以下命令然后输入 root 密码就 OK 了：
    sudo apt-get install mysql-server

安装完成后 MySQL 会要求你输入 root 密码两次，这时候就不要回车跳过了，省得日后还要设置。

MySQL 的配置优化还请自行 google，因为我也不懂。

## JDBC 连接测试
首先下载[Connector/J](http://dev.mysql.com/downloads/connector/j/)，解压之后將其中的`mysql-connector-java-*-bin.jar`添加到 eclipse 工程的 Build Path，然后保存下面的测试程序到 eclipse，并修改 url 变量中的 password 为你安装 MySQL 时设置的密码，运行程序就可以查看 MySQL 的安装是否成功了。

<!-- more -->

```java
package mysql;

import java.sql.*;

public class MySqlTest {

    private String createUserTable = "CREATE TABLE User (" + "id INTEGER, "
                                    + "name VARCHAR(20),"
                                    + "password VARCHAR(20))";
    private String dropUserTable = "DROP TABLE User";
    private String insertUser = "insert into User(id, name, password)" +
        "select ifNull(max(id),0)+1,?,? FROM User";
    private String selecrUsers = "select * from User";
    private String url = "jdbc:mysql://localhost:3306/test?user=root&password=123456";

    private Connection conn;
    private Statement stat;
    private ResultSet results;
    private PreparedStatement preStat;

    public MySqlTest() {
        try {
            Class.forName("com.mysql.jdbc.Driver");
            conn = DriverManager.getConnection(url);
        } catch (ClassNotFoundException e) {
            System.err.println("### Driver not found! ###");
            e.printStackTrace();
        } catch (SQLException e) {
            System.err.println("### SQL exception! ###");
            e.printStackTrace();
        }
    }

    public void createTable() {
        try {
            stat = conn.createStatement();
            stat.execute(createUserTable);
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            close();
        }
    }

    public void insertUser(String name, String password) {
        try {
            preStat = conn.prepareStatement(insertUser);

            preStat.setString(1, name);
            preStat.setString(2, password);
            preStat.executeUpdate();
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            close();
        }
    }

    public void dropTable() {
        try {
            stat = conn.createStatement();
            stat.executeUpdate(dropUserTable);
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            close();
        }
    }

    public void showUsers() {
        try {
            stat = conn.createStatement();
            results = stat.executeQuery(selecrUsers);

            System.out.println("id\t\tname\t\tpasswor");

            while (results.next()) {
                System.out.println(results.getInt(1) + "\t\t"
                        + results.getString("name") + "\t\t"
                        + results.getString("password"));
            }
        } catch (SQLException e) {
            e.printStackTrace();
        } finally {
            close();
        }
    }

    public void close() {
        try {
            if (results != null) {
                results.close();
                results = null;
            }

            if (stat != null) {
                stat.close();
                stat = null;
            }

            if (preStat != null) {
                preStat.close();
                preStat = null;
            }
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }

    public static void main(String[] args) {
        MySqlTest test = new MySqlTest();

        // test.dropTable();
        test.createTable();
        test.insertUser("zhaqiang", "111222");
        test.insertUser("hugo", "4326");
        test.showUsers();
    }
}
```

### 参考链接
 1. [MySQL安裝指南](http://wiki.ubuntu.org.cn/index.php?title=MySQL%E5%AE%89%E8%A3%85%E6%8C%87%E5%8D%97&variant=zh-hant)
 2. [Eclipse設定JDBC連接MySQL資料庫](http://blog.yslifes.com/archives/918)

 **-EOF-**

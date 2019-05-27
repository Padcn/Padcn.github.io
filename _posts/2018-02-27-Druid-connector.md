---
title: Gradle使用
date: 2018-02-27
tags: 
- java 
- DB
---
# Druid 数据库连接池使用

> https://www.cnblogs.com/longshiyVip/p/5117441.html


@DruidTest.java
```java
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import com.alibaba.druid.pool.DruidDataSource;

public class DruidTest {
	public static void main(String[] args) {
		DruidDataSource dataSource=new DruidDataSource();
		dataSource.setDriverClassName("com.mysql.jdbc.Driver");
		dataSource.setUrl("jdbc:mysql://127.0.0.1:3306/com");
		dataSource.setUsername("root");
		dataSource.setPassword("pwd123");
		dataSource.setInitialSize(5); 
		dataSource.setMinIdle(1); 
		dataSource.setMaxActive(10);
		try {
			Connection conn=dataSource.getConnection();
			Statement st=conn.createStatement();
			String sql="select * from person";
			ResultSet rs=st.executeQuery(sql);
			while(rs.next()){
				for(int i=1;i<=2;i++){
                    System.out.println(rs.getObject(i));
                }
			}
		} catch (SQLException e) {
			e.printStackTrace();
		}
	}
}
```

maven依赖

```xml
<dependency>
  <groupId>com.alibaba</groupId>
  <artifactId>druid</artifactId>
  <version>1.0.2</version>
</dependency>
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>5.1.27</version>
</dependency>
```

另web项目中启用监控,在web.xml中配置，访问监控页面为 {ip}:{port}/{applicationName}/druid

```xml
<servlet>
      <servlet-name>DruidStatView</servlet-name>
      <servlet-class>com.alibaba.druid.support.http.StatViewServlet</servlet-class>
  </servlet>
  <servlet-mapping>
      <servlet-name>DruidStatView</servlet-name>
      <url-pattern>/druid/*</url-pattern>
  <servlet-mapping>
```


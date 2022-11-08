---
title: spring 기초(3) maven & db연동
date: 2021-05-31
tags: spring
---


-----
![DB 설정 과정](/assets/images/DB 설정 과정.JPG){: width="1000" height="500"}

![DB데이터 전달 과정](/assets/images/_posts/DB데이터 전달 과정.JPG){: width="1000" height="500"}

![Select resultType](/assets/images/_posts/Select resultType.JPG){: width="1000" height="500"}

```markdown
Maven - 통합 라이브러리 관리 및 배포 지원
라이브러리 : 특정 목적에 따라 미리 구현된 프로그램 파일 집합(jar파일, ear파일)
통합 라이브러리 관리 : 설정된 내용을 기반으로 jar파일을 버전별 보관 및 제공

프로젝트 import 시  X표시로 update maven 처리를 했으나 해결이 안된 경우
=> 라이브러리 파일이 정상적으로 다운이 안 된 경우
해결방법 : 이클립스 종료 -> .m2폴더 제거 -> 이클립스 재실행 -> update maven
update maven : 프로젝트 우클릭 - maven - update Project
```

-----

-----

##### TestContoller.java

```java
package com.spring.sample.web.test.controller;

import java.util.HashMap;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.servlet.ModelAndView;

import com.spring.sample.web.test.service.ITestService;

@Controller
public class TestContoller {
	@Autowired
	public ITestService iTestService;
	
	@RequestMapping(value="/test1")
	public ModelAndView test1(ModelAndView mav) throws Throwable {
		
		List<HashMap<String, String>> list = iTestService.getBList();
		
		mav.addObject("list", list);
		
		mav.setViewName("test/test1");

		return mav;
	}
}
```

-----

##### ITestService.java

```java
package com.spring.sample.web.test.service;

import java.util.HashMap;
import java.util.List;

public interface ITestService {

	public List<HashMap<String, String>> getBList() throws Throwable;

}
```

-----

##### TestService.java

```java
package com.spring.sample.web.test.service;

import java.util.HashMap;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.spring.sample.web.test.dao.ITestDao;

@Service
public class TestService implements ITestService{
	@Autowired
	public ITestDao iTestDao;

	@Override
	public List<HashMap<String, String>> getBList() throws Throwable {
		return iTestDao.getBList();
	}
}
```

-----

##### ITestDao.java

```java
package com.spring.sample.web.test.dao;

import java.util.HashMap;
import java.util.List;

public interface ITestDao {

	public List<HashMap<String, String>> getBList() throws Throwable;

}
```

------

##### TestDao.java

```java
package com.spring.sample.web.test.dao;

import java.util.HashMap;
import java.util.List;

import org.apache.ibatis.session.SqlSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

@Repository
public class TestDao implements ITestDao{
	@Autowired
	public SqlSession sqlSession;

	@Override
	public List<HashMap<String, String>> getBList() throws Throwable {
		return sqlSession.selectList("B.getBList");
	}
}
```

------

##### root-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
	
	<!-- Root Context: defines shared resources visible to all other web components -->
	<bean id="propertyConfigurer" class="org.springframework.beans.factory.config.PropertyPlaceholderConfigurer">
		<property name="locations">
			<list>
				<value>/WEB-INF/spring/jdbc.properties</value>							
			</list>
		</property>
	</bean>
	
	<!-- dataSource : 접속설정객체 -->
	<bean id="dataSource" class="org.apache.commons.dbcp.BasicDataSource" destroy-method="close">
        <property name="driverClassName" value="${jdbc.driverClassName}" />
        <property name="url" value="${jdbc.url}" />
        <property name="username" value="${jdbc.username}" />
        <property name="password" value="${jdbc.password}" />
    </bean>
    
    <!-- log설정 : dataSource에 로그 지정(선택사항) -->
    <bean id="dataSourceLog" class="net.sf.log4jdbc.Log4jdbcProxyDataSource">
        <constructor-arg ref="dataSource" />
        <property name="logFormatter">
            <bean class="net.sf.log4jdbc.tools.Log4JdbcCustomFormatter">
                <property name="loggingType" value="MULTI_LINE" />
                <property name="sqlPrefix" value="SQL : "/>
            </bean>
        </property>
    </bean>
    
    <!-- sessionFactory : DB접속 객체 -->
    <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
    	<!-- Mybatis 설정 -->
    	<property name="configLocation" value="classpath:mybatis-config.xml" />
    	<!-- DB접속 설정 -->
        <!-- <property name="dataSource" ref="dataSource" /> -->
        <property name="dataSource" ref="dataSourceLog" />
        <!-- mapper : 쿼리가 보관된 파일 -->
        <property name="mapperLocations" value="classpath*:mapper/**/*_SQL.xml" />
    </bean>
    <!-- sqlSession : SessionFactory 사용 객체 -->
    <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
        <constructor-arg ref="sqlSessionFactory" />
    </bean>
</beans>
```

------

##### B_SQL.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="B"><!-- namespace - 클래스명과 동일 -->
	<!-- id - 메소드명과 동일 -->
	<!-- resultType : row 1줄의 형태를 지정 -->
	<!-- 쿼리 작성 시 ;이 들어가면 실행되지 않음 -->
	<select id="getBList" resultType="hashmap">
		SELECT B_NO, B_TITLE, B_WRITER, TO_CHAR(B_DT, 'YYYY-MM-DD') AS B_DT
		FROM B
		ORDER BY B_NO DESC
	</select>
</mapper>
```

------

##### test1.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.05.31</title>
</head>
<body>
	<table>
	<thead>
		<tr>
			<th>번호</th>
			<th>제목</th>
			<th>작성자</th>
			<th>작성일</th>
		</tr>
	</thead>
	<tbody>
		<c:forEach var="data" items="${list}">
			<tr>
				<td>${data.B_NO}</td>
				<td>${data.B_TITLE}</td>
				<td>${data.B_WRITER}</td>
				<td>${data.B_DT}</td>
			</tr>
		</c:forEach>
	</tbody>
	</table>
</body>
</html>
```

-----

##### jdbc.properties

```tex
#Oracle Database Datasource Properties
################ Oracle DB ##################
jdbc.driverClassName=oracle.jdbc.driver.OracleDriver
jdbc.url=jdbc:oracle:thin:@127.0.0.1:1521:XE
jdbc.username=TEST
jdbc.password=1234

#Mysql Database Datasource Properties
################ Mysql DB ##################
#jdbc.driverClassName=org.gjt.mm.mysql.Driver
#jdbc.url=jdbc:mysql://127.0.0.1:3306/test
#jdbc.username=test
#jdbc.password=test

#MSSQL Database Datasource Properties
################ MSSQL DB ##################
#jdbc.jdbctype=MSSQL
#jdbc.driverClassName=com.microsoft.sqlserver.jdbc.SQLServerDriver
#jdbc.url=jdbc:sqlserver://127.0.0.1:1433;DatabaseName=SID
#jdbc.username=test
#jdbc.password=test
```

-----

##### mybatis-config.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE configuration PUBLIC "-//mybatis.org//DTD Config 3.0//EN" "http://mybatis.org/dtd/mybatis-3-config.dtd">
<configuration>
	<typeAliases>
		<typeAlias alias="hashmap" type="java.util.HashMap" />
	</typeAliases>
</configuration>
```

-----

##### B테이블

```sql
INSERT INTO B(B_NO, B_TITLE, B_WRITER, B_CON)
VALUES(B_SEQ.NEXTVAL, 'TEST' || B_SEQ.CURRVAL, 'TESTER', 'TEST중입니다.')
;

SELECT B_NO, B_TITLE, B_WRITER, TO_CHAR(B_DT, 'YYYY-MM-DD') AS B_DT
FROM B
ORDER BY B_NO DESC
;

COMMIT;
```
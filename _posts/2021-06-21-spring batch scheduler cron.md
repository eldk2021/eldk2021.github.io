---
title: spring batch, scheduler, cron
date: 2021-06-21
tags: spring
---


##### servlet-context.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans:beans xmlns="http://www.springframework.org/schema/mvc"
	xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	xmlns:beans="http://www.springframework.org/schema/beans"
	xmlns:context="http://www.springframework.org/schema/context"
	xmlns:task="http://www.springframework.org/schema/task"
	xmlns:aop="http://www.springframework.org/schema/aop"
	xsi:schemaLocation="http://www.springframework.org/schema/mvc http://www.springframework.org/schema/mvc/spring-mvc.xsd
		http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
		http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
		http://www.springframework.org/schema/task http://www.springframework.org/schema/task/spring-task-4.0.xsd
		http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop-4.3.xsd">

	<!-- DispatcherServlet Context: defines this servlet's request-processing infrastructure -->
	
	<!-- Enables the Spring MVC @Controller programming model -->
	<annotation-driven />

	<!-- Handles HTTP GET requests for /resources/** by efficiently serving up static resources in the ${webappRoot}/resources directory -->
	<resources mapping="/resources/**" location="/resources/" />

	<!-- Resolves views selected for rendering by @Controllers to .jsp resources in the /WEB-INF/views directory -->
	<beans:bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
		<beans:property name="prefix" value="/WEB-INF/views/" />
		<beans:property name="suffix" value=".jsp" />
	</beans:bean>
	
	<context:component-scan base-package="com.gd.test" />
	
	<!-- multipartResolver -->
	<beans:bean id="multipartResolver" class="org.springframework.web.multipart.commons.CommonsMultipartResolver">
		<beans:property name="maxUploadSize" value="10000000" /> <!-- 업로드 가능한 파일 크기 설정 -->
	</beans:bean>
	
	<!-- Exception Resolver -->
	<beans:bean id="exceptionMapping" 	class="org.springframework.web.servlet.handler.SimpleMappingExceptionResolver">
		<beans:property name="exceptionMappings">
			<beans:props>
				<beans:prop key="java.lang.Exception">/exception/EXCEPTION_INFO</beans:prop>
				<beans:prop 
					key="com.gd.test.exception.UserNotFoundException">
					/exception/USER_NOT_FOUND_EXCEPTION
				</beans:prop>
			</beans:props>
		</beans:property>
	</beans:bean>
	
	<!-- Spring scheduler -->
	<!-- pool-size : 작업 개수 -->
	<task:scheduler id="scheduler" pool-size="50"/>
	<task:annotation-driven scheduler="scheduler"/>
	
	<!-- AOP -->
	<aop:aspectj-autoproxy />
	<beans:bean id="commonAop" class="com.gd.test.common.controller.CommonAOP" /> 
</beans:beans>

```

-----

##### BatchComponent.java

```java
package com.gd.test.batch.controller;

import org.springframework.scheduling.annotation.Scheduled;
import org.springframework.stereotype.Component;

@Component
public class BatchComponent {

	// 초 분 시 일 월 요일
	// * - 모든
	// ? - 월, 요일에 사용. 신경안씀이라는 의미
	// 월은 1 - 12
	// 요일은 1(일) - 7(토). ,(컴마)로 복수지정가능. 예)월수금 - 2,4,6
	// 숫자1/숫자2의 경우 숫자1에서 시작하여 매 숫자2마다 실행. 예) 분에 0/5이면 0분부터 5분마다 실행
	// 일에서 L은 달의 마지막날. W는 지정일자가 휴일(토, 일)이면 인접한 평일에 수행.
	// 예) 25W인경우 25일이 일요일이면 26일 월요일 실행.
	// LW는 마지막 평일
	// 요일에서 숫자1#숫자2의경우 숫자2번째 주의 숫자1번 요일에 실행.
	// 예) 2#4 - 4번째주 월요일에 실행.
	@Scheduled(cron = "0 0 0 * * *")
	public void cronTest1() {
		System.out.println("batch!!");
	}

}
```
---
title: spring AOP
date: 2021-06-21
tags: spring
---




![AOP 로직](/assets/images/AOP 로직.JPG){: width="1000" height="500"}
![패키지 변경 과정](/assets/images/패키지 변경 과정.JPG){: width="1000" height="500"}

-----

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

##### CommonAOP.java

```java
package com.gd.test.common.controller;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpSession;

import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Pointcut;
import org.springframework.web.context.request.RequestContextHolder;
import org.springframework.web.context.request.ServletRequestAttributes;
import org.springframework.web.servlet.ModelAndView;

@Aspect
public class CommonAOP {
	//Pointcut -> 적용범위
	//@Pointcut(범위설정)
	/*
	 * 범위
	 * execution -> include필터
	 * !execution -> exclude필터
	 * * -> 모든것
	 * *(..) -> 모든 메소드
	 * .. -> 모든 경로
	 * && -> 필터 추가, 그리고
	 * || -> 필터 추가, 또는
	 */
	@Pointcut("execution(* com.gd.test..TestAController.*(..))"
			+ "&&!execution(* com.gd.test..TestAController.*Login(..))"
			+ "&&!execution(* com.gd.test..TestAController.*Logout(..))"
			+ "&&!execution(* com.gd.test..TestAController.*s(..))")

	public void testAOP() {}
	
	//ProceedingJoinPoint -> 대상 적용 이벤트 필터
	/*
	 * @Before -> 메소드 실행 전
	 * @After -> 메소드 실행 후
	 * @After-returning -> 메소드 정상실행 후
	 * @After-throwing -> 메소드 예외 발생 후
	 * @Around -> 모든 동작시점
	 */
	// testAOP() pointcut범위의 대상의 Around관점에 아래 메소드를 적용시킨다. 
	@Around("testAOP()")
	public ModelAndView testAOP(ProceedingJoinPoint joinPoint)
														throws Throwable {
		ModelAndView mav = new ModelAndView();
		
		//Request 객체 취득
		HttpServletRequest request
		= ((ServletRequestAttributes)RequestContextHolder.currentRequestAttributes()).getRequest();
		
		HttpSession session = request.getSession();
		
		if(session.getAttribute("sMNo") != null) { // 로그인 상태
			mav = (ModelAndView) joinPoint.proceed(); // 기존 이벤트 처리 행위를 이어서 진행			
		} else { // 비로그인 상태
			mav.setViewName("redirect:testALogin");
		}
		
		
		System.out.println("------- testAOP 실행됨 ------");
		
		return mav;
	}
}
```

-----

##### TestAController.java

```java
package com.gd.test.web.testa.controller;

import java.util.HashMap;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpSession;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.servlet.ModelAndView;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.gd.test.common.bean.PagingBean;
import com.gd.test.common.service.IPagingService;
import com.gd.test.util.Utils;
import com.gd.test.web.test.service.ITestLService;
import com.gd.test.web.test.service.ITestService;

@Controller
public class TestAController {
	@Autowired
	public ITestLService iTestLService;
	
	@Autowired
	public ITestService iTestService;
	
	@Autowired
	public IPagingService iPagingService;
	
	@RequestMapping(value = "/testALogin")
	public ModelAndView testALogin(HttpSession session, ModelAndView mav) {
		
		if(session.getAttribute("sMNo") != null) { // 로그인 된 상태
			mav.setViewName("redirect:testABList");						
		} else {
			mav.setViewName("testa/testALogin");			
		}
		
		return mav;
	}
	
	// RequestMapping : value - 주소
	//					method - 전송방식지정
	//					produces - 돌려줄 형식
	// text/json : text 또는 json
	@RequestMapping(value = "/testALogins",
					method = RequestMethod.POST,
					produces = "text/json;charset=UTF-8") // 오타 절대 금지
	@ResponseBody // Spring에 View임을 제시
	public String testALogins(HttpSession session, 
							  @RequestParam HashMap<String, String> params) throws Throwable {
		
		// ObjectMapper : 객체를 문자열로 변환(주체) - JackSon라이브러리
		ObjectMapper mapper = new ObjectMapper();
		// 데이터 보관용 map
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		// mPw의 값을 암호화 후 mPw로 넣겠다.
		params.put("mPw", Utils.encryptAES128(params.get("mPw")));
		
		// System.out.println(Utils.decryptAES128(params.get("mPw")));
		
		HashMap<String, String> data = iTestLService.getM(params);
		
		if(data != null) { // 사용자 정보가 있음
			session.setAttribute("sMNo", data.get("M_NO"));
			session.setAttribute("sMNm", data.get("M_NM"));
			modelMap.put("resMsg", "success");
		} else { // 사용자 정보가 없음
			modelMap.put("resMsg", "failed");
		}
		
		// writeValueAsString : 객체를 문자열로 변환(기능)
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testABList")
	public ModelAndView testABList(@RequestParam HashMap<String, String> params,
								   ModelAndView mav) {
		
		int page= 1;
		if(params.get("page") != null) {
			page = Integer.parseInt(params.get("page"));
		}
		
		mav.addObject("page", page);
		mav.setViewName("testa/testABList");
		
		return mav;
	}
	
	@RequestMapping(value = "/testABLists",
					method = RequestMethod.POST,
					produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testABLists(@RequestParam HashMap<String, String> params) throws Throwable {
		
		ObjectMapper mapper = new ObjectMapper();

		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		// 현재 페이지
		int page = Integer.parseInt(params.get("page"));
		
		// 총 게시글 수
		int cnt = iTestService.getBCnt(params);

		// 페이징 정보 취득
		PagingBean pb = iPagingService.getPagingBean(page, cnt);
		
		// 게시글 시작, 종료번호 할당
		params.put("startCnt", Integer.toString(pb.getStartCount()));
		params.put("endCnt", Integer.toString(pb.getEndCount()));
				
		// 목록 취득
		List<HashMap<String, String>> list = iTestService.getBList(params);
		
		modelMap.put("list", list);		
		modelMap.put("pb", pb);
		
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testALogout")
	public ModelAndView testALogout(HttpSession session, ModelAndView mav) {

		session.invalidate();
		
		mav.setViewName("redirect: testALogin");
		
		return mav;
	}
	
	@RequestMapping(value = "/testABWrite")
	public ModelAndView testABWrite(ModelAndView mav) {

		mav.setViewName("testa/testABWrite");

		
		return mav;
	}
	
	@RequestMapping(value = "/testABWrites",
			method = RequestMethod.POST,
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testABWrites(@RequestParam HashMap<String, String> params) throws Throwable {
	
		ObjectMapper mapper = new ObjectMapper();
	
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		try {
			int cnt = iTestService.addB(params);
			if(cnt > 0) {		
				modelMap.put("msg", "success");
			} else {
				modelMap.put("msg", "failed");
			}
			
		} catch (Throwable e) {
			e.printStackTrace();
			modelMap.put("msg", "error");
		}
	
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testAB")
	public ModelAndView testAB(@RequestParam HashMap<String, String> params,
							   ModelAndView mav) throws Throwable {
		
		HashMap<String, String> data = iTestService.getB(params);
		
		mav.addObject("data", data);
		
		mav.setViewName("testa/testAB");
		
		return mav;	
	}
	
	@RequestMapping(value = "/testABUpdate")
	public ModelAndView testABUpdate(@RequestParam HashMap<String, String> params,
									 ModelAndView mav) throws Throwable {
		
		HashMap<String, String> data = iTestService.getB(params);
		
		mav.addObject("data", data);
		
		mav.setViewName("testa/testABUpdate");
		
		return mav;	
	}
	
	@RequestMapping(value = "/testABUpdates",
			method = RequestMethod.POST,
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testABUpdates(@RequestParam HashMap<String, String> params) throws Throwable {
	
		ObjectMapper mapper = new ObjectMapper();
	
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		try {
			int cnt = iTestService.updateB(params);
			if(cnt > 0) {		
				modelMap.put("msg", "success");
			} else {
				modelMap.put("msg", "failed");
			}
			
		} catch (Throwable e) {
			e.printStackTrace();
			modelMap.put("msg", "error");
		}
	
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testABDeletes",
			method = RequestMethod.POST,
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testABDeletes(@RequestParam HashMap<String, String> params) throws Throwable {
	
		ObjectMapper mapper = new ObjectMapper();
	
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		try {
			int cnt = iTestService.deleteB(params);
			if(cnt > 0) {		
				modelMap.put("msg", "success");
			} else {
				modelMap.put("msg", "failed");
			}
			
		} catch (Throwable e) {
			e.printStackTrace();
			modelMap.put("msg", "error");
		}
	
		return mapper.writeValueAsString(modelMap);
	}
}
```
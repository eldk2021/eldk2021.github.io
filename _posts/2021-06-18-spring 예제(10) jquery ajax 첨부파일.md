---
title: spring 예제(10) jquery ajax 첨부파일
date: 2021-06-18
tags: spring jquery ajax
---




![FileUploadAjax 로직](/assets/images/FileUploadAjax 로직.JPG){: width="1000" height="500"}

-----

##### CommonProperties.java

```java
package com.spring.sample.common;

public class CommonProperties {
	/**
	 * 기본 셋
	 */
	//기본 리스트 사이즈
	public static final int VIEWCOUNT = 10;
	//기본 페이지 사이즈
	public static final int PAGECOUNT = 10;
	//검색 기준일
	public static final int SEARCHDATEAREA = 30;
	
	/**
	 * Ajax Result
	 */
	public static final String RESULT_SUCCESS = "SUCCESS";
	
	public static final String RESULT_ERROR = "ERROR";
	
	/**
	 * 파일 업로드
	 */
	//파일 업로드 경로
	public static final String FILE_UPLOAD_PATH = "C:\\MyWork\\workspace\\.metadata"
												+ "\\.plugins\\org.eclipse.wst.server.core"
												+ "\\tmp0\\wtpwebapps\\SampleSpring\\resources\\upload";
	
	//파일 다운로드 경로
	public static final String FILE_DOWNLOAD_PATH = "http://localhost:8080/sample";
	
	//허용파일 확장자
	public static final String FILE_EXT = "xls|ppt|doc|xlsx|pptx|docx|hwp|csv|jpg|jpeg|png|gif|bmp|tld|txt|pdf|zip|alz|7z|ico|mp4|avi";
	public static final String IMG_EXT = "jpg|jpeg|png|gif|bmp|ico";
	
	/**
	 * 암호화키(AES기반 16글자)
	 */
	public static final String SECURE_KEY = "goodeesmart12345";
}
```

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
	
	<context:component-scan base-package="com.spring.sample" />
	
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
					key="com.spring.sample.exception.UserNotFoundException">
					/exception/USER_NOT_FOUND_EXCEPTION
				</beans:prop>
			</beans:props>
		</beans:property>
	</beans:bean>
	
	<!-- Spring scheduler -->
	<task:scheduler id="scheduler" pool-size="50"/>
	<task:annotation-driven scheduler="scheduler"/>
	
	<!-- AOP -->
	<aop:aspectj-autoproxy />
	<beans:bean id="commonAop" class="com.spring.sample.common.controller.CommonAOP" /> 
</beans:beans>
```

-----

##### FileUploadController.java

```java
package com.spring.sample.web.fileUpload.controller;

import java.io.File;
import java.io.PrintWriter;
import java.util.ArrayList;
import java.util.HashMap;
import java.util.Iterator;
import java.util.List;
import java.util.Map;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.servlet.http.HttpSession;

import org.apache.commons.io.FilenameUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.ResponseBody;
import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.MultipartHttpServletRequest;
import org.springframework.web.servlet.ModelAndView;

import com.fasterxml.jackson.databind.ObjectMapper;
import com.spring.sample.common.CommonProperties;
import com.spring.sample.util.Utils;

@Controller
public class FileUploadController {
	private static final Logger logger = LoggerFactory.getLogger(FileUploadController.class);

	@RequestMapping(value = "/fileUpload", method = RequestMethod.GET)
	public ModelAndView fileUpload(HttpServletRequest request, HttpSession session, ModelAndView modelAndView) {

		modelAndView.setViewName("fileUpload/fileUpload");

		return modelAndView;
	}

	@RequestMapping(value = "/fileUploadAjax",
					method = RequestMethod.POST, 
					produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String fileUploadAjax(HttpServletRequest request, 
								 ModelAndView modelAndView) throws Throwable {
		ObjectMapper mapper = new ObjectMapper();
		HashMap<String, Object> modelMap = new HashMap<String, Object>();

		/* File Upload Logic */
		MultipartHttpServletRequest multipartRequest 
					= (MultipartHttpServletRequest) request;

		String uploadExts = CommonProperties.FILE_EXT;
		String uploadPath = CommonProperties.FILE_UPLOAD_PATH;
		String fileFullName = "";
		
		File folder = new File(uploadPath);

		if (!folder.exists()) { // 폴더 존재여부
			folder.mkdir(); // 폴더 생성
		}

		List<String> fileNames = new ArrayList<String>();
		try {
			/* @SuppressWarnings("rawtypes") */
			final Map<String, MultipartFile> files = multipartRequest.getFileMap(); // map<파일명, 파일>
			Iterator<String> iterator = multipartRequest.getFileNames(); // 파일명 취득

			while (iterator.hasNext()) { // 파일명 개수만큼 동작
				String key = iterator.next(); // 파일명 취득
				MultipartFile file = /* (MultipartFile) */ files.get(key); // 파일 취득
				if (file.getSize() > 0) { // 파일 크기가 0이 아닐 경우에만 업로드
					String fileRealName = file.getOriginalFilename(); // 실제파일명
					String fileTmpName = Utils.getPrimaryKey(); // 고유 날짜키 받기
					String fileExt = FilenameUtils.getExtension(
										file.getOriginalFilename()).toLowerCase(); // 파일확장자추출

					if (uploadExts.toLowerCase().indexOf(fileExt) < 0) {
						throw new Exception("Not allowded file extension : " 
												+ fileExt.toLowerCase());
					} else {
						// 물리적으로 저장되는 파일명(실제파일명을 그대로 저장할지 rename해서 저장할지는 협의 필요)
						fileFullName = fileTmpName + fileRealName;
						//File(경로) - 폴더
						//File(경로, 파일명) - 파일
						// new File(new File(uploadPath), fileFullName)
						// uploadPath경로의 폴더에 fileFullName 이름의 파일
						// 파일.transferTo(새파일) - 새파일에 파일의 내용을 전송
						file.transferTo(new File(new File(uploadPath), fileFullName));

						fileNames.add(fileFullName);
					}
				}
			}

			modelMap.put("result", CommonProperties.RESULT_SUCCESS);
		} catch (Exception e) {
			// 공통 Exception 처리
			e.printStackTrace();
			modelMap.put("result", CommonProperties.RESULT_ERROR);
		}

		modelMap.put("fileName", fileNames);

		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/imageUpload", method = RequestMethod.POST)
	public void editorImageUpload(HttpServletRequest request, HttpServletResponse response,
			@RequestParam MultipartFile upload, ModelAndView modelAndView) throws Throwable {
		PrintWriter printWriter = null;
		try {
			String uploadExts = CommonProperties.IMG_EXT; // 확장자
			String uploadPath = CommonProperties.FILE_UPLOAD_PATH; // 업로드경로
			String fileFullName = "";

			File fileDir = new File(uploadPath);

			if (!fileDir.exists()) {
				fileDir.mkdirs(); // 디렉토리가 존재하지 않는다면 생성
			}

			if (upload.getSize() > 0) {
				String fileRealName = upload.getOriginalFilename().replace(" ", "_").toLowerCase(); // 실제파일명
				String fileTmpName = Utils.getPrimaryKey(); // 고유 날짜키 받기
				String fileExt = FilenameUtils.getExtension(upload.getOriginalFilename()).toLowerCase(); // 파일

				if (uploadExts.toLowerCase().indexOf(fileExt) >= 0) {
					fileFullName = fileTmpName + fileRealName;
					upload.transferTo(new File(fileDir, fileFullName));

				} else {
					// 파일 확장자가 틀릴 경우
					printWriter = response.getWriter();

					printWriter.println("<script type='text/javascript'>alert('파일 확장자가 지원을 하지 않습니다.');</script>");
					printWriter.flush();
					printWriter.close();
				}

				// 성공 시
				String callback = request.getParameter("CKEditorFuncNum");

				printWriter = response.getWriter();

				printWriter.println("<script type='text/javascript'>" + "window.parent.CKEDITOR.tools.callFunction("
						+ callback + ",'" + "resources/upload/" + fileFullName + "','이미지를 업로드 하였습니다.'" + ")</script>");
				printWriter.flush();
				printWriter.close();

			} else {
				// 파일 크기가 0이거나 없는 경우
				printWriter = response.getWriter();

				printWriter.println("<script type='text/javascript'>alert('파일 업로드에 실패하였습니다.');</script>");
				printWriter.flush();
				printWriter.close();
			}
		} catch (Exception e) {
			e.printStackTrace();
		} finally {
			if (printWriter != null) {
				printWriter.close();
			}
		}
	}
}
```

-----

##### fileUpload.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core"%>
<!DOCTYPE html>
<html>
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>FileUpload Test</title>
<script type="text/javascript" 
		src="resources/script/jquery/jquery-1.12.4.min.js"></script>
<script type="text/javascript" 
		src="resources/script/jquery/jquery.form.js"></script>

<script type="text/javascript">
$(document).ready(function() {
	$("#uploadBtn").on("click", function(){
		var fileForm = $("#fileForm"); // form객체 취득
		
		// ajaxFrom : form을 submit할 경우 ajax의 형태로 구동하겠다.
		fileForm.ajaxForm({ //보내기전 validation check가 필요할경우
			// beforeSubmit : submit 실행 전 처리
			beforeSubmit: function (data, frm, opt) { 
				alert("전송전!!");
				return true;
			}, //submit이후의 처리
			success: function(res){
				if(res.result =="SUCCESS"){

					alert("저장완료");
					
					console.log(res);

				} else {
					alert("저장실패");
				} 
			}, //ajax error
			error: function(){
				alert("에러발생!!"); 
			}
		});
		
		fileForm.submit();
	});
});
</script>
</head>
<body>
<form id="fileForm" name="fileForm" action="fileUploadAjax" method="post" enctype="multipart/form-data">
<h3> 첨부파일</h3>
<table width="770"border="0" cellspacing="0" cellpadding="0" class="table_1">
	<colgroup>
		<col width="150px" />
		<col width="600px" />
	</colgroup>
	<tr>
		<th>첨부파일1</th>
		<td><input type="file" name="attFile1" size="85" /></td>
	</tr>
	<tr>
		<th>첨부파일2</th>
		<td><input type="file" name="attFile2" size="85" /></td>
	</tr>
	<tr>
		<th class="th_bot">첨부파일3</th>
		<td class="th_bot"><input type="file" name="attFile3" size="85" /></td>
	</tr>
</table>
</form>
<input type="button" value="저장" id="uploadBtn" />
</body>
</html>
```

-----

##### TestAContoller.java

```java
package com.spring.sample.web.testa.controller;

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
import com.spring.sample.common.bean.PagingBean;
import com.spring.sample.common.service.IPagingService;
import com.spring.sample.util.Utils;
import com.spring.sample.web.test.service.ITestLService;
import com.spring.sample.web.test.service.ITestService;

@Controller
public class TestAController {
	@Autowired
	public ITestLService iTestLService;
	
	@Autowired
	public ITestService iTestService;
	
	@Autowired
	public IPagingService iPagingService;
	
	@RequestMapping(value = "/testALogin")
	public ModelAndView testALogin(ModelAndView mav) {
		
		mav.setViewName("testa/testALogin");
		
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
		
		System.out.println(Utils.decryptAES128(params.get("mPw")));
		
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

-----

##### B_SQL.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="B"><!-- namespace - 클래스명과 동일 -->
	<!-- id - 메소드명과 동일 -->
	<!-- resultType : row 1줄의 형태를 지정 -->
	<!-- 쿼리 작성 시 ;이 들어가면 실행되지 않음 -->
	<select id="getBCnt" parameterType="hashmap" resultType="Integer">
		SELECT COUNT(*) AS CNT
		FROM B
		WHERE 1 = 1
		<if test="searchTxt != null and searchTxt != ''">
			<choose>
				<when test="searchGbn == 0">
					AND B_TITLE LIKE '%' || #{searchTxt} || '%'
				</when>
				<when test="searchGbn == 1">
					AND B_WRITER LIKE '%' || #{searchTxt} || '%'
				</when>
			</choose>
		</if>
	</select>
	<select id="getBList" parameterType="hashmap" resultType="hashmap">
		SELECT B.B_NO, B.B_TITLE, B.B_WRITER, B.B_DT, B.B_FILE
		FROM(SELECT B_NO, B_TITLE, B_WRITER, TO_CHAR(B_DT, 'YYYY-MM-DD') AS B_DT,
									B_FILE, ROW_NUMBER() OVER(ORDER BY B_NO DESC) AS RNUM
		     FROM B
		     WHERE 1 = 1
		     <if test="searchTxt != null and searchTxt != ''">
				<choose>
					<when test="searchGbn == 0">
						AND B_TITLE LIKE '%' || #{searchTxt} || '%'
					</when>
					<when test="searchGbn == 1">
						AND B_WRITER LIKE '%' || #{searchTxt} || '%'
					</when>
				</choose>
			</if>
		     ) B
		WHERE B.RNUM BETWEEN #{startCnt} AND #{endCnt}
	</select>
	
	<select id="getB" parameterType="hashmap" resultType="hashmap">
		SELECT B_NO, B_TITLE, B_WRITER, B_CON, TO_CHAR(B_DT, 'YYYY-MM-DD') AS B_DT,
						B_FILE, SUBSTR(B_FILE, 21) AS B_UFILE
		FROM B
		WHERE B_NO = #{bNo}
	</select>
	
	<insert id="addB" parameterType="hashmap"><!-- resultType 없음 -->
		INSERT INTO B(B_NO, B_TITLE, B_WRITER, B_CON, B_FILE)
		VALUES(B_SEQ.NEXTVAL, #{bTitle}, #{bWriter}, #{bCon}, #{bFile})
	</insert>
	
	<update id="updateB" parameterType="hashmap">
		UPDATE B SET B_TITLE = #{bTitle}, B_CON = #{bCon}, B_FILE = #{bFile}
		WHERE B_NO = #{bNo}
	</update>
	
	<delete id="deleteB" parameterType="hashmap">
		DELETE FROM B
		WHERE B_NO = #{bNo}
	</delete>
	
</mapper>
```

-----

##### testABList.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.06.11</title>
<style type="text/css">
.list_wrap table {
	border-collapse: collapse;
}

.list_wrap thead tr {
	border-top: 1px solid #000;
	border-bottom: 1px solid #000;
	background-color: #66b5ff;
	height: 30px;
}

.list_wrap tbody tr {
	border-bottom: 1px solid #000;
	height: 25px;
	text-align: center;
	cursor: pointer;
}

.list_wrap tbody tr td:nth-child(2) {
	text-align: left;
}

.list_wrap tbody tr:nth-child(2n) {
	background-color: #ccebff;
}

.list_wrap tbody tr:hover, .list_wrap tbody tr:nth-child(2n):hover {
	background-color: #66c2ff;
	color: white;
}

.list_wrap tbody img {
	width: 15px;
}

.paging_wrap {
	margin-top: 10px;
}

.paging_wrap div {
	display: inline-block;
	padding: 5px;
	margin-left: 3px;
	margin-right: 3px;
	background-color: #cce6ff;
	border: 1px solid #444;
	border-radius: 3px;
	cursor: pointer;
	width: 80px;
	text-align: center;
}

.paging_wrap div:active {
	background-color: #80c1ff;
}

.paging_wrap .on {
	background-color: #80c1ff;
}
</style>
<script type="text/javascript" src="resources/script/jquery/jquery-1.12.4.min.js"></script>
<script type="text/javascript">
	$(document).ready(function() {
		if("${param.searchGbn}" != "") {
			$("#searchGbn").val("${param.searchGbn}");
		}
		
		reloadList();
		
		$("#loginBtn").on("click", function() {
			location.href = "testALogin";
		});
		
		$("#logoutBtn").on("click", function() {
			location.href = "testALogout";
		});
		
		$("#searchBtn").on("click", function() {
			$("#page").val(1);
			$("#searchOldTxt").val($("#searchTxt").val());
			reloadList();
		});
		
		$(".paging_wrap").on("click", "div",  function() {
			$("#page").val($(this).attr("page"));
			$("#searchTxt").val($("#searchOldTxt").val());
			reloadList();
		});
		
		$("#writeBtn").on("click",  function() {
			$("#searchTxt").val($("#searchOldTxt").val());
			$("#actionForm").attr("action", "testABWrite");
			$("#actionForm").submit();
		});
		
		$(".list_wrap tbody").on("click", "tr", function() {
			$("#bNo").val($(this).attr("bno"));
			$("#searchTxt").val($("#searchOldTxt").val());
			$("#actionForm").attr("action", "testAB");
			$("#actionForm").submit();
		});
	}); // document ready end
	
	function reloadList() {
		var params= $("#actionForm").serialize();
		
		$.ajax({
			url: "testABLists", // 접속 주소
			type: "post", // 전송 방식: get, post
			dataType: "json", // 받아올 데이터 형태
			data: params, // 보낼 데이터(문자열 형태)
			success: function(res) { // 성공 시 다음 함수 실행
				drawList(res.list);
				drawPaging(res.pb);		
			},
			error: function(request, status, error) { // 실패 시 다음 함수 실행
				console.log(error);
			}
		});
	}
	
	// 목록 그리기
	function drawList(list) {
		var html ="";
	// + " " + 만들어 놓으면 편함
		for(var d of list) {
			html += "<tr bno=\"" + d.B_NO + "\">";
			html += "<td>" + d.B_NO + "</td>";
			html += "<td>" + d.B_TITLE;
			if(d.B_FILE != null) {
				html += "<img src=\"resources/images/attFile.png\" alt=\"첨부파일\" />";
			}
			html += "</td>";
			html += "<td>" + d.B_WRITER + "</td>";
			html += "<td>" + d.B_DT + "</td>";
			html += "</tr>";
		}
		
		$(".list_wrap tbody").html(html);
	}
	// 페이징 그리기
	function drawPaging(pb) {
		var html ="";
		
		html += "<div page=\"1\">처음</div>";
		if($("#page").val() == "1") {
			html += "<div page=\"1\">이전</div>";		
		} else {
			html += "<div page=\"" + ($("#page").val() - 1) + "\">이전</div>"; // 마이너스는 문자열에서 마이너스가 적용되지 않기 때문에 바로 사용가능
		}
		
		for(var i = pb.startPcount ; i <= pb.endPcount; i++){
			if($("#page").val() == i) {
				html += "<div class=\"on\" page=\"" + i + "\">" + i + "</div>";			
			} else {
				html += "<div page=\"" + i + "\">" + i + "</div>";			
				
			}
		}
		
		if($("#page").val() == pb.maxPcount) {
			html += "<div page=\"" + pb.maxPcount + "\">다음</div>";
		} else {
			html += "<div page=\"" + ($("#page").val() * 1 + 1) + "\">다음</div>"; // 문자열에 그냥 + 1하게 되면 11이 되기 때문에 * 1 해줘야함
		}
		
		html += "<div page=\"" + pb.maxPcount + "\">마지막</div";
		
		$(".paging_wrap").html(html);
	}
</script>
</head>
<body>
	<c:choose>
		<c:when test="${empty sMNo}">
			<input type="button" value="로그인" id="loginBtn" />
		</c:when>
		<c:otherwise>
			${sMNm}님 어서오세요.<input type="button" value="로그아웃" id="logoutBtn" />
		</c:otherwise>
	</c:choose>
	<div class="search_area">
		<form action="#" id="actionForm" method="post">
			<input type="hidden" id="bNo" name="bNo" />
			<input type="hidden" id="page" name="page" value="${page}" />
			<select id="searchGbn" name="searchGbn">
				<option value="0">제목</option>
				<option value="1">작성자</option>
			</select>
			<input type="hidden" id="searchOldTxt" value="${param.searchTxt}" />
			<input type="text" name="searchTxt" id="searchTxt" value="${param.searchTxt}" />
			<input type="button" value="검색" id="searchBtn" />
			<input type="button" value="작성" id="writeBtn" />
		</form>
	</div>
	<div class="list_wrap">
		<table>
			<colgroup>
				<col width="100px" />
				<col width="400px" />
				<col width="200px" />
				<col width="100px" />
			</colgroup>
			<thead>
				<tr>
					<th>번호</th>
					<th>제목</th>
					<th>작성자</th>
					<th>작성일</th>
				</tr>
			</thead>
			<tbody></tbody>
		</table>
	</div>
	<div class="paging_wrap"></div>
</body>
</html>
```

-----

##### testAB.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!-- ELTag 확장기능 -->
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.06.14</title>
<script type="text/javascript" src="resources/script/jquery/jquery-1.12.4.min.js"></script>
<script type="text/javascript">
	$(document).ready(function() {
		$("#listBtn").on("click", function() {
			$("#goForm").attr("action", "testABList");
			$("#goForm").submit();
		});
		
		$("#updateBtn").on("click", function() {
			$("#goForm").attr("action", "testABUpdate");
			$("#goForm").submit();
		});
		
		$("#deleteBtn").on("click", function() {
			if(confirm("삭제하시겠습니까?")) {
				var params= $("#goForm").serialize();
				
				$.ajax({
					url: "testABDeletes", // 접속 주소
					type: "post", // 전송 방식: get, post
					dataType: "json", // 받아올 데이터 형태
					data: params, // 보낼 데이터(문자열 형태)
					success: function(res) { // 성공 시 다음 함수 실행
						if(res.msg == "success") {
							location.href = "testABList";
						} else if(res.msg == "failed") {
							alert("삭제에 실패하였습니다.")
						} else {
							alert("삭제 중 문제가 발생하였습니다.")
						}
					},
					error: function(request, status, error) { // 실패 시 다음 함수 실행
						console.log(error);
					}
				});
			}
		});
	});
</script>
</head>
<body>
	<form action="#" id="goForm" method="post">
		<input type="hidden" name="bNo" value="${data.B_NO}" />
		<input type="hidden" name="page" value="${param.page}" />
		<input type="hidden" name="searchGbn" value="${param.searchGbn}" />
		<input type="hidden" name="searchTxt" value="${param.searchTxt}" />
	</form>
	번호 : ${data.B_NO}<br/>
	제목 : ${data.B_TITLE}<br/>
	작성자 : ${data.B_WRITER}<br/>
	작성일 : ${data.B_DT}<br/>
	내용<br/>
	${data.B_CON}<br/>
	<c:if test="${!empty data.B_FILE}">
	<!-- set : 변수 -->
	<%-- <c:set var="len" value="${fn:length(data.B_FILE)}"></c:set>
		첨부파일 :
		<!-- a의 download : 해당 주소를 다운로드하겠다. 값이 있는 경우 해당 이름으로 다운받겠다. -->
		<a href="resources/upload/${data.B_FILE}" download="${fn:substring(data.B_FILE, 20, len)}">
			${fn:substring(data.B_FILE, 20, len)}
		</a>
	<br/> --%>
	첨부파일 : <a href="resources/upload/${data.B_FILE}" download="${data.B_UFILE}">
			${data.B_UFILE}
		</a>
	<br/>
	</c:if>
	<input type="button" value="수정" id="updateBtn" />
	<input type="button" value="삭제" id="deleteBtn" />
	<input type="button" value="목록으로" id="listBtn" />
</body>
</html>
```

-----

##### testABWrite.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.06.14, 2021.06.17</title>
<style type="text/css">
#att {
	display: none;
}

#addBtn {
	margin-top: 20px;
}
</style>
<script type="text/javascript" src="resources/script/jquery/jquery-1.12.4.min.js"></script>
<script type="text/javascript" src="resources/script/jquery/jquery.form.js"></script>
<script type="text/javascript" src="resources/script/ckeditor/ckeditor.js"></script>
<script type="text/javascript">
	$(document).ready(function(){
		CKEDITOR.replace("bCon", {
			resize_enabled : false,
			language : "ko",
			enterMode : "2",
			width: "600",
			height: "300"
		});
		
		$("#fileBtn").on("click", function () {
			$("#att").click();
		});
		
		$("#att").on("change", function () {
			$("#fileName").html($(this).val().substring($(this).val().lastIndexOf("\\") + 1));
		});
	
		$("#listBtn").on("click", function() {
			$("#goForm").submit();
		});
		
		$("#addForm").on("keypress", "input", function(event) {
			if(event.keyCode == 13) { // 엔터키를 눌렀을 때
				return false; // 페이지가 안넘어간다.
			}
		});
		
		$("#addBtn").on("click", function() {
			
			var fileForm = $("#fileForm");
			
			fileForm.ajaxForm({
				beforeSubmit : function() {
					$("#bCon").val(CKEDITOR.instances['bCon'].getData());
					
					if($.trim($("#bTitle").val()) == "") {
						alert("제목을 입력해주세요.");
						$("#bTitle").focus();
						return false; // ajaxForm 실행 불가
					} else  if($.trim($("#bWriter").val()) == "") {
						alert("작성자를 입력해주세요.");
						$("#bWriter").focus();
						return false;
					} else if($.trim($("#bCon").val()) == "") {
						alert("내용을 입력해주세요.");
						$("#bCon").focus();
						return false;
					}
				},
				success : function(res) {
					if(res.result = "SUCCESS") {
						 // 올라간 파일명 저장
						 if(res.fileName.length > 0) {
							 $("#bFile").val(res.fileName[0]);
						 }
						 // 글저장
						 var params= $("#addForm").serialize();
							
							$.ajax({
								url: "testABWrites", // 접속 주소
								type: "post", // 전송 방식: get, post
								dataType: "json", // 받아올 데이터 형태
								data: params, // 보낼 데이터(문자열 형태)
								success: function(res) { // 성공 시 다음 함수 실행
									if(res.msg == "success") {
										location.href = "testABList";
									} else if(res.msg == "failed") {
										alert("작성에 실패하였습니다.")
									} else {
										alert("작성 중 문제가 발생하였습니다.")
									}
								},
								error: function(request, status, error) { // 실패 시 다음 함수 실행
									console.log(error);
								}
							});
					} else {
						alert("파일업로드 중 문제 발생");
					}
				},
				error : function() {
					alert("파일업로드 중 문제 발생");
				} 
			}); // ajaxForm End
			
			fileForm.submit();
		});	// addBtn click End
}); // document ready End
</script>
</head>
<body>
	<form id="fileForm" action="fileUploadAjax" method="post" enctype="multipart/form-data">
		<input type="file" name="att" id="att" />
	</form>
	<form action="testABList" id="goForm" method="post">
		<input type="hidden" name="page" value="${param.page}" />
		<input type="hidden" name="searchGbn" value="${param.searchGbn}" />
		<input type="hidden" name="searchTxt" value="${param.searchTxt}" />
	</form>
	<form action="#" id="addForm" method="post">
		 제목 <input type="text" id="bTitle" name="bTitle" /><br/>
		 작성자 <input type="text" id="bWriter" name="bWriter" value="${sMNm}" /><br/>
		 내용<br/>
		 <textarea rows="20" cols="50" id="bCon" name="bCon"></textarea><br/>
		  첨부파일
		 <input type="button" value="첨부파일선택" id="fileBtn" />
		 <span id="fileName"></span>
		 <input type="hidden" name="bFile" id="bFile" /> <!-- 보내기용 -->
	</form>
	<input type="button" value="등록" id="addBtn" />
	<input type="button" value="목록으로" id="listBtn" />
</body>
</html>
```

-----

##### testABUpdate.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.06.14, 2021.06.18</title>
<style type="text/css">
.off_btn {
	display: none;
}

#att {
	display: none;
}
</style>
<script type="text/javascript" src="resources/script/jquery/jquery-1.12.4.min.js"></script>
<script type="text/javascript" src="resources/script/jquery/jquery.form.js"></script>
<script type="text/javascript" src="resources/script/ckeditor/ckeditor.js"></script>
<script type="text/javascript">
	$(document).ready(function(){
		CKEDITOR.replace("bCon", {
			resize_enabled : false,
			language : "ko",
			enterMode : "2",
			width: "600",
			height: "300"
		});
		
		$("#fileBtn").on("click", function () {
			$("#att").click();
		});
		
		$("#att").on("change", function () {
			$("#fileName").html($(this).val().substring($(this).val().lastIndexOf("\\") + 1));
		});
		
		$("#fileDelBtn").on("click", function () {
			$("#fileName").html("");
			$("#bFile").val("");
			$("#fileBtn").attr("class", "");
			$(this).remove();
		});
	
		$("#backBtn").on("click", function() {
			history.back();
		});
		
		$("#updateForm").on("keypress", "input", function(event) {
			if(event.keyCode == 13) { // 엔터키를 눌렀을 때
				return false; // 페이지가 안넘어간다.
			}
		});
		
		$("#updateBtn").on("click", function() {
			
			var fileForm = $("#fileForm");
			
			fileForm.ajaxForm({
				beforeSubmit : function() {
					$("#bCon").val(CKEDITOR.instances['bCon'].getData());
					
					if($.trim($("#bTitle").val()) == "") {
						alert("제목을 입력해주세요.");
						$("#bTitle").focus();
						return false; // ajaxForm 실행 불가
					} else if($.trim($("#bCon").val()) == "") {
						alert("내용을 입력해주세요.");
						$("#bCon").focus();
						return false;
					}
				},
				success : function(res) {
					if(res.result = "SUCCESS") {
						 // 올라간 파일명 저장
						 if(res.fileName.length > 0) {
							 $("#bFile").val(res.fileName[0]);
						 }
						 // 글수정
						 var params= $("#updateForm").serialize();
							
							$.ajax({
								url: "testABUpdates", // 접속 주소
								type: "post", // 전송 방식: get, post
								dataType: "json", // 받아올 데이터 형태
								data: params, // 보낼 데이터(문자열 형태)
								success: function(res) { // 성공 시 다음 함수 실행
									if(res.msg == "success") {
										$("#updateForm").attr("action", "testAB");
										$("#updateForm").submit();
									} else if(res.msg == "failed") {
										alert("수정에 실패하였습니다.")
									} else {
										alert("수정 중 문제가 발생하였습니다.")
									}
								},
								error: function(request, status, error) { // 실패 시 다음 함수 실행
									console.log(error);
								}
							});				 
					} else {
						alert("파일업로드 중 문제 발생");
					}
				},
				error : function() {
					alert("파일업로드 중 문제 발생");
				} 
			}); // ajaxForm End
			
			fileForm.submit();
			
	}); // upadateBtn click End
}); // document ready End
</script>
</head>
<body>
	<form id="fileForm" action="fileUploadAjax" method="post" enctype="multipart/form-data">
		<input type="file" name="att" id="att" />
	</form>
	<form action="#" id="updateForm" method="post">
		<input type="hidden" name="page" value="${param.page}" />
		<input type="hidden" name="searchGbn" value="${param.searchGbn}" />
		<input type="hidden" name="searchTxt" value="${param.searchTxt}" />
		<input type="hidden" name="bNo" value="${data.B_NO}" />
		 제목 <input type="text" id="bTitle" name="bTitle" value="${data.B_TITLE}"/><br/>
		 작성자: ${data.B_WRITER}<br/>
		 내용<br/>
		 <textarea rows="20" cols="50" id="bCon" name="bCon">${data.B_CON}</textarea><br/>
		 <input type="hidden" id="bFile" name="bFile" value="${data.B_FILE}" />
		   첨부파일
		  <c:choose>
		  	<c:when test="${!empty data.B_FILE}">
			  <input type="button" class="off_btn" value="첨부파일선택" id="fileBtn" />
		  	</c:when>
		  	<c:otherwise>
			  <input type="button" value="첨부파일선택" id="fileBtn" />
		  	</c:otherwise>
		  </c:choose>
		 <span id="fileName">${data.B_UFILE}</span>
		 <c:if test="${!empty data.B_FILE}">
		 	<input type="button" value="파일삭제" id="fileDelBtn" />
		 </c:if>
	</form>
	<input type="button" value="수정" id="updateBtn" />
	<input type="button" value="뒤로가기" id="backBtn" />
</body>
</html>
```
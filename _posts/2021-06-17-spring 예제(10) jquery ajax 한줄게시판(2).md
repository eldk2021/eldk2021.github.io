---
title: spring 예제(10) jquery ajax 한줄게시판(2)
date: 2021-06-17
tags: spring jquery ajax
---



##### TestL2Contoller.java

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
import com.spring.sample.web.testa.service.ITestL2Service;

@Controller
public class TestL2Controller {
	@Autowired
	public ITestL2Service iTestL2Service;
	
	@Autowired
	public IPagingService iPagingService;
	
	@RequestMapping(value = "/testLogin2")
	public ModelAndView testAMLogin(ModelAndView mav) {
		
		mav.setViewName("testa/testLogin2");
		
		return mav;
	}
	
	@RequestMapping(value = "/testLogin2s",
					method = RequestMethod.POST,
					produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testLogin2s(HttpSession session, 
							  @RequestParam HashMap<String, String> params) throws Throwable {
		
		ObjectMapper mapper = new ObjectMapper();
		
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		HashMap<String, String> data = iTestL2Service.getM(params);
		
		if(data != null) {
			session.setAttribute("sMNo", data.get("M_NO"));
			session.setAttribute("sMNm", data.get("M_NM"));
			modelMap.put("resMsg", "success");
		} else {
			modelMap.put("resMsg", "failed");
		}
		
		return mapper.writeValueAsString(modelMap);
	}
	
	
	@RequestMapping(value = "/testLogout2")
	public ModelAndView testLogout(HttpSession session, ModelAndView mav) {

		session.invalidate();
		
		mav.setViewName("redirect:testLogin2");
		
		return mav;
	}
	
	@RequestMapping(value = "/testO2")
	public ModelAndView testO2(@RequestParam HashMap<String, String> params,
							  ModelAndView mav) throws Throwable {
		
		int page = 1;
		
		if(params.get("page") != null) {
			page = Integer.parseInt(params.get("page"));
		}
		
		mav.addObject("page", page);
		
		mav.setViewName("testa/testO2");
		
		return mav;
	}
	
	@RequestMapping(value = "/testO2s",
			method = RequestMethod.POST,
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testAMLists(@RequestParam HashMap<String, String> params) throws Throwable {
	
		ObjectMapper mapper = new ObjectMapper();
		
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		// 현재 페이지
		int page = Integer.parseInt(params.get("page"));
		
		// 총 게시글 수
		int cnt = iTestL2Service.getObCnt(params);
		
		// 페이징 정보 취득
		PagingBean pb = iPagingService.getPagingBean(page, cnt);
		
		// 게시글 시작, 종료번호 할당
		params.put("startCnt", Integer.toString(pb.getStartCount()));
		params.put("endCnt", Integer.toString(pb.getEndCount()));
				
		// 목록 취득
		List<HashMap<String, String>> list = iTestL2Service.getObList(params);
		
		modelMap.put("list", list);		
		modelMap.put("pb", pb);
	
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testO2Write",
			method = RequestMethod.POST,
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testO2Write(@RequestParam HashMap<String, String> params) throws Throwable {
		
		ObjectMapper mapper = new ObjectMapper();
		
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		System.out.println(params);
		
		try {
			int cnt = iTestL2Service.writeOb(params);
			System.out.println(cnt);
		
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
	
	@RequestMapping(value = "/testO2Update",
			method = RequestMethod.POST,
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testO2Update(@RequestParam HashMap<String, String> params) throws Throwable {
					
		ObjectMapper mapper = new ObjectMapper();
		
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		try {
			int cnt = iTestL2Service.updateOb(params);
			if(cnt > 0) {		
				modelMap.put("msg", "success");
			} else {
				modelMap.put("msg", "failed");
			}
			
		} catch(Throwable e) {
			e.printStackTrace();
			modelMap.put("msg", "error");
		}
		
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testO2Delete",
			method = RequestMethod.POST,
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testO2Delete(@RequestParam HashMap<String, String> params) throws Throwable {
				
		ObjectMapper mapper = new ObjectMapper();
		
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		System.out.println(params);
		
		try {
			int cnt = iTestL2Service.deleteOb(params);
		
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

##### ITestL2Service.java

```java
package com.spring.sample.web.testa.service;

import java.util.HashMap;
import java.util.List;

public interface ITestL2Service {

	public HashMap<String, String> getM(HashMap<String, String> params) throws Throwable;

	public int writeOb(HashMap<String, String> params) throws Throwable;

	public int getObCnt(HashMap<String, String> params) throws Throwable;

	public List<HashMap<String, String>> getObList(HashMap<String, String> params) throws Throwable;

	public int updateOb(HashMap<String, String> params) throws Throwable;

	public int deleteOb(HashMap<String, String> params) throws Throwable;

}
```

-----

##### TestL2Service.java

```java
package com.spring.sample.web.testa.service;

import java.util.HashMap;
import java.util.List;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.spring.sample.web.testa.dao.ITestL2Dao;

@Service
public class TestL2Service implements ITestL2Service {
	@Autowired
	public ITestL2Dao iTestL2Dao;
	
	@Override
	public HashMap<String, String> getM(HashMap<String, String> params) throws Throwable {
		return iTestL2Dao.getM(params);
	}

	@Override
	public int writeOb(HashMap<String, String> params) throws Throwable {
		return iTestL2Dao.writeOb(params);
	}

	@Override
	public int getObCnt(HashMap<String, String> params) throws Throwable {
		return iTestL2Dao.getObCnt(params);
	}

	@Override
	public List<HashMap<String, String>> getObList(HashMap<String, String> params) throws Throwable {
		return iTestL2Dao.getObList(params);
	}

	@Override
	public int updateOb(HashMap<String, String> params) throws Throwable {
		return iTestL2Dao.updateOb(params);
	}

	@Override
	public int deleteOb(HashMap<String, String> params) throws Throwable {
		return iTestL2Dao.deleteOb(params);
	}
}
```

-----

##### ITestL2Dao.java

```java
package com.spring.sample.web.testa.dao;

import java.util.HashMap;
import java.util.List;

public interface ITestL2Dao {

	public HashMap<String, String> getM(HashMap<String, String> params) throws Throwable;

	public int writeOb(HashMap<String, String> params) throws Throwable;

	public int getObCnt(HashMap<String, String> params) throws Throwable;

	public List<HashMap<String, String>> getObList(HashMap<String, String> params) throws Throwable;

	public int updateOb(HashMap<String, String> params) throws Throwable;

	public int deleteOb(HashMap<String, String> params) throws Throwable;
}
```

------

##### TestL2Dao.java

```java
package com.spring.sample.web.testa.dao;

import java.util.HashMap;
import java.util.List;

import org.apache.ibatis.session.SqlSession;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;

@Repository
public class TestL2Dao implements ITestL2Dao {
	@Autowired
	public SqlSession sqlSession;
	
	@Override
	public HashMap<String, String> getM(HashMap<String, String> params) throws Throwable {
		return sqlSession.selectOne("L2.getM", params);
	}

	@Override
	public int writeOb(HashMap<String, String> params) throws Throwable {
		return sqlSession.insert("L2.writeOb", params);
	}

	@Override
	public int getObCnt(HashMap<String, String> params) throws Throwable {
		return sqlSession.selectOne("L2.getObCnt", params);
	}

	@Override
	public List<HashMap<String, String>> getObList(HashMap<String, String> params) throws Throwable {
		return sqlSession.selectList("L2.getObList", params);
	}

	@Override
	public int updateOb(HashMap<String, String> params) throws Throwable {
		return sqlSession.update("L2.updateOb", params);
	}

	@Override
	public int deleteOb(HashMap<String, String> params) throws Throwable {
		return sqlSession.update("L2.deleteOb", params);
	}
}
```

------

##### L2_SQL.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="L2">
	<select id="getM" parameterType="hashmap" resultType="hashmap">
		SELECT M_NO, M_NM
		FROM M
		WHERE M_ID = #{mId}
		AND M_PW = #{mPw}
	</select>
	
	<insert id="writeOb" parameterType="hashmap">
		INSERT INTO OB(OB_NO, M_NO, OB_CON)
		VALUES(OB_SEQ.NEXTVAL, #{mNo} , #{obCon})
	</insert>
	
	<select id="getObCnt" parameterType="hashmap" resultType="Integer">
		SELECT COUNT(*) AS CNT
		FROM OB O INNER JOIN M M
		        ON O.M_NO = M.M_NO
		WHERE O.OB_DEL = 1
		<if test="searchTxt != null and searchTxt != ''">
			<choose>
				<when test="searchGbn == 0">
					AND M.M_NM LIKE '%' || #{searchTxt} || '%'
				</when>
				<when test="searchGbn == 1">
					AND O.OB_CON LIKE '%' || #{searchTxt} || '%'
				</when>
			</choose>
		</if>
	</select>
	
	<select id="getObList" parameterType="hashmap" resultType="hashmap">
		SELECT OM.OB_NO, OM.OB_CON, OM.M_NO, OM.M_NM
		FROM (
		        SELECT O.OB_NO, O.OB_CON, M.M_NO, M.M_NM, ROW_NUMBER() OVER(ORDER BY O.OB_NO DESC) AS RNUM
		        FROM OB O INNER JOIN M M
		                ON O.M_NO = M.M_NO
		        WHERE O.OB_DEL = 1
		        <if test="searchTxt != null and searchTxt != ''">
				<choose>
					<when test="searchGbn == 0">
						AND M.M_NM LIKE '%' || #{searchTxt} || '%'
					</when>
					<when test="searchGbn == 1">
						AND O.OB_CON LIKE '%' || #{searchTxt} || '%'
					</when>
				</choose>
			</if>
		    ) OM
		WHERE OM.RNUM BETWEEN #{startCnt} AND #{endCnt}
	</select>
	
	<update id="updateOb" parameterType="hashmap">
		UPDATE OB SET OB_CON = #{obCon}
		WHERE OB_NO = #{obNo}
		AND M_NO = #{mNo}
	</update>
	
	<update id="deleteOb" parameterType="hashmap">
		UPDATE OB SET OB_DEL = 0
		WHERE OB_NO = #{obNo}
		AND M_NO = #{mNo}
	</update>
</mapper>
```

------

##### testLogin2.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.06.09</title>
<script type="text/javascript" src="resources/script/jquery/jquery-1.12.4.min.js"></script>
<script type="text/javascript">
	$(document).ready(function() {
		
		$("#loginForm").on("keypress", "input", function(event) {
			if(event.keyCode == 13) { // 엔터키를 눌렀을 때
				return false; // 페이지가 안넘어간다.
			}
		});
		
		$("#loginBtn").on("click", function() {
			if($.trim($("#mId").val()) == "") {
				alert("아이디를 입력해주세요.");
				$("#mId").focus();
			} else if($.trim($("#mPw").val()) == "") {
				alert("비밀번호를 입력해주세요.");
				$("#mPw").focus();
			} else {
				var params = $("#loginForm").serialize();
				
				// ajax
				$.ajax({
					url: "testLogin2s", // 접속 주소
					type: "post", // 전송 방식: get, post
					dataType: "json", // 받아올 데이터 형태
					data: params, // 보낼 데이터(문자열 형태)
					success: function(res) { // 성공 시 다음 함수 실행
						if(res.resMsg == "success") {
							location.href = "testO2";
						} else {
							alert("아이디 또는 비밀번호가 일치하지 않습니다.")
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
	<form action="#" id="loginForm" method="post">
	 	아이디 <input type="text" id="mId" name="mId" /><br/>
	 	비밀번호 <input type="password" id="mPw" name="mPw" /><br/>
	 	<input type="button" value="로그인" id="loginBtn" />
	</form>
</body>
</html>
```

-----

##### testO2.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>    
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.06.16</title>
<style type="text/css">
#actionForm #updateBtn, #actionForm #cancelBtn {
	display: none;
}

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
		
		list();
		
		$("#logoutBtn").on("click",  function() {
			location.href = "testLogout2";
		});
		
		$("body").on("click", "#loginBtn", function() {
			location.href = "testLogin2";
		});
		
		// 작성 버튼
		$("#writeBtn").on("click",  function() {
			if($.trim($("#obCon").val()) == "") {
				alert("내용을 넣어주세요.");
				$("#obCon").focus();				
			} else {
				$("#searchTxt").val($("#searchOldTxt").val());
				
				var params = $("#actionForm").serialize();
				
				$.ajax({
					url: "testO2Write",
					type: "post",
					dataType: "json",
					data: params,
					success: function(res) {
						if(res.msg == "success") {
							location.href = "testO2";
						} else if(res.msg == "failed") {
							alert("작성에 실패하였습니다.")
						} else {
							alert("작성 중 문제가 발생하였습니다.")
						}	
					},
					error: function(request, status, error) {
						console.log(error);
					}
				});
			}
		});
		
		// 검색 버튼
		$("#searchBtn").on("click",  function() {
			$("#page").val(1);
			$("#sg").val($("#searchGbn").val());
			$("#st").val($("#searchTxt").val());
			$("#searchOldTxt").val($("#searchTxt").val());
			
			list();
		});
		
		// 페이징 이벤트
		$(".paging_wrap").on("click", "div", function() {
			$("#page").val($(this).attr("page"));
			$("#searchTxt").val($("#searchOldTxt").val());
			list();
		});
		
		// 목록의 수정버튼 클릭
		$("table").on("click", "#updateBtn", function() {
			$("#actionForm #writeBtn").hide();
			$("#actionForm #updateBtn, #actionForm #cancelBtn").show();
			
						// button    td       tr
			$("#obNo").val($(this).parent().parent().attr("ono"));
			
			// nth-child : 몇개째
			// console.log($(this).parent().parent().children(":nth-child(2)").html());
			// eq : 몇번째 인덱스 요소
			// console.log($(this).parent().parent().children(":eq(1)").html());
			// console.log($(this).parent().parent().children().eq(1).html());
			
			$("#obCon").val($(this).parent().parent().children(":nth-child(2)").html());
		});
		
		// 취소 버튼
		$("#cancelBtn").on("click", function() {
			$("#obCon").val("");
			$("#obNo").val("");
			
			$("#actionForm #writeBtn").show();
			$("#actionForm #updateBtn, #actionForm #cancelBtn").hide();	
		});
		
		// 수정 버튼
		$("#actionForm #updateBtn").on("click", function() {
			if($.trim($("#obCon").val()) == "") {
				alert("내용을 넣어주세요.");
				$("#obCon").focus();				
			} else {
				var params = $("#actionForm").serialize();
				
				$.ajax({
					url: "testO2Update",
					type: "post",
					dataType: "json",
					data: params,
					success: function(res) {
						if(res.msg == "success") {
							location.href = "testO2";
						} else if(res.msg == "failed") {
							alert("수정에 실패하였습니다.")
						} else {
							alert("수정 중 문제가 발생하였습니다.")
						}		
					},
					error: function(request, status, error) {
						console.log(error);
					}
				});
			}
		});
		
		// 삭제 버튼
		$("table").on("click", "#deleteBtn", function() {
			if(confirm("삭제하시겠습니까?")) {
				$("#obNo").val($(this).parent().parent().attr("ono"));
				
				var params = $("#actionForm").serialize();
				
				$.ajax({
					url: "testO2Delete",
					type: "post",
					dataType: "json",
					data: params,
					success: function(res) {
						if(res.msg == "success") {
							location.href = "testO2";
						} else if(res.msg == "failed") {
							alert("삭제에 실패하였습니다.")
						} else {
							alert("삭제 중 문제가 발생하였습니다.")
						}
					},
					error: function(request, status, error) {
						console.log(error);
					}
				});
			}
		});
	});
	
	function list() {
		var params = $("#actionForm").serialize();
		
		$.ajax({
			url: "testO2s",
			type: "post",
			dataType: "json",
			data: params,
			success: function(res) {
				drawList(res.list);
				drawPaging(res.pb);		
			},
			error: function(request, status, error) {
				console.log(error);
			}
		});
	}
	
	
	function drawList(list) {
		var html ="";

		for(var o of list) {
			html += "<tr ono=\"" + o.OB_NO + "\">";
			html += "<td>" + o.M_NM + "</td>";
			html += "<td>" + o.OB_CON + "</td>";
			html +=	"<td>";
			if("${sMNo}" == o.M_NO) {
				html +=	"<input type=\"button\" value=\"수정\" id=\"updateBtn\" />";
				html += "<input type=\"button\" value=\"삭제\" id=\"deleteBtn\" />"
			}	
			html += "</td>";
			html += "</tr>";
		}
		
		$(".list_wrap tbody").html(html);
	}
	
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
		<!-- 비로그인 -->
			<input type="button" value="로그인" id="loginBtn"/>
		</c:when>
		<c:otherwise>
		<!-- 로그인 -->
			${sMNm}님 어서오세요.<input type="button" value="로그아웃" id="logoutBtn" />
		</c:otherwise>
	</c:choose>
	<div class="write_wrap">
		<form action="#" id="actionForm" method="post">
			<!-- 기본값 : hidden -->
			<input type="hidden" id="sg" name="searchGbn" value="${param.searchGbn}" />
			<input type="hidden" id="searchOldTxt" value="${param.searchTxt}" />
			<input type="hidden" id="st" name="searchTxt" value="${param.searchTxt}" />
			<input type="hidden" id="page" name="page" value="${page}" />
			
			<!-- 글작성, 편집영역 -->
			<c:choose>
				<c:when test="${empty sMNo}">
					<input type="button" value="로그인" id="loginBtn" />
				</c:when>
				<c:otherwise>
					<input type="hidden" name="mNo" value="${sMNo}" />
					<input type="hidden" name="obNo" id="obNo" />
					${sMNm} <textarea rows="3" cols="50" name="obCon" id="obCon"></textarea>
					<input type="button" value="작성" id="writeBtn" />
					<input type="button" value="수정" id="updateBtn" />
					<input type="button" value="취소" id="cancelBtn" />
				</c:otherwise>
			</c:choose>
		</form>
	</div>
	<div class="list_wrap">
		<table>
			<colgroup>
				<col width="100px" />
				<col width="400px" />
				<col width="300px" />
			</colgroup>
			<thead>
				<tr>
					<th>작성자</th>
					<th>내용</th>
					<th></th>
				</tr>
			</thead>
			<tbody></tbody>
		</table>
	</div>
	<div class="select_wrap">
		<select id="searchGbn">
				<option value="0">작성자</option>
				<option value="1">내용</option>
		</select>
		<input type="text" id="searchTxt" value="${param.searchTxt}" />
		<input type="button" value="검색" id="searchBtn" /><br/>
	</div>
	<div class="paging_wrap"></div>
</body>
</html>
```
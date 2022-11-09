---
title: spring 예제(9) jquery ajax 로그인, 로그아웃, 목록, 페이징
date: 2021-06-1
tags: spring, jquery, ajax
---



![Request](/assets/images/Request.JPG){: width="1000" height="500"}

![Ajax Spring Workflow](/assets/images/Ajax Spring Workflow.JPG){: width="1000" height="500"}

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
		
		// ObjectMapper : 객체를 문자열로 변환 - JackSon라이브러리
		ObjectMapper mapper = new ObjectMapper();
		// 데이터 보관용 map
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		HashMap<String, String> data = iTestLService.getM(params);
		
		if(data != null) { // 사용자 정보가 있음
			session.setAttribute("sMNo", data.get("M_NO"));
			session.setAttribute("sMNm", data.get("M_NM"));
			modelMap.put("resMsg", "success");
		} else { // 사용자 정보가 없음
			modelMap.put("resMsg", "failed");
		}
		
		// writeValueAsString : 객체를 문자열로 변환
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
	
}

```

-----

##### testALogin.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.06.11</title>
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
				// form의 data를 문자열로 전환
				var params = $("#loginForm").serialize();
				
				// ajax
				$.ajax({
					url: "testALogins", // 접속 주소
					type: "post", // 전송 방식: get, post
					dataType: "json", // 받아올 데이터 형태
					data: params, // 보낼 데이터(문자열 형태)
					success: function(res) { // 성공 시 다음 함수 실행
						if(res.resMsg == "success") {
							location.href = "testABList";
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
	<form action="#" id="loginForm">
	 	아이디 <input type="text" id="mId" name="mId" /><br/>
	 	비밀번호 <input type="password" id="mPw" name="mPw" /><br/>
	 	<input type="button" value="로그인" id="loginBtn" />
	</form>
</body>
</html>
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
			reloadList();
		});
		
		$(".paging_wrap").on("click", "div",  function() {
			$("#page").val($(this).attr("page"));
			reloadList();
		});
	});
	
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
			html += "<td>" + d.B_TITLE + "</td>";
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
			<input type="hidden" id="page" name="page" value="${page}" />
			<select id="searchGbn" name="searchGbn">
				<option value="0">제목</option>
				<option value="1">작성자</option>
			</select>
			<input type="text" name="searchTxt" value="${param.searchTxt}" />
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
---
title: spring 예제(10) jquery ajax 한줄게시판(1)
date: 2021-06-17
tags: spring jquery ajax
---




##### TestAOBContoller.java

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

@Controller
public class TestAOBController {

	@Autowired
	public ITestLService iTestLService;
	
	@Autowired
	public IPagingService iPagingService;

	@RequestMapping(value = "testAOB")
	public ModelAndView testAOB(ModelAndView mav) {
		mav.setViewName("testa/testAOB");
		
		return mav;
	}
	
	@RequestMapping(value = "/testAOBLoginAjax", 
			method = RequestMethod.POST, 
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testAOBLoginAjax(HttpSession session,
		@RequestParam HashMap<String, String> params) throws Throwable {
		ObjectMapper mapper = new ObjectMapper();
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		HashMap<String, String> data = iTestLService.getM(params);
		
		if(data != null) { // 사용자 정보가 있음
			session.setAttribute("sMNo", data.get("M_NO"));
			session.setAttribute("sMNm", data.get("M_NM"));
			modelMap.put("resMsg", "success");
			modelMap.put("mNo", data.get("M_NO"));
			modelMap.put("mNm", data.get("M_NM"));
		} else { // 사용자 정보가 없음
			modelMap.put("resMsg", "failed");
		}
		
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testAOBLogoutAjax", 
			method = RequestMethod.POST, 
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testAOBLogoutAjax(HttpSession session) throws Throwable {
		ObjectMapper mapper = new ObjectMapper();
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		session.invalidate();
		
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testAOBListAjax", 
			method = RequestMethod.POST, 
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testAOBListAjax(
			@RequestParam HashMap<String, String> params) throws Throwable {
		ObjectMapper mapper = new ObjectMapper();
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		// 페이지 취득
		int page = Integer.parseInt(params.get("page"));
		
		// 총 글 개수
		int cnt = iTestLService.getObCnt(params);
		
		// 페이징 계산
		PagingBean pb = iPagingService.getPagingBean(page, cnt);
		
		params.put("startCnt", Integer.toString(pb.getStartCount()));
		params.put("endCnt", Integer.toString(pb.getEndCount()));
		
		// 목록 취득
		List<HashMap<String, String>> list
			= iTestLService.getObList(params);
		
		modelMap.put("list", list);
		modelMap.put("pb", pb);
		
		return mapper.writeValueAsString(modelMap);
	}
	
	@RequestMapping(value = "/testAOBWriteAjax", 
			method = RequestMethod.POST, 
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testAOBWriteAjax(
			@RequestParam HashMap<String, String> params) throws Throwable {
		ObjectMapper mapper = new ObjectMapper();
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		try {
			int cnt = iTestLService.writeOb(params);
			
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
	
	@RequestMapping(value = "/testAOBUpdateAjax", 
			method = RequestMethod.POST, 
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testAOBUpdateAjax(
			@RequestParam HashMap<String, String> params) throws Throwable {
		ObjectMapper mapper = new ObjectMapper();
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		try {
			int cnt = iTestLService.updateOb(params);
			
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
	
	@RequestMapping(value = "/testAOBDeleteAjax", 
			method = RequestMethod.POST, 
			produces = "text/json;charset=UTF-8")
	@ResponseBody
	public String testAOBDeleteAjax(
			@RequestParam HashMap<String, String> params) throws Throwable {
		ObjectMapper mapper = new ObjectMapper();
		Map<String, Object> modelMap = new HashMap<String, Object>();
		
		try {
			int cnt = iTestLService.deleteOb(params);
			
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

##### testAOB.java

```java
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %> 
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<style type="text/css">
#actionForm #updateBtn, #actionForm #cancelBtn {
	display: none;
} 

#actionForm div {
	display: none;
}

#loginArea {
	text-align: right;
}

#loginArea div {
	display: none;
}

tr [type=button] {
	visibility: hidden;
}

tr:hover [type=button] {
	visibility: visible;
}
</style>
<script type="text/javascript"
		src="resources/script/jquery/jquery-1.12.4.min.js"></script>
<script type="text/javascript">
$(document).ready(function() {
	if("${sMNo}" != "") { // 로그인 상태
		$("#infoWrap, #writeInfo").show();
	} else { // 비 로그인 상태
		$("#loginWrap, #loginInfo").show();
	}
	
	reloadList();
	
	$("#writeLoginBtn").on("click", function() {
		$("#mId").focus();
	});
	
	$("#loginBtn").on("click", function() {
		if($.trim($("#mId").val()) == "") {
			alert("아이디를 입력해 주세요.");
			$("#mId").focus();
		} else if($.trim($("#mPw").val()) == "") {
			alert("비밀번호를 입력해 주세요.");
			$("#mPw").focus();
		} else {
			var params = $("#loginForm").serialize();
			
			$.ajax({
				url: "testAOBLoginAjax",
				type: "post",
				dataType: "json",
				data: params,
				success: function(res) {
					if(res.resMsg == "success") {
						$("#mNo").val(res.mNo);
						$("#infoMsg").html(res.mNm + "님 어서오세요.");
						$("#writer").html(res.mNm);
						$("#mId").val("");
						$("#mPw").val("");
						
						$("#infoWrap, #writeInfo").show();
						$("#loginWrap, #loginInfo").hide();
						
						reloadList();
					} else {
						alert("아이디 또는 비밀번호가 일치하지 않습니다.");
					}
				},
				error: function(request, status, error) {
					console.log(error);
				}
			});
		}
	});
	
	$("#logoutBtn").on("click", function() {
		$.ajax({
			url: "testAOBLogoutAjax",
			type: "post",
			dataType: "json",
			success: function(res) {
					$("#mNo").val("");
					$("#infoMsg").html("");
					$("#writer").html("");
					
					$("#infoWrap, #writeInfo").hide();
					$("#loginWrap, #loginInfo").show();
					
					reloadList();
			},
			error: function(request, status, error) {
				console.log(error);
			}
		});
	});
	
	//작성버튼
	$("#writeBtn").on("click", function() {
		if($.trim($("#obCon").val()) == "") {
			alert("내용을 넣어주세요.");
			$("#obCon").focus();
		} else {
			var params = $("#actionForm").serialize();
			
			$.ajax({
				url : "testAOBWriteAjax",
				type : "post",
				dataType : "json",
				data : params,
				success : function(res) {
					if(res.msg == "success") {
						$("#obCon").val("");
						resetVal();
						reloadList();
					} else if(res.msg == "failed") {
						alert("작성에 실패하였습니다.");
					} else {
						alert("작성중 문제가 발생하였습니다.");
					}
				},
				error : function(request, status, error) {
					console.log(error);
				}
			});
		}
	});
	
	//검색버튼
	$("#searchBtn").on("click", function() {
		$("#page").val(1);
		$("#sg").val($("#searchGbn").val());
		$("#st").val($("#searchTxt").val());
		reloadList();
	});
	
	// 페이징 이벤트
	$("#pagingWrap").on("click", "span", function() {
		$("#page").val($(this).attr("name"));
		reloadList();
	});
	
	// 목록의 수정버튼 클릭
	$("table").on("click", "#updateBtn", function() {
		$("#actionForm #writeBtn").hide();
		$("#actionForm #updateBtn, #actionForm #cancelBtn").show();
		$("#obNo").val($(this).parent().parent().attr("name"));
		$("#obCon").val($(this).parent().parent().children(":nth-child(2)").html());
	});
	
	// 취소버튼
	$("#cancelBtn").on("click", function() {
		$("#obCon").val("");
		$("#obNo").val("");
		
		$("#actionForm #writeBtn").show();
		$("#actionForm #updateBtn, #actionForm #cancelBtn").hide();
	});
	
	$("#actionForm #updateBtn").on("click", function() {
		if($.trim($("#obCon").val()) == "") {
			alert("내용을 넣어주세요.");
			$("#obCon").focus();
		} else {
			var params = $("#actionForm").serialize();
			
			$.ajax({
				url : "testAOBUpdateAjax",
				type : "post",
				dataType : "json",
				data : params,
				success : function(res) {
					if(res.msg == "success") {
						$("#cancelBtn").click();
						
						reloadList();
					} else if(res.msg == "failed") {
						alert("작성에 실패하였습니다.");
					} else {
						alert("작성중 문제가 발생하였습니다.");
					}
				},
				error : function(request, status, error) {
					console.log(error);
				}
			});
		}
	});
	
	// 목록의 삭제버튼 클릭
	$("table").on("click", "#deleteBtn", function() {
		if(confirm("삭제하시겠습니까?")) {
			$("#obNo").val($(this).parent().parent().attr("name"));
			
var params = $("#actionForm").serialize();
			
			$.ajax({
				url : "testAOBDeleteAjax",
				type : "post",
				dataType : "json",
				data : params,
				success : function(res) {
					if(res.msg == "success") {
						$("#obNo").val("");
						resetVal();
						reloadList();
					} else if(res.msg == "failed") {
						alert("작성에 실패하였습니다.");
					} else {
						alert("작성중 문제가 발생하였습니다.");
					}
				},
				error : function(request, status, error) {
					console.log(error);
				}
			});
		}
	});
});

function resetVal() {
	$("#page").val(1);
	$("#sg").val("0");
	$("#st").val("");
	$("#searchGbn").val("0");
	$("#searchTxt").val("");
}

function reloadList() {
	var params = $("#actionForm").serialize();
	
	$.ajax({
		url : "testAOBListAjax",
		type : "post",
		dataType : "json",
		data : params,
		success : function(res) {
			redrawList(res.list);
			redrawPaging(res.pb);
		},
		error : function(request, status, error) {
			console.log(error);
		}
	});
}

function redrawList(list) {
	var html = "";
	
	for(var d of list) {
		html += "<tr name=\"" + d.OB_NO + "\">";
		html += "<td>" + d.M_NM + "</td>";
		html += "<td>" + d.OB_CON + "</td>";
		html += "<td>";
		if($("#mNo").val() == d.M_NO) {
			html += "<input type=\"button\" value=\"수정\" id=\"updateBtn\" />";
			html += "<input type=\"button\" value=\"삭제\" id=\"deleteBtn\" />";
		}
		html += "</td>";
		html += "</tr>";
	}
	
	$("tbody").html(html);
}

function redrawPaging(pb) {
	var html = "";
	
	html += "<span name=\"1\">처음</span>";
	
	if($("#page").val() == "1") {
		html += "<span name=\"1\">이전</span>";
	} else {
		html += "<span name=\"" + ($("#page").val() - 1) + "\">이전</span>";
	}
	
	for(var i = pb.startPcount ; i <= pb.endPcount ; i++) {
		if($("#page").val() == i) {
			html += "<span name=\"" + i + "\"><b>" + i + "</b></span>";
		} else {
			html += "<span name=\"" + i + "\">" + i + "</span>";
		}
	}
	
	if($("#page").val() == pb.maxPcount) {
		html += "<span name=\"" + pb.maxPcount + "\">다음</span>";
	} else {
		html += "<span name=\"" + ($("#page").val() * 1 + 1) + "\">다음</span>";
	}
	
	html += "<span name=\"" + pb.maxPcount + "\">마지막</span>";
	
	$("#pagingWrap").html(html);
}
</script>
</head>
<body>
<div id="loginArea">
	<div id="loginWrap">
		<form action="#" id="loginForm">
			아이디<input type="text" id="mId" name="mId" />
			비밀번호<input type="password" id="mPw" name="mPw" />
			<input type="button" value="로그인" id="loginBtn" />
		</form>
	</div>
	<div id="infoWrap">
		<span id="infoMsg">${sMNm}님 어서오세요.</span><input type="button" value="로그아웃" id="logoutBtn" />
	</div>
</div>
<div>
	<div class="write_area">
		<form action="#" id="actionForm" method="post">
			<!-- 기본값 : hidden -->
			<input type="hidden" id="sg" name="searchGbn" />
			<input type="hidden" id="st" name="searchTxt" />
			<input type="hidden" id="page" name="page" value="1" />
			
			<!-- 글작성,편집영역 -->
			<div id="loginInfo">
				로그인이 필요한 서비스 입니다.
				<input type="button" value="로그인" id="writeLoginBtn" />
			</div>
			<div id="writeInfo">
				<input type="hidden" name="mNo" id="mNo" value="${sMNo}" />
				<input type="hidden" name="obNo" id="obNo" />
				<span id="writer">${sMNm}</span><textarea rows="3" cols="50" name="obCon" id="obCon"></textarea>
				<input type="button" value="작성" id="writeBtn" />
				<input type="button" value="수정" id="updateBtn" />
				<input type="button" value="취소" id="cancelBtn" />
			</div>
		</form>
	</div>
	<div class="list_area">
		<table>
			<colgroup>
				<col width="100" />
				<col width="500" />
				<col width="100" />
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
	<div class="paging_area">
		<!-- 검색 -->
		<select id="searchGbn">
			<option value="0">작성자</option>
			<option value="1">내용</option>
		</select>
		<input type="text" id="searchTxt" />
		<input type="button" value="검색" id="searchBtn" /><br/>
		<!-- 페이징 -->
		<div id="pagingWrap"></div>
	</div>
</div>
</body>
</html>
```
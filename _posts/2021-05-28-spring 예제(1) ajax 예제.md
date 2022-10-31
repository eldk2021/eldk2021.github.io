---
title: spring 예제(1) ajax 예제
date: 2021-05-28
tags: spring
---


-----
##### TestController.java

```java
package com.test.spring.test.controller;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;

import com.test.spring.test.service.ITestService;;

@Controller
public class TestController {
	//@Autowired 아래 지정된 변수에 대상 객체를 주입한다.
	@Autowired
	public ITestService iTestService;

	/*
	 * @RequestMapping - 지정된 주소가 요청되면 아래 메소드를 실행한다.
	 */
	@RequestMapping(value="/test1")
	
	/*
	 * ModelAndView : 데이터와 뷰를 담을 수 있는 클래스
	 */
	public ModelAndView test1(ModelAndView mav) {
		
		// ViewResolver : "/WEB-INF/views/" + "test/test1" + ".jsp" => "/WEB-INF/views/test/test1.jsp"
		mav.setViewName("test/test1"); // jsp 위치 지정
		
		return mav;
	}
	
	@RequestMapping(value="/test2")
	// HttpServletRequest : 요청과 관련된 데이터 집합(사용자에게 넘어온것)
	// @RequestParam(value=값) 변수타입 변수명 : 값에 해당하는 Key가 넘어올 때 Key에 연결된 값을 변수에 담는다.
	// @RequestParam 변수타입 변수명 : 변수명과 동일한 Key가 넘어올 때 key에 연결된 값을 변수에 담는다.
	// @RequestParam HashMap<String, String> 변수명 : 넘어오는 key와 value들을 map에 담는다.
	// @RequestParam(value=값) List 변수명 : 값에 해당하는 Key들이 넘어올 때 Key에 연결된 값들을 리스트에 담는다.(거의 checkbox쓸 때만 사용)
	// HttpServletResponse : 응답에 대한 정보(보낼 형태, 헤더정보 등)
	public ModelAndView test2(HttpServletRequest req,
							  @RequestParam(value="txt") String s,
							  @RequestParam String txt,
							  @RequestParam HashMap<String, String> params,
							  @RequestParam(value="txt") ArrayList<String> list,
													ModelAndView mav) {
		
		System.out.println(req.getParameter("txt"));
		System.out.println(s);
		System.out.println(txt);
		System.out.println(params.get("txt"));
		System.out.println(list.get(0));
		
		// req.setAttribute("test", "Hi~");
		
		mav.addObject("test", "Hi2!!"); // Model에 값을 담는다.(자바쪽에서 강제로 만든 데이터를 JSP에 넘겨줌)
		
		List<HashMap<String, String>> data = new ArrayList<HashMap<String, String>>();
		
		for(int i = 10 ; i > 0 ; i--) {
			HashMap<String, String> temp = new HashMap<String, String>();
			
			temp.put("no", Integer.toString(i));
			temp.put("title", "test" + i);
			
			data.add(temp);
		}
		mav.addObject("data", data);
		
		mav.setViewName("test/test2");
		
		return mav;
	}
	
	@RequestMapping(value="/ajaxTest")
	public ModelAndView at(ModelAndView mav) {
		
		mav.setViewName("test/ajaxTest");
		return mav;
	}
	
	@RequestMapping(value="/practice")
	public ModelAndView pt(ModelAndView mav) {
		
		mav.setViewName("test/practice");
		return mav;
	}
	
	@RequestMapping(value="/practice_1")
	public ModelAndView pt2(ModelAndView mav) {
		
		mav.setViewName("test/practice_1");
		return mav;
	}
	
	@RequestMapping(value="/test3")
	public ModelAndView test3(ModelAndView mav) {
		
		iTestService.test();
		
		mav.setViewName("test/test3");
		return mav;
	}
}
```

-----

##### practice.json(내가 한 방법)

```json
{
	"list1" : [
		{
		 	"OrderDate" : "OrderDate",
		 	"Region" : "Region",
		 	"Rep" : "Rep",
		 	"Item" : "Item",
		 	"Units" : "Units",
		 	"UnitCos" : "UnitCos",
		 	"Total" : "Total"
		 }
	],
	"list2" : [
		 {
		 	"OrderDate" : "2020-01-06",
		 	"Region" : "East",
		 	"Rep" : "Jones",
		 	"Item" : "Pencil",
		 	"Units" : 95,
		 	"UnitCos" : 1.99,
		 	"Total" : 189.05
		 },
		 {
		 	"OrderDate" : "2020-01-23",
		 	"Region" : "Central",
		 	"Rep" : "Kicell",
		 	"Item" : "Binder",
		 	"Units" : 50,
		 	"UnitCos" : 19.99,
		 	"Total" : 999.5
		 },
		 {
		 	"OrderDate" : "2020-02-09",
		 	"Region" : "Central",
		 	"Rep" : "Jardine",
		 	"Item" : "Pencil",
		 	"Units" : 36,
		 	"UnitCos" : 4.99,
		 	"Total" : 179.64
		 },
		 {
		 	"OrderDate" : "2020-02-26",
		 	"Region" : "Central",
		 	"Rep" : "Gill",
		 	"Item" : "Pen",
		 	"Units" : 27,
		 	"UnitCos" : 19.99,
		 	"Total" : 539.73
		 },
		 {
		 	"OrderDate" : "2020-03-15",
		 	"Region" : "West",
		 	"Rep" : "Sorvino",
		 	"Item" : "Pencil",
		 	"Units" : 56,
		 	"UnitCos" : 2.99,
		 	"Total" : 167.44
		 },
		 {
		 	"OrderDate" : "2020-04-01",
		 	"Region" : "East",
		 	"Rep" : "Jones",
		 	"Item" : "Binder",
		 	"Units" : 60,
		 	"UnitCos" : 4.99,
		 	"Total" : 299.4
		 },
		 {
		 	"OrderDate" : "2020-04-18",
		 	"Region" : "Central",
		 	"Rep" : "Andrews",
		 	"Item" : "Pencil",
		 	"Units" : 75,
		 	"UnitCos" : 1.99,
		 	"Total" : 149.25
		 },
		 {
		 	"OrderDate" : "2020-05-05",
		 	"Region" : "Central",
		 	"Rep" : "Jardine",
		 	"Item" : "Pencil",
		 	"Units" : 90,
		 	"UnitCos" : 4.99,
		 	"Total" : 449.1
		 },
		 {
		 	"OrderDate" : "2020-05-22",
		 	"Region" : "West",
		 	"Rep" : "Thompson",
		 	"Item" : "Pencil",
		 	"Units" : 32,
		 	"UnitCos" : 1.99,
		 	"Total" : 63.68
		 },
		 {
		 	"OrderDate" : "2020-06-08",
		 	"Region" : "East",
		 	"Rep" : "Jones",
		 	"Item" : "Binder",
		 	"Units" : 60,
		 	"UnitCos" : 8.99,
		 	"Total" : 539.4
		 },
		 {
		 	"OrderDate" : "2020-06-25",
		 	"Region" : "Central",
		 	"Rep" : "Morgan",
		 	"Item" : "Pencil",
		 	"Units" : 90,
		 	"UnitCos" : 4.99,
		 	"Total" : 449.1
		 }
	]

}
```

-----

##### practice.jsp(내가 한 방법)

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.05.28</title>
<style type="text/css">
	table {
	 	border: 1px solid #444444;
     	border-collapse: collapse;
     	text-align: center;
	}
	
	thead {
		background-color: #b380ff;
		font-weight: bold;
	}
	
	th, td {
    	border: 1px solid #444444;
    	height: 30px;
  	}

</style>
<script type="text/javascript" src="resources/jquery/jquery-1.12.4.js"></script>
<script type="text/javascript">
	$(document).ready(function () {
		list();
		
		function list() {
			$.ajax({
				url: "resources/json/practice.json", // 접속 주소
				type: "get", // 전송 방식: get, post
				dataType: "json", // 받아올 데이터 형태
				success: function(res) { // 성공 시 다음 함수 실행
					console.log(res);
					var html = "";
					var html2 = "";
					for(var p of res.list1) {
						html += "<tr>";
						html += "<td>" + p.OrderDate + "</td>";
						html += "<td>" + p.Region + "</td>";
						html += "<td>" + p.Rep + "</td>";
						html += "<td>" + p.Item + "</td>";
						html += "<td>" + p.Units + "</td>";
						html += "<td>" + p.UnitCos + "</td>";
						html += "<td>" + p.Total + "</td>";
						html += "</tr>";
					}
					for(var pp of res.list2) {
						html2 += "<tr>";
						html2 += "<td>" + pp.OrderDate + "</td>";
						html2 += "<td>" + pp.Region + "</td>";
						html2 += "<td>" + pp.Rep + "</td>";
						html2 += "<td>" + pp.Item + "</td>";
						html2 += "<td>" + pp.Units + "</td>";
						html2 += "<td>" + pp.UnitCos + "</td>";
						html2 += "<td>" + pp.Total + "</td>";
						html2 += "</tr>";
					}
					
					$("#list thead").html(html);
					$("#list tbody").html(html2);
				},
				error: function(request, status, error) { // 실패 시 다음 함수 실행
					console.log(request);
					console.log(status);
					console.log(error);
				}
			});
		}
	});
</script>

</head>
<body>
	<table id="list">
		<colgroup>
					<col width="100px" />
					<col width="70px" />
					<col width="90px" />
					<col width="60px" />
					<col width="50px" />
					<col width="70px" />
					<col width="70px" />
		</colgroup>
		<thead></thead>
		<tbody></tbody>
	</table>
</body>
</html>
```

-----

-----

##### practice_1.json(쌤이 한 방법)

```json
{
	"orderCol":["OrderDate", "Region", "Rep", 
				"Item", "Units", "UnitCos", "Total"],
	"order":[{
	"OrderDate":"2020-01-06",
	"Region":"East",
	"Rep":"Jones",
	"Item":"Pencil",
	"Units":"95",
	"UnitCos":"1.99",
	"Total":"189.05"
	},
	{
	"OrderDate":"2020-01-23",
	"Region":"Central",
	"Rep":"Kivell",
	"Item":"Binder",
	"Units":"50",
	"UnitCos":"19.99",
	"Total":"999.5"
	},
	{
	"OrderDate":"2020-02-09",
	"Region":"Central",
	"Rep":"Jardine",
	"Item":"Pencil",
	"Units":"36",
	"UnitCos":"4.99",
	"Total":"179.64"
	},
	{
	"OrderDate":"2020-02-26",
	"Region":"Central",
	"Rep":"Gill",
	"Item":"Pen",
	"Units":"27",
	"UnitCos":"19.99",
	"Total":"539.73"
	},
	{
	"OrderDate":"2020-03-15",
	"Region":"West",
	"Rep":"Sorvino",
	"Item":"Pencil",
	"Units":"56",
	"UnitCos":"2.99",
	"Total":"167.44"
	},
	{
	"OrderDate":"2020-04-01",
	"Region":"East",
	"Rep":"Jones",
	"Item":"Binder",
	"Units":"60",
	"UnitCos":"4.99",
	"Total":"299.4"
	},
	{
	"OrderDate":"2020-04-18",
	"Region":"Central",
	"Rep":"Andrews",
	"Item":"Pencil",
	"Units":"75",
	"UnitCos":"1.99",
	"Total":"149.25"
	},
	{
	"OrderDate":"2020-05-05",
	"Region":"Central",
	"Rep":"Jardine",
	"Item":"Pencil",
	"Units":"90",
	"UnitCos":"4.99",
	"Total":"449.1"
	},
	{
	"OrderDate":"2020-05-22",
	"Region":"West",
	"Rep":"Thompson",
	"Item":"Pencil",
	"Units":"32",
	"UnitCos":"1.99",
	"Total":"63.68"
	},
	{
	"OrderDate":"2020-06-08",
	"Region":"East",
	"Rep":"Jones",
	"Item":"Binder",
	"Units":"60",
	"UnitCos":"8.99",
	"Total":"539.4"
	},
	{
	"OrderDate":"2020-06-25",
	"Region":"Central",
	"Rep":"Morgan",
	"Item":"Pencil",
	"Units":"90",
	"UnitCos":"4.99",
	"Total":"449.1"
	}
	]
}
```

-----

##### practice_1.jsp(쌤이 한 방법)

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<style type="text/css">
#orderTable {
	border-collapse: collapse;
}

#orderTable td, #orderTable th {
	padding: 5px;
	border: 1px solid #444444;
	text-align: center;
}

#orderTable th {
	background-color: #A099FF;
}
</style>
<script type="text/javascript"
		src="resources/jquery/jquery-1.12.4.js"></script>
<script type="text/javascript">
$(document).ready(function() {
	$.ajax({
		url: "resources/json/practice_1.json",
		type: "get",
		dataType: "json",
		success: function(res) {
			var html = "";
			// thead
			html += "<tr>";
			for(var c of res.orderCol) {
				html += "<th>" + c + "</th>";
			}
			html += "</tr>";
			
			$("#orderTable thead").html(html);
			
			html = "";
			
			// tbody
			for(var o of res.order) {
				html += "<tr>";
				for(var c of res.orderCol) {
					html += "<td>" + o[c] + "</td>";
				}
				html += "</tr>";
			}
			
			$("#orderTable tbody").html(html);
		},
		error: function(request, status, error) { // 실패 시 다음 함수 실행
			console.log(request);
			console.log(status);
			console.log(error);
		}
	});
});
</script>
</head>
<body>
<table id="orderTable">
	<thead></thead>
	<tbody></tbody>
</table>
</body>
</html>
```
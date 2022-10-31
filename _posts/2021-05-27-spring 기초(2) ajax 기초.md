---
title: spring 기초(2) jquery ajax 기초
date: 2021-05-27
tags: spring
---



-----
```markdown
Ajax - 비동기 방식의 <u>전송</u> 방법(기술이 아닌 방식)
	   주는 것 = 데이터, 받는 것 = 결과데이터

동기화(Synchronize) - 주소와 화면이 일치(앞에서 처리)
비동기화(Asynchronize) - 주소와 화면이 별개로 인식(뒤에서 처리)

Ajax에서의 데이터
- 문자열
- json - javascript Object를 문자열화
- xml - 정보를 tag 형태로 관리
```

-----

-----

##### TestController.java

```java
package com.test.spring.test.controller;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;

import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.servlet.ModelAndView;;

@Controller
public class TestController {

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
}

```

-----

##### test.json

```json
j{
	"list" : [
		 {
		 	"no" : "1",
		 	"name" : "홍길동"
		 },
		 {
		 	"no" : "2",
		 	"name" : "김철수"
		 },
		 {
		 	"no" : "3",
		 	"name" : "박대리"
		 }
	]

}
```

-----

##### ajaxTest.jsp

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8"
    pageEncoding="UTF-8"%>
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.05.27</title>
<style type="text/css">
	#weatherIcon {
	display: none;
	}

</style>
<script type="text/javascript" src="resources/jquery/jquery-1.12.4.js"></script>
<script type="text/javascript">
$(document).ready(function () {
	$("#weatherIcon").hide();
	
	getWeather();
	getWeatherHistorical();
	
	$("#getBtn").on("click", function () {
		$.ajax({
			url: "resources/json/test.json", // 접속 주소
			type: "get", // 전송 방식: get, post
			dataType: "json", // 받아올 데이터 형태
			success: function(res) { // 성공 시 다음 함수 실행
				var html = "";
			for(var d of res.list) {
				html += d.no + "," + d.name + "<br/>"
				
			}
			
			$("#box").append(html);
			},
			error: function(request, status, error) { // 실패 시 다음 함수 실행
				console.log(request);
				console.log(status);
				console.log(error);
			}
		});
	});
});

	function getWeather() {
		$.ajax({
			url: "http://api.openweathermap.org/data/2.5/weather", // 접속 주소
			type: "get", // 전송 방식: get, post
			dataType: "json", // 받아올 데이터 형태
			data: "q=seoul&appid=8ab7c9b5db69d415e162335a4d306548&lang=kr&units=metric", // 보낼 데이터(문자열 형태)
			success: function(res) { // 성공 시 다음 함수 실행
				console.log(res);
				console.log(res.main.temp);
				console.log(res.weather[0].icon);
				$("#weatherIcon").attr("src", "http://openweathermap.org/img/wn/" + res.weather[0].icon + "@2x.png");
				$("#temp").html(res.main.temp + "℃ - " + res.weather[0].description);
				$("#weatherIcon").show();
			},
			error: function(request, status, error) { // 실패 시 다음 함수 실행
				console.log(request);
				console.log(status);
				console.log(error);
			}
		});
	}
	
	function getWeatherHistorical() {
		$.ajax({
			url: "https://api.openweathermap.org/data/2.5/onecall/timemachine", // 접속 주소
			type: "get", // 전송방식 : get, post
			dataType: "json", // 받아올 데이터 형식
			data: "lat=37.5683&lon=126.9778&dt=1622095350&appid=8ab7c9b5db69d415e162335a4d306548&lang=kr&units=metric&lang=kr&units=metric", //보낼 데이터(문자열형태)
			success: function(res) { // 성공 시 다음 함수 실행
				console.log(res);
			var html = "";
			for(var i = res.hourly.length - 1 ; i > res.hourly.length -6 ; i--) {
				html += "<tr>";
				html += "<td>" + new Date(res.hourly[i].dt * 1000) + "</td>";
				html += "<td><img alt=\"날씨\" src=\"http://openweathermap.org/img/wn/" + res.hourly[i].weather[0].icon + "@2x.png\" /></td>";
				html += "<td>" + res.hourly[i].temp + "℃</td>";
				html += "</tr>";
			}
			$("#weatherHistory tbody").html(html);
			},
			error: function(request, status, error) { // 실패 시 다음 함수 실행
				console.log(request);
				console.log(status);
				console.log(error);
			}
		});
	}
</script>
</head>
<body>
	<div id="weatherWrap">
		<img alt="날씨" id="weatherIcon" /><span id="temp"></span>
	</div>
	<table id="weatherHistory">
	<tbody></tbody>
</table>
	<input type="button" value="가져오기" id="getBtn"/>
	<div id="box"></div>
</body>
</html>
```

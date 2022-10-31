---
title: javascript 기초(1)
date: 2021-04-29
tags: js
---

-----

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>Insert title here</title>
<script type="text/javascript">
	var a = 10;
	console.log(a);

	if (a > 0) {
		console.log("a는 0보다 크다.");
	} else {
		console.log("a는 0보다 작거나 같다.");
	}

	/* ------------------------------------*/

	/* 선언부. 건드리지 말것 */
	a = 5;
	var b = 7;
	var c = 3;

	/* 오름차순 정렬 구현 */
	if (a > b) {
		var temp = b;
		a = b;
		b = temp;
	}
	if (a > c) {
		var temp = a;
		a = c;
		c = temp;
	}
	if (b > c) {
		var temp = b;
		b = c;
		c = temp;
	}

	/* 출력부. 건드리지 말것 */
	console.log(a);
	console.log(b);
	console.log(c);

	/* ------------------------------------*/

	for (var i = 1; i < 10; i++) {
		console.log("2 * " + i + " = " + (2 * i)); // 2 * 1 = 2
	}

	/* ------------------------------------*/

	/* 선언부. 건드리지 말것 */
	var arr = [ 5, 7, 3 ];

	/* 내림차순 정렬 구현 */
	for (var i = 0; i < arr.length - 1; i++) {
		for (var j = i + 1; j < arr.length; j++) {
			if (arr[i] < arr[j]) {
				var temp = arr[i];
				arr[i] = arr[j];
				arr[j] = temp;
			}
		}
	}

	/* 출력부. 건드리지 말것 */
	for (var i = 0; i < arr.length; i++) {
		console.log(arr[i])
	}

	/* ------------------------------------*/

	/* 함수(function) - method와 동일, 기능, 동작
	   function 함수명(인자명, ...) {
				내용
				return 값;(반환값 존재 시 ))
	}
	 */
	function test() {
		alert("Hi~!");
	}

	var w;
	function test2() {
		/* location : 주소창
		   href : 주소 */
		// location.href = "http://google.co.kr"; // 현재 창에 띄울 경우
		w = window.open("http://google.co.kr", "_blank"); // 새 창에 띄울 경우
	}

	function test3() {
		w.close(); // 권장하지 않음
	}

	var s = "3.1415";
	console.log(s * 1 + 1); // 문자열(숫자로만 구성)을 숫자로 변환

	var s = "3.1415a";
	console.log(s * 1 + 1); // NaN
	console.log(isNaN(s * 1)); // isNaN(값) : 값이 NaN인지 확인

	function getTxt1() {
		// document : 해당 html
		// getElementById : id를 찾아 화면객체를 취득
		// 화면객체.value : 화면객체의 속성 중 value의 값을 가져옴
		var t = document.getElementById("txt1");
		alert(t.value);
		t.value = "가져갔다";
	}
	
	function cc() {
		document.getElementById("box").className = "b"; 
	}
	
	function cc2(obj) {
		obj.className = "a";
	}
</script>
<style type="text/css">
.a, .b {
	display: inline-block;
	vertical-align: top;
	width: 100px;
	height: 100px;
}

.a {
	background-color: red;
}

.b {
	background-color: blue;
}
</style>
</head>
<body>
	<input type="button" value="버튼" onclick="test();" />
	<input type="button" value="이동버튼" onclick="test2();" />
	<input type="button" value="새탭닫기버튼" onclick="test3();" />
	<br />
	<input type="text" id="txt1" />
	<input type="button" value="취득" onclick="getTxt1();" />
	<br />

	<div class="a" id="box" onclick="cc2(this);"></div>
	<br />
	<input type="button" value="바꿔라" onclick="cc();" />
</body>
</html>
``` 
---
title: javascript 기초(2)
date: 2021-05-03
tags: js
---

-----

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.05.03</title>
</head>
<style type="text/css">
#box, #box2 {
	display: inline-block;
	vertical-align: top;
	width: 200px;
	height: 200px;
	border: 1px solid #000000;
	background-color: rgb(0, 0, 0);
	width: 200px
}
</style>
<script type="text/javascript">
	var r = 0;
	var g = 0;
	var b = 0;

	function boxColor(btn) {
		var box = document.getElementById("box");
		switch (btn.value) {
		case "rUp":
			if (r < 255) {
				r += 5;
			}
			break;

		case "rDown":
			if (r > 0) {
				r -= 5;
			}
			break;

		case "gUp":
			if (g < 255) {
				g += 5;
			}
			break;

		case "gDown":
			if (g > 0) {
				g -= 5;
			}
			break;

		case "bUp":
			if (b < 255) {
				b += 5;
			}
			break;

		case "bDown":
			if (b > 0) {
				b -= 5;
			}
			break;
		}
		box.style.backgroundColor = "rgb(" + r + "," + g + "," + b + ")";
	}

	function tog() {
		var box = document.getElementById("box");

		console.log(box.style.display);

		if (box.style.display == "none") { // display : none(영역이 없음) => 영역제거
			box.style.display = ""; // 값을 안 줄 경우 해당 style 속성이 제거됨
		} else {
			box.style.display = "none";

		}
	}

	function tog2() {
		var box = document.getElementById("box");

		console.log(box.style.display);

		if (box.style.visibility == "hidden") { // visibility : hidden(화면이 보이지 않음. 영역은 있음) => 감추기
			box.style.visibility = ""; // 값을 안 줄 경우 해당 style 속성이 제거됨
		} else {
			box.style.visibility = "hidden";
		}
	}

	function bigger() {
		var box2 = document.getElementById("box2");

		console.log(box2.clientWidth);

		box2.style.width = box2.clientWidth * 2 + "px"; // css에 적용된 너비
		box2.style.height = box2.clientHeight * 2 + "px"; // css에 적용된 높이
	}

	function smaller() {
		var box2 = document.getElementById("box2");

		console.log(box2.clientWidth);

		box2.style.width = box2.clientWidth / 2 + "px";
		box2.style.height = box2.clientHeight / 2 + "px";
	}
	
</script>

<body>
	<div id="box"></div>
	<div id="box2"></div>
	<br />
	<input type="button" value="rUp" onclick="boxColor(this);" />
	<input type="button" value="rDown" onclick="boxColor(this);" />
	<input type="button" value="gUp" onclick="boxColor(this);" />
	<input type="button" value="gDown" onclick="boxColor(this);" />
	<input type="button" value="bUp" onclick="boxColor(this);" />
	<input type="button" value="bDown" onclick="boxColor(this);" />
	<br />
	<input type="button" value="버튼" onclick="tog();" />
	<input type="button" value="버튼2" onclick="tog2();" />
	<input type="button" value="커져라!" onclick="bigger();" />
	<input type="button" value="작아져라!" onclick="smaller();" />
</body>
</html>
```

-----

-----

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.05.03</title>
<style type="text/css">
#box {
	display: inline-block;
	width: 100px;
	height: 100px;
	position: absolute;
	top: 0px;
	left: 0px;
	background-color: skyblue;
}
</style>
<script type="text/javascript">
	var t = 0;
	var l = 0;
	function m(e) {
		/* console.log(e);
		console.log(e.keyCode); */

		// w - 119, s - 115, a - 97, d - 100
		// keyCode : 13 => 엔터키
		var box = document.getElementById("box");

		switch (e.keyCode) {
		case 119: // w 위
			if (t - 10 >= 0) {
				t -= 10;
			}
			break;
		case 115: // s 아래
			t += 10;
			break;
		case 97: // a 왼쪽			
			if (l - 10 >= 0) {
				l -= 10;
			}
			break;
		case 100: // d 오른쪽
			l += 10;
			break;
		}

		box.style.top = t + "px";
		box.style.left = l + "px";
	}
</script>
<!-- 이벤트 호출 시 event 객체를 받아올 수 있음 -->
<body onkeypress="m(event);">
	<div id="box"></div>
</body>
</html>
```

-----

-----

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.05.03</title>
<script type="text/javascript">
	console.log(Math.round(3.141592));
	console.log(Math.floor(3.141592));
	console.log(Math.ceil(3.141592));
	console.log(Math.abs(-123));
	console.log(Math.random());

	var str = "Hello World!!\"Hi\"";
	console.log(str);

	str = "Hello World!!";
	console.log(str.length);
	console.log(str.indexOf("l", 3));
	console.log(str.indexOf("x")); // 찾는 문자가 없을 땐 -1
	console.log(str.substring(7, 10)); //7번째부터 9번째까지의 문자
	console.log(str.replace("l", "k")); // 첫번째 l만 k로 바뀜
	/* 정규식 사용시
	  /대상 문자/옵션
	      의 형태를 사용
	      옵션 - i : 대소문자 구분없음
	       - g : 모든 대상 */
	console.log(str.replace(/orl/ig, "k")); // orl을 k로 바꿈
	console.log(str.toUpperCase());
	console.log(str.toLowerCase());

	str = "      Hello World!!       ";
	console.log("[" + str.trim() + "]"); // 맨앞, 맨뒤 공백 제거
	
	str = "Hello World!!";
	console.log(str.charAt(0)); // 0번째 자리 문자
	console.log(str[0]); // 0번째 자리 문자
	
	var arr = "apple, banana, mango".split(",");
	console.log(arr);
	
	var d = new Date();
	console.log(d);
	console.log(d.toString());
	console.log(d.getFullYear()); // 연도
	console.log(d.getMonth() + 1); // 인덱스 기반 월
	console.log(d.getDate()); // 일
	console.log(d.getHours()); // 시
	console.log(d.getMinutes()); // 분
	console.log(d.getSeconds()); // 초
	console.log(d.getMilliseconds()); // 밀리초
	console.log(d.getDay()); // 주의 몇번째 날인지
	
</script>
</head>
<body>
</body>
</html>
```
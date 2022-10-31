---
title: js 예제(1) 계산기
date: 2021-05-04
tags: js
---

-----

#### Javascript Object

```
<생성>

변수명 = {

	키 : 값,

	...

};

<취득>

변수명.키

<할당>

변수명.키 = 값;
```

-----

-----

```html
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8">
<title>2021.05.04</title>
<script type="text/javascript">
	var arr = [ "Apple", "Banana", "Mango" ];

	console.log(arr.toString());
	console.log(arr.join("    하       하하       "));

	var a = arr.pop(); // 배열의 마지막 것 잘라내기
	console.log(arr.toString());
	console.log(a);

	arr.push("orange"); // 배열의 마지막에 추가
	console.log(arr.toString());

	arr.shift(); // 배열의 첫번째 것 잘라내기
	console.log(arr.toString());

	arr.unshift("peach"); // 배열의 첫번째에 추가
	console.log(arr.toString());

	arr[arr.length] = "apple";
	console.log(arr.toString());

	/* 배열.splice(숫자1, 숫자2, 값1, ...) - 배열의 숫자 1번째부터 숫자 2의 개수만큼 제거
									   - 그 이후 숫자 1번째에 지정된 값들을 넣는다.(선택적)*/
	arr.splice(1, 1, "용과");
	console.log(arr.toString());

	arr.splice(1, 1);
	console.log(arr.toString());

	arr.splice(1, 0, "pear");
	console.log(arr.toString());

	arr.sort(); // 오름차순 정렬
	console.log(arr.toString());

	arr.reverse(); // 내림차순 정렬
	console.log(arr.toString());

	var user = {
		name : "홍길동",
		age : 300,
		addr : "한양"
	};

	console.log(user);
    user.name = "김철수";
	console.log(user.name);
	console.log(user.age);
	console.log(user.addr);

	window.onload = function() { // 화면 로드가 끝난 시점에서 해당 함수를 실행한다.
		// window.onload는 여러개 지정시 마지막 것만 동작
		console.log(document.getElementById("txt").value);
		test(function() {
			alert("aaa");
		});
	};

	function test(func) {
		func.call(); // 변수에 담긴 함수를 실행
	}
</script>
</head>
<body>
	<input type="text" id="txt" value="입력창이다" />
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
<title>2021.05.04</title>
<script type="text/javascript">
	var no = 1;

	function b() {
		var wrap = document.getElementById("wrap");

		no++;

		var html = "<div id=\"d" + no + "\" onclick=\"d(this);\">" + no
				+ "</div>";

		// innerHTML : Entity
		wrap.innerHTML += html;
		console.log(wrap.innerHTML);
	}

	function d(obj) {
		var a = document.getElementById(obj.id); // 명시적 객체 취득
		a.remove(); // 객체 삭제
	}
</script>
</head>
<body>
	<input type="button" value="뒤" onclick="b();" />
	<br />
	<div id="wrap">
		<div id="d1" onclick="d(this);">1</div>
	</div>
</body>
</html>
```
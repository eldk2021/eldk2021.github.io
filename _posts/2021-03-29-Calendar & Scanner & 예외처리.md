---

title: Calendar & Scanner & 예외처리
date: 2021-03-29
tags: java
---


## Scanner

```java
package com.gd.test.controller;

import java.text.SimpleDateFormat;
import java.util.Calendar;
import java.util.Date;
import java.util.Scanner;

public class TestController3 {

	public static void main(String[] args) {

		// Calendar는 시스템에서 객체를 취득한다.
		Calendar c = Calendar.getInstance();
		// 29-MAR-2021
		System.out.println(c.get(Calendar.YEAR)); // 연
		System.out.println(c.get(Calendar.MONTH) + 1); // 월은 인덱스 기반이기 때문에 +1해야 함
		System.out.println(c.get(Calendar.DATE)); // 일
		System.out.println(c.get(Calendar.AM_PM)); // 0 - AM , 1 - PM
		System.out.println(c.get(Calendar.HOUR)); // 12시간 기준
		System.out.println(c.get(Calendar.HOUR_OF_DAY)); // 24시간 기준
		System.out.println(c.get(Calendar.MINUTE)); // 분
		System.out.println(c.get(Calendar.SECOND)); // 초
		System.out.println(c.get(Calendar.MILLISECOND)); // 밀리초
		System.out.println(c.get(Calendar.DAY_OF_WEEK)); // 요일. 1 - 일요일

		// y : 연
		// M : 월
		// d : 일
		// a : am / pm
		// H : 시(24시간 기준)
		// h : 시(12시간 기준)
		// m : 분
		// s : 초
		String p = "yyyy-MM-dd HH:mm:ss";

		SimpleDateFormat sdf = new SimpleDateFormat(p);
		// 현재 날짜를 가져와서 해당 포맷으로 변환하여 문자열로 돌려줌.
		String d = sdf.format(new Date());

		System.out.println(d);

		// ---------------------------//

		Scanner sc = new Scanner(System.in);

		String s = sc.nextLine(); // 한줄을 받을 때 사용(엔터가 나올 때까지)
		System.out.println(s);

		int a = sc.nextInt(); // 숫자만 입력받음(공백있을 시 공백 전까지의 값만 취득)
		System.out.println(a);

		sc.nextLine();

		s = sc.nextLine();
		System.out.println(s);
	}
}
```

-----

#### 실행결과

```java
2021
3
29
1
9
21
50
53
157
2
2021-03-29 21:50:53
```

-----

-----

```java
package com.gd.test.controller;

import java.util.Scanner;

public class TestController4 {

	public static void main(String[] args) {
		
		Scanner sc = new Scanner(System.in);
		
		while(true) {
			System.out.println("메뉴를 선택하시오.");
			System.out.println("1.계속\t9.종료"); // \t : 탭 
			int input = sc.nextInt();
			sc.nextLine();
			
			if(input == 9) {
				break;
			}
		}
		
		System.out.println("종료되었습니다.");
	}

}
```

-----

#### 실행결과

```java
메뉴를 선택하시오.
1.계속	9.종료
9
종료되었습니다.
```

-----

-----

## 예외처리

```java
package com.gd.test.controller;

import com.gd.test.service.TestService5;

public class TestController5 {

	public static void main(String[] args) {
		try {
			String s = "abc";
			// int a = Integer.parseInt(s);
			throw new NumberFormatException();
			// System.out.println("?????"); // dead code
		} catch (NumberFormatException ne) {
			ne.printStackTrace();
		} catch (Exception e) {
			System.out.println(e.toString());
			e.printStackTrace();
		}
		System.out.println("끝난거 맞음");

		TestService5 ts = new TestService5();

		try {
			ts.test();
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
}
```

-----

```java
package com.gd.test.service;

public class TestService5 {
	// throws : 발생할 수도 있는 예외들을 미리 정의
	//			위험성이 있다.
	public void test() throws Exception {
		System.out.println("메소드");
	}
}
```

-----

#### 실행결과

```java
java.lang.NumberFormatException
	at com.gd.test.controller.TestController5.main(TestController5.java:11)
끝난거 맞음
메소드
```


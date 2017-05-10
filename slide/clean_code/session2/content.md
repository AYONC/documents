#  Chapter 3. 함수

-----

## Intro

어떤 프로그램이든 기본적인 단위가 함수다.

-----

```java
public static String testableHtml(PageData pageData, boolean includeSuiteSetup) {
  WikiPage wikiPage = pageData.getWikiPage();
  StringBuffer buffer = new StringBuffer();
  if (pageData.hasAttribute("Test")) {
    if(includeSuiteSetup) {
      WikiPage suiteSetup = 
        PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_SETUP_NAME, wikiPage);
      if(suiteSetup != null) {
        WikiPagePath = pagePath = suiteSetup.getPageCrawler().getFullPath(suiteSetup);
        String pagePathName = PathParser.render(pagePath);
        buffer.append("!include -setup .")
          .append(pagePathName)
          .append("\n");
      }
    }
    WikiPage setup = PageCrawlerImpl.getInheritedPage("SetUp", wikiPage);
    if(setup != null) {
      WikiPagePath setupPath = wikiPage.getPageCrawler().getFullPath(setup);
      String setupPathName = PathParser.render(setupPath);
      buffer.append("!include -setup .")
        .append(setupPathName)
        .append("\n");
    }
  }
  buffer.append(pageData.getContent());
  if(pageData.hasAttribute("Test")) {
    WikiPage teardown = PageCrawlerImpl.getInheritedPage("TearDown", wikiPage);
    if(teardown != null) {
      WikiPagePath tearDownPath = wikiPage.getPageCrawler().getFullPath(teardown);
      String tearDownPathName = PathParser.render(tearDownPath);
      buffer.append("\n")
        .append("!include -teardown .")
        .append(tearDownPathName)
        .append("\n");
    }
    if(includeSuiteSetup) {
      WikiPage suiteTeardown =
			PageCrawlerImpl.getInheritedPage(SuiteResponder.SUITE_TEARDOWN_NAME, wikiPage);
      if(suiteTeardown != null) {
        WikiPagePath pagePath = suiteTeardown.getPateCrawler().getFullPath(suiteTeardown);
        String pagePathName = PathParser.render(pagePath);
        buffer.append("!include -teardown .")
          .append(pagePathName)
          .append("\n");
      }
    }
  }
  pageData.setContent(buffer.toString());
  return pageData.getHtml();
}
```



-----



**길이가 길고,** <!-- .element: class="fragment" -->

**중복된 코드에,** <!-- .element: class="fragment" -->

**괴상한 문자열에,** <!-- .element: class="fragment" -->

**낯설고 모호한 자료 유형과 API,** <!-- .element: class="fragment" -->

**다양한 추상화 수준,** <!-- .element: class="fragment" -->

-----

<!-- .slide: data-background="#e7ad52" -->

<h5>그렇다면 읽시 쉽고 이해하기 쉬운 함수는 어떻게 작성해야하는가?</h5>

-----

## 1. 작게 만들어라!

-----

#### 함수를 만들 때 최대한 ‘작게!’ 만들어라.

**가로 140자 이내,** <!-- .element: class="fragment" -->

**100줄을 넘어가지 않도록.** <!-- .element: class="fragment" -->

-----

```java
public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception {
	boolean isTestPage = pageData.hasAttribute("Test"); 
	if (isTestPage) {
		WikiPage testPage = pageData.getWikiPage(); 
		StringBuffer newPageContent = new StringBuffer(); 
		includeSetupPages(testPage, newPageContent, isSuite); 
		newPageContent.append(pageData.getContent()); 
		includeTeardownPages(testPage, newPageContent, isSuite); 
		pageData.setContent(newPageContent.toString());
	}
	return pageData.getHtml(); 
}
```

-----

 위 코드도 길다.

되도록 한 함수당 3~5줄 이내로 줄이는 것을 권장한다 <!-- .element: style="color: #ffffff" -->

-----

```java
public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite) throws Exception { 
	if (isTestPage(pageData)) 
		includeSetupAndTeardownPages(pageData, isSuite); 
	return pageData.getHtml();
}
```

-----

#### 블록과 들여쓰기  

* 중첩구조(if/else, while문 등)에 들어가는 블록은 한 줄이어야 한다.

* 각 함수 별 들여쓰기 수준이 2단을 넘어서지 않고,

* 각 함수가 명백하다면 함수는 더욱 읽고 이해하기 쉬워진다.

-----


## 한 가지만 해라  

-----

**함수는 한가지를 해야 한다. **

**그 한가지를 잘 해야 한다. **

**그 한가지만을 해야 한다.**

-----

<h4>한가지를 한다는 것?</h4>

지정된 함수 이름 아래에서 <!-- .element: style="color: #ffffff" -->

<span class="yellow">추상화 수준이 하나인 단계만 수행</span>한다면 <!-- .element: style="color: #ffffff" -->

그 함수는 한 가지 작업만 하는 것이다. <!-- .element: style="color: #ffffff" -->

-----

### 함수 내 섹션

함수 내부를 여러 **섹션**으로 나눌 수 있다면

그 함수는 여러작업을 하는 셈이다.

-----

## 함수 당 추상화 수준은 하나로

함수가 ‘한가지’ 작업만 하려면

함수 내 모든 문장의 **추상화 수준이 동일**해야된다.

 <!-- .element: class="fragment" -->
```
pageData.getHtml()
pathParser.render(pagePath)
```

-----

### 위에서 아래로 코드 읽기:**내려가기** 규칙

코드는 위에서 아래로 이야기처럼 읽혀야 좋다.

함수 추상화 부분이

<span class="yellow">한번에 한단계씩 낮아지는 것</span>이 가장 이상적이다.(내려가기 규칙)

-----

## Switch문

-----

Switch문은 작게 만들기 어렵다.

Switch문은 N가지를 처리한다.

-----

```java
public Money calculatePay(Employee e) throws InvalidEmployeeType {
	switch (e.type) {
		case COMMISSIONED:
			return calculateCommissionedPay(e);
		case HOURLY:
			return calculateHourlyPay(e);
		case SALARIED:
			return calculateSalariedPay(e);
		default:
			throw new InvalidEmployeeType(e.type);
	}
}
```

-----

다형성을 이용하여 switch문을 **abstract factory**에 숨겨

다형적 객체를 생성하는 코드 안에서만 switch를 사용하도록 한다.

-----

```java
public abstract class Employee {
	public abstract boolean isPayday();
	public abstract Money calculatePay();
	public abstract void deliverPay(Money pay);
}
-----------------
public interface EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType;
}
-----------------
public class EmployeeFactoryImpl implements EmployeeFactory {
	public Employee makeEmployee(EmployeeRecord r) throws InvalidEmployeeType {
		switch (r.type) {
			case COMMISSIONED:
				return new CommissionedEmployee(r) ;
			case HOURLY:
				return new HourlyEmployee(r);
			case SALARIED:
				return new SalariedEmploye(r);
			default:
				throw new InvalidEmployeeType(r.type);
		}
	}
}
```

-----

**하지만 switch문은 불가피하게 써야될 상황이 많으므로, 상황에 따라서는 사용 할 수도 있다.**

-----

## 서술적인 이름을 사용하라!
> “코드를 읽으면서 짐작했던 기능을 각 루틴이 그대로 수행한다면 깨끗한 코드라 불러도 되겠다” - 워드 커닝햄

-----

작은 함수는 그 **기능이 명확**

    -> 이름을 붙이기가 더 쉬우며,

일관성 있는 서술형 이름을 사용한다면

    -> 코드를 순차적으로 이해하기도 쉬워진다.

```java
includeSetupAndTeardownPages
includeSetupPages
includeSuiteSetupPage
includeSetupPage
```

-----

<h5> 서술적인 이름을 사용하면 설계가 뚜렸해지므로 코드를 개선하기 쉬워진다.</h5>

-----

## 함수 인수

인수는 코드 이해에 방해가 되는 요소

최선의 인수 개수는 **0개**(무항)

차선은 **1개**

출력인수(함수의 반환 값이 아닌 입력 인수로 결과를 받는 경우)는 이해하기 어려우므로

<span class="yellow">왠만하면 쓰지 않는 것이 좋다</span>

-----

### 많이 쓰는 단항 형식
  * 인수에 질문을 던지는 경우
	`boolean fileExists(“MyFile”);`
  * 인수를 뭔가로 변환해 결과를 변환하는 경우
	`InputStream fileOpen(“MyFile”);`
  * 이벤트 함수일 경우 (이 경우에는 이벤트라는 사실이 코드에 명확하게 드러나야 한다.)

-----

### 플래그 인수

플래그 인수는 **추하다**.

<span class="yellow">쓰지마라.</span>

```
`render(true)`
`render(boolean inSuite)`
```

bool 값을 넘기는 것 자체가

그 함수는 한꺼번에 여러가지 일을 처리한다고 공표하는 것

-----

#### 이항 함수

* 단항 함수보다 이해하기가 어렵다.

* Point 클래스의 경우에는 이항 함수가 적절하다.

* 2개의 인수간의 자연적인 순서가 있어야함

```
`Point p = new Point(x,y);`
```

무조건 나쁜 것은 아니지만,

**이해가 어렵고 위험이 따르므로** 가능하면 단항으로 바꾸자

-----

#### 삼항 함수
```
`assertEquals(message, expected, actual)`
```

* 이해하기가 훨씬 어려우므로, 위험도 2배 이상 늘어난다.

* 삼항 함수를 만들 때는 신중히 고려하라.

-----

#### 인수 객체

<div>인수가 많이 필요할 경우, </div>

<div>일부 인수를 독자적인 클래스 변수로 선언할 가능성을 살펴보자 </div>

<div4>x,y를 인자로 넘기는 것보다 Point를 넘기는 것이 더 낫다.</div>

```java
Circle makeCircle(double x, double y, double radius)
Circle makeCircle(Point center, double radius)
```

-----

#### 인수 목록

때로는 String.format같은 함수들처럼

**인수 개수가 가변적인 함수도 필요**하다.

```
`public String format(String format, Object... args)`
```

String.format의 인수는 List형 인수이기 때문에 이항함수

-----

#### 동사와 키워드

* 단항 함수는 함수와 인수가 동사/명사 쌍을 이뤄야한다.
```
`writeField(name);`
```
함수이름에 키워드(인수 이름)을 추가하면 인수 순서를 기억할 필요가 없어진다.
```
`assertExpectedEqualsActual(expected, actual);`
```

-----

## 부수 효과를 일으키지 마라!

-----

<h4>부수효과는 코드에서 거짓말을 하는 것</h4>

<h4>함수에서 한가지를 하겠다고 약속하고는 남몰래 다른 짓을 하는 것</h4>

**한 함수에서는 딱 한가지만 수행할 것!**

-----

```java
public class UserValidator {
	private Cryptographer cryptographer;
	public boolean checkPassword(String userName, String password) {
		User user = UserGateway.findByName(userName);
		if (user != User.NULL) {
			String codedPhrase = user.getPhraseEncodedByPassword();
			String phrase = cryptographer.decrypt(codedPhrase, password);
			if ("Valid Password".equals(phrase)) {
				Session.initialize();
				return true;
			}
		}
		return false;
	}
}
```

-----

#### 출력인수

* 일반적으로 출력 인수는 피해야 한다.

* 함수에서 상태를 변경해야 한다면 함수가 속한 객체 상태를 변경하는 방식을 택하라.


-----

## 명령과 조회를 분리하라

함수는

<div class="fragment"> <span class="yellow">객체 상태를 변경</span>하거나, </div>

<div class="fragment"><span class="yellow">객체 정보를 반환</span>하거나</div>

**둘다 수행해서는 안된다.**  <!-- .element: class="fragment" -->

-----

```
`if(set(“username”, “unclebob”))...`

`public boolean set(String attribute, String value);` <!-- .element: class="fragment" -->
```

<div>혼란스럽다... </div><!-- .element: class="fragment" -->

-----

## 오류코드보다 예외를 사용하라!


try/catch를 사용하면

오류 처리 코드가 원래 코드에서 분리되므로

코드가 깔끔해 진다.

-----

#### Try/Catch 블록 뽑아내기

```java
if (deletePage(page) == E_OK) {
	if (registry.deleteReference(page.name) == E_OK) {
		if (configKeys.deleteKey(page.name.makeKey()) == E_OK) {
			logger.log("page deleted");
		} else {
			logger.log("configKey not deleted");
		}
	} else {
		logger.log("deleteReference from registry failed");
	}
} else {
	logger.log("delete failed"); return E_ERROR;
}
```

-----

정상 작동과 오류 처리 동작을 뒤섞는 추한 구조

    -> if/else와 마찬가지로 블록을 별도 함수로 뽑아내는 편이 좋다.

-----

```java
public void delete(Page page) {
	try {
		deletePageAndAllReferences(page);
  	} catch (Exception e) {
  		logError(e);
  	}
}

private void deletePageAndAllReferences(Page page) throws Exception {
	deletePage(page);
	registry.deleteReference(page.name);
	configKeys.deleteKey(page.name.makeKey());
}

private void logError(Exception e) {
	logger.log(e.getMessage());
}
```

-----

```java
public enum Error {
	OK,
	INVALID,
	NO_SUCH,
	LOCKED,
	OUT_OF_RESOURCES,
	WAITING_FOR_EVENT;
}
```

Error.java 의존성 자석이 생긴다.  <!-- .element: class="fragment" -->

-----

<h4>오류를 처리하는 곳곳에서 오류코드를 사용</h4>

 -> 추가하거나 변경할 때 비용이 많이 필요

**그러므로 예외를 사용하는 것이 더 안전하다.**

-----

## 반복하지 마라! DRY<small>Don't repeat yourself</small>

<div>중복은 모든 소프트웨어에서 **모든 악의 근원••이므로 늘 중복을 없애도록 노력해야한다.</div>

-----

<h4> 중복제거 전략 </h4>

* 관계형 데이터 베이스 정규화

* 객체 지향 프로그래밍 상속 개념

* 구조적프로그래밍

* AOP<small>Aspect Oriented Programming</small>

* COP<small>component oriented Programming</small>

-----

## 구조적 프로그래밍

에르허츠 데이크스트라<small>다익스크라x/데이크스트라o</small>

구조적 프로그래밍의 원칙을 따르자면

**모든 함수와 함수 내 모든 블록에 입구와 출구가 하나**

*루프 안에서 break나 continue를 사용해선 안되며 goto는 절대로, 절대로 사용하지 말자.*

-----

<div> 함수가 클 경우에만 상당 이익을 제공</div>
    -> 함수가 작다면 위 규칙은 별 이익을 제공하지 못한다.

    -> 그러므로 함수를 작게 만든다면 간혹 return, break, continue를 사용해도 괜찮다.


**오히려 때로는 단일 입/출구 규칙보다 의도를 표현하기 쉬워진다.**

-----

## 함수를 어떻게 짜죠?

**글짓기와 비슷하다.**

처음에는 길고 복잡하고, 들여쓰기 단계나 중복된 루프도 많다.

인수목록도 길지만,

-----

단위 테스트 케이스도 만들고, <!-- .element: class="fragment" -->

코드를 다듬고, <!-- .element: class="fragment" -->

함수를 만들고, <!-- .element: class="fragment" -->

이름을 바꾸고, <!-- .element: class="fragment" -->

중복을 제거하고 <!-- .element: class="fragment" -->

메서드를 줄이고 순서를 바꾸고 <!-- .element: class="fragment" -->

클래스를 쪼개기도한다. <!-- .element: class="fragment" -->

처음부터 완벽한 형태일 수 없다 <!-- .element: class="fragment" -->

-----

## 결론

**시스템은 구현할 프로그램이 아닌, 풀어갈 이야기**

프로그래밍 언어라는 표현 수단을 통해 <span class="yellow">소통 언어(함수)</span>를 만들고

함수를 잘 만드는 <span class="white">기교를 통해 자신만의 이야기를 풀어나가자.</span>


-----

# Chapter 4. 주석

-----

> 나쁜 코드에 주석을 달지 마라. 새로 짜라.
> - 브라이언 W.커니핸, P.J.플라우거

-----

<h3>주석은 필요악</h3>

코드로 의도를 표현하지 못해, 실패를 만회하기 위해 쓰는 것

주석은 언제나 실패를 의미

주석 없이는 자신을 표현할 방법을 찾지 못해 *할 수 없이 주석을 사용*

-----
### 결국...

> 개발자는 코드로 말한다

-----

## 주석을 무시하는 이유가 무엇이냐고?

주석이 오래될수록 코드에서 멀어져서

    > 거짓말을 하게 될 가능성이 커지기 때문

    > 주석을 계속 유지보수하기란 현실적으로 불가능

-----

## 주석은 나쁜 코드를 보완하지 못한다

코드에 주석을 추가하는 일반적인 이유

    <spen class="yellow">코드 품질이 나빠서</span>

<div>깔끔하고 주석이 거의 없는 코드가,</div>
<div>복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다.</div>

**주석으로 설명하려 애쓰는 대신에 그 난장판을 깨끗이 치우는 데 시간을 보내라!** <!-- .element: class="fragment" -->

-----

## 코드를 의도로 표현하라!

```java
// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다. 
if ((emplotee.flags & HOURLY_FLAG) && (employee.age > 65)
```
-----

```java
if (employee.isEligibleForFullBenefits())
```

<div>주석도 필요없이 함수 이름만으로 충분히 깔끔하게 표현가능</div>

-----

## 좋은 주석

-----

#### 법적인 주석

<div>각 소스 파일 첫머리에 들어가는 저작권 정보와 소유권 정보 등</div>

```
  `// Copyright (C) 2003, 2004, 2005 by Object Montor, Inc. All right reserved.`

  `// GNU General Public License`
```

-----

#### 정보를 제공하는 주석 

```java
// 테스트 중인 Responder 인스턴스를 반환
protected abstract Responder responderInstance();
```

-----

**물론 이 주석도 함수 이름에 정보를 담아 responderBeingTested로 바꾸면 없앨 수 있다. **

  더 나은 예:
```java
// kk:mm:ss EEE, MMM dd, yyyy 형식이다.
Pattern timeMatcher = Pattern.compile("\\d*:\\d*\\d* \\w*, \\w*, \\d*, \\d*");
```

-----

#### 의도를 설명하는 주석

```java
// 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다. 
for (int i = 0; i > 2500; i++) {
    WidgetBuilderThread widgetBuilderThread = 
        new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
    Thread thread = new Thread(widgetBuilderThread);
    thread.start();
}
```

<div>함수의 이름으로 표현하는 것보다 분명한 경우 주석</div>

-----

#### 결과를 경고하는 주석    

```java
// 여유 시간이 충분하지 않다면 실행하지 마십시오.
public void _testWithReallyBigFile() {

}
```

-----


#### TODO 주석    

```java
// TODO-MdM 현재 필요하지 않다.
// 체크아웃 모델을 도입하면 함수가 필요 없다.
protected VersionInfo makeVersion() throws Exception {
    return null;
}
```

-----

<div> TODO 주석은 프로그래머가 필요하다 여기지만 당장 구현하기 어려운 업무를 기술한다.</div>

* 더 이상 필요 없는 기능을 삭제하라는 알림,

* 누군가에게 문제를 봐달라는 요청,

* 더 좋은 이름을 떠올려달라는 부탁,

* 앞으로 발생할 이벤트에 맞춰 코드를 고치라는 주의 등에 유용

*요즘은 IDE가 남은 TODO를 쉽게 볼 수 있으므로 편리하게 이용할 수 있음*

-----


#### 중요성을 강조하는 주석    

```java
String listItemContent = match.group(3).trim();
// 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
// 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다. 
new ListItemWidget(this, listItemContent, this.level + 1);
return buildList(text.substring(match.end()));
```

-----


#### 공개 API에서 Javadocs    

설명이 잘 된 공개 API는 참으로 유용하고 만족스럽다.

공개 API를 구현한다면 반드시 훌륭한 Javadocs 작성을 추천한다.

*하지만 여느 주석과 마찬가지로 Javadocs 역시 독자를 오도하거나,

잘못 위치하거나, 그릇된 정보를 전달할 가능성이 존재하는 것 역시 잊으면 안 된다.*

-----


## 나쁜 주석

-----

<div>대다수의 주석이 이 범주에 속한다.</div>

일반적으로 대다수 주석은 허술한 코드를 지탱하거나, <!-- .element: class="fragment" -->

엉성한 코드를 변명하거나, <!-- .element: class="fragment" -->

미숙한 결정을 합리화 <!-- .element: class="fragment" -->

**프로그래머가 주절거리는 독백에서 크게 벗어나지 못한다.** <!-- .element: class="fragment" -->


-----

#### 주절거리는 주석    
특별한 이유 없이 달리는 주석이다. 

```java
public void loadProperties() {
    try {
        String propertiesPath = propertiesLocation + "/" + PROPERTIES_FILE;
        FileInputStream propertiesStream = new FileInputStream(propertiesPath);
        loadedProperties.load(propertiesStream);
    } catch (IOException e) {
        // 속성 파일이 없다면 기본값을 모두 메모리로 읽어 들였다는 의미다. 
    }
}
```

-----

catch 블록에 있는 주석은 저자에게야 의미가 있겠지만 다른 사람들에게는 전해지지 않는다.

저 주석의 의미를 알아내려면 다른 코드를 뒤져보는 수밖에 없다.

**이해가 안되어 다른 모듈까지 뒤져야 하는 주석은 제대로 된 주석이 아니다.** <!-- .element: class="fragment" -->

-----


#### 같은 이야기를 중복하는 주석

코드 내용을 그대로 중복하는 주석이 있다.

전혀 필요없는 코드

```java
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예외를 던진다. 
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if (!closed) {
        wait(timeoutMillis);
        if (!closed) {
            throw new Exception("MockResponseSender could not be closed");
        }
    }
}
```

-----


#### 오해할 여지가 있는 주석


```java
// this.closed가 true일 때 반환되는 유틸리티 메서드다.
// 타임아웃에 도달하면 예외를 던진다.
public synchronized void waitForClose(final long timeoutMillis) throws Exception {
    if (!closed) {
        wait(timeoutMillis);
        if (!closed) {
            throw new Exception("MockResponseSender could not be closed");
        }
    }
}
```

<div>this.closed가 true로 변하는 순간에 메서드는 반환되지 않는다.</div>

<small>무조건 타임아웃을 기다렸다 this.closed가 그래도 true가 아니면 예외를 던진다.</small>

-----

주석에 담긴 '살짝 잘못된 정보'로 인해 어느 프로그래머가

경솔하게 함수를 호출에 자기 코드가 아주 느려진 이유를 못찾게 되는 것이다.

-----


#### 의무적으로 다는 주석    

모든 함수에 Javadocs를 달거나 모든 변수에 주석

이런 주석은 코드를 복잡하게 만들며,

거짓말을 퍼뜨리고,

혼동과 무질서를 초래한다.

------
#### 가치 없는 주석
```java
/**
 *
 * @param title CD 제목
 * @param author CD 저자
 * @param tracks CD 트랙 숫자
 * @param durationInMinutes CD 길이(단위: 분)
 */
public void addCD(String title, String author, int tracks, int durationInMinutes) {
    CD cd = new CD();
    cd.title = title;
    cd.author = author;
    cd.tracks = tracks;
    cd.duration = durationInMinutes;
    cdList.add(cd);
}
```

-----

#### 이력을 기록하는 주석    
*지금은 소스 코드 관리 시스템이 있으니 전혀 필요없다.*

````java
* 변경 이력 (11-Oct-2001부터)
* ------------------------------------------------
* 11-Oct-2001 : 클래스를 다시 정리하고 새로운 패키징
* 05-Nov-2001: getDescription() 메소드 추가
* 이하 생략
````

-----


#### 있으나 마나 한 주석

```java
/*
 * 기본 생성자
 */
protected AnnualDateRule() {

}
```

-----


#### 함수나 변수로 표현할 수 있다면 주석을 달지 마라    

```java
// 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
if (module.getDependSubsystems().contains(subSysMod.getSubSystem()))
```

-----

<div>주석을 제거하고 다시 표현하면 다음과 같다.</div>

```java
ArrayList moduleDependencies = smodule.getDependSubSystems();
String ourSubSystem = subSysMod.getSubSystem();
if (moduleDependees.contains(ourSubSystem))
```

-----


#### 위치를 표시하는 주석    

* 소스 파일에서 특정 위치를 표시하려 주석을 사용

````java
// Actions /////////////////////////////////////////////
````

-----


<div>

이런 주석은 가독성만 낮춘다

너무 자주 사용하지 않을때만 배너는 눈에 띄며 주위를 환기한다.

</div>

<div class="fragment">그러므로 <span class="yellow">반드시 필요할 때 아주 드몰게 사용</span>하는 편이 좋다.<div>

-----

#### 닫는 괄호에 다는 주석    

중첩이 심하고 장황한 함수(중첩된 html tag)

<div>작고 캡슐화면 함수에는 잡음일 뿐이다.</div>

그러므로 닫는 괄호에 주석을 달아야겠다는 생각이 든다면

*대신에 함수를 줄이려 시도하자.*

-----

#### 공로를 돌리거나 저자를 표시하는 주석    

<div>소스 코드 관리 시스템의 Annotate기능으로 대체가능</div>

````java
/* 릭이 추가함 */
````

-----


#### 주석으로 처리한 코드    

```java
this.bytePos = writeBytes(pngIdBytes, 0);
//hdrPos = bytePos;
writeHeader();
writeResolution();
//dataPos = bytePos;
if (writeImageData()) {
    wirteEnd();
    this.pngBytes = resizeByteArray(this.pngBytes, this.maxPos);
} else {
    this.pngBytes = null;
}
return this.pngBytes;
```
소스 코드 관리 시스템을 사용하기 때문에 우리를 대신에 코드를 기억해준다.

*그냥 삭제하라. 잃어버릴 염려는 없다. 약속한다.*

-----

#### 전역 정보    

* 주석을 달아야 한다면 근처에 있는 코드만 기술

* 시스템의 전반적인 정보를 기술하지 마라.

* 해당 시스템의 코드가 변해도 아래 주석이 변하리라는 보장은 전혀 없음

* 심하게 중복되어 있는 주석인 것도 확인

```java
/**
 * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
 *
 * @param fitnessePort
 */
public void setFitnessePort(int fitnessePort) {
    this.fitnewssePort = fitnessePort;
}
```

-----


#### 비공개 코드에서 Javadocs    

공개 API는 Javadocs가 유용하지만 공개하지 않을 코드라면 Javadocs는 쓸모가 없다.

<div>코드만 보기싫고 산만해질 뿐이다.</div>


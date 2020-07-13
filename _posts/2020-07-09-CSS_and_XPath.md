---
layout: post
title: CSS and XPath Syntax 간단 요약 (Cheat Sheet)
#subtitle: Cheat sheet
#cover-img: /assets/img/path.jpg
#image: /assets/img/path.jpg
#share-img: /assets/img/path.jpg
categories: data_engineering
comments: true
---
<style>
	* {
	text-align: justify}
</style>

Web Scraping 을 할 때, `CSS` 와 `Xpath`를 사용하여 HTML이나 XML속에 존재하는 원하는 데이터의 위치를 찾아 추출할 수 있다. 주로 `Selenium`, Python 에서 주로 사용하는 `Scrapy`나 `BeautifulSoup` 그리고 R에서 주로 사용하는 `Rvest` 등등을 이용해서 Web Page에 대한 직접적 parsing을 통한 Web Scraping을 하고자 할 때 필요하다. 이 때, `CSS` 와 `Xpath` syntax가 헷갈리거나 가물가물한 경우가 종종 생길 수 있다. 본 포스팅에서는 참고하기 용이 하게끔 자주 쓰이는 syntax들을 간략하게 정리 해 보았다. **참고로 `CSS`가 `Xpath` 보다 더 간결하고 퍼포먼스가 좋기 때문에 `CSS` 사용이 더 권장된다.**

## Dictionary

> (e.g. \<a id="jung">바보</a>)  
> `element`는 document (Web Scraping 상황에서는 통상적으로 Web Page)에 존재하는 node 일컫는다 (\<a id="jung">바보</a>)  
> `tagname`은 "<" 다음에 나오는 node의 이름(a)을 말한다  
> `attribute`는 "< >" 안에서 tagename이 아닌 요소들을 일컫는다 (id). 여기서 jung을 `attribute's value` 라고 한다  
> `text`는 > 와 < 사이에 있는 것을 일컫는다 (바보)   

*****

## CSS (Cascading Style Sheets)

일반적인 CSS Selector의 syntax는 다음과 같은 모양을 가진다.

```
Syntax : tagname[attribute = 'attribute value']
```

### Cheat sheet

|**Syntax**|	**Example**|	**Description**|
|:--------:|:---------:|:---------------:|
|.class	|.intro |	class="intro" 인 모든 element들 추출|
|#id |	#firstname	| id="firstname" 인 element 추출|
|*	|*	|전체 element들 추출|
|element	|p	|<p> element들 추출|
|element,element	|div, p	|모든 <div> element들과 <p> element들 추출|
|element element	|div p	|모든 <div> element들 안에 있는 모든 <p> element들 추출|
|element>element	|div > p	|부모가 <div> element인 모든 <p> element들 추출 (위의 경우와 동일한 결과)|
|element+element	|div + p	|<div> element들 바로 다음에 위치한 모든 <p> element들 추출|
|element1~element2	|p ~ ul	|<p> element들 바로 앞에 존재하는 모든 <ul> element들 추출|
|[attribute]|	[target]	|target 이라는 attribute을 가진 모든 element들 추출|
|[attribute=value]	|[target=\_blank] |target="\_blank" 인 모든 element들 추출|
|[attribute~=value]	|[title~=flower]	|title attribute의 value가 "flower" 라는 단어를 포함하고 있는 모든 element들 추출|
|[attribute\|=value]	|[lang\|=en]	|lang attribute의 value가 "en" 이라는 단어로 시작하는 모든 element들 추출 (attribute value는 완전한 단어여야 함 e.g. en 혹은 en-US 등등) |
|[attribute^=value]	|a[href^="https"]	|href attribute의 value가 "https" 라는 단어로 시작하는 모든 \<a> element들 추출 (attribute value는 완전한 단어일 필요 없음 e.g. httpsavage)|
|[attribute$=value]	|a[href$=".pdf"]	|href attribute의 value가 ".pdf" 라는 단어로 끝나는 모든 \<a> element들 추출 |
|[attribute*=value]	|a[href*="qatest"]	|href attribute의 value가 "qatest" 라는 단어를 포함하는 모든 \<a> element들 추출 |

### Pseudo-classes

이 syntax들은 다양한 특수 상황을 이용해서 elemet들을 추출하고 싶을때 사용할 수 있다. 양이 너무 많으므로 [`w3schools`](https://www.w3schools.com/css/css_pseudo_classes.asp)를 참고하기 바란다.

### Examples  

1. 하나의 elemet에 다수의 class가 존재할 경우 (아래 예시의 경우 expand 와 icon)

	```
	Html Code : <label class='expand icon'>데이터</label>

	Css selector value : .expand.icon
	Css selector value : [class='expand'] [class='icon']
	```

2. Id로 추출하는 경우

	```
	Html Code : <label id='fourth'>데이터</label>

	Css selector value : #fourth
	```

3. Tagname으로 추출하는 경우

	```
	Html Code : <label id='fourth'>데이터</label>

	Css selector value : label
	```

4. 원하는 element가 유니크 하지 않을 때, 즉 parent가 존재할 때

	```
	Html Code : <div id='abc' class='column'>
					      <label id='fourth' class='expand'>데이터</label>
				      </div>

	Css with class : .column.expand
	Css with id : .abc.fourth
	Css with tagname : div label
	Css with mix 1: div.column#fourth
	Css with mix 1: #abc label.expand
	```

5. 소문자 대문자를 무시하고 싶을 때 (i의 사용)

	```
	Html Code : <div id='abcDE' class='column'>

	Css with class : div[id='abcde' i]
	```

******
  
## Xath

Xpath는 이름 그대로 XML Path를 뜻한다. HTML도 XML의 구조를 가지기 때문에 대부분의 Web Scraping 상황에서 Xpath를 사용하여 원하는 elemet를 추출하는 것이 가능하다. 기본적인 Xpath syntax는 다음과 같다:

```
Syntax : tagname[@attribute = 'attribute value']
```

Xpath에는 두개의 종류가 있다:

1. Absolute Xpath

	슬래쉬 한개는 HTML의 첫번째 node(tag)만을 select한다. 
	```
	Syntax : /tagname[@attribute = 'attribute value']
	```

2. Relative Xpath

	슬래쉬 두개는 web page의 어떤 node(tag)든지 다 select 한다. 우리는 이 Xpath를 살펴보자.
	```
	Syntax : //tagname[@attribute = 'attribute value']
	```

### Cheat sheet

text() function - text를 이용한 추출 (소문자 대문자 조심! + text function은 @ 필요없음)

```
//xpath[text()='text value']
```

contains() fucntion - 어떤 value의 한 부분을 이용한 추출. partial values나 dynamically changing values를 추출할 때 유용

```
//xpath[contains(@attribute, 'attribute value')]
//xpath[contains(text(), 'attribute value')]
```

만약 contains function을 두개 이상 사용하고 싶을 경우 다음과 같이 하면 된다.

```
//xpath[contains(text(), 'text1')][contains(text(), 'text2')]
```
<br><br>

|**Syntax**|**Example**|**Description**|
|:--------:|:---------:|:---------------:|
|\*|\*| wildcard. tagname이나 attribute대신 사용될 수 있다 (어떤 element node든지 매칭시킴 i.e. 모든 것)|
|@\* | @\* | 어떤 attribute node 든지 매칭 |
|node()|node()| 어떤 종류의 어떤 node든지 (element node 든지 attribute node든지) 매칭|
|//\* |//\* | web page상의 모든 element들 추출|
|//tagname/* |//div/* | div tag안의 모든 element들 추출|
| //tagname \| //tagname | //title \| //price | web page상의 모든 title element와 price element 추출 | 
|//tagname[@\*] | //input[@\*] | input tag와 최소 하나의 attribute를 가지는 모든 element들 추출. attribute value는 존재하든 안하든 상관 없음|
|//\*[@\*] | //\*[@\*] | 최소 하나의 attribute를 가지는 모든 element들 추출|

### Examples

1. tagname의 활용

	```
	<html>
		<body>
			<div id="pancakes">
				<button type="button">데이터</button><br><br>
			</div>
		</body>
	</html>

	Syntax for Xpath
	데이터: //button
	```

2. index의 활용

	```
	<html>
		<body>
			<div id="pancakes">
				<button type="button">데이터1</button><br><br>
				<button type="button">데이터2</button><br><br>
				<button type="button">데이터3</button><br><br>
			</div>
		</body>
	</html>

	Syntax for Xpath
	데이터1: //button[1]
	데이터2: //button[2]
	데이터3: //button[3]
	```

3. attribute의 활용

	```
	<html>
		<body>
			<div id="pancakes">
				<button type="button">데이터1</button><br><br>
				<button type="button" name='데이터2'>데이터2</button><br><br>
				<button type="button" name='데이터3' id="1">데이터3</button><br><br>
			</div>
			<div id="pancakes">
				<button type="button">데이터4</button><br><br>
				<button type="button">데이터5</button><br><br>
				<button type="button">데이터6</button><br><br>
			</div>
		</body>
	</html>

	Syntax for Xpath
	데이터2: //button[@name='banana']
	데이터3: //button[@name='banana'][@id='1']
	```

4. 부모-자식 관계의 활용

	```
	<html>
		<body>
			<div id="pancakes">
				<button type="button">데이터1</button><br><br>
				<button type="button">데이터2</button><br><br>
				<button type="button">데이터3</button><br><br>
			</div>
			<div id="fruits">
				<button type="button">데이터4</button><br><br>
				<button type="button">데이터5</button><br><br>
				<button type="button">데이터6</button><br><br>
			</div>
		</body>
	</html>

	Syntax for Xpath
	데이터2: //div[@id='pancakes']/button[2]
	```

5. group index의 활용

	group index는 모든 매칭되는 element들을 리스트에 넣고 각각에게 index를 부여한다. 그러므로 같은 매칭이 서로 구분 될 수 있게끔 한다. 아래의 예시에서는 모든 button을 매칭되는 element로 간주하고 총 여섯개의 button들에 대해 index를 부여한다고 볼 수 있다.

	```
	<html>
		<body>
			<div id="fruits"><br><br><br>
				<button type="button">데이터1</button><br><br>
				<button type="button">데이터2</button><br><br>
				<button type="button">데이터3</button><br><br>
			</div>
			<div id="fruits">
				<button type="button">데이터4</button><br><br>
				<button type="button">데이터5</button><br><br>
				<button type="button">데이터6</button><br><br>
			</div>
		</body>
	</html>

	Syntax for Xpath
	데이터5: (//button)[5]
	```

6. text function의 활용

	```
	<button type="button">데이터1</button><br><br>

	xpath with text : //button[text()='데이터1']

	번외
	//button[text()] 혹은 //button/text(): 텍스트를 가지는 모든 element들 추출
	```

7. contains function의 활용

	```
	<html>
	<body>
		<div id="fruit"><br><br><br><br><br><br><br><br><br><br><br>
			<button type="button">데이터 Blue</button><br><br>
			<button type="button" >데이터2</button><br><br>
			<button type="button">데이터3</button><br><br>
			<button type="button">데이터4</button><br><br>
			<button type="button">데이터5</button><br><br>
			</div>
		</body>
	</html>

	Syntax for Xpath
	데이터 Blue: //button[contains(text(),'Blue')]
	데이터2:  //button[contains(text(),'2')]

	attribute 사용시 예제
	e.g. //button[contains(@type,'but')]
	```

******

**Reference List**

- `CSS`

[w3schools-CSS](https://www.w3schools.com/css/default.asp)  
[chercher.tech-CSS](https://chercher.tech/python/css-selector-selenium-python)  

- `Xpath`

[w3schools-Xpath](https://www.w3schools.com/xml/xpath_intro.asp)  
[chercher.tech-Xpath](https://chercher.tech/java/relative-xpath-selenium-webdriver)


# Critical Rendering Path

## 들어가며

Critical Rendering Path(이하 CRP)는 브라우저가 html을 전달받아 화면에 초기(initial)에 그리는 일련의 순서를 의미하는 단어다.
화면이 그려지는 시간을 단축해서 사용자에게 빠르게 전달 및 반영하기 위해서는 화면을 그리는 과정에 대해 이해할 필요가 있다.

CRP는 크게 세가지 스텝으로 정리할 수 있다.

1. 우선 HTML과 CSS를 활용해 Render Tree를 구축하는 과정
2. Viewport를 계산하여 각 요소들의 위치와 크기를 정하는 Layout 과정
3. pixel로 화면을 그리는 paint 과정

위 순서에 따라 최종적으로 브라우저에 HTML이 어떻게 표현되는지 알아보자.

## Render Tree 구축

Render tree는 마크업과 스타일에 대한 정보를 tree 자료구조로 추상화한 데이터 모델이라고 할 수 있다.
이를 구축하기 위해서는 별개의 두 트리, DOM Tree와 CSSOM Tree를 만들어야 하는데 Render Tree는 앞 두 트리를 결합하여 새롭게 생성한 또 다른 Tree이다. 🌲🎄🌳

### DOM Tree

DOM은 무엇일까? 흔히 들어보았던 DOM은 Document of Model의 약자로 마크업에 대한 정보(tag name, attribute, child element ...)를 추상화한 객체를 의미한다.
Javascript를 사용해 조작하던 DOM api는 Web에서 제공하는 api로 DOM Tree에서 구축된 DOM을 참조하여 조작(manufacturing)한다.

Render Tree를 형성하기 위한 첫 단계로 DOM Tree는 아래의 순서에 따라 구축된다.

1. Conversion: 브라우저는 디스크나 네트워크를 통해 HTML의 raw bytes를 읽는다. 그리고 head 태그에 명시된 인코딩된 문자로 번역을 시도한다.
2. Tokenizing: 브라우저는 문자를 별개의 토큰(`<html>`, `<body>`)으로 변환한다.
3. Lexing: 생성된 토큰은 속성과 규칙에 의해 object로 변환된다.
4. DOM cunstruction: 마크업은 객체간의 관계를 파악하여 트리 자료구조로 연결된다.

위 프로세스를 거친 결과물이 바로 DOM이고 이 과정은 브라우저에서 마크업을 처리할 때마다 반복된다.
해당 과정은 복잡한 HTML을 처리할 때 다소 시간이 소요될 수 있다.

생성된 DOM Tree는 오직 마크업에 대한 정보만을 포함한다. 즉 어떻게 보일지는 DOM Tree만으로는 충분하지 않다. 그것은 CSSOM Tree의 역할이다.

### CSSOM

브라우저가 위에서 순차적으로 읽는 중 Link 태그를 만나면 href 속성에 정의된 스타일을 참조하여 CSSOM Tree를 구축하게 된다. HTML과 마찬가지로 CSS 또한 브라우저가 처리할 수 있는 형태로 변형을 거친다. 그 과정은 HTML을 DOM Tree로 변경하는 과정과 유사하며 차이점이라면 CSSOM Tree를 구축하는 과정에는 사용자가 정의한 스타일 뿐 아니라 agent-style이라고 불리는 브라우저에 따라 다른 요소들의 스타일 또한 포함하고 있다는 것이다.

그렇다면 CSS는 굳이 Tree 자료구로조 파싱하는 이유는 무엇일까?
해당 이유는 CSS의 첫 약자 cascading과 관련이 있다.
일반적으로 스타일은 부모에 선언한 스타일 속성이 포함된 자식들 모두에도 적용되는 동작을 한다. 예로 body에 font-size를 30px로 정의했을 떄 따로 선언하지 않은 div 태그나 p태그 또한 같은 font-size를 전달 받는 현상이 그러하다. 이는 스타일 또한 Tree 형태의 자료구조로 구축되어 부모에 선언한 스타일이 자식 요소에도 cascading down 된다.

#### Rendering block resource

CSS는 렌더링 블록 요소라고 취급된다. CSSOM Tree를 형성하기 전까지는 Render Tree를 형성할 수 없기 때문이다. 그렇기에 CRP 시에 CSS는 브라우저에 최대한 빠르게 전달하는 것이 중요하다.

이 외에도 Rendering Block을 최소화하기 위한 방법은 무엇이 있을까?

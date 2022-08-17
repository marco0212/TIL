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
이에 대한 답은 CSS를 non blocking 요소로 만드는 것이다.

how?
스타일의 여러 선언들을 분리하여 필요한 요소들만을 render blocking 요소로 만드는 것이다.
우리는 반응형 스타일과 프린트 시 표시되는 스타일, orientation에 따른 스타일을 정의한다. 해당 스타일들은 특정 컨디션에서만 사용되는 스타일로 media query라고 부른다.
media query는 특정 상황에 스타일을 표기할 수 있는 유용한 기능이지만 때때로 우리의 스타일을 불필요하게 무겁게 만들어 render tree를 구축하는데 방해가 되고는 한다.
해당 요소들을 적절하게 분리하여 화면을 그리는데 필요한 sheet들만을 다운로드하도록 하여 화면 구성에 필요한 시간을 개선할 수 있다.

```
<link href="style.css" rel="stylesheet" />
<link href="print.css" rel="stylesheet" media="print" />
<link href="other.css" rel="stylesheet" media="(min-width: 40em)" />
```

### Render Tree

위 과정을 거쳐왔다면 이제 Render Tree를 구축하기 위한 준비물은 모두 준비되었다. Render Tree는 HTML을 실제 그리기 위해 필요한 데이터들의 조합이다.
Dom Tree의 정보(Mark up)와 CSSOM Tree의 정보를 결합하여 새로운 트리를 형성하게 된다. 이 트리에서 주목할만한 부분은 실제 그리기 위한 데이터들의 조합이라는 부분이다.
브라우저에서 그리기 위해 필요하지 않은 부분들은 제외하고 트리가 구축되는 것이 Render Tree의 특성이다.
HTML의 `<head>`, `<meta>`, `<script>` 등의 요소들은 마크업에서 필요하지만 화면을 그리는데는 관여하지 않는 태그이므로 트리 구축 시 제외된다.
또한 css의 `display:none;` 속성 역시 브라우저가 렌더링 할 때 제외할 요소이기 때문에 제외된다.

> `display:none`의 사촌 `visiblity: hidden`은 보이지 않을 뿐 화면에 추가되기 때문에 Render Tree에 포함된다.

## Layout (reflow)

Render Tree를 성공적으로 구축했다면 다음 단계는 Layout이다. Render Tree에는 어떤 요소들을 어떻게 보일지에 대한 정보를 포함하고 있지만 어느정도 크기에 어느 곳에 위치할 것인지 전체 시각적인 구조(layout)에 대한 연산이 이루어져 있지 않기 때문에 바로 그릴 수는 없다.

layout 과정에는 viewport에 대한 정보가 필요하다. viewport는 화면의 전체 크기에 대한 값이며 head안에서 설정이 가능하다.

> viewport를 설정하지 않으면 default viewport는 960px이다.

```html
<html>
  <head>
    <meta name="viewport" content="width=device-width,initial-scale=1" />
  </head>
  <body>
    <div style="width: 100%">
      <!-- 100% of viewport -->
      <div style="width: 50%">hello</div>
      <!-- 50% of parent element (has same width of viewport) -->
    </div>
  </body>
</html>
```

## Paint

CRP의 마지막 과정으로 요소들을 pixel로 화면에 그리는 단계이다. 이전의 과정들을 통해서 각각의 요소들을 그리는데 필요한 정보를 가지고 실제 화면에 그려가는 과정이다.
이 과정은 브라우저에 상당히 큰 비용을 필요로 하는 작업이다. 더구나 box-shadow나 background-image: gradient 같은 속성은 다른 속성들보다 더 많은 연산을 필요로 하기에 더 많은 시간이 소요될 수 있다.

## 마치며

Critical Rendering Path는 위의 일련의 과정을 통해 화면에 HTML을 전달받아 그리는 과정을 의미하며 이를 최적화한다는 것은 각 스텝에 따른 연산을 최소한으로 하여 더 빠르게 유저에게 첫 화면을 보여주기 위한 목적이다. 만일 이를 최적화한다면 추후 DOM 조작으로 인해 그리는 과정 또한 효율적으로 처리할 수 있게 된다. 매 브라우저의 요소가 바뀌는 순간마다 CPR이 반복적으로 처리되기 때문이다.

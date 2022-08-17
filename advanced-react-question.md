# Advanced React Question

This document is for prepairing front end (react.js) developer

## 1. What is the React Virtual DOM?

Virtual DOM은 실제 DOM을 추상화하여 다양한 이점을 취하기 위해 고안된 사고입니다.
React에서는 React DOM이라는 라이브러리를 통해 렌더링을 관리하게 되는데 이는 실제 DOM을 추상화한 Javascript 객체입니다. 실제 Document의 Model에 대한 정보를 메모리에 저장하고 변경이 일어나면 실제 DOM과 일치하는 형태로 동작하게 됩니다.

Virtual DOM은 React가 좋은 성능을 유지하는 것에 크게 기여합니다. 이를 설명하기 위해서는 우선 모던 웹 어플리케이션에서는 잦은 DOM 변경이 있다는 가정과 DOM 변경은 비용을 많이 소모하는 작업이라는 상호 인지가 필요할 거 같습니다.

> 왜 DOM 조작은 비용이 많이 소모되는 작업인가요?

> > 이를 잘 설명하기 위해서는 브라우저가 HTML을 전달받아 화면에 그리는 과정, 즉 Critical Rendering Path를 설명드려야 할 거 같습니다.
> > Critical Rendering Path는 브라우저가 초기에 화면을 그리는 과정을 의미하지만 DOM의 변경이 있은 후 해당 내용을 화면에 반영하기 위해서는 Render Tree 형성, 레이아웃, 페인트 과정을 반복하기 때문에 성능을 고려한다면 이를 잘 이해하고 있는 것이 중요합나디.

> > Critical Rendering Path는 매우 복잡한 프로세스이지만 간략하게 크게 세단계로 구분할 수 있습니다.

> > 첫 단계는 Render Tree를 형성하는 과정입니다. Render Tree는 마크업과 스타일에 대한 정보를 구조화하여 브라우저가 그릴 수 있게 모델링하는 과정을 가르킵니다. Render Tree를 구축하기 위해서는 우선 DOM Tree, 마크업에 대한 정보를 포함한 트리 형태의 데이터를 구축합니다. 이 과정이 마무리되면 각 요소들의 스타일에 대한 트리 형태의 데이터도 구축하게 됩니다. 두 트리를 구축하게 되면 이를 결합한 새로운 트리 Render Tree를 구축하게 됩니다. Render Tree는 브라우저에 표현되는 요소들만을 포함하는 특징이 있습니다. 그렇기 떄문에 meta, script 태그와 같이 box model이 없는 태그들은 제외되고 또한 display:none과 같은 스타일 속성을 포함하는 요소도 트리 형성에서 제외 됩니다.

> > 두번째 단계는 Layout 단계 혹은 reflow 단계라고 불리는 과정입니다. 이 과정은 요소를 실제 브라우저에 표현할 때 어느 정도의 크기로 표현할지 위치는 어떤 지 등 레이아웃 설계와 관련된 값을 연산하고 표현하는 단계입니다. 위에서 문서에 대한 정보와 스타일에 대한 정보를 조합하여 Render Tree를 형성하였으니 해당 데이터를 참조하여 실제 view port를 기준으로 요소들의 위치와 크기 정보를 연산합니다. 이 때 viewport는 마크업의 viewport 설정을 한 메타 태그를 기반으로 설정합니다. 만약 해당 메타 태그가 없다면 기본값 960px을 기준으로 합니다.

> > 마지막 단계는 요소들의 스타일을 브라우저에 pixel로 그리는 paint 과정입니다. 이 단계는 다른 과정보다 더 많은 연산을 필요로 하는 것으로 알고 있습니다. 실제 요소들의 배경색, 그림자, 서체 색 기타 등등 스타일을 브라우저에 표현합니다. 물론 shadow나 gradient와 같은 스타일들은 다른 속성들에 비해 더 많은 연산과 처리하는데 소요되는 시간을 필요로 합니다.

> > DOM은 변경사항을 반영하기 위해 위 과정을 반복하기 때문에 DOM 조작하는 일은 브라우저로 하여금 매우 비싼 연산이라고 취급할 수 있습니다.

위와 같은 이유로 실제 DOM을 조작하는 일은 매우 값이 비싼 연산이라고 할 수 있습니다. 이러한 이유로 React에서는 보다 좋은 성능을 위해 lazy하게 DOM을 Update하는 방식을 채택하였습니다. Lazy하게 DOM을 업데이트한다는 것은 유저의 인터렉션에 의해서 혹은 타이머 등 어떤 방식으로든 DOM의 변경이 있을 때 이를 바로 실제 DOM에 반영시키지 않고 memory에 구조화된 VDOM에 선 반영한 후 update sync가 종료되면 이를 한번에 실제 DOM에 반영하여 더 빠르고 효율적으로 UI의 변경을 반영하는 전략을 취합니다.

## 2. Why do we need to tranpile React code?

React에서는 JSX라는 javascript의 확장된 syntax를 사용하여 웹 어플리케이션을 구축합니다. 브라우저는 html, css, javascript 세가지 언어만을 인식하고 실행할 수 있고 JSX는 실행할 수 없기 때문에 브라우저가 읽을 수 있는 형태로 변경해줄 필요가 있습니다. 보통 babel과 같은 transpilier를 활용해 JSX로 작성된 코드를 javascript로 변환해주는 과정을 거치게 됩니다.

## 3. What is the significance of keys in React?

키는 VDOM이 해당 요소를 식별하는 ID 역할을 합니다. ID를 통해 요소를 식별하여 요소를 업데이트하여 재렌더링할지 unmount할지 새롭게 추가해야할지를 결정합니다. key값은 동일 레벨의 자식 요소들에서만 유일하면 되며 이는 VDOM의 성능에 영향을 미칩니다.

## 4. What is the significance of Refs in React?

Ref는 리엑트에서 참조값을 유지시켜야 할 때 사용되는 api입니다. 일반적으로 리액트 컴포넌트가 리렌더링이 일어나면 선언된 참조값들은 새로운 참조를 반환하므로 기존의 참조를 유지해야 경우에 사용됩니다.

## 5. What are the most common approaches for styling a React application?

저는 CSS in JS 방식을 선호합니다. 이유는 크게 두가지 입니다. CSS는 특성상 모든 스타일 시트의 스코프가 글로벌리하게 선언되기 때문에 중복되는 스타일을 핸들링해야 합니다. 그에 반해 CSS in JS 방식은 모듈 스코프와 동일하게 동작하기 때문에 스타일을 핸들링하기에 수월합니다. 또한 설계한 컴포넌트의 Props에 따라 스타일에 변화를 줄 수 있습니다. CSS in JS는 결국 Javascript에 의해 구성되는 CSS이기 때문에 condition에 따른 스타일을 정의할 수 있고 이는 CSS in JS가 주는 이점이라고 생각합니다.

## 6. What are some of the performance optimization strategies for React?

lazy download, memoization

## 7. What is prop drilling and how to avoid it?

context api, state management library

## 8. What is the StrictMode component and why would you use it?

## 9. What are synthetic events in React?

react내에서 event 관련 api를 구현한 방식입니다. 이를 통해 실행되는 브라우저에 따른 크로스 브라우징 이슈를 해결해줍니다.

## 10. Why is it not advisable to update state directly, but use the setState call?

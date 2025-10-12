## JSX 핵심 문법과 자바스크립트 변환을 돌아봐야 하는 이유

빌드 과정에서 의미를 알기 어려운 에러 메시지와 마주허간, 코드는 같은데 예상치 못하게 발생하는 리렌더링으로 어딘가 모르는 찝찝함을 남겨본 경험이 있습니다. 이러한 문제들의 근본적인 원인은 우리가 작성한 JSX 코드 자체보다, 브라우저가 이해할 수 있는 순수 자바스크립트로 변환된 결과물에 숨어 있는 경우가 더 많습니다.

따라서 JSX의 변환 원리를 파고드는 것은 단순한 지적 탐구를 넘어, 중급 개발자로서 반드시 갖춰야할 실용적인 역량을 길러주는 필수 과정입니다.

JSX가 React.createElement() 혹은 최신 자동 런타임의 jsx() 함수 호출로 변환되는 과정을 알면, 렌더링마다 불필요한 함수나 객체가 재생성되는 코드를 즉시 식별할 수 있습니다. 이는 불필요한 최적화와 필수 최적화를 구분하는 날카라온 시각을 제공합니다.

## JSX 변환하기

JSX는 리액트 함수를 편하게 호출하는 문법적 설탕이기 때문에 JSX의 많은 구성 요소들을 브라우저가 직접 이해할 수 있게 순수 자바스크립트로 변환하는 과정이 필요합니다.

### 자동 런타임이란

리액트 17버전부터 자동 런타임(Automatic runtime)이 도입되면서, JSX 변환의 패러다임이 크게 바뀌었습니다. 리액트 17버전 이전, 즉 클래식 런타임(Classic runtime) 환경에서는 JSX를 사용하는 모든 파일 상단에 `import React from 'react';` 구문을 반드시 포함해야 했습니다. 그 이유는 트랜스파일러가 JSX 구문을 `React.createElement()` 함수 호출로 변환했기 때문이ㅏㅂ니다.

```jsx
import React from 'react';

// 변환 전 JSX
const element = <h1 className="greeting">Hello, world</h1>;


// 변환 후 자바스크립트(클래식 런타임)
const element = React.createElement('h1', {className: 'greeting'}), 'hello, world');
```

위 코드처럼 변환된 React.createElement() 코드가 정상적으로 실행되려면, 리액트 객체가 해당 파일의 스코프 내에 존재해야만 했습니다.

리액트 17버전은 이러한 ㅈ=불편함을 개선하기 위해 새로운 JSX 변환 방식인 자동 런타임을 도입했습니다. 이제 개발자는 더 이상 명시적으로 React를 import할 필요가 없게 되었습니다. 자동 런타임 환경에서 트랜스파일러는 React.createElement() 대신 리액트 패키지에 내장된 별도의 함수를 자동으로 import하여 사용합니다.

**프로덕션 환경**: react/jsx-runtime에서 jsx()와 jsxs() 함수를 가져와 사용합니다.
**개발 환경**: react/jsx-dev-runtime에서 jsxDEV()를 가져와 사용하며, 여기에는 개발에 유용한 추가 검증 및 경고 기능이 포함됩니다.

이는 다음과 같은 이점을 제공합니다.

**코드 간소화**: 불필요한 import 구문이 사라져 코드가 간결해지고 깔끔해집니다.
**번들 크기 감소**: 모든 파일에 React 전체를 포함하지 않아도 되므로 최종 번들 크기가 최적화 될 여지가 있습니다.
**개선된 개발 경험**: import 구문을 사용하지 않아 에러를 마주할 일이 거의 사라집니다. 단 `useState()`, `useEffect()` 등의 훅을 사용할 떄는 여전히 React에서 임포트해야 합니다.

## 바벨(Babel)

바벨은 자바스크립트의 트랜스파일러로 최신 JS 문법이나 JSX를 구형 브라우저에서도 실행 가능한 코드로 변환하는 도구입니다. 이를 위해 보통 바벨 플러그인인 @babel/preset-react를 사용하는데, 이 프리셋 내부적으로 JSX 변환 플러그인인 @babel/plugin-transform-react-jsx 플러그인을 사용하여 JSX를 설정합니다. 바벨 설정은 JSON 포맷 파일이나 .babelrc에서 작성할 수 있습니다.

```json
// 바벨 설정 파일 작성하기, 자동 런타임 적용 전
{
  "presets": ["@babel/preset-env", "@babel/preset-react"]
}
```

### 자동 런타임 미적용

#### JSX 원본 코드

```jsx
// JSX 원본 코드 (Babel 변환 전)
import React from "react";

// 간단한 JSX 예제
const element = <h1 className="welcome">Hello, JSX!</h1>;

// 중첩된 JSX 예제
const nestedElement = (
  <div className="container">
    <h2 className="title">중첩된 JSX 예제</h2>
    <p className="content">JSX는 중첩 구조를 쉽게 표현할 수 있습니다.</p>
    <button onClick={() => alert("클릭됨!")}>클릭해보세요</button>
  </div>
);

// 커스텀 컴포넌트 예제
const MyButton = ({ color, children }) => (
  <button style={{ backgroundColor: color, color: "white", padding: "10px" }}>
    {children}
  </button>
);

const customComponentExample = <MyButton color="blue">Click Me</MyButton>;

// 조건부 렌더링 예제
const showMessage = true;
const conditionalExample = (
  <div>
    {showMessage ? <p>메시지가 표시됩니다.</p> : <p>메시지가 숨겨집니다.</p>}
  </div>
);

console.log("JSX 예제 파일이 로드되었습니다.");
```

#### 트랜스파일된 자바스크립트 파일

package.json에 있는 빌드 명령어를 수행하면 dist 폴더 아래에 자바스크립트 파일이 생성됩니다.

```js
"use strict";

var _react = _interopRequireDefault(require("react"));
function _interopRequireDefault(e) {
  return e && e.__esModule ? e : { default: e };
}
// JSX 원본 코드 (Babel 변환 전)

// 간단한 JSX 예제
var element = /*#__PURE__*/ _react["default"].createElement(
  "h1",
  {
    className: "welcome",
  },
  "Hello, JSX!"
);

// 중첩된 JSX 예제
var nestedElement = /*#__PURE__*/ _react["default"].createElement(
  "div",
  {
    className: "container",
  },
  /*#__PURE__*/ _react["default"].createElement(
    "h2",
    {
      className: "title",
    },
    "\uC911\uCCA9\uB41C JSX \uC608\uC81C"
  ),
  /*#__PURE__*/ _react["default"].createElement(
    "p",
    {
      className: "content",
    },
    "JSX\uB294 \uC911\uCCA9 \uAD6C\uC870\uB97C \uC27D\uAC8C \uD45C\uD604\uD560 \uC218 \uC788\uC2B5\uB2C8\uB2E4."
  ),
  /*#__PURE__*/ _react["default"].createElement(
    "button",
    {
      onClick: function onClick() {
        return alert("클릭됨!");
      },
    },
    "\uD074\uB9AD\uD574\uBCF4\uC138\uC694"
  )
);

// 커스텀 컴포넌트 예제
var MyButton = function MyButton(_ref) {
  var color = _ref.color,
    children = _ref.children;
  return /*#__PURE__*/ _react["default"].createElement(
    "button",
    {
      style: {
        backgroundColor: color,
        color: "white",
        padding: "10px",
      },
    },
    children
  );
};
var customComponentExample = /*#__PURE__*/ _react["default"].createElement(
  MyButton,
  {
    color: "blue",
  },
  "Click Me"
);

// 조건부 렌더링 예제
var showMessage = true;
var conditionalExample = /*#__PURE__*/ _react["default"].createElement(
  "div",
  null,
  showMessage
    ? /*#__PURE__*/ _react["default"].createElement(
        "p",
        null,
        "\uBA54\uC2DC\uC9C0\uAC00 \uD45C\uC2DC\uB429\uB2C8\uB2E4."
      )
    : /*#__PURE__*/ _react["default"].createElement(
        "p",
        null,
        "\uBA54\uC2DC\uC9C0\uAC00 \uC228\uACA8\uC9D1\uB2C8\uB2E4."
      )
);
console.log("JSX 예제 파일이 로드되었습니다.");
```

위 결과에서 볼 수 있듯이 JSX는 React.createElement() 함수 호출로 변환되었습니다. React.createElement()는 다음과 같이 3개의 인자를 받습니다.

**타입**: 생성할 요소의 타입을 나타냅니다. 문자열 h1처럼 HTML 기본 태그 이름이 올 수도 있고 커스텀 컴포넌트라면 해당 컴포넌트를 가리키는 변수나 클래스를 전달합니다.
**프로퍼티 객체**: 요소에 전달할 속성들을 객체로 표현합니다. 예시 코드에서는 JSX의 className 속성을 전달합니다. 만약 속성이 없다면 null 또는 {}로 표현됩니다.
**자식 요소**: 세 번째 이후의 인자들은 모두 해당 요소의 자식 요소들로 예제 코드에서는 문자열이 하나이므로 하나의 자식만 전달했지만 여러 자식이 있다면 더 많은 인자 혹은 배열로 전달됩니다.

### 자동 런타임 적용

#### 자동 런타임 바벨 설정

```json
{
  "presets": [
    "@babel/preset-env",
    [
      "@babel/preset-react",
      {
        "runtime": "automatic" // 자동 런타임 설정
      }
    ]
  ]
}
```

#### 변환된 자바스크립트

```js
"use strict";

// react/jsx-runtime에서 jsx, jsxs 함수를 자동으로 임포트함
var _jsxRuntime = require("react/jsx-runtime");
// 새로운 JSX 변환(React 17+) 예제
// 이 파일은 React를 import하지 않아도 됩니다

// 간단한 JSX 예제
var element = /*#__PURE__*/ (0, _jsxRuntime.jsx)("h1", {
  className: "welcome",
  children: "Hello, New JSX Transform!",
});

// 중첩된 JSX 예제
var nestedElement = /*#__PURE__*/ (0, _jsxRuntime.jsxs)("div", {
  className: "container",
  children: [
    /*#__PURE__*/ (0, _jsxRuntime.jsx)("h2", {
      className: "title",
      children: "React 17+ JSX \uBCC0\uD658",
    }),
    /*#__PURE__*/ (0, _jsxRuntime.jsx)("p", {
      className: "content",
      children:
        "React 17\uBD80\uD130\uB294 JSX\uB97C \uC704\uD574 React\uB97C import\uD560 \uD544\uC694\uAC00 \uC5C6\uC2B5\uB2C8\uB2E4.",
    }),
    /*#__PURE__*/ (0, _jsxRuntime.jsx)("button", {
      onClick: function onClick() {
        return console.log("클릭됨!");
      },
      children: "\uD074\uB9AD\uD574\uBCF4\uC138\uC694",
    }),
  ],
});

// 기능적으로는 동일하지만, import React가 필요하지 않음
console.log("새로운 JSX 변환 예제 파일이 로드되었습니다.");
```

```js
children: [
    /*#__PURE__*/ (0, _jsxRuntime.jsx)("h2", {
      className: "title",
      children: "React 17+ JSX \uBCC0\uD658",
    }),
    ...
```

트랜스파일러가 react/jsx-runtime 모듈로부터 \_jsxRuntime, jsx()를 임포트한 뒤 사용하고 있습니다. 이 함수는 리액트 내부에서 제공하는 것으로 최정적으로는 리액트 앨리먼트 객체를 생성하는 역할을 합니다.

자동 런타임 덕분에 JSX만 사용한다면 React를 임포트하지 않아도 되지만, `React.createContext()`와 같은 구문으로 리액트 훅이나 API를 사용하려면 여전히 React를 임포트해야합니다.

즉 react/jsx-runtime은 JSX 변환을 위한 전용 엔트리포인트일 뿐 JSX 변환과 리액트 API 사용은 구분해야 합니다. jsx()는 리액트에서 제공하는 공식 API에 포함되지 않습니다. 따라서 실제 개발을 하면서 직접 jsx()를 호출 할 일은 없을 겁니다.

### SWC로 JSX 변환해보기

최근에는 바벨의 대안으로 SWC가 많이 사용됩니다.

SWC는 러스트로 작성된 초고속 컴파일러이자 번들러로 싱글 스레드에서 바벨보다 20배가량 빠른 속도로 변환 작업을 수행합니다. Next.js Parcel, Deno와 같이 SSR 프레임워크부터 자바스크립트 런타임까지 많은 분야에서 활용되고 있습니다. 예를 들어 Next.js는 바벨대신 SWC를 기본 컴파일러로 사용하여 JSX와 타입스크립트 코드를 변환하고 있습니다. SWC를 이용하면 바벨과 같은 결과의 JSX 변환을 얻을 수 있으며 설정 방법에 따라 바벨과 호환되는 방식 또는 최신 리액트 방식으로 JSX를 트랜스파일할 수 있습니다.

SWC는 `.swcrc` 파일에 JSON 형식으로 설정 옵션을 지정할 수 있습니다.

#### SWC 설정 파일 예시

```json
{
  "jsc": {
    "parser": {
      "syntax": "ecmascript",
      "jsx": true
    },
    "transform": {
      "react": {
        "runtime": "classic",
        "pragma": "React.createElement",
        "pragmaFrag": "React.Fragment"
      }
    },
    "target": "es2015"
  },
  "module": {
    "type": "commonjs"
  }
}
```

## SWC 자동 런타임 지정

```json
{
  "jsc": {
    "parser": {
      "syntax": "ecmascript",
      "jsx": true
    },
    "transform": {
      "react": {
        "runtime": "automatic",
        "importSource": "react",
        "pragma": "React.createElement",
        "pragmaFrag": "React.Fragment"
      }
    },
    "target": "es2015"
  },
  "module": {
    "type": "commonjs"
  }
}
```

## ESBuild로 JSX 변환해보기

ESBuild는 Go 언어로 작성된 초고속 번들러로 JSX 역시 지원합니다. 비트와 같은 최신프론트엔드 빌드 도구들은 내부적으로 Esbuild를 사용하여 타입스크립트와 JSX 코드를 빠르게 변환하고 있습니다.

바벨이나 SWC와 달리 ESBuild는 별도의 설정 없이도 기본적으로 JSX를 인식합니다. 파일 확장자가 .jsx, .tsx인 경우 ESBuild는 JSX 구문을 파싱하여 표준 자바스크립트로 변환하는데 기본 동작은 JSX를 React.createElement() 호출로 변환하는 겁니다. ESBuild에서 제공하는 커맨드라인 인터페이스를 사용하여 JSX파일을 변환할 수 있습니다.

```js
// classic mode
runCommand( // runCommand()는 child_process의 execSync() 함수를 래핑한 함수
    'npx esbuild src/App.jsx --bundle --outfile=dist/cli-classic.js',
    'Classic 모드로 JSX 변환 (기본값)'
):

// automatic mode
runCommand(
    'npx esbuild src/AppNoReact.jsx --jsx=automatic --bundle --outfile=dist/cli-automatic.js',
    'Automatic 모드로 JSX 변환 (React 17+)'
)
```

## React.createElement

React.createElement()와 jsx()가 무엇을 반환하는지 이해하면 JSX의 역할을 더욱 명확히 할 수 있습니다. React.createElement()와 jsx()는 리액트 엘리먼트라고 불리는 자바스크립트 불변 객체를 반환합니다.

이 객체는 리액트가 실제 DOM을 만들기 전에 가지고 있는 일조으이 청사진으로 바로 가상 DOM이라고 불리는 구조의 한 조각입니다.

```json
{
  $$typeof: Symbol(react.element),  // React 엘리먼트 타입 식별자
  type: 'div',                      // 태그 이름 또는 컴포넌트 함수/클래스
  props: {                          // 속성과 자식 요소
    className: 'container',
    children: [...]
  },
  key: null,                        // 리스트 렌더링 시 요소 식별용
  ref: null,                        // DOM 노드 또는 컴포넌트 참조용
  _owner: null                      // 내부 사용 필드
}
```

이 객체가 바로 리액트 엘리먼트입니다.

`$$typeof`: 이 객체가 리액트 엘리먼트임을 나타내는 내부 식별용 속성으로 리액트 내부에서는 심보릉ㄹ 사용하여 정의되어 있습니다.
`type`: 요소의 타입으로 여기서는 div 태그를 나타냅니다.
`key`, `ref`: 리액테으에서 리스트 렌더링이나 참조에 사용하는 추가 빌드입니다.
`props`: 해당 요소에 전달된 속성들 및 children을 포함하는 객체입니다.

이 객체는 브라우저의 실제 DOM 노드가 아닌 가상 DOM을 표현하기 위한 객체 트리를 구성합니다. 리액트 클래스 컴포넌트의 render() 함수나 함수 컴포넌트에서는 이런 리액트 엘리먼트 객체를 반환받아 가상 DOM 트리를 구축합니다. 그리고 ReactDOM은 이 가상 DOM을 바탕으로 실제 DOM을 생성하거나 업데이트합니다.

즉, 우리가 작성한 JSX는 JSX => React.createElement() => 리액트 엘리먼트 객체 생성이라는 과정을 거치고 이런 객체 변환을 위해 리액트는 UI를 선언적이고 효율적으로 다룰 수 있습니다. 리액트는 이 불변 객체를 메모리 상에서 비교하고 필요한 부분만 실제 DOM에 반영함으로써 성능을 최적화합니다.

## JSX의 핵심 문법 돌아보기

### 템플릿 러터럴

ES6에서 소개된 템플릿 리터럴은 `기호로 문자열을 감싸 동적 값을 포함하거나 여러 줄 문자열을 작성할 수 있는 문법입니다. 작은따옴표와 큰따옴표대신 백틱 기호를 사용해 문자열을 표시할 수 있게 합니다. 템플릿 리터럴은 각 문자열을 덧셈이나 String.prototype.concat() 연산 없이 내부에 변수를 포함할 수 있고, 공백이나 개행을 표기하는 데 이스케이프 시퀀스인 \ 기호를 추가로 사용할 필요가 없다는 장점이 있습니다.

```jsx
// 템플릿 리터럴 기본 예제

// 기본 문자열 연결
const color = "golden";
const rabbit = "Rabbit";
const cat = "Cat";

// 1. 기존 방식 문자열 연결
const goldenRabbit = color + rabbit; // goldenRabbit

// 2. 템플릿 리터럴 사용
const goldenCat = `${color}${cat}`; // goldenCat

// 3. 템플릿 리터럴의 여러 줄 문자열 지원
const multiLineText = `
첫 번째 줄
두 번째 줄
세 번째 줄
`;

// 4. 템플릿 리터럴 내에서 표현식 사용
const count = 5;
const message = `${color}${rabbit}이 ${count}마리 있습니다.`;
console.log(message); // goldenRabbit이 5마리 있습니다.

// 5. 템플릿 리터럴 내에서 계산 수행
const price = 1000;
const tax = 0.1;
const totalPrice = `총 가격: ${price * (1 + tax)}원`;
console.log(totalPrice); // 총 가격: 1100원
```

### 태그드 템플릿 리터럴

ES6에서 처음 소개된 태그드 템플릿 리터럴은 템플릿 리터럴에 태그 함수를 적용하여 문자경롹 동적 값을 분리하고 커스터마이즈된 처리 로직을 구현할 수 있는 문법입니다. 함수 호출 시 템플릿 리터럴을 사용할 수 있게 해줍니다.

```jsx
// 태그드 템플릿 리터럴 예제

// 1. 기본적인 태그 함수 정의
function fn(strings, ...values) {
  console.log("strings:", strings);
  console.log("values:", values);
  return "함수 실행 결과";
}

// 2. 태그드 템플릿 호출 방식 비교
console.log("--- 예제 1: 인자 없는 태그드 템플릿 ---");
fn`rabbit jump`; // ➊ 태그드 템플릿 호출
// 위 코드는 아래와 동일함
fn(["rabbit jump"]); // ➋ 일반 함수 호출로 표현

console.log("--- 예제 2: 인자가 있는 태그드 템플릿 ---");
const color = "Golden";
fn`this is a ${color} Rabbit`; // ➌ 인자가 있는 태그드 템플릿
// 위 코드는 아래와 동일함
fn(["this is a ", " Rabbit"], color); // ➍ 일반 함수 호출로 표현

// 3. 태그 함수를 활용한 실용적인 예제
console.log("--- 예제 3: 하이라이트 태그 함수 ---");
function highlight(strings, ...values) {
  // 결과 문자열을 담을 배열
  let result = [];

  // strings와 values를 번갈아가며 처리
  strings.forEach((str, i) => {
    result.push(str);
    if (i < values.length) {
      // 값을 강조 표시로 감싸기
      result.push(`<strong style="color: #FFD700;">${values[i]}</strong>`);
    }
  });

  // 배열을 문자열로 합치기
  return result.join("");
}

// highlight 태그 함수 사용
const animal = "토끼";
const action = "뛰어오르다";
const highlightedText = highlight`${animal}가 재빠르게 ${action}!`;

console.log(highlightedText);
// 출력: "<strong style="color: #FFD700;">토끼</strong>가 재빠르게 <strong style="color: #FFD700;">뛰어오르다</strong>!"

// 4. 인자 값들을 서식화하는 태그 함수
console.log("--- 예제 4: 통화 포맷팅 태그 함수 ---");
function currency(strings, ...values) {
  return strings.reduce((result, str, i) => {
    // 마지막 문자열이면 값 없이 처리
    if (i >= values.length) {
      return result + str;
    }

    // 값이 숫자인 경우 통화 형식으로 변환
    const value = values[i];
    const formatted =
      typeof value === "number"
        ? new Intl.NumberFormat("ko-KR", {
            style: "currency",
            currency: "KRW",
          }).format(value)
        : value;

    return result + str + formatted;
  }, "");
}

// currency 태그 함수 사용
const productName = "골드 토끼 인형";
const price = 25000;
const formattedPrice = currency`${productName}의 가격은 ${price}입니다.`;

console.log(formattedPrice);
// 출력: "골드 토끼 인형의 가격은 ₩25,000입니다."
```

JSX를 사용하는 리액트 환경에서는 동적으로 스타일을 적용할 때 CSS-in-js 방식을 사용할수 있습니다. styled-components로 태그드 템플릿을 사용해보면 다음과 같습니다.

```jsx
// styled-components 예제
import React from "react";
import styled from "styled-components";

// 1. 기본 styled-components 사용법
const StyledDiv = styled.div`
  color: white;
  background-color: gray;
  padding: 16px;
  border-radius: 4px;
  margin: 8px;
  font-family: Arial, sans-serif;
`;

// 2. props를 활용하는 styled-components
const Button = styled.button`
  background-color: ${(props) => (props.primary ? "#FFD700" : "#FFFFFF")};
  color: ${(props) => (props.primary ? "#000000" : "#FFD700")};
  padding: 8px 16px;
  border: 2px solid #ffd700;
  border-radius: 4px;
  cursor: pointer;
  font-weight: bold;

  &:hover {
    background-color: ${(props) => (props.primary ? "#E5C100" : "#FFF8E0")};
  }
`;

// 3. 컴포넌트 확장
const ExtendedButton = styled(Button)`
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  text-transform: uppercase;
`;

// 4. 예제 컴포넌트
const StyledComponentExample = () => {
  return (
    <div>
      <StyledDiv>Golden Rabbit</StyledDiv>

      <Button>일반 버튼</Button>
      <Button primary>프라이머리 버튼</Button>

      <ExtendedButton>확장된 버튼</ExtendedButton>
      <ExtendedButton primary>확장된 프라이머리 버튼</ExtendedButton>
    </div>
  );
};

export default StyledComponentExample;
```

## 합성 이벤트

기존에 바닐라 자바스크립트 별도 번들러 없이 개발하는 환경에서는 각 브라우저 버전에서 지원하는 기능인지 미리 확인하는 감지 코드를 작성해야 했습니다.

```jsx
// 이벤트 리스너 기능 감지 및 할당 예제
var addEvent = function (element, event, handler) {
  if (element.addEventListener) {
    element.addEventListener(event, handler, false);
  } else if (element.attachEvent) {
    // IE 지원
    element.attachEvent("on" + event, handler);
  } else {
    // 매우 구형 브라우저의 경우
    element["on" + event] = handler;
  }
};

// 사용 예시
addEvent(window, "load", function () {
  console.log("페이지가 로드되었습니다.");
});
```

리액트에서는 합성 이벤트로 인해 여러 브라우저에서 일관된 이벤트 인터페이스를 사용할 수 있습니다. 그 에제를 위해 드래그앤 드롭 컴포넌트 생성해봅니다.

```tsx
import React from "react";

function DragDrop() {
  // 드래그 시작 이벤트 핸들러
  const handleDragStart = (event: React.DragEvent<HTMLDivElement>) => {
    // 드래그되는 요소의 ID를 데이터로 설정
    event.dataTransfer.setData(
      "text/plain",
      (event.target as HTMLDivElement).id
    );

    // 드래그 효과 설정
    event.dataTransfer.effectAllowed = "move";

    // 드래그 이미지 커스터마이징 (선택 사항)
    // const dragIcon = document.createElement('img');
    // dragIcon.src = 'drag-icon.png';
    // event.dataTransfer.setDragImage(dragIcon, 0, 0);
  };

  // 드래그 오버 이벤트 핸들러
  const handleDragOver = (event: React.DragEvent<HTMLDivElement>) => {
    // 기본 동작 방지 (필수: 드롭 영역으로 인식하기 위함)
    event.preventDefault();

    // 커서 모양 변경을 위한 드롭 효과 설정
    event.dataTransfer.dropEffect = "move";
  };

  // 드롭 이벤트 핸들러
  const handleDrop = (event: React.DragEvent<HTMLDivElement>) => {
    // 기본 동작 방지
    event.preventDefault();

    // 드래그 시작 시 저장한 데이터 가져오기
    const data = event.dataTransfer.getData("text/plain");
    console.log("드롭된 데이터:", data);

    // 실제 애플리케이션에서는 여기서 상태 업데이트 등을 수행
    // setItems(prev => [...]);

    // 합성 이벤트의 native 이벤트 접근 예시
    console.log("네이티브 이벤트:", event.nativeEvent);
  };

  // 드래그 엔터 이벤트 핸들러 (선택적 구현)
  const handleDragEnter = (event: React.DragEvent<HTMLDivElement>) => {
    event.preventDefault();
    // 드롭 영역에 진입했을 때 스타일 변경 등을 위해 사용
    event.currentTarget.classList.add("drag-over");
  };

  // 드래그 리브 이벤트 핸들러 (선택적 구현)
  const handleDragLeave = (event: React.DragEvent<HTMLDivElement>) => {
    // 드롭 영역에서 나갔을 때 스타일 원복 등을 위해 사용
    event.currentTarget.classList.remove("drag-over");
  };

  return (
    <div className="drag-drop-container">
      <h2>리액트 합성 이벤트: 드래그앤드롭 예제</h2>

      {/* 드래그 가능한 요소 */}
      <div
        id="draggableItem"
        draggable="true"
        onDragStart={handleDragStart}
        style={{
          width: "150px",
          height: "75px",
          backgroundColor: "gold",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          cursor: "grab",
          borderRadius: "8px",
          boxShadow: "0 2px 4px rgba(0,0,0,0.2)",
          userSelect: "none", // 드래그 중 텍스트 선택 방지
        }}
      >
        이 요소를 드래그하세요
      </div>

      {/* 드롭 영역 */}
      <div
        className="drop-zone"
        onDrop={handleDrop}
        onDragOver={handleDragOver}
        onDragEnter={handleDragEnter}
        onDragLeave={handleDragLeave}
        style={{
          width: "300px",
          height: "200px",
          border: "2px dashed #aaa",
          borderRadius: "8px",
          marginTop: "20px",
          display: "flex",
          alignItems: "center",
          justifyContent: "center",
          color: "#666",
        }}
      >
        여기에 드롭하세요
      </div>

      <div
        className="explanation"
        style={{ marginTop: "20px", fontSize: "14px" }}
      >
        <p>
          위 예제는 리액트 합성 이벤트(Synthetic Event)를 사용한 드래그앤드롭
          구현을 보여줍니다. 각 이벤트 핸들러는 React.DragEvent 타입의 합성
          이벤트를 받습니다.
        </p>
      </div>
    </div>
  );
}

export default DragDrop;
```

리액트는 대부분의 이벤트 리스너를 개별 DOM 노드에 직접 부착하지 않고 애플리케이션의 최상위 레벨에 하나의 이벤트 리스너를 부착합니다. 이런 패턴을 이벤트 위임이라고 하는데 이랙트 16버전까지는 이벤트 리스너가 부착되는 최상위 레벨이 document 객체였고 17버전부터는 ReactDOM.render()가 호출되는 컨테이너 요소로 변경되었습니다.

유저가 특정 DOM 요소를 클릭하는 이벤트가 발생하면 브라우저는 해당 요소에 네이티브 클릭 이벤트를 발생시키고 이 이벤트는 DOM 트리를 따라 상위 요소로 버블링됩니다. 최상위에 부착된 리액트의 이벤트 리스너는 버블링되어 올라온 네이티브 이벤트를 감지하고 합성 이벤트 객체로 래핑하여 JSX에 바인딩 되었던 onClick()과 같은 적절한 이벤트 핸들러를 찾아 합성 이벤트 객체를 인자로 전달해 호출합니다.

리액트 16버전까지는 성능 최적화를 위해 합성 이벤트 객체를 pool이라는 장소에 저장해두고, 이벤트가 발생하면 이 풀에 있는 객체를 재사용했습니다. 이벤트 핸들러 호출이 끝나면 해당 이벤트 객체의 프로퍼티를 초기화한 뒤, 다시 풀로 반환하는 이벤트 풀링 방식을 사용했습니다. 그런데 이 이벤트 풀링은 이벤트 핸들러 호출 이후 합성 이벤트 객체의 속성이 null로 초기화된다는 부분 때문에 비동기적으로 이벤트 객체의 속성에 접근할 때 주의를 요했습니다.

```tsx
import React, { useState } from "react";

/**
 * 이벤트 풀링(Event Pooling) 예제
 *
 * React 16 이하에서는 성능 최적화를 위해 SyntheticEvent 객체를 재사용하는
 * '이벤트 풀링' 메커니즘을 사용했습니다. 이로 인해 이벤트 핸들러가 완료된 후에는
 * 이벤트 객체의 모든 속성이 null로 설정되었습니다.
 *
 * React 17부터는 이 메커니즘이 제거되어 이벤트 객체를 비동기적으로 사용해도
 * 문제가 발생하지 않습니다.
 */

// React 16 이하에서의 이벤트 풀링 문제 예시
function EventPoolingExample() {
  const [message, setMessage] = useState("이벤트를 클릭하세요");
  const [asyncMessage, setAsyncMessage] = useState("");

  // React 16 이하에서의 문제 상황
  const handleClickLegacy = (event: React.MouseEvent<HTMLButtonElement>) => {
    // 이벤트 객체를 비동기적으로 사용하려고 시도
    setTimeout(() => {
      // React 16 이하에서는 이 시점에 event 객체가 이미 재설정됨
      // TypeError 발생 가능성 있음
      try {
        // @ts-ignore: 이 코드는 실제로는 React 16 이하에서 오류 발생
        const eventType = event.type; // React 16에서는 null이 됨
        setAsyncMessage(`비동기 접근 결과 (React 16): ${eventType || "null"}`);
      } catch (error) {
        setAsyncMessage(`비동기 접근 오류 (React 16): ${error}`);
      }
    }, 0);

    // 동기적 사용은 정상 작동
    setMessage(`동기적 접근 결과: ${event.type}`);
  };

  // React 16에서의 해결책: event.persist() 사용
  const handleClickLegacyFixed = (
    event: React.MouseEvent<HTMLButtonElement>
  ) => {
    // 이벤트 객체 지속 유지
    event.persist(); // React 16에서 필요한 메서드

    setTimeout(() => {
      // event.persist()를 호출했으므로 여전히 유효함
      const eventType = event.type;
      setAsyncMessage(`event.persist() 사용 후 비동기 접근: ${eventType}`);
    }, 0);

    setMessage(`동기적 접근 결과: ${event.type}`);
  };

  // React 17 이후 정상 동작
  const handleClickModern = (event: React.MouseEvent<HTMLButtonElement>) => {
    // 이벤트 객체를 비동기적으로 사용
    setTimeout(() => {
      // React 17부터는 이벤트 풀링이 제거되어 문제 없음
      const eventType = event.type;
      setAsyncMessage(`비동기 접근 결과 (React 17+): ${eventType}`);
    }, 0);

    setMessage(`동기적 접근 결과: ${event.type}`);
  };

  // 권장되는 안전한 패턴: 필요한 속성만 미리 추출
  const handleClickSafest = (event: React.MouseEvent<HTMLButtonElement>) => {
    // 필요한 데이터만 미리 추출하여 변수에 저장
    const eventType = event.type;
    const eventTarget = event.currentTarget.textContent;

    setTimeout(() => {
      // 추출된 변수 사용 (버전에 관계없이 안전)
      setAsyncMessage(`안전한 접근 방식: ${eventType}, 대상: ${eventTarget}`);
    }, 0);

    setMessage(`동기적 접근 결과: ${event.type}`);
  };

  return (
    <div className="event-pooling-container">
      <h2>리액트 이벤트 풀링 예제</h2>

      <div className="event-buttons">
        <button onClick={handleClickLegacy} style={{ marginRight: "10px" }}>
          React 16 스타일 (오류 발생 가능)
        </button>

        <button
          onClick={handleClickLegacyFixed}
          style={{ marginRight: "10px" }}
        >
          React 16 + event.persist()
        </button>

        <button onClick={handleClickModern} style={{ marginRight: "10px" }}>
          React 17+ 스타일
        </button>

        <button onClick={handleClickSafest}>가장 안전한 방식</button>
      </div>

      <div className="event-results" style={{ marginTop: "20px" }}>
        <p>{message}</p>
        <p>{asyncMessage}</p>
      </div>

      <div
        className="explanation"
        style={{
          marginTop: "30px",
          fontSize: "14px",
          backgroundColor: "#f8f8f8",
          padding: "15px",
          borderRadius: "5px",
        }}
      >
        <h3>이벤트 풀링(Event Pooling) 이해하기</h3>
        <p>
          <strong>React 16 이하에서의 동작:</strong>
          <br />
          이벤트 핸들러가 호출되고 나면, 합성 이벤트 객체가 이벤트 풀로 반환되어
          재사용됩니다. 이는 이벤트 핸들러가 완료된 후에 이벤트 객체에 접근하면
          모든 속성이 null로 설정된다는 것을 의미합니다. 비동기 콜백에서 이벤트
          객체에 접근하려면 <code>event.persist()</code>를 호출해야 했습니다.
        </p>
        <p>
          <strong>React 17 이후의 변경점:</strong>
          <br />
          React 17부터는 이벤트 풀링이 제거되었습니다. 이제 합성 이벤트 객체가
          재사용되지 않으므로,
          <code>event.persist()</code>를 호출할 필요가 없으며 비동기 콜백에서도 이벤트
          객체에 안전하게 접근할 수 있습니다.
        </p>
        <p>
          <strong>권장되는 안전한 패턴:</strong>
          <br />
          그러나 버전에 관계없이 가장 안전한 방법은 비동기적으로 사용할 이벤트
          데이터를 미리 변수에 저장하는 것입니다. 이 방식은 코드가 명확해지고
          이벤트 객체에 대한 의존성을 줄입니다.
        </p>
      </div>
    </div>
  );
}

export default EventPoolingExample;
```

현대 자바스크립트 엔진과 브라우저가 객체 할당 및 가비지 컬렉션을 효율적으로 처리하기 떄문에 풀링의 성능 이점이 줄어들어 17버전부터는 이벤트 풀링은 제거되었으며 이로 인해 이벤트 시스템이 단순화되었습니다.

## 단일 루트 엘리먼트

JSX에서 각 컴포넌트는 루트 엘리먼트를 여러 개 동시에 가질 수 없기 떄문에 리액트 컴포넌트가 JSX를 반환할 떄는 항상 하나의 리액트 노드여야 합니다.

\_jsxs() 혹은, React.createElement()의 함수값을 반환합니다 리액트 컴포넌트는 항상 함숫값 한 개를 반환합니다.

따라서 리액트 컴포넌트가 JSX를 반환할 때는 항상 하나의 리액트 노드여야 합니다. 루트가 여러개면 에러가 발생합니다.

```jsx
var element = /*#__PURE__*/ _react["default"].createElement(
  "h1",
  {
    className: "welcome",
  },
  "Hello, JSX!"
);
```

## 아하 모먼트

JSX에서 트랜스파일러로 JS로 변환되어 브라우저로 가는 과정까지를 살펴보았습니다. 아무 생각없이 사용했던 선언형 프로그래밍을 위한 친절한 DSL이었다는 사실을 새삼 깨닫게 되었습니다. 어떤 프레임워크를 쓰던 결국 JS로 트랜스파일러가 변환해주어야 하기에 JS를 더 깊이 공부해야겠다는 생각을 하게되었습니다.

React.createElement()로 트랜스파일링 되는것을 어림잡아 알고 있었지만, 정확하게 어떻게 동작하지 몰랐는데 직접 babel로 변환된 예시 코드를 살펴보면서 리액트 컴포넌트의 객체를 좀 더 깊게 알 수 있었습니다.

어느날 부터인가 import React from 'react' 구문을 안써도 되었었는데, 이는 리액트가 버전업을 하면서 자동 런타임 기능을 도입했고, 설정해주면 된다는 사실을 이제야 알게 되었습니다.

styled-component의 함수 뒤쪽에 백틱으로 태그드 템플릿 리터럴을 아무 생각없이 사용했었는데, 이는 JSX에서 제공하는 기능이었다는 사실을 알게 되었습니다.

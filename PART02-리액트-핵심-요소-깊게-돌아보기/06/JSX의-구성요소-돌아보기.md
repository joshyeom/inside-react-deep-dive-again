## JSX

JSX는 구성 요소를 작성하는데 사용하는 단순한 구문이 아닙니다. 본질적으로 자바스크립트에서 UI를 표현하는 도메인 특화 DSL(Domain Specific Language)입니다.

JSX는 자바스크립트 코드에서 HTML과 유사한 구문을 사용하여 UI를 선언적으로 설계하는 문법 확장입니다.

JSX의 가장 큰 장점은 직관성입니다.

HTML과 거의 흡사한 문법 덕분에 디자이너와의 협업이 용이하고, 개발자들은 복잡한 UI구조도 마치 그림을 그리듯 쉽게 구성할 수 있습니다.

이런 편리함은 개발자가 상태 관리 로직과 비즈니스 로직에 더 집중하도록 하여 **생산성**을 크게 향상 시킵니다.

하지만 이 강력한 추상화에 이면에는 대가가 따릅니다. JSX가 단순한 HTML 템플릿처럼 보이기 때문에, 이것이 실제로는 자바스크립트 객체를 생성하는 별도의 함수 호출로 변환되는 핵심적인 동작 원리를 간과하기 쉽습니다.

## DSL(Domain Specific Language)

도메인 특화 언어로 데이터베이스의 쿼리, 스타일링, 서버 설정 등 특정 도메인의 문제를 해결할 목적으로 설계된, 간결하고 직관적인 문법을 가진 언어입니다.

### 외부 DSL

독자적인 문법과 파서를 가지며, 별개의 파일에서 독립적으로 실행되는 언어입니다.

CSS처럼 HTML 엘리먼트가 어떻게 보여야 하는지 묘사하는 스타일링이라는 특정 목적을 달성하는데 사용합니다.

### 내부 DSL

자바스크립트, 루비, C#과 같이 범용적인 프로그래밍 언어의 문법과 기능을 확장하며, 마치 새로운 언어처럼 보이게 만들며 특정 기능을 수행하는 역할을 하는 언어를 내부 DSL이라고 합니다.

JSX는 HTML의 태그와 유사하게 생긴 코드를 사용해 자바스크립트 파일 내부에서 UI 구조를 작성하는 용도로 만들어졌으며 자바스크립트에서 파생된 내부 DSL입니다.

중요한 점은 JSX가 자바스크립트 표준이 아니라는 사실입니다. 따라서 브라우저는 JSX 문법을 직접 해석하지 못하며, 바벨과 같은 트랜스파일러를 사용해 자바스크립트로 변경해야 합니다.

## JSXElements

JSX 구문의 단위를 가리키며 UI를 구성할 때 벽돌과 같은 역할을 하는 HTML, XML 요소와 같이 UI를 선언적으로 구조화 합니다.

### JSXElement

HTML 태그처럼 보이지만 리액트 컴포넌트로 변환되어 렌더링되는 기본 단위입니다.

JSXElement는 두 가지의 구성 요소를 가지고 있습니다.

#### JSXOpeningElement, JSXChildren, JSXSelfClosingElement로 이루어진 그룹

**JSXOpeningElement**: 태그 이름(div, p, h1, h2)이 JSXOpeningElement가 됩니다.

**JSXChildren**: 태그 사이의 내용이 JSXChildren가 됩니다.

**JSXSelfClosingElement**: JSXSelfClosingElement는 children 없이 혼자 사용되는 JSXElement입니다.

## 문법

HTML에서 기본적으로 제공되는 태그는 소문자로 사용 가능합니다.

커스텀 컴포넌트일때는 파스칼 케이스로 사용해야 합니다.

커스텀 컴포넌트를 소문자로 작성 시 경고 메시지가 뜹니다. 그 이유는 커스텀 컴포넌트가 캌멜 케이스일 경우에는 커스텀 컴포넌트인지 구분하기 힘들기 때문입니다.

또한 JSX를 트랜스파일링 하는 과정에서 문제가 발생할 수 있습니다.

### JSX를 트랜스파일링 했을때 변환되는 HTML 기본 태그

```jsx
function Button(){  // 트랜스파일링 전
    return (
        <div>Hello, JSX!</div>
    )
}

function Button(){  // 트랜스파일링 후
    return nn.jsx("div", {
        className: e.className,
        onClick: e.onClick,,
        children: e.children
    })
}

```

### JSX를 트랜스파일링 했을때 변환되는 커스텀 컴포넌트

```jsx
function Button() {
  // 트랜스파일링 전
  return <Button>Hello, JSX!</Button>;
}

function Button() {
  // 트랜스파일링 후
  return En.jsx(Button, {
    children: "Hello, JSX!",
  });
}
```

커스텀 컴포넌트의 경우 문자열이 아니라 커스텀 컴포넌트가 첫 번째 인수가 됩니다.

빌드 과정에서 미니파이가 되어 각 변수들이 무엇을 의미하는지 알아보기 어려울 수 있지만 함수 인수의 타입을 알아볼 수 있습니다.

소문자로 사용했다면 다음처럼 빌드되어 커스텀 컴포넌트와 기본 태그를 구분할 수 없게 됩니다.

```jsx
function Button2() {
  // 트랜스파일링 전
  return <customButton>Hello, JSX!</customButton>;
}

// 난수가 아닌 문자열로 변환되어 HTML 기본 태그로 인식되는 customButton
function Button2() {
  // 트랜스파일링 후
  return En.jsx("customButton", {
    children: "Hello, JSX!",
  });
}
```

### JSX Fragment

JSX에서 부모 요소 없이 여러 자식 요소를 그룹화하에 사용되는 특벼랗ㄴ 문법으로 빈 태그 `<></>`를 사용합니다.

### JSXElementName

JSX에서 요소의 태그 이름으로, 기본 HTML 태그나 유저 정의 리액트 컴포넌트를 지정하는데 사용됩니다.

### JSXIdentifier

단일 식별자를 나타냅니다. 예를 들어 `<Component />`에서 Component는 JSXIdentifier가 됩니다.

### JSXNamespacedName

특정 그룹안에 속해 있는 컴포넌트를 `<namespace:Component />`와 같이 표현하는데 사용하는 요소입니다.

대부분에 프레임워크에서 이를 지원하지 않는데 이는 XML이라고 하는 HTML과 유사한 템플릿 언어 내부에서 JSX를 표현할 목적으로 만든 스펙이었습니다.

### JSXMemberExpression

객체의 타입 속성처럼 `.` 기호로 구분된 멤버 표현식을 나타냅니다.

이는 네임스페이스나 객체의 중첩된 구조를 표현할 때 사용됩니다.

특정 모듈이나 객체를 중심으로 다양한 컴포넌트를 하나의 그룹으로 묶는데 사용합니다. **합성 컴포넌트 패턴**에서 유용하게 사용됩니다.

### JSXAttributes

JSX 엘리먼트에 추가 정보를 전달하는 키 값 형태의 설정으로 HTML 속성과 유사하게 React 컴포넌트는 props로 전달됩니다.

#### JSXAttribute

JSX 요소에 전달되는 개별 속성을 나타냅니다. 이는 속성의 이름과 값으로 구성됩니다.

#### JSXAttributeName

속성 이름을 나타냅니다. 이는 속성의 식별자로 문자열 형태로 표현됩니다.

#### JSXAttributeInitializer

속성값을 할당하는 부분을 나타냅니다.

#### JSXSpreadAttribute

기존 객체의 속성을 JSX 요소에 한꺼번에 전달할 떄 사용하는 속성입니다. 이는 스프레드 연산자로 사용하여 구현됩니다.

### JSXChildren

JSX 요소에 전달되는 자식 요소를 나타냅니다. 이는 문자열, 숫자, JSXElement, JSXFragment 등 다양한 형태로 표현됩니다.

#### Props, Attribute로 봐야하는가?

Props는 맞고 Attribute는 아닙니다.

그 이유는 children은 `props.children`으로 전달되고, Attribute는 HTML 태그에 추가 정보를 제공하기 위해 사용되는 속성이기 때문입니다.

### JSXStrings

JSX에서 렌더링되는 문자열은 알파벳 뿐만이 아니라 유니코드로 표기할 수 있는 아이콘이나 이모지들도 포함됩니다.

#### HTML 엔티티

HTML에서 태그를 구성하는 `<>` 같은 특수문자를 표현하는데 `&lt;`와 같은 문자열을 사용하는데 이를 HTML 엔티티(entity)라고 합니다.

개행 처리나 빈 칸이 제대로 노출이 안 되는 경우가 있어 이때는 HTML 엔티티를 사용해 표시해야합니다.

```jsx
function App() {
  return (
    <div>
      <p>Hello \n World \n I'm &lt;p&lt; tag</p>
    </div>
  );
}
```

## 아하 모먼트

책에서 JSX 스펙 문서를 꼭 읽어보길 권장하여 잠시 훑어보면서 중요한 정보를 얻을 수 있었습니다. JSX에 대한 개발 철학인데 내용은 다음과 같습니다.

> JSX는 정의된 의미 체계 없이 ECMAScript에 대한 XML과 유사한 구문 확장입니다. 엔진이나 브라우저에서 구현하도록 의도되지 않았습니다. JSX를 ECMAScript 사양 자체에 통합하자는 제안은 아닙니다. 이는 다양한 전처리기(트랜스파일러)에서 이러한 토큰을 표준 ECMAScript로 변환하는 데 사용되도록 고안되었습니다.

> 기존 언어에 새로운 구문을 포함시키는 것은 위험한 모험입니다. 다른 구문 구현자 또는 기존 언어는 호환되지 않는 다른 구문 확장을 도입할 수 있습니다. 독립 실행형 사양을 통해 다른 구문 확장의 구현자가 자신의 구문을 설계할 때 JSX를 더 쉽게 고려할 수 있도록 합니다. 이를 통해 다양한 새로운 구문 확장이 공존할 수 있기를 바랍니다.

JSX는 ECMAScript의 공식 기술로 채택되지 않았고, 문서를 보니 그럴 계획도 없어 보입니다.

오로지 기존 HTML/JS를 더 쉽게 구현하도록 빠르고 확장적이게 만들기 위한 철학을 엿볼 수 있었습니다.

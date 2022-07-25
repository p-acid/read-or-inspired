---
title: "React Pattern 학습"
created_at: "2022-07-19"
last_updated_at: "2022-07-19"
---

# VAC Pattern

> [워드프레스 : React에서 View의 렌더링 관심사 분리를 위한 VAC 패턴 소개](https://wit.nts-corp.com/2021/08/11/6461) 글 요약

## Why VAC

- 대규모의 기업에서 원활한 협업을 위해 FE 개발을 UI 개발과 인터랙션 및 기능 개발 영역으로 구분
- 리액트로 개발을 진행하면 JS와 마크업이 혼합된 상태로 결과물이 나오고 이를 각 영역에서 수정하는 과정에서 충돌이 발생
- View 영역과 로직 영역을 분리할 필요성이 제기

## What VAC

- VAC는 View Asset Component의 약자로 렌더링에 **필요한 JSX와 스타일을 관리하는 컴포넌트**를 의미
- JSX를 추상화 한 Props Object를 생성하여 VAC에 전달하는 방식

```tsx
// View Component
const SpinBox = () => {
  const [value, setValue] = useState(0);

  const props = {
    value,
    onDecrease: () => setValue(value - 1),
    onIncrease: () => setValue(value + 1),
  };

  // JSX를 VAC로 교체
  return <SpinBoxView {...props} />;
};
```

## How VAC

- 올바른 VAC는 상태나 핸들러를 올바르게 바인딩 할 뿐, **직접적으로 무엇을 하는 지에 대해서 관여하지 않음**
- 컨테이너 컴포넌트 영역과 VAC 컴포넌트를 분리하여 **협업에 용이하게 작업**할 수 있음
- VAC 속성을 정의하면서 사용하여 **렌더링에 직관적인 상태 관리**도 가능

# 2022 React Component Design Pattern

> [Log Rocket : React component design patterns for 2022](https://blog.logrocket.com/react-component-design-patterns-2022/) 글 요약

## Introduction

- **디자인 패턴**은 일반적인 소프트웨어 개발 문제에 대한 **솔루션 템플릿**
- 해당 포스팅에선 다음 부분들에 효과적인 패턴들을 소개
  - [횡단 관심사](https://ko.reactjs.org/docs/higher-order-components.html#use-hocs-for-cross-cutting-concerns)[^1)]
  - 글로벌 데이터 공유
  - 복잡한 상태 저장 로직과 같은 관심사를 컴포넌트와 분리하는 것

[^1)]: [횡단 관심사란?](https://ko.wikipedia.org/wiki/%ED%9A%A1%EB%8B%A8_%EA%B4%80%EC%8B%AC%EC%82%AC)

## [The higher-order component pattern(HOC pattern)](https://ko.reactjs.org/docs/higher-order-components.html#use-hocs-for-cross-cutting-concerns)

- 애플리케이션 전체에서 컴포넌트 로직을 재사용하는데 사용되는 고급 리액트 패턴
- 구체적으로 **컴포넌트를 가져와 새 컴포넌트를 반환하는 함수**
- 입력된 컴포넌트를 수정하지 않고, 상속을 사용하여 동작을 복사하지도 않음
- 사이드 이펙트가 전혀 없는 순수 함수

```jsx
const EnhancedComponent = higherOrderComponent(WrappedComponent);
```

- 유사한 개념으로는 Redux의 [connect](https://github.com/reduxjs/react-redux/blob/master/docs/api/connect.md#connect)와 Relay의 [createFragmentContainer](https://relay.dev/docs/v10.1.3/fragment-container/#createfragmentcontainer)

## [The provider pattern](https://www.patterns.dev/posts/provider-pattern/)

- 컴포넌트 트리의 **여러 컴포넌트에서 전역 데이터를 공유하는데 사용되는 고급 패턴**
- 리액트에서는 [Context API](https://ko.reactjs.org/docs/context.html)[^2)]를 통해 구현할 수 있다.
- **prop drilling**[^3)] 문제를 해결하기 위해 사용한다.
  - 즉, prop drilling 없이 컴포넌트 트리 내에서 데이터 공유가 가능하다.
- 현재 Context 값을 구독하기 위한 `useContext` Hook도 존재

[^2)]: 간단하게 말하자면, 리액트 컴포넌트 트리 안에서 전역적(global)이라고 볼 수 있는 데이터를 공유할 수 있도록 고안된 방법이다.
[^3)]: 하향식 데이터 흐름에서 최종 prop 전달 컴포넌트까지 각 단계별로 prop을 명시적으로 전달하는 프로세스

## [The compound components pattern](https://kentcdodds.com/blog/compound-components-with-react-hooks)

- **여러 컴포넌트가 상태를 공유하고 로직을 처리할 수 있는 간단하고 효율적인 방법**을 제공하는 고급 리액트 컨테이너 패턴
- `context API` 혹은 `React.cloneElement` 를 통해 구현 가능
- [점 표기법을 사용](https://ko.reactjs.org/docs/jsx-in-depth.html#using-dot-notation-for-jsx-type)하여 하나의 컴포넌트에서 새로운 컴포넌트를 만들어 포함시킬 수 있음

## [The presentational and container component patterns](https://www.patterns.dev/posts/presentational-container-pattern/)

- 관심사 분리를 위한 패턴. 복잡한 상태 저장 로직을 분리하는데 도움이 됨.
- 그러나 Hooks를 사용하면 간단히 이를 가능케 하므로 Hooks 사용을 권장

## [The Hooks pattern](https://ko.reactjs.org/docs/hooks-rules.html#gatsby-focus-wrapper)

- 함수형 컴포넌트에서 생명주기 관련 로직을 활용하기 위한 기능
- 클래스형 컴포넌트[^4)]를 보다 더 깔끔하게 리팩토링 가능

[^4)]: [클래스형 컴포넌트가 갖는 문제](https://ko.reactjs.org/docs/hooks-intro.html#classes-confuse-both-people-and-machines)를 해결할 수 있다.

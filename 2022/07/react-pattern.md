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

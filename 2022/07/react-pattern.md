---
title: "React Pattern 학습"
created_at: "2022-07-19"
last_updated_at: "2022-07-19"
---

# VAC Pattern

## Why VAC

> [워드프레스 : React에서 View의 렌더링 관심사 분리를 위한 VAC 패턴 소개](https://wit.nts-corp.com/2021/08/11/6461) 글 요약

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

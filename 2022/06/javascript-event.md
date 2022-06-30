---
title: "자바스크립트 이벤트"
created_at: "2022-06-28"
last_updated_at: "2022-06-28"
---

# 문제 해결 과정

## 문제 상황

- 래핑하고 있는 HTML 요소와 내부 HTML 요소 둘 다 `onClick` 이벤트를 갖고 있을 때, **내부 요소 `onClick` 이벤트 동작에서 상위 요소 이벤트 핸들러를 실행시키지 않아야 한다.**


```jsx
/* 내부 구조 */

<div onClick={() => parentEventHandler()}>
  <button {() => childrenEventHandler()}></button>
</div>
```

## 해결 내용

- `childrenEventHandler` 내부에 [`event.stopPropagation()`](https://developer.mozilla.org/ko/docs/Web/API/Event/stopPropagation) 메서드를 추가한다.

## 확장 논의

- 실제 상황에서 `event.stopPropagation()` 이 없는 경우, `parentEventHandler` 의 `alert` 메서드가 먼저 실행되었다.
  - 버블링과 캡쳐링, 이벤트 실행 순서에 대한 학습이 필요할 것 같음

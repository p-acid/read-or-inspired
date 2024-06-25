> 원본 영상: [Mastering memoization in React - Advanced React course, Episode 5](https://www.youtube.com/watch?v=huBxeruVnAM)

<br/>

# 들어가며

- 리액트 성능과 관련된 내용으로 **메모이제이션**이 많이 언급된다.
- 큰 프로젝트에서 많은 함수를 `useCallback` 으로 래핑하는 경우, 실제로 불필요한 리렌더링을 방지하고 있는지 의심해야 한다.
- 이를 위해 메모이제이션의 적절한 활용 방식과 `useMemo`, `useCallback` 의 동작 방식 그리고 `memo` 의 역할과 한계에 대해서 알아본다.

<br/>

# 리렌더링의 의존성 문제 해결하기

- 원시 값의 경우 값을 직접 비교하지만 참조 값의 경우 메모리 주소 값을 참조하기 때문에 동일하게 작성된 함수 내지 객체여도 동일하지 않은 경우가 있다.
  - 이것을 얕은 비교라고 한다.
- 그렇기에 `useEffect` 의 의존성 배열에 참조 값이 들어가는 경우 리렌더링 마다 변수 값이 재생성되기에 실제로 변경사항이 없더라도 의존성 배열의 변화가 감지되어 `useEffect` 훅이 매번 트리거된다.
- 이때 `useCallback` 등을 활용하여 함수의 재생성을 방지할 수 있어 명확하게 변경사항을 감지할 수 있다.
- 함수가 아닌 경우 `useMemo` 를 통해 값을 메모이징 할 수 있다.

<br/>

## `useMemo` 가 `useCallback` 보다 성능이 우수하다는 미신

- 이러한 미신의 전제는 `useCallback` 은 매 렌더링마다 함수를 재생성하는 반면에 `useMemo` 는 그렇지 않다는 것이다.
- 일단 `useCallback` 이 렌더링마다 함수를 재생성되는 것은 맞다.
  - 정확히 말하자면 참조 값을 일치시키기 위해 첫 번째 인자로 전달 받은 콜백 함수를 캐싱하여 값의 변화가 있을 경우에만 변경된 함수를 저장하고 이를 반환한다.
- 미신이 틀린 이유는 `useMemo` 또한 이와 동일하게 동작하기 때문이다. `useMemo` 도 콜백 함수를 인수로 받는다.
  - 다만 `useMemo` 가 다른 점은 콜백 함수 그대로를 저장하는 `useCallback` 과 달리 반환된 값을 저장한다는 것이다.
- 결론적으로 두 훅의 성능은 동일하다.

<br/>

# Prop을 메모이제이션 할 때 유의점

- 메모이제이션을 할 때 prop으로 전달되는 값을 메모이제이션 하는 경우가 많은데 이를 다음과 같이 사용하는 경우가 있다.

<br/>

## 안티 패턴

```tsx
const Component = () => {
  const onClick = useCallback(() => {
    // do something
  }, []);

  return <button onClick={onClick}>click me</button>
}
```

- 위와 같은 경우엔 컴포넌트가 리렌더링 될 때 컴포넌트 내 모든 요소가 다시 렌더링 된다.
- 그렇기에 여기서 `useCallback` 을 사용한 것은 불필요한 작업이며 오히려 가독성을 낮춘다.

<br/>

## 첫 번째 메모이제이션 대상

```tsx
const Parent = () => {
  const fetch = () => {};

  return <Child onMount={fetch} />;
}

const Child = ({ onMount }) => {
  useEffect(() => {
    onMount()
  }, [onMount])

  return ...
}
```

- 우선 첫 번째로 상위 컴포넌트로 부터 전달된 prop 값이 하위 컴포넌트의 다른 훅 내에서 활용되는 경우이다.
- 이 경우 `fetch` 함수가 메모이징 되지 않으면 `Parent` 함수가 리렌더링 될 때 불필요한 리렌더링이 발생할 수 있다.

## 두 번째 메모이제이션 대상

```tsx
const MemoChild = React.memo(Child)

const Child = ({ onMount }) => {
  const memoData = useMemo(() => ({ id: 1 }), []);

  return <MemoChild id="1" data={memoData} />
}
```

- 두 번째는 컴포넌트가 `React.memo` 로 감싸진 경우다.
- `React.memo` 로 감싼 컴포넌트는 상위 컴포넌트가 리렌더링 되더라도 해당 컴포넌트의 prop 값의 변화가 없다면 리렌더링 하지 않아 불필요한 리렌더링을 줄일 수 있다.
- 그렇기에 위와 같이 객체 형태의 값이 prop으로 전달되면 참조 값의 유지를 위해 메모이징을 진행해야 한다.

<br/>

## 놓칠 수 있는 케이스

```tsx
const useForm = () => {
  // not memoized
  const submit = () => {
    // do something
  };

  return { submit };
}

const Component = () => {
  const { submit } = useForm();

  return <MemoChild onChange={submit} />
}
```

- prop으로 전달되는 함수가 커스텀 훅에서 반환된 함수고 해당 함수가 메모이징 되지 않은 경우

  
```tsx
const Component = () => {

  return (
    <MemoChild>
      <div>Some text here</div>
    </MemoChild>
  );
}
```

- 전달된 `children` 요소가 메모이징 되지 않은 상태


<br/>

# 리액트 성능 개선을 위한 메모이제이션 다시 생각하기

- 계산을 기억하는 것에 메모이제이션이 많이 사용되며 대부분의 사람들은 값 비싼 계산을 메모이징 하려고 한다.
- 이를 이해하기 위해서 **값 비싼 계산은 무엇인가**에 대해 우선적으로 이해할 필요가 있다.

> [!NOTE]
> 값 비싼 계산인지 여부는 실행 환경, 빈도에 많이 의존한다. 당연한 이야기지만 환경에 따라 실행 속도가 상이하고 빈도가 많을 수록 자주 호출되기에 부하가 더 클 것이다. 또한 일반적으로 웹 프론트엔드에선 소수 계산을 제외하고 컴포넌트 렌더링이 일반적으로 큰 부하를 준다.
>
> 그렇기에 실행 환경 및 빈도를 고려하고 그러한 계산이 리렌더링에 영향을 주고 있는 지에 대해 파악하고 있다면 효율적인 메모이징을 진행할 수 있을 것이다.

- 메모이제이션은 렌더링에 영향이 없다면 의미가 없다. 오히려 초기 렌더링에 추가적인 작업을 늘릴 뿐이다.
- 단 한 번의 메모이제이션으론 측정이 어렵지만, 큰 앱에서 많은 메모이제이션을 사용하다 보면 초기 렌더링을 느리게 만들 수 있다.

<br/>

# 소감

- 메모이제이션 대상에 대해 다시 생각하게 되었다.
  - 무엇을 위해 메모이제이션 하는 가에 대해 깊게 생각해보는 계기가 되었고, 실질적으로 어떤 문제를 해결하기 위해 메모이제이션을 하는 지도 생각해볼 수 있었다.
- `useMemo`, `useCallback` 의 메커니즘에 대해 간단히 알아봤는데 둘의 차이에 대해 더 명확히 알 수 있었던 순간이었다.

---
title: "Compound Component 학습"
created_at: "2022-07-04"
last_updated_at: "2022-07-04"
---

## 요약

---

- `<select>` 요소를 활용한 컴포넌트를 구성하는 것은 **지원하는 CSS 스타일이 한정적이기에** 스타일 적용이 어렵다.
- `<select>` 요소는 `<label>`, `<select>`, `<option>` 까지 세 가지 인터페이스를 갖는다.
- `value` 를 배열로 받아 내부적으로 렌더링 하게 되면 **확장이 어려워 변경에 취약**해지기에 같은 인터페이스, 기능을 유지해야 한다.
- **필수 기능 정의**
  - 키보드, 마우스로 조작 가능(이동 및 선택)
  - 리스트가 열려있을 경우, 옵션에 포커스
  - 옵션 검색 가능
- **구현 사항 정리**
  - 옵션 리스트 렌더링 여부 상태(`isOpen`) 필요
  - 선택되었는지 여부 판단을 위한 `ref` 와 `value` 필요
- **Compound Component API**
  - **요구되는 기능 수행을 위해 두 개 이상의 컴포넌트가 협력하는 형태**
  - 보통 **부모 - 자식 컴포넌트로 구성**되며, 컴포넌트 간 드러나지 않는 상태 공유가 존재
  - **컴포넌트 구성**
    ```tsx
    <Select open={open} defaultValue="value">
      <Select.Trigger>open select</Select.Trigger>
      <Select.OptionList>
        <Select.Option value="value1">value1</Select.Option>
        <Select.Option value="value2">value2</Select.Option>
        <Select.Option value="value3">value3</Select.Option>
      </Select.OptionList>
    </Select>
    ```
    - `open` 과 `value` 를 `<Select>` 컴포넌트와 `<Select.Option>` 컴포넌트에서 암묵적으로 공유하는 상태

# Context API?

---

- 모든 수준에서 `props` 를 전달(하향식)하지 않고도 구성 요소 트리를 통해 데이터를 전달할 수 있는 방법을 제공

## Context를 사용해야 하는 이유

---

```tsx
class App extends React.Component {
  render() {
    return <Toolbar theme="dark" />;
  }
}

function Toolbar(props) {
  /*
		Toolbar 컴포넌트는 ThemedButton에 "theme" prop을 전달하기 위해 
		해당 prop을 전달 받아야 한다.
		이렇게 발생되는 prop drilling은 사이에 거치는 컴포넌트가 많을 수록
		더 큰 문제가 된다.
	*/
  return (
    <div>
      <ThemedButton theme={props.theme} />
    </div>
  );
}

class ThemedButton extends React.Component {
  render() {
    return <Button theme={this.props.theme} />;
  }
}
```

```tsx
// 컨텍스트를 사용하면 아래와 같이 prop drilling 문제를 해결할 수 있다.

// 1. createContext로 context 생성
const ThemeContext = React.createContext("light");

class App extends React.Component {
  render() {
    // 2. context 전달을 위한 Provider 생성
    return (
      <ThemeContext.Provider value="dark">
        <Toolbar />
      </ThemeContext.Provider>
    );
  }
}

function Toolbar() {
  return (
    <div>
      <ThemedButton />
    </div>
  );
}

class ThemedButton extends React.Component {
  // 3. 위에서 생성한 ThemeContext를 통해 참조
  static contextType = ThemeContext;
  render() {
    return <Button theme={this.context} />;
  }
}
```

## Context를 사용하기 전에

---

- Context는 중첩이 많은 컴포넌트에서 일부 데이터에 접근할 때 주로 사용한다.
  - 컴포넌트 재사용을 어렵게 만들기에 자주 사용하지 않도록 한다.
  - 대안으로 [컴포넌트 합성(Composition)](https://ko.reactjs.org/docs/composition-vs-inheritance.html#gatsby-focus-wrapper)이 훨씬 수월할 수 있다.
    - **합성**이란 서로 다른 객체를 여러개 붙여 새로운 기능이나 객체를 구성하는 것으로, 특정 `prop` 을 통해 컴포넌트를 전달하여 구성하는 방식
      ```tsx
      /*
      	아래와 같이 Avatar에 avatarSize를 전달하기 위해
      	여러 컴포넌트를 거쳐가게 됨
      */

      <Page user={user} avatarSize={avatarSize} />
      // ... 그 아래에 ...
      <PageLayout user={user} avatarSize={avatarSize} />
      // ... 그 아래에 ...
      <NavigationBar user={user} avatarSize={avatarSize} />
      // ... 그 아래에 ...
      <Link href={user.permalink}>
        <Avatar user={user} size={avatarSize} />
      </Link>
      ```
      ```tsx
      /*
      	이때 컴포넌트 합성을 통해 Link 컴포넌트 자체를 전달하는 것으로 해결
      */

      function Page(props) {
        const user = props.user;
        const userLink = (
          <Link href={user.permalink}>
            <Avatar user={user} size={props.avatarSize} />
          </Link>
        );
        return <PageLayout userLink={userLink} />;
      }

      // 이제 이렇게 쓸 수 있습니다.
      <Page user={user} avatarSize={avatarSize} />
      // ... 그 아래에 ...
      <PageLayout userLink={...} />
      // ... 그 아래에 ...
      <NavigationBar userLink={...} />
      // ... 그 아래에 ...
      {props.userLink}
      ```
      - 이러면 `Link` 및 `Avatar` 컴포넌트가 해당 두 가지 `props` (`user`, `avatarSize`)를 쓴다는 것을 알아야 하는건 `Page` 뿐이다.
  - 더 나아가 [**render props**](https://ko.reactjs.org/docs/render-props.html) 를 통해 렌더링 전에 부모 - 자식 컴포넌트 간의
    소통을 가능하게 할 수 있다. - **render props** 는 **무엇을 렌더링할 지 컴포넌트에게 알려주는 함수이다.**
            ```tsx
            class Cat extends React.Component {
              render() {
                const mouse = this.props.mouse;
                return (
                  <img src="/cat.jpg" style={{ position: 'absolute', left: mouse.x, top: mouse.y }} />
                );
              }
            }

            class Mouse extends React.Component {
              constructor(props) {
                super(props);
                this.handleMouseMove = this.handleMouseMove.bind(this);
                this.state = { x: 0, y: 0 };
              }

              handleMouseMove(event) {
                this.setState({
                  x: event.clientX,
                  y: event.clientY
                });
              }

              render() {
                return (
                  <div style={{ height: '100vh' }} onMouseMove={this.handleMouseMove}>

                    {/*
                      <Mouse>가 무엇을 렌더링하는지에 대해 명확히 코드로 표기하는 대신,
                      `render` prop을 사용하여 무엇을 렌더링할지 동적으로 결정할 수 있습니다.
                    */}
                    {this.props.render(this.state)}
                  </div>
                );
              }
            }

            class MouseTracker extends React.Component {
              render() {
                return (
                  <div>
                    <h1>Move the mouse around!</h1>
                    <Mouse render={mouse => (
                      <Cat mouse={mouse} />
                    )}/>
                  </div>
                );
              }
            }
            ```

            - 위와 같이 컴포넌트를 동적으로 전달하는 것이 가능해진다.
            - **“render props pattern“** 이라고 불린다고 꼭 `prop` 명으로 `render` 를 사용할 필요는 없다.
- 흔한 예시로는 로케일, 테마, 데이터 캐시 등의 관리가 있다.

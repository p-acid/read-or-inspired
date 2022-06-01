---
title: 모노레포 실습
created_at: "2022-05-30"
---

# 모노레포 툴 학습

## Yarn workspaces

- **`yarn link` 혹은 `npm link` 기능을 선언적으로 사용**하는 방식
- `workspace` 에 대한 **심볼릭 링크**가 생성
  - 이를 통해 하나의 레포지토리 내 여러 프로젝트가 상호 참조

<br/>

### 용어

- **project** : 저장소
  - 하나 이상의 worktree 포함
  - 최소 한 개의 workspace(즉, 루트 workspace) 존재
- **workspace** : 모노레포 패키지
- **worktree** : 자식 workspace를 갖는 workspace
- **glob pattern** : 와일드카드 문자를 사용하여 일정한 형식의 파일 이름들을 저장하기 위한 패턴
  - ex) `testMatch: ["**/test/**/*.test.(ts|js)"]`

<br/>

### worktree 선언

- worktree를 구성하는 workspace의 위치를 glob 패턴의 배열로 나타낸다.

```json
// packages 폴더 내 모든 폴더가 workspace가 되도록 함
{
  "private": true,
  "workspaces": ["packages/*"]
}
```

<br/>

### workspace 추가

![모노레포 추가](../../asset/monorepo-practice/add-workspace.png)

- `client`, `server`, `common` 세 개의 `workspace` 를 추가하고 `yarn` 명령을 실행하면 다음과 같이 루트 디렉토리에 `node_modules` 생성

<br/>

### workspace에 대한 명령 실행

특정 `workspace` 에 정의된 스크립트 실행

```sh
yarn workspace <WORKSPACE_NAME> <COMMAND_NAME>
```

<br/>

### workspace를 의존성으로 추가

패키지 간 의존성을 만들려면 다음 방법으로 가능

- `package.json` 에 의존성 명시
- `yarn workspace` 명령 활용

```sh
yarn workspace client add common@1.0.0
```

위 명령을 사용하면 다음과 같이 된다.

![의존성 추가](../../asset/monorepo-practice/add-dependencies.png)

이렇게 되면 npm 레지스트리에 배포된 `common` 이란 이름의 패키지가 있다 해도, **의존성 버전을 충족하면 로컬에 존재하는 `common` workspace를 우선하여 설치**한다.

`client` 에서 `common` 의존성을 잘 부르는지 확인하기 위해 테스트를 진행한다.

우선 아래 파일들을 추가한다.

```js
// client/client.js
const common = require("common");
common.hello();
```

```js
// common/index.js
module.exports = {
  hello() {
    console.log("This is common");
  },
};
```

```json
// client/package.json
{
  "name": "client",
  "version": "1.0.0",
  "license": "MIT",
  "dependencies": {
    "common": "1.0.0"
  },
  "scripts": {
    "start": "node client.js"
  }
}
```

그리고 아래 명령어를 실행하면,

```sh
yarn workspace client run start
```

![의존성 호출 테스트](../../asset/monorepo-practice/dependency-test.png)

<br/>

### workspace 의존 관계 확인

아래 명령어를 통해 확인 가능

```sh
yarn workspaces info
```

> Yarn 2.x 부터는 아래 명령어로 확인
>
> ```sh
> yarn workspaces list
> ```

![의존성 호출 테스트](../../asset/monorepo-practice/check-dependencies.png)

<br/>

### 모든 workspace에 대해 명령 실행

아래 명령어를 통해 모든 workspace들을 순회하며 명령어 스크립트를 실행한다.

```sh
yarn workspaces run <command>
```

> Yarn 2.x 부터는 아래 명령어로 확인
>
> ```sh
> yarn workspaces foreach <command>
> ```

<br/>

### 루트 프로젝트에 의존성 추가

다음 명령을 통해 workspace가 아닌 **루트 프로젝트에 의존성**을 추가

```sh
yarn add <PACKAGE_NAME> -W
```

루트 디렉토리에 리액트 추가

<div style={{display: "flex", gap: "8px"}}>
    <img src="../../asset/monorepo-practice/add-dependencies-in-root-1.png" alt="add-dependencies-in-root-1" />
    <img src="../../asset/monorepo-practice/add-dependencies-in-root-2.png" alt="add-dependencies-in-root-2" />
</div>

<br/>

### 호이스팅(의존성 끌어올리기)

npm, yarn 등은 중복 의존성 설치 방지를 위해 [호이스팅 기법](https://classic.yarnpkg.com/blog/2018/02/15/nohoist/)을 사용

> **Yarn Hoisting 관련 글 : 유령 의존성 부분**
>
> [node_modules로부터 우리를 구원해 줄 Yarn Berry](https://toss.tech/article/node-modules-and-yarn-berry)

![모노레포 호이스팅](../../asset/monorepo-practice/monorepo-hoisting.png)

이게 프로젝트 루트 `node_modules` 에서 모든 모듈에 액세스할 수 있는 것처럼 보이지만 **로컬 프로젝트에서 각 패키지를 빌드하는 경우**가 많음

이때 일부 모듈 로더는 [심볼릭 링크](https://github.com/facebook/metro/issues/1)를 지원하지 않기도 하기에, `“module not found”` 오류를 발생시키기도 함

이때 [nohoist](https://classic.yarnpkg.com/blog/2018/02/15/nohoist/) 사용 가능

```json
{
  "workspaces": {
    "packages": ["packages/*"],
    "nohoist": ["**/react-native"]
  }
}
```

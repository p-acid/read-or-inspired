---
title: "Three.js 학습 일지"
created_at: "2022-06-27"
---

> 이전 학습 내용 스킵

# 220626 : `gltf` 파일 로드하고 애니메이션 추가

![빙글빙글 애니메이션](../../asset/three-js-practice/animate_react.gif)

```tsx
import { useGLTF } from "@react-three/drei";
import { useFrame } from "@react-three/fiber";
import { useRef } from "react";

const Gltf = () => {
  const mesh = useRef(null);
  const gltf = useGLTF("/shiba/scene.gltf");
  useFrame(() => (mesh.current.rotation.y = mesh.current.rotation.z += 0.1));
  return (
    <mesh ref={mesh}>
      <primitive object={gltf.scene} />
    </mesh>
  );
};

useGLTF.preload("/shiba/scene.gltf");

export default Gltf;
```

## `gltf` 확장자 파일 로드하기

- `useGLTF` 훅 사용
  - 인자로 `gltf` 파일 경로(`string)` 할당
  - 이를 `primitive` 태그에 `gltf` 객체의 `scene` 참조하여 할당
- `preload` 부분 예제에 있어서 작성하긴 했는데 왜 추가되었는지 모름

## 애니메이션 추가

- `useFrame` 훅과 `useRef` 훅 사용
  - `mesh` 태그 생성하여 앞의 `primitive` 태그 래핑
  - 해당 `mesh` 태그에 `ref` 값 할당
    - `ref` 초기값 `null` 할당
  - `useFrame` 훅을 통해 프레임당 추가될 애니메이션 작성

<br/>

# 220627 : `react-three-fiber` 공식 문서 분석

## `<Canvas>`

- 렌더링에 필요한 기본 빌딩 블록인 **`Scene` 과 `Camera` 를 설정한다.**
- 매 프레임마다 장면을 렌더링하므로 기존의 `render-roop` 가 필요하지 않다.

> Canvas는 부모 노드에 맞게 반응하므로 부모 너비와 높이를 변경하여 크기를 제어할 수 있다(이 경우 #canvas-container).

## `<mesh>`

장면 에서 실제로 무엇을 보기 위해 `<mesh />` 를 추가한다.

- 이는 `new THREE.Mesh()` 와 같다.

> 이때 **아무것도 `import` 할 필요가 없다.** 왜냐하면 리액트에서 **모든 `three.js` 객체는 기본 `JSX` 요소로 처리**되기 때문이다.
>
> 일반적으로 `Fiber` 컴포넌트는 **카멜 케이스**로 작성한다.

- `<mesh>` 는 `three.js` 의 기본 장면 객체이며 3D 공간에서 `shape` 을 나타내는데 필요한 `geometry` 와 `material` 을 유지하는데 사용된다.

아래 두 코드는 동일하다.

```jsx
// react-three-fiber

<Canvas>
  <mesh>
    <boxGeometry />
    <meshStandardMaterial />
  </mesh>
</Canvas>
```

```js
// three.js

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, width / height, 0.1, 1000);

const renderer = new THREE.WebGLRenderer();
renderer.setSize(width, height);
document.querySelector("#canvas-container").appendChild(renderer.domElement);

const mesh = new THREE.Mesh();
mesh.geometry = new THREE.BoxGeometry();
mesh.material = new THREE.MeshStandardMaterial();

scene.add(mesh);

function animate() {
  requestAnimationFrame(animate);
  renderer.render(scene, camera);
}

animate();
```

<br/>

## 생성자 인자

다음 [Three.js 공식 문서](https://threejs.org/docs/#api/en/geometries/BoxGeometry)에선 `width`, `height`, `depth` 에 대한 세 가지 인자를 선택적으로 전달할 수 있다.

```js
new THREE.BoxGeometry(2, 2, 2);
```

이와 같이 동작하기 위해 `Fiber` 에서는 `args` 라는 `prop` 으로 배열을 전달합니다.

```jsx
<boxGeometry args={[2, 2, 2]} />
```

> `args` 변경마다 객체를 다시 구성해야 합니다.

### 응용

![scale-control](../../asset/three-js-practice/scale_control.gif)

`state` 값을 `type="range"` 인 `input` 으로 핸들링하여 값 변경

<br/>

## 조명 추가하기

- 요소에 빛과 관련된 요소를 핸들링 할 수도 있습니다.

```jsx
<directionalLight color="blue" position={[5, 5, 5]} intensity={0.2} /> // 카메라 기준 정면 쪽 빛
<ambientLight color="green" position={[5, 5, 5]} intensity={0.2} /> // 카메라 기준 외곽 쪽 빛
```

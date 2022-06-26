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

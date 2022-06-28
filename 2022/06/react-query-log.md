---
title: "리액트 쿼리 로그"
created_at: "2022/06/28"
---

# 비동기 값이 업데이트 되고 쿼리문을 실행시킬 때

> 참고
> 
> [Dependent Queries : React Query ](https://react-query.tanstack.com/guides/dependent-queries)

- `options` 중 `enable` 필드 활용

```js
 const { isIdle, data: projects } = useQuery(
   ['projects', userId],
   getProjectsByUser,
   {
     // The query will not execute until the userId exists
     enabled: !!userId,
   }
 )
 ```

- 해석 : `userId` 가 온전한 값이 되었을 때, 쿼리문을 실행 가능하게 변경한다.

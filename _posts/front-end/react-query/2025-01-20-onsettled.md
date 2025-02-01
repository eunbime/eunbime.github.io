---
title: React Query의 onSuccess와 onSettled의 차이점과 활용 방법
date: 2025-01-20 10:00:00 +09:00
categories: [프론트엔드, React Query]
tags: [React Query]
---

## **1. `onSuccess`와 `onSettled`의 차이점**

### **🔹 `onSuccess`**

- **Mutation이 성공적으로 완료되었을 때만 실행된다.**
- **에러가 발생하면 실행되지 않는다.**
- 데이터가 서버에 저장된 직후 실행되므로, 성공적인 응답을 기반으로 UI를 업데이트할 때 사용된다.

```tsx
const { mutate } = useMutation({
  mutationFn: updatePost,
  onSuccess: () => {
    console.log("데이터 저장 성공!");
  },
  onError: () => {
    console.log("에러 발생!");
  }
});
```

### **🔹 `onSettled`**

- **Mutation의 성공/실패 여부와 관계없이 항상 실행된다.**
- **모든 관련 작업(캐시 업데이트, 데이터 무효화 등)이 완료된 후 실행된다.**
- 요청이 실패해도 실행되므로, UI 상태를 항상 업데이트해야 하는 경우 유용하다.

```tsx
const { mutate } = useMutation({
  mutationFn: updatePost,
  onSuccess: () => {
    console.log("데이터 저장 성공!");
  },
  onSettled: () => {
    console.log("모든 작업 완료!");
  },
  onError: () => {
    console.log("에러 발생!");
  }
});
```

---

## **2. `onSuccess` vs `onSettled`: 언제 사용해야 할까?**

| 상황                                                  | `onSuccess`              | `onSettled`    |
| ----------------------------------------------------- | ------------------------ | -------------- |
| 성공적으로 데이터를 업데이트한 후 UI를 갱신해야 할 때 | ✅ 사용 가능             | ✅ 사용 가능   |
| 에러가 발생해도 항상 실행되어야 할 작업이 있을 때     | ❌ 실행되지 않음         | ✅ 항상 실행됨 |
| 요청 완료 후 UI 상태를 업데이트해야 할 때             | ❌ 실패 시 실행되지 않음 | ✅ 항상 실행됨 |
| 데이터 캐싱을 무효화해야 할 때                        | ✅ 사용 가능             | ✅ 사용 가능   |

실무에서는 다음과 같이 사용한다:

- **데이터 업데이트 후 UI 변경** → `onSuccess`
- **성공/실패 여부와 관계없이 후처리 필요** → `onSettled`
- **로딩 상태를 관리하면서 Mutation 처리** → `onSettled`

---

## **3. `onSettled`을 활용한 안전한 상태 관리**

다음은 `onSettled`을 활용하여 안전하게 모달을 닫고 상태를 갱신하는 코드이다.

```tsx
const PostUploadModal = () => {
  const [isSettling, setIsSettling] = useState(false);

  const onSubmit = async (values: z.infer<typeof PostUploadSchema>) => {
    try {
      setIsSettling(true);

      if (postData?.post || postData?.formData) {
        await editPost(
          { values, postId: postData?.post?.id as string },
          {
            onSettled: async () => {
              await queryClient.invalidateQueries(); // 데이터 캐싱 무효화
              setIsSettling(false);
              closeModal(); // 모달 닫기
            }
          }
        );
      } else {
        await uploadPost(values, {
          onSettled: async () => {
            await queryClient.invalidateQueries();
            setIsSettling(false);
            closeModal();
          }
        });
      }
    } catch (error) {
      setIsSettling(false);
      console.log("[POST_SUBMIT_ERROR]", error);
    }
  };

  return (
    <Button
      type="submit"
      disabled={isUploadPending || isEditPending || isSettling}
    >
      {isUploadPending || isEditPending || isSettling
        ? "Loading..."
        : postData?.post || postData?.formData
        ? "Edit"
        : "Upload"}
    </Button>
  );
};
```

### **🔹 이 코드의 주요 포인트**

1. **`setIsSettling(true)`**: Mutation이 시작되면 로딩 상태를 활성화한다.
2. **`await queryClient.invalidateQueries()`**: Mutation 후 **최신 데이터를 다시 불러온다.**
3. **`setIsSettling(false)`**: Mutation이 완료된 후 로딩 상태를 해제한다.
4. **`closeModal()`**: 작업이 완료된 후 모달을 닫는다.

---

## **4. 최적의 Mutation 처리 방식**

`onSuccess`와 `onSettled`을 적절히 조합하면, UI 상태를 안전하게 관리할 수 있다.

### ✅ **데이터가 변경된 후 특정 작업을 실행해야 한다면?**

```tsx
onSuccess: () => {
  console.log("업데이트 성공!");
  queryClient.invalidateQueries(["posts"]); // 관련 데이터 갱신
};
```

### ✅ **성공/실패 여부와 관계없이 항상 실행해야 하는 작업이 있다면?**

```tsx
onSettled: () => {
  console.log("작업 완료!");
  closeModal(); // 모달 닫기
};
```

### ✅ **실패했을 때만 특정 작업을 실행해야 한다면?**

```tsx
onError: (error) => {
  console.error("에러 발생:", error);
};
```

---

## **5. 결론**

✅ `onSuccess`는 **mutation이 성공했을 때만 실행되므로** 성공적인 응답을 기반으로 UI를 업데이트할 때 적합하다.

✅ `onSettled`는 **mutation이 성공하든 실패하든 항상 실행되므로** UI 상태를 항상 업데이트해야 할 때 적합하다.

✅ `onError`는 실패 시 특정 작업을 실행하는데 유용하다.

실무에서는 이 세 가지 콜백을 조합하여 **안전한 UI 상태 관리**와 **데이터 동기화**를 최적화할 수 있다. 🚀

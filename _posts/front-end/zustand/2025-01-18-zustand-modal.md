---
title: Zustand를 활용한 모달 관리 구현
date: 2025-01-18 10:00:00 +09:00
categories: [프론트엔드, Zustand]
tags: [Zustand, Modal]
---

이 글에서는 **Next.js의 App Router**와 **Zustand 상태 관리 라이브러리**를 사용하여 모달을 구현하고 관리하는 방법을 설명한다.

---

### 1. **Provider 생성 및 설정**

모달 컴포넌트를 전역적으로 관리하기 위해 `ModalProvider`를 작성한다. 이 Provider는 모든 모달을 포함하며, 전역 상태를 통해 어떤 모달을 표시할지 제어한다.

### 코드 예제

```tsx
//@/components/providers/modal-provider.tsx

"use client";

import { useEffect, useState } from "react";
import PostUploadModal from "@/components/modals/post-upload-modal";
import PostViewModal from "@/components/modals/post-view-modal";
import ConfirmModal from "@/components/modals/confirm-modal";

export default function ModalProvider() {
  const [isMounted, setIsMounted] = useState(false);

  // 클라이언트 측에서만 렌더링 되도록 함 (Hydration 오류 방지)
  useEffect(() => {
    setIsMounted(true);
  }, []);

  if (!isMounted) return null;

  return (
    <>
      {/* 모든 모달 컴포넌트를 전역적으로 렌더링 */}
      <PostUploadModal />
      <PostViewModal />
      <ConfirmModal />
    </>
  );
}
```

### Provider를 Layout에 추가

`ModalProvider`를 최상위 Layout에 추가하여 모든 페이지에서 모달이 사용 가능하도록 설정한다.

```tsx
// /app/layout.tsx
import ModalProvider from "@/components/providers/modal-provider";

export default async function RootLayout({
  children
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="en" suppressHydrationWarning>
      <body className="antialiased">
        {/* ModalProvider를 Layout에 추가 */}
        <ModalProvider />
        <div className="w-full h-full bg-white dark:bg-black text-black dark:text-white">
          {children}
        </div>
      </body>
    </html>
  );
}
```

---

### 2. **모달 상태 관리 - Zustand**

모달의 상태를 관리하기 위해 `Zustand`를 사용한다. 각 모달의 열림/닫힘 상태, 모달 타입, 그리고 전달받는 데이터를 관리한다.

### 코드 예제

```tsx
//@/store/modal/modal-store.ts
import { create } from "zustand";
import { PostUploadSchema } from "@/schemas";
import { Post } from "@prisma/client";
import { z } from "zod";

// 모달 타입 정의
export type ModalType =
  | "post-upload"
  | "post-view"
  | "profile"
  | "timeline"
  | "delete-confirm"
  | "edit-confirm";

// 모달에 전달할 데이터
export type ModalData = null | {
  post?: Post | null;
  onConfirm?: () => void;
  title?: string;
  description?: string;
  formData?: z.infer<typeof PostUploadSchema> | null;
};

// Zustand 상태 관리
interface ModalState {
  type: ModalType | null;
  setType: (type: ModalType) => void;
  isOpen: boolean;
  openModal: () => void;
  closeModal: () => void;
  data: ModalData;
  setData: (data: ModalData) => void;
}

const useModal = create<ModalState>((set) => ({
  type: null,
  setType: (type) => set({ type }),
  isOpen: false,
  openModal: () => set({ isOpen: true }),
  closeModal: () =>
    set({
      type: null,
      isOpen: false,
      data: null
    }),
  data: null,
  setData: (data) => set({ data })
}));

export default useModal;
```

### 주요 동작

- **`type`**: 현재 활성화된 모달의 유형을 관리한다.
- **`isOpen`**: 모달의 열림 상태를 제어한다.
- **`data`**: 모달에 전달할 데이터를 저장한다.

---

### 3. **모달 컴포넌트 구현**

모달 컴포넌트는 Zustand 상태를 구독하여 모달 열림/닫힘 상태와 데이터를 기반으로 렌더링된다.

### 코드 예제

```tsx
//@/components/modals/post-upload-modal.tsx
"use client";

import useModal from "@/store/modal/modal-store";
import { formatDateForTimeline } from "@/lib/formatDate";
import Image from "next/image";
import { Button } from "../ui/button";
import { useScrollLock } from "@/hooks/use-scroll-lock";
import { useMutation, useQueryClient } from "@tanstack/react-query";
import axios from "axios";
import useUser from "@/store/user/user-store";

const PostUploadModal = () => {
  const { user } = useUser();

  // zustand에 저장된 modal state를 가져와서 사용
  const { isOpen, closeModal, type, data: postData, setType } = useModal();

  const queryClient = useQueryClient();

  // 포스트 삭제 요청
  const { mutate: deletePost, isPending: isDeletePending } = useMutation({
    mutationFn: async () => {
      await axios.delete(`/api/posts/${postData?.post?.id}`);
    },
    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["posts"] });
    }
  });

  // 스크롤 락 처리
  useScrollLock(isOpen);

  // isOpen이 true이고 type이 post-view일 때만 모달이 열리도록 함
  if (!isOpen || type !== "post-view") return null;

  // 모달 타입을 post-upload로 변경
  const handleEdit = () => setType("post-upload");

  const handleDelete = () => {
    deletePost();
    closeModal(); // 모달 닫기
  };

  return (
    <div className="fixed inset-0 z-50 bg-black/50 flex items-center justify-center">
      <div className="w-[400px] h-[800px] bg-white rounded-md">
        <div className="flex justify-between items-center p-4 border-b">
          <h1 className="text-2xl font-bold">
            {postData?.post?.date
              ? formatDateForTimeline(new Date(postData.post.date))
              : ""}
          </h1>
          <button onClick={closeModal} className="text-2xl font-bold">
            X
          </button>
        </div>
        <div className="flex flex-col items-center gap-4 p-4">
          <Image
            src={postData?.post?.imageUrl || "https://via.placeholder.com/350"}
            alt="Post Image"
            width={350}
            height={350}
            className="object-cover"
          />
          <p className="text-md font-bold">{postData?.post?.title}</p>
          <p className="text-sm text-gray-500">{postData?.post?.content}</p>
          {user?.id === postData?.post?.authorId && (
            <div className="flex gap-2">
              <Button onClick={handleEdit}>Edit</Button>
              <Button onClick={handleDelete} disabled={isDeletePending}>
                Delete
              </Button>
            </div>
          )}
        </div>
      </div>
    </div>
  );
};

export default PostUploadModal;
```

![스크린샷 2025-01-18 오후 2.20.31.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e8142744-55a6-4844-a830-3be10006c0af/871ac429-e060-4c8b-af8e-c28dc6c75f9f/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2025-01-18_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.20.31.png)

![스크린샷 2025-01-18 오후 2.21.20.png](https://prod-files-secure.s3.us-west-2.amazonaws.com/e8142744-55a6-4844-a830-3be10006c0af/d8917943-45d4-43d9-a7ae-fb15025454b7/%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2025-01-18_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_2.21.20.png)

---

### 주요 기능 요약

1. **`ModalProvider`**
   - 모달의 컨테이너 역할.
   - 모든 모달을 전역에서 관리.
2. **Zustand로 상태 관리**
   - 모달 타입, 데이터, 열림/닫힘 상태를 효율적으로 관리.
3. **모달 컴포넌트**
   - Zustand의 상태를 구독하여 필요한 UI를 렌더링.
   - 데이터 패칭 및 삭제 요청과 같은 기능 포함.

---

이 구현은 **전역 모달 관리**가 필요한 프로젝트에서 유용하며, 상태 관리와 UI 처리를 분리하여 코드 가독성을 높인다. Next.js와 Zustand의 조합으로 간결하면서도 강력한 모달 시스템을 구축할 수 있다.

---
title: 提取到 Memoized 组件
impact: MEDIUM
impactDescription: 启用提前返回
tags: rerender, memo, useMemo, optimization
---

## 提取到 Memoized 组件

将昂贵的工作提取到 memoized 组件中，以便在计算之前启用提前返回。

**错误示例（即使在 loading 时也计算头像）：**

```tsx
function Profile({ user, loading }: Props) {
  const avatar = useMemo(() => {
    const id = computeAvatarId(user);
    return <Avatar id={id} />;
  }, [user]);

  if (loading) return <Skeleton />;
  return <div>{avatar}</div>;
}
```

**正确示例（loading 时跳过计算）：**

```tsx
const UserAvatar = memo(function UserAvatar({ user }: { user: User }) {
  const id = useMemo(() => computeAvatarId(user), [user]);
  return <Avatar id={id} />;
});

function Profile({ user, loading }: Props) {
  if (loading) return <Skeleton />;
  return (
    <div>
      <UserAvatar user={user} />
    </div>
  );
}
```

**注意：** 如果你的项目启用了 [React Compiler](https://react.dev/learn/react-compiler)，则不需要使用 `memo()` 和 `useMemo()` 进行手动记忆。编译器会自动优化重新渲染。

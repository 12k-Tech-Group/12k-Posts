---
title: 关于随机数的一些讨论
date: 2020-08-31 19:50
author: mq
---

# 关于随机数的一些讨论

## `rand()` 真的够用了

linux下rand是线程安全的

## `randint(min, max)` 错误实现导致溢出UB可能造成一些无法预料的结果

* 错误姿势❌:
```
range = max-min // 溢出UB
return rand() % range + min // 模可能不均匀
```

* 正确姿势✔
```
umin = 移码(min)
umax = 移码(max)
range = umax - umin
// 这里就不说如何解决区间映射的问题了
```
其中`移码`, 是把区间`[INT_MIN, INT_MAX]`映射到`[0, UINT_MAX]`

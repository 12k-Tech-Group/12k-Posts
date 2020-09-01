---
title: unique_ptr的缺陷
date: 2020-09-01 12:30
author: mq
---

# unique_ptr的缺陷

## unique_ptr 不能被优化为裸指针

* 尽管`sizeof(unique_ptr<T, default_delete<T>>) == sizeof(void*)`, ABI 限制了 unique_ptr 不能被寄存器传递, 而是传递 unique_ptr 的地址作为参数

```c++
#include <memory>

void func(std::unique_ptr<int>& a) {
    *a = 2;
}
```
- MSVC汇编输出

    ```asm
    a$ = 8
    void func(std::unique_ptr<int,std::default_delete<int> > &) PROC ; func, COMDAT
            mov     rax, QWORD PTR [rcx]
            mov     DWORD PTR [rax], 2
            ret     0
    void func(std::unique_ptr<int,std::default_delete<int> > &) ENDP ; func
    ```

- clang汇编输出
    ```assembly
    func(std::unique_ptr<int, std::default_delete<int> >&): # @func(std::unique_ptr<int, std::default_delete<int> >&)
            mov     rax, qword ptr [rdi]
            mov     dword ptr [rax], 2
            ret
    ```

均可以观察到两次内存地址mov

而裸指针
```c++
void func(int* a) {
    *a = 2;
}
```

- MSVC
    ```asm
    a$ = 8
    void func(int *) PROC                             ; func, COMDAT
            mov     DWORD PTR [rcx], 2
            ret     0
    void func(int *) ENDP                             ; func
    ```

- clang

    ```asm
    func(int*):                              # @func(int*)
            mov     dword ptr [rdi], 2
            ret
    ```

## unique_ptr 不是 trivially movable

当`vector<unique_ptr<T>>`扩容时，vector调用了for循环依此move原buffer中的对象到新buffer，而不是memcpy

当然这也怪C++没有destructive move语义，如果有的话，unique_ptr应当是"trivially destructive movable"

`reference_wrapper<T>`在此处与`unique_ptr<T>`有相同的问题

## 当API需要传递`T*[]`(裸指针的数组)时，`vector<unique_ptr<T>>` 就sb了

vector的data()是`unique_ptr<T>*`，不是`T**`，除非你`reinterpret_cast`

*（不过话说回来，non-strict-aliasing也是ABI的一部分）*

## non-trivial的dtor可能会影响尾调用优化

众所周知，有non-trivial析构对象的函数的尾调用不是尾调用

*（吱又要喷我了，我不懂PTC，哪位巨巨给补充一下）*

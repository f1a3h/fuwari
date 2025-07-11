---
title: "王爽「汇编语言」学习笔记（六）"
published: 2023-09-08
description: 'Chapter.10 ~ Chapter.11'
image: ''
tags: [Assembly]
category: 'Study Notes'
draft: false 
lang: ''
---

## call 和 ret 指令

### ret 和 retf

- ret 实现近转移
  - `pop IP`
- retf 实现远转移
  - `pop IP`
  - `pop CS`

### call 指令

- 不能实现短转移

### 转移的目的地址在指令中的 call 指令

- `call far ptr` 实现段间转移
  - `push CS`
  - `push IP`

### 转移地址在内存中的 call 指令

- `call word ptr 内存单元地址`
  - `push IP`
  - `jmp word ptr 内存单元地址`
- `call dword ptr 内存单元地址`
  - `push CS`
  - `push IP`

### mul 指令

- 乘数
  - 都是 8 位
    - AL
    - 8 位寄存器或内存字节单元
  - 都是 16 位
    - AX
    - 16 位寄存器或内存字单元
- 结果
  - 8 位：AX
  - 16 位：高位 DX ，低位 AX

## Lab 10

### 显示字符串

```asm
assume cs:code

data segment
    db 'Welcome to masm!', 0
data ends

code segment
start:  mov dh, 8
        mov dl, 3
        mov cl, 2
        mov ax, data
        mov ds, ax
        mov si, 0
        call show_str

        mov ax, 4c00h
        int 21h

show_str: ; 使用 es:[bp] 载入显存
        mov ax, 0b800h
        mov es, ax

        mov al, 160
        mul dh
        mov dh, 0
        add ax, dx
        add ax, dx
        mov bp, ax

copy: ; 将字符串复制入显存
        mov al, cl
        mov ch, 0
        mov cl, ds:[si]
        jcxz finish
        
        mov es:[bp], cl
        mov cl, al
        mov es:[bp+1], cl
        add bp, 2
        inc si
        jmp short copy

finish: ; 结束
        ret
code ends
end start
```

### 解决除法溢出的问题

```asm
assume cs:code

stack segment
    dw 6 dup (0)
stack ends

code segment
start:
    mov ax, stack
    mov ss, ax
    mov sp, 12

    mov ax, 4240H
    mov dx, 00FH
    mov cx, 0AH
    call divdw

    mov ax, 4c00h
    int 21h

divdw:
    mov bx, ax
    mov ax, dx
    mov dx, 0
    div cx
    push ax

    mov ax, bx
    div cx

    mov cx, dx
    pop dx

    ret
code ends
end start
```

### 数值显示

```asm
assume cs:code

data segment
    db 'Welcome to masm!', 0
data ends

code segment
start:  mov dh, 8
        mov dl, 3
        mov cl, 2
        mov ax, data
        mov ds, ax
        mov si, 0
        call show_str

        mov ax, 4c00h
        int 21h

show_str: ; 使用 es:[bp] 载入显存
        mov ax, 0b800h
        mov es, ax

        mov al, 160
        mul dh
        mov dh, 0
        add ax, dx
        add ax, dx
        mov bp, ax

copy: ; 将字符串复制入显存
        mov al, cl
        mov ch, 0
        mov cl, ds:[si]
        jcxz finish
        
        mov es:[bp], cl
        mov cl, al
        mov es:[bp+1], cl
        add bp, 2
        inc si
        jmp short copy

finish: ; 结束
        ret
code ends
end start
```

## 标志寄存器

标志寄存器有 16 位，其中存储的信息被称为程序状态字 (PSW)

影响标志寄存器的大都是运算指令，不影响的大都是传送指令

`inc` 和 `loop` 指令不影响 CF 位

- ZF：零标志位
- PF：奇偶标志位
- SF：符号标志位，即记录结果最高有效位是否为 1
- CF：进位标志位，针对**无符号数运算**，记录结果的**最高有效位**向更高位的进位或借位情况
- OF：溢出标志位，针对**有符号数运算**

### cmp 指令

对于无符号数运算 `cmp ax, bx` :

- ZF = 1，则 (ax) = (bx)
- ZF = 0，则 (ax) != (bx)
- CF = 1，则 (ax) < (bx)
- CF = 0，则 (ax) >= (bx)
- CF = 0 且 ZF = 0，则 (ax) > (bx)
- CF = 1 或 ZF = 1，则 (ax) <= (bx)

对于有符号数运算 `cmp ah, bh` :

- SF = 1 且 OF = 0，则 (ah) < (bh)
- SF = 1 且 OF = 1，则 (ah) > (bh)
- SF = 0 且 OF = 0，则 (ah) < (bh)
- SF = 0 且 OF = 1，则 (ah) >= (bh)

### 检测比较结果的条件转移指令

它们所检测的标志位都是 cmp 指令进行**无符号数**比较时的

### DF 标志和串传送指令

- DF：方向标志位，在串处理指令中，控制每次操作后 si, di 的增减
  - DF = 0, 递增
  - DF = 1, 递减
  - cld：令 DF = 0
  - std：令 DF = 1
- movsb
  - 相当于 `(es:[di]) = (ds:[si])`
  - 之后根据 DF 改变 si, di
- movsw
  - movsb 的字版本
  - si 和 di 改变量为 2
- rep
  - 根据 cx 的值重复执行后面的串传送指令

### pushf 和 popf

- pushf：将标示寄存器的值压栈
- popf：从栈中弹出数据送入标志寄存器

## Lab 11

```asm
assume cs:codesg

datasg segment
    db "Beginner's All-purpose Symbolic Instruction Code.", 0
datasg ends

codesg segment
begin:
    mov ax, datasg
    mov ds, ax
    mov si, 0
    call letterc

    mov ax, 4c00h
    int 21h

letterc:
    mov cx, 0
    mov cl, ds:[si]
    jcxz finish
    cmp cl, 97
    jb next
    cmp cl, 122
    ja next
    and cl, 11011111B
    mov ds:[si], cl

next:
    inc si
    jmp short letterc

finish:
    ret
codesg ends
end begin
```
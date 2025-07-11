---
title: "王爽「汇编语言」学习笔记（四）"
published: 2023-09-01
description: '旧文搬运'
image: ''
tags: [Assembly]
category: 'Study Notes'
draft: false 
lang: ''
---

## 可执行文件

可执行文件 = 描述信息 + 程序

程序：源程序中的汇编指令和定义的数据

描述信息：主要是 `masm.exe`、`link.exe` 对源程序中相关伪指令进行处理所得到的信息

`end start` 指明程序入口，被转化为一个入口地址，存储在可执行文件的描述信息中

## 段容量

一个段段容量不能大于 64KB ，这是受 8086CPU 的限制

## Lab 5: 编写、调试具有多个段的程序

### 任务一

1. 0123H, 0456H, 0789H, 0abch, 0defh, 0fedh, 0cbah, 0987H
2. cs = 076C, ss = 076B, ds = 076A
3. data: X - 2, stack: X - 1

### 任务二

1. 0123H, 0456H
2. cs = 076C, ss = 076B, ds = 076A
3. data: X - 2, stack: X - 1
4. 16 的倍数个字节且大于等于 N

### 任务三

1. 0123H, 0456H
2. cs = 076A, ss = 076E, ds = 0766D
3. data: X + 3, stack: X + 4

### 任务四

第 3 题，因为没有入口地址的话 IP 默认为 0 ，前两题从 0 到后面一段的内容都是数据内容而不是正确的指令内容，无法被正常执行

### 任务五

```asm
assume cs:code

a segment
	db 1, 2, 3, 4, 5, 6, 7, 8
a ends

b segment
	db 1, 2, 3, 4, 5, 6, 7, 8
b ends

c segment
	db 0, 0, 0, 0, 0, 0, 0, 0
c ends

code segment
start:	mov ax, a
		mov ds, ax
		mov bx, 0
		mov cx, 8
	s:	mov al, ds:[bx]
		add al, ds:[bx+16]
		mov ds:[bx+32], al
		inc bx
		loop s

		mov ax, 4c00h
		int 21h
code ends
end start
```

### 任务六

```asm
assume cs:code

a segment
	dw 1, 2, 3, 4, 5, 6, 7, 8
a ends

b segment
	dw 0, 0, 0, 0, 0, 0, 0, 0
b ends

code segment
start:	mov ax, a
		mov ss, ax
		mov sp, 32
		mov bx, 0
		mov cx, 8
	s:	push ss:[bx]
		add bx, 2
		loop s

		mov ax, 4c00h
		int 21h
code ends
end start
```

## Lab 6: 实践课程中的程序

### 任务一

略

### 任务二

```asm
assume cs:codesg, ss:stacksg, ds:datasg

stacksg segment
	dw 0, 0, 0, 0, 0, 0, 0, 0
stacksg ends

datasg segment
	db '1. display      '
	db '2. brows        '
	db '3. replace      '
	db '4. modify       '
datasg ends

codesg segment
start:	mov ax, stacksg
		mov ss, ax
		mov sp, 16
		mov ax, datasg
		mov ds, ax
		mov bx, 0

		mov cx, 4
	s0:	push cx
		mov si, 0

		mov cx, 4
	s:	mov al, [bx+si+3]
		and al, 11011111b
		mov [bx+si+3], al
		inc si
		loop s

		pop cx
		add bx, 16
		loop s0

		mov ax, 4c00H
		int 21H
codesg ends
end start
```
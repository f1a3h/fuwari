---
title: "王爽「汇编语言」学习笔记（七）"
published: 2023-09-14
description: 'Chapter.12 ~ Chapter.15'
image: ''
tags: [Assembly]
category: 'Study Notes'
draft: false 
lang: ''
---

## 內中断

中断类型码为 1 byte 的数据，可以表示 256 种中断信息的来源，但是实际上装载好的远少于 256 种。

在 8086CPU 中，中断向量表存放在 `0000:0000~0000:03E8` 单元中，一个表项占 2 word ，偏移地址 word 存在低地址，段地址 word 存在高地址中。

中断过程如下：

1. 取得中断类型码 N
2. `pushf`
3. `TF = 0, IF = 0`
4. `push CS`
5. `push IP`
6. `IP = (N * 4), CS = (N * 4 + 2)`

在编写中断处理程序时，需在结尾使用 `iret` 指令返回，作用与 `ret` 相似。

CPU 在执行完一条指令之后，如果检测到 `TF = 1` ，就会产生类型码为 1 的单步中断，转去执行 1 号中断处理程序，从而为单步跟踪程序的执行提供了可能。

但是为了避免出现 1 号中断程序的套娃现象，需要在每次进入中断程序之前需要将 TF 置为 0 。

**特殊情况**：在执行完 ss 寄存器传送数据的指令后，即便发生中断，CPU 也不会响应。
-> 我们应该将设置 ss 和 sp 的指令连续存放。

## Lab 12

编写 0 号中断的处理程序

```asm
assume cs:code

code segment
start:
	mov ax, cs
	mov ds, ax
	mov si, offset do0
	mov ax, 0
	mov es, ax
	mov di, 200h
	mov cx, offset do0end - offset do0
	cld
	rep movsb

	mov ax, 0
	mov es, ax
	mov word ptr es:[0], 200h
	mov word ptr es:[2], 0

	mov ax, 4200h
	int 21h

do0:
	jmp short do0start
	db "divide error!"

do0start:
	mov ax, cs
	mov ds, ax
	mov si, 202h

	mov ax, 0b800h
	mov es, ax
	mov di, 12*160+36*2

	mov cx, 13
s:
	mov al, [si]
	mov es:[di], al
	inc si
	add di, 2
	loop s

	mov ax, 4200h
	int 21h
do0end:
	nop
code ends
end start
```

## Lab 13

### 显示用 0 结束的字符串

```asm
assume cs:code

data segment
	db "welcome to masm!", 0
data ends

stack segment
	dw 8 dup (0)
stack ends

code segment
start:
	mov ax, stack
	mov ss, ax
	mov sp, 16

	mov ax, cs
	mov ds, ax
	mov si, offset showstr
	mov ax, 0
	mov es, ax
	mov di, 200h
	mov cx, offset showstrend - offset showstr
	cld
	rep movsb

	mov ax, 0
	mov es, ax
	mov word ptr es:[7ch*4], 200h
	mov word ptr es:[7ch*4+2], 0

	mov dh, 10
	mov dl, 10
	mov cl, 2
	mov ax, data
	mov ds, ax
	mov si, 0
	int 7ch
	
	mov ax, 4c00h
	int 21h

showstr:
	push ax
	push bx
	push cx
	push ds
	push es
	push di
	push si

	mov ax, 0b800h
	mov es, ax
	mov ah, 0
	mov al, 160
	mul dh
	mov dh, 0
	add ax, dx
	mov di, ax

	mov al, cl
	mov cx, 0

s:
	mov cl, ds:[si]
	jcxz se
	mov es:[di], cl
	mov es:[di+1], al
	inc si
	add di, 2
	loop s

se:
	pop si
	pop di
	pop es
	pop ds
	pop cx
	pop bx
	pop ax
	iret

showstrend:
	nop
code ends
end start
```

### 完成 loop 指令的功能

```asm
assume cs:code

stack segment
	dw 8 dup (0)
stack ends

code segment
start:
	mov ax, stack
	mov ss, ax
	mov sp, 16

	call load
	call setIVT

	mov ax, 0b800h
	mov es, ax
	mov di, 160*12
	mov bx, offset s - offset se
	mov cx, 80

s:
	mov byte ptr es:[di], '!'
	add di, 2
	int 7ch

se:
	nop
	mov ax, 4c00h
	int 21h

setIVT:
	push ax
	push es

	mov ax, 0
	mov es, ax
	mov word ptr es:[7ch*4], 200h
	mov word ptr es:[7ch*4+2], 0

	pop es
	pop ax
	ret

load:
	push ax
	push cx
	push ds
	push es
	push di
	push si

	mov ax, 0
	mov es, ax
	mov di, 200h
	mov ax, cs
	mov ds, ax
	mov si, offset myloop
	mov cx, offset myloopend - offset myloop
	cld
	rep movsb

	pop si
	pop di
	pop es
	pop ds
	pop cx
	pop ax
	ret

myloop:
	push bp
	mov bp, sp
	dec cx
	jcxz myloopret
	add [bp+2], bx

myloopret:
	pop bp
	iret

myloopend:
	nop
code ends
end start
```

### 显示四句英文诗

```asm
assume cs:code

code segment
s1: db 'Good,better,best,', '$'
s2: db 'Never let it rest,', '$'
s3: db 'Till good is better,', '$'
s4: db 'And better,best.', '$'
s:  dw offset s1, offset s2, offset s3, offset s4
row:db 2, 4, 6, 8

start:
	mov ax, cs
	mov ds, ax
	mov bx, offset s
	mov si, offset row
	mov cx, 4

ok:
	mov bh, 0
	mov dh, ds:[si]
	mov dl, 0
	mov ah, 2
	int 10h

	mov dx, ds:[bx]
	mov ah, 9
	int 21h
	inc si
	add bx, 2
	loop ok

	mov ax, 4200h
	int 21h
code ends
end start
```

## 端口

CPU 通过主线相连的芯片除了各种存储器之外，还有以下 3 种芯片：

1. 各种接口卡的芯片
2. 主板上的接口芯片，用来访问外设
3. 其他芯片，存储系统相关信息或进行相关输入输出

### in 和 out 指令

CPU 使用地址线和控制线向端口发送指令，从而与芯片进行交互，间接访问芯片的存储单元。

在 `in` 和 `out` 指令中，只能使用 ax 或 al 来存放端口读/写的数据，其中访问 8 位端口时用 al ，访问 16 位端口时用 ax 。

### shl 和 shr 指令

它们都是逻辑位移指令，都会将**最后移出的一位写入 CF 中**，如果移动位数大于 1 ，则需要将移动位数放在 cl 中。

## Lab 14

访问 CMOS RAM

```asm
assume cs:code

stack segment
	db 128 dup (0)
stack ends

code segment
start:
	mov ax, stack
	mov ss, ax
	mov sp, 128

	mov bx, 0b800h
	mov es, bx
	mov bx, 160*12+28*2

	call show_ymd
	mov ah, ' '
	call print_char
	call show_hms

	mov ax, 4c00h
	int 21h

show_ymd:
	push ax
	push cx

	mov al, 9
	call show_single

	mov ah, '/'
	call print_char

	mov al, 8
	call show_single

	mov ah, '/'
	call print_char
	
	mov al, 7
	call show_single

	pop cx
	pop ax
	ret

show_hms:
	push ax
	push cx

	mov al, 4
	call show_single

	mov ah, ':'
	call print_char

	mov al, 2
	call show_single

	mov ah, ':'
	call print_char
	
	mov al, 0
	call show_single

	pop cx
	pop ax
	ret

; al 存端口号
show_single:
	call get_char
	call print_char
	mov ah, al
	call print_char

	ret

; ah 存要打印的字符
; es:[bx] 要打印的屏幕位置
print_char:
	mov byte ptr es:[bx], ah
	add bx, 2

	ret

; al 存放单元
; ah 存十位数 ASCII 码
; al 存个位数 ASCII 码
get_char:
	push cx

	out 70h, al
	in al, 71h
	mov ah, al
	mov cl, 4
	shr ah, cl
	and al, 00001111b

	add ah, 30h
	add al, 30h
	
	pop cx
	ret
code ends
end start
```

## 外中断

外中断信息来自 CPU 外部，外中断源一共有两种：

1. 可屏蔽中断
   - 如果 `IF = 0` ，则会禁止进入可屏蔽中断。
   - 进入内中断时会将 IF 置为 0 。
   - 如果在內中断过程中需要处理可屏蔽中断，则需要用指令将 IF 置为 1 。
     - `sti` : IF = 1
     - `cli` : IF = 0
2. 不可屏蔽中断
   - 中断类型码固定为 2
   - 中断过程与內中断一致，除了最后是 `(IP) = (8), (CS) = (0AH)`

## Lab 15

安装新的 int9 中断例程

```asm
assume cs:code

stack segment
	db 128 dup (0)
stack ends

code segment
start:
	mov ax, stack
	mov ss, ax
	mov sp, 128

	push cs
	pop ds
	mov si, offset int9

	mov ax, 0
	mov es, ax
	mov di, 204h

	mov cx, offset int9end - offset int9
	cld
	rep movsb

	push es:[9*4]
	pop es:[200h]
	push es:[9*4+2]
	pop es:[202h]

	cli
	mov word ptr es:[9*4], 204h
	mov word ptr es:[9*4+2], 0
	sti

	mov ax, 4c00h
	int 21h

int9:
	push ax
	push bx
	push cx
	push es

	in al, 60h
	pushf
	call dword ptr cs:[200h]
	cmp al, 1eh
	jne int9ret
	pushf

	in al, 60h
	pushf
	call dword ptr cs:[200h]
	cmp al, 1eh+80h
	jne int9ret

	mov ax, 0b800h
	mov es, ax
	mov bx, 0
	mov cx, 2000
s:
	mov byte ptr es:[bx], 'A'
	add bx, 2
	loop s

int9ret:
	pop es
	pop cx
	pop bx
	pop ax
	iret

int9end:
	nop
code ends
end start
```

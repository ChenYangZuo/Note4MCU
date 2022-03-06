# 4.1 汇编语言的概述

## 4.1.1 汇编语言的特点

## 4.1.2 汇编语言的伪指令

### 1.ORG

确定下一条指令的地址

### 2.DB

保存字节常数或字符或表达式

```assembly
DB 73H,01H,90H
DB "Hello!"
```

### 3.DW

保存字数据（16位）

```assembly
DB 1234H,90H
```

### 4.EQU

赋值标号

```assembly
AREA EQU 1000H
STK EQU AREA
```

### 5.END

源程序结束

### 6.DATA

指定的数据地址赋予规定的字符名称

### 7.DS

保留指定表达式的若干字节空间

```assembly
ORG 1000H
DS 07H
DB 20H,25H,33H,46H			;1007H为20H
```

### 8.BIT

将位地址赋予字符名

```assembly
FLG BIT 20H					;标志位FLG定义在位地址20H
AI BIT P1.0					;把P1.0的位地址赋值给AI
```



# 4.2 汇编语言源程序的编辑和汇编

## 4.2.1 手工编程和汇编

## 4.2.2 机器编辑和交叉汇编



# 4.3 汇编语言程序设计

## 4.3.1 简单程序设计

### 1.BCD转十进制

片内RAM内35H单位中BCD码转换至片内RAM中40H单元中

```assembly
        ORG 0800H		;代码所在位置
        BCD EQU 35H		;数据源地址
        BIN EQU 40H		;数据目标地址
START:
        MOV A , BCD		;BCD内容赋值A
        ANL A , #F0H	;取高四位
        SWAP A			;交换高低四位
        MOV B , #0AH	;B=10
        MUL AB			;10*原本高位，得16位乘积，高位在A
        MOV R0 , A		;R0=A
        MOV A , BCD		;BCD内容赋值A
        ANL A , #0FH	;取低四位
        ADD A , R0		;A=A+R0
        MOV BIN , A		;BIN地址赋值A
        SJMP $			;跳转至当前地址（死循环）
        END
```

### 2.BCD转ASCII

片内RAM内20H单位中BCD码转换至片内RAM中21H、22H单元中

```assembly
		ORG 1000H
		BCD EQU 20H
		ASC EQU 21H
START:
		MOV A , BCD
		ANL A #0FH
		ADD A #30H
		MOV ASC , A
		MOV A , BCD
		ANL A #F0H
		SWAP A
		ADD A #30H
		MOV ASC+1 , A
		SJMP $
		END
```

## 4.3.2 分支程序设计

### 1.单/双分支

在片内RAM中40H处存在X，按规则将Y写入片内RAM中41H处

$$Y=\begin{cases} 1, \text {X>0} \\ 0, \text{X=0} \\ -1, \text{X<0}\end{cases}$$

```assembly
		ORG 1000H
START:
		MOV A 40H
		JZ DONE					;A=0时跳转
		JNB ACC.7 , POST		;A[7]=0（正数）时跳转
		MOV A , #FFH			;A=-1
		SJMP DONE				
POST:
		MOV A , #01H
DONE:
		MOV 41H , A
		SJMP $
		END
```

### 2.多分支

P1口：被操作数、结果低8位

P3口：操作数、结果高8位

R2：0-加 1-减 2-乘 3-除

```assembly
		ORG 1000H
START:
		MOV P1 , #DATA1
		MOV P2 , #DATA2
		MOV DPTR , #TABLE
		MOV A , R2
		RL A
		JMP @A+DPTR
TABLE:
		AJMP PRG0
		AJMP PRG1
		AJMP PRG2
		AJMP PRG3
PRG0:
		MOV A , P1			;A=P1
		ADD A , P3			;A=A+P3
		MOV P1 , A			;P1=A
		CLR A
		ADDC A , #00H		;A=A+CY
		MOV P3 , A			;P3=A
		RET
PRG1:
		MOV A , P1
		CLR C
		SUBB A , P3
		MOV P1 , A
		CLR A
		RLC A
		MOV P3 , A
		RET
PRG2:
		MOV A , P1
		MOV B , P3
		MUL AB
		MOV P1 , A
		MOV P3 , B
		RET
PRG3:
		MOV A , P1
		MOV B , P3
		DIV AB
		MOV P1 , A
		MOV P3 , B
		RET
```

## 4.3.3 循环程序设计

### 一、单重循环

1. 片内RAM38H-47H存放16个二进制无符号数，求和存放R4、R5

```assembly
		ORG 0800H
START:	MOV R0,#38H		;起始地址
		MOV R2,#10H		;计次循环
		MOV R4,#00H
		MOV R5,#00H
LOOP:	MOV A,R5		;低位送A
		ADD A,@R0		;A=A+R0
		MOV R5,A		;R5=A
		CLR A			;A=0
		ADDC A,R4		;A=A+R4+CY
		MOV R4,A		;R4=A
		INC R0			;R0++
		DJNZ R2,LOOP	;if R2!=0 goto LOOP
		SJMP $			;阻塞
		END
```

2. 片内RAM的30H-4FH传送至片外1800H开始

```assembly
		ORG 1000H
START:	MOV R0,#30H
		MOV DPTR,#1800H		;访问片外RAM应使用MOVX，并使用DPTR做地址
		MOV R2,#20H
LOOP:	MOV A,@R0
		MOVX @DPTR,A
		INC R0
		INC DPTR
		DJNZ R2,LOOP
		SJMP $
		END
```

### 二、多重循环

设计50ms延时程序

```assembly
		ORG 0800H
DELAY:	MOV R7,#200		;外循环次数，1T
DLY1:	MOV R6,#123		;内循环次数，1T
DLY2:	DJNZ R6,DLY2	;R6不为1时循环，2T
		NOP				;1T
		DJNZ R7,DLY1	;R7不为1时循环，2T
		RET				;2T
```

$f_{OSC}=12MHz$时，1T为1us

总延迟=(246+2+1+1)T*200+2T+1T=50.003ms

## 4.3.4 数值转换程序

### 一、BIN - BCD

入口：16bit无符号数R3、R2

出口：R6、R5、R4

```assembly
BINBCD:	CLR A
		MOV R4,A
		MOV R5,A
		MOV R6,A
		MOV R7,#10H			;计次
LOOP:	CLR C
		MOV A,R2
		RLC A
		MOV R2,A
		MOV A,R3
		RLC A
		MOV R3,A
		MOV A,R4
		ADDC A,R4
		DA A
		MOV R4,A
		MOV A,R5
		ADDC A,R5
		DA A
		MOV R5,A
		MOV A,R6
		ADDC A,R6
		MOV R6,A
		DJNZ R7,LOOP
		RET
```

### 二、BCD - BIN

### 三、ASCII - BIN

```assembly
ASCBIN:	MOV A,R1			;A=R1
		CLR C
		SUBB A,#30H			;A-30H
		MOV R1,A			;R1=A
		SUBB A,#0AH			;A-10
		JC LOOP				;IF CY=1(A<10) GOTO LOOP
		XCH A,R1			;A=R1
		SUBB A,#07H			;A-7
		MOV R1,A			;R1=7
LOOP:	RET
```

## 4.3.5 查表程序设计

查表计算$Y=X^2$

```assembly
		ORG 1000H
START:	MOV A,30H
		MOV DPTR,#TABLE
		MOVC A.@A+DPTR
		MOV 31H,A
TABLE:	DB 0,1,4,9,16,25,36,49,64,81
		END
```

```assembly
		ORG 1000H
START:	MOV A,30H
		ADD A,#02H
		MOVC A.@A+PC
		MOV 31H,A
TABLE:	DB 0,1,4,9,16,25,36,49,64,81
		END
```


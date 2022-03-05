# 4.1 汇编语言的概述

## 4.1.1 汇编语言的特点

## 4.1.2 汇编语言的伪指令

### 1.ORG

确定下一条指令的地址

### 2.DB

保存字节常数或字符或表达式

```
DB 73H,01H,90H
DB "Hello!"
```

### 3.DW

保存字数据（16位）

```
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

## 4.3.4 数值转换程序

## 4.3.5 查表程序设计
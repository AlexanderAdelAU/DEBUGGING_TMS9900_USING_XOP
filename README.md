# Using TMS 9000 XOP Instruction to implement a Debugging Function
You can use the XOP to implement a very simple but effective debugging function for use in your TMS 9900/99000 assembler code.

You first define your XOP instruction using the familiar declaration:

```
DXOP	DEBUG,15
```
Which declares that we are using XOP 15 and DEBUG is its name to declare in the assembler.

A sample test programme that demonstrates how to use it.   The following is a simple set of instructions
with the DEBUG XOP inserted into your code.

```
	 
  014E   0200 0003      	LI	R0,3
  0152   0201 0006      	LI	R1,6
  0156   8040           	C	R0,R1
  0158   2FE0 0208      	DEBUG	@TRACE_BUF
  015C   8001           	C	R1,R0
  015E   2FE0 0228      	DEBUG	@TRACE_BUF
  0162   0A10           	SLA	R0,1
  0164   2FE0 0248      	DEBUG	@TRACE_BUF

 0200             TRACE_BUF: BSS  36			;Used to store the trace data

```

and the code in DEBUG(XOP) will produce, during execution, something such as this:

```
PC = 015C  ST = 1004
R0 =0003 R1 =0006 R2 =0088 R3 =0000 R4 =0500 R5 =7285 R6 =8D77 R7 =574D
R8 =F2C0 R9 =01C6 R10=0204 R11=9044 R12=8000 R13=D411 R14=1C7B R15=0004

PC = 0162  ST = D004
R0 =0003 R1 =0006 R2 =0088 R3 =0000 R4 =0500 R5 =7285 R6 =8D77 R7 =574D
R8 =F2C0 R9 =01C6 R10=0204 R11=9044 R12=8000 R13=D411 R14=1C7B R15=0004

PC = 0168  ST = C004
R0 =0006 R1 =0006 R2 =0088 R3 =0000 R4 =0500 R5 =7285 R6 =8D77 R7 =574D
R8 =F2C0 R9 =01C6 R10=0204 R11=9044 R12=8000 R13=D411 R14=1C7B R15=0004
```

The source code for the XOP DEBUG function is:

```
;
;
;************************************************
;	DEBUG AND TRACING INFORMATION
;	STORE DEBUG TRACE DATA IF DEBUG_FLAG IS SET
;
;*************************************************
;
XOP15:	MOV	@TRACER_INDEX,R8
	AI	R8,TRACER_LIST		;THIS IS THE INDEX INTO THE LIST
	MOV	R11,*R8			;SAVE THE CURRENT TRACE POINTER IN THE LIST
;	INCT	@TRACER_INDEX		;MAYBE USE THIS LATER TO STORE A SEQUENCE AND PRINT OUT LATER
;
; 	NOW SAVE THE TRACE DATA
;
;	MOV	R14,R9			;ADJUST PC TO ACTUAL LOCATION AFTER THE STATEMENT
;					;THAT IS BEING DEBUGGED
;	AI	R9,-4			;NEED TO JUMP OVER DEBUG STATEMENT AS WELL
	MOV	R14,*R11+		;STORE NEXT STATEMENT PROGRAMME COUNTER
	MOV	R15,*R11+		;STORE STATUS
	LI	R8,16			;16 REGISTERS
XOP15_LOOP:
	MOV 	*R13+,*R11+		;COPY REGISTERS
	DEC	R8
	JNE	XOP15_LOOP
	AI	R13,-32			;RESTOR WORKSPACE REGISTER LOCATION
	JMP	REGIST
;	RTWP

;
; PRINT OUT DEBUGGING/TRACE PC, STATUS AND REGISTERS
;
REGIST:	CLR	R8		;START WITH INDEX = 0
	MOV	R8,R9
	AI	R9,TRACER_LIST	;THIS IS THE STARTING ADDRESS
	MOV	*R9,R9		;GET ADDRESS
	LI	R8,16
REGIST0:
 	MESG	@CRLF		;PRINT INDENTATION
	MESG	@PC_REG		;PRINT "PC="
	WHEX	*R9+
	MESG	@ST_REG		;PRINT "ST="
	MOV	*R9+,R11	;GET STATUS REGISTER VALUE
	WHEX	R11
	MESG	@PC_REG+4	;PRINT SPACE
REGISTA	LI	R10,30H*256	;PRINT 0
	SLA	R11,1
	JOC	REGISTB
	WRITE	R10
	JMP	REGISTC
REGISTB	AI	R10,1*256	;PRINT 1
	WRITE	R10
REGISTC	DEC	R8
	JNE	REGISTA

	CLR	R10
REGIST1:
	MESG	@CRLF		;PRINT CR,LF
REGIST2:
	LI	R8,'R'*256
	WRITE	R8		;PRINT "R"
	WRITE	@NUMTAB(R10)	;PRINT REGISTER NO
	WRITE	@NUMTAB+1(R10)	;PRINT REGISTER NO
	LI	R8,'='*256
	WRITE	R8
	WHEX	*R9+		;PRINT REGISTER CONTENTS
	INCT	R10
	CI	R10,20H
	JEQ	XOP15_X
	LI	R8,' '*256	;PRINT A SPACE
	WRITE	R8
	CZC	@MASKF,R10
	JEQ	REGIST1
	JMP	REGIST2

XOP15_X:
	MESG	@CRLF	;PRINT INDENTATION
	RTWP

ST_REG:	TEXT	'  ST = '
	BYTE	0
PC_REG:	TEXT	'PC = '
	BYTE	0
NUMTAB	TEXT	'0 1 2 3 4 5 6 7 8 9 101112131415'
MASKF	WORD	00FH
TRACER_INDEX:
	WORD	0
TRACER_LIST:
	BSS	20		;DEBUG TRACE LIST

```


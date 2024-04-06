# Using TMS 9000 XOP Instruction to implement a Debugging Function
Adding a software based debugging function to the TMS9900 is straight forward by using the XOP to implement a very simple but effective debugging function.

You first define your XOP instruction using the familiar declaration:

```
DXOP	DEBUG,15
```
Which declares that we are using XOP 15 and DEBUG is its name to declare in the assembler.

A sample test programme that demonstrates how to use it.   The following is a simple set of instructions
with the DEBUG XOP inserted into your code.

```
	 
                        ;
                        ;---perform simple tests
                        ;
                        ;
  0102   02E0 0136      TEST1:	LWPI	WORKSP
  0106   0200 7FFF      	LI	R0,7FFFH
  010A   0201 0002      	LI	R1,2
  010E   A040           	A	R0,R1
  0110   2FE0 0124      	DEBUG	@ADDTEST
  0114   2DA0 011C      	CALL	@TEST2
  0118   0420 E000      	BLWP	@MONITOR
                        
  011C   0A11           TEST2:	SLA	R1,1
  011E   2FE0 012D      	DEBUG	@SLATEST
  0122   2DC0           	RET
                        ;
                        ; NAMES OF THE ROUTINES BEING DEBUGGED
                        ;
  0124   5445 5354      ADDTEST:	TEXT	"TEST1-ADD"
  0128   312D 4144      
  012C   44             
  012D   5445 5354      SLATEST:	TEXT	"TEST2-SLA"
  0131   322D 534C      
  0135   41                     

```

and the code in DEBUG(XOP) will produce, during execution, something such as this:

```
TEST1-AD PC = 0114 ST = 8807 1000100000000111
R0 =7FFF R1 =8001 R2 =C020 R3 =0104 R4 =1303 R5 =0C0B R6 =0003 R7 =2DA0
R8 =E018 R9 =161A R10=C0E0 R11=0102 R12=0243 R13=00FF R14=0204 R15=0500

TEST2-SL PC = 0122 ST = D807 1101100000000111
R0 =7FFF R1 =0002 R2 =C020 R3 =0104 R4 =1303 R5 =0C0B R6 =0003 R7 =2DA0
R8 =E018 R9 =161A R10=C0DE R11=0102 R12=0243 R13=00FF R14=0204 R15=0500
```

The source code for the XOP DEBUG function is:

```
                        ;=================================================
                        
                        ;
                        ;
                        ;************************************************
                        ;	DEBUG AND TRACING INFORMATION
                        ;
                        ;	DEBUG @MODULE_NAME
                        ;
                        ;	THE MODULE_ID WILL BE PRINTED SO THAT THE USER CAN TELL WHICH
                        ;	MODULE IS BEING DEBUGGED.  KEEP TO 8 BYTES AND NULL TERMINATED
                        ;
                        ;
                        ;*************************************************
                        ;
  E778   0208 0008      XOP15:	LI	R8,8		;KEEP NAMES TO 8 BYTES
  E77C   0209 DED0      	LI	R9,DEBUG_NAME
  E780   DE7B           XOP15_NAME:	MOVB	*R11+,*R9+		;SAVE THE NAME
  E782   1305           	JEQ	XOP15_MAIN
  E784   0608           	DEC	R8
  E786   16FC           	JNE	XOP15_NAME
  E788   0208 0000      	LI	R8,0;		;NULL TERMINATE
  E78C   D648           	MOVB	R8,*R9
                        
                        ;
                        ; 	NOW SAVE THE TRACE DATA
                        ;
  E78E   020B DEDA      XOP15_MAIN:	LI	R11, DEBUG_BUFFER
  E792   CECE           	MOV	R14,*R11+		;STORE NEXT STATEMENT PROGRAMME COUNTER
  E794   CECF           	MOV	R15,*R11+		;STORE STATUS
  E796   0208 0010      	LI	R8,16		;16 REGISTERS
                        XOP15_REGS:
  E79A   CEFD           	MOV 	*R13+,*R11+		;COPY REGISTERS
  E79C   0608           	DEC	R8
  E79E   16FD           	JNE	XOP15_REGS
  E7A0   022D FFE0      	AI	R13,-32		;RESTOR WORKSPACE REGISTER LOCATION
  E7A4   1000           	JMP	LIST_REG
                        ;
                        ;
                        ; PRINT OUT DEBUGGING/TRACE PC, STATUS AND REGISTERS
                        ;
  E7A6   0209 DEDA      LIST_REG:	LI	R9,DEBUG_BUFFER
  E7AA   0208 0010      	LI	R8,16
  E7AE   2FA0 E038       	MESG	@CRLF		;PRINT INDENTATION
  E7B2   2FA0 DED0       	MESG	@DEBUG_NAME		;PRINT THE NAME OF THE MODULE
  E7B6   2FA0 E824      	MESG	@PC_REG		;PRINT " PC="
  E7BA   2EB9           	WHEX	*R9+
  E7BC   2FA0 E81D      	MESG	@ST_REG		;PRINT " ST="
  E7C0   C2F9           	MOV	*R9+,R11		;GET STATUS REGISTER VALUE
  E7C2   2E8B           	WHEX	R11
  E7C4   020A 2000      	LI	R10,' '*256		;PRINT SPACE
  E7C8   2F0A           	WRITE	R10
  E7CA   020A 3000      LIST_REGA:	LI	R10,30H*256		;PRINT 0
  E7CE   0A1B           	SLA	R11,1
  E7D0   1802           	JOC	LIST_REGB
  E7D2   2F0A           	WRITE	R10
  E7D4   1003           	JMP	LIST_REGC
  E7D6   022A 0100      LIST_REGB:	AI	R10,1*256		;PRINT 1
  E7DA   2F0A           	WRITE	R10
  E7DC   0608           LIST_REGC:	DEC	R8
  E7DE   16F5           	JNE	LIST_REGA
  E7E0   04CA           	CLR	R10
  E7E2   2FA0 E038      LIST_REG1:	MESG	@CRLF		;PRINT CR,LF
  E7E6   0208 5200      LIST_REG2:	LI	R8,'R'*256
  E7EA   2F08           	WRITE	R8		;PRINT "R"
  E7EC   2F2A E078      	WRITE	@NUMTAB(R10)	;PRINT REGISTER NO
  E7F0   2F2A E079      	WRITE	@NUMTAB+1(R10)	;PRINT REGISTER NO
  E7F4   0208 3D00      	LI	R8,'='*256
  E7F8   2F08           	WRITE	R8
  E7FA   2EB9           	WHEX	*R9+		;PRINT REGISTER CONTENTS
  E7FC   05CA           	INCT	R10
  E7FE   028A 0020      	CI	R10,20H
  E802   1307           	JEQ	LIST_EXIT
  E804   0208 2000      	LI	R8,' '*256		;PRINT A SPACE
  E808   2F08           	WRITE	R8
  E80A   26A0 E0D4      	CZC	@MASK15,R10
  E80E   13E9           	JEQ	LIST_REG1
  E810   10EA           	JMP	LIST_REG2
  E812   2FA0 E038      LIST_EXIT:	MESG	@CRLF		;PRINT INDENTATION
  E816   0380           	RTWP
                        
  E818   2057 503D      WP_REG:	TEXT	' WP='
  E81C   00             	BYTE	0
  E81D   2053 5420      ST_REG:	TEXT	' ST = '
  E821   3D20           
  E823   00             	BYTE	0
  E824   2050 4320      PC_REG:	TEXT	' PC = '
  E828   3D20           
  E82A   00             	BYTE	0


                        ;
                        ;	NOTE THIS IS USED BY DEBUG TO STORE MULTIPLE DEBUG POINTERS TO
                        ;	PROGRAMMES CALLING DEBUG
                        ;
  DED0                  DEBUG_NAME:	BSS	9		;;NAME OF MODULE 8 CHARS LONG NULL TERMINATED
  DED9   00             	        EVEN
                        DEBUG_BUFFER:
  DEDA                  	        BSS	20		;DEBUG TRACE LIST

```


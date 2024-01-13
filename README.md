For example, if you define an XOP as say DEBUG using code similar to 

DXOP	DEBUG,15
Then, in your programme code that needs to be debugged you can add the following:

  .
  .
  014E   0200 0003      	LI	R0,3
  0152   0201 0006      	LI	R1,6
  0156   8040           	C	R0,R1
  0158   2FE0 0208      	DEBUG	@TRACE_BUF
  015C   8001           	C	R1,R0
  015E   2FE0 0228      	DEBUG	@TRACE_BUF
  0162   0A10           	SLA	R0,1
  0164   2FE0 0248      	DEBUG	@TRACE_BUF
.
. 0200             TRACE_BUF: BSS  36			;Used to store the trace data
.
and the code in DEBUG(XOP) can produce, during execution, something such as this:

PC = 015C  ST = 1004
R0 =0003 R1 =0006 R2 =0088 R3 =0000 R4 =0500 R5 =7285 R6 =8D77 R7 =574D
R8 =F2C0 R9 =01C6 R10=0204 R11=9044 R12=8000 R13=D411 R14=1C7B R15=0004

PC = 0162  ST = D004
R0 =0003 R1 =0006 R2 =0088 R3 =0000 R4 =0500 R5 =7285 R6 =8D77 R7 =574D
R8 =F2C0 R9 =01C6 R10=0204 R11=9044 R12=8000 R13=D411 R14=1C7B R15=0004

PC = 0168  ST = C004
R0 =0006 R1 =0006 R2 =0088 R3 =0000 R4 =0500 R5 =7285 R6 =8D77 R7 =574D
R8 =F2C0 R9 =01C6 R10=0204 R11=9044 R12=8000 R13=D411 R14=1C7B R15=0004


;
; 	XOP 15:  Debug source code
;
                  XOP15:      
  EE30   CECE           	MOV	R14,*R11+	;STORE PROGRAMME COUNTER FIRST
  EE32   CECF           	MOV	R15,*R11+	;STORE STATUS REGISTER
  EE34   0208 0010      	LI	R8,16		;16 REGISTERS
  EE38   CEFD     XOP_LOOP:	MOV	*R13+,*R11+	;COPY REGISTERS
  EE3A   0608           	DEC	R8
  EE3C   16FD           	JNE	XOP_LOOP
  EE3E   022D FFE0      	AI	R13,-32		RESTOR WORKSPACE REGISTER LOCATION
  

## Modem Initialization

* Lines 29000-29002: Define constants like CR, BELL etc
* Lines 29010-29050: Dimension variables and strings
* Lines 29060-29075: Get date, time, disk from user
* Line 29080: Define subroutine pointers like CLM (close modem)
* Lines 29100-29140: Open config file, Read start sectors, bytes, message counters, Calculate buffer size
* Lines 29160-29180: Set time variables from user input
* Lines 29187-29189: POKE to modem registers - speed, command mode, Enable relay for auto-answer
* Line 29190: OPEN modem port

## Wait for Call

* Lines 10170-10180:STATUS check for carrier (bit 7 of 749), Loop waiting for carrier
* Lines 11000-11050: PRINT answering message, Set timeout in case no carrier, STATUS check for carrier, GET first character to detect Xmodem "I", Loop waiting for carrier

## User Input Loop

* Line 10150: Print prompt, time and date
* Lines 10160-10162: Check joystick trigger for local input, Check keyboard for local input
* Line 10170: Jump to answer call if carrier
* Line 400: Print prompt, GET line input
* Line 70-140: Input routine, Get character, Handle deletes, backspaces,  Add to line, check length, Sound bell on overflow, Return line
* Line 1000: Check first char against command codes
* Line 1050: Dispatch to command handlers
* Lines 1500-1600: Toggle expert mode
* Lines 2000-2140: Print time and date
* Lines 2180-2480: Message scan/read
* Lines 2600-3400: Message input, edit, save
* Lines 5000-6800: Upload and download files

```
5 GOTO 29000
8 STATUS #MD,X:IF PEEK(749)>ZR THEN GOTO C8
9 IF PEEK(764)=C255 THEN IF  NOT PEEK(747) OR LOCAL THEN RETURN 
10 GOSUB GC:IF X<>19 THEN RETURN 
11 GOSUB GC:IF X=17 OR X=152 THEN RETURN 
12 IF X=C24 THEN TST=ZR:RETURN 
15 GOTO 11
20 IF LOCAL THEN GET #3,X:RETURN 
25 STATUS #MD,X:IF PEEK(747) THEN GET #MD,X:? CHR$(X);:RETURN 
30 IF PEEK(764)<C255 THEN GET #3,X:? CHR$(X);:RETURN 
40 TOUT=TOUT+WON:IF TOUT>1000 THEN GOSUB OPM
50 GOTO GC
70 L$="":IF PEEK(53279)=5 THEN POP :GOTO 13000
80 TOUT=ZR:GOSUB GC:X$=CHR$(X)
90 IF ((X$=BS$) OR (X$=CHR$(C127) AND AMODE=ZR)) AND L$<>"" THEN ? #MD;BS$;" ";BS$;:L$(LEN(L$))="":GOTO 80
100 IF X$=DEL$ AND L$<>"" THEN FOR X=1 TO LEN(L$):? #MD;BS$;" ";BS$;:NEXT X:L$="":GOTO 80
110 IF X$=CR$ THEN ? #MD:RETURN 
120 ? #MD;X$;:L$(LEN(L$)+WON)=X$:IF LEN(L$)<IL-3 THEN 80
130 IF LEN(L$)=IL THEN ? #MD:RETURN 
140 ? #MD;BEL$;:GOTO 80
160 ? #MD:IF  NOT XMODE THEN ? #MD;CTRL$:REM SEND
170 L=LEN(BF$):IF  NOT L THEN RETURN 
190 F=WON:T=ZR
200 T=T+C8:IF T>=L THEN ? #MD;BF$(F,L);:RETURN 
210 ? #MD;BF$(F,T);:F=F+C8:GOSUB C8:IF X=24 OR X=152 THEN ? #MD;CR$;"Interrupted";CR$:RETURN 
215 IF X=16 THEN ? #MD;CR$;"Node:2";CR$:TST=ZR:GOTO 30400
220 GOTO 200
292 TRAP C8080:IF PEEK(864)=C255 OR LOCAL THEN RETURN 
294 STATUS #MD,X:IF PEEK(749)>ZR THEN 294
296 X=WON^WON:CLOSE #MD:RETURN 
310 POKE 77,ZR:GOSUB CLM:IF LOCAL THEN RETURN 
315 OPEN #MD,13,ZR,"R:"
320 XIO 36,#MD,ZR,WON,"R:":XIO 34,#MD,240,ZR,"R:"
325 XIO 38,#MD,AMODE+LMODE,ZR,"R:"
330 STATUS #MD,X:TOUT=ZR
335 X=832+(16*MD)
340 POKE X+MD,C40:POKE X+4,ZR:POKE X+5,6
345 POKE X+8,C120:POKE X+9,ZR:POKE X+10,13
350 X=USR(ADR(SCIO$),16*MD)
355 IF PEEK(16*MD+835)=WON THEN RETURN 
360 GOTO OPM
372 ? #MD;" <Y/N>";:GOSUB GC:IF X>96 THEN X=X-32
374 IF X=89 THEN X=WON:? #MD;"Y":RETURN 
376 IF X=78 THEN X=ZR:? #MD;"N":RETURN 
378 GOTO 372
400 ? #MD;"HIT <RETURN> ";:GOSUB GC
410 IF X=155 THEN ? #MD:GOTO 500
420 IF X<>13 AND X<>141 THEN 400
430 BEL$=CHR$(7):BS$=CHR$(C8):DEL$=CHR$(C24):AMODE=ZR:GOSUB OPM
440 ? #MD;CHR$(10);CR$;"Atari's Don't Need Line Feeds";CR$;"Do You";:GOSUB 372
460 IF X THEN LMODE=64:GOSUB OPM
500 FL$="D1:WELCOME":GOSUB RF:GOSUB 170
510 IL=C40:? #MD;CR$;"Enter your Name >";:GOSUB GL:IF LEN(L$)<3 THEN 510
511 GOSUB CLM:CLOSE #FI
512 IF L$="SYSOP" THEN 535
513 IF L$="sysop" THEN 535
514 IF L$="ADOLPH HITLER" THEN 3356
515 IF L$="JustJefF" THEN NAM$="Sysop":AD$="":TXA=60:GOTO MN
533 NAM$=L$:GOTO 540
535 IF  NOT LOCAL THEN ? #MD;CR$;"YOU ARE NOT THE SYSOP":GOTO 510
540 GOSUB OPM:? #MD;"From City,State >";:GOSUB GL:IF LEN(L$)<3 THEN 540
560 AD$=L$:TXA=45:XMODE=0
580 ? #MD;"You are ";NAM$:? #MD;"Calling from ";AD$
590 ? #MD;"CORRECT";
600 GOSUB 372
610 IF  NOT X THEN 510
640 CALLNO=CALLNO+WON:? #MD;"You are caller No ";CALLNO
650 ? #MD;"Just a sec..."
660 GOSUB CLM:CLOSE #FI:TXO=VAL(TM$(WON,MD))*60+VAL(TM$(4,5))
670 OPEN #FI,12,ZR,CDF$:NOTE #FI,A,I:POINT #FI,CSECT,CBYTE
680 GOSUB 2010:? #FI;NAM$;CR$;AD$;CR$;TD$;CR$;TM$
690 NOTE #FI,CSECT,CBYTE:? #FI;"*":CLOSE #FI:GOSUB 28100
700 LPRINT "ON ";NAM$;"  ";AD$:LPRINT " ";TD$;" ";TM$;" CALLER NO.";CALLNO;"  ";CSECT-A
800 GOTO MN
900 GOSUB RF
910 GOSUB 160
912 IF TST=WON THEN GOSUB CLM:GOTO 5060
950 ? #MD:? #MD
1000 GOSUB C2010:TXC=VAL(TM$(WON,MD))*60+VAL(TM$(4,5)):TXE=TXC-TXO:IF TXE<ZR THEN TXE=TXE+C24*60
1001 GOSUB OPM:IF TXE>=TXA THEN ? #MD;" Sorry, time's up! ";CR$:GOTO 3300
1002 IF TXA-TXE<=10 THEN ? #MD;CR$;"        ";TXA-TXE;" minutes left"
1003 TRAP 8080:GOSUB OPM:? #MD;CR$;PRMT$;:IL=40:IF  NOT LOCAL THEN ? :? "TXE=";TXE;" CALL#";CALLNO:? PRMT$;
1010 GOSUB GC:L$=CHR$(X):IF L$="" THEN GOTO MN
1030 X=ZR:IF L$(WON,WON)>="a" THEN L$(WON,WON)=CHR$(ASC(L$)-32)
1035 ? #MD;L$;CR$
1040 X=X+WON:IF X>12 THEN ? #MD;"I DON'T KNOW WHAT ";L$;" MEANS.";BEL$:GOTO MN
1050 IF L$(WON,WON)=FC$(X,X) THEN ON X GOTO 1100,1200,1300,1400,1500,2180,2000,2190,2600,12000
1055 IF L$(WON,WON)=FC$(X+10,X+10) THEN ON X GOTO 3300,3400,3500,3600,2200,3900,5000,5200,5400,1450,1350,30200
1060 GOTO 1040
1100 FL$="D1:BULLET":GOTO C900
1200 FL$="D1:FUNC":GOTO C900
1300 FL$="D1:HELP":GOTO C900
1350 FL$="D1:NEWUSER":GOTO C900
1400 FL$="D1:WELCOME":GOTO C900
1450 FL$="D1:LOCALBBS":GOTO C900
1500 XMODE= NOT XMODE
1510 IF XMODE THEN ? #MD;"EXPERT USER MODE":PRMT$="Command?>":GOTO MN
1520 ? #MD;"NORMAL USER MODE":GOSUB 1530:GOTO MN
1530 PRMT$="A,B,C,D,E,F,G,H,I,J,K,L,M,Q,R,S,T,U,W"
1540 PRMT$(LEN(PRMT$)+WON)=CR$
1550 PRMT$(LEN(PRMT$)+WON)=",X,Y or ? >"
1560 RETURN 
2000 GOSUB 2010:? #MD;"TIME: ";TM$;"   DATE: ";TD$;CR$;"   ";TXA-TXE;" Minutes Remaining";CR$:GOTO MN
2010 X=PEEK(20):T=((PEEK(18)*C256+PEEK(19))*C256+PEEK(20))/60
2020 IF X>PEEK(20) THEN 2010
2030 X=INT(T/3600):F=INT(T/60)-X*60
2040 T=T-X*3600-F*60:IF X<C24 THEN 2100
2050 X=X-C24:TD$(5,5)=CHR$(ASC(TD$(5))+WON)
2060 IF TD$(5,5)>"9" THEN TD$(5,5)="0":TD$(4,4)=CHR$(ASC(TD$(4))+WON)
2062 LI=VAL(TD$)*MD
2064 IF TD$(4,5)<=MTH$(LI-WON,LI) THEN 2070
2065 IF TD$(4,5)="29" AND VAL(TD$(7))/4=INT(VAL(TD$(7))/4) THEN 2070
2066 TD$(4,5)="01":TD$(MD,MD)=CHR$(ASC(TD$(MD))+WON)
2067 IF TD$(MD,MD)>"9" THEN TD$(MD,MD)="0":TD$(WON,WON)="1"
2068 IF TD$(WON,MD)<="12" THEN 2070
2069 TD$(WON,MD)="01":TD$(C8,C8)=CHR$(ASC(TD$(C8))+WON)
2070 X=(T+X*3600+F*60)*60+7000
2080 F=INT(X/65536):T=INT(X/C256)-F*C256
2090 X=X-F*65536-T*C256:POKE 20,X:POKE 19,T:POKE 18,F:GOTO 2010
2100 TP$=STR$(X+100):TM$=TP$(MD,3)
2120 TM$(3,5)=STR$(F+100):TM$(6,8)=STR$(T+100)
2140 TM$(3,3)=":":TM$(6,6)=":":RETURN 
2180 QRS=-WON:? #MD;"Quick Scan";:GOTO 2210
2190 QRS=ZR:? #MD;"Retrieve";:GOTO 2210
2200 QRS=WON:? #MD;"Summary of";
2210 ? #MD;" Messages":FL$=MIF$:GOSUB RF
2212 ? #MD;MSGS;" Messages":IF  NOT XMODE THEN ? #MD;CTRL$
2220 L=INT(LEN(BF$)/C40)*C40:IF L=ZR THEN ? #MD;"No Messages":GOTO MN
2230 MSGNO$=BF$:F=VAL(MSGNO$):MSGNO$=BF$(L-39):T=VAL(MSGNO$)
2235 MSGS=INT(LEN(BF$)/C40)
2240 ? #MD;"First Msg# ";F;"  Last Msg# ";T
2250 IF LEN(L$)>MD THEN L$=L$(3):GOTO 2280
2260 ? #MD;"RETURN=Exit, Msg# From-To >";
2270 GOSUB GL:? #MD:IF L$="" THEN GOTO MN
2280 GOSUB PR:IF DIR=ZR THEN ? #MD:GOTO 2260
2283 GOSUB SR
2285 IF F<WON OR F*C40>LEN(BF$) THEN GOTO 2280
2290 MSGNO$=BF$(F*C40-39):T=VAL(MSGNO$):IF DIR=WON THEN IF T<FROM OR T>TU THEN 2280
2295 IF DIR=-WON THEN IF T<TU OR T>FROM THEN 2280
2297 IF QRS<ZR THEN ? #MD;T;". ";BF$(F*C40-35,F*C40-3):GOSUB C8:GOTO 2302
2300 GOSUB CLM:CLOSE #FI:OPEN #FI,4,ZR,MDF$
2301 GOSUB 2305
2302 IF X=C24 THEN 2260
2303 F=F+DIR:GOTO 2285
2305 BYTE=ASC(BF$(F*C40))
2306 SECT=ASC(BF$(F*C40-MD))*C256+ASC(BF$(F*C40-1))
2310 POINT #FI,SECT,BYTE
2320 INPUT #FI,MSGNO$,SUBJ$,PAS$,DATE$,TM$,FROM$,FR$,LI
2322 MSG$="MSG# ":MSG$(6)=MSGNO$:MSG$(10)=" DATE:":MSG$(16)=DATE$:MSG$(C24)=" TIME:":MSG$(30)=TM$:MSG$(38)=CR$
2324 MSG$(39)="FROM: ":MSG$(45)=FROM$:MSG$(LEN(MSG$)+WON)=CR$:MSG$(LEN(MSG$)+WON)="  TO: ":MSG$(LEN(MSG$)+WON)=FR$
2326 MSG$(LEN(MSG$)+WON)=CR$:MSG$(LEN(MSG$)+WON)="SUBJ: ":MSG$(LEN(MSG$)+WON)=SUBJ$:MSG$(LEN(MSG$)+WON)=CR$
2330 IF QRS>ZR THEN MSG$(LEN(MSG$)+WON)="=========":GOTO 2400
2340 FOR X=WON TO LI
2350 INPUT #FI,TP$
2360 MSG$(LEN(MSG$)+WON)=TP$
2370 MSG$(LEN(MSG$)+WON)=CR$
2380 NEXT X
2400 L=LEN(MSG$):GOSUB OPM
2410 Y=WON:T=ZR
2420 T=T+C8:IF T>=L THEN ? #MD;MSG$(Y,L);:GOTO 2470
2430 ? #MD;MSG$(Y,T);:Y=Y+C8
2440 GOSUB C8
2450 IF X=C24 OR X=14 THEN ? #MD:RETURN 
2460 GOTO 2420
2470 ? #MD;" ":IF LOCAL THEN ? "PRINT <Y/N>":GET #3,X:? :IF X=89 THEN LPRINT MSG$
2480 RETURN 
2600 FROM$=NAM$
2610 ? #MD;"Enter Message---":? #MD;"SUBJECT: ";
2630 GOSUB GL:IF L$="" THEN GOTO MN
2640 SUBJ$=L$
2650 ? #MD;"TO: <RETURN>=All ";
2660 GOSUB GL:IF L$="" THEN L$="ALL":? #MD;"TO: ";L$;CR$
2670 FR$=L$:? #MD;"Enter PASSWORD Required to Kill Msg:":GOSUB GL:PAS$=L$
2680 LI=ZR:? #MD;"Enter Message, Two <CR>s when done"
2690 IL=C120:LI=LI+WON:IF LI>16 THEN 2790
2700 IF LI>13 THEN ? #MD;"Only ";17-LI;" Lines left"
2710 ? #MD;LI;" ";
2720 GOSUB GL
2730 IF L$="" THEN 2790
2740 MSG$(LI*C121-C120)=CHR$(LEN(L$))
2750 MSG$(LI*C121-C119)=L$
2760 GOTO 2690
2790 LI=LI-WON
2800 ? #MD;"(A)dd,(E)dit,(L)ist,(Q)uit,(S)ave ?";
2810 GOSUB GC:? #MD;CHR$(X):IF X>96 THEN X=X-32
2820 IF X=65 THEN 2690
2830 IF X=69 THEN 2900
2840 IF X=76 THEN 3000
2850 IF X=81 THEN ? #MD;"MESSAGE ABORTED":GOTO MN
2860 IF X=83 THEN 3100
2870 GOTO 2800
2900 IF LI=ZR THEN 2800
2910 ? #MD;"EDIT WHICH LINE 1-";LI;" ?";
2920 GOSUB GL:IF L$="" THEN 2800
2930 TRAP 2910:Z=INT(VAL(L$)):TRAP C8080
2940 IF Z<WON OR Z>LI THEN 2910
2950 ? #MD;"OLD LINE ";Z;" READS:";CR$
2960 ? #MD;Z;" ";MSG$(C121*Z-C119,C121*Z-C120+ASC(MSG$(Z*C121-C120)))
2970 ? #MD;"CHANGE TO:  <RETURN>=NO CHANGE";CR$;Z;" ";:GOSUB GL
2980 IF L$<>"" THEN MSG$(Z*C121-C120,Z*C121-C120)=CHR$(LEN(L$)):MSG$(Z*C121-C119,Z*C121)=L$
2990 GOTO 2910
3000 IF LI=ZR THEN 2800
3010 FOR X=WON TO LI
3020 ? #MD;X;" ";MSG$(X*C121-C119,X*C121-C120+ASC(MSG$(X*C121-C120)))
3030 NEXT X
3040 GOTO 2800
3100 IF LI=ZR THEN 2800
3105 HMSG=HMSG+WON:MSGNO$="0000":MSGNO$(5-LEN(STR$(HMSG)))=STR$(HMSG):? #MD;"SAVING MESSAGE...."
3110 GOSUB CLM:CLOSE #FI:OPEN #FI,12,ZR,MDF$:NOTE #FI,A,I:POINT #FI,MSECT,MBYTE
3120 GOSUB 2010:SECT=MSECT:BYTE=MBYTE
3130 ? #FI;MSGNO$;CR$;SUBJ$;CR$;PAS$;CR$;TD$;CR$;TM$;CR$;FROM$;CR$;FR$;CR$;LI
3140 FOR X=WON TO LI:? #FI;MSG$(X*C121-C119,X*C121-C120+ASC(MSG$(X*C121-C120))):NEXT X:NOTE #FI,MSECT,MBYTE
3150 CLOSE #FI:FL$=MIF$:GOSUB RF:GOSUB CLM
3160 TP$="                                     ":TP$=MSGNO$:TP$(5)=SUBJ$
3170 T=INT(SECT/C256):SECT=SECT-T*C256
3180 TP$(38)=CHR$(T):TP$(39)=CHR$(SECT):TP$(C40)=CHR$(BYTE)
3190 BF$(INT(LEN(BF$)/C40)*C40+WON)=TP$:CLOSE #FI
3200 OPEN #FI,C8,ZR,FL$:? #FI;BF$;:CLOSE #FI:GOSUB 28100
3210 LPRINT "MESSAGE ";MSGNO$;"  ";MSECT-A:GOSUB OPM:MSGS=MSGS+WON
3220 ? #MD;"SAVED AS MSG#";MSGNO$
3230 GOTO MN
3300 ? #MD;"Any Comments";
3310 GOSUB 372:IF  NOT X THEN 3350
3320 ? #MD;"Enter comments"
3330 ? #MD;">";:GOSUB GL:IF L$="" THEN 3350
3340 GOSUB CLM:LPRINT L$:GOSUB OPM:GOTO 3330
3350 GOSUB CLM:GOSUB 2010
3351 LPRINT "LOG-OFF ";TM$;"   TXE=";TXE:GOSUB OPM
3352 ? #MD;"Thanks for calling ";NAM$
3353 ? #MD;"Please call again..."
3356 GOTO WR
3400 LMODE=64-LMODE:GOSUB OPM:? #MD;"LINE-FEED ";
3410 IF LMODE THEN ? #MD;"ON":GOTO MN
3420 ? #MD;"OFF":GOTO MN
3500 AMODE=32:GOSUB OPM:? #MD;"HIT <RETURN>";:GOSUB GC:IF X<>155 THEN AMODE=ZR
3505 GOSUB OPM
3510 IF AMODE THEN BEL$=CHR$(253):DEL$=CHR$(156):BS$=CHR$(126):? #MD;"ATASCII Mode":GOTO MN
3520 BEL$=CHR$(7):DEL$=CHR$(C24):BS$=CHR$(C8):? #MD;"ASCII Mode":GOTO MN
3600 ? #MD;CALLNO;" Callers":GOSUB CLM:CLOSE #FI:OPEN #FI,4,ZR,CDF$
3610 GOSUB 3680:GOSUB OPM:? #MD;"First Date:";DATE$;"-Last Date:";TD$;CR$;"ENTER Starting Date <MM/DD/YY>";
3612 GOSUB GL:IF LEN(L$)<>8 THEN GOTO MN
3616 IF L$(3,3)<>"/" OR L$(6,6)<>"/" THEN 3610
3617 IF (L$>=DATE$ AND L$<="12/31/99") OR (L$<=TD$ AND L$>="01/01/00") THEN 3619
3618 GOTO 3610
3619 ? #MD;"SEARCHING CALLERS..."
3620 IF L$<>DATE$ THEN GOSUB 3680:GOTO 3620
3622 IF  NOT XMODE THEN GOSUB OPM:? #MD;CTRL$
3625 GOSUB OPM
3630 ? #MD;"CALLER: ";TP$;CR$;"FROM: ";A$;CR$;"AT: ";TM$;"  ON: ";DATE$;CR$
3640 GOSUB C8:IF X=C24 THEN GOTO MN
3650 GOSUB 3680:GOTO 3625
3680 GOSUB CLM:TRAP 3690:INPUT #FI,TP$,A$,DATE$,TM$:TRAP C8080:IF TP$<>"*" THEN RETURN 
3690 POP :CLOSE #FI:GOSUB OPM:GOTO 950
3900 ? #MD;"Kill Message"
3910 FL$=MIF$:GOSUB RF
3920 L=INT(LEN(BF$)/C40)*C40:IF L=ZR THEN ? #MD;"No Messages":GOTO MN
3930 MSGNO$=BF$:F=VAL(MSGNO$):MSGNO$=BF$(L-39):T=VAL(MSGNO$)
3940 ? #MD;"First Msg# ";F;"  Last Msg# ";T
3950 IF LEN(L$)>MD THEN L$=L$(3):GOTO 3980
3960 ? #MD;"<CR>=EXIT, Kill <Msg#>";
3970 GOSUB GL:IF L$="" THEN GOTO MN
3980 GOSUB PR:IF DIR=ZR THEN 3960
3990 GOSUB SR
3995 IF F<WON OR F*C40>LEN(BF$) THEN 3960
4000 MSGNO$=BF$(F*C40-39):T=VAL(MSGNO$)
4010 IF T<>FROM THEN CLOSE #FI:GOSUB OPM:? #MD;"MESSAGE NOT FOUND":GOTO MN
4020 GOSUB CLM:CLOSE #FI:OPEN #FI,4,ZR,MDF$
4030 QRS=WON:GOSUB 2305
4080 ? #MD;"PASSWORD=";
4090 GOSUB GL
4100 IF L$<>PAS$ AND L$<>"JustJefF" THEN ? #MD;"INVALID PASSWORD";BEL$:GOTO MN
4110 IF F*C40+WON>LEN(BF$) THEN BF$(F*C40-39)="":GOTO 4130
4120 BF$(F*C40-39)=BF$(F*C40+WON)
4130 GOSUB CLM:CLOSE #FI:OPEN #FI,C8,ZR,MIF$
4140 ? #FI;BF$:CLOSE #FI:GOSUB 28100
4150 LPRINT "KILLED MSG ";MSGNO$:GOSUB OPM:? #MD;"MESSAGE DELETED":GOTO MN
5000 GOSUB 5100:L=X:GOSUB CLM:CLOSE #FI:TRAP 5800:OPEN #FI,4,ZR,FL$:CLOSE #FI
5010 TRAP 8080:GOSUB 2010:IF L THEN 6000
5020 LPRINT "DL ";FL$;" ";TM$
5030 GOSUB 5900:GOSUB OPM:? #MD;"FILE: ";FL$:GOSUB 160
5040 IF BFLAG OR X=24 OR X=152 THEN 950
5050 GOSUB CLM:BF$="":GOSUB 5920:GOSUB OPM:GOSUB 170:GOTO 5040
5100 FL$="D1:":? #MD;"RETURN=Exit, File Name >";:IF LEN(L$)>1 THEN IF L$(2,2)="N" THEN FL$(2,2)="1"
5110 GOSUB GL:IF L$="" THEN POP :GOTO MN
5120 IF LEN(L$)>C8 THEN L$(9)=""
5130 FOR X=WON TO LEN(L$):IF L$(X,X)>"Z" THEN L$(X,X)=CHR$(ASC(L$(X,X))-32)
5140 IF L$(X,X)>="A" AND L$(X,X)<="Z" THEN FL$(LEN(FL$)+WON)=L$(X,X)
5150 IF L$(X,X)>="0" AND L$(X,X)<="9" THEN FL$(LEN(FL$)+WON)=L$(X,X)
5160 NEXT X:FL$(LEN(FL$)+WON)=".UDL":IF FL$(4,4)<"A" THEN POP :GOTO MN
5170 IF LEN(FL$)<C8 THEN POP :GOTO MN
5180 ? #MD;"Are you using the Christensen XMODEM":? #MD;"File transfer protocol";
5190 GOSUB 372:RETURN 
5200 GOSUB CLM:CLOSE #FI:REM U
5210 OPEN #FI,6,ZR,"D1:*.XXX":BF$="":TP$=""
5220 INPUT #FI,BF$:IF BF$(4,5)<>" F" THEN 5220
5230 BF$(LEN(BF$)+WON)=CR$:CLOSE #FI:GOSUB OPM:? #MD;BF$:GOSUB 5100:L=X
5240 GOSUB CLM:GOSUB 2010:CLOSE #FI:FL$(2,2)="1":IF L THEN LPRINT "XUP ";FL$;" ";TM$:GOTO 5260
5250 LPRINT "UP ";FL$;" ";TM$
5260 TRAP 5290:OPEN #FI,4,ZR,FL$:TRAP 8080
5270 CLOSE #FI:GOSUB OPM:? #MD;CR$;BEL$;"FILE ALREADY EXISTS!!!"
5280 GOTO MN
5290 CLOSE #FI:GOSUB OPM:TRAP 8080:BF$="":IF L THEN 6500
5300 GOSUB CLM:OPEN #FI,C8,ZR,FL$:GOSUB OPM
5310 ? #MD;"Upload --- Enter file <CR>=Exit "
5320 ? #MD;">";:IL=120:GOSUB GL
5330 IF L$="" THEN GOSUB CLM:CLOSE #FI:GOTO MN
5340 GOSUB CLM:? #FI;L$:GOSUB OPM:GOTO 5320
5400 GOSUB CLM:CLOSE #FI:FL$="D1:*.UDL":IF LEN(L$)>1 THEN IF L$(2,2)="N" THEN FL$(2,2)="1"
5410 OPEN #FI,6,ZR,FL$:BF$="":TP$="FILE DIRECTORY---":GOTO 5440
5420 INPUT #FI,TP$:IF TP$(4,5)=" F" THEN 5470
5430 TP$(11,13)="  -"
5440 BF$(LEN(BF$)+WON)=TP$
5450 BF$(LEN(BF$)+WON)=CR$
5460 GOTO 5420
5470 BF$(LEN(BF$)+WON)="* = Binary or Tokenized File"
5480 CLOSE #FI:GOSUB 310:GOSUB 160:GOTO MN
5800 TRAP 8080:GOSUB OPM:? #MD
5810 ? #MD;"Can't find that file"
5820 GOTO MN
5900 GOSUB CLM:CLOSE #FI:OPEN #FI,4,ZR,FL$:REM INF. BUFF
5910 POKE 195,ZR:A$(255)=" ":BF$=""
5920 TRAP 5930:FOR I=1 TO 4:XIO 7,#FI,4,ZR,A$:BF$(LEN(BF$)+WON)=A$:NEXT I:BFLAG=ZR:RETURN 
5930 IF PEEK(856) THEN BF$(LEN(BF$)+WON)=A$(WON,PEEK(856))
5940 BFLAG=PEEK(195):RETURN 
5950 T=LEN(BF$):F=((T/128)-INT(T/128))*128
5960 FOR I=F+1 TO 128:BF$(LEN(BF$)+1)=CHR$(F):NEXT I
5970 RETURN 
6000 CLOSE #FI
6010 LPRINT "XDL ";FL$;" ";TM$
6020 GOSUB 5900:IF BFLAG THEN GOSUB 5950
6030 GOSUB OPM:? #MD;"FILE: ";FL$;" Ready to Send":? #MD;"^X to cancel"
6040 BLOCK=WON:GOSUB GC:IF X<>21 THEN GOTO MN
6050 AM=AMODE:LM=LMODE:AMODE=32:LMODE=ZR:GOSUB OPM
6060 DIR=ZR
6070 FOR T=WON TO 10:PUT #MD,WON:PUT #MD,BLOCK:PUT #MD,255-BLOCK:F=ZR
6080 A=DIR*128+ADR(BF$)
6090 FOR I=0 TO 127:X=PEEK(A+I):PUT #MD,X:F=F+X:NEXT I
6100 F=ASC(CHR$(F)):PUT #MD,F:GOSUB GC:IF X=21 THEN 6120
6110 T=10
6120 NEXT T:DIR=DIR+WON:BLOCK=BLOCK+WON
6130 IF X<>6 THEN 6300
6140 F=(DIR+1)*128:T=LEN(BF$):IF F<=T THEN 6070
6150 IF BFLAG THEN 6200
6160 IF F=T THEN BF$="":GOTO 6180
6170 BF$=BF$(DIR*128+1,T)
6180 GOSUB CLM:GOSUB 5920:IF BFLAG THEN GOSUB 5950
6190 GOSUB OPM:GOTO 6060
6200 PUT #MD,4:GOTO 6350
6300 ? #MD:? #MD;"* ABORTED *"
6350 DIR=0:AMODE=AM:LMODE=LM:GOTO MN
6500 AM=AMODE:LM=LMODE:AMODE=32:LMODE=64:L=21:NSEC=ZR
6510 TRAP 6700:A$(131)=" ":A=ADR(A$):GOSUB CLM:OPEN #FI,C8,ZR,FL$
6520 GOSUB OPM:? #MD;"FILE: ";FL$;" Ready to Receive,":? #MD;"^X to Cancel"
6530 FOR T=WON TO 10:TOUT=0:? CHR$(L);
6540 STATUS #MD,X:IF PEEK(747) THEN GET #MD,X:GOTO 6540
6550 PUT #MD,L:L=6:GET #MD,SOH:F=SOH:IF SOH<>WON THEN 6620
6560 FOR I=0 TO 130
6570 STATUS #MD,X:IF PEEK(747) THEN GET #MD,X:POKE A+I,X:F=F+X:NEXT I:GOTO 6600
6580 TOUT=TOUT+1:IF TOUT<100 THEN 6570
6590 GOTO 6610
6600 F=F-X:F=ASC(CHR$(F)):IF X=F THEN BF$(LEN(BF$)+1)=A$(3,130):NSEC=NSEC+WON:GOTO 6620
6610 FOR L=WON TO 200:NEXT L:GOSUB OPM:L=21:TRAP 6700:GOTO 6630
6620 T=10
6630 NEXT T:IF SOH=WON AND L=6 AND NSEC<NUMSECT THEN 6530
6640 IF NSEC=NUMSECT THEN TRAP 8080:GOSUB CLM:? #FI;BF$;:NSEC=ZR:BF$="":GOSUB OPM:TRAP 6700:GOTO 6530
6650 IF SOH=4 AND L=6 THEN 6800
6700 TRAP 8080:? #MD:? #MD;"* ABORTED *"
6710 GOSUB CLM:CLOSE #FI:GOTO 6890
6800 PUT #MD,L:GOSUB OPM:? #MD:? #MD;"* SAVING FILE *"
6810 TRAP 8080:GOSUB CLM:IF LEN(BF$)=ZR THEN CLOSE #FI:GOTO 6890
6820 X$=BF$(LEN(BF$)):X=ASC(X$)
6830 FOR I=LEN(BF$)-127+X TO LEN(BF$):IF BF$(I,I)<>X$ THEN X=128
6840 NEXT I
6860 ? #FI;BF$(WON,LEN(BF$)-128+X);:CLOSE #FI
6890 I=WON:T=WON:AMODE=AM:LMODE=LM:GOTO MN
7000 MSGNO$="0000":MSGNO$(5-LEN(STR$(FROM)))=STR$(FROM)
7020 T=INT(LEN(BF$)/C40):F=INT(T*0.5+0.5):Y=F
7040 FOR X=WON TO CLOG(T+MD)/CLOG(MD)
7060 TSS=F*C40:Y=INT(Y*0.5+0.5)
7070 IF MSGNO$>BF$(TSS-39,TSS-36) THEN F=F+Y+Y
7080 F=F-Y:IF F<WON THEN F=WON
7100 IF F>T THEN F=T
7120 NEXT X:TSS=F*C40
7130 IF MSGNO$>BF$(TSS-39,TSS-36) AND DIR=WON THEN F=F+WON
7140 IF MSGNO$<BF$(TSS-39,TSS-36) AND DIR=-WON THEN F=F-WON
7150 RETURN 
8000 GOSUB CLM:CLOSE #FI:OPEN #FI,4,ZR,FL$:IF TST=WON THEN POINT #FI,DSEC,DBIT
8010 TST=ZR:TRAP 8070:A$(C255)=" ":BF$=""
8020 XIO 7,#FI,4,ZR,A$:BF$(LEN(BF$)+WON)=A$:NOTE #FI,DSEC,DBIT:GOTO 8020
8070 TRAP 8075:IF PEEK(856) THEN BF$(LEN(BF$)+WON)=A$(WON,PEEK(856))
8075 IF PEEK(195)=5 THEN POKE 195,WON:TST=WON:CLOSE #FI:GOTO 8100
8076 IF PEEK(195)<>5 THEN TST=ZR
8080 IF PEEK(195)<>136 AND PEEK(195)<>139 THEN 8120
8085 TRAP C8080:IF PEEK(195)=139 THEN POP :POP :POP :POP :POKE 195,WON:TST=ZR:GOTO WR
8100 IF LOCAL THEN RETURN 
8105 GOTO OPM
8120 ERR=PEEK(195):GOSUB CLM:CLOSE #FI
8130 LPRINT "Error- ";ERR;" LINE # ";C256*PEEK(187)+PEEK(186)
8140 GOSUB OPM
8150 ? #MD;CR$;"SYSTEM ERROR --- TRY AGAIN."
8160 POP :POP :POP :GOTO MN
9000 X=ZR:FROM=X:TU=X:DIR=X:IF L$="" THEN RETURN 
9020 X=X+WON:IF X>LEN(L$) THEN 9100
9030 IF L$(X,X)>="0" AND L$(X,X)<="9" THEN FROM=FROM*10+VAL(L$(X,X)):GOTO 9020
9040 IF L$(X,X)<>"-" THEN 9100
9050 X=X+WON:IF X>LEN(L$) THEN 9300
9060 IF L$(X,X)>="0" AND L$(X,X)<="9" THEN TU=TU*10+VAL(L$(X,X)):GOTO 9050
9070 GOTO 9300
9100 TU=FROM:DIR=WON:GOTO 9400
9300 DIR=WON:IF TU<FROM THEN DIR=-WON
9400 IF X<LEN(L$) THEN L$=L$(X+WON):RETURN 
9420 L$="":RETURN 
10000 LOCAL=ZR:BEL$=CHR$(253):DEL$=CHR$(156):BS$=CHR$(126):NAM$=" "
10001 POKE 54018,60:FOR X=1 TO 100:NEXT X
10010 AMODE=32:LMODE=ZR:XMODE=ZR:GOSUB 1530
10020 GRAPHICS ZR:? :? 
10130 POKE 752,WON:POKE 77,ZR
10150 GOSUB C2010:POSITION MD,5:? "TIME: ";TM$;"   DATE: ";TD$:IF PEEK(53279)=3 THEN 28000
10151 IF STRIG(0)=0 THEN POKE 54018,52:TRAP C8080:GOSUB OPM:FOR X=WON TO 50:NEXT X:GOTO 400
10160 IF PEEK(53279)=5 THEN GOSUB CLM:LOCAL=WON:OPEN #MD,13,ZR,"E:":GOTO 400
10161 IF PEEK(53279)=6 THEN GOTO 11000
10162 GOTO 10150
10170 STATUS #MD,X:IF PEEK(747) THEN GOSUB 10900:? CHR$(253);:GOTO 11000
10180 GOTO 10150
10900 STATUS #MD,X:IF PEEK(747) THEN GOSUB GC:GOTO 10900
10910 ? CHR$(253);:RETURN 
11000 ? "ANSWERING CALL"
11010 TOUT=1900
11020 STATUS #MD,X:IF PEEK(747)=0 THEN 11020
11030 STATUS #MD,X:IF PEEK(747)=0 THEN TRAP C8080:GOSUB OPM:GOTO 400
11040 GOSUB GC:IF X=73 THEN GOTO WR
11050 GOTO 11030
12000 ? #MD;"CHAT MODE, I'll get the SYSOP....."
12010 ? NAM$;" FROM ";AD$:? :? "PRESS SELECT TO BEGIN, ^X TO END"
12020 FOR F=WON TO 5:? CHR$(253):FOR T=WON TO 50:NEXT T:IF PEEK(CON)=5 THEN 13000
12030 NEXT F:FOR X=WON TO 500:IF PEEK(CON)=5 THEN 13000
12040 NEXT X:? #MD;"SYSOP is not here":GOTO MN
13000 GOSUB CLM:? "IN CHAT MODE":GOSUB OPM:? #MD;"CHAT MODE: PRESS CTRL<X> TO END":POKE 764,C255
13010 STATUS #MD,X:IF PEEK(747)=0 THEN 13040
13020 GET #MD,X:PUT #MD,X:IF X=8 THEN X=126
13030 ? CHR$(X);:IF X=C24 THEN GOSUB CLM:GOTO MN
13040 IF PEEK(764)=C255 THEN 13010
13050 GET #3,X:IF X=126 THEN ? #MD;BS$;" ";BS$;:GOTO 13030
13051 IF X=10 THEN ? #MD;"Jeff Bell,784 Concord Ave #1,San Jose,CA,95128":GOTO 13030
13052 IF X=19 THEN ? #MD;"Yes...may I help you?":GOTO 13030
13053 IF X=20 THEN TXA=TXA+10:GOTO 13030
13054 IF X=6 THEN ? #MD;"Ya?  Well same to you!":GOTO 13030
13055 IF X=25 THEN ? #MD;"You rang?":GOTO 13030
13060 PUT #MD,X:GOTO 13030
28000 GOSUB 28100:POKE 752,ZR
28020 END 
28100 GOSUB CLM:CLOSE #FI:OPEN #FI,C8,ZR,"D:CONFIG"
28110 ? #FI;CSECT;CR$;CBYTE;CR$;CALLNO;CR$;MSECT;CR$;MBYTE;CR$;MSGS;CR$;HMSG:CLOSE #FI
28120 RETURN 
29000 ZR=0:WON=1:MD=2:C8=8:CON=53279:FI=WON:C24=24:C128=128:C256=256:C2010=2010:C8080=8080:C255=255
29001 C40=40:C120=120:C121=121:C127=127:C119=119
29002 DIM L$(C120),FL$(16),NAM$(C40),AD$(C40),FC$(22),MTH$(C24),CTRL$(C40)
29010 DIM MSG$(2100),PRMT$(50),FROM$(C40),FR$(C40),MSGNO$(4),SUBJ$(33),TP$(C120),PAS$(C40),A$(C255)
29020 DIM SCIO$(7):SCIO$="hhh*LVd":C900=900
29025 DSEC=ZR:DBIT=ZR:TST=ZR
29030 DIM CR$(WON),BEL$(WON),DEL$(WON),BS$(WON),X$(WON),TD$(C8),DATE$(C8),TM$(C8)
29035 DIM MIF$(14),MDF$(14),CDF$(14):READ MIF$,MDF$,CDF$
29040 MTH$="312831303130313130313031"
29045 CTRL$="/CTRL=^/ ^S PAUSE, ^Q RESUME, ^X QUIT"
29050 ? CHR$(125):? "B.B.S."
29060 GOSUB 1530:? "Enter date mm/dd/yy ";:INPUT TD$
29070 ? "Enter time hh:mm:ss ";:INPUT TM$:? "Work disk in ";:INPUT A$
29075 POKE 54017,128:REM RELAY OFF
29100 CLM=292:OPM=310:MN=1000:GL=70:LET GC=20:PR=9000:WR=10000:RF=8000:SR=7000
29110 FC$="B?HWXQTREYGLACSKDUFIJM":CR$=CHR$(155)
29120 OPEN #FI,4,ZR,"D:CONFIG"
29130 INPUT #FI,CSECT,CBYTE,CALLNO,MSECT,MBYTE,MSGS,HMSG:CLOSE #FI
29140 X=FRE(ZR)-100:LPRINT "BUFF = ";X:DIM BF$(X):MAXM=INT(X/40):NUMSECT=8:REM INT(X/128)-WON
29160 X=((VAL(TM$(WON,MD))*60+VAL(TM$(4,5)))*60+VAL(TM$(7,C8)))*60
29170 F=INT(X/65536):T=INT(X/C256)-F*C256
29180 X=X-F*65536-T*C256:POKE 20,X:POKE 19,T:POKE 18,F
29187 POKE 54019,48
29188 POKE 54017,128
29189 POKE 54019,52
29190 OPEN #3,4,0,"K:":GOTO WR
30000 DATA D1:MESSAGE.ISM,D1:MESSAGE.DAT,D1:CALLERS.DAT
30200 ? #MD;"Watch this spot for a new option soon!":GOTO MN
30400 POKE 54017,0:FOR X=WON TO 100:NEXT X:IF STRIG(1)=1 THEN POKE 54017,128:FOR X=WON TO 100:NEXT X:GOTO MN
30401 GOTO 30400
30500 ? #MD;"Watch this spot for a new option soon!":GOTO MN
```

# ELEC323 / 마이크로프로세서 / 2019년도 / 1학기 / 김종화
## 어셈블리어 코딩을 통해 프로그램을 작성한다. 그리고 EDSIM51 가상 시뮬레이터를 통해 프로그램 동작을 검증한다.

![image](https://user-images.githubusercontent.com/58419421/99379121-9c625080-290b-11eb-9c22-7526c9e4f4ed.png)
![image](https://user-images.githubusercontent.com/58419421/99379904-99b42b00-290c-11eb-824d-0301ad0387b6.png)
시간지연 만들기
1초의 시간지연을 만들기 위해서 아래와 같이 작은 단위의 시간지연부터 만들고 이를 다시 다음 시간지연 루틴에서 불
러들여서 결국 1초의 시간지연을 만들 수 있다. D_10US: ; 20*(12/CLK) = 10US
NOP ; 1 cycle
NOP ; 1 cycle ㆍㆍㆍㆍㆍㆍㆍㆍ
이렇게 NOP 명령어를 반복하여 10US에 해당하는 시간지연을 만들고
D_50US:
CALL D_10US을 5번 수행하면 된다, 이 행위의 반복을 통해 1S의 시간지연을 만들 수 있다.

1) 소스코드
ORG 0000H ; 프로그램이 저장 될 주소값
JMP MAIN ; 프로그램이 시작되면 MAIN 으로 점프
ORG 0030H ; 프로그램이 저장 될 주소값
MAIN: MOV SP, #60H ; 내부 램 60H에 스택 설정
WHILE1: MOV R7, #000H
FOR11: MOV A, R7
 MOV DPTR, #fnd_data
 MOVC A, @A+DPTR
 MOV P0, A
 MOV P1, A
 MOV P2, A
 CALL DELAY
 INC R7
 CJNE R7, #010H,FOR11
 SJMP WHILE1
; 시간지연 함수
DELAY: MOV R0, #80H ; R0 레지스터에 16진수 90H 값을 저장
LOOP1: MOV R1, #00H ; R1 레지스터에 16진수 00H 값을 저장
LOOP2: NOP ; 아무 일도 수행하지 않는 명령어
 DJNZ R1, LOOP2 ; R1=R1-1이 0이 아니면 –-> LOOP2로 점프, 0이면 다음 명령어
 DJNZ R0, LOOP1 ; R0=R0-1이 0이 아니면 –-> LOOP1로 점프, 0이면 다음 명령어
 RET ; 함수를 호출한 곳으로 RETURN
fnd_data: ; FND 데이터 0~9
 DB 0C0H, 0F9H, 0A4H, 0B0H, 099H, 092H, 082H, 0F8H, 080H, 090H
 END ; 프로그램을 종료
![image](https://user-images.githubusercontent.com/58419421/99380169-edbf0f80-290c-11eb-9c0c-500af6eaf6bf.png)
3) 분석 및 고찰
0에서 9까지의 UP COUNTER를 만들었습니다. FND(7 SEGMENT)를 포트에 직접 연결하여 코딩하였습니다. 최초에 ORG 0000H –--> 프로그램이 저장 될 주소 값을 설정하고
JMP MAIN –--> 프로그램이 시작되면 MAIN 으로 점프합니다. ORG 0030H ---> MAIN LABLE이 끝났을 때의 주소값
MAIN: MOV SP, #60H ; 내부 램 60H에 스택 설정
CALL DELAY에서 DELAY LABLE을 호출하여 시간을 지연 시킵니다. DELAY 레이블은 다음과 같습니다. DELAY: MOV R0, #80H ; R0 레지스터에 16진수 90H 값을 저장
LOOP1: MOV R1, #00H ; R1 레지스터에 16진수 00H 값을 저장
LOOP2: NOP ; 아무 일도 수행하지 않는 명령어
 DJNZ R1, LOOP2 ; R1=R1-1이 0이 아니면 –-> LOOP2로 점프, 0이면 다음 명령어
 DJNZ R0, LOOP1 ; R0=R0-1이 0이 아니면 –-> LOOP1로 점프, 0이면 다음 명령어
 RET ; 함수를 호출한 곳으로 RETURN
fnd_data: ; FND 데이터 0~9
 DB 0C0H, 0F9H, 0A4H, 0B0H, 099H, 092H, 082H, 0F8H, 080H, 090H
 END ; 프로그램을 종료

 ![image](https://user-images.githubusercontent.com/58419421/99379154-a71ce580-290b-11eb-8bbc-a624c03867d9.png)
 1) 설계
포트에 직접 연결한 FND 테스트 프로그램 ==> 각 FND 마다 0~16 까지의 UP COUNTER 설계
2) 소스코드 – C
#include <reg51.h>             
code unsigned char fnd_data[16] = {0xC0, 0xF9, 0xA4, 0xB0, 0x99, 0x92, 0x82, 0xF8,
                                    0x80, 0x90, 0x88, 0x83, 0xC6, 0xA1, 0x86, 0x82};
void delay(unsigned int d_cnt)
{
      while(d_cnt--);         
}
void main(void)                  
{
      unsigned char i;
      while(1)
      {
              for(i=0; i<16; i++)
              {
                      P0 = fnd_data[i]; 
                      P1 = fnd_data[i];
                      P2 = fnd_data[i]; 
                      delay(0x8000);
              }
      }
}
3) 소스코드 – 어셈블리어      
               ORG 0000H        
               JMP MAIN                                  
       ORG 0030H                           
MAIN:  MOV SP, #60H                      
WHILE1: MOV     R7, #000H
FOR11:  MOV     A, R7
        MOV     DPTR, #fnd_data
        MOVC    A, @A+DPTR
        MOV     P0, A
        MOV     P1, A
        MOV     P2, A
        CALL   DELAY
        INC     R7
        CJNE    R7, #010H,FOR11
        SJMP    WHILE1
DELAY:   MOV R0, #80H          
LOOP1:   MOV R1, #00H           
LOOP2:   NOP                       
              DJNZ R1, LOOP2        
              DJNZ R0, LOOP1        
              RET                           
fnd_data:                               
        DB  0C0H, 0F9H, 0A4H, 0B0H, 099H, 092H, 082H, 0F8H, 080H, 090H
        DB  088H, 083H, 0C6H, 0A1H, 086H, 082H
          END                       
4) 동작 (프로그램 실행 캡쳐화면, 프로그램 실행 영상은 파일 첨부 하였음.)
==> 어셈블리어로 코딩한 프로그램 동작 샘플
![image](https://user-images.githubusercontent.com/58419421/99379562-33c7a380-290c-11eb-8f20-9c527bfcbdf5.png)
5) 분석 및 고찰
1. C 코딩 분석
#include <reg51.h> ==> 레지스터 변수를 위한 헤더파일
code unsigned char fnd_data[16] = {0xC0, 0xF9, 0xA4, 0xB0, 0x99, 0x92, 0x82, 0xF8,
                                    0x80, 0x90, 0x88, 0x83, 0xC6, 0xA1, 0x86, 0x82};
==> 애노드 공통 FND를 이용하여 16진수를 표시
void delay(unsigned int d_cnt)
{
      while(d_cnt--);  ==> 입력된 숫자에서 1씩 감소하면서 0이 되면 while() 빠져나감
}
==> 시간 지연을 위한 함수, 입력되는 숫자가 지연 시간 결정  
for(i=0; i<16; i++) ==> 0~15까지 숫자를 증가하면서 FND에 표시
2. 어셈블리어 코딩 분석
시간 지연을 위한 함수, 입력되는 숫자가 지연 시간 결정 ==>
DELAY:   MOV R0, #80H            ==> R0 레지스터에 16진수 90H 값을 저장한다. LOOP1:   MOV R1, #00H            ==> R1 레지스터에 16진수 00H 값을 저장한다. LOOP2:   NOP                          
              DJNZ R1, LOOP2         ==> R1=R1-1을 해서 0이 아니면 LOOP2로 점프, 0이면 다음 명령어 
              DJNZ R0, LOOP1         ==> R0=R0-1을 해서 0이아니면 LOOP1로 점프, 0이면 다음 명령어
              RET                            ==> 함수를 호출한 곳으로 돌아 간다. fnd_data:                                   ==> FND 데이터 0~9, A~F
        DB  0C0H, 0F9H, 0A4H, 0B0H, 099H, 092H, 082H, 0F8H, 080H, 090H
        DB  088H, 083H, 0C6H, 0A1H, 086H, 082H
          END        



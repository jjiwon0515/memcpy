# memcpy

```c
// compiled with : gcc -o memcpy memcpy.c -m32 -lm  --> 아마도 컴파일을 직접하라는 뜻인거 같다.
#include <stdio.h>
#include <string.h>
#include <stdlib.h>
#include <signal.h>
#include <unistd.h>
#include <sys/mman.h>
#include <math.h>


//Time Stamp Counter를 가져오는 명령어이다. -> TSC란? 아마도 레지스터인 거 같다. 근데 64비트 레지스터라고 뜬다.   *의문점 : 해당 코드는 32비트로 컴파일한건데 64비트 레지스터...? 모르겠다 더 알아보자
//상위 32비트의 자료를 EDX 레지스터에 로드하고, 하위 32비트의 자료를 EAX에 로드한다.
unsigned long long rdtsc(){
        asm("rdtsc");    
}



//char * slow_memcpy : 
char* slow_memcpy(char* dest, const char* src, size_t len){
        int i;
        for (i=0; i<len; i++) {
                dest[i] = src[i];
        }
        return dest;
}






char* fast_memcpy(char* dest, const char* src, size_t len){
        size_t i;
        // 64-byte block fast copy
        if(len >= 64){
                i = len / 64;
                len &= (64-1);
                while(i-- > 0){
                        __asm__ __volatile__ (
                        "movdqa (%0), %%xmm0\n"
                        "movdqa 16(%0), %%xmm1\n"
                        "movdqa 32(%0), %%xmm2\n"
                        "movdqa 48(%0), %%xmm3\n"
                        "movntps %%xmm0, (%1)\n"
                        "movntps %%xmm1, 16(%1)\n"
                        "movntps %%xmm2, 32(%1)\n"
                        "movntps %%xmm3, 48(%1)\n"
                        ::"r"(src),"r"(dest):"memory");
                        dest += 64;
                        src += 64;
                }
        }

        // byte-to-byte slow copy
        if(len) slow_memcpy(dest, src, len);
        return dest;
}





//위에 있는 gcc -o memcpy memcpy.c -m32 -lm 명령어의 결과가 nc 0 9022에 있는 것 같다. 
//옵션 -m32와 -lm이 의미하는 것은 무엇일까?
//m32 : 32비트로 컴파일하는 명령어 옵션이다. -> 굳이 32비트로 한 이유가 있을까? -> 특징?
//lm : 수학라이브러리를 포함시키는 옵션이다. -> <math.h>는 여러 수학 함수들을 포함하는 C언어의 표준 라이러리(해당 코드는 math.h를 포함하고 있다.)
int main(void){

        setvbuf(stdout, 0, _IONBF, 0);
        setvbuf(stdin, 0, _IOLBF, 0);

        printf("Hey, I have a boring assignment for CS class.. :(\n");
        printf("The assignment is simple.\n");

        printf("-----------------------------------------------------\n");
        printf("- What is the best implementation of memcpy?        -\n");
        printf("- 1. implement your own slow/fast version of memcpy -\n");
        printf("- 2. compare them with various size of data         -\n");
        printf("- 3. conclude your experiment and submit report     -\n");
        printf("-----------------------------------------------------\n");

        printf("This time, just help me out with my experiment and get flag\n");
        printf("No fancy hacking, I promise :D\n");

        unsigned long long t1, t2;
        int e;
        char* src;  //포인터 변수 src 선언
        char* dest; //포인터 변수 dest 선언
        unsigned int low, high;
        unsigned int size;
        
        
        
        
        // 메모리 맵핑을 하는 함수 -> 메모리 맵핑 : 파일을 프로세스의 메모리에 맵핑
        // --> * 따라서 read나 write 함수를 사용하지 않아도 프로그램 내부에서 정의한 변수를 사용해 데이터 조작가능
        // void *mmap(void *addr, size_t len, int prot , int flags, int fildes, off_t off)
        // addr(맵핑할 메모리 주소-> 맵핑하려는 주소 직접 지정), len(메모리 공간의 크기-> 맵핑할 메모리 공간의 크기 지정), prot(보호 모드-> 맵핑한 데이터를 읽기만 할지, 쓰기나 실행도 허용할지 등 보호 ), flags(맵핑된 데이터의 처리 방법을 지정하는 상수), fildes(파일 기술자), off(파일 오프셋)
        // MAP_PRIVATE : 다른 사용자와 데이터의 변경 내용을 공유하지 않는다. 최초의 쓰기 동작에서 맵핑된  메모리의 사본을 새로 만들고 맵핑 주소는 새롭게 만들어진 사본을 가르킨다.
        // MAP_ANONYMOUS :  ?
        
        // allocate memory
        char* cache1 = mmap(0, 0x4000, 7, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0);
        char* cache2 = mmap(0, 0x4000, 7, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0);
        src = mmap(0, 0x2000, 7, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0)
        size_t sizes[10];
        int i=0;


        // pow 함수 : Ex) pow(2,3) = 2 ^ 3 = 8
        // setup experiment parameters
        for(e=4; e<14; e++){    
                //해당 코드를 실행했을 때 8~16, 16~32...부분
                low = pow(2,e-1);
                high = pow(2,e);
                printf("specify the memcpy amount between %d ~ %d : ", low, high);
                
                scanf("%d", &size);
                // high < size < low 만약 이런 형태로 입력값을 넣어버리면 경고 메시지와 함께 프로그램이 종료                 // 된다.
                if( size < low || size > high ){
                        printf("don't mess with the experiment.\n");
                        exit(0);
                }
                
                
                // ?
                sizes[i++] = size;
        }

        sleep(1);
        printf("ok, lets run the experiment with your configuration\n");
        sleep(1);

        // run experiment
        for(i=0; i<10; i++){
                //i =0~10
                //sizes[0] == 8, sizes[1] == 16, size[2] == 32....
                size = sizes[i];
                printf("experiment %d : memcpy with buffer size %d\n", i+1, size);
                
                
                dest = malloc( size );
                
                
                
                
                // memcpy 함수(memory + copy) : void* memcpy(void* dest, const void* source, size_t num)
                // 첫 번째 인자(void* dest) : 복사 받을 메모리를 가리키는 포인터
                // 두 번째 인자(const void* source) : 복사할 메모리를 가리키는 포인터 
                // 세 번째 인자(size_t num) : 복사할 데이터(값)의 길이(바이트 단위)
                // 결론 : 두 번째 인자에 있는 포인터가 가리키는 메모리값에 있는 데이터를 세 번째 인자만큼 
                // 복사해서 첫 번째 인자에 있는 포인터가 가리키는 메모리값에 저장한다.
                
                
                // 캐싱이란? : "캐시" 라는 빠른 메모리 영역으로 데이터를 가져와서 접근하는 방식
                // Ex) 속도가 느린 하드디스크의 데이터를 메모리로 가지고 와서 메모리 상에서 읽기 쓰기를
                // 수행하는 것을 '데이터를 메모리에 캐싱한다'라고 한다.
             
                
                #slow_memcpy
                memcpy(cache1, cache2, 0x4000);         // to eliminate cache effect 
                                                        // 캐시효과란?
                t1 = rdtsc();
                slow_memcpy(dest, src, size);           // byte-to-byte memcpy
                                                        // int i;
                                                             for (i=0; i<len; i++) {
                                                             dest[i] = src[i];
                                                           }
                                                           return dest;
                t2 = rdtsc();
                printf("ellapsed CPU cycles for slow_memcpy : %llu\n", t2-t1);
                
                
                
                
                
                #fast_memcpy
                memcpy(cache1, cache2, 0x4000);         // to eliminate cache effect
                t1 = rdtsc();
                fast_memcpy(dest, src, size);           // block-to-block memcpy
                t2 = rdtsc();
                printf("ellapsed CPU cycles for fast_memcpy : %llu\n", t2-t1);
                printf("\n");
        }

        printf("thanks for helping my experiment!\n");
        // 이 소스코드를 지워라.... -> 위를 다시보면 mmap함수를 통해서 데이터 입출력이 가능하다고 했다.
        // 그 점을 이용하면 문제를 해결할 수 있을 것 같다.
        printf("flag : ----- erased in this source code -----\n");
        return 0;
}

* 테스트로 입력값들을 넣었을 때는 5에서 끝났는데 해당 코드에서는 10까지 실행을 할 수 있도록 프로그래밍 되어있다. 이 점도 찝찝하다.


<cat readme> 
the compiled binary of "memcpy.c" source code (with real flag) will be executed under memcpy_pwn privilege if you connect to port 9022.
execute the binary by connecting to daemon(nc 0 9022).-->내가 포트번호 9022로 접속하면 코드가 실행될거라고 한다. nc 0 9022는 뭐지? -> nc 명령어는 뭐지? -> 사용법을 찾아보자 



<nc 0 9022를 실행했을 때>
memcpy@pwnable:~$ nc 0 9022
Hey, I have a boring assignment for CS class.. :( --> CS : Computer Science?
The assignment is simple.
-----------------------------------------------------
- What is the best implementation of memcpy?        -
- 1. implement your own slow/fast version of memcpy -
- 2. compare them with various size of data         -
- 3. conclude your experiment and submit report     -
-----------------------------------------------------
This time, just help me out with my experiment and get flag
No fancy hacking, I promise :D
specify the memcpy amount between 8 ~ 16 : 


specify the memcpy amount between 8 ~ 16 : 8
specify the memcpy amount between 16 ~ 32 : 16
specify the memcpy amount between 32 ~ 64 : 32
specify the memcpy amount between 64 ~ 128 : 64
specify the memcpy amount between 128 ~ 256 : 128
specify the memcpy amount between 256 ~ 512 : 256
specify the memcpy amount between 512 ~ 1024 : 512
specify the memcpy amount between 1024 ~ 2048 : 1024
specify the memcpy amount between 2048 ~ 4096 : 2048
specify the memcpy amount between 4096 ~ 8192 : 4096
ok, lets run the experiment with your configuration
experiment 1 : memcpy with buffer size 8
ellapsed CPU cycles for slow_memcpy : 5768
ellapsed CPU cycles for fast_memcpy : 512

experiment 2 : memcpy with buffer size 16
ellapsed CPU cycles for slow_memcpy : 568
ellapsed CPU cycles for fast_memcpy : 428

experiment 3 : memcpy with buffer size 32
ellapsed CPU cycles for slow_memcpy : 840
ellapsed CPU cycles for fast_memcpy : 672

experiment 4 : memcpy with buffer size 64
ellapsed CPU cycles for slow_memcpy : 1156
ellapsed CPU cycles for fast_memcpy : 328

experiment 5 : memcpy with buffer size 128
ellapsed CPU cycles for slow_memcpy : 2236
--> ?

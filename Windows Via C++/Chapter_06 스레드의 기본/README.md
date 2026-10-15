# 쓰레드
* 프로세스 내부 실제 실행의 단위
* 한 프로세스에 여러개 존재가능
* 프로세스의 가상주소공간 내부에서 독립적인 본인의 스택을 가지고 나머지는 다른 스레드와 공유

# 쓰레드 커널오브젝트
* 운영체제가 내부적으로 사용하는 쓰레드 관련 데이터 구조체
* cpu에 올라갔을때 필요한 정보 = 컨텍스트를 가지고있음. sp, ip, bp등 여러 레지스터 값
* 사용카운트, 정지카운트, 종료코드, 시그널 상태
* 종료코드는 실행중일 때에는 STILL_ACTIVE 상수값. 종료되면 스레드함수의 리턴값
* 쓰레드 생성 시 ip레지스터 값에 RtlUserThreadStart() 함수를 가리키고 있음

# CreateThread() WinAPI 제공
* 운영체제 내부적으로 사용하는 쓰레드 커널오브젝트를 생성
* 쓰레드 컨텍스트 구조체 생성
* 쓰레드가 실행할 함수와 인자, 스택사이즈를 지정할 수 있음

### CreateThread() 로 쓰레드를 생성한 경우
```text
쓰레드를 생성할 때
CreateThread(); -> 쓰레드 컨텍스트 및 커널오브젝트 생성

=========≈===============================

쓰레드가 스케쥴링 되어 cpu에서 실행될 때
RtlUserThreadStart();
    |
ThreadFunc(void* param); -> ptd 개념이 없음
    |
ExitThread();
```

# _beginthreadex() CRT 제공
* CRT가 사용하는 여러 전역데이터들을 쓰레드단위로 독립적으로 지원하기 위해 만들어진 쓰레드 생성 함수 errno, strtok의 token 등..
* ptd(per thread data)라는 구조체를 CRT힙에 생성하여 쓰레드마다 본인의 TLS에 이어줌.
* _endthreadex()와 짝으로 사용하여 c++ 객체 소멸자 호출과 ptd 삭제
* C++ 에서는 필수적으로 CreateThread()를 대체하여 사용

### _beginthreadex() 로 쓰레드를 생성한 경우
```text
쓰레드를 생성할 때
_beginthreadex(); -> crt힙에 ptd할당 및 CreatThread() 호출
    |
CreateThread(); 쓰레드 컨텍스트 및 커널오브젝트 생성

============================================

쓰레드가 스케쥴링 되어 cpu에서 실행될 때
RtlUserThreadStart();
    |
_threadstartex(); -> ptd를 본인의 tls와 이어줌
    |
_callthreadstartex(); -> 실제 쓰레드함수 호출
    |
ThreadFunc();
    |
_endthreadex(); -> ptd 해제 및 C++ 객체 정리
    |
ExitThread();
```

# GetCurrentProcess(), GetCurrentThread()
* 현재 프로세스, 쓰레드의 pseudo handle 반환. 프로세스 핸들테이블에 등록되지 않은 가짜핸들
* DuplicateHandle() 함수를 통해 pseudo handle을 실제 핸들로 복제 가능. 프로세스 핸들테이블에 등록됨.
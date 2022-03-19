# Context Switching

### Context Switching이란

- 기존 프로세스의 상태 또는 레지스터 값을 저장하고 CPU가 다음 프로세스를 수행하도록 새로운 프로세스의 상태 또는 레지스터 값을 교체하는 작업
- CPU 실행 중 어느 일정 지점에서의 CPU 전체 상황을 Context라 한다.

### 언제 Context Switching이 발생할까

- Interrupts
    - time out
    - I/O interrupt
    - memory fault
- Trap
    - program fault 때문에 더 진행을 할 수 없을 때
- System call
    - 프로그램이 의도한 상황
    - 어떤 요청인지에 따라 swithcing이 발생하지 않을 수도 있다.
        - switching이 발생할 수도, 발생하지 않을 수도 있다.
- Interrupt와 Trap은 프로그램이 의도한 switching이 아니다.

### Context Switching이 필요한 이유

- 하나의 task만 처리할 수 있다면 해당 task가 끝날 때까지 기다려야 함
    - 자원 낭비일 뿐만 아니라 사용자 응답성이 좋지 않음
    - ex) I/O interrupt가 발생하면 입출력이 완료될 때까지 기다려야 함
- CPU가 task를 바꿔가며 실행하기 위해 Context Switching이 필요

### Context Switching의 비용

- Context Switching에는 많은 비용이 소요된다.
    - 캐시 초기화
    - 메모리 매핑 초기화
    - 메모리에 접근해야 하기 때문에 kernel이 실행됨
- Thread보다 Process에서 Context Switching 비용이 더 많이 든다.
    - Thread는 스택 메모리를 제외한 모든 메모리를 공유하기 때문에 switching 시 스택 메모리만 변경하면 된다.

### Context Switching 진행 과정

1. 중단된 지점의 CPU 레지스터 값을 저장
2. 현재 running 상태에 있던 프로세스의 PCB 값을 업데이트
    - 프로세스 상태와 왜 상태가 바뀌었는지
    - 지금까지 사용하고 있던 자원들에 대한 정보
3. 해당 프로세스의 PCB를 적당한 큐로 옮김
    - blocked나 ready큐로
4. 다음에 실행할 프로그램 선택
    - 스케쥴링 우선순위에 따라 정함
5. 실행할(선택된) 프로세스의 PCB 값 업데이트
    - ex) ready → running
6. 메모리 관리에 관한 data structure 업데이트
    - 메모리 테이블에는 현재 실행중인 프로세스에 대한 정보만 관리
    - 프로세스가 바뀌면 그 프로세스에 대한 정보로 업데이트
7. 프로세스에 저장된 context restore
    - 중단된 시점부터 다시 실행시켜야 함
    - restore 하는 순간 새로운 프로세스 실행 시작
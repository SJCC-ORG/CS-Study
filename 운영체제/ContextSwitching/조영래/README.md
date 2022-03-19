# Context Switching

> CPU 가 이전의 프로세스 상태를 PCB 에  보관하고, 또 다른 프로세스의 정보를 PCB 에 읽어 레지스터에 적재하는 과정

보통 인터럽트가 발생하거나, 실행 중인 CPU 사용 허가시간을 모두 소모하거나, 입출력을 위해 대기해야 하는 경우에 Context Switching이 발생

즉, 프로세스가 Ready → Running, Running → Ready, Running → Waiting처럼 상태 변경 시 발생

## Context Switching 시나리오

1. P0 프로세스가 인터럽트되면서 PCB0에 P0 프로세스의 상태 정보를 저장합니다.
2. 다음 수행할 P1 프로세스의 PCB1에서 P1 프로세스의 상태 정보가 CPU에 재로딩됩니다.
3. P1 프로세스를 일정 시간 수행합니다.
4. P1 프로세스가 인터럽트되면서 PCB1에 P1 프로세스의 상태 정보를 저장합니다.
5. 다음 수행할 P0 프로세스의 PCB0에서 P0 프로세스의 상태 정보가 CPU에 재로딩됩니다.
6. P0 프로세스를 일정 시간 수행합니다.

## Context Switching의 OverHead

> 프로세스 작업 중 OverHead를 감수해야 하는 상황

프로세스를 수행하다가 입출력 이벤트가 발생해서 대기 상태로 전환시킴.
이때, CPU를 그냥 놀게 놔두는 것보다 다른 프로세스를 수행시키는 것이 효율적.

다른 프로세스를 수행시키려면 Context Switching 이 발생하는 Overhead 를 감수

## Reference

- <https://github.com/gyoogle/tech-interview-for-developer/blob/master/Computer%20Science/Operating%20System/PCB%20%26%20Context%20Switcing.md>
- <https://junhyunny.github.io/information/operating-system/process-control-block-and-context-switching/>
- <https://jhnyang.tistory.com/33>
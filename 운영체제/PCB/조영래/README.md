# PCB (Process Controll Block)

## Process Management

> CPU가 프로세스가 여러개일 때, CPU 스케줄링을 통해 관리하는 것을 말함

이때, CPU는 각 프로세스들이 누군지 알아야 관리가 가능함

프로세스들의 특징을 갖고있는 것이 바로 `Process Metadata`

이 메타데이터는 프로세스가 생성되면 PCB(Process Control Block)이라는 곳에 저장됨

## PCB 란?

- 프로세스 메타데이터들을 저장해놓는 곳
- 한 PCB 안에는 한 프로세스의 정보가 담김
- 프로세스 생성 시 만들어지며 주기억장치에 유지된다.

![program pcb](image/program_pcb.png)

> 프로그램 실행 -> 프로세스 생성 -> 프로세스 주소 공간에 (코드, 데이터 스택) 생성 -> 이 프로세스의 메타데이터들이 PCB에 저장

### PCB 포함 정보

운영체제에 따라 PCB에 포함되는 항목이 다를 수 있지만, 일반적으로는 다음과 같은 정보가 포함되어 있다.

- 프로세스 식별자(Process ID)
- 프로세스 상태(Process State): 생성(create), 준비(ready), 실행(running), 대기(waiting), 완료(terminated) 상태가 있다. - 유예준비상태suspended ready, 유예대기상태suspended wait는 스택이 아닌 disk에 저장된다.
- 프로그램 계수기(Program Counter): 프로그램 계수기는 이 프로세스가 다음에 실행할 명령어의 주소를 가리킨다.
- CPU 레지스터 및 일반 레지스터
- CPU 스케줄링 정보: 우선 순위, 최종 실행시각, CPU 점유시간 등
- 메모리 관리 정보: 해당 프로세스의 주소 공간 등
- 프로세스 계정 정보: 페이지 테이블, 스케줄링 큐 포인터, 소유자, 부모 등
- 입출력 상태 정보: 프로세스에 할당된 입출력장치 목록, 열린 파일 목록 등

## PCB 가 왜 필요한가?

CPU에서는 프로세스의 상태에 따라 교체작업이 이루어진다. (interrupt가 발생해서 할당받은 프로세스가 wating 상태가 되고 다른 프로세스를 running으로 바꿔 올릴 때)

이때, 앞으로 다시 수행할 대기 중인 프로세스에 관한 저장 값을 PCB에 저장해두는 것이다.

## PCB 는 어떻게 관리 되나?

Linked List 방식으로 관리함

PCB List Head에 PCB들이 생성될 때마다 붙게 된다. 주소값으로 연결이 이루어져 있는 연결리스트이기 때문에 삽입 삭제가 용이함.

즉, 프로세스가 생성되면 해당 PCB가 생성되고 프로세스 완료시 제거됨

이렇게 수행 중인 프로세스를 변경할 때, CPU의 레지스터 정보가 변경되는 것을 `Context Switching`이라고 합니다.

## Reference

- <https://github.com/gyoogle/tech-interview-for-developer/blob/master/Computer%20Science/Operating%20System/PCB%20%26%20Context%20Switcing.md>
- <https://ko.wikipedia.org/wiki/%ED%94%84%EB%A1%9C%EC%84%B8%EC%8A%A4_%EC%A0%9C%EC%96%B4_%EB%B8%94%EB%A1%9D>

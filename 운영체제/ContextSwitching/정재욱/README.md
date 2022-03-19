# Context Switching

Context : CPU가 해당 프로세스를 실행하기 위한 해당 프로세스의 정보, PCB에 저장된다

Switching : 말 그대로 교환

- 말 그대로 Context Switching이란
  - PCB의 정보를 읽어 전에 프로세스가 하던 일을 알고 이어서 수행할수 있게 하는 것
  - Context Switching이 일어날 경우 해당 CPU는 일을 하지 못한다
    - 즉 Context Switching이 자주 일어나게 된다며 overhead가 발생할 수 있다.

### Interrupt

- CPU가 프로그램을 실행하고 있는데 다른 곳에서 예외처리가 필요한 경우 현재 실행하고 있는 CPU를 중단시키지 않고 예외 상황을 처리
  - 입출력 요청
  - CPU 사용시간 만료
  - 자식 프로세스를 만들 때
  - 인터럽트 처리를 기다릴 때
  - 등등..
- 인터럽트 처리를 위해서 수많은 우선 순위 알고리즘이 있다.
  - 알고리즘에 의해 CPU와 Context Switching의 효율성을 결정할 수 있다.

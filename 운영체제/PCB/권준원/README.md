# PCB

PCB(Process Control Block)은 프로세스를 제어하기 위해 사용하는 자료구조이다.

PCB는 크게 3가지 영역으로 나뉠 수 있다.

- Process Identification
- Processor State Information
- Process Control Information

### Process Identification

- Process Identification에는 Process Identifier, Parent Process Identifier, User Identifier 정보가 있다.
- OS는 각 프로세스에게 유일한 식별자를 할당한다.
- 식별자는 프로세스 테이블의 인덱스로 사용된다.
- OS에 있는 다른 테이블들도 프로세스 테이블을 참조하기 위해 프로세스 식별자를 사용한다.
    - 메모리 테이블을 구성할 때 메모리의 각 영역에 어떤 프로세스가 할당되었는지 알기 위해 프로세스 식별자를 사용
    - 입출력 및 파일 테이블에서도 사용
- 프로세스들이 서로 통신할 때 통신할 프로세스를 OS에 알려주기 위해 식별자를 사용할 수 있다.
- 프로세스가 다른 프로세스를 생성할 때 식별자를 이용하여 부모 프로세스와 자식 프로세스를 구분할 수 있다.
- 작업에 대해 책임지고 있는 사용자를 지시하기 위해 User Identifier각 할당될 수 있다.

### Processor State Information

- CPU 상태 정보로 프로그램이 중단되었을 때 CPU의 모든 정보들을 저장한다.
- Processor State Information은 User-Visible Register, Control & Status Register, Stack Pointer을 포함한다.
    - User-Visible Register는 사용자가 사용 가능한 레지스터이다. 즉, 사용자 모드에서 사용자 코드에 의해 참조될 수 있는 레지스터이다.
    - Control & Status Register에는 Program Counter, PSW(Program Status Word) 정보를 포함한다.
        - Program Counter : 다음에 실행할 명령어의 주소
        - PSW : 조건 코드, 상태 정보가 저장된다. 조건 코드는 최근에 수행된 산술 또는 논리 연산의 결과이다.
    - Stack Pointer는 프로시저와 시스템 호출의 매개변수와 호출 주소를 저장하는데 사용된다.

### Process Control Information

- 프로세스가 사용하고 있는 모든 자원들과 다른 프로세스와의 관계와 관련된 정보들을 포함한다.
- scheduling and state information
    - 프로세스 상태에 대한 정보와 스케쥴링 정보
    - 프로세스 상태 : 스케쥴될 프로세스의 준비 상황을 정의(수행, 준비, 대기, 중단)
    - 우선순위 : 프로세스의 스케쥴링 우선순위를 나타냄
    - 스케쥴링 정보 : 스케쥴링 기법에 따라 다름(대기하고 있었던 시간, 마지막에 수행되었던 시간 등)
- memory management
    - 메모리 어디를 어떻게 사용하고 있는지에 대한 정보
    - 세그먼트나 페이지 테이블로의 포인터를 가질 수 있음
- resource ownership and utilization
    - 프로세스가 만든 자원들 및 사용하는 자원들
    - 프로세서 및 자원들의 이용률에 대한 정보도 포함될 수 있는데 이 정보는 스케쥴러에 의해 사용
- inter-process communication
    - 프로세스와 프로세스간의 통신
    - 정보를 주고받는 메일 박스 관리
- process privileges
    - 프로세스가 가지고 있는 권한
    - 시스템 안에 있는 자원들에 대해 어떤 권한을 갖고 있는지
- data structuring : 프로세스간의 수많은 포인터에 관한 정보
    - 우선순위를 가지고 대기 상태에 있는 프로세스들은 하나의 큐에 연결될 수 있는데 이를 포인터로 연결
    - parent, child 프로세스는 무엇인지
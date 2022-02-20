# IPC
프로세스와 쓰레드에서 정리했듯이 프로세스는 독립된 실행 객체이다.

따라서 각자의 메모리 공간을 가지며 다른 프로세스의 자원에 접근할 수 없기 때문에 서로 통신하기 위해서는 별도의 기술이 필요하다.

이를 위해 커널에서 `IPC(Inter Process Communication)`을 제공하고, 프로세스는 이를 이용하여 통신할 수 있다.

IPC의 종류는 다음과 같다.

- pipe
- fifo
- message passing
- shared memory
- memory mapping

위처럼 프로세스간 메세지 전송을 목적으로 하는 것이 아닌 동기화를 목적으로 하는 설비들도 있다.

- semaphore
- record locking

## pipe

- 한 프로세스에서 다른 프로세스로의 단방향 통신 채널이다.
- read, write 명령으로 데이터를 주고 받는다.

```c
#include<unistd.h>
int pipe(int fildes[2]);
//하나의 descriptor로 동시에 읽기/쓰기 할 수 없다.
//filedes[0]: 읽기용
//filedes[1]: 쓰기용
```

- 즉, 양방향 통신을 위해서는 2개의 pipe가 필요하다.
- 데이터를 FIFO(선입선출)로 처리한다.
- 부모-자식 프로세스간 통신에 사용된다.
    - pipe는 `fork()` 명령에 의해 상속 가능하다.
- 시스템은 pipe를 파일처럼 관리하기 때문에 만들 수 있는 pipe의 수가 제한되어 있다.
    - process당 open할 수 있는 파일의 수와 시스템 내의 파일 수는 제한되어 있다.(파일 테이블은 배열이기 때문에)
- 기본적으로 blocking을 지원하기 때문에 특별한 동기화가 필요하지 않다.
    - read가 blocking 되는 경우 : pipe가 비어 있는 경우(pipe에 데이터가 들어오면 깨어난다.)
    - write가 blocking 되는 경우 : pipe가 가득 찬 경우(pipe가 빌 때까지 block)
- 간단하게 사용할 수 있다는 장점이 있다.

### 예시

- 단방향 통신

```c
int main() {
	char ch[10];
	int pid, p[2];
	
	//pipe 만들기에 실패했을 경우
	if(pipe(p) == -1) {
		perror("pipe call");
		exit(1);
	}
	
	//자식 프로세스 생성
	pid = fork();
	//자식 프로세스일 경우
	if(pid == 0) {
		close(p[1]);  //읽기만 가능
		read(p[0], ch, 10);  //blocking read, 데이터가 들어올 대까지 block
		printf("%s\n", ch);
		exit(0);
	}

	close(p[0]);  //쓰기만 가능
	scanf("%s", ch);
	write(p[1], ch, 10);  //blocking write, pipe가 가득 차있으면 block
	wait(0);
	exit(0);
}
```

- 양방향 통신

```c
int main() {
	int i, in, pid, p[2][2];

	//pipe 생성, 양방향 통신이기 때문에 2개 필요
	for(i=0; i<2; i++) {
		pipe(p[i]);

	pid = fork();
	//자식 프로세스일 경우
	if(pid == 0) {
		//pipe는 fork()에 의해 상속되기 때문에 따로 생성하지 않아도 됨
		close(p[0][1]);  //첫 번째 pipe의 쓰기 닫음
		close(p[1][0]);  //두 번째 pipe의 읽기 닫음
		read(p[0][0], &in, sizeof(int));  //blocking read, 데이터가 들어올 대까지 block
		in++;
		write(p[1][1], &in, sizeof(int)); //다시 부모 프로세스에 전송
		exit(0);
	}
	//부모 프로세스일 경우
	close(p[0][0]);  //첫 번째 pipe의 읽기 닫음
	close(p[1][1]);  //두 번째 pipe의 쓰기 닫음
	scanf("%d", &in);
	write(p[0][1], &in, sizeof(int));  //blocking write, pipe가 가득 차있으면 block
	read(p[1][0], &in, sizeof(int));  //자식 프로세스로부터 다시 읽어옴
	printf("%d\n", in);
	wait(0);
	exit(0);
}
```

## fifo

- fifo라는 파일을 이용하여 통신한다.
    - 실제로 파일 이름을 부여 받음
    - 소유자, 크기, permission을 갖음
    - 일반 파일처럼 open, close, read, write 가능
- pipe는 동일한 ancestor를 갖는 프로세스들만 연결 가능하지만 fifo는 모든 프로세스들을 연결 가능하다.
    - fifo 파일을 별도로 생성하여 사용하기 때문에
    - 좀 더 넓은 의미의 pipe

```c
#include<sys/types.h>
#include<sys/stat.h>
int mkfifo(const char *pathname, mode_t mode);
//pathname: 파일 이름
//mode: permission 정보
```

- open 호출은 다른 프로세스가 읽기 또는 쓰기를 위해 open될 때까지 blocking
    - 동시에 읽기 fifo를 open하면 둘 다 block되므로 데드락 발생
    - non-blocking open의 경우 상대 프로세스가 준비되지 않으면 -1을 return
- pipe와 마찬가지로 blocking read, blocking write을 지원한다.

### 예시

- Reader

```c
int main() {
	int fd, n;
	char buf[512];
	//fifo 생성
	mkfifo("fifo", 0600);
	//일반 파일처럼 open
	fd = open("fifo", O_RDWR);
	for( ; ; ) {
		n = read(fd, buf, 512);  //blocking read, 데이터가 들어올 대까지 block
		write(1, buf, n);
	}
}
```

- Writer

```c
int main() {
	int fd, i, nread;
	char buf[512];
	//fifo를 open하지 않아 실패한 경우	
	if((fd=open("fifo", O_WRONLY|O_NONBLOCK)) < 0) {
		printf("fifo open failed");
		exit(1);
	}

	for(i=0; i<3; i++) {
		nread = read(0, buf, 512);
		write(fd, buf, nread);  //blocking write, pipe가 가득 차있으면 block
	}

	exit(0);
}
```

## message passing

- 커널 메모리 영역에 채널을 만들어서 프로세스들 사이에 메시지 형태로 정보를 send/receive하는 방법

```c
#include<sys/msg.h>
int msgget(key_t key, int permflags);
//key: queue의 key값
//permflags: queue에 대한 access permission
```

```c
#include<sys/msg.h>
int msgsnd(int mqid, const void *message, size_t size, int flags);
//mqid: queue id
//message: 보낼 message가 저장된 주소
//size: message의 크기
//flags: 0 - 성공할 때까지 blocking
//       IPC_NOWAIT - send가 불가능하면(queue가 가득 찬 경우) 즉시 return
```

```c
#include<sys/msg.h>
int msgrcv(int mqid, void *message, size_t size, long msg_type, int flags);
//mqid: queue id
//message: 받은 message를 저장할 장소의 주소
//size: 저장 장소의 크기
//msg_type: 어떤 메세지를 받을지 선택
//flags: 0 - 성공할 때까지 blocking
//       IPC_NOWAIT - receive가 불가능하면(queue에 해당 메세지가 없는 경우) 즉시 return
//       MSGNOERROR - message가 size보다 길면 초과분을 자름
```

- 커널에서 데이터 통신을 컨트롤할 수 있어 별도의 동기화 로직이 없어도 된다.
    - flag를 설정하지 않을 경우 자동으로 blocking read, blocking write가 가능
- 커널을 통해 데이터를 주고받기 때문에 shared memory 모델보다 느리다.
- message queue 이름처럼 선입선출로 데이터를 처리한다.
- 파일을 이용하는 fifo와 달리 메모리 공간을 이용한다.
- message queue는 큐 하나로 여러 프로세스와 통신 가능하다.
    - 하지만 데이터에 번호를 붙임으로써(mtype) 메세지를 구분한다.
    - 예시를 보면 더 이해가 잘 될 것!

### 예시

- client

```c
//주고받을 메세지 구조, 송신자와 수신자 동일한 구조 사용
struct q_entry{
	//mtype 값은 0 또는 음수이면 안됨, id라 생각하면 편함
	long mtype;
	int data;
};

int main(int argc, char** argv){
	int i, qid, in, id;
	key_t key;
	struct q_entry msg;

	id=atoi(argv[1]);

	key=ftok("key", 3);
	//message queue 생성
	qid=msgget(key, IPC_CREAT|0600);

	for(i=0; i<5; i++){
		scanf("%d", &in);
		//mtype 값 설정
		msg.mtype=id;
		msg.data=in;
		//message 전송, flag가 0이므로 blocking write
		msgsnd(qid, &msg, sizeof(int), 0);
		//현재 mtype+3 메세지를 수신
		msgrcv(qid, &msg, sizeof(int), id+3, 0);
		printf("%d\n", msg.data);
	}

	return 0;
}
```

- server

```c
//주고받을 메세지 구조, 송신자와 수신자 동일한 구조 사용
struct q_entry{
	//mtype 값은 0 또는 음수이면 안됨, id라 생각하면 편함
	long mtype;
	int data;
};

int main(void){
	int i, qid;
	key_t key;
	struct q_entry msg;

	key=ftok("key", 3);
	//message queue 생성
	qid=msgget(key, IPC_CREAT|0600);

	for(i=0; i<15; i++){
		//mtype 값이 절대값보다 작거나 같은 message 수신(여기서는 1~3)
		msgrcv(qid, &msg, sizeof(int), -3, 0);
		//들어온 message의 mtype+3 하여 특정 client와 통신
		msg.mtype+=3;
		msg.data+=8;
		//message 전송, flag가 0이므로 blocking write
		msgsnd(qid, &msg, sizeof(int), 0);
	}

	return 0;
}
```

## memory mapping

- 파일을 이용한 동기화 방법이다.
- 파일을 메모리에 매핑하기 전에 미리 open 해야 한다.
- 주로 파일로 대용량 데이터를 공유해야 할 때 사용한다.

```c
#include<sys/mman.h>
void *mmap(void *addr, size_t len, int prot, int flags, int filedes, off_t off);
```

> `filedes`가 가리키는 파일에서 `off`로 지정한 위치부터 `len`만큼의 데이터를 읽어 `addr`이 가리키는 메모리 공간에 매핑한다.
> 

`void *addr` :  공유할 공간을 정확한 주소로 지정, null을 사용하면 OS가 적절한 공간을 할당해준다.

`size_t len` : 지정한 크기를 포함하는 페이지를 페이지 단위로 가져온다.

`int prot` : 읽기를 할지 쓰기를 할지 지정한다. OS가 동기화하는데 필요하다.

`int flags` : 변경된 내용을 공유할지 안할지(원본 파일에 반영할지) 결정한다.

`int filedes` : 공유할 파일을 지정한다.

`off_t off` : 파일의 어느 위치부터 매핑할 것인지 블락의 시작 위치를 블락 단위(페이지 크기)로 적는다.

### 예시

```c
int main() {
	int fd;
	char *addr;
	
	fd = open("data", O_RDWR);
	//addr은 설정된 공간의 시작 주소로 배열처럼 사용 가능하다.
	addr = mmap(NULL, 512, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
	printf("%s\n", addr);
}
```

## shared memory

- 둘 이상의 프로세스가 물리적 메모리의 일부를 공유
    - memory mapping과 달리 파일을 이용하지 않고 메모리를 직접 공유
- 가장 효율적인 IPC 기법
    - 다른 작업들과 달리 생성할 때를 제외하고 OS 호출이 없음
    - 메모리에 직접 접근
    - 하지만 별도의 동기화 기술이 필요함

```c
#include<sys/types.h>
#include<sys/ipc.h>
#include<sys/shm.h>
int shmget(key_t key, size_t size, int permflag);
//key: 공유 메모리 영역의 identifier
//size: 공유 메모리 영역의 최소 크기(page 단위)
//permflag: access permission
//공유 메모리 영역의 identifier를(shmid) return
```

- shmat을 호출하여 shmget 호출에 의해 할당된 메모리 영역을 자신의 논리적 자료 공간에 부착한다.

```c
#include<sys/shm.h>
int *shmat(int shmid, const void *daddr, int shmflag);
//shimid: 공유 메모리 identifier
//daddr: 프로세스 내의 부착 위치, NULL인 경우 시스템이 위치 결정
//shmflag: 공유 메모리에 대해 읽기/쓰기 여부를 지정
//process 내의 주소(할당된 shared memory) return, 포인터처럼 사용 가능
```

- shmdt을 호출하여 공유 메모리 영역을 프로세스의 논리적 주소 공간으로부터 떼어낸다.

```c
int shmdt(memptr);
//memptr: 공유 메모리 영역에 대한 유효 주소
```

### 예시

- reader

```c
//동기화를 위해 semaphore 사용
union semun{
	int val;
	struct semid_ds *buf;
	ushort *array;
};

int main(void){
	int shmid, semid, i, n, *buf;
	key_t key1, key2;
	union semun arg;
	struct sembuf p_buf;

	key1=ftok("key", 1);
	semid=semget(key1, 1, 0600|IPC_CREAT|IPC_EXCL);
	if(semid==-1){
		semid=semget(key1, 1, 0);
	}
	else{
		arg.val=0;
		semctl(semid, 0, SETVAL, arg);
	}

	key2=ftok("key", 2);
	//shared memory 생성
	shmid=shmget(key2, 10*sizeof(int), 0600|IPC_CREAT);
	//프로세스 공간에 부착, 포인터처럼 사용 가능
	buf=(int *)shmat(shmid, 0, 0);

	for(i=0; i<10; i++){
		scanf("%d", (buf+i));
		p_buf.sem_num=0;
		p_buf.sem_op=1;
		p_buf.sem_flg=0;
		semop(semid, &p_buf, 1);
	}

	exit(0);
}
```

- writer

```c
//동기화를 위해 semaphore 사용
union semun{
	int val;
	struct semid_ds *buf;
	ushort *array;
};

int main(void){
	int shmid, semid, i, n, *buf;
	key_t key1, key2;
	union semun arg;
	struct sembuf p_buf;

	key1=ftok("key", 1);
	semid=semget(key1, 1, 0600|IPC_CREAT|IPC_EXCL);
	if(semid==-1){
		semid=semget(key1, 1, 0);
	}
	else{
		arg.val=0;
		semctl(semid, 0, SETVAL, arg);
	}

	key2=ftok("key", 2);
	//shared memory 생성
	shmid=shmget(key2, 10*sizeof(int), 0600|IPC_CREAT);
	//프로세스 공간에 부착, 포인터처럼 사용 가능
	buf=(int *)shmat(shmid, 0, 0);

	for(i=0; i<10; i++){
		p_buf.sem_num=0;
        	p_buf.sem_op=-1;
        	p_buf.sem_flg=0;
		semop(semid, &p_buf, 1);
		printf("%d\n", *(buf+i));
	}
	//공유 메모리 제거
	shmctl(shmid, IPC_RMID, 0);
	exit(0);
}
```

## semaphore

- 프로세스 간 데이터를 동기화하고 보호하는 역할
- semaphore 관련해서는 따로 정 는하느게을나듯

```c
semwait(sem);
//do something;
semsignal(sem);
```

### 예시

- writer

```c
//semaphore 구조체
union semun{
	int val;  //단일 semaphore 사용시
	struct semid_ds *buf;  //stat 정보를 가져올 때ㅠ
	ushort *array;  //여러 semaphore 사용시
};

int main(){
	int fd, i, semid, *addr;
	key_t key;
	union semun arg;
	struct sembuf p_buf;

	key=ftok("key", 1);
	//semaphore 생성
	semid=semget(key, 1, 0600|IPC_CREAT|IPC_EXCL);
	if(semid==-1){
		semid=semget(key, 1, 0);
	}
	else{
		//semaphore 값을 0으로 초기화
		arg.val=0;
		semctl(semid, 0, SETVAL, arg);
	}

	fd=open("data", O_RDWR|O_CREAT, 0600);
	addr=mmap(NULL, 512, PROT_READ|PROT_WRITE, MAP_SHARED, fd , 0);
	ftruncate(fd, 10*sizeof(int));
	//signal연산
	p_buf.sem_num=0;
	p_buf.sem_op=1;
	p_buf.sem_flg=0;

	for(i=0; i<10; i++){
		scanf("%d", addr+i);
		semop(semid, &p_buf, 1);
	}

	exit(0);
}
```

- reader

```c
//semaphore 구조체
union semun{
	int val;  //단일 semaphore 사용시
	struct semid_ds *buf;  //stat 정보를 가져올 때ㅠ
	ushort *array;  //여러 semaphore 사용시
};

int main(){
	int fd, i, semid, *addr;
	key_t key;
	union semun arg;
	struct sembuf p_buf;

	key=ftok("key", 1);
	//semaphore 생성
	semid=semget(key, 1, 0600|IPC_CREAT|IPC_EXCL);
	if(semid==-1){
		semid=semget(key, 1, 0);
	}
	else{
		//semaphore 값을 0으로 초기화
		arg.val=0;
		semctl(semid, 0, SETVAL, 1);
	}
	fd=open("data", O_RDWR|O_CREAT, 0600);
	//memory mapping 동기화에 사용
	addr=mmap(NULL, 512, PROT_READ|PROT_WRITE, MAP_SHARED, fd, 0);
	ftruncate(fd, 10*sizeof(int));
	//wait연산, semaphore 값이 음수가 되면 block
	p_buf.sem_num=0;
	p_buf.sem_op=-1;
	p_buf.sem_flg=0;

	for(i=0; i<10; i++){
		semop(semid, &p_buf, 1);
		printf("%d\n", *(addr+i));
	}
	//semaphore 제거
	semctl(semid, 0, IPC_RMID, 0);
	exit(0);
}
```

## record locking

- 파일의 일부를 순차적으로 하나의 프로세스만 읽어가도록 하여 동기화 진행
    - critical section 구현
- locking : 특정 record에 대한 다른 프로세스의 읽기/쓰기 제한
    - read lock : 읽기 허용, 쓰기 제한
    - write lock : 읽기, 쓰기 모두 제한
    - unlocking : 제한 해제

```c
#include<fcntl.h>
int fcntl(int filedes, int cmd, struct flock *ldata);
//filedes: lock을 설정하려는 file의 descriptor
//cmd: F_GETLK - lock 정보 얻기
//     F_SETLK: non-blocking locking or unlocking
//     F_SETLKW: blocking locking(일반적인 locking)
//ldata: lock을 걸 위치 지정
```

- lock 정보는 fork()에 의해 계승되지 않음
- 모든 lock은 프로세스 종료시 자동으로 unlcok
- locking 순서에 따라 데드락이 발생할 수 있음
    - F_SETLKW 명령을 통해 데드락이 발생하는지 검사할 수 있음(가능성이 있으면 -1 return)

### 예시
```c
int main() {
	int fd, i, num;
	struct flock lock;

	fd = open("data1", O_RDWR|O_CREAT, 0600);

	//lock 위치 지정
	lock.l_whence = SEEK_CUR;
	lock.l_len = sizeof(int);

	for(i=0; i<10; i++) {
		//쓰기 lock 설정
		lock.l_type = F_WRLCK;
		lock.l_start = 0;
		fcntl(fd, F_SETLKW, &lock);

		read(fd, &num, sizeof(int));
		num = num + 10;
		sleep(1);
		lseek(fd, -sizeof(int), SEEK_CUR);
		wrtie(fd, &num, sizeof(int));
		//lock 해제
		lock.l_type = F_UNLCK;
		//lock이 걸려 있는 위치로 돌아가야함
		lock.l_start = -sizeof(int);
		fcntl(fd, F_SETLK, &lock);

	}

	lseek(fd, 0, SEEK_SET);
	for(i=0; i<10; i++) {
		read(fd, &num, sizeof(int));
		printf(”%d\n”, num);
	}

	return 0;
}
```

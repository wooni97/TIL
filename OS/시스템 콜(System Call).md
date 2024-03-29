# 시스템 콜(System Call)

### **시스템 콜(System Call)이란?**

*일종의 프로그래밍 인터페이스로*, 커널 영역의 기능을 응용 프로그램이 사용 가능하게 한다. 즉 프로세스가 하드웨어에 직접 접근해서 필요한 기능을 할 수 있게 해준다.

이렇게 정의만 간단하게 보면 이해가 잘 안될 수 있다. 커널과 커널 영역? 그리고 프로그래밍 인터페이스라고? 🤔

시스템 콜을 이해하기 위해서 우선 커널부터 살펴보자.

### 커널(Kernel)

![Untitled (1).png](..%2F..%2F..%2FDownloads%2FUntitled%20%281%29.png)

운영체제는 두 부분으로 나뉜다.

- 커널에 명령을 전달하고 실행 결과를 사용자와 응용 프로그램에 돌려주는 **인터페이스**
- 운영체제의 핵심 기능을 모아놓은 **커널**

**커널이란? 운영체제의 핵심 서비스를 담당하는 부분이다**.

운영체제는 프로그램 규모가 가장 큰 프로그램 중 하나이다. 그래서 운영체제가 응용 프로그램에 제공하는 기능들은 매우 다양하다. 그중에서도 가장 핵심적인 서비스들이 있다. 자원에 조작하고 접근하는 기능, 프로그램이 올바르고 안전하게 실행되게 하는 기능이 운영체제의 핵심 서비스에 속한다. 그리고 이러한 서비스를 커널이 제공한다.

커널의 일을 조금 더 자세히 살펴보자. 커널이 주로 하는 일은 메모리 관리, 프로세스 관리, 파일 시스템 관리, 입출력 관리, 프로세스 간 통신 관리 등이 있다.

- 프로세스 관리 : 프로세스에 CPU를 분배하고 작업에 필요한 제반 환경 제공
- 메모리 관리 : 프로세스에 작업 공간을 배치하고 실제 메모리보다 큰 가상 공간 제공
- 파일 시스템 관리 : 데이터를 저장하고 접근할 수 있는 인터페이스 제공
- 입출력 관리 : 필요한 입출력 서비스를 제공
- 프로세스 간 통신 관리 : 공동 작업을 위한 각 프로세스 간 통신 환경 제공

이러한 작업들을 수행하는 커널의 가장 큰 목표는 **컴퓨터의 물리적(하드웨어) 자원과 추상화 자원을 관리**하는 것이다. 추상화란 물리적으로 하나뿐인 하드웨어를 여러 사용자들이 번갈아 사용할 수 있도록 마치 여러개처럼 보이게 하는 기술이다. 즉, 커널이 관리함에 따라 각 사용자들은 하나의 하드웨어를 독점하는 것처럼 느낄 수 있다.

- Task(Process) Management : 물리적 자원인 CPU를 추상적 자원인 Task로 제공
- Memory Management : 물리적 자원인 메모리를 추상적 자원인 Page 또는 Segment로 제공
- File System : 물리적 자원인 디스크를 추상적 자원인 File로 제공
- Network Managment : 물리적 자원인 네트워크 장치를 추상적 자원인 Socket으로 제공
- Device Driver Management : 각종 외부 장치에 대한 접근
- Interrupt Handling : 인터럽트 핸들러
- I/O Communication : 입출력 통신 관리

이처럼 커널은 다양한 자원을 관리하는데 이는 사용자(응용 프로그램)가 물리적인 하드웨어에 접근하고 사용할 수 있도록 **매개하고 자원을 보호하기 위해서다.** 즉, 응용 프로그램과 하드웨어간의 인터페이스 역할을 수행한다.

### 시스템 콜 & 이중 모드

앞서 말한대로 커널은 자원을 보호하는 역할을 한다고 했다. 만약 응용 프로그램이 하드웨어에 마음대로 접근하고 조작할 수 있다면 어떻게 될까?
![Untitled (2).png](..%2F..%2F..%2FDownloads%2FUntitled%20%282%29.png)
그렇게 된다면 자원은 무질서하게 관리될 것이고, 응용 프로그램이 조금만 실수해도 컴퓨터 전체에 큰 악영향을 끼칠 수 있다. 그래서 운영체제는 응용 프로그램들이 자원에 접근하려고 할 때 오직 자신을 통해서만 접근하도록 자원을 보호한다.

일종의 문지기 역할을 하는 것으로, 응용 프로그램이 자원에 접근하기 위해서는 운영체제에 도움을 요청해야 한다.

운영체제에 도움이 요청되면 요청 내역을 커널 영역 내에서 수행해준다.
![Untitled (3).png](..%2F..%2F..%2FDownloads%2FUntitled%20%283%29.png)

운영체제의 문지기 역할은 이중 모드로써 구현이 된다.

<aside>
💡 이중 모드란 CPU가 명령어를 실행하는 모드를 사용자 모드와 커널 모드로 구분하는 방식이다.

</aside>

> **사용자 모드**
>

운영체제 서비스를 제공받을 수 없는 실행 모드이다. 즉, 커널 영역의 코드를 실행할 수 없는 모드이다. 일반적인 응용프로그램은 기본적으로 사용자 모드로 실행된다. 사용자 모드로 실행되는 일반적인 응용 프로그램은 자원에 접근할 수 없다.

> **커널 모드**
>

운영체제 서비스를 제공받을 수 있는 실행모드이다. 즉, 커널 영역의 코들르 실행할 수 있는 모드이다. CPU가 커널 모드로 명령어를 실행하면 자원에 접근하는 명령어를 비롯한 모든 명령어를 실행할 수 있다.

CPU가 사용자 모드로 실행 중인지, 커널 모드로 실행 중인지는 플래그 레지스터 속 슈퍼바이저 플래그를 보면 알 수 있다.

- 플래그가 1일 경우 → 커널 모드
- 플래그가 0일 경우 → 사용자 모드

사용자 모드로 실행되는 프로그램이 자원에 접근하는 운영체제 서비스를 제공받으려면 운영체제에 요청을 보내 커널 모드로 전환되어야 한다.

<aside>
💡 이때 운영체제 서비스를 제공 받기 위한 요청을 시스템 콜이라고 한다.

</aside>

사용자 모드로 실행되는 프로그램은 시스템 콜을 통해 커널 모드로 전환하여 운영체제 서비스를 제공 받는다.

### **시스템 콜 동작 과정**
![Untitled (4).png](..%2F..%2F..%2FDownloads%2FUntitled%20%284%29.png)

1. 응용 프로그램에서 시스템 콜 호출 : 응용 프로그램이 운영체제의 서비스를 필요로 할 때, 해당 서비스를 요청하는 시스템 콜을 호출한다.
2. 커널 모드 전환 : 시스템 콜 호출로 인해 사용자 모드에서 커널 모드로 전환된다.
3. 시스템 콜 처리 : 커널은 호출된 시스템 콜을 식별하고 해당 서비스를 처리하기 위해 필요한 작업을 수행한다.
4. 결과 반환 : 시스템 콜이 성공적으로 처리되면, 커널은 결과를 응용 프로그램에 반환한다. 이때 커널 모드에서 사용자 모드로 다시 전환된다.
5. 응용 프로그램 계속 실행

시스템 콜은 일종의 소프트웨어 인터럽트이다. 그래서 CPU가 시스템 호출을 처리하는 순서는 인터럽트 처리 순서와 유사하다. 시스템 콜이 호출되면 CPU는 지금까지의 작업을 백업하고, 커널 영역 내에 시스템 호출을 수행하는 코드(인터럽트 서비스 루틴)를 실행한 뒤 다시 기존에 실행하던 응용 프로그램으로 복귀하여 실행을 계속해 나간다.

### **시스템 콜 구분 방법**

커널은 내부적으로 시스템 콜 구분을 위해 기능별로 고유 번호 할당 & 그 번호에 해당하는 제어루틴을 커널 내부에 정의한다.

- 시스템 콜이 발생하면 해당 기능번호 확인
- 커널은 인터럽트를 발생시킨 명령을 검사하여 어떤 시스템 호출이 발생했는지 결정
- 이 때 전달된 인자가 사용자 프로그램이 요청하는 서비스 유형을 표시함
- 커널은 인자가 정확하고 합법적인지 검증하고 요청 실행
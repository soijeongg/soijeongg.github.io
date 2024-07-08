---
layout: post
read_time: true
show_date: true
title: "[항해 99] 주특기 1차 과제"
date: 2024-04-15 15:00:20 -0600
description: "[항해 99] 주특기 1차 과제"
image: https://velog.velcdn.com/images/soijeongg/post/4a3b4529-7783-4201-90f4-03de54197e08/image.png
tags: 
    - coding
    - diary
    - hh99
author: soi

toc: no # leave empty or erase for no TOC
---

### Basic CPU / Mem / Net / Disk

1. cpu basic
![](https://velog.velcdn.com/images/soijeongg/post/73b8d9f8-e75f-4a22-bcf1-d67ed6e1e973/image.png)

- cpu idle: 어떠한 프로그램에 의해서도 사용되지 않은 상태 (남아도는 느낌)? 59.9
- cpu busy user: 시스템의 CPU를 사용하는 사용자 프로세스에 의해 소비된 시간의 백분율, 사용자가 실행하는 응용 프로그램 및 작업에 의해 발생한 CPU 활동
- cpu busy system: 시스템 레벨에서 실행되는 프로세스에 의해 소비된 CPU 시간의 백분율
시스템 관리 작업 또는 커널 모듈 등 시스템 레벨의 프로세스에 의해 사용되는 CPU의 비율
- CPU Busy LRQS:스템에서 발생하는 Long-Running Queue의 프로세스에 의해 사용된 CPU 시간의 비율을 나타냅니다. Long-Running Queue는 일반적으로 시스템에서 오랫동안 실행되는 프로세스

![](https://velog.velcdn.com/images/soijeongg/post/1783c385-a47c-4450-901c-b4e306385334/image.png)
- ram total: 시스템에 설치된 전체 램의 용량
- ram used: 사용한 램의 용량
- ram Cache +Buffer:  운영 체제가 디스크에서 데이터를 읽거나 쓸 때 사용하는 메모리의 일부 최근에 액세스한 데이터의 사본을 보관하여 재액세스할 때 빠른 속도로 데이터에 액세스할 수 있도록 도와줌
- ram free: 시스템에서 현재 사용 가능한 랜덤 액세스 메모리(RAM)의 양
![](https://velog.velcdn.com/images/soijeongg/post/b7a722d4-4eb4-4308-a5d4-30a5f6a06d96/image.png)
- recv eth0: 이더넷 인터페이스(eth0)를 통해 수신된 네트워크 패킷의 양, b/s는 초당 바이트(b) 수
- recv io: 수신된 입력/출력(IO) 시스템에서 수신된 입력 또는 출력 작업을 나타내는 것
0 trans eth0: 네트워크 인터페이스를 통한 전송"을 나타냅니다. 이것은 시스템에서 eth0 네트워크 인터페이스를 통해 전송된 데이터양을 나타냄
![](https://velog.velcdn.com/images/soijeongg/post/00d937b2-02d2-4e3e-b1df-8043eea35693/image.png)

- "System - Processes executing in kernel mode: 커널에서 실행되는 프로세스 6.67
높을 수록 많이 처리 하고 있음
- User - Normal processes executing in user mode: 시스템에서 사용자 모드에서 실행 중인 일반 프로세스의 비율 
용자 모드에서 실행되는 프로세스가 많을수록 시스템은 사용자 요청을 처리하거나 응용 프로그램을 실행하는 데 더 많은 시간을 할애하고 있음을 의미
- Iowait - Waiting for I/O to complete:  시스템이 I/O 작업이 완료될 때까지 대기하는 시간의 비율
디스크 또는 네트워크와 같은 입출력 작업이 완료될 때까지 프로세스가 대기하는 시간을 의미
디스크나 네트워크에 대한 입출력이 많은 경우 이 시간은 높아질 수 있음

![](https://velog.velcdn.com/images/soijeongg/post/c7fbdc35-1acf-4dfa-8031-75ab7f7241a3/image.png)

- xvda - Writes completed: 디스크 xvda에 대한 쓰기 작업이 완료된 횟수를 나타냅니다. 이는 시스템이 디스크에 데이터를 쓴 횟수를 의미
초당 디스크 입출력 작업 수
![](https://velog.velcdn.com/images/soijeongg/post/31bfc665-d2a6-4a77-9146-dd9a6de87e2f/image.png)
- xvda - Successfully written bytes: 디스크에 성공적으로 기록된 데이터의 양을 나태냄, 이 높을수록 시스템이 디스크에 대한 데이터를 효율적으로 처리하고 있다는 것을 의미
- xvda - Successfully read bytes:디스크로부터 성공적으로 읽혀진 바이트 수

이 경우 디스크에서 읽는 속도가 디스크에 쓰는 속도보다 현저히 낮습니다. 일반적으로 디스크 읽기 속도는 디스크 쓰기 속도보다 빠를 수 있지만, 이러한 차이는 매우 극적입니다. 이것은 디스크 읽기가 디스크 쓰기보다 훨씬 느리다는 것을 나타낼 수 있습니다. 종합적으로, 시스템의 디스크 I/O 퍼포먼스를 개선하기 위해 디스크 읽기 및 쓰기 속도를 모두 향상시키는 것이 이상적일 것입니다.
![](https://velog.velcdn.com/images/soijeongg/post/7ef24ede-110e-4c8e-949d-983d9a76a605/image.png)

- xvda: 리눅스 환경에서 디스크를 나타내는 표기 방식 중 하나 낮으면 디스크 활동이 상대적으로 낮다 -> 시스템의 디스크 부하가 상당히 낮다 

![](https://velog.velcdn.com/images/soijeongg/post/86206d91-45f6-44f7-8dd7-ab6bee9903f3/image.png)

-Dirty - Memory which is waiting to get written back to the disk: Dirty"는 디스크에 쓰여진 메모리의 양 
이 메모리는 디스크로 다시 기록되기를 기다리는 상태입니다. 즉, 시스템이 디스크에 쓰기 작업을 수행 중이거나 최근에 쓰기 작업을 수행하여 메모리에 저장된 데이터가 아직 디스크로 옮겨지지 않은 상태
=> 디스크로 쓰기 작업이 완료되지 않은 메모리의 양,일반적으로 디스크가 비교적 바쁘지 않고 디스크에 대한 쓰기 작업이 비교적 빨리 완료될 경우 작은 양으로 유지

- Writeback - Memory which is actively being written back to disk: 
현재 디스크로 쓰여지고 있는 메모리, 메모리는 시스템이 디스크에 데이터를 쓰는 작업을 진행 중인 상태
![](https://velog.velcdn.com/images/soijeongg/post/e133e094-653e-40e1-99fd-3e53e40df946/image.png)

- Pagesin - Page in operations: 시스템 내에서 페이지를 디스크로부터 메모리로 읽어들이는 작업의 수를 나타냄 15초동안   84.5번의 페이지 인 동작을 수행했
- Pagesout - Page out operations:  페이지 아웃은 메모리에서 디스크로 데이터를 쓰는 동작을 의미합니다
![](https://velog.velcdn.com/images/soijeongg/post/2391d708-a411-4994-a869-a3bd9283a22c/image.png)

- Pgminfault - Minor page fault operations: . 페이지 결함은 프로그램이 요청한 메모리 페이지가 현재 물리적으로 메모리에 로드되어 있지 않을 때 발생
시스템이 작은 페이지 결함을 처리하는 빈도를 나타내며, 높은 값은 시스템이 이러한 작은 결함을 자주 겪고 있음을 나타냅

![](https://velog.velcdn.com/images/soijeongg/post/c2182ee1-e8b2-4480-b729-11a55eec6001/image.png)

- Processes forks second: 시스템에서 초당 생성된 프로세스의 수
 프로세스가 생성될 때마다 새로운 프로세스가 형성되고 이러한 프로세스가 시스템 자원을 사용하게 됩니다. 따라서 초당 생성된 프로세스의 수가 많으면 시스템 부하가 증가
![](https://velog.velcdn.com/images/soijeongg/post/134f7a6e-3d85-42cf-bfac-e050ab02274b/image.png)

- CPU 0 - seconds spent running a process:  CPU 0에서 프로세스가 실행된 시간(초
해당 CPU 코어에서 실행된 프로세스들이 소비한 시간을 측정
 CPU 시간은 일반적으로 사용 중인 프로세스에 의해 소비되며, 이 정보는 시스템의 부하와 성능을 모니터링하고 디버깅하는 데 사용
 - CPU 0 - seconds spent by processing waiting for this CPU: "는 해당 CPU에서 대기 중인 프로세스가 소비한 시간(초)을 나타냄
 ![](https://velog.velcdn.com/images/soijeongg/post/d6faf164-f97c-4d4d-a37b-048700a59927/image.png)

- Context switches: 컨텍스트 스위치는 CPU가 현재 실행 중인 프로세스에서 다른 프로세스로 전환하는 과정
컨텍스트 스위치는 일반적으로 시스템 성능에 영향을 미치며, 많은 수의 컨텍스트 스위치가 발생할 경우 CPU 시간이 소비되어 처리량이 감소할 수 있습니다. 따라서 최적의 성능을 얻기 위해서는 컨텍스트 스위치를 최소화하는 것이 중요

- Interrupts: CPU가 프로그램 실행 중에 하드웨어나 소프트웨어에서 발생하는 이벤트를 처리하기 위해 현재 실행 중인 작업을 일시적으로 중단하고 해당 이벤트에 대한 처리를 수행

![](https://velog.velcdn.com/images/soijeongg/post/4fc356b2-9a4e-454a-893f-8c07b4a6c21d/image.png)

- load 1m: 시스템 로드의 1분 평균(1-minute load average)은 일반적으로 시스템의 CPU 및 I/O 리소스 사용률을 나타내는 지표 중 하나
 1분 동안 시스템에 대한 부하를 나타내는데 사용됩니다. 시스템 로드는 대기 중인 프로세스 수를 포함하여 시스템에서 실행 중인 프로세스의 평균 수를 나타냅니다. 따라서 1분 평균 로드가 높을수록 시스템이 과부하 상태에 있을 가능성이 높음
 일반적으로 로드가 CPU 코어의 수보다 높은 경우 시스템에 부하가 걸려있는 것으로 
 ![](https://velog.velcdn.com/images/soijeongg/post/85dc5d34-ec00-4bdd-baf7-7b277f7eb3e6/image.png)
- Schedule timeslices executed by each cpu:
 CPU가 실행하는 스케줄 타임슬라이스는 해당 CPU가 얼마나 많은 시간 동안 프로세스를 실행하는지를 나타냄
 이것은 CPU가 얼마나 많은 시간 동안 활성화되어 있는지를 보여주며, 일반적으로 CPU 부하와 관련
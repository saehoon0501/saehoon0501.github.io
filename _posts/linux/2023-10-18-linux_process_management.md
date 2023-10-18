---
title: Linux Process Management
date: 2023-10-18 13:30:00 +/-0
categories: [linux]
tags: [linux_process_management] # TAG names should always be lowercase
---

## Process

process란 저장되어 있던 프로그램이 메모리에 적재되어 실행되고 있는 것을 말한다.  
Terminal에서 명령어를 입력하면 명령어가 실행 가능한 파일이면 process가 생성되게 되며, shell built in 의 경우 생성되지 않는다. 이러한 type은 명령어 type으로 확인 가능하다.

process는 각각의 id인 PID를 가지고 이를 수행시킨 user와 group에 대한 정보를 가진다. 따라서 만약 실행되는 파일에 SUID가 설정되지 않았다면 실행시키는 user의 권한으로 수행되게 된다.

Linux에서 어떤 process에서 fork()를 실행하면 새로운 process 생성되게 되며, 이때 fork를 수행한 process는 parent 생성된 process는 child가 되게 된다. Linux 시스템이 처음 부팅될 때 실행되는 init or systemd process의 경우는 예외이다.
Ex) shell에서 ls 명령어를 수행하면 bash가 parent ls 프로세스가 child가 된다.

Daemon process란 background에서 수행되는 process를 말하며 상호작용될 수 없기에 terminal에서 어떠한 control도 할 수 없다. 보통 openSSH Server Daemon, web server Daemon 등이 존재한다.

Zombie process란 OS에서 수행 종료된 process에 대한 할당된 자원과 정보를 관련 process로 넘겨주는 과정에서 어떠한 데이터도 관련 process에서도 받아지지 않은 상태를 말한다. 이 경우 OS에서 빠르게 메모리에서 제거하고 사라진다.

Orphan process란 반대로 parent process가 child보다 먼저 종료되어 사라지는 경우를 말한다. 이 경우 기본적으로 모든 child들은 hangup signal을 받게되고 종료되게 된다.

각각의 process는 저마다 리소스와 정보를 가지고 있으며, 하나의 process 안에 여러 Thread가 존재할 수 있다. Thread의 경우 같이 있는 process 안에서 리소스 등을 공유할 수 있다.
task는 process와 같은 뜻으로 사용된다.

## Listing Processes

특정한 process 그리고 이와 관련된 property를 확인할 수 있는 명령어 3가지로 ps,pgrep,pstree가 존재한다.

### ps

process status의 약자로 기본적으로 현재 terminal에서 실행되고 있는 process와 아래와 같은 정보를 같이 보여준다.

- PID: process id로 각 process 마다 고유하다. 이를 통해 process에 대한 상호작용을 할 수 있다.
- TTY: process의 control을 하는 terminal을 나타낸다. ?의 경우 어떠한 terminal도 해당 process와 연관되어 있지 않음을 나타낸다.
- TIME: 해당 process의 누적된 cpu time을 의미한다.
- CMD: process를 수행시킨 명령어를 나타낸다.

이러한 output보다 더 자세한 사항들을 알아보기 위해 여러 옵션들이 존재한다.

- e는 시스템에서 수행되고 있는 모든 process를 나타낸다.
- f는 full-listing format으로 process들에 대한 자세한 정보를 나타내준다.
- a는 controlling terminal이 없는 process를 제외한 나머지 process들을 나타낸다.
- u는 user-oriented format을 통해 process에 대한 user id 그리고 cpu/memory 사용량을 같이 보여준다.
- x는 다른 옵션들에 의해 해당되는 process들과 controlling terminal이 없는 process들을 보여준다.
- sort 옵션을 통해 특정 property를 통해 process를 정렬하여 나타낼 수 있다. -의 경우 기존 sorting에서 reverse를 의미한다.
- u는 특정 user에 연관된 process만을 가져온다.

보통 aux가 함께 자주 사용되며 이를 통해 아래와 같은 정보를 확인할 수 있다.

- USER: process를 수행시킨 username
- PID
- %CPU: process의 cpu 사용량
- %MEM: memory 사용량
- VSZ: process의 가상화 memory 크기로 process가 접근 가능한 모든 memory를 포함해서 나타낸다.
- RSS: 실제 물리적인 memory 사용량을 나타낸다.
- TTY
- STAT: 현재 process 상태를 나타내며, Z는 Zombie, T는 stopped, I는 Idle Kernel Thread를 의미한다. < 표시는 우선순위가 높은 process를 말하며, N은 이와 반대로 우선순위가 낮다.
- START
- TIME
- COMMAND

### pgrep

보통 ps와 grep을 pipe lining하여 원하는 process를 찾는 경우가 많으며, 이를 하나의 명령어 축약한 것이다.

- l을 통해 찾은 process의 PID와 이름을 모두 가져올 수 있다.
- u를 통해 특정 username에 연관된 process에 대한 PID를 가져올 수 있다.

### pstree

수행되는 process들의 hierarchical tree 구조를 나타낸다.
{}로 나타낸지는 것들은 process의 thread를 의미한다.

- c 옵션을 통해 합쳐진 동일한 branch들을 merging하지 않고 하나하나 다 나타낼 수 있다.

## Dynamic Real-Time View of the Processes

ps는 process들에 대한 static view를 제공하며, 이와 다르게 실시간으로 process 상태를 확인할 수 있는 명령어로 top이 있다.

top을 실행하는 상태에서 사용할 수 있는 유용한 기능들은 아래와 같다.

- 1을 통해 각 cpu core가 수행하는 정보로 나눠서 process를 볼 수 있다.
- t를 통해 cpu 사용량에 대한 표시를 ASCII Graph로 볼 수 있다.
- m을 통해 memory 사용량에 대한 표시를 다르게 볼 수 있다.
- d를 통해 process update 간격을 직접 설정할 수 있다.
- y를 통해 현재 cpu에서 실행되는 process를 따로 강조 표시된 상태로 볼 수 있다.
- x를 통해 강조 표시된 process에 대해 sort할 수 있다.
- b를 통해 강조 표시를 highlight으로 대체할 수 있다.
- <>를 통해 강조 표시를 열들 간 이동할 수 있다.
- R를 통해 reverse order sorting
- e를 통해 크기를 나타내는 unit size를 변경
- P를 통해 processor 또는 CPU별로 sort
- M을 통해 memory로 sort
- u를 통해 특정 user에 대한 process만 표시(prompt에 username 작성)
- F를 통해 열들로 표시되는 정보들을 수정할 수 있다.
- W를 통해 top에서 수정한 설정 사항들을 저장한다.

```shell
# top에 대한 수행결과를 1초 딜레이로 3번 반복하여 txt파일에 append한다.
# -b옵션은 batch mode를 의미하며, 이를 통해 txt로 top 결과를 보낸다.
top -d 1 -n 3 -b > top_proc.txt
```

top에 대한 수행결과를 따로 저장하여 grep을 통해 시간 별로 process의 변화를 비교해 볼 수 있다.

## Signal and Killing Processes

process가 반응하지 않거나 너무나 많은 리소스를 잡아먹을 경우 강제로 종료해야 될 때가 있다. 이럴 때 해당 process들을 제거할 수 있는 명령어로 kill을 사용한다.

kill에서 보내는 signal에 따라 process가 다르게 반응(특정 signal은 무시하고 진행할 수 있다.)하며, 주요한 signal들은 아래와 같다. signal -l option을 통해 확인할 수 있다.  
signal을 받으면 process는 종료 상태로 진입할 수 있게 된다.  
일반 user들은 자신이 수행 시킨 process에 대해서만 signal을 보낼 수 있으며, root의 경우 모든 process에 해당한다.

- 1(SIGHUP): process가 재시작하도록 한다. 이는 config를 바꿨을 때 이를 반영하기 위해 사용될 수 있다.
- 2(SIGINT): ctrl+c를 누르면 보내지는 signal로 process를 중단 상태로 보낸다. 하지만 이는 process에 의해 무시 될 수 있음
- 15(SIGTERM): 별도의 명시가 없으면 기본적으로 사용되는 종료 signal이다. 이는 무시될 수 있으며 process가 알아서 종료될 때까지 기다리는 방식이기에 soft kill이라 불린다.

- 9(SIGKILL): hard kill이라 불리며, process에 상태와 상관없이 바로 종료 시킨다.

만약 여러 process들에 대해 한번에 kill signal을 보내고 싶으면 killall 명령어를 사용한다. killall에는 process 이름을 인자로 사용할 수 있으며, 이는 whole name이 일치하는 process에 대해서만 kill을 수행한다.

## Foreground and Background Processes

process의 2가지 종류이다.

- Foreground의 경우 user에 의해 실행되며, 실행되는 동안 user가 다른 작업을 terminal에서 수행할 수 없다.
- Background의 경우 system service 또는 user에 의해 실행되며, 실행되는 동안 user가 terminal에서 다른 작업을 수행할 수 있다. 실행되는 동안 상호작용 할 수 없다.

Background에서 작업을 수행하고 싶을 경우 명령어를 다 작성한 후 마지막에 &(ampersand)를 붙이면 된다.  
보통 Background의 경우 그 수행결과와 에러를 모두 file로 redirect하여 저장한다. 만약 /dev/null 파일로 redirect한다면 해당 파일에서는 작성되는 모든 내용을 바로바로 제거하는 '블랙홀'로 사용되기에 아무것도 저장되지 않게 된다.

## Job Control

Background로 task 수행 시 2가지 번호가 출력되는 것을 알 수 있다.  
하나는 Job id이며, 마지막은 PID이다.

job id를 통해 background process들을 식별할 수 있으며, jobs 명령어를 통해 이러한 process들에 대한 job id를 출력 가능하다.
job id는 shell에 의해 유지되며, 여러 shell에서 다른 process들에 대해 같은 job id를 부여할 수 있다. 이는 PID와 같이 system kernel에서 모두 관리되는 것과 다르다.

background를 foreground로 가져오려면 명령어 fg와 %job_id를 입력하면 된다.

만약 foreground에 가져온 후 수행을 멈추고 싶다면, ctrl+z를 누른다.  
그리고 재개하고 싶다면, bg %job_id 명령어를 입력한다.

foreground/background 모두 작업을 수행시킨 후 terminal을 닫으면, parent process가 사라졌기에 hup signal이 보내져 작업이 중단된다. 만약 이를 원하지 않는다면, nohup 명령어 맨 앞에 추가한다.  
nohup 사용 시 그 수행 결과는 nohup.out 파일에 저장되게 되며, 만약 현재 dir에 권한이 없으면 user의 home dir에 저장되게 된다.

## Commands

```shell
##########################
## Process Viewing (ps, pstree, pgrep)
##########################
# checking if a command is shell built-in or executable file
type rm        # => rm is /usr/bin/rm
type cd        # => cd is a shell built-in

# displaying all processes started in the current terminal
ps

# displaying all processes running in the system
ps -ef
ps aux
ps aux | less       # => piping to less

# sorting by memory and piping to less
ps aux --sort=%mem | less

# ASCII art process tree
ps -ef --forest

# displaying all processes of a specific user
ps -f -u username

# checking if a process called sshd is running
pgrep -l sshd
ps -ef | grep sshd

#displaying a hierarchical tree structure of all running processes
pstree

# prevent merging identical branches
pstree -c

##########################
## Dynamic Real-Time View of Processes(top)
##########################

# starting top
top

## top shortcuts while it's running
h       # => getting help
space   # => manual refresh
d       # => setting the refresh delay in seconds
q       # => quitting top
u       # => display processes of a user
m       # => changing the display for the memory
1       # => individual statistics for each CPU
x/y     # => highlighting the running process and the sorting column
b       # => toggle between bold and text highlighting
<       # => move the sorting column to the left
>       # => move the sorting column to the right
F       # => entering the Field Management screen
W       # => saving top settings

# running top in batch mode (3 refreshes, 1 second delay)
top -d 1 -n 3 -b > top_processes.txt

# Interactive process viewer (top alternative)
sudo apt update && sudo apt install htop    # => Installing htop
htop

##########################
## Killing processes (kill, pkill, killall)
##########################

# listing all signals
kill -l

# sending a signal (default SIGTERM - 15) to a process by pid
kill pid        # => Ex: kill 12547

# sending a signal to more processes
kill -SIGNAL pid1 pid2 pid3 ...

# sending a specific signal (SIGHUP - 1) to a process by pid
kill -1 pid
kill -HUP pid
kill -SIGHUP pid

# sending a signal (default SIGTERM - 15) to process by process name
pkill process_name          # => Ex: pkill sleep
killall process_name
kill $(pidof process_name)  # => Ex: kill -HUP $(pidof sshd)

# running a process in the background
command &   # => Ex: sleep 100 &

# Showing running jobs
jobs

# Stopping (pausing) the running process
Ctrl + Z

# resuming and bringing to the foreground a process by job_d
fg %job_id

# resuming in the background a process by job_d
bg %job_id

# starting a process immune to SIGHUP
nohup command &     # => Ex: nohup wget http://site.com &
```

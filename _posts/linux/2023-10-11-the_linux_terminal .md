---
title: The Linux Terminal In Depth
date: 2023-10-11 22:10:00 +/-0
categories: [linux]
tags: [linux_terminal] # TAG names should always be lowercase
---

Linux Terminal에서 shell을 통해 Kernel에 작업을 수행시키는 cmd를 입력할 수 있으며 이와 관련된 것들에 대해 알아본다.

## Terminals, Consoles, Shells and Commands

- Terminal과 Console은 서로 동의어로 사용되며, Terminal의 경우 GUI 환경에서 작업할 경우 Console은 GUI가 없는 text only 환경에서 사용될 경우를 일컫는다.  
   둘다 Shell을 사용한다. (기본적으로 Bash를 사용)
- Shell은 Kernel에 명령어를 보낼 수 있게 해주는 Interface로, 작성된 명령어가 문법적으로 맞는지 그리고 존재하는지 확인한 후 Kernel로 전달하는 역할을 한다.
- Command는 OS에서 제공하는 서비스를 수행하기 위한 명령어로 옵션과 인자를 추가적으로 같이 작성 가능하며 case-sensitive 하다.  
  옵션의 경우 -가 하나 있으면 짧은 형태의 옵션이며 --는 옵션의 fullname을 의미한다. 보통 여러 옵션을 사용하는 경우 - 이후에 원하는 옵션들을 붙여서 작성한다.  
  ex) ls -lh
  보통 명렁어 옵션 인자 순으로 작성한다.

## Getting Help, Man Pages(man, type, help, apropos)

사용 가능한 명령어 앞에 man을 넣으면 명령어 관련 정보를 얻을 수 있다.
[] 안에 있는 것들은 선택 사항으로 작성할 수 있는 것들이며 ...의 경우 1개 이상이 반복되어 작성될 수 있다는 뜻이다.  
Bold체의 경우 정확히 그대로 작성되어야 하는 것들을 의미하며 밑줄 또는 기울림체는 상황에 맞는 것들로 대체되어야 하는 것들을 의미한다.

man page에 들어간 후 원하는 키워드를 검색하고 싶다면 Backslash 이후에 원하는 단어를 작성하면 된다. 이는 page 앞에서 뒤로 search하는 것을 의미한다.  
만약 뒤에서 앞으로 search하고 싶다면 먼저 G를 입력하여 page 맨 뒤로 이동 후 ? 이후에 원하는 단어를 작성한다.  
n과 N을 통해 검색된 결과를 왔다갔다 할 수 있다.

type 명령어를 통해 shell built-in 명령어와 실행 가능한 명령어를 구별할 수 있다.  
shell built-in의 경우 man page가 따로 존재하지 않는다.  
명령어 이후 --help를 통해 명령어 type에 상관없이 명령어 관련 정보를 받을 수 있다.

man -k 키워드를 통해 키워드를 참조 가능한 명령어들을 검색해 볼 수 있다.

```shell
##########################
## Getting Help in Linux
##########################

# MAN Pages
man command     # => Ex: man ls

# The man page is displayed with the less command
# SHORTCUTS:
# h         => getting help
# q         => quit
# enter     => show next line
# space     => show next screen
# /string   => search forward for a string
# ?string   => search backwards for a string
# n / N     => next/previous appearance

# checking if a command is shell built-in or executable file
type rm        # => rm is /usr/bin/rm
type cd        # => cd is a shell builtin

# getting help for shell built-in commands
help command    # => Ex: help cd
command --help  # => Ex: rm --help

# searching for a command, feature or keyword in all man Pages
man -k uname
man -k "copy files"
apropos passwd
```

## The TAB Key

Terminal에 명령어나 인자를 모두 작성하지 않고 일부만 작성한 후 TAB키로 auto-complete이 가능하다.  
만약 TAB가 작동하지 않는다면 두번 눌러서 match되는 결과가 2개 이상인지 또는 오타가 있는지 확인한다.  
항상 TAB을 통해 명령어를 완성하여 오타로 인한 실수 가능성을 줄인다.

## Keyboard Shortcuts

```shell
##########################
## Keyboard Shortcuts
##########################
TAB  # autocompletes the command or the filename if its unique
TAB TAB (press twice)   # displays all commands or filenames that start with those letters

# clearing the terminal
CTRL + L

# closing the shell (exit)
CTRL + D

# cutting (removing) the current line
CTRL + U

# moving the cursor to the start of the line
CTRL + A

# moving the cursor to the end of the line
Ctrl + E

# stopping the current command
CTRL + C

# sleeping a the running program
CTRL + Z

# opening a terminal
CTRL + ALT + T
```

## The Bash History

사용했던 명령어들은 기본적으로 파일과 메모리에 저장되며 명령어 history를 통해 이를 확인해 볼 수 있다.  
! 이후에 -숫자 형태를 통해 가장 최근에 실행했던 숫자 번째 명령어를 다시 실행할 수 있으며, !명령어를 통해 history에 있는 해당 명령어 내용을 그대로 다시 수행 가능하다.  
만약 실행하는 대신 조회만 해보고 싶다면 뒤에:p를 붙이면 된다. ex) !ping:p  
CTRL+R을 통해 history에서 사용했던 내용들을 검색하여 다시 수행 가능하며 도중 CTRL+G를 통해 검색을 취소 가능하다.

history 명령어에 -c를 통해 전체 history를 정리 가능하며, -d 줄번호 를 통해 history 특정 줄의 내용을 삭제 가능하다.

## Running Commands Without Leaving a Trace

상황에 따라 수행하는 명령어와 내용들이 기록에 남아서는 안될 경우가 존재한다.  
이를 위해 명령어를 작성하기 전에 whitespace를 먼저 하고 입력하면 수행 내용이 남지 않는다.

만약 수행되지 않는다면 관련 설정을 담은 환경변수 $HISTCONTROL=ignoreboth를 넣는다. 이는 중복되는 명령어와 whitespace 이후 명령어를 작성할 경우 history에 기록을 남기지 않는 것을 의미한다.
shell에서 환경변수는 항상 앞에 $를 붙여 접근해야 하며 설정을 변경한 후 bashsrc파일에 저장해줘야 설정이 계속 유지된다.  
설정을 저장하기 위한 명령어는 아래와 같다.

```shell
#bashrc 파일 맨 뒤에 아래 문자열을 추가한다.
echo "HISTCONTROL=ignoreboth" >> .bashrc
```

만약 history에 해당 명령어가 수행된 시간까지 기록하고 싶다면, HISTTIMEFORMAT 환경변수를 변경하면 된다.

```shell
#bashrc 파일 맨 뒤에 아래 문자열을 추가한다.
echo 'HISTTIMEFORMAT="%d/%m/%y %T"' >> .bashrc
```

```shell
##########################
## Bash History
##########################

# showing the history
history

# removing a line (ex: 100) from the history
history -d 100

# removing the entire history
history -c

# printing the no. of commands saved in the history file (~/.bash_history)
echo $HISTFILESIZE

# printing the no. of history commands saved in the memory
echo $HISTSIZE

# rerunning the last command from the history
!!

# running  a specific command from the history (ex: the 20th command)
!20

# running the last nth (10th) command from the history
!-10

# running the last command starting with abc
!abc

# printing the last command starting with abc
!abc:p

# reverse searching into the history
CTRL + R

# recording the date and time of each command in the history
HISTTIMEFORMAT="%d/%m/%y %T"

# making it persistent after reboot
echo "HISTTIMEFORMAT=\"%d/%m/%y %T\"" >> ~/.bashrc
# or
echo 'HISTTIMEFORMAT="%d/%m/%y %T"' >> ~/.bashrc
```

## Root Access

기본적으로 root user와 그렇지 않은 user 이렇게 두가지로 나눠서 linux 계정을 볼 수 있다.  
root는 시스템에 관리작업과 같은 작업의 경우에 요구되는 권한을 말한다.  
기본적으로 sudo를 통해 root 권한으로 명령을 실행할 수 있으며, 한번 passwd를 작성하면 5분 동안 caching되어 다음 sudo 작업은 바로 진행이 가능하다.
root로 작업할 경우 맨 앞이 $ 대신 #로 바뀐다.

```shell
##########################
## Running commands as root (sudo, su)
##########################

# running a command as root (only users that belong to sudo group [Ubuntu] or wheel [CentOS])
sudo command

# becoming root temporarily in the terminal
sudo su      # => enter the user's password

# setting the root password
sudo passwd root

# changing a user's password
passwd username

# becoming root temporarily in the terminal
su     # => enter the root password
```

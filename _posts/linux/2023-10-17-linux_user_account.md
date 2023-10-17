---
title: User Account Management
date: 2023-10-17 7:30:00 +/-0
categories: [linux]
tags: [linux_user_account] # TAG names should always be lowercase
---

## passwd & shadow

### passwd

user account에 대한 기본적인 정보를 저장한 파일
user마다 가진 정보를 :을 통해 구분하여 저장하며 각 필드에 대한 정보는 아래와 같다.

1. user의 로그인 name
2. encrypted password를 저장하는 부분이였지만 현재 account에 password가 할당되었는지 표시하는 부분으로 'x'는 할당되지 않았다는 의미이다. password는 shadow 파일로 이전되었다.
3. user ID로 양수 integer가 할당
4. group id
5. 주석, 빈 칸으로 되어있을 수 있다.
6. user의 home dir
7. 해당 user가 사용하는 기본 shell, 만약 nologin이 적혀있다면 이는 해당 user로 시스템에 로그인 할 수 없다는 뜻이다. 이는 특정 서비스(web server 등)을 실행하는 목적으로만 사용되는 계정을 활용할 때 주로 사용되는 방식이다. 따라서 특정 목적 외 로그인할 필요가 없게 만든다.

### shadow

user에 대한 encrypted password들이 저장된 파일이다.  
오직 root 유저만 이 파일을 볼 수 있게 설정되어 있다.

1. user name으로 이를 통해 passwd 파일과 shadow 파일에 user를 매칭할 수 있다.
2. password, 만약 \*나 !로 나타나져 있다면 이는 해당 user로 시스템에 로그인 할 수 없다는 뜻이다. key-based authentication이나 switching user와 같은 방식으로는 user에 대한 접근이 가능하다.
   $을 통해 password format type을 나타내며 그 그 다음 $는 salt 그리고 마지막 $는 password+salt에 대한 hash 값을 나타낸다. 참고로 type에서 6은 SHA-512를 의미한다.
   서로 다른 user가 같은 password를 가진다 해도 서로 다른 고유한 salt를 가지기에 같은 hash값을 가질 수 없다.
   이후 7개의 필드에는 password 만기일, 마지막 변경일 최소,최대 유효일 등이 나타내어 있다.

## groups

user들에 대한 권한 관리를 편하게 하기 위해 사용된다.
Linux에서는 파일은 ownership은 user와 group 둘다에 속하게 된다.

primary group: user가 파일을 생성할 때 부여되는 group이다. 보통 user name과 동일하며 모든 user는 단 하나의 primary group에 속한다.
secondary groups: 부가적인 group으로 여러 user들이 같은 group에 속할 수 있다.

groups 명령어를 통해 현재 user가 속하는 모든 group을 볼 수 있다.  
만약 인자로 userName을 추가한다면 해당 user가 속하는 모든 group을 출력한다.

id 명령어를 통해 특정 user와 이에 속한 group에 대한 정보를 보여준다.

## Creating user account

### useradd

기본적으로 새로운 user와 같은 이름의 primary group에 속한 계정을 생성한다. 이와 관련된 config 파일인 login.def와 /default/useradd를 참조한다.

- 만약 어떠한 옵션도 넣지 않는다면 해당 user에 대한 home dir을 생성하지 않는다. 이를 위해서는 -m 옵션을 활용한다.  
  다른 옵션으로는 -d를 통해 home dir를 다른 이름이나 다른 위치에 생성 가능하다.
- c 옵션을 통해 해당 user에 대한 comment를 추가할 수 있다. 이를 위해서는 값을 ""로 감싸 할당한다.
- s 옵션을 통해 user가 기본적으로 사용할 shell을 직접 설정 해줄 수 있다. ex)/bin/bash
- G 옵션을 통해 user를 생성함과 동시에 기존에 존재하는 secondary group을 할당할 수 있다.
- e 옵션을 통해 user 계정이 만료되는 시점을 설정하여 해당 user를 임시로만 활용도 가능하다.

### chage

기존 user에 대한 필드 값을 변경할 수 있다.

- l 옵션을 통해 해당 user의 expiration policy를 확인 가능하다.

### passwd

생성한 유저 이름과 passwd 명령어를 통해 해당 유저의 password 설정이 가능하다.

## Changing & Removing user account

user와 관련된 정보를 가지는 파일들은 다음과 같다.

- /etc/passwd
- /etc/shadow
- /etc/group
- /etc/gshadow
- /etc/login.defs: user가 생성될 때 할당되는 기본 설정이 나타나있다.

### usermod

user와 이에 대해 변경되는 값을 옵션 이후 인자로 넣어주면 해당 user에 대한 attr을 변경할 수 있다.
set과 append를 구분해야한다. 만약 옵션에 -a를 넣지 않는다면 기존 값들은 전부 새로운 값들로 대체되기에 기존에 추가만하고 싶다면 -a를 넣어야 함을 잊지 말자.

- c 옵션을 통해 user에 대한 주석을 바꿀 수 있다.
- g 옵션을 통해 user의 primary group 변경
- G 옵션을 통해 secondary group 변경, group이 여려 개일 경우 ,로 구분

### userdel

유저를 삭제하는 명령어이다.
기본적으로 user account를 삭제해도 해당 유저에 대한 ownership을 가지는 파일들은 삭제되지 않는다. 따라서 직접 찾아서 삭제시켜야 한다.

- r 옵션을 통해 삭제되는 user와 관련된 home,mail spool dir까지 삭제 가능하다.

## Creating admin user

생성하는 user에게 admin 권한을 부여하기 위해서는 사전에 정의된 group인 sudo(Ubuntu 기준)에 user를 할당한다.
이를 통해 해당 user는 sudo 명령어를 사용해 root 권한으로 명령어를 수행할 수 있다.

이를 위해 usermod에서 -aG 옵션을 통해 유저를 sudo group에 속하게 하면 된다.

## Group Management

/etc/group 파일에 group들에 대한 정보가 저장된다.
user의 primary group은 삭제하기 전에 해당 user를 먼저 삭제해야함을 기억한다.

group관련 추가/삭제/수정에 대한 관련 명령어들이다.

- groupadd
- groupdel
- groupmod

## User Account Monitoring

각 user들의 시스템에 대한 log를 볼 수 있는 명령어들이다.
이와 관련된 user에 대한 개념 2가지가 존재한다.

- RUID(Real User Id): 처음 로그인한 user로 바뀌지 않는다.
- EUID(Effective User Id): 실질적으로 명령어를 수행하는 userid이다.

만약 처음 student 계정인 상태에서 sudo su를 통해 root 유저에 로그인한다면 RUID는 student이고 EUID는 root가 된다.

- whoami: 현재 EUID를 출력한다.
- who: RUID를 출력한다.
- id: EUID와 해당 유저와 관련된 group들을 출력한다.
- w: 현재 시스템에 로그인한 user 수와 정보들을 출력한다.
- last: user들의 활동 또는 보안 체크 시 활용될 수 있으며, 가장 최근에 로그인한 유저들에 대해 보여준다. 이는 /var/log/wtmp 파일을 읽어들여 출력하는 것이며 log in/out에 대한 정보가 있다.
  원하는 user에 대한 이름을 인자로 넣으면 해당 user에 대한 정보만 출력된다.

## Commands

```shell
##########################
## Account Management
##########################

## IMPORTANT FILES
# /etc/passwd # => users and info: username:x:uid:gid:comment:home_directory:login_shell
# /etc/shadow # => users' passwords
# /etc/group # => groups

# creating a user account
useradd [OPTIONS] username
# OPTIONS:
# -m => create home directory
# -d directory => specify another home directory
# -c "comment"
# -s shell
# -G => specify the secondary groups (must exist)
# -g => specify the primary group (must exist)

Exemple:
useradd -m -d /home/john -c "C++ Developer" -s /bin/bash -G sudo,adm,mail john

# changing a user account
usermod [OPTIONS] username # => uses the same options as useradd
Example:
usermod -aG developers,managers john # => adding the user to two secondary groups

# deleting a user account
userdel -r username # => -r removes user's home directory as well

# creating a group
groupadd group_name

# deleting a group
groupdel group_name

# displaying all groups
cat /etc/groups

# displaying the groups a user belongs to
groups

# creating admin users
# add the user to sudo group in Ubuntu and wheel group in CentOS
usermod -aG sudo john


## Monitoring Users ##
who -H # => displays logged in users
id # => displays the current user and its groups
whoami # => displays EUID

# listing who’s logged in and what’s their current process.
w
uptime

# printing information about the logins and logouts of the users
last
last -u username
```

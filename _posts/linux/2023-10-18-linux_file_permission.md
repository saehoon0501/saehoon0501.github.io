---
title: Linux File Permissions
date: 2023-10-18 4:30:00 +/-0
categories: [linux]
tags: [linux_file_permission] # TAG names should always be lowercase
---

Linux에서는 File permission을 통해 파일에 대한 access, change, execute 수행 가능한 user/group을 명시한다.
따라서 오직 허가된 user들만 해당 파일에 대한 작업을 수행할 수 있게 되는 것이다.

모든 파일들은 owner와 group 정보를 가지고 있으며, owner를 파일을 생성한 user, group은 해당 user의 primary group에 해당하게 된다.  
이러한 owner/group은 root 권한으로 별도의 명령어를 통해 변경이 가능하다.

파일에 대한 3가지 user category들:

- file owner
- group owner
- others

파일 권한 종류 3가지:

- read(r)
- write(w)
- execute(e)

요약하자면, 3가지 user category마다 3가지 권한이 존재한다.
파일 권한의 경우 dir과 파일에 대해 다른 영향을 가짐을 유의하자.

ls -l 명령어를 통해 각 파일들의 type 이후 오는 9자리 rwx- 조합을 통해 부여된 권한을 알 수 있다.

## Octal(Numeric) Notation of File Permission

각 user에 대해 rwe 파일 권한을 직접 명시할 수 있지만, 쉽게 숫자로 축약해 표현이 가능하다.

- r = 4
- w = 2
- x = 1
- \- = 0

만약 owner에 0755가 부여된다면 이는 755와 동일하며, -rwxr-xr-x 권한이 파일에 부여된 것이다.
만약 0644라면, -rw-r--r-- 권한이 부여된다.

## Changing File Permissions

명령어 chmod [who][OPERATION][permissions] filename을 통해 파일 권한을 수정 가능하다.

- who: 파일 권한이 변경될 user category, u=owner,g=group,o=others를 a의 경우 전체를 나타낸다.
- OPERATION: 어떤 권한이 삭제/추가/set 될지 나타낸다. \-의 경우 remove를 의미하며, +의 경우 add, =의 경우 set을 의미한다.

-R 옵션을 통해 dir에 있는 모든 dir과 파일에 대한 권한을 바꿀 수 있다. 하지만 파일과 dir에 적용되는 효과가 다르기에 이러한 작업은 매우 비추천

```shell
chmod u-x,g+w,o-rwx user.txt
chomd ug=rw,o= user.txt
chomd -R 750 dir1/
chomd --reference=i.txt user.txt # i.txt의 권한을 그래도 user.txt로 부여한다.
```

## Effect of Permissions on Directories

dir에 대한

- r 권한은 해당 dir에 존재하는 파일들에 대한 ls 작업을 가능케하는 것을 의미한다. 하지만 ls 명령어의 경우 자동적으로 파일의 type을 알기 위해 접근하기에 이를 방지하도록 \ls를 따로 입력해줘야 한다.
- w 권한은 파일에 대한 작업 권한을 부여하지만 e권한 없이는 아무것도 못한다.
- e 권한은 파일에 내용에 대한 접근 권한을 의미한다.

만약 dir에서는 모든 권한이 부여되어 있지만 어떠한 권한도 부여하지 않은 파일이 존재하는 경우 해당 파일의 삭제가 가능할까?
가능하다.  
parent dir의 권한이 file 권한보다 우선시 되기 때문이다.

## Combining Find and Chmod

chmod작업을 -R을 통해 한번에 하고 싶으면서도 특정 파일들만을 대상으로 하고 싶은 때 find를 같이 조합해서 사용하면 유용하다.

```shell
# 현재 dir에서 파일만을 대상으로 권한을 수정한다.
find ~ -type f -exec chmod 640 {} \;
```

## Changing File Ownership

명령어 chown과 chgrp을 통해 파일에 대한 owner와 group을 변경할 수 있다.  
이 작업은 root 권한만 가능하며, 일반 user들은 자신 또는 자신이 속한 group이 소유한 파일에 대해서만 작업 가능하다.

```shell
# cpu.txt파일의 owner를 변경
sudo chown toor cpu.txt

# userId를 통해 owner를 변경
sudo chown +1005 cpu.txt

# 한번에 owner와 group을 변경
sudo chown student.daemon cpu.txt
sudo chown student:daemon cpu.txt

# group만 변경
sudo chown :daemon cpu.txt
sudo chgrp daemon cpu.txt

# 모든 파일과 dir에 대해 변경 작업을 원하면 -R 옵션
sudo chown -R student:student ~
```

## Extra Permissions

3가지 추가적 permission이 있으며, 각각 특정한 목적을 가지고 사용된다.
이는 파일 권한을 숫자로 표현 시 맨 앞자리 숫자에서 나타난다.  
문자로 표현할 경우 x자리에 s로 표시된다. 만얀 S가 존재한다면 e권한이 없다는 뜻이다.

### SUID(4XXX)

기본적으로 명령어를 수행할 때 현재 로그인한 계정의 권한으로 이를 수행한다.  
하지만 SUID를 설정하면 해당 명령어를 수행하는 권한은 그 명령어 파일에 owner가 되게 된다.  
이는 잘못된 사용의 경우 보안 문제가 발생할 수 있지만 경우에 따라 user들이 root권한을 필요로하는 작업이 요구되기 때문에 그럴 때 사용된다.

```shell
# cat 명령어에 대한 권한과 timestamp를 출력
stat /usr/bin/cat

#cat 명령어에 SUID 부여
sudo chmod 4755 /usr/bin/cat
sudo chmod u+s /usr/bin/cat

#SUID 설정이 된 명령어들 찾기
find /usr/bin/ -perm -4000
```

### SGID(2XXX)

SGID는 SUID와 유사하며, 다른 점은 파일의 group의 권한으로 실행된다는 점이다.  
만약 dir에 SGID를 설정하면, 그 dir내 생성되는 모든 dir과 파일들은 모두 parent dir의 group과 같은 group에게 소유되게 된다.
SGID는 공유 폴더를 운영할 때 유용하며, parent dir에 부여된 group에 속한 user들만 해당 dir내 파일들에 작업을 수행할 수 있게 관리가 가능해진다.

```shell
# dir에 대한 SGID 설정
chmod 2770 /dir/
chmod g+s /dir/
```

### Sticky Bit(1XXX)

dir에 적용되며, user는 자신이 소유하거나 명시적으로 w 권한을 부여받은 파일에 대해서만 삭제 작업이 가능하다.  
이는 dir에서 we권한을 부여받은 상황에서도 적용된다.
따라서 공유되는 폴더에서 user들은 삭제 작업에 대한 수행 제한 등에 유용하게 사용될 수 있다.

```shell
chmod 1770 /dir/
chmod o+t /dir/
```

## Umask

umask 설정을 통해 기본적으로 생성되는 파일의 권한을 설정할 수 있다.  
기본적으로 파일에 대한 권한은 0666, dir에 대한 권한은 0777이다.  
여기서 umask를 설정을 0002로 하면,

- 파일 생성 시, 0664
- dir 생성 시, 0775
  가 기본 설정이 되게 되는 것이다.

umask 명령어를 통해 umask를 변경할 수 있지만 이는 현재 shell session에서만 유지되며 영구적인 반영은 bashrc 파일에 가서 명령어를 추가해야 한다.

```shell
umask 0002
```

## Files Attributes

Linux에서는 기본적인 3가지 파일 권한 뿐 아니라 ACLs(Access Control List)를 제공하고 그 attr를 제공한다.

chattr 명령어를 통해 attr를 설정 가능하며, lsattr를 통해 파일의 attr 확인이 가능하다.

- i 옵션을 통해 파일을 수정 불가 상태로 만들 수 있다.
- a 옵션을 통해 파일에 추가 작업만 가능하게 할 수 있다.
- A 옵션을 통해 파일의 acces time 수정을 방지할 수 있다.(이를 통해 불필요한 disk io를 줄일 수 있음)

```shell
sudo chattr +i i.txt
sudo chattr +a i.txt
sudo chattr -i i.txt
```

## Commands

```shell
##########################
## File Permissions
##########################

## LEGEND
u = User
g = Group
o = Others/World
a = all

r = Read
w = write
x = execute
- = no access

# displaying the permissions (ls and stat)
ls -l /etc/passwd
    -rw-r--r-- 1 root root 2871 aug 22 14:43 /etc/passwd

stat /etc/shadow
    File: /etc/shadow
    Size: 1721      	Blocks: 8          IO Block: 4096   regular file
    Device: 805h/2053d	Inode: 524451      Links: 1
    Access: (0640/-rw-r-----)  Uid: (    0/    root)   Gid: (   42/  shadow)
    Access: 2020-08-24 11:31:49.506277118 +0300
    Modify: 2020-08-22 14:43:36.326651384 +0300
    Change: 2020-08-22 14:43:36.342652202 +0300
    Birth: -

# changing the permissions using the relative (symbolic) mode
chmod u+r filename
chmod u+r,g-wx,o-rwx filename
chmod ug+rwx,o-wx filename
chmod ugo+x filename
chmod a+r,a-wx filename

# changing the permissions using the absolute (octal) mode
PERMISSIONS      EXAMPLE
u   g   o
rwx rwx rwx     chmod 777 filename
rwx rwx r-x     chmod 775 filename
rwx r-x r-x     chmod 755 filename
rwx r-x ---     chmod 750 filename
rw- rw- r--     chmod 664 filename
rw- r-- r--     chmod 644 filename
rw- r-- ---     chmod 640 filename

# setting the permissions as of a reference file
chmod --reference=file1 file2

# changing permissions recursively
chmod -R u+rw,o-rwx filename

## SUID (Set User ID)

# displaying the SUID permission
ls -l /usr/bin/umount
    -rwsr-xr-x 1 root root 39144 apr  2 18:29 /usr/bin/umount

stat /usr/bin/umount
    File: /usr/bin/umount
    Size: 39144     	Blocks: 80         IO Block: 4096   regular file
    Device: 805h/2053d	Inode: 918756      Links: 1
    Access: (4755/-rwsr-xr-x)  Uid: (    0/    root)   Gid: (    0/    root)
    Access: 2020-08-22 14:35:46.763999798 +0300
    Modify: 2020-04-02 18:29:40.000000000 +0300
    Change: 2020-06-30 18:27:32.851134521 +0300
    Birth: -

# setting SUID
chmod u+s executable_file
chmod 4XXX executable_file      # => Ex: chmod 4755 script.sh


## SGID (Set Group ID)

# displaying the SGID permission
ls -ld projects/
    drwxr-s--- 2 student student 4096 aug 25 11:02 projects/

stat projects/
    File: projects/
    Size: 4096      	Blocks: 8          IO Block: 4096   directory
    Device: 805h/2053d	Inode: 266193      Links: 2
    Access: (2750/drwxr-s---)  Uid: ( 1001/ student)   Gid: ( 1002/ student)
    Access: 2020-08-25 11:02:15.013355559 +0300
    Modify: 2020-08-25 11:02:15.013355559 +0300
    Change: 2020-08-25 11:02:19.157290764 +0300
    Birth: -

# setting SGID
chmod 2750 projects/
chmod g+s projects/


## The Sticky Bit

# displaying the sticky bit permission
ls -ld /tmp/
    drwxrwxrwt 20 root root 4096 aug 25 10:49 /tmp/

stat /tmp/
    File: /tmp/
    Size: 4096      	Blocks: 8          IO Block: 4096   directory
    Device: 805h/2053d	Inode: 786434      Links: 20
    Access: (1777/drwxrwxrwt)  Uid: (    0/    root)   Gid: (    0/    root)
    Access: 2020-08-22 14:46:03.259455125 +0300
    Modify: 2020-08-25 10:49:53.756211470 +0300
    Change: 2020-08-25 10:49:53.756211470 +0300
    Birth: -

# setting the sticky bit
mkdir temp
chmod 1777 temp/
chmod o+t temp/
ls -ld temp/
    drwxrwxrwt 2 student student 4096 aug 25 11:04 temp/


## UMASK
# displaying the UMASK
umask

# setting a new umask value
umask new_value     # => Ex: umask 0022

## Changing File Ownership (root only)

# changing the owner
chown new_owner file/directory      # => Ex: sudo chown john a.txt

# changing the group owner
chgrp new_group file/directory

# changing both the owner and the group owner
chown new_owner:new_group file/directory

# changing recursively the owner or the group owner
chown -R new-owner file/directory

# displaying the file attributes
lsattr filename

#changing the file attributes
chattr +-attribute filename     # => Ex: sudo chattr +i report.txt
```

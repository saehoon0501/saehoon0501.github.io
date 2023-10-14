---
title: The Linux File System
date: 2023-10-12 9:30:00 +/-0
categories: [linux]
tags: [linux_file_system] # TAG names should always be lowercase
---

Linux에서는 모든 저장된 데이터를 file로 보여주며 이를 관리하는 논리적 규칙을 file system이라 한다.  
Linux에서는 하나의 root dir에서 시작하는 Hiearchy를 통해 file system이 이뤄져 있으며, root dir에는 각 목적에 맞게 파일들이 저장되는 여러 dir들이 존재하며 이를 FHS(Filesystem Hiearchy Standard)라 부른다.  
Window에서 disk partition에 따라 나눠서 파일을 보여주지만 Linux에서는 실제 물리적으로는 다른 장소에 저장된 데이터일지라도 Linux 상에서는 모두 root dir 아래에 나타나진다.

## FHS(Filesystem Hiearchy Standard)

가장 기초적인 dir들에 대해서 나열하면,

- /bin: 모든 유저들이 접근 가능한 바이너리 또는 유저 실행 파일들
- /sbin: superuser만 수행 가능한 파일들
- /boot: Linux를 시작하는데 필요한 파일들
- /home: user마다 고유하게 가지는 dir로 해당 유저에 관한 파일들만 저장
- /etc: 대부분의 system 전체적인 config 파일들
- /tmp: 실행 중인 어플리케이션에서 임시적으로 파일을 저장하는 곳
- /proc: 가상 dir로 컴퓨터 하드웨어에 대한 정보를 컴퓨터 실행 중에 생성하는 곳으로 이 정보를 가진 파일은 계속해서 업데이트 된다.
- /sys: device, driver 등의 정보를 가진다.

## File Path

파일이나 dir를 위한 고유한 location으로 absolute와 relative가 존재한다.

- Absolute Path: 항상 root dir에서 시작해 해당 파일 위치까지 heiarchy를 나타내는 방식으로 path를 표현한다.
- Relative Path: 현재 작업 위치를 기준으로 상대적으로 해당 파일의 위치의 heiarchy를 나타내는 방식으로 path를 표현한다.
  .은 현재 위치를 의미하며 ..은 현재 위치에서 부모 dir을 의미한다. ~는 현재 유저의 home dir를 의미한다.

명령어 pwd를 통해 현재 작업 중인 위치의 absolute path를 가져올 수 있다.
명령어 tree를 통해 해당 위치에 있는 모든 파일 또는 dir을 출력할 수 있다.

```shell
##########################
## Linux Paths
##########################

.       # => the current working directory
..      # => the parent directory
~       # => the user's home directory

cd      # => changing the current directory to user's home directory
cd ~    # => changing the current directory to user's home directory
cd -    # => changing the current directory to the last directory
cd /path_to_dir    # => changing the current directory to path_to_dir
pwd     # => printing the current working directory

# installing tree
sudo apt install tree

tree directory/     # => Ex: tree .
tree -d .           # => prints only directories
tree -f .           # => prints absolute paths
```

## LS

dir에 존재하는 파일들을 listing하는 명령어로 매우 자주 사용된다.
인자로 path를 주어 해당 path에 존재하는 파일들을 볼 수 있다.

- 옵션 -a는 숨겨진 파일들가지 모두 보여준다.
- 옵션 -l은 long format을 통해 파일들과 dir를 보여주며, 형식은 아래와 같다.
  owner,group owner,others 순으로 이에 대한 file permission, hardlink 수, owner, group owner, inode size, 수정일, 이름
- 실제 파일 크기는 du -sh를 통해 표시 가능하다.
- --hide=regex를 통해 특정 이름의 파일이나 dir들을 filtering하여 출력할 수 있다.
- -X를 통해 extension별로 sorting 가능하다.
- -R을 통해 재귀적으로 dir들을 ls 가능하다.

```shell
##########################

## The ls Command

## ls [OPTIONS] [FILES]

##########################

# listing the current directory
# ~ => user's home directory
# . => current directory
# .. => parent directory
ls

ls .

# listing more directories
ls ~ /var /

# -l => long listing
ls -l ~

# -a => listing all files and directories including hidden ones
ls -la ~

# -1 => listing on a single column
ls -1 /etc

# -d => displaying information about the directory, not about its contents
ls -ld /etc

# -h => displaying the size in human readable format
ls -h /etc

# -S => displaying sorting by size
ls -Sh /var/log

# Note: ls does not display the size of a directory and all its contents. Use du instead
du -sh ~

# -X => displaying sorting by extension
ls -lX /etc

# --hide => hiding some files
ls --hide=*.log /var/log

# -R => displaying a directory recursively
ls -lR ~

# -i => displaying the inode number
ls -li /etc
```

## File Timestamps

Linux에서는 1/1/1970을 시작으로 지금까지 흐른 시간을 초단위 integer로 timestamp를 나타낸다.
timestamp에는 3가지 종류가 있다.

- atime: 가장 최근 접근된 시간
- mtime: 가장 최근 수정된 시간
- ctime: 가장 최근 파일의 metadata가 수정된 시간

ls를 통해 timestamp 전체를 보고 싶다면 --full-time 옵션을 추가한다.  
파일의 timestamp을 수정하고 싶다면 touch -mac 옵션을 원하는 timestamp에 맞게 넣으면 된다.
다른 파일의 timestamp로 파일의 timestamp을 수정하고 싶다면 -r 옵션을 사용한다.

만약 file 이름에 whitespace를 굳이 넣어야 된다면 파일 이름을 ""로 감싸자.

### Sorting Files by Timestamp

파일의 mtime으로 sort하고 싶다면 ls에 -lt 옵션을 추가한다.
atime으로는 -ltu를 사용한다.  
이름으로 sorting은 -lu를 사용한다.  
만약 sorting을 reverse하고 싶다면 sorting 옵션 뒤에 -r 또는 --reverse를 추가한다.

```shell
# displaying atime
ls -lu

# displaying mtime
ls -l
ls -lt

# displaying ctime
ls -lc

# displaying all timestamps
stat file.txt

# displaying the full timestamp
ls -l --full-time /etc/

# creating an empty file if it does not exist, update the timestamps if the file exists
touch file.txt

# changing only the access time to current time
touch -a file

# changing only the modification time to current time
touch -m file

# changing the modification time to a specific date and time
touch -m -t 201812301530.45 a.txt

# changing both atime and mtime to a specific date and time
touch -d "2010-10-31 15:45:30" a.txt

# changing the timestamp of a.txt to those of b.txt
touch a.txt -r b.txt

# displaying the date and time
date

# showing this month's calendar
cal

# showing the calendar of a specific year
cal 2021

# showing the calendar of a specific month and year
cal 7 2021

# showing the calendar of previous, current and next month
cal -3

# setting the date and time
date --set="2 OCT 2020 18:00:00"

# displaying the modification time and sorting the output by name.
ls -l

# displaying the output sorted by modification time, newest files first
ls -lt

# displaying and sorting by atime
ls -ltu

# reversing the sorting order
ls -ltu --reverse
```

## File Types

Linux에서는 File의 Type은 extension이 아닌 File Header에 저장된다. 따라서 extension 이름을 바꿔도 Type은 변하지 않는다.  
명령어 file을 통해 해당 파일의 metadata를 확인할 수 있다.
GUI에서는 특정 파일을 실행하기 위해 파일 이름에 적절한 extension이 필요할 수 있다.

ls -l에서 출력되는 파일들의 내용 중 맨 앞 문자를 통해 File Type을 알 수 있다.

- -는 실행 불가능한 일반 파일
- d는 dir
- l는 symbolic link
- b는 block device
- c는 char device
- p는 named pipe
- s는 socket(Networking과 관련없고 process간 통신에 사용된다.)

또 다른 방법은 ls -F를 통해 출력되는 파일 맨 뒤의 문자를 보고 알 수 있다.

## Viewing Files

Linux의 모든 config 파일들은 text 파일로 저장되기에 이러한 text파일을 Terminal 상에서 보는 방법에 대해 알아본다.

- cat 명령어는 text파일의 모든 내용을 한번에 출력해준다. 따라서 작은 파일을 대상으로 사용하기 좋으며, -n 옵션을 통해 n번째 줄까지 표시 가능하다.  
  cat 파일path... 이후 >을 통해 여러 파일들을 concatenate한 새로운 파일을 생성할 수 있다.
- less 명령어를 통해 큰 text 파일을 Terminal 크기에 맞춰 출력 후 스크롤 가능하다.(man page가 이를 사용)  
  방향키로 스크롤이 가능하며 g와 G를 통해 파일의 맨 앞뒤로 이동 가능하다.
- tail 명령어를 통해 파일의 마지막 10줄을 출력 가능하다. -n 옵션을 통해 원하는 마지막 n개의 줄을 출력 가능하다. ex) tail -n 20
  숫자 앞에 +를 붙이면 해당 번째 줄에서 시작해 끝까지 출력이 가능하다.
  -f를 통해 해당 파일을 watch하여 실시간으로 추가되는 내용들을 확인 가능하다. 보통 log 파일에 생성되는 내용들을 모니터링할 때 유용하다.
- head 명령어는 파일의 시작 10줄을 출력하며, -n을 통해 시작에서 n번째 줄까지 출력 가능하다.
- watch와 다른 명령어를 통해 주기적으로 명령어를 실행하여 변화를 모니터링 가능하다. -n 옵션을 통해 interval을 설정 가능하다.

```shell
# displaying the contents of a file
cat filename

# displaying more files
cat filename1 filename2

# displaying the line numbers
can -n filename

# concatenating 2 files
cat filename1 filename2 > filename3

# viewing a file using less
less filename

# less shortcuts:
# h         => getting help
# q         => quit
# enter     => show next line
# space     => show next screen
# /string   => search forward for a string
# ?string   => search backwards for a string
# n / N     => next/previous appearance


# showing the last 10 lines of a file
tail filename

# showing the last 15 lines of a file
tail -n 15 filename

# showing the last lines of a file starting with line no. 5
tail -n +5 filename

# showing the last 10 lines of the file in real-time
tail -f filename


# showing the first 10 lines of a file
head filename

# showing the first 15 lines of a file
head -n 15 filename

# running repeatedly a command with refresh of 3 seconds
watch -n 3 ls -l
```

## Creating Files and Directories

- touch 명령어를 통해 기존에 파일이름이 존재하지 않는다면 해당 이름의 파일을 생성한다.(파일 이름은 case-sensitive하다.)  
  만약 이미 기존 파일이 존재한다면 해당 파일의 timestamp를 현재 date로 수정한다.
- mkdir 명령어를 통해 dir를 생성 가능하다. backslash를 통해 dir structure을 한번에 생성하며 파일 path를 끝에서부터 수행하기에 -p옵션을 통해 parent가 없으면 생성하도록 하게해야 한다.
- cp 명령어를 통해 파일 또는 dir을 기존 파일에서 원하는 파일로 내용을 복사 가능하다. 만약 원하는 파일이 존재하지 않는다면 생성한다.
  만약 기존 파일에 복사를 한다면 전부 덮어쓰기가 되며, 이를 묻는 prompt를 원한다면 -i 옵션을 추가한다.
  -v 옵션을 통해 복사되는 내용을 복사를 수행 후 출력해준다.
  -r 옵션을 통해 dir에 있는 모든 파일을 원하는 위치에 재귀적으로 복사한다.
  cp를 수행 시 이를 수행하는 user로 owner가 변경되기에 기존 파일의 attribute(permission, ownership, timestamp)와 다르게 된다. 만약 attribute까지 복사하고 싶다면 -p 옵션을 추가한다.
- mv 명령어를 통해 기존 파일의 path을 원하는 위치로 바꾸거나 파일명을 수정할 수 있다.  
  만약 기존 파일의 path을 regex로 작성한다면 해당 regex에 맞는 모든 파일들이 옮겨진다.
  옮겨지는 위치에 같은 이름의 파일이 있다면 묻지않고 덮어쓰기에 -i 옵션을 추가해 이를 묻는 prompt를 만들 수 있다.  
  -n옵션을 통해 옮기는 위치에 이미 존재하는 파일은 옮기지 않는다.  
  -u옵션을 통해 src파일이 destin파일보다 더 최근에 생성된 파일인 경우 또는 존재하지 않을 때에만 옮기기를 수행한다.  
  만약 파일을 옮기는 위치가 기존 위치와 동일하다면 파일명을 수정한다. 따라서 기존 파일을 옮길 때 이름을 바꿔서 옮기면 파일명과 위치를 모두 변경 가능하다.
- rm 명령어를 통해 파일을 삭제하면 영구적으로 반영되고 복구 불가능하니 유의한다.  
  -i옵션을 통해 삭제 허가 prompt를 생성할 수 있다.
  -v옵션을 통해 삭제된 파일의 결과를 출력할 수 있다.
  단순히 rm 명령어를 통해 dir를 삭제할 수 없으며, 이를 위해서는 -r옵션을 추가해야 한다. 그리고 -f를 추가해 삭제 허가를 묻지않고 수행 가능하다.  
  ex) rm -rf dir/dir2 해당 dir에 존재하는 모든 파일과 dir을 묻지않고 삭제한다. 복구 불가능
  typo로 인해 실수를 방지하기 위해 항상 tab을 통해 auto-completion을 사용하자.
  만약 regex를 통한 pattern을 통해 파일 삭제를 하고 싶으면 먼저 echo를 통해 대상이 되는 파일들을 확인하자.
- shred 명령어를 통해 hard disk에서 아예 복구하지 못하도록 삭제하기 전에 대상이 되는 파일들을 여러번 덮어쓰기를 수행한다.
  -n 옵션을 통해 덮어쓰기 수행 횟수를 설정 가능하다.

```shell
  # creating a new file or updating the timestamps if the file already exists
touch filename

# creating a new directory
mkdir dir1

# creating a directory and its parents as well
mkdir -p mydir1/mydir2/mydir3

######################
### The cp command ###
######################
# copying file1 to file2 in the current directory
cp file1 file2

# copying file1 to dir1 as another name (file2)
cp file1 dir1/file2

# copying a file prompting the user if it overwrites the destination
cp -i file1 file2

# preserving the file permissions, group and ownership when copying
cp -p file1 file2

# being verbose
cp -v file1 file2

# recursively copying dir1 to dir2 in the current directory
cp -r dir1 dir2/

# copy more source files and directories to a destination directory
cp -r file1 file2 dir1 dir2 destination_directory/


######################
### The mv command ###
######################
# renaming file1 to file2
mv file1 file2

# moving file1 to dir1
mv file1 dir1/

# moving a file prompting the user if it overwrites the destination file
mv -i file1 dir1/

# preventing a existing file from being overwritten
mv -n file1 dir1/

# moving only if the source file is newer than the destination file or when the destination file is missing
mv -u file1 dir1/

# moving file1 to dir1 as file2
mv file1 dir1/file2

# moving more source files and directories to a destination directory
mv file1 file2 dir1/ dir2/ destination_directory/

######################
### The rm command ###
######################
# removing a file
rm file1

# being verbose when removing a file
rm -v file1

# removing a directory
rm -r dir1/

# removing a directory without prompting
rm -rf dir1/

# removing a file and a directory prompting the user for confirmation
rm -ri fil1 dir1/

# secure removal of a file (verbose with 100 rounds of overwriting)
shred -vu -n 100 file1
```

## Pipe

Pipe(|)을 통해 Linux command들을 chaining하여 각각의 일을 수행하는 command들을 조합해 복잡한 일을 수행할 수 있게 된다.  
Pipe를 통해 한 command의 수행결과를 다른 command의 input으로 활용할 수 있다.
ex) ls -lSh /etc/ | head

wc 명령어는 word count의 약자로써 output이 몇 개의 줄, 몇개의 단어 그리고 몇 byte인지 counting한 결과를 보여준다.
ex) wc /etc/passwd
grep 명령어는 text output에서 원하는 keyword를 포함하는 내용만 찾아 출력해준다.

이러한 명령어들을 Pipe와 함께 사용하여 output을 원하는 output 형태로 filtering하는 stream을 만들 수 있다.

## Command Redirection

기본적으로 output을 화면에 출력하지만 redirection(>)을 통해 file에 save하는 등 output을 다른 곳으로 보내줄 수 있다.  
ex) ifconfig > output.txt  
output이 저장되는 파일은 없으면 생성되고 있으면 덮어쓰기 된다. 만약 덮어쓰기를 원하지 않으면 >>을 통해 output을 기존의 내용 끝에 추가할 수 있다.

Linux 표준 입력은 0, 표준 결과는 1, 표준 에러는 2의 숫자를 가진다.  
이러한 숫자를 >앞에 붙여서 특정 stream으로 output을 보내줄 수 있다.  
ex) tail -n 2 /etc/passwd /etc/shadow > output.txt 2>&1
&은 stream과 파일명을 구분하기 위해 붙여주는 것으로 tail의 결과를 output.txt로 보내고 에러 결과는 표준 결과 stream으로 보내는 것을 의미한다. 여기서는 표준 결과 stream이 output.txt로 결과를 보내고 있기에 에러 결과 또한 output.txt에 보내지게 된다.  
만약 & 없이 단순히 숫자만 작성한다면 이는 숫자를 파일명으로 인식한다.

redirection는 pipe와도 사용 가능하다, ex) ifconfig | grep ether | cut -d" " -f10 > mac.txt
tee 명령어를 통해 화면과 파일 둘다 동시에 결과를 저장할 수 있다. ex) ifconfig | grep ether | tee mac.txt

```shell
## Piping Examples:

ls -lSh /etc/ | head            # see the first 10 files by size
ps -ef | grep sshd              # checking if sshd is running
ps aux --sort=-%mem | head -n 3  # showing the first 3 process by memory consumption

## Command Redirection

# output redirection
ps aux > running_processes.txt
who -H > loggedin_users.txt

# appending to a file
id >> loggedin_users.txt

# output and error redirection
tail -n 10 /var/log/*.log > output.txt 2> errors.txt

# redirecting both the output and errors to the same file
tail -n 2 /etc/passwd /etc/shadow > output_errors.txt 2>&1

cat -n /var/log/auth.log | grep -ai "authentication failure" | wc -l
cat -n /var/log/auth.log | grep -ai "authentication failure" > auth.txt     # => piping and redirection
```

## Finding Files and Dir

file을 찾는 관련 명령어로는 locate과 which가 존재한다.
Ubuntu에서는 locate을 위해 mlocate를 apt를 통해 설치해야 한다.

- find는 실시간으로 모든 파일들을 찾으며, 다양한 옵션들을 제공한다. 따라서 파일명 뿐만아니라 owner, permission을 대상으로 파일을 찾을 수 있다.  
  -name으로 파일명을 찾을 수 있으며, -iname을 통해 case-insensitive하게 파일명을 검색 가능하다.
  -delete를 통해 찾은 파일을 삭제할 수 있다.  
  -ls를 통해 찾은 파일에 대한 ls를 수행한 결과를 받아 볼 수 있다.
  -type을 통해 찾은 결과 중 해당 type에 해당하는 결과만 받아 볼 수 있다.  
  -maxdepth 옵션을 통해 결과를 찾는 검색 내용의 최대 depth를 설정할 수 있다.  
  -perm을 통해 결과에서 특정 permission에 해당하는 파일만 가져올 수 있다.  
  -size를 통해 결과에서 특정 크기에 해당하는 파일만 가져올 수 있으며, 이러한 크기 범위 또한 +숫자와 -숫자를 통해 지정할 수 있다.  
  -atime/mtime/ctime 옵션을 통해 특정 timestamp에 해당하는 결과만 가져올 수 있다.
  -user를 통해 결과에서 특정 owner에 해당하는 파일만 가져올 수 있다.
  -not을 통해 나열한 조건에 대한 negation 결과를 가져올 수 있다.
  -exec을 통해 찾은 파일에 대해 원하는 command를 수행할 수 있다. ex) sudo find /etc -type f -mtime 0 -exec cat {} \; {}에는 찾은 파일명이 들어가게 된다. 그리고 \;을 통해 command가 끝난다는 것을 알려준다.
  -ok는 exec와 같은 기능을 수행하지만 모든 결과에 대한 작업 허가 prompt를 출력해준다.
- locate은 이름을 찾아 해당 이름을 포함하는 path 리스트를 리턴한다. 이는 이전에 빌드된 DB를 이용한다. locate을 위한 DB를 새로 빌드하기 위해서는 sudo updatedb를 입력하면 되며 주기적으로 업데이트 해줘야 locate을 통해 파일을 찾을 수 있다.  
  기본적으로 absolute path 전체를 보지만 -b를 통해 파일 basename만을 보고 결과를 가져올 수 있다.
  포함이 아닌 정확히 일치하는 결과만 가져오고 싶다면 \을 이름앞에 사용 후 ''로 감싼다.
  때로는 db가 업데이트 되지 않아 더이상 존재하지 않는 결과를 가져올 수 있기에 -e 옵션을 통해 이를 확실히 체크할 수 있다.
  옵션 -r는 regex를 사용해 파일을 찾을 수 있게 해준다. ex) locate -br '^shadow\.[0-9]'
- which 명령어는 실행 가능한 명령어가 저장된 absolute path에 대한 정보를 출력해준다.

```shell
## LOCATE ##
# updating the locate db
sudo updatedb

# displaying statistics
locate -S

# finding file by name
locate filename # => filename is expanded to *filename*
locate -i filename # => the filename is case insensitive
locate -b '\filename' # => finding by exact name

# finding using the basename
locate -b filename

# finding using regular expressions
locate -r 'regex'

# checking that the file exists
locate -e filename

# showing command path
which command
which -a command


## FIND ##
find PATH OPTIONS

# Example: find ~ -type f -size +1M # => finding all files in ~ bigger than 1 MB

## Options:
# -type f, d, l, s, p
# -name filename
# -iname filename # => case-insensitive
# -size n, +n, -n
# -perm permissions
# -links n, +n, -n
# -atime n, -mtime n, ctime n
# -user owner
# -group group_owner
```

## Searching for String Pattern in text files

grep을 통해 원하는 keyword를 text파일에서 찾을 수 있으며, pipe와 함께 매우매우 자주 사용되는 명령어이다.  
ex) grep "SSH" /etc/ssh/ssh_config

-i 옵션을 통해 case-insensitive하게 keyword를 찾을 수 있다.  
-n 옵션을 통해 찾은 결과에 대한 n번째 줄에 대한 정보를 출력한다.  
-w 옵션을 통해 단어와 정확히 일치하는 결과만을 가져올 수 있다.
-v 옵션을 통해 기존 결과를 invert한 결과를 가져올 수 있다.
-a 옵션을 통해 binary 파일에 대한 keyword를 검색한 결과를 가져올 수 있다.  
-r 옵션을 통해 하위 dir들까지 검색한 결과를 가져올 수 있다.

추가적으로 string 명령어를 통해 바이너리 파일에 있는 문자를 출력할 수 있다.

```shell
grep [OPTIONS] pattern file

Options:
-n          # => print line number
-i          # => case insensitive
-v          # inverse the match
-w          # search for whole words
-a          # search in binary files
-R          # search in directory recursively
-c          # display only the no. of matches
-C n        # display a context (n lines before and after the match)


# printing ASCII chars from a binary file
strings binary_file
```

## Comparing Files

명령어의 서로 다른 2개의 output을 비교하거나 config 파일을 비교하는 등 유용하게 쓰이는 명령어들이 존재한다.

- cmp 명령어를 통해 두 개의 파일(바이너리 포함)을 비교할 수 있다. Byte별로 비교하여 처음으로 다른 부분을 출력한다.  
  만약 동일하다면 아무런 출력을 하지 않는다.
- sha256 명령어를 통해 두 명령어의 hash 값을 비교해 동일한지 알 수 있다.
- diff 명령어를 통해 text 파일간 줄별 비교결과를 가져올 수 있다. -B 옵션을 통해 빈 줄을 -w를 통해 빈 칸을 -i를 통해 대소문자 다른점 무시를 할 수 있다.
  만약 두 파일을 양쪽에 띄워 비교하고 싶다면 -y 옵션을 사용한다.
- patch 명령어를 통해 diff의 결과를 기존 오리지널 파일에 적용하여 수정을 가할 수 있다.

## VIM Text Editor

많은 Linux에서 기본적으로 사용하는 VIM에 대해서 알아보자.

VIM에서는 3가지 기본 모드가 존재한다.

- command: 입력하는 문자들이 명령어로 인식된다.
- insert: i(또는 l,a,A,o,O)를 눌러 text를 수정할 수 있는 모드로 들어갈 수 있다.
- last line: :문자를 입력하면 들어가는 모드로 맨 마지막 줄로 커서가 이동한다. 이를 통해 vim을 나가거나 파일을 저장할 수 있다.

```shell
Modes of operation: Command, Insert, and Last Line Modes.
VIM Config File: ~/.vimrc

# Entering the Insert Mode from the Command Mode
i  => insert before the cursor
I  => insert at the beginning of the line
a  => insert after the cursor
A  => insert at the end of the line
o  => insert on the next line

# Entering the Last Line Mode from the Command Mode
:

# Returning to Command Mode from Insert or Last Line Mode
ESC

# Shortcuts in Last Line Mode
w!  => write/save the file
q!  => quit the file without saving
wq! => save/write and quit
e!  => undo to the last saved version of the file
set nu => set line numbers
set nonu  => unset line numbers
syntax on|off
%s/search_string/replace_string/g

# Shortcuts in Command Mode
x   => remove char under the cursor
dd  => cut the current line
5dd => cut 5 lines
ZZ  => save and quit
u   => undo
G   => move to the end of file
$   => move to the end of line
0 or ^  => move to the beginning of file
:n (Ex :10) => move to line n
Shift+v     => select the current line
y           => yank/copy to clipboard
p           => paste after the cursor
P           => paste before the cursor
/string     => search for string forward
?string     => search for string backward
n           => next occurrence
N           => previous occurrence

# Opening more files in stacked windows
vim -o file1 file2

# Opening more files and highlighting the differences
vim -d file1 file2
Ctrl+w => move between files
```

home dir에 .vimrc파일을 vim을 통해 vim에 대한 config파일을 작성할 수 있다.
만약 vim 실행을 비정상적으로 종료하면 swp파일을 통해 복구 가능하며 이를 원하지 않을 시 swp파일을 삭제한다.

## Compressing and Archiving Files

Compressing: 파일 압축(크기 감소)
Archiving: 파일 모음(크기 동일)

명령어 tar를 통해 Archiving과 Compressing을 둘 다 수행할 수 있다.

- c 옵션을 통해 archive를 생성한다.
- z 옵션을 통해 gzip을 같이 이용해 압축과 archive를 둘다 수행한다.
- f 옵션을 통해 tar을 수행하는 대상 또는 결과의 archive명을 정할 수 있다.
- v 옵션을 통해 수행에 포함되는 파일들을 출력한 결과를 확인할 수 있다.
- j 옵션을 통해 압축에 gzip 대신 bz2을 사용할 수 있다.
- --exclude 옵션을 통해 특정 파일 이름들은 수행에서 제외할 수 있다.
- x 옵션을 통해 archive에서 파일들을 추출할 수 있다. 추출의 경우 압축에 사용된 프로그램을 알맞게 명시하지 않으면 에러 발생한다. 아예 명시하지 않으면 알아서 알맞는 것을 사용한다.

Linux에서 작업하면서 config파일들이 저장되는 /etc/ dir을 백업하는 경우가 많다.  
그리고 이러한 백업을 수행하는 날짜를 이름에 포함하는 경우가 많으며 이 때 사용하는 명령어는 다음과 같다.

```shell
tar -cjvf etc-$(date +%F).tar.bz2 /etc/
```

## Hard Links and the inode Structure

Linux에서 dir은 table로 구성되며, 한쪽은 파일명 그리고 다른 쪽은 이와 매치되는 inode 번호를 가진다. 그리고 Linux의 모든 파일마다 연관된 index node(inode)를 가진다.  
inode는 파일의 Type,Permission,Owner,Group 등등의 metadata를 가지며 실제 파일의 내용과 이름을 제외한 모든 것을 포함한다.
각 inode는 고유한 식별자를 정수로 가지며 이는 ls - i를 통해 확인 가능하다.

inode number는 실제 파일구조와 이름을 연결하는 hard link로, 이를 통해 하나의 파일이 여러 파일명과 연결될 수 있다.
inode number는 다른 dir path에 있는 파일명과 매치될 수 없다.

### Symbolic links and Hard links

- Hard link: inode와 파일 이름의 연관 관계를 나타낸다. dir에 대해서는 생성 불가능
- Symbolic Link: 다른 파일이나 dir path에 대한 reference를 가지는 파일 타입이다. 따라서 reference 대상이 되는 파일의 위치가 변경되면 symlink는 작동하지 않는다.

```shell
#현재 dir에서 inode 번호와 매치되는 파일명들을 출력한다.
find . -inum number

#/usr dir에 있는 파일 중 hard link가 1개 이상인 파일들을 출력한다.
find /usr/ -type f -links +1

#pswd 이름으로 etc/passwd 파일에 대한 symlink 생성
ln -s /etc/passwd ./pswd
```

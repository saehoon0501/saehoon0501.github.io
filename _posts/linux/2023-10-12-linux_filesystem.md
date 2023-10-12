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

- find는 실시간으로 모든 파일들을 찾으며, 다양한 옵션들을 제공한다.
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

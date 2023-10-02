---

layout: post
title: creat(), close(), lseek(), write()
date: 2023-10-01 00:00:00 +0800
categories: [CS Study, unix]
tags: [Sophomore]
pin: true

---

creat()
--------

```c
#include <fcntl.h>

int creat(const char *pathname, mode_t mode); // 파일 경로 및 이름, 파일의 권한
```

#### 예시  

```c
#include <stdio.h>
#include <fcntl.h>

int main(void)
{
        int fd;

        if ((fd = creat("testfile", 0744))<0) // testfile을 생성, 0744는 아래 설명
                printf("create error\n"); // fd가 음수일 때, 즉 생성을 실패했을 때
}
```  


#### 파일 권한  
  
0. 첫 번째 자리 - 특별 권한  
    * 4: 소유자 권한으로 실행  
    * 2: 파일 그룹 권한으로 실행  
    * 1: Sticky Bit(부여된 디렉토리 내의 파일은 파일 소유자만이 삭제 가능)
1. 두 번째 자리 - 소유자 권한 (Owner Permissions)
2. 세 번째 자리 - 그룹 권한 (Group Permissions)
3. 네 번째 자리 - 기타 사용자 권한 (Others Permissions)  

각각의 권한은 이를 나타낸다.  

* 4: 읽기 (Read) 권한
* 2: 쓰기 (Write) 권한
* 1: 실행 (Execute) 권한  
  
예를 들어, 0744는  
특별 권한 없음  
소유자 : 읽기, 쓰기, 실행  
그룹 : 읽기  
기타 사용자 : 읽기  
  

  
close()
--------

```c
#include <unistd.h>
#include <fcntl.h>
#include <stdlib.h>

int main(void)
{
        int fd, n;
        char buf[10];
        fd = open("testfile", O_RDWR | O_CREAT | O_TRUNC, 0644); // 읽기&쓰기 권한, 파일 없으면 생성, 파일에 존재할 경우 내용 잘라내기
        if (fd < 0) { // 열기 실패할 경우
                perror("open");
                exit(1);
        }
        system("ls testfile"); // testfile 찾기 -> 존재
        unlink("testfile"); // testfile 삭제
        system("ls testfile"); // testfile 찾기 -> 존재하지 않음

        write(fd, "hello\n", 6); // 입력한 문자열 6개를 fd에 쓰기
        lseek(fd, 0, SEEK_SET); // 파일을 읽기 위해 오프셋을 파일의 시작 위치로 이동
        n = read(fd, buf, 10); // fd가 가리키는 파일에서 최대 10바이트를 읽어와서 buf에 저장
        write(STDOUT_FILENO, buf, n); // 표준 출력에 buf를 쓰기

        close(fd); // fd가 가리키는 파일 닫기
}
```  
  
#### unlink를 해도 fd가 유지되는 이유  
unlink를 하면 파일의 이름은 삭제되지만, 파일 디스크립터는 여전이 열려있다. 따라서 fd를 이용하여 읽기$쓰기를 진행할 수 있는 것이다. 그런데 close로 파일 디스크립터를 닫으면 시스템은 fd와 연결된 파일 리소스를 해제한다.  


lseek()
-------

```c
#include <unistd.h>
#include <fcntl.h>

int main(void)
{
        int fd;

        fd = creat("holefile", 0644);
        write(fd, "hello", 5); // holefile에 문자열 5개 쓰기

        lseek(fd, 10, SEEK_CUR); // 오프셋을 현재 위치에서 10만큼 뒤로 옮기기
        write(fd, "world", 5); // holefile에서 현재 위치에 문자열 5개 쓰기

        lseek(fd, 8192, SEEK_SET); // 오프셋을 파일 시작 위치에서 8192만큼 뒤로 옮기기
        write(fd, "bye", 3); // holefile에서 현재 위치에 문자열 3개 쓰기

        close(fd); // 닫기
        return 0;
}
```

write()
------  
  
  
기본 틀은 buf에 있는 내용을 fd에 write한다.

```c
#include <unistd.h>
#include <stdlib.h>
#include <stdio.h>

#define BUFFSIZE 8192

int main(void)
{
        int n;
        char buf[BUFFSIZE];

        while ((n=read(STDIN_FILENO, buf, BUFFSIZE))>0) // 표준 입력이 있다면
                if (write(STDOUT_FILENO, buf, n)!=n) // 그 입력을 표준 출력하기
                        printf("write error\n"); 
        
        if (n<0)
                printf("read error\n");
        
        exit(0);
}
```  
---

layout: post
title: creat(), close(), lseek(), write()
date: 2023-10-01 00:00:00 +0800
categories: [CS Study, Sophomore]
tags: [unix]
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
    * 1: 
1. 두 번째 자리 - 소유자 권한 (Owner Permissions)
2. 세 번째 자리 - 그룹 권한 (Group Permissions)
3. 네 번째 자리 - 기타 사용자 권한 (Others Permissions)  

각각의 권한은 이를 나타낸다.  

* 4: 읽기 (Read) 권한
* 2: 쓰기 (Write) 권한
* 1: 실행 (Execute)  
  
예를 들어, 0744는  
소유자 : 읽기, 쓰기, 실행  
그룹 : 읽기  
기타 사용자 : 읽기  
---

layout: post
title: GIT branch 공부
date: 2023-09-18 00:00:00 +0800
categories: [CS Study, Sophomore]
tags: [git]
pin: true

---


&#128203;GIT branch
===================

미리해야할 일
----------

`git log --all --graph --oneline` 모든 branch들을 시각적으로, 한줄로 표현

`git branch` branch 목록 보여주기

branch 정리
----------

`git branch 브랜치이름` 현재 버전으로 브랜치 만들기

`git check out 브랜치이름` HEAD가 해당 브랜치로 전환하기

❕**base : 합치려고하는 두 브랜치의 공통된 조상 브랜치**

`git merge 브랜치이름` 현재 브랜치와 해당 브랜치 병합하기

가상의 프로젝트로 간단하게 branch 만들어보기
-----------------------------------

```bash
parkjinhong@bagjinhong-ui-MacBookAir git % mkdir branch-study
parkjinhong@bagjinhong-ui-MacBookAir git % cd branch-study
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git init .
Initialized empty Git repository in /Users/parkjinhong/Documents/git/branch-study/.git/
```
실습(?)을 진행할 branch-study라는 디렉토리 만들기  
  
    
```bash
parkjinhong@bagjinhong-ui-MacBookAir branch-study % nano project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git add project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git commit -m "main-version1"
[main (root-commit) f0e686c] main-version1
 1 file changed, 2 insertions(+)
 create mode 100644 project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git log
commit f0e686cb9cd71bbd2b9fa49ac2e541b76d1165bc (HEAD -> main)
Author: JiinHong <com5942@seoultech.ac.kr>
Date:   Tue Sep 19 23:34:45 2023 +0900

    main-version1
parkjinhong@bagjinhong-ui-MacBookAir branch-study % nano project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % commit -am "main-verison2"
zsh: command not found: commit
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git commit -am "main-version2"
[main dad6f68] main-version2
 1 file changed, 1 insertion(+)
parkjinhong@bagjinhong-ui-MacBookAir branch-study % nano project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git commit -am "main-version3"
[main cacd15b] main-version3
 1 file changed, 1 insertion(+)
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git log --all --graph --oneline
* cacd15b (HEAD -> main) main-version3
* dad6f68 main-version2
* f0e686c main-version1
```
main의 version1을 commit해주고 version2와 version3를 만들어주기  
  
    
```bash
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git branch naver
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git branch kakao
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git branch line
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git log --all --graph --oneline
* cacd15b (HEAD -> main, naver, line, kakao) main-version3
* dad6f68 main-version2
* f0e686c main-version1
```
main-version3에서 branch 3개 만들어주기. branch이름은 각각 naver, kakao, line.  
  
    
```bash
parkjinhong@bagjinhong-ui-MacBookAir branch-study % nano project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git commit -am "main-version4"
[main 2b20fd2] main-version4
 1 file changed, 1 insertion(+)
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git checkout naver
Switched to branch 'naver'
parkjinhong@bagjinhong-ui-MacBookAir branch-study % nano project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git commit -am "naver-version1"
[naver 7c4ae32] naver-version1
 1 file changed, 3 insertions(+)
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git checkout kakao
Switched to branch 'kakao'
parkjinhong@bagjinhong-ui-MacBookAir branch-study % nano project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git commit -am "kakao-version1"
[kakao e8e1179] kakao-version1
 1 file changed, 3 insertions(+)
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git checkout line
Switched to branch 'line'
parkjinhong@bagjinhong-ui-MacBookAir branch-study % nano project.txt
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git commit -am "line-version1"
[line 7cc4125] line-version1
 1 file changed, 3 insertions(+)
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git log --all --graph --oneline
* 7cc4125 (HEAD -> line) line-version1
| * e8e1179 (kakao) kakao-version1
|/  
| * 7c4ae32 (naver) naver-version1
|/  
| * 2b20fd2 (main) main-version4
|/  
* cacd15b main-version3
* dad6f68 main-version2
* f0e686c main-version1
```
main, naver, kakao, line 각각 새로운 버전 만들어주기
나뭇가지가 만들어졌다!  
  
    
이제 merge 시도해볼 차례  
**(참고)**
main-version4의 텍스트는
```
main
version1
version2
version3
version4
```

naver-version1의 텍스트는
```
main
version1
version2
version3

naver
version1
```
commit될 때마다 텍스트에 표시해놓는 방식으로 만들어왔다.  
  
    
```bash
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git checkout main
Switched to branch 'main'
parkjinhong@bagjinhong-ui-MacBookAir branch-study % git merge naver
Auto-merging project.txt
CONFLICT (content): Merge conflict in project.txt
Automatic merge failed; fix conflicts and then commit the result.
```
main-version4와 naver-version1을 merge해보기
실패됐다고 뜨지만, 텍스트에 들어가보면,

```
main
version1
version2
version3
<<<<<<< HEAD
version4
=======

naver
version1
>>>>>>> naver
```
'=='를 기준으로 두 branch가 어떻게 충돌되어있고, 사용자에게 어떻게 수정할지 결정하라고 친절하게 표시해준다.


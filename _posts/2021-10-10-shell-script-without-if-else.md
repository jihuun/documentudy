---
layout: post
title:  "If 없이 Shell Script 분기 처리하기"
date:   2021-10-10 21:00:00 +0800
categories: UNIX
tags: UNIX ShellScript Bash Sh POSIX
author: soopsaram
---

* content
{:toc}


```sh
if [ condition ]; then
    cmd
fi
```
나는 쉘스크립트를 처음 배울때, 일반적으로 사용되는 이 If 구문이 다소 난해하게 느껴졌다. `then` 의 존재가 생소했고, `[]`의 대괄호가 정확히 무엇을 의미하는지 그리고 세미콜론이 왜 붙는지 등. 이해하지 않고 그냥 외웠다. 그래서 한때는 `;`를 `then`뒤에 붙여야하는지 앞에 붙여아하는지 햇갈려서 매번 구글 검색을 했다. 이것들의 정확한 의미는 무엇일까?  

한편 쉘스크립트를 작성할때, If Statement를 사용하지 않고도  분기 처리하는게 가능하다. 꽤 많은 경우에 `if``else` 대신 `&&``||`를 사용한다면 훨씬 깔끔하고 명료한 분기문을 작성 할 수 있다.  이 글에서는 `if``else` 대신 `&&``||`를 사용하는 분기에 대해 알아보고, 쉘스크립트의 IF Statement의 구조에 대해 조금더 명확하게 이해해 볼 것이다.



# Exit Status란 무엇인가
우선 exit statue(혹은 exit code)에 대해 이해할 필요가 있다. UNIX운영체제 표준인 POSIX 표준에서 프로그램은 성공적으로 수행했을때 exit status 0을 리턴한다. 실패했을때는 1이상의 값을 리턴한다. 그래서 프로그램이나 쉘스크립트를 작성할때 exit status를 리턴하는것을 빼먹지 않도록 유의해야한다. 다른 프로그램과 스크립트에서 해당 프로그램이 정상적으로 수행되었는지 정확히 확인을 하기 위해서이다. 

참고로 bash에서 exit status를 만들기 위해서는 `exit` 과 `return` 유틸리티(명령어)를 이용한다. (둘다 아마도 shell built-in commnad일것이다.) 

- `return <number>`
해당 함수를 종료하고 exit status를 `<number>` 으로 만든다. 함수에서만 사용가능. **함수 종료이후** echo $? 로 `<number>` 값이 무엇이었는지 확인가능하다. `$?`에는 이전 명령의 exit code값이 저장되어있다.
- `exit <number>`
전체 스크립트를 종료하고 exit status를 `<number>`으로 만든다. **스크립트 종료 이후**, 마찬가지로 echo $? 로 `<number>` 값을 확인가능하다.


# IF없이 커맨드 분기 처리

cmd1이 정상적인 exit status를 리턴한다고 가정하고, 이전 명령의 성공/실패 여부에따라 cmd2를 수행해보자. 먼저 if/else를 사용한다면 다음과 같이 길고 다소 복잡해진다.

```sh
cmd1
if [ $? -eq 0 ]; then
    cmd2
fi
```

하지만 `&&`와 `||`를 사용하면, 이를 한줄로 처리할수 있다. 코드가 훨신 간결해진다.  

-  cmd1 성공시에만 cmd2 실행

```sh
cmd1 && cmd2
```

-  cmd1 실패시에만 cmd2 실행

```sh
cmd1 || cmd2
```

만약 cmd1이 성공했을때 cmd2를 실행하고, 실패할때 cmd3를 실행하고자 한다면 아래와 같이 한줄로 처리 가능하다.

```sh
cmd1 && cmd2 || cmd3
```

명령어에 대한 exit status 처리는 가능하면 `&&` `||`를 사용하는게 코드를 훨씬 간결하게 만든다.  


# 대괄호`[]`의 정체

`[ ` 는 `test` 와 동일한 커맨드이다. 따라서 `[` 도 하나의 shell 명령어이자 프로그램이다. 다만 `[` 는 반드시 마지막에 인자 `]` 를 전달해야한다는점이 다를 뿐이다. `test`는 전달된 인자를 평가(evaluate)하는 유틸리티이다. 이 명령의 결과는 exit status 인데 전달된 인자가 True 인경우는 0 이고 False인 경우는 1이된다. `[ ` 와 `test`는 [POSIX 표준](https://pubs.opengroup.org/onlinepubs/9699919799/utilities/test.html)이기 때문에 대부분의 쉘에서 지원한다. 

참고로 `[[` 는 bash shell 에서 사용하는 `[`의 개선된 버전이다. bash가 아닌 shell에서 정상적으로 동작한다는 보장은 없지만, ksh, bash, zsh, yash, busybox sh에서는 지원한다고 한다.



```sh
if [ "aa" == "aa" ]
then 
    echo 'yes'
fi
```

```sh
if test "aa" == "aa"
then 
    echo 'yes'
fi
```
따라서 위 두 구문은 동일한 동작을 한다.




# IF 없이 조건 분기 처리

위에서 대괄호도 결국 test 커맨드임을 알게 되었다. 따라서 명시적인 If Statement없이도 분기처리가 가능하다. 기존의 If 조건에 따른 분기 대신에 &&를 사용하여 작성하면 다음과 같이 할 수 있다.

- condition이 true일때만 cmd 수행

```sh
[ condition ] && cmd
```

- condition이 false 일때만 cmd가 수행.

```sh
[ condition ] || cmd
```

- condition에 따라 if/else 처리

```sh
[ condition ] && cmd1 || cmd2
```
condition이 true이면 cmd1 false이면 cmd2가 수행된다. 아래와 동일한 동작이다. 


```sh
if [ condition ]
then
    cmd1
else
    cmd2
fi
```



한가지 주의할 점은 shell에서는 무조건 왼쪽에서 오른쪽으로 순차적으로 명령과 연산자를 수행한다는 점이다. c언어처엄 논리 연산자 우선순위에 따라 명령을 처리하지 않는다. 가령 `[ condition ] && cmd1 && cmd2 || cmd3` 은 먼저 `[ condition ]` 명령이 성공하면 `cmd1`명령을 실행하고 `cmd1`이 성공하면 `cmd2`가 실행된다. 그리고 만약 `cmd1`이 실패하면 `cmd2`는 실행되지 않고 `cmd3`이 실행된다.

```sh
#! /bin/bash

[ condition ] && cmd1
echo "hello"
```

또 주의할 사항이 있다. 만약 위와 같이 스크립트를 작성한경우 condition이 False이면 exit 1이 리턴된다. 그 뜻은 `echo "hello"` 라인이 수행되기도 전에 이 스크립트 자체가 비정상 종료된다는 뜻이다. 만약 If Statement를 사용했다면 "hello"는 출력되지 않았겠지만 전체스크립트가 비정상 종료되지는 않았을것이다. 따라서 `if/else`대신 `&&` `||`를 사용한다면 exit status에 대한 처리에 주의해야할 것이다.

그렇다면 위에서 condition이 False인경우 아무것도 하지 않고 다음 라인 `echo "hello"` 을 실행하기 위해서는 어떻게 하면 될까? 두가지 해결 방안이 있다. 

먼저 `set -e` 처리를 하면 exit 1이 발생하지 않는다. 참고로 set -e 는 POSIX 표준이다. 

```sh
#! /bin/bash

set -e
[ condition ] && cmd1
echo "hello"
```

두번째로는 `|| true` 로 false 인 경우도 추가하여 아무동작도 수행하지 않는것이다. 

```sh
#! /bin/bash

[ condition ] && cmd1 || true
echo "hello"
```

이렇게 하면 condition이 false더라도 해당 스크립트가 비정상 종료 되지 않고 `echo "hello"`가 출력된다.



# If Statement의 작동원리

```sh
if
   cmd1
then
   cmd2
else
   cmd3
fi
```

이제 쉘스크립트의 If Statement 를 정확히 알아보자. 중요한점은 If 바로 다음에는 명령어가 온다는것이다. 여기에는 보통 `[]`로 표현되는 `test`명령을 사용하지만 이전에 설명했드시 일반 명령어를 사용할수도 있다.  그렇다면 `then`이 하는 역할은 무엇인가? `then`이 바로 이전명령의 exit status를 포착하는 역할이다. 이전 exit status가 0 인경우에 `then` state의 구문이 수행되는 것이다. 따라서 위의 예에서 cmd1 이 exit code 0을 리턴하면 cmd2 가 수행되고 1 이상의 값을 리턴하면 cmd3이 수행된다. 참고로 우리가 이미 알고있듯이 많은 쉘스크립트가 `if cmd1; then` 과 같이 한줄로 표기한다. `;`는 명령어 뒤에 붙으며, 다음명령까지 한줄에서 처리하게 만드는 역할을 한다. 



# References

- [https://www.gnu.org/software/bash/manual/html_node/Conditional-Constructs.html](https://www.gnu.org/software/bash/manual/html_node/Conditional-Constructs.html)  
- [https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Lists](https://www.gnu.org/savannah-checkouts/gnu/bash/manual/bash.html#Lists)  
- [https://stackoverflow.com/questions/4419952/difference-between-return-and-exit-in-bash-functions](https://stackoverflow.com/questions/4419952/difference-between-return-and-exit-in-bash-functions)  
- [https://unix.stackexchange.com/questions/609092/what-is-the-difference-between-square-brackets-and-test-command-on-bash](https://unix.stackexchange.com/questions/609092/what-is-the-difference-between-square-brackets-and-test-command-on-bash)  
- [YouTube: Never say "If" writing a Bash script!](https://youtu.be/p0KKBmfiVl0)  
- [https://unix.stackexchange.com/questions/306111/what-is-the-difference-between-the-bash-operators-vs-vs-vs](https://unix.stackexchange.com/questions/306111/what-is-the-difference-between-the-bash-operators-vs-vs-vs)  


---
layout: post
title:  "Bash Auto Completion"
date:   2021-09-02 21:00:00 +0800
categories: Shell
tags: UNIX Bash Shell
author: soopsaram
---

* content
{:toc}

## 요약
bash 쉘에서 본인이 작성한 스크립트나 프로그램 명령에 argument를 입력할때 TAB으로 자동완성 시켜주는 기능 만들어보자

## 목표 
- cdd 전달인자인 경로 자동완성
cdd는 `cd ../../../../` 를 입력하는 대신 `cdd 4`로 손쉽게 뒤로이동 하는 내가 만든 명령도구다.
[cdd 바로가기](https://github.com/scriptworld/cdd)
- `cdd 3` + `TAB` ->  ../../../ 경로의 디렉터리 이름 출력 및 prefix로 디렉터리 자동완성
- ~~`cd -` 원래 디렉터리로 복귀~~

## 기초
bash completion 을 만들때, 기본적으로 이 두가지 외부 툴이 함께 사용된다. `complete` `compgen`
그리고 내부 변수 `COMPREPLY`, `COMP_CWORD`,  `COMP_WORDS` 를 활용한다. 
이들을 잘 조합한 bash스크립트로 자동완성 기능을 만들 수 있다.
> 강추 문서 https://mug896.github.io/bash-shell/command_completion.html


## complete 명령 사용법

`complete -F <함수> <명령>` : 쉘에서<명령> 작성 후 한칸 띄고 `TAB`치면 <함수> 호출됨. 엔터가 아니라 탭을 쳤을때 사용되는 추가 명령(?) 이라고 보면 될것같음.  

- cdd/completion/bash_cdd 생성

```sh
#!/bin/bash

_cdd()
{
    echo 'auto completion test'
}
complete -F _cdd cdd 
```


- 심볼릭링크 생성 (MacOS: /usr/local/etc/bash_completion.d)

```sh
 $ ln -s ~/project/cdd/completions/bash_cdd /usr/local/etc/bash_completion.d/cdd
 $ source /usr/local/etc/bash_completion.d/cdd
```
- 그 뒤 `cdd 3` + `TAB`하면 작성한 _cdd 함수가 동작하는것을 알 수 있음. 따라서 앞으로 _cdd() 함수에 원하는 동작을 작성하면 됨.  

```
$ cdd 3 auto completion test

```
> 그 뒤 엔터를 치면 실제 cdd명령 수행됨 `go to ./../../../ `

- complete `-o dirnames` 옵션

TAB 하면 디렉터리 이름을 자동완성 한다. 
```
$ complete -o dirnames hello 
$ hello [TAB]
aa/ bb/ cc/
```

- `-o nospace` 옵션
이름을 자동완성하고 나면 다음 이름을 위해 공백을 띄우게 되는데, 이 옵션은 그걸 방지.

```
$ complete -o nospace -W 'aaa= bbb= ccc='  hello

# 이름을 완성하고 나서 공백을 띄우지 않는다.
$ hello aaa=[stop]
```

## COMP_CWORD 와 COMP_WORDS로  인자 다루기
`int main(int argc, char *argv[])` 의 인자와 동일한 역할  
> `$COMP_CWORD` -> `int argc`  argument 갯수
> `$COMP_WORDS` -> `char *argv[]`  모든 argument 문자열 리스트


- `TAB` 을 칠때, cdd명령으로 전달되는 인자 확인 

```sh
#!/bin/bash

_cdd()
{
    echo ''
    echo 'argument count : '$COMP_CWORD
    echo 'argument words0 : '${COMP_WORDS[0]}
    echo 'argument words1 : '${COMP_WORDS[1]}
    echo 'argument words2 : '${COMP_WORDS[2]}
}
complete -F _cdd cdd 
```

```sh
$ cdd 3 
argument count : 2
argument words0 : cdd
argument words1 : 3
argument words2 : 
```
> `cdd 3 ` space를 치고 `TAB`을 친 경우임. 만약 `cdd 3` 뒤에 space를 치지 않고 `TAB`을 치면 argument count : 1 이 된다. 

## COMPREPLY 전역변수 사용하기
`TAB`을 쳤을때 COMPREPLY에 저장된 문자열로 대체된다. 우리가 최종에 값을 저장해야할 변수. bash_completion의 내장 변수.  

```sh
#!/bin/bash

_cdd()
{
    COMPREPLY='aabbcc'
}
complete -F _cdd cdd 
```
> 그 뒤 cdd + `TAB`을 하면 aabbcc가 자동으로 대체된다. 
> 아무문자를 입력하고 `TAB`해도 바뀐다. 가령 $ cdd ef`TAB` 하면 -> $ cdd aabbcc 로 바뀜

따라서 cdd 뒤에 오는 인자에 따라 COMPREPLY 값을 바꿔주면 될듯! 

가령 cdd 3 하고 (스페이스를 치지 않고 바로) `TAB`을 치면, 아직 인자를  전달한것이 아니기 때문에 `$COMP_CWORD` 이 `2`가 아니라 `1`이다. 하지만 작성중인 인자 3은 `${COMP_WORDS[1]}` 에 저장되어 있다. 
> argument count : 1
argument words0 : cdd
argument words1 : 3

이를 이용하면 ${COMP_WORDS[1]가 `3`인 경우에, COMPREPLY에 "../../../"을 저장하면, `$ cdd ../../../` 로 대체되게 만들수 있다! 그리고 내부적으로 ../../../ 경로의 실제 디렉터리를 자동완성으로 출력하거나 변환하면 된다. (하지만 현재 cdd 로는 이 기능 수행못함, cdd명령 추가구현 필요)
> cdd 내부 동작 수정해야할 내용
> 1. cdd 인자로 `숫자`들어오는 경우 -> 기존 cdd내용 수행
> 2. cdd 인자로 `경로`들어오는 경우 -> 일반 `cd 경로` 수행
> 참고: 숫자 문자 구분 방법: https://stackoverflow.com/questions/806906/how-do-i-test-if-a-variable-is-a-number-in-bash/806923
> 숫자/문자로 구분하는게 아니라 전달받은 인자가 exist한 directory인지 아닌지로 구분이 더 나을듯?






## compgen 사용하기

```sh
compgen -W '단어1 단어2 ... 단어N' -- "<Prefix>"
```
> 단어 중에서 <Prefix>가 포함 되어있는 단어만 출력한다.

```sh
$ compgen -W 'soopsaram soopnorm great man' -- 'soop'
soopsaram
soopnorm

```

```sh
$ ls
complate/   test/   aa   bb
$ compgen -W '$(find . -mindepth 1 -maxdepth 1 -type d)' -- "./co"
complate/ 
```
> compgen을 find와 조합해서 디렉터리에 존재하는 경로명을 추출할 수 있다.



---

## 참고
- https://z-wony.tistory.com/3 [끄적끄적 프로그래밍]
- https://www.tuwlab.com/ece/29643
- https://www.gnu.org/software/bash/manual/html_node/Programmable-Completion-Builtins.html
- complete 명령 안내(KOR)
  - https://mug896.github.io/bash-shell/command_completion.html

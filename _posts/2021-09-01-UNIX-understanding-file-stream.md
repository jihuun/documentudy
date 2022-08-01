---
layout: post
title:  "All That Stream"
date:   2021-08-31 21:00:00 +0800
categories: UNIX
tags: UNIX C Shell
author: soopsaram
---

* content
{:toc}

# 1. What is Stream

UNIX에서는 모든것이 file이다. Stream을 사용하는것은 UNIX 응용프로그램에서 파일을 읽고 쓰기 위한 일반적인 방법이기 때문에 UNIX 에서 Stream에 대한 이해하는것은 중요하다. 

## 1.1. Stream 과 File

우선 file과 stream의 차이에 대해 알아보자. file은 바이트의 연속적인 모음이다(sequences of bytes). stream은 이 bytes를 입출력하기 위한 인터페이스이자 도구이다. stream은 file뿐만 아니라 socket, keyboard, USB port, printer등 sequential data 로 이루어진 데이터를 다루기 위한 common 인터페이스인데, 이 하나의 인터페이스로 모든 소스를 다룰 수 있다. 모든 소스를 단일한 인터페이스로 다룰 수 있기 때문에 동일한 코드로 위에 나열된 장치들을 모두 file로 여기고 읽고 쓸수 있게 된다. 다시말하지만 UNIX에서는 모든것이 file이다. 

stream은 모든 입력 출력을 바이트의 흐름으로 생각하는것이다. 입출력 장치, 하드웨어, 단말에 상관없이 프로그램을 작성할수 있는 독립성을 갖는다. 이를 장치독립성이라고 한다.  그리고 입출력장치와 CPU의 속도차이를 보완하기 위해서 버퍼를 갖는다. 

아직 Stream의 정체가 무엇인지 매우 추상적이고 모호하다. 그렇다면 물리적으로 Stream은 어떻게 표현될까?  C언어에서 Standard Library 함수인 `fopen()` 을 사용해 file을 열어보면, 해당 함수는 `FILE *` 이라는 `struct FILE`의 포인터 타입을 리턴한다. 이것이 Stream이다(file stream). `FILE *` 포인터는 실제 파일을 가리키는 포인터가 아니다. 파일의 스트림을 가리키는 포인터이다. 그리고 그 스트림에는 파일에 대한 여러가지 정보가 담겨있다. 

stream에는 standard stream이 있는데 이것도 결국 file 에 대한 stream이다.  모두 file descriptor 번호를 가지고 있다. standard stream은 프로그램이 실행되면 OS에서 자동으로 생성한다 (stdin/stdout/stderr). stdin 은 FILE * 타입으로 stdio.h 에 정의되어있다. 자세한 내용은 챕터 2에서 다시 설명한다.

vi나 nano같은 에디터로 파일을 열면 에디터 프로그램은 해당 파일의 스트림을 생성하고 읽고 쓴다. 


- references
> [stackoverflow: what-does-stream-mean-in-c](https://stackoverflow.com/questions/38652953/what-does-stream-mean-in-c)
> [youtube: 스트림 (널널한 교수의 기초 C언어) ft. C 코딩](https://youtu.be/iuEdJ9wg8wU)

# 2. Standard Stream (표준 스트림)

## 2.1. Standard Streams: `stdin` , `stdout`, `stderr`  
프로그래머가 생성하지 않아도 자동으로 생성되는 스트림. 모든 Process에 각각 생성된다. 프로그램이 실행되면 운영체제는 실행되는 프로그램(프로세스)에게 3개의 기본 file descripter 를 할당해준다. 번호는 `stdin`:0 , `stdout`:1 , `stderr`:2. 그리고 만약 프로그램이 내부에서 다른 파일을 open() 하게 되면 운영체제는 해당 프로세스에게 3번 file descripter를 할당한다.   `stdin` , `stdout`, `stderr` 모두 FILE * 타입으로 stdio.h 에 정의되어있다. 

- the OS gives the address to the standard stream made for a process to it automatically. 
- Standard stream 은 Unix C runtime environment 에서 환경을 제공한다.
Since Unix provided standard streams, the __Unix C runtime environment__ was obliged to support it as well. As a result, most C runtime environments (and C's descendants), regardless of the operating system, provide equivalent functionality.

- references
> [https://stackoverflow.com/questions/38652953/what-does-stream-mean-in-c](https://stackoverflow.com/questions/38652953/what-does-stream-mean-in-c)
> [https://blogger.pe.kr/369](https://blogger.pe.kr/369)

## 2.2. Shell I/O Redirect 와 PIPE 의 차이

일반적으로 UNIX shell은 stdout으로 전달된 것을 console로 출력해준다. 그런데 shell 에서 `>` 를 사용하면 stdout을 console이 아닌 파일에 출력할 수 있다(stdout to the file). 가령 아래와 같이 ` echo "a b c d"`의 stdout 이 console로 출력되지 않고 tmp.txt 파일로 출력된다. 즉 tmp.txt 파일에 써진다. 


```sh
 $ echo "a b c d" > tmp.txt
 $ cat tmp.txt 
a b c d
```
- `1>` 기본적으로 `>` 와 동일 fd 1번인 stdout 을 stdout으로 redirect. 문법은 file_descriptor `>` file_name
- `2>` fd 2번인 2번 stream. 즉 stderr를 stdout으로 redirect.
- `&>` `&>name` 은 `1>name 2>name` 와 같다. 하지만 name을 두번 open하기 때문에 좋지 않다.
- `2>&1` 아래 에서 따로 설명한다. 결론적으로 stderr를 stdout으로 지정하는의미.

반면 pipe `|` 는 stdout을 다른 프로그램의 stdin으로 사용할 수 있다. pipe는 
아래와 같이 실행하면 ` cat tmp.txt` 의 stdout 출력결과가 `wc -w` 의 stdin으로 입력된다. tmp.txt에 a b c d 즉 4개의 word가 있기때문에 `wc -w`하면 4가 출력된다.
 
```sh
 $ cat tmp.txt | wc -w
4
```

한편 `<` 를 사용하면 keyboard가 아닌 < 다음 입력된 파일로 부터 stdin 입력된다.  순서는 딱히 상관 없음.

```sh
 $ wc -w < tmp.txt
4
 $  < tmp.txt wc -w
4
```

|  | 요약 |
| --- | --- | 
| `>` file | stdout을 console 이 아닌 file에 출력  |
| `<` file | keyboard가 아닌 file로 부터 stdin 입력  |
| cmd1 `|` cmd2 |  한 프로그램(cmd1)의 stdout출력을 다음 프로그램(cmd2)의 stdin 으로 사용  |

- references
> [https://thoughtbot.com/blog/input-output-redirection-in-the-shell](https://thoughtbot.com/blog/input-output-redirection-in-the-shell)
> [https://github.com/kennyyu/bootcamp-unix/wiki/stdin,-stdout,-stderr,-and-pipes](https://github.com/kennyyu/bootcamp-unix/wiki/stdin,-stdout,-stderr,-and-pipes)


## 2.3.  `2>&1` 이해하기
Shell 에서 error 메시지를 없앨때 빈번히 사용하는 표현이다. `2>` 는 stderr을 stdout으로 redirect. `&1` 는 이전에 지정한 stdout이 출력하는 파일을 의미한다고 생각하면 쉽다. 엄밀히 말하면 `>`연산자 대신에 `>&` 를 사용한것이다.  `2>&1` 이전에 fd번호 1을 다른 파일로 재지정 했다면, fd번호 2는 재지정한 stdout으로  redirect 한다는 의미. 

예를들어, `cat asdf.txt` 이 아래와 같이 에러메시지를 출력한다고 해보자.

```
$ cat asdf.txt
cat: asdf.txt: No such file or directory
```

 `cat asdf.txt > /dev/null 2>&1` 을 수행하면 먼저 stdout은 /dev/null 로 redirect되고, 그 뒤 `2>&1`을 통해 stderr는 `&1`로 redirect된다. 즉 stderr도 /dev/null로 redirect되는것이다. 따라서 console에는 아무것도 출력되지 않는다. 

그런데 만약 `cat asdf.txt 2>&1`을 하면 어떻게 될까? stdout 을 따로 지정하지 않았기 때문에 &1 는 default인 console이다. 따라서 2> 즉 stderr 는 그대로 console에 출력된다. 

관련해서 구체적인 문법은 아래와 같다. 두번째 항목을 보면된다. `>&` 연산자의 우측에는 file descriptor 번호가 오는것. 

- `>`
file_descriptor `>` file_name
- `>&` 
file_descriptor `>&` file_descriptor
- `&>`
`&>` file_name

# 3. Stream with C language

## fopen(), open() 의 차이
가장 큰 차이는 `fopen()` 은 Standard Library이고, `open()`은  UNIX System Call이라는 점이다.
 fopen은 내부적으로 open을 사용한다. fopen은 버퍼를 사용하기 때문에 빈번한 system call 을 방지해준다. 잦은 system call은 성능 저하를 발생시킬 수 있다.

- struct FILE 구조체 

```c
c:linenumbers,wraptext
typedef strut _iobuf {
        int cnt;        /* characters left */
        char *ptr;      /* next character position */
        char *base;     /* location of buffer */
        int flag;       /* mode of file access */
        int fd;         /* file descriptor */
} FILE;
```
> from K&R (C Programming Language,  ANSI C, p.176)


## File Pointer vs File Descripter
file pointer는 파일 스트림 포인터 즉 `FILE *`를 의미한다. file descripter는 OS레벨에서 해당 파일스트림을 구분하기 위한 번호이다. OS가 생성한다.  

## C 프로그램에서 외부 입출력

stdin/stdout으로 입출력하거나, argument로 입력받는 방법들 비교.

- fopen()
c프로그램 내에서 파일로부터 파일 스트림 생성.  

###  File Stream에 쓰기 (파일)

fopen을 통해 file.txt 파일의 파일 스트림 포인터 fw를 얻고, fprintf를 통해 write.  

```c
        FILE *fp = fopen("file.txt", "w");
        fprintf(fp, "%s %d\n", "hello", 23);
        fclose(fp);
```

컴파일하고 실행하면, file.txt파일내에 값이 쓰여있다.  

```sh
$ cat file.txt
hello 23
```

### File Stream에 쓰기 (STDOUT)

반면 아래와 같이 stdout에 write하면  

```c
        FILE *fp = stdout;
        fprintf(fp, "%s %d\n", "hello stdin", 100);
```

혹은

```c
        fprintf(stdout, "%s %d\n", "hello stdin", 100);
```

(사실 `printf`는 `fprintf(stdout` 과 동일하다. )  

컴파일 후 실행시 stdout, 즉 콘솔화면에 출력된다. 참고로 stdout은 stdio.h에 정의되어있으며, stdout 스트림은 fclose()로 종료하면 안된다.  

```sh
$ ./run
hello stdin 100
```

### File Stream 으로부터 입력받기 (STDIN)
```c
        char str[10];
        int num;

        FILE *fp = stdin;
        fscanf(fp, "%s %d", str, &num);
        printf("> %s %d\n", str, num);
```

- 1. pipe 통해서
```
 $ echo "test_string 99" | ./run
> test_string 99
```

- 2. redirect 를 통해 파일로부터
```
 $ cat file.txt
hello 23
 $ ./run < file.txt
> hello 23
```

### Argument 로부터 입력받기

```c
int main(int argc, char *argv[])
{
        for (int i = 0; i < argc; i++)
                printf("argument [%d]: %s\n", i, argv[i]);
....
```
컴파일후 생성된 executable파일 run을 실행하면  

```
$ ./run --option hello world
argument [0]: ./run
argument [1]: --option
argument [2]: hello
argument [3]: world
```

---
layout: post
title:  "Publishing my project to Homebrew"
date:   2021-09-05 21:00:00 +0800
categories: UNIX
tags: UNIX Shell Python
author: soopsaram
---

* content
{:toc}

# 1. 개요
UNIX 계열 운영체제는 각각 여러가지 Package Manager가 존재한다. 우분투 같은 Debian 계열의 리눅스 배포판은 [APT-based](https://www.debian.org/doc/manuals/debian-reference/ch02.en.html) package management 도구를 제공한다. Red Hat계열의 리눅스 배포판에서는 [RPM](https://rpm.org/)이라는 Package Manager를 제공한다. Package Manager 종류에 대한 자세한 내용은 [위키피디아 문서](https://en.wikipedia.org/wiki/List_of_software_package_management_systems)를 참고. MacOS 에서는 Homebrew라는Package Manager를 많이 사용하는데. Homebrew에 내가만든 소프트웨어를 Publish하여 사람들이 쉽게 내 소프트웨어를 다운받고 설치할수 있게 만들어보자. 내가 조사한 바로는 아래와 같이 두 가지 방법이 있다.   

1. **Homebrew-core 에 등록**  
`brew install <my-project>` 로 설치가능하게 되는 공식적인 방법. [homebrew-core github repo](https://github.com/Homebrew/homebrew-core)에 [Pull Request](https://github.com/Homebrew/homebrew-core/pulls)를 보내면 되는듯 하다. 자세한 내용은 다음의 Documentation를 참고하면 된다. [Adding-Software-to-Homebrew](https://docs.brew.sh/Adding-Software-to-Homebrew)

2. **Brew tap 으로 설치**  
homebrew 공식 릴리즈에 포함되지 않더라도 등록할 수 있는 방법이다. 결과는 brew tap 으로 repo를 추가할수 있게된다. 참고로 Tap 은 homebrew에서 제공하는 third-party 저장소를 등록하는 방법이다. 등록 이후에 아래와 같은 방법으로 해당 소프트웨어를 설치 할 수 있다. 참고 문서: [https://federicoterzi.com/blog/how-to-publish-your-rust-project-on-homebrew/](https://federicoterzi.com/blog/how-to-publish-your-rust-project-on-homebrew/)
    ```sh
     $ brew tap jihuun/rsdic
     $ brew install rsdic
    ```

이 문서에서는 2번 방법에대해 알아볼 것이다. 예제로 사용한 Publish 대상은 내가 만든 터미널용 영어사전인 [rsdic](https://github.com/jihuun/rsdic)이다. 이 프로젝트는 Rust로 작성되었으며 이 문서에는 Rust 관련 내용이 일부 포함될 수 있다.  

# 2. Homebrew에 프로젝트 등록하기

## 2.1.  내 프로젝트 바이너리 생성 및 release

- **`rsdic.tar.gz` 생성**  
조금 뒤에 살펴볼 homebrew `Formula`에서 *.tar.gz 로 압축된 바이너리를 필요로한다. 바이너리를 빌드한뒤에 *.tar.gz로 압축 하면된다. Rust에서는 `cargo build --release` 하면 쉽게 릴리즈 바이너리를 빌드할 수 있다.  

	```sh
	cargo build --release
	cd target/release
	tar -zcvf rsdic.tar.gz rsdic
	```

- **sha256 해시 생성**   
생성한 압축파일에 대한 해시값도 추후 homebrew `Formula`에 추가해야 한다. 빠르게 검색하기 위한 용도인듯 하다.  

	```sh
	$ shasum -a 256 rsdic.tar.gz
	f65f3265654fd662fd37334122cd2f97f3b85de12969dae81d0484f30cebeceb  rsdic.tar.gz
	```

- **바이너리 링크 생성**  
Github Release에 <my-project>.tar.gz  바이너리 업로드 하고 아래와 같은 링크를 생성한다.
https://github.com/jihuun/rsdic/releases/download/v0.1.0/rsdic.tar.gz


## 2.2. `homebrew-rsdic` Github repo 생성


- **`homebrew-rsdic` Github repo 생성**   
Github repository를 하나 생성한다. 저장소 이름은 homebrew-<projectname> 와 같은 방식으로 해야한다. 이유는 [Homebrew 네이밍 컨벤션](https://docs.brew.sh/Taps#repository-naming-conventions-and-assumptions) 참고.

- **`Formula` 작성**   
Formula는 Ruby 파일인데 딱히 Ruby지식을 필요로 하지는 않는다(나도 모름). 생성한 repository는 아래와 같은 디렉터리 구조를 필요로 한다.  

	```
	- Formula/
		- <projectname>.rb
	- README.md
	```

- **Formula/rsdic.rb 생성**  
Formula/rsdic.rb 파일을 생성하고 아래의 내용을 추가한다.  

	```rb
	# Documentation: https://docs.brew.sh/Formula-Cookbook
	#                https://rubydoc.brew.sh/Formula
	# PLEASE REMOVE ALL GENERATED COMMENTS BEFORE SUBMITTING YOUR PULL REQUEST!
	class Rsdic < Formula
	  desc "A eng-kor dictionary for the terminal users"
	  homepage "https://github.com/jihuun/rsdic"
	  url "https://github.com/jihuun/rsdic/releases/download/v0.1.0/rsdic.tar.gz"
	  sha256 "f65f3265654fd662fd37334122cd2f97f3b85de12969dae81d0484f30cebeceb"
	  version "0.1.0"

	  def install
	    bin.install "rsdic"
	  end
	end
	```

- **commit  생성하고 push**  
여기까지 하면 Homebrew를 통해 내 프로젝트를 다운로드 받을 수 있다. 실제 커밋 참고. [https://github.com/jihuun/homebrew-rsdic/commit/a02f6db3f50a5111ade52e2f31f3c963f97db844](https://github.com/jihuun/homebrew-rsdic/commit/a02f6db3f50a5111ade52e2f31f3c963f97db844)

# 3. 패키지 설치해보기

- **`brew tap` 으로 third party repo등록**

	```sh
	 $ brew tap jihuun/rsdic

	==> Tapping jihuun/rsdic
	Cloning into '/usr/local/Homebrew/Library/Taps/jihuun/homebrew-rsdic'...
	remote: Enumerating objects: 5, done.
	remote: Counting objects: 100% (5/5), done.
	remote: Compressing objects: 100% (4/4), done.
	remote: Total 5 (delta 0), reused 5 (delta 0), pack-reused 0
	Unpacking objects: 100% (5/5), done.
	Tapped 1 formula (29 files, 24.2KB).
	```

- **Package install**

	```sh
	 $ brew install rsdic

	==> Installing rsdic from jihuun/rsdic
	==> Downloading https://github.com/jihuun/rsdic/releases/download/v0.1.0/rsdic.tar.gz
	==> Downloading from https://github-releases.githubusercontent.com/364442112/3c316f00-adc4-11eb-88cd-b249eab44c28?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Crede
######################################################################## 100.0%

	```


# 4. 추후 릴리즈 자동화
이제 새 버전 릴리즈시 homebrew-rsdic repo에서 rsdic.rb 파일에 내용을 업데이트 하여 반영하면 된다. Travis 같은 CI/CD 툴로 버전 릴리즈후 brew에 바이너리 업데이트 과정을 자동화 하면 좋을 것 같다.  



# Docker 사용하기

- [Docker 사용하기](#docker-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
  - [Minimum Viable Usage ( 최소 사용법 )](#minimum-viable-usage--%EC%B5%9C%EC%86%8C-%EC%82%AC%EC%9A%A9%EB%B2%95)
  - [docker 설치하기 (linux 버젼)](#docker-%EC%84%A4%EC%B9%98%ED%95%98%EA%B8%B0-linux-%EB%B2%84%EC%A0%BC)
  - [도커 사용하기](#%EB%8F%84%EC%BB%A4-%EC%82%AC%EC%9A%A9%ED%95%98%EA%B8%B0)
    - [내 어플리케이션을 도커를 써서 실행하기](#%EB%82%B4-%EC%96%B4%ED%94%8C%EB%A6%AC%EC%BC%80%EC%9D%B4%EC%85%98%EC%9D%84-%EB%8F%84%EC%BB%A4%EB%A5%BC-%EC%8D%A8%EC%84%9C-%EC%8B%A4%ED%96%89%ED%95%98%EA%B8%B0)
    - [내 컴퓨터에 mysql 설치 안하고 도커로 써보기](#%EB%82%B4-%EC%BB%B4%ED%93%A8%ED%84%B0%EC%97%90-mysql-%EC%84%A4%EC%B9%98-%EC%95%88%ED%95%98%EA%B3%A0-%EB%8F%84%EC%BB%A4%EB%A1%9C-%EC%8D%A8%EB%B3%B4%EA%B8%B0)
    - [내 컴퓨터에 레디스 설치안하고 도커로 써보기](#%EB%82%B4-%EC%BB%B4%ED%93%A8%ED%84%B0%EC%97%90-%EB%A0%88%EB%94%94%EC%8A%A4-%EC%84%A4%EC%B9%98%EC%95%88%ED%95%98%EA%B3%A0-%EB%8F%84%EC%BB%A4%EB%A1%9C-%EC%8D%A8%EB%B3%B4%EA%B8%B0)
    - [도커로 지킬 블로그 생성기 써보기](#%EB%8F%84%EC%BB%A4%EB%A1%9C-%EC%A7%80%ED%82%AC-%EB%B8%94%EB%A1%9C%EA%B7%B8-%EC%83%9D%EC%84%B1%EA%B8%B0-%EC%8D%A8%EB%B3%B4%EA%B8%B0)
  - [정리](#%EC%A0%95%EB%A6%AC)

## Minimum Viable Usage ( 최소 사용법 )

제품을 만들때 MVP가 있는것처럼 툴도 복잡하게 사용하기 이전에 최소한의 설정으로 우선 실행 시켜볼수 있는것이 필요하다.
도커는 여러가지 자유로운 사용법 때문에 처음 접하게 되는 사용자는 오히려 혼란스럽기만 할수 있는데 제일 간단한 버젼의
설정 방법만 알고 일단 실행해 본다면 오히려 더 빨리 이해할수 있다. 아래 기술해 놓은 명령어를 따라하기만 하면 도커는 이미 실행되고 있다.

## docker 설치하기 (linux 버젼)

설치는 모든 것이 그렇듯이 APT로 한다.

[도커설치하기](https://docs.docker.com/install/linux/docker-ce/ubuntu/) 다른 OS도 해당링크에 가면 모두 있다.

1. 기본설정

```sh
$ sudo apt-get remove docker docker-engine docker.io containerd runc  # 기존 설치되어있을수 있는 올드버젼을 선삭제
$ sudo apt-get update                                                 # apt 패키지 인덱스 업데이트. 패키지 다운받기전 항상 수행하는 명령어이므로 도커와 직접적인 상관없음
$ sudo apt-get install \                                              # apt 가 https 연결을 사용할수 있도록 관련 유틸 설치
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -  # 도커 오피셜 GPG 키 설치
$ sudo add-apt-repository \                                                     # 도커 리파지토리 등록 이곳에 최신버젼들이 있음
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
```

2. 도커설치

```sh
$ sudo apt-get update
$ sudo apt-get install docker-ce docker-ce-cli containerd.io
```

3. 도커 설치 확인

```sh
$ sudo docker run hello-world
```

## 도커 사용하기

### 내 어플리케이션을 도커를 써서 실행하기

1. 내가 갖고 있는 노드 어플리케이션 최상단 폴더에 Dockerfile 한개 생성

```Dockerfile
FROM node:9.8-alpine                  # 사용할 기본 이미지, 노드 기본설치되어있음 ( 타 프레임웍도 찾아보면 해당 이미지 있음)
COPY . /var/lib/my-service/           # 내 소스를 복사함 ( 도커 컨테이너 안에서 사용할 폴더에 내 소스를 복사 폴더는 원하는 대로 설정하면 됨)
WORKDIR /var/lib/my-service           # 작업 폴더 설정 ( 위에서 지정한 폴더를 워크디렉토리로 설정 )
EXPOSE 3000                           # 외부에 오픈할 포트 (노드 어플리케이션에서 쓰는 포트임. 타 프레임웍은 원하는 포트 오픈)
RUN npm install                       # 노드 관련 ( 타 프레임웍은 최초 설치 해당 명령어 수행하면 됨 )
ENTRYPOINT npm start                  # 노드 어플리케이션 관련 명령어 ( 어플리케이션 기동 명령어 )
```

2. 이미지 생성

```sh
docker build -t any-image-name-you-want .     # 위의 Dockerfile을 참조하여 내 어플리케이션을 포함하고 있는 이미지를 만듬.
```

3. 실행

```sh
docker run -p 3000:3000 image-name-you-created       # 전단계에서 만들어진 이미지를 실행! 끝
```

벌써 도커는 실행중이다. 한번 만들어진 이미지는 계속적으로 쓸수 있기 때문에 step 3의 명령어만 반복적으로 수행하면 된다.

### 내 컴퓨터에 mysql 설치 안하고 도커로 써보기

1,2번의 단계를 누군가가 나대신 이미 수행하여 만들어진 이미지를 업로드 해놨다. 나는 가져다가 쓰기만 하면 된다. 기본적으로 step 3만 실행하면 되는것인데, 다만
1,2번 대신 다운로드 명령 필요하다.

1. 다운로드 (전 단계에서 내가 직접 이미지를 만들었지만. 여기서는 이미 만들어진 이미지를 다운로드한다. 결과적으로 1,2번을 대체)

```sh
docker pull mysql
```

2. 실행

```sh
docker run -it mysql
```

이제 끝. mySql을 이제 사용할수 있다. workbench 를 실행해서 mysql에 접속이 가능하다.

### 내 컴퓨터에 레디스 설치안하고 도커로 써보기

### 도커로 지킬 블로그 생성기 써보기

## 정리

1. 내가 만든 어플리케이션을 도커로 실행하기 위해서 최저 필요 조건 딱 하나 : Dockerfile
2. 도커파일은 기본적으로
3. 기존 이미지를 가져옴 ( 예. 알파인 리눅스, 우분투 리눅스, 노드가 기본으로 설치되어있는 리눅스 )
4. 오픈할 포트 지정 (보통 포트 3000 )
5. 내 어플리케이션을 실행할 명령어를 실행하면 됨 (npm install, npm start)
6. 남이 만든 도커 이미지 간단히 쓸수 있음. 그들이 이미 Dockerfile를 작성해서 이미지 생성후 올려놓은것을 그냥 가져다 쓰는것임
7. 좀더 편하게 쓰려면 docker-compose 를 써도 됨 : 도커 명령어 수행시 여러가지 옵션을 저장해두고 쓸수 있음. Dockerfile의 성격에 익숙해지면 docker-compose도 금방 익숙해짐. 한번에 알필요없음
8. 그 다음은 도커스웜/쿠버네티스. 하지만 기본은 언제나 Dockerfile

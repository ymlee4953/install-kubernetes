# Install jenkins in Docker on centOS

Reference : https://docs.docker.com/engine/install/centos/

## 1. Install Docker

### 1.1 Uninstall old versions

    $ sudo yum remove docker \
                    docker-client \
                    docker-client-latest \
                    docker-common \
                    docker-latest \
                    docker-latest-logrotate \
                    docker-logrotate \
                    docker-engine


### 1.2 SET UP THE REPOSITORY (Install using the repository)

Install the yum-utils package (which provides the yum-config-manager utility) and set up the stable repository.

    $ sudo yum install -y yum-utils

    $ sudo yum-config-manager \
        --add-repo \
        https://download.docker.com/linux/centos/docker-ce.repo


### 1.3 INSTALL DOCKER ENGINE

Install the latest version of Docker Engine and containerd, or go to the next step to install a specific version:

    $ sudo yum install docker-ce docker-ce-cli containerd.io

If prompted to accept the GPG key, verify that the fingerprint matches 060A 61C5 1B55 8A7F 742B 77AA C52F EB6B 621E 9F35, and if so, accept it.

### 1.4 Start Docker

    $ sudo systemctl start docker


### 1.5 Verify Docker Engine is installed

Verify that Docker Engine is installed correctly by running the hello-world image.

    $ sudo docker run hello-world


## 2. Install Jenkins In Docker

Reference : https://github.com/jenkinsci/docker/blob/master/README.md

### 2.1 Execute Jenkins Install

    docker run -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts 
    그런데, 이렇게 실행하면 터미널을 닫는 순간 컨테이너가 종료되므로 docker run -it ..........명령으로 실행을 시키고 최종적으로 Ctrl +P, Q를 눌러 docker에서 빠져나와야 됨

    Reference : [Docker 사용법 | 컨테이너 생성/종료/실행/진입](https://here4you.tistory.com/267)

따라서, 이렇게 실행하면 설치가 시작됨

    docker run -it -p 8080:8080 -p 50000:50000 jenkins/jenkins:lts


### 2.2 Initialize Jenkins Service in browser

In a browser, navigate to the Jenkins IP address with port (ex:8080)

    [public ip]:8080

Then you will see the "Getting Started" screen in the browser.

### 2.3 Copy and paste the temporary password.

In the terminal, which run the jenkins install task, you will see the temporary passwork like 

    *************************************************************
    *************************************************************
    *************************************************************

    Jenkins initial setup is required. An admin user has been created and a password generated.
    Please use the following password to proceed to installation:

    17df2c676e424b6294fbac6b163715e6

    This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

    *************************************************************
    *************************************************************
    *************************************************************

and 

    1. Copy the "temporary password" in the terminal (ex: 17df2c676e424b6294fbac6b163715e6)
    2. and paste into the "Administrator password" box in the browser
    3. and click the "Continue" button


### 2.4 Customize Jenkins

    Click one of two option :
        1. Install suggested plugins
        2. Select plugins to install

then plugins will be installed
  
### 2.5 Create First Admin User

    1. Account Name
    2. Password * 2
    3. Name
    4. e-mail address
    and click the "Save and Continue" button

### 2.6 Check the Jenkins URL

    Check and click the "Save and Finish" button
    click the "Start using Jenkins" button in the next screen

### 2.7 Escape from Container in the terminal

    Ctrl + P, Q를 입력

Now you can turn off the terminal and the Jenkins service continues


## To do Next 

Jenkins 컨테이너가 종료되더라도 기존 등록된 파이프라인 정보를 다시 읽을 수 있도록 세팅하는 방법을 추가해야 됨


EOF




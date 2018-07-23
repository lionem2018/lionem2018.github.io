---
layout: post
title: "[Docker/Tip]가상 하드디스크 파일(disk.vmdk) 옮기기#1 - Oracle VM VirtualBox 상에서 옮기기"
date: 2018-07-23 14:50:00 +0900
description: 도커의 가상 하드디스크 파일(disk.vmdk)를 다른 드라이브로 옮기는 방법을 소개해보려 한다.
img: /docker/docker_logo.png
tags: [Docker, Tip]
author: # Add name author (optional)
---
부득이하게 C 드라이브 용량이 부족해 프로그램이나 파일들을 정리하던 중, 내 윈도우 계정 폴더에서 약 7GB의 엄청난 용량을 자랑하는 .docker 파일을 발견했다.

도커 프로그램 자체는 D 드라이브에 설치했지만, 각각의 가상 머신 대한 컨테이너, 이미지, 정보 등이 .dokcer 파일에 저장되는 듯했다. 이 중에서도 가상 하드디스크 파일인 **disk.vmdk**가 해당 용량의 대부분을 차지하고 있었다.

![docker_tip1_capture1]({{site.baseurl}}/assets/img/docker/tip1/capture1.jpg)

이 거대한 파일을 어떻게 다른 드라이브에 옮길 수 있는지 구글링을 하다가 이것저것 해보고 나서야 겨우 해결 할 수 있었다. 혹여 나처럼 헤매고 있을 다른 사람들이나, 다시 이 자료를 찾게 될 미래의 나를 위해 조금 더 깔끔하고 자세하게 정리해보려 한다.

해당 글은 **윈도우10** 환경, **도커 버전 18.03.0-ce** 상에서 작성되었으며, **Docker QuickStart Terminal** 또는 **Oracle VM VirtualBox**를 사용했음을 미리 알린다.

 


# **disk.vmdk 파일을 어떻게 옮길 수 있을까**

disk.vmdk를 옮길 수 있는 방법은 크게 두 가지가 있다.

첫번째는 **Oracle VM VirtualBox 상에서 vmdk 파일을 옮기는 것**이고, 두번째는 **손수 vmdk 파일을 옮겨 가상 하드디스크로 재등록하는 것**이다.

위 두 방법은 disk.vmdk를 어떻게 옮기느냐 차이지만, 두번째 방법이 더 복잡하다.

다만 Oracle VM VirtualBox 상에서 옮길 수 없다면 두번째 방법을 사용하여야 한다.

이번 포스트에서는 첫번째 방법인 Oracle VM VirtualBox 상에서 옮기는 방법을 설명하려고 한다.

순서는 다음과 같다.

1. 작동중인 도커 가상 머신(Virtual Machine) 중지하기
2. disk.vmdk 파일 옮기기
3. 도커 가상 머신의 작동 여부 확인하기

사실상 작동 여부를 확인하는 절차만 빼면 두 단계 뿐이다.

 


# **작동중인 도커 가상 머신(Virtual Machine) 중지하기**

도커 가상 머신이 작동중이라면 가상 하드디스크 파일을 변경할 수 없다.

**가상 머신의 상태를 확인**하여 실행중인지 확인하여야 한다.

터미널에서 **명령어를 사용하여 종료**하거나, **Oracle VM VirtualBox에서 종료**하거나 자신에게 편한 방법을 사용하면 된다.

 

## **명령어로 중지시키기**

먼저 **Docker QuickStart Terminal**을 실행시켜, 터미널 상에서명령어로 제어하도록 한다.

아래 명령어는 **가상 머신을 조회**할 수 있는 명령어다.

    $ docker-machine ls

결과화면:

![docker_tip1_capture2]({{site.baseurl}}/assets/img/docker/tip1/capture2.jpg)

만약 가상 머신이 실행중이라면 **작동을 중지**하도록 하자.

    $ docker-machine stop
    또는
    $ docker-machine stop <가상 머신 이름>

가상 머신 이름을 명령어의 argument로 명시하지 않는다면, "default"라는 이름의 머신이 중지될 것이다.

결과화면:

![docker_tip1_capture3]({{site.baseurl}}/assets/img/docker/tip1/capture3.jpg)

다시 위의 명령어를 통해 작동이 중지된 것인지 확인하고 다음 단계로 넘어간다.

 

## **Oracle VM VirtualBox에서 중지시키기**

터미널에서 명령어를 입력하는 것 외에 손쉽게 가상 머신을 중지시키는 방법도 있다.

Oracle VM VritualBox를 실행시키면 좌측에 가상 머신과 해당 머신의 상태를 리스트로 볼 수 있을 것이다.

![docker_tip1_capture4]({{site.baseurl}}/assets/img/docker/tip1/capture4.png)

옮기고자 하는 가상 하드디스크를 사용하는 머신을 '**마우스 우클릭-닫기(C) > 전원 끄기(W)**'로 작동을 중지시킨다.

![docker_tip1_capture5]({{site.baseurl}}/assets/img/docker/tip1/capture5.png)

가상 머신의 상태가 '**전원 꺼짐**'으로 되어있는 것을 확인한 후 다음 단계로 넘어간다.

 


# **disk.vmdk 파일 옮기기**

Oracle VM VirtualBox에서 '**파일(F) - 가상 미디어 관리자(V)**'를 클릭하여 가상 미디어 관리자 창을 연다.

가상 미디어 관리자 창을 보면 툴바와 가상 디스크 목록이 보일 것이다.

![docker_tip1_capture6]({{site.baseurl}}/assets/img/docker/tip1/capture6.png)

우리가 옮기고자 하는 **disk.vmdk를 선택**한 후 상단 툴바에서 '**Move**' 버튼을 클릭한다.

그러면 어디에 disk.vmdk 파일을 옮길 것인지 선택할 수 있도록 파일 탐색기가 열릴 텐데,

적절한 공간을 선택해주면 vmdk 파일을 자동으로 옮겨줄 것이다.

![docker_tip1_capture7]({{site.baseurl}}/assets/img/docker/tip1/capture7.jpg)

0% 에서 멈춰있는 것으로 보일 수 있지만(본인은 그랬다), 조금 기다리면 위 다이얼로그가 사라질 것이다.

따로 이동이 완료되었다는 문구가 나타나지 않으니 파일이 잘 옮겨졌는지 확인하고 싶다면,

다시 'Move' 버튼을 클릭하여 파일 탐색기에서 우리가 옮기고자 했던 경로에 disk.vmdk 파일이 잘 옮겨졌는지 확인하면 된다.

옮기는 과정에서 오류가 발생했다면, 다음 포스트로 넘어가기 전에 먼저 포스트 하단의 [추가](#추가)부분을 확인하길 바란다.

 


# **도커 가상 머신의 작동 여부 확인하기**

Oracle VM VirtualBox에서 문제 없이 가상 하드디스크가 추가된 것을 확인했다면 **잘 인식되어 동작하고 있는지 확인**해아 한다.

 

## **가상 머신에 인식된 가상 하드디스크의 경로 확인하기**

가상 머신에 등록되어있는 가상 하디디스크의 경로 정보가 옮긴 파일로 제대로 인식하고 있는지 확인해야한다.

VM VirtualBox에서 가상 머신 선택 후 '**설정(S) - 저장소**'를 클릭한 후,

저장 목록에서 **disk.vmdk를 클릭**하여 우측에서 파일경로를 확인하도록 한다.

![docker_tip1_capture8]({{site.baseurl}}/assets/img/docker/tip1/capture8.png)

 

## **도커 가상 머신 실행하여 확인하기**

VM VirtualBox에서 실행하든, 터미널에서 확인하든 상관은 없다. 

**VM VirtualBox**에서는 '**시작(T)**' 버튼을 눌러 실행해보면 되고,

**터미널**의 경우, 아래 명령어를 통해 **가상 머신을 실행**시킬 수 있다.

    $ docker-machine start
    또는
    $ docker-machine start <가상 머신 이름>

결과화면:

![docker_tip1_capture9]({{site.baseurl}}/assets/img/docker/tip1/capture9.jpg)

가상 머신이 잘 실행되었다면 **기존 이미지들도 잘 동작하는지** 확인해본다.

아래 명령어는 **실행중이거나 중단된 모든 이미지를 조회**하는 명령어이다.

    $ docker images

결과화면:

![docker_tip1_capture10]({{site.baseurl}}/assets/img/docker/tip1/capture10.jpg)

오류 없이 작동 여부가 모두 확인되었다면 성공적으로 vmdk 파일이 옮겨진 것이다.

 

# **추가**

VM VirtualBox 상에서 disk.vmdk를 옮기는 과정에서 오류가 발생했다고 해서 무조건 두번째 방법으로 넘어갈 필요는 없다.

만일 옮길 수 없다는 오류가 발생했다면 **VM VirtualBox를 재시작**하여 시도 해보거나,

VM VirtualBox에서 가상 머신 선택 후 '**설정(S) - 저장소**'에서 **disk.vmdk를 연결 삭제**한 후 다시 시도해보기를 바란다.

연결을 삭제한다고 해서 파일이 삭제되는 것은 아니니 따로 백업할 필요는 없다.

![docker_tip1_capture11]({{site.baseurl}}/assets/img/docker/tip1/capture11.png)

그래도 오류가 여전히 발생한다면 다음 포스트의 방법을 시도하길 바란다.

# **참고**

해당 방법은 두 글을 참고하였다.

- [StackOverflow](https://stackoverflow.com/questions/33392133/move-boot2docker-and-docker-folder-in-other-drive)
- [NO, SERIOUSLY… 블로그](https://forgetfulprogrammer.wordpress.com/2016/10/28/how-to-move-your-large-virtualbox-vm-disk-created-by-docker/)


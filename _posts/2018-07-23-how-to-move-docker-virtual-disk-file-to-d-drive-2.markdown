---
layout: post
title: "[Docker/Tip]도커 가상 하드디스크 파일(disk.vmdk) 옮기기#2 - 직접 disk.vmdk 파일 옮기기"
date: 2018-07-23 16:25:00 +0900
description: 도커의 가상 하드디스크 파일(disk.vmdk)를 다른 드라이브로 옮기는 방법을 소개해보려 한다.
img: /docker/docker_logo.png
tags: [Docker, Tip]
author: # Add name author (optional)
---
해당 글은 **윈도우10** 환경, **도커 버전 18.03.0-ce** 상에서 작성되었으며, **Docker QuickStart Terminal** 또는 **Oracle VM VirtualBox**를 사용했음을 미리 알린다.


# **직접 disk.vmdk 파일을 옮기려면**

disk.vmdk를 옮길 수 있는 방법은 크게 두 가지가 있다.

첫번째는 **Oracle VM VirtualBox 상에서 vmdk 파일을 옮기는 것**이고, 두번째는 **손수 vmdk 파일을 옮겨 가상 하드디스크로 재등록하는 것**이다.

위 두 방법은 disk.vmdk를 어떻게 옮기느냐 차이지만, 두번째 방법이 더 복잡하다.

다만 Oracle VM VirtualBox 상에서 옮길 수 없다면 두번째 방법을 사용하여야 한다.

[이전 포스트](/how-to-move-docker-virtual-disk-file-to-d-drive-1)에서는 첫번째 방법을 설명했고,

이번 포스트에서는 두번째 방법인 직접 disk.vmdk 파일을 옮기는 방법을 설명하려고 한다.

순서는 다음과 같다.

1. 작동중인 도커 가상 머신(Virtual Machine) 중지하기
2. disk.vmdk 파일 옮기기
3. 기존의 가상 하드디스크 등록 해제 및 삭제하기
4. 옮긴 disk.vmdk 파일을 가상 하드디스크로 추가하기
5. 도커 가상 머신의 작동 여부 확인하기

1번과 5번 내용은 이전 포스트와 동일하다.

이미 도커 가상 머신을 중단했다면 2번부터 따라오면 된다.

# **작동중인 도커 가상 머신(Virtual Machine) 중지하기**

도커 가상 머신이 작동중이라면 가상 하드디스크 파일을 변경할 수 없다.

**가상 머신의 상태를 확인**하여 실행중인지 확인하여야 한다.

터미널에서 **명령어를 사용하여 종료**하거나, **Oracle VM VirtualBox에서 종료**하거나 자신에게 편한 방법을 사용하면 된다.

 

## **명령어로 중지시키기**

먼저 **Docker QuickStart Terminal**을 실행시켜, 터미널 상에서명령어로 제어하도록 한다.

아래 명령어는 **가상 머신을 조회**할 수 있는 명령어다.

```
$ docker-machine ls
```

![docker_tip1_capture2](/assets/img/docker/tip1/capture2.jpg)

만약 가상 머신이 실행중이라면 **작동을 중지**하도록 하자.

```
$ docker-machine stop
또는
$ docker-machine stop <가상 머신 이름>
```

가상 머신 이름을 명령어의 argument로 명시하지 않는다면, "default"라는 이름의 머신이 중지될 것이다.

![docker_tip1_capture3](/assets/img/docker/tip1/capture3.jpg)

다시 위의 명령어를 통해 작동이 중지된 것인지 확인하고 다음 단계로 넘어간다.

 

## **Oracle VM VirtualBox에서 중지시키기**

터미널에서 명령어를 입력하는 것 외에 손쉽게 가상 머신을 중지시키는 방법도 있다.

Oracle VM VritualBox를 실행시키면 좌측에 가상 머신과 해당 머신의 상태를 리스트로 볼 수 있을 것이다.

![docker_tip1_capture4](/assets/img/docker/tip1/capture4.png)

옮기고자 하는 가상 하드디스크를 사용하는 머신을 '**마우스 우클릭-닫기(C) > 전원 끄기(W)**'로 작동을 중지시킨다.

![docker_tip1_capture5](/assets/img/docker/tip1/capture5.png)

가상 머신의 상태가 '**전원 꺼짐**'으로 되어있는 것을 확인한 후 다음 단계로 넘어간다.


# **disk.vmdk 파일 옮기기**

문제의 disk.vmdk 파일을, 남은 용량이 넉넉한 아무 공간에 **복사**하자.

만일 **disk.vmdk의 위치를 정확히 모르겠다**면 포스트 하단의 [추가](#추가)부분을 참고했으면 한다.

내 경우는 '**C:\Users\계정명\\.docker\machine\machines\default**' 안에 저장되어 있었다.

![docker_tip1_capture12](/assets/img/docker/tip1/capture12.jpg)

나는 D 드라이브 내에 docker-machines/default 안에 disk.vmdk 파일을 복사하였다.

주의할 점은 vmdk 파일을 이동하는 것이 아니라, **복사**한다는 것이다.

또한, **이전 경로에 있는 disk.vmdk 파일을 삭제하지 않도록** 하자.


# **기존의 가상 하드디스크 해제 및 삭제하기**

VM VirtualBox을 실행하여 메인 화면에서 '**파일(F) - 가상 미디어 관리자(V)**'를 클릭한다. 가상 하드 디스크 파일 목록이 뜰 것이다.

![docker_tip1_capture6]({{site.baseurl}}/assets/img/docker/tip1/capture6.png)

가상 미디어 관리자 창에서 우리가 옮기고자 하는 disk.vmdk가 보일 것이다.

disk.vmdk 파일을 옮기기 위해서는 이를 등록 해제하여야 한다.

'**disk.vmdk 선택 - 등록 해제(L)**', 또는 이름 위에서 '**마우스 우클릭 - 등록 해제(L)**'를 누른다.

![docker_tip1_capture13]({{site.baseurl}}/assets/img/docker/tip1/capture13.png)

위와 같은 다이얼로그 창이 뜨면 '**연결 해제**'를 누른다.

이제 가상 미디어 관리자 창에서 기존의 disk.vmdk를 없애야 한다.

disk.vmdk 선택 후, 상단의 '**삭제**' 버튼을 눌러 삭제하도록 한다.

![docker_tip1_capture14]({{site.baseurl}}/assets/img/docker/tip1/capture14.png)

![docker_tip1_capture15]({{site.baseurl}}/assets/img/docker/tip1/capture15.png)

두번째 질문창에서 '**삭제**'를 선택하면 기존의 disk.vmdk가 **자동적으로 삭제**될 것이고, '**유지**'를 선택하면 **기존 파일이 그대로 존재**할 것이다. 나는 편의를 위해 '삭제'를 선택했다. '유지'를 선택한 경우, 모든 작업이 끝난 후 직접 삭제하도록 하자.

그러면 하드디스크 목록에서 disk.vmdk가 사라질 것이다.

![docker_tip1_capture16]({{site.baseurl}}/assets/img/docker/tip1/capture16.png)


# **옮긴 disk.vmdk 파일을 가상 하드디스크로 추가하기**

VM VirtualBox 상으로 기존 가상 하드디스크 파일을 삭제했기 때문에 가상 머신의 가상 디스크가 자동으로 해제되어있을 것이다.

우리가 옮긴 disk.vmdk 파일을 가상 디스크로 추가해주는 작업이 필요하다.

다시 **Oracle VM VirtualBox의 메인 화면**으로 돌아간다.

'**가상 머신 선택 - 설정**' 또는 '**우클릭 - 설정**'을 눌러 설정 창을 연다.

![docker_tip1_capture17]({{site.baseurl}}/assets/img/docker/tip1/capture17.png)

설정 창에서 '**저장소**' 메뉴를 누른다.

![docker_tip1_capture18]({{site.baseurl}}/assets/img/docker/tip1/capture18.png)

여기서는 해당 가상 머신에 가상 저장소를 추가하거나 해제할 수 있는데,

우리가 복사해두었던 disk.vmdk를 가상 하드디스크로 추가해주어야 하므로

**저장 장치** 목록 상단의 '**컨트롤러:...**' 옆 하드디스크 그림에 + 표시가 있는 아이콘(**가상 하드디스크 추가**)을 클릭하여 추가하도록 한다.

![docker_tip1_capture19]({{site.baseurl}}/assets/img/docker/tip1/capture19.png)

'**기존 디스크 선택하기(C)**'를 클릭하여 **vmdk 파일을 추가**해준다.

![docker_tip1_capture20]({{site.baseurl}}/assets/img/docker/tip1/capture20.png)

저장 장치 목록 마지막에 disk.vmdk가 추가될 것이다.

![docker_tip1_capture8]({{site.baseurl}}/assets/img/docker/tip1/capture8.png)

우측에 위치가 옮겨진 disk.vmdk 파일의 경로와 동일한 것을 확인하고, '**확인**' 버튼을 누른 후 다음 단계로 넘어가자. 

# **도커 가상 머신의 작동 여부 확인하기**

Oracle VM VirtualBox에서 문제 없이 가상 하드디스크가 추가된 것을 확인했다면 **잘 인식되어 동작하고 있는지 확인**해아 한다.

## **도커 가상 머신 실행하여 확인하기**

VM VirtualBox에서 실행하든, 터미널에서 확인하든 상관은 없다. 

**VM VirtualBox**에서는 '**시작(T)**' 버튼을 눌러 실행해보면 되고,

**터미널**의 경우, 아래 명령어를 통해 **가상 머신을 실행**시킬 수 있다.

```
$ docker-machine start
또는
$ docker-machine start <가상 머신 이름>
```

![docker_tip1_capture9](/assets/img/docker/tip1/capture9.jpg)

가상 머신이 잘 실행되었다면 **기존 이미지들도 잘 동작하는지** 확인해본다.

아래 명령어는 **실행중이거나 중단된 모든 이미지를 조회**하는 명령어이다.

```
$ docker images
```

![docker_tip1_capture10](/assets/img/docker/tip1/capture10.jpg)

오류 없이 작동 여부가 모두 확인되었다면 성공적으로 vmdk 파일이 옮겨진 것이다.


# **추가**

## **기존 disk.vmdk 파일의 경로 찾기**

disk.vmdk 파일을 직접 옮겨야 하지만 어디에 저장되어있는지 모르는 경우가 있을 수 있다.

이 경우, **Oracle VM VirtualBox**를 실행시켜 가상 머신의 하드 디스크 파일의 경로를 확인하면 된다.

![docker_tip1_capture4](/assets/img/docker/tip1/capture4.png)

'**가상 머신 선택 - 설정**' 또는 '**우클릭 - 설정**'을 눌러 설정 창을 연다.

![docker_tip1_capture17]({{site.baseurl}}/assets/img/docker/tip1/capture17.png)

설정 창에서 '**저장소**' 메뉴를 누른다.

**저장 장치** 목록에서 **disk.vmdk를 클릭**하여 위치 정보를 확인할 수 있다.

![docker_tip1_capture8]({{site.baseurl}}/assets/img/docker/tip1/capture8.png)

# **참고**

해당 방법은 두 글을 참고하였다.

- [StackOverflow](https://stackoverflow.com/questions/33392133/move-boot2docker-and-docker-folder-in-other-drive)
- [NO, SERIOUSLY… 블로그](https://forgetfulprogrammer.wordpress.com/2016/10/28/how-to-move-your-large-virtualbox-vm-disk-created-by-docker/)


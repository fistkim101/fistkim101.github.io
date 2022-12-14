---
layout: article
title: Kubernetes 시작하기
tags: Infra Kubernetes
---

<br>
<br>

해당 포스팅은 인프런에서 subicura님이 강의하신 [쿠버네티스 강의](https://www.inflearn.com/course/%EC%BF%A0%EB%B2%84%EB%84%A4%ED%8B%B0%EC%8A%A4-%EC%9E%85%EB%AC%B8#) 를 들으며
학습한 내용과 더 깊은 이해를 위해서 추가적으로 찾아본 자료들을 정리한 것이다.

<br>
<br>

{% include youtube.html id="PH-2FfFD2PU" %}

<br>

'쿠버네티스'라는 단어와 발음 자체가 너무 한국적이어서 원어민이 어떻게 쿠버네티스를 발음하는지 찾아보려다가 우연히 보게된 자료인데 상당히 설명을 잘 해놓은 것 같다.
물론 아직 쿠버네티스를 잘 알지 못해서 이 영상이 쿠버네티스의 흐름을 잘 설명해놓은건지 아닌지 판단할 수준이 되지 못하지만, 그래도 전체적인 쿠버네티스의 흐름이
5분 안에 이렇게 깔끔하게 전달되는 것을 느껴보니 잘 정리되었다고 생각이 든다.

subicura님의 강의는 아래와 같은 순서로 진행되고, 포스팅 역시 동일한 순서로 할 계획이다.

* 쿠버네티스 시작하기 <br>
    * 컨테이너 오케스트레이션
    * 왜 쿠버네티스인가
* 쿠버네티스 알아보기 <br>
    * 쿠버네티스 데모
    * 쿠버네티스 아키텍쳐 & 오브젝트 & API 호출
* 쿠버네티스 실습하기 <br>

이번 포스팅은 '쿠버네티스 알아보기'를 다 듣고 정리한 것으로 해당 챕터에서 가장 중요하다 생각되는 두 꼭지는 '컨테이너 오케스트레이션'이라는 것의 개념, 필요성과
'컨테이너 오케스트레이션' 을 위한 도구로 왜 쿠버네티스를 학습해야하는지이다.

<br>
<br>
<br>

### 쿠버네티스 시작하기

<br>

#### 컨테이녀 오케스트레이션

<br>

{% include youtube.html id="Ia8IfowgU7s" %}

<br>

잘 정리된 자료를 찾으려 했더니 내가 들은 유료강의의 일부가 아예 유투브에 공개되어있었다. 아마 강의 유입을 위한 도입부 노출(?) 인 것 같다.

[orchestrate](https://en.dict.naver.com/#/entry/enko/b39600f55ec24048b6bf7fcc0c6a3391) 은 흔히 우리가 알고 있는 오케스트라와 의미가 통하는 부분이 있는데
'조직하다'라는 의미로 이해하면 될 것 같다. 그래서 '컨테이너 오케스트레이션' 이라는 뜻은 결국 '컨테이너(들)을 관리하는 행위' 를 의미한다고 할 수 있다. 여기서 말하는 '관리' 라는 행위를
여러가지 행위로 세분화 해보면 결국 '컨테이너 오케스트레이션' 이 왜 필요한지도 알 수 있다.

강의에서 소개하고 있는 '컨테이너 오케스트레이션' 의 필요성들은 아래와 같다.

* cluster(중앙제어) : 동일한 목적을 위한 컨테이너들이 수가 점점 많아지면 결국 하나하나 접근하여 관리하는 것이 힘들기 때문에 이를 클러스터 단위로 추상화하여 묶어서 '중앙제어'를 통해서 편하게 관리한다. <br>
* state(상태관리) : 서버 관리자가 원하는 컨테이너(들)의 상태를 직접 조치 없이 자동으로 상태를 만들거나 유지 시켜 준다. <br>
* scheduling(배포관리) : 호스트들의 상태들을 파악하고 있다가 적절한 호스트에 추가적인 배포를 해주거나 적절한 상황으로 리소스를 사용한다. <br>
* rollout & rollback(버전관리) : 다수의 컨테이너를 한 번에 버전업을 해주거나 버전 롤백을 처리해준다. <br>
* service discovery(서비스 등록 및 조회) : 저장소를 모니터링 하고 있으면서 알아서 서비스가 추가가되면 프록시 추가 & 재시작 처리를 해준다. <br>
* volume(볼륨 스토리지) : 각 호스트 별로 필요한 볼륨을 마운트 하는 것을 조금 더 손쉽게 처리할 수 있게 도와준다. <br>

되게 내용이 많은 것 같지만 결국 핵심은 위와 같은 기능들을 통해서 **Desired State Management** 를 가능하게 해준다는 것이 핵심이다.

<br>

#### 왜 쿠버네티스인가
컨테이너 오케스트레이션 도구로 많은 것들이 이미 존재하고, 나왔으나 결국 쿠버네티스가 표준처럼 압도적인 위치를 차지하게 되었는데, 강의에서는 그 이유를 구체적으로 이야기하고 있지는 않다.
강의 내용을 바탕으로 추론해보면 구글에서 만들었고, 이미 구글은 컨테이너 관리를 오랫동안 사내 도구로 사용했었고 그 노하우를 녹여서 내놓은 것이 쿠버네티스라서 아마 기능적으로 다른 도구들을 압도한 것 같다.

<br>
<br>

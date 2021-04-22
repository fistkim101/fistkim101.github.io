---
layout: article
title: Kubernetes 알아보기
tags: Infra Kubernetes
---

<br>
<br>

#### 포스팅 목차

* 쿠버네티스 시작하기 <br>
    * 컨테이너 오케스트레이션
    * 왜 쿠버네티스인가
* 쿠버네티스 알아보기 <br>
    * 쿠버네티스 데모
* 쿠버네티스 실습준비 <br>
* 쿠버네티스 기본실습<br>

<br>
<br>
<br>

### 쿠버네티스 알아보기

<br>

#### 쿠버네티스 데모
쿠버네티스 시연을 통해서 학습 전에 미리 대략적으로 쿠버네티슫가 어떤 일을 해주는지에 대해서 구경(?) 하는 시간이었다.
이 부분 까지는 아직 유투브에 강의 영상이 이미 공개되어 있어 여기에도 첨부한다. 내가 각각의 인스턴스에 ssh로 들어가서
손으로 했던 것들을 한번에 다 해주고 있다.

{% include youtube.html id="dlI1PFCtfm0" %}

<br>
<br>

#### 쿠버네티스 아키텍쳐

강의에서 나온 그림인데 전체를 조망하기 좋은 이미지라서 첨부한다.
{% include image.html src="/assets/images/infra/k8s-architecture.png" %}

<br>

그리고 아래는 개별 컨트롤러들이 어떤 역할을 하는지에 대해서 Desired State개념과 같이 나타낸 그림이다.
컨트롤러는 특정한 상태를 감시하는 주체로 계속해서 '원하는 상태'가 지속되고 있는지를 모니터링 해주고 현재의 상태가 '원하는 상태'가 아니면
'원하는 상태'로 만들도록 조치를 취해주는 주체이다.
{% include image.html src="/assets/images/infra/k8s-desired-state.png" %}




<br>
<br>









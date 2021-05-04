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
    * 쿠버네티스 아키텍쳐 & 오브젝트 & API 호출
* 쿠버네티스 실습하기<br>

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

#### 쿠버네티스 아키텍쳐 & 쿠버네티스 오브젝트 & 쿠버네티스 API 호출

<br>

{% include image.html src="/assets/images/infra/k8s-architecture-1.png" %}

<br>

{% include image.html src="/assets/images/infra/k8s-architecture-2.png" text="pod는 k8s에서 가장 작은 배포단위이며 고유 IP를 할당 받으며 여러 컨테이너가 하나의 pod안에 들어갈 수 있다" %}

<br>

{% include image.html src="/assets/images/infra/k8s-architecture-3.png" %}

<br>

{% include image.html src="/assets/images/infra/k8s-architecture-4.png" %}

<br>

{% include image.html src="/assets/images/infra/k8s-architecture-5.png" %}

<br>

{% include image.html src="/assets/images/infra/k8s-architecture-6.png" %}

<br>

쿠버네티스는 결국 이게 전부이고 이 과정에서 쿠버네티스가 무엇을 어떻게 추상화 해두었는지, 사용법은 무엇인지를 학습하는 것이 쿠버네티스 학습의 핵심이다.

<br>
<br>

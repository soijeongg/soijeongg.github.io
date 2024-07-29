---
layout: post
read_time: true
show_date: true
title: "django project 시작"
date: 2024-07-28 15:00:20 -0600
description: "프로젝트 시작"
image: https://images.velog.io/images/dldhk97/post/35a88d4c-ef9f-4d8e-a361-c6d53e4e2162/django-logo-negative.png
tags: 
    - coding
    - diary
   
author: soi

toc: no

---

### MVT
장고에서 모델, 뷰, 템플릿 3가지를 말함

1. 모델
장고의 데이터 구조를 생성하고 데이터 베이스와 소통을 담당한다 

2. 템플릿 
웹 사이트의 화면 구성을 담당
장고 템플릿에서는 템플릿 랭귀지 라는 것이 있는데 진자처럼 변하는 데이터에 따라 화면을 다르게 구성할 수 있다 

3. view
로직을 담당하는 파트로 모델과 템플릿 사이를 연결하는 역할을 함
요청이 들어오면 그 요청을 처리하는 역할(필요하면 모델이나 템플릿과 상호작용함)

### mvc와의 차이
mvt는 mvc를 기본으로 해서 만들어졌다 
모델은 똑같이 데이터베이스와의 연결을 담당하지만 mvt의 뷰의 역할을 mvc에서는 컨트롤러(나눠서 서비스, 레포지토리로 나뉘기도 함)이 담당하고 템플릿의 역할을 뷰가 담당한다 

mvc에서는 컨트롤러가 중심에서 모든 요청을 맡아 처리하지만 mvt에서는 요청을 받아들이는 부분인 컨트롤러의 역할의 일부를 django가 직접 처리해 달흔 개발에 집중할 수 있다
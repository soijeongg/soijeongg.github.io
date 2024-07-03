---
layout: post
read_time: true
show_date: true
title: "타입스크립트와 typeORM으로 만드는 레이어드 패턴에 IRequest와 DTO"
date: 2024-06-26 15:01:20 -0600
description: "타입스크립트와 typeORM으로 만드는 레이어드 패턴에 IRequest와 DTO "
image: https://i.pinimg.com/564x/61/c3/b3/61c3b3ae85f68d1f742d58a5e8907815.jpg
tags: 
    - coding
    - cs
author: soi

toc: no # leave empty or erase for no TOC
---

먼저 타입스크립트를 설치한 다음에 시작지점인 app과 index.ts를 만든다 
그 후 기초 설정을 한 다음에 인터체이스를 만든다 
### 인터페이스란
객체지향 프로그래밍에서의 개념으로 클래스나 객체가 구현해야하는 메서드의 속성의 집합을 정의 
어떤 메서드가 포함되어야 한다는 명세만 있는것
코드의 일관성을 유지하고 표준화된 방식으로 객체를 다룰 수 있게 하고 다형성을 구현하는데 도움 된다 
인테페이스를 사용하게 되면 인터페이스에 의존하게 되기 때문에 실제 구현을 쉽게 교체할 수 있어 유지보수와 확장성이 좋다 

-> 정리하면 인터페이스는 설계도, 청사진

이런 인터페이스를 컨트롤러, 서비스, 레포지토리, 엔티티를 정의한다
컨트롤러, 서비스, 레포지토리는 각 계층의 구조를 설계하고 리퀘스트는 어떤게 들어올지 미리 정의한다 
엔티티 인터페이스는 엔티티의 구조를 정의하고 타입 검사를 한다  

컨트롤러 인터페이스는 req, res, next를 인자로 받고 Response를 반환한다 
서비스 인터페이스는 컨트롤러에서 들어오는 DTO를 받아  유효성 검사등 비즈니스 로직을 실행한다 
그리고 다시 레포지토리계층으로 보낼때 DTO로 묶어 보낸다 

```typescript
//엔티티 인터페이스 
export interface IUser {
  userId: number;
  email: string;
  password: string;
  name: string;
}
```
이렇게 정의한 후 컨트롤러 인터페이스를 정의한다 
```typescript
export interface IuserController {
  signupController(
    req: Request,
    res: Response,
    next: NextFunction,
  ): Promise<Response | void>;
}
```
그 후 서비스 계층의 인터페이스를 정의한다  서비스계층은 컨트롤러에서 dto로 묶어서 오는 것을 받는다 
``` typescript
export interface IuserService {
  createUserService(user: createUserDTO): Promise<IUser>;
}
```
이렇게 넘어온다고 가정하고 create하고 save한것을 반환한다는 것으로 설정해서  Iuser의 형식을 반환한다고 설게했다 

```typescript
export interface IuserRespository {
  createUser(user: createUserDTO): Promise<IUser>;
}
```
레포지토리는 넘어오는 DTo를 받아 처리한다 

컨트롤러 -> 서비스: DTO 사용
서비스 내부: DTO를 도메인 모델이나 비즈니스 로직에 맞는 형태로 변환
서비스 -> 리포지토리: 필요에 따라 다른 DTO로 변환하여 전달


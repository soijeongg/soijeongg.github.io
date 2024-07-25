---
layout: post
read_time: true
show_date: true
title: "nest.js api 구성하기"
date: 2024-07-10 15:00:20 -0600
description: "커스텀 데코레이터,예외처리, 로깅"
image: https://miro.medium.com/v2/resize:fit:820/0*xqTsRQ3MOnE1j2D9.png
tags: 
    - coding
    - diary
   
author: soi

toc: no # leave empty or erase for no TOC
---
### nest.js의 데코레이터

nest.js의 데코레이터는 크게 클래스, 메서드, 매개변수, 속성으로 나눌 수 있음
 1. 클래스 데코레이터
 @Module(): 모듈 클래스를 정의 
 @Controller(): 컨트롤러 클래스 정의 
 @Injectable(): 서비스 클래스 정의 

 2. 메서드 데코레이터
 http 요청 데코레이터: 컨트롤러에서 http메서드를 정의할때 get, post, put등을 시용

 3. 매개변수 데코레이터
 @Req, @Res, @Body, @query @Params

 4. 속성데코레이터
 클래스의 속성에 추가적인 메타데이터를 제공하거나 속성의 동작을 변경하는데 사용
 
 일반적으로 orm 라이브러리에서 많이 사용

### 커스텀 데코레이터
간단하게 만들어볼 수 있는 요청이 들어오는 클라이언트의 아이피를 트래킹하는 매개변수 데토레이터를 만들기
소스폴더에 가서 데토레이터 폴더 안에 만들기
**매개번수 데토레이터를 만들때는 createParamDecorator를 사용해 만든다 
```typeScript
import { ExecutionContext, createParamDecorator } from "@nestjs/common";

export const Ip = createParamDecorator(( data:unknown, ctx: ExecutionContext):string => {
    const request = ctx.switchToHttp().getRequest();
    return request.ip;
  },
);

```
이걸 다른 데코레이터들처럼 @를 사용
데코레이터를 커스텀하여 사용하는 것은 코드의 생산성이나 유지보수, 기능을 모듈화하는데 장점이 있음

### Exception filters
파이프는 라우트 핸들러에 도달하기 전에 동작하기 전에 사용되는 것이고 Exception filters는 클라이언트 요청이 들어오게 되고 라우터 핸들러에서 수행하다 예외가 발생하면 해당 예외를 처리하는 코드로 라우팅
자주 쓰이는 Exception은 unauthorized Exception , NotFoundException, Bad Request Exception, Forbidden Exception,NotAceeptableException

throw new HttpException로 에러 발생 가능
입세션 필터 사용시 전역적으로 사용하거나 해당 클래스에서만 사용하는 것도 가능
```typescript
import { ArgumentsHost, Catch, ExceptionFilter, HttpException } from "@nestjs/common";
import { Request, Response } from "express";
@Catch(HttpException)
export class httpExceptionFilter implements ExceptionFilter{
  catch(exception: HttpException, host: ArgumentsHost) {
    const ctx = host.switchToHttp();
    const response = ctx.getResponse<Response>();
    const status = exception.getStatus();
    const request = ctx.getRequest<Request>();

    response.status(status).json({
      statusCode: status,
      timestamp: new Date().toISOString,
      path: request.url,
    });
  }
}
//에러가 발생했을떄 에러에 대한 코드와 시간 경로를 리턴
//ctx로 가져와야 한다
```
이 이후 main파일에 가서  app.useGlobalFilters(new httpExceptionFilter()); 이렇게 적용

이 에러는 모든 HttpException을 처리하기 떄문에 unauthorized Exception , NotFoundException, Bad Request Exception, Forbidden Exception,NotAceeptableException 이 모든것이 포함된다 

따로따로 적용을 할려면 exception instanceof NotFoundException으로 찾아서 이때만 다르게 하거나 catch(exception: NotFoundException, host: ArgumentsHost) 로 잡아 작성한다 

### logger module를 사용한 로깅처리
logger module를 가져와서 사용한다 근데 이게 인젝터블해서 DI에서 쓰는게 아니라 가져와서 사용해야 한다

private readonly logger = new Logger();













































---
layout: post
read_time: true
show_date: true
title: "nest.js에서 jwt발급 및 검증 "
date: 2024-09-10 10:00:20 -0600
description: "nest.js에서 jwt발급 및 검증"
image:  https://miro.medium.com/v2/resize:fit:820/0*xqTsRQ3MOnE1j2D9.png
tags: 
    - coding
    - diary
   
author: soi

toc: no

---

### jwt 모듈 설치
jwt 발급을 위한 모듈을 설치한다 
```shell
npm install @nestjs/jwt @nestjs/passport passport passport-jwt

```

jwt모듈을 설치한 뒤 jwt를 발급할 userModule이나 authModule에 가서 JwtModule.registerAsync()를 설정한다 

이는 jwt 설정을 비동기적으로 정의할 수 있다 
```javascript
JwtModule.registerAsync({
      // 2. 비동기 방식으로 JwtModule을 등록
      imports: [ConfigModule], //환경변수 사용
      useClass: JwtConfigService, // JwtConfigService를 사용하여 설정 제공
      inject: [ConfigService],
    }),
```
useClass의 JwtConfigService를 사용해 설정을 제공하는데 이 설정에 환경변수를 사용하기 위해 ConfigService를 주입한다 

### JwtConfigService에서 비밀 키 설정
```javascript
import { Injectable } from '@nestjs/common';
import { JwtModuleOptions, JwtOptionsFactory } from '@nestjs/jwt';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class JwtConfigService implements JwtOptionsFactory {
  constructor(private configService: ConfigService) {}

  createJwtOptions(): JwtModuleOptions {
    return {
      secret: this.configService.get<string>('JWT_SECRET'),  // 환경 변수에서 비밀 키를 가져옴
      signOptions: {
        expiresIn: '60m',  // 만료 시간 (여기서는 60분으로 설정)
      },
    };
  }
}
```
JwtOptionsFactory를 사용해 jwt 모듈의 설정을 제공한다 

secret으로 비밀키를 사용하고 signOption옵션을 사용해 만료시간을 제공한다 

### jwt 토큰 발급
로그인을 하는 서비스로 들어가 이메일과 해시한 비밀번호가 맞는지 검사한 후에 맞다면 토큰을 발급한다 
```javascript
 async loginUserService(LoginDto: loginDTO) {
    const { email, password } = LoginDto;
    const user = await this.validateUser(email, password);
    if (!user) {
      throw new HttpException('로그인에 실패했습니다', HttpStatus.BAD_REQUEST);
    }
    const payload = { sub: user.userId, status: user.status };
    const accessToken = this.jwtService.sign(payload, { expiresIn: '1h' });
    const refreshToken = this.jwtService.sign(payload, {
      secret: this.configService.get<string>('JWT_REFRESH_SECRET'),
      expiresIn: '7d',
    });
    return {
      access_token: accessToken,
      refresh_token: refreshToken,
    };
  }
  ```
  jwtService.sign을 사용해 페이로드 안에 있는 정보를 가지고 토큰을 발급한다 
  
  비밀키 만료시간은 명시적으로 발급할떄 지정하지 않으면 JwtConfigService 설정이 들어가게 된다 
  
  JwtConfigService에서 엑세스 토큰의 비밀키와 시간을 담당하고 리프래시 토큰은 발급할때 비밀키와 만료시간을 설정한다 

이후 컨트롤러에서 해당 엑세스 토큰을 헤더에 할당하고 리프래시 토큰을 쿠키에 할당한다 

```javascript
@Post('/login')
  async loginController(
    @Body() LoginDTO: loginDTO,
    @Res() res: Response,
  ): Promise<void> {
    const { access_token, refresh_token } =
      await this.userService.loginUserService(LoginDTO);
    res.setHeader('Authorization', `Bearer ${access_token}`);
    res.cookie('refresh_token', refresh_token, {
      maxAge: 7 * 24 * 60 * 60 * 1000,
    });
    res.status(HttpStatus.OK).json({ message: '로그인 성공', access_token });
  }
  ```

  ### JWT 가드 
  이렇게 발급한 토큰이 헤더에 저장되어 있는데 이를 가드로 검증하고 request.user에 저장한다

  ```javascript
  import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtAuthGuard extends AuthGuard('jwt') {}
```
AuthGuard('jwt')는 기본가드로 passport를 사용해 인증로직을 처리하는 가드다 

클라이언트에서 전달된 토큰을 검증하고 인증을 처리한다 

AuthGuard('jwt')가 요청을 받으면 JwtStrategy로 넘어가 토큰을 검증한다 
```javascript

import { Injectable, UnauthorizedException } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';
import { PassportStrategy } from '@nestjs/passport';
import { ExtractJwt, Strategy } from 'passport-jwt';
import { UserService } from '../user/user.service';

@Injectable()
export class JwtStrategy extends PassportStrategy(Strategy) {
  constructor(
    private readonly configService: ConfigService,
    private readonly userService: UserService,
  ) {
    super({
      jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
      ignoreExpiration: false,
      secretOrKey: configService.get<string>('JWT_SECRET'),
      algorithms: ['HS256'],
    });
  }
  async validate(payload: any) {
    const user = await this.userService.findUserByID(+payload.sub);
    if (!user) {
      throw new UnauthorizedException('User not found');
    }
    return { ...user };
  }
}
// 발급하고 검증하는 코드
```
이곳에 만약 토큰 블랙리스트 검증하는 로직이 필요하다면 여기에 추가한다 
```javascript
  async validate(payload: any) {
    const user = await this.userService.findUserByID(+payload.sub);
    if (!user) {
      throw new UnauthorizedException('User not found');
    }
    const isBlacklisted = await this.tokenBlacklistRepository.findOne({
      where: { token: payload.jti },
    });
    if (isBlacklisted) {
      throw new UnauthorizedException('Token is blacklisted');
    }
    return { ...user };
  }
  ```
  이러면 검증 통과시 request.user에 저장된다 

  ### req.user 에러 해결
 로그인시 req.user 정의되지 않음 오류가 날 수도 있다 

 타입 지정해주는 부분을 파일로 빼서 코드를 작성해준다 
 
 ```javascript
 import { User } from '../user/entities/user.entity'; // User 엔티티 경로에 맞춰서 import

declare global {
  namespace Express {
    interface Request {
      user?: User; // Request 객체에 user 속성 추가
    }
  }
}
```
 Express의 Request 객체에 대한 타입 선언을 확장하는 부분으로 user를 추가함으로써 req.user가 User속성을 가지게 된다 

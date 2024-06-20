---
layout: post
read_time: true
show_date: true
title: "[node.js]passport.js 설정방법"
date: 2024-03-17 15:00:20 -0600
description: "[node.js]passport.js 설정방법"
image: https://velog.velcdn.com/images/soijeongg/post/39a726ef-36b8-470e-950c-b51e6e39d175/image.png
tags: 
    - coding
    - diary
    - hh99
author: soi

toc: no # leave empty or erase for no TOC
---

## 도입 배경
원래는 생으로 로그인시 들어온 이메일을 가지고 검색해 있으면 비밀번호검사하고 맞으면 user에 저장한다
``` javascript

import { prisma } from '../utils/prisma/index.js';

// 유저 인증에 실패하면 403 상태 코드를 반환한다.
export default async function (req, res, next) {
  try {
   
     const { userId,nickname } = req.session.user;

     if (!userId || !nickname)
       throw new Error('로그인이 필요합니다.');
      
     const dbUser = await prisma.User.findFirst({
       where: {
         userId: +userId,
         nickname: nickname,
       },
     });
    if (!dbUser) throw new Error('사용자가 존재하지 않습니다.');
    res.locals.user = dbUser;
    next();
  } catch (error) {
    // 인증에 실패하였을 경우 Cookie를 삭제합니다.
    console.error(error);
    return res.status(403).json({
      errorMessage: '로그인에 실패하였습니다.',
    });
  }
}
```
이 부분을 더 찾아보니 노드에는 passport.js가 존재했다 
우리조는 좀더 진행되면 구글이나 카카와 로그인또한 구현할 생각이기에 oauth을 지원하는 passport를 도입해야 겠다고 생각했다 

### passport.js
[passport.js 공식사이트](https://www.passportjs.org/)
passport.js는 노드에서 인증을 쉽게 구현할 수 있도록 도와주는 미들웨어
기본 인증부터 Facebook, Twitter 같은 소셜 로그인까지 다양한 인증 메커니즘을 지원
카카오, 구글, 트위터, 스포티파이, 깃허브 등을 지원!
(전에 스포티파이로 로그인하면 자기가 자주들은 음악 영수증 형태로 만든 사이트가 있는데 이것을 사용했을것 같다!)

### 사용방법
1. 먼저 yarn 이나 npm을 활용해 passport.js을 설치한다 
2. 그후 어떤전략을 사용할지 결정한다
로컬전략 사용시 본인의 db사용, 소셜로그인 사용시 각각 맞는 전략을 사용한다 (google, twitter)

3. passport 설정파일을 만들어 passport.js파일을 만든다 
4. 


### 작동 구조 
로그인 요청이 미들웨어로 들어오면 라우터 안에 있는 함수  passport.authenticate()로 넘어간다 
이 함수가 passport설정파일를 호출(passport.use를 찾아 넘어간다)
이 로그인 전략을 다 실행하고 나면 다시 그 아래 함수로 넘어간다 

done요청은 로깅이 성공하면 유저 정보를 반환하고 실패시 false를 반환한다 
있다면 req.login을 자동 호출한다 
req.login 메서드가 passport.serializeUser() 호출 
이안의 들어간 값이 done(null,) 뒤에 두번째로 들어간 값이 세션으로 넘어가서 req.session.user에 저장된다 
이 세션 적용은 express-session 을 사용해 적용한다 


### 함수로 보기
1. 먼저 passport.js를 만든다
```javascript

```
라우터에서 로그인시 적용한 함수가 호출된다 ( passport.authenticate)
```javascript
router.post('/login', (req, res, next) => {
  passport.authenticate('local', async (err, user, info) => {
    try {
      if (err) {
        return next(err);
      }
      if (!user) {
        return res.status(401).json({ message: info });
      }
      req.login(user, async (err) => {
        if (err) {
          console.log(err)
          return next(err);
        }
        req.user = user;
        res.locals.user = user;

        return res.json({ message: `${user.nickname}님 환영합니다!~` });
      });
    } catch (err) {
      return next(err);
    }
  })(req, res, next);
});
```
passport.use가 호출이되서 안에서 프리즈마 orm을 사용해 찾고 맞으면  done(null, user);호출
이후  다시 라우터로 넘어간다 

```javascript
passport.use("local", new LocalStrategy({
  usernameField: 'email',
  passwordField: 'password',
}, async (email, password, done) => {
  try {
    // 사용자 데이터베이스에서 이메일로 사용자 찾기
    const user = await prisma.User.findFirst({ where: { email: email } });
    if (!user) {
      return done(null, false, { message: '이메일이나 비밀번호가 틀립니다.' });
    }
    // 비밀번호 확인
    const isValidPassword = await argon2.verify(user.password, password);
    if (!isValidPassword) {
      return done(null, false, { message: '비밀번호가 일치하지 않습니다.' });
    }
    return done(null, user);
  } catch (error) {
    return done(error);
  }
}));
```
이후 다시 로그인 부분의 req.login으로 넘어간다 들어온 user의 값이 없으면 에러가 발생해 req.login으로 안넘어가고 에러가 발생한다 
req.login으로 넘어가 들어온 user의 값을 사용해 로그인을 한다 -> 성공시 세션에 저장을 한다 
req.login 메서드가 passport.serializeUser() 호출한다 
```javascript
// 사용자 정보를 세션에 저장
passport.serializeUser((user, done) => {
  done(null, user.userId);
});
passport.deserializeUser(async (userId, done) => {
  try {
    const user = await prisma.User.findFirst({ where: { userId } });
    done(null, user);
  } catch (error) {
    done(error);
  }
});
```
serializeUser은 세션에 저장하는 함수
이 이후 바로 아래에 있는 deserializeUser로 넘어간다
이 함수는 세션의 식별자를 받아 사용자를 식별하는데 사용된다 
식별이 되면 done(null, user) 부분이 req.user에 설정된다 
코드 리팩토링하면 이 이 부분에 req.user의 값을 여기서 설정해도 될거 같다 
다시 위의 로그인 라우터에 있는 req.login으로 넘어간다 
이후 이 req.login의 응답부분으로 가 값을 실행하면 세션쿠키에 이 값이 전달된다 

### 그외 
app.js에 passport가 세션을 사용한다고 설정을해야한다
app.use(passport.initialize()); 
app.use(passport.session());
그리고 설정파일 import 해주기
로그인한것을 확인하는 미들웨어 구현
``` javascript
//로그인되었는지를 검사하는 인증 미들웨어
export default function authMiddleware(req, res, next) {
  if (req.isAuthenticated()) {
    //console.log(req.user);
    res.locals.user = req.user;

    return next();
  }
  res.clearCookie('connect.sid'); // 세션 쿠키 이름이 'connect.sid'인 경우. 실제 쿠키 이름에 맞게 변경하세요.
  return res.status(401).json({ message: '인증이 필요합니다.' });
};
//로그인이 되었는데 올려고 하면이걸로 막기
export default function isNotLoggedIn = (req, res, next) => {
   if (!req.isAuthenticated()) {
      next(); 
   } else {
      const message = encodeURIComponent('로그인한 상태입니다.');
      res.redirect(`/?error=${message}`);
   }
};
```

로그아웃시 사용한 세션을 부순다 
```javascript
router.delete('/logout',authMiddleware,  (req, res, next) => {
  req.logOut(function (err) {
    if (err) {
      return next(err);
    }
    req.session.destroy()
    return res.json({ message: '로그아웃' });
  });
});
```


### 해야할것 
이제 passport.js를 이해했으니 다시 코드 리팩토링 하러 가야겠다...
근데 진짜 어렵기는 해

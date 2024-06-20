---
layout: post
read_time: true
show_date: true
title: "nodemailer로 인증메일 보내기"
date: 2024-03-22 15:00:20 -0600
description: "nodemailer로 인증메일 보내기"
tags: 
    - coding
    - diary
    - hh99
author: soi

toc: no # leave empty or erase for no TOC
---

### 사전설정
원래는 구글메일로 할려고 했으나 구글메일이 이제 낮은 수준의 보완수준을 안사용한다고 해서 네이버로 옮겼다 
네이버메일에가서 환경설정에 들어가 POP3/IMAP 설정으로 들어간다 
여기에 사용한다고 설정하고 IMAP/SMTP 설정에 을 눌러보면 아래 있는 포트를 사용한다 

### 설정
`yarn add nodemailer`를 설치한다 
```javascript
import nodemailer from 'nodemailer';
import crypto from 'crypto';
import dotenv from 'dotenv';

dotenv.config();

export function generateRandomPassword() {
  return crypto.randomBytes(20).toString('hex');
}

const transporter = nodemailer.createTransport({
  service: 'naver',
  host: 'smtp.naver.com',
  port: 587,
  secure: false,
  requireTLS: true,
  auth: {
    user: process.env.USER,
    pass: process.env.PASS,
  },
});

export const sendVerificationEmail = (userEmail, verificationToken) => {
  const mailOptions = {
    from: `${process.env.USER}@naver.com`,
    to: userEmail,
    subject: '트렐로 회원가입 인증 이메일입니다',
    html: `<p>아래의 링크를 클릭하여 회원가입을 완료하세요.</p>
           <p><a href="https://api.nodejstrello.site/api/verify?token=${verificationToken}">회원가입 인증하기</a></p>`,
  };

  transporter.sendMail(mailOptions, (error, info) => {
    if (error) {
      console.log(error);
    } else {
      console.log(`인증 이메일 발송: ${info.response}`);
    }
  });
};
```
위의 auth에 내 네이버 아이디와 비밀번호를 넣어준다 
그리고 mailOptions를 사용해 메일에 보낼 값을 넣어준다 
이 값을 회원가입시 사용되는 컨트롤러에 create후에 넣어준다 
그리고 이렇게만 하면 메일만 보내지기 때문에 인증미들웨어에 이에 관한 내용을 넣어준다 

### 인증이 안될때 막기
인증이 안될때 막아주기 위해서 유저 테이블에 인증이 된지 안된는지를 추적하는 테이블을 만든다 
그리고 지금은 링크를 사용해서 그냥 받아오는 걸로 하고 싶었다 
그러기 위해서 위에 링크에 토큰을 넣어서 보내주고 그 토큰 값을 테이블에 저장하기로 했다 

1. 유저 테이블에 isVerified, verificationToken,provider를 넣는 다 
isVerified: 불린으로 인증되면 1, 안되면 0
verificationToken: 인증할때 사용하는 토큰 
provider = 어느 sns를 통해 로그인했는지 null값 허용
구글 로그인시에는 google이 들어가고 local은 null이 들어감

만약 isVerified가 0이면 아예 로그인이 안되게 해놨다 
인증시  이 라우터로 들어온다 
```javascript
router.get('/verify', UserController.getVerifyController);

verifyUserEmail = async (token) => {
    let findUser = await this.userRespository.findUserByToken(token);
    if (!findUser) {
      const error = new Error('만료되었거나 없는 토큰 입니다');
      throw error;
    }
    let checkEmail = await this.userRespository.updateStatus(findUser.userId);
    return checkEmail;
  };

```
들어온 토큰을 검사해 토큰이 있으면 토큰을 업데이트 한다 
인증을 1로 바꿔서 로그인이 되게한다 

### 처리 
지금 구글 로그인 부분에는 토큰을 넣는 부분이 없으니 거기에 토큰 부분 추가
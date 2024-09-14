---
layout: post
read_time: true
show_date: true
title: "nest.js에서 multer 사용해 파일 전송 "
date: 2024-09-10 10:00:20 -0600
description: "multer 사용해 파일 전송"
image:  https://miro.medium.com/v2/resize:fit:820/0*xqTsRQ3MOnE1j2D9.png
tags: 
    - coding
    - diary
   
author: soi

toc: no

---

nest.js에서 multer를 사용하기 위해서는 multer 미들웨어를 사용해야 한다
mnest.js에서는 전용모둘인  @nestjs/platform-express을 사용해야 한다

### 사용방법
1. 설치 : yarn add @nestjs/platform-express

2. 모듈에 파일 업로드 설정 정의하기
MulterModule을 사용해 파일 업로드 설정을 모듈에서 등록하고 distStorge를 사용해 파일의 저장경로 및 파일명을 설정한다 
```javascript
 MulterModule.register({
      storage: diskStorage({
        destination: './profile', // 파일 저장 경로
        filename: (req, file, callback) => {
          // 원래 파일명과 확장자를 유지한 채 파일 저장
          const fileExtName = extname(file.originalname); // 확장자 추출
          const fileName = `${Date.now()}-${Math.round(Math.random() * 1e9)}${fileExtName}`; // 유니크한 파일 이름 생성
          callback(null, fileName); // 파일 이름 설정
        },
      }),
    }),
```
컨트롤러에서는 이설정을 바탕으로 UseInterceptors 데코레이터를 사용해 업로드를 처리한다

- 하나의 파일 업로드를 할 경우
FileInterceptor는 클라이언트의 단일 파일을 처리하고 멀터의 설정을 통합해 사용한다 
```javascript
@Controller('upload')
export class UploadController {
  @Post('single')
  @UseInterceptors(FileInterceptor('file'))  // 'file'은 클라이언트에서 보내는 파일 필드 이름
  uploadFile(@UploadedFile() file: Express.Multer.File) {
    console.log(file);  // 업로드된 파일의 정보 확인
    return {
      message: 'File uploaded successfully!',
      file: file,  // 업로드된 파일 정보를 반환
    };
  }
}

```

- 여러 파일 업로드를 할경우 
``` javascript
@Controller('upload')
export class UploadController {
  @Post('single')
    @UseInterceptors(FilesInterceptor('files', 10))  // 파일 필드 이름 'files', 최대 10개 파일 처리
  uploadFile(@UploadedFile() files: Array<Express.Multer.File>) {
    console.log(file);  // 업로드된 파일의 정보 확인
    return {
      message: 'File uploaded successfully!',
      files: files,  // 업로드된 파일 정보를 반환
    };
  }
}
``` 
이렇게 파일의 정보를 받아서 처리하고 파일의 이름을 추출해 string타입으로 데이터베이스에 저장하면 된다 
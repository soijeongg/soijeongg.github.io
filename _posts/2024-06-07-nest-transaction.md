---
layout: post
read_time: true
show_date: true
title: "nest에서 트랜잭션 사용하기"
date: 2024-05-11 15:00:20 -0600
description: "쿼리러너사용/트랜잭션 데코레이터 사용"
image: https://miro.medium.com/v2/resize:fit:820/0*xqTsRQ3MOnE1j2D9.png
tags: 
    - coding
    - diary
 
author: soi

toc: no # leave empty or erase for no TOC
---
### 트랜잭션
트랜잭션이란 데이터베이스의 상태를 변경시키기 위한 하나의 작업단위로 더이상 분할이 불가능한 업무단위를 말한다 
트랜잭션이 전부 실패하던지 전부 성공하던지 해야한다 

### 쿼리빌더 
쿼리빌더를 사용할려면 엔티티 매니저나 커넥션을 주입해 사용하고 쿼리러너를 선언해야한다

엔티티 매니저를 쓸때는 레포지토리를 사용하지 않는다 

```javascript
@Injectable()
export class CategoriesService {
  constructor(
    @InjectRepository(Category)
    private categoriesRepository: Repository<Category>,
    @InjectEntityManager() private entityManager: EntityManager
  ) {}

  async createUserWithProfile(userData: any, profileData: any) {
    return await this.entityManager.transaction(async (manager) => {
      // 새 유저 생성
      const user = await manager.save(User, userData); 
      // 유저 테이블에 유저 데이터값을 저장하겠다

      // 프로필 데이터에 유저를 연결해 프로필 데이터에 유저 객체 넣기
      profileData.user = user;
      const profile = await manager.save(Profile, profileData);
      // 프로필 테이블에 프로필 데이터 저장

      // 트랜잭션이 자동 커밋됨
      return { user, profile };
    });
  }
}
```
만약 커넥션을 사용하고 싶다면 위와 비슷하게 커넧션을 사용해 쿼리러너를 불러오면 된다
```javascript
mport { Injectable } from '@nestjs/common';
import { Connection } from 'typeorm';
import { InjectConnection } from '@nestjs/typeorm';
import { User } from './user.entity';
import { Profile } from './profile.entity';

@Injectable()
export class UserService {
  constructor(@InjectConnection() private connection: Connection) {}

  async createUserWithProfile(userData: Partial<User>, profileData: Partial<Profile>) {
    const queryRunner = this.connection.createQueryRunner();

    await queryRunner.connect();
    await queryRunner.startTransaction();

    try {
      const user = await queryRunner.manager.save(User, userData);
      profileData.userId = user.id;
      await queryRunner.manager.save(Profile, profileData);

      await queryRunner.commitTransaction();
      return user;
    } catch (err) {
      await queryRunner.rollbackTransaction();
      throw err;
    } finally {
      await queryRunner.release();
    }
  }
}
```

- 둘의 차이 
엔티티매니저는 자동으로 트랜잭션을 관리해 시작, 커밋, 롤백을 수동으로 하지않아도 되고 코드가 간결하다

커넥션을 사용한 트랜잭션은 쿼리러너 객체를 사용해 수동으로 제어 할 수 있고 수동으로 트랜잭션 관리를 해야한다 

또한 격리 수준을 지정하거나 여러 엔티티를 활용해 할 수 있다

### 트랜잭션 데코레이터 사용하는 방법
@Transaction(): 트랜잭션을 시작하는 메서드를 지정
@TransactionManager(): 트랜잭션에서 사용할 EntityManager를 주입
@TransactionRepository(): 트랜잭션 내에서 특정 엔티티의 Repository를 주입받을 때 사용

```javascript
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository, EntityManager, Transaction, TransactionManager, TransactionRepository } from 'typeorm';
import { User } from './user.entity';
import { Profile } from './profile.entity';

@Injectable()
export class CategoriesService {
  constructor(
    @InjectRepository(User)
    private readonly userRepository: Repository<User>,

    @InjectRepository(Profile)
    private readonly profileRepository: Repository<Profile>,
  ) {}

  @Transaction() //트랜잭션임을 지정
  async createUserWithProfile(
    userData: any,
    profileData: any,
    @TransactionManager() manager: EntityManager, //사용할 엔티티 매니저 주입
  ) {
    // 새 유저 생성
    const user = await manager.save(User, userData); 
    // 유저 테이블에 유저 데이터값을 저장

    // 프로필 데이터에 유저를 연결해 프로필 데이터에 유저 객체 넣기
    profileData.user = user;
    const profile = await manager.save(Profile, profileData); 
    // 프로필 테이블에 프로필 데이터 저장

    return { user, profile };
  }
}

```

데코레이터 사용시 명시적인 제어 작성이 필요없고 코드가 간결해진다

하지만 이 데코레이터는 TypeORM 환경에 의존적으로 다른 ORM이나 다른 트랜잭션 관리방식시에는 이 데코레이터를 사용할 수 없고 복잡한 로직 처리가 힘들고 테스트가 어렵다
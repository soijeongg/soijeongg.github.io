---
layout: post
read_time: true
show_date: true
title: "typeORM과 queryBuilder"
date: 2024-06-05 15:00:20 -0600
description: "typeORM과 queryBuilder의 옵션"
image: https://miro.medium.com/v2/resize:fit:820/0*xqTsRQ3MOnE1j2D9.png
tags: 
    - coding
    - diary
 
author: soi

toc: no # leave empty or erase for no TOC
---

### typeORM의 옵션

- relation

관계된 엔티티를 함께 로드할떄 사용
```javascript
const category = await repository.findOne({
  where: { id: 1 },
  relations: ['subCategories']
});
```

- select

특정 컬럼만 선택적으로 가져올떄 사용
```javascript
const category = await repository.findOne({
  where: { id: 1 },
  select: ['id', 'name']
});
```
- where

조건을 지정할때 사용
```javascript
const categories = await repository.find({
  where: { isActive: true }
});
```

- order

결과를 정렬할때 사용
```javascript
const categories = await repository.find({
  order: { name: 'ASC' }
});
```

- skip and Take

페이지네이션을 구현할때 사용
```javascript
const categories = await repository.find({
  skip: 5, //이전 페이지의 항목을 건너뜀
  take: 10 //한 페이지에서 가죠올 할목의 수 
});
```

### 쿼리빌더

기본 쿼리빌더 사용

```javascript
const queryBuilder = repository.createQueryBuilder('category');
```

#### 조건설정

- leftJoinAndSelect

관계된 엔티티 조인 및 선택

```javascript
queryBuilder
    .leftJoinAndSelect('category.subCategories', 'subCategory')
```

- select

특정 컬럼만 선택적으로 가져올떄 사용
```javascript
queryBuilder
  .select(['category.id', 'category.name'])
```
- where

조건을 지정할때 사용
```javascript
queryBuilder
 .where('category.isActive = :isActive', { isActive: true })
 .andWhere('category.name LIKE :name', { name: '%test%' })
```

- order

결과를 정렬할때 사용
```javascript
queryBuilder
 .orderBy('category.name', 'DESC')
```

- 그룹화 
```javascript
queryBuilder
  .groupBy('category.id')
```
- 그룹조건
```javascript
queryBuilder
  .having('COUNT(subCategory.id) > :count', { count: 5 })

```
- skip and Take

페이지네이션을 구현할때 사용
```javascript
queryBuilder
  .skip(5)
  .take(10)

```
쿼리빌더의 메서드로 조회전 조건을 지정하고 .getMany, .getOne, .getRawMany, getCount로 가져온다

이거 하나만 가져올려면 조건 아래 붙이면 되고 한조건에 대해 여러 값들이 필요허면 쿼리필더의 메서드로 활용해 가져온다

```javascript
 // getMany: 여러 결과 가져오기
  const categories = await queryBuilder.getMany();

  // getOne: 단일 결과 가져오기
  const category = await queryBuilder.getOne();

  // getRawMany: Raw 데이터로 여러 결과 가져오기
  const rawCategories = await queryBuilder.getRawMany();

  // getCount: 결과 개수 가져오기
  const count = await queryBuilder.getCount();

  // rightJoinAndSelect 예제
  const categoriesWithRightJoin = await getRepository(Category)
    .createQueryBuilder('category')
    .rightJoinAndSelect('category.subCategories', 'subCategory')
    .where('subCategory.isActive = :isActive', { isActive: true })
    .getMany();
```
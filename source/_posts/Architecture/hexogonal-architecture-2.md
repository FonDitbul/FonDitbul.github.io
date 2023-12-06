---
title: Clean Architecture 구현 해보기 (2) 
toc: true
published: true
date: 2023-12-06 18:45:51
tags:
  - Clean Architecture
  - Hexagonal Architecture
categories:
  - Backend
  - Architecture
---


* 이번에는 NestJs 프레임워크를 활용해 클린 아키텍처를 구현하는 예제를 만들어보고자 합니다.
  * Clean Architecture 에 대한 자세한 설명은 1편을 참고해주세요.
* 기술스택
  * TypeScript, NestJs, PostgreSQL, TypeORM
  * [Node](https://nodejs.org/en) 와 npm, [NestJs](https://docs.nestjs.com/), ORM 등 에 대한 자세한 내용은 다루지 않겠습니다.
  * Node, npm, NestCLI, PostgreSQL 등은 이미 설치, 설정 되어 있다는 가정하에 진행하겠습니다.

# 유저의 회원가입 만들기
* 먼저 개발을 진행할 유저의 회원가입에 대한 요구사항 기획, 설계를 간단하게 작성 후 구현해보도록 하겠습니다.

## 요구사항
***
* 유저는 입력받은 유저 id와 email로 회원가입 할 수 있다.
* 유저의 id는 중복될 수 없다.
* 유저의 email은 중복될 수 없다.
* 회원가입 한 데이터는 데이터베이스에 저장한다.

## 요구사항에 대한 설계
*** 
### DB
* User 스키마 
```
  id        int4        increment PK
  user_id   varchar(40)
  email     varchar(100)
```
### API
```
method POST
path   /user/sign-up
body   key    type    commnet
       userId string  필수, 사용하고자 하는 유저 아이디 'ex) test'
       email  string  필수, 이메일 형식 'ex) test@gmail.com' 
```

# 구현하기
***
## NestJS Project 시작 하기
***
* 먼저 Nest 프로젝트를 생성합니다.
```shell
nest new hexagonal-architecture
```
```shell
cd hexagonal-architecture
```


* 필요 없는 파일들을 삭제하고 다음과 같은 폴더구조를 생성하겠습니다.  
  {% asset_img structure.PNG %}
* user 폴더 안에 application, domain, infrastructure, interface 를 위와 같이 생성합니다.

## Domain Layer
***
1. **user domain class 만들기**
```typescript
// /user/domain/user.ts
export class User {
  id: number;
  userId: string;
  email: string;
}
```

2. **service interface 만들기**
```typescript
// /user/domain/user.service.ts
import { UserSignUpIn } from "../interface/user.in";

export interface IUserService {
  signUp: (signUpIn: UserSignUpIn) => Promise<void>;
}
```
* in, out interface port는 하단에서 작성합니다.
* 회원가입 로직을 위한 ```signUp``` 메서드를 생성합니다.

3. **repository interface 만들기**
```typescript
// /user/domain/user.repository.ts
import { UserSignUpOut } from '../interface/user.out';
import { User } from './user';

export interface IUserRepository {
  signUp: (signUpOut: UserSignUpOut) => Promise<void>;
  findOneOrNullByUserId: (userId: User['userId']) => Promise<User | null>;
  findOneOrNullByEmail: (userId: User['email']) => Promise<User | null>;
}

```
* ```findOneOrNullByUserId``` 메서드는 user_id 컬럼으로 user 테이블을 조회 하여 존재 할 경우 return 없을 경우 null을 return 합니다.  
* ```findOneOrNullByEmail``` email 을 통해 user 테이블을 조회합니다. return 값은 위와 같습니다.


## Interface Layer
***
1. **in port interface 만들기**
```typescript
// /user/interface/user.in.ts
import { User } from '../domain/user';

export type UserSignUpIn = Pick<User, 'userId' | 'email'>;
```
* 외부에서 요청 받은 값이 서버 안으로 들어오는 interface 는 in 으로 정의합니다.
* 회원가입을 위해 userId와 email이 필요하기 때문에 Pick을 통해 정의합니다.

2. **out port interface 만들기**
```typescript
// /user/interface/user.out.ts
import { User } from '../domain/user';

export type UserSignUpOut = Pick<User, 'userId' | 'email'>;
```
* 서버에서 처리하는 데이터가 infrastructure 레이어를 통해 외부로 나가므로 out으로 정의합니다.


## Application Layer
*** 
1. **비즈니스 로직 구현하기**
* 도메인에 정의한 user.service.ts 의 구현체를 작성합니다.

```typescript
// /user/application/user.service.ts
import { IUserService } from '../domain/user.service';
import { Inject, Injectable } from '@nestjs/common';
import { UserSignUpIn } from '../interface/user.in';
import { IUserRepository } from '../domain/user.repository';

@Injectable()
export class UserService implements IUserService {
  constructor(
    @Inject('IUserRepository') private userRepository: IUserRepository,
  ) {}

  async signUp(signUpIn: UserSignUpIn): Promise<void> {
    const { userId, email } = signUpIn;

    const userByUserId =
      await this.userRepository.findOneOrNullByUserId(userId);
    if (userByUserId) {
      throw new Error('중복된 userId가 존재합니다.');
    }

    const userByEmail = await this.userRepository.findOneOrNullByEmail(email);
    if (userByEmail) {
      throw new Error('중복된 email 이 존재합니다.');
    }

    await this.userRepository.signUp({ userId, email });

    return;
  }
}
```
* Domain 레이어를 통해 구현체를 작성하였고 해당 Layer 에서는 repository 의 구현체에 대한 의존하고 있지 않습니다.
  * 따라서 Repository 의 도메인만 참고하여 로직을 작성할 수 있게 됩니다.
* 로직
  1. 입력받은 userId를 통해 중복된 User 가 있는지 검사합니다.
     * 같은 유저 아이디가 존재할 경우 에러가 발생하며 회원가입이 실패합니다.
  2. 입력받은 email을 통해 중복된 user 가 존재하는지 확인합니다.
     * 같은 이메일이 존재할 경우 에러가 발생하며 회원가입이 실패합니다.
  3. 중복된 userId와 email이 없다면 새로운 유저를 생성합니다.

*** 
* **로직에 따른 test 코드 작성하기**

```typescript
// /user/application/user.service.spec.ts
import { UserService } from './user.service';
import { IUserRepository } from '../domain/user.repository';
import { User } from '../domain/user';
import { UserSignUpOut } from '../interface/user.out';
import { UserSignUpIn } from '../interface/user.in';

class UserRepositoryMocking implements IUserRepository {
  findOneOrNullByEmail(email: User['email']): Promise<User | null> {
    return Promise.resolve(null);
  }

  findOneOrNullByUserId(userId: User['userId']): Promise<User | null> {
    return Promise.resolve(null);
  }

  signUp(signUpOut: UserSignUpOut): Promise<void> {
    return Promise.resolve(null);
  }
}

describe('User Service test', () => {
  const userRepositoryMocking = new UserRepositoryMocking();
  const sut: UserService = new UserService(userRepositoryMocking);

  describe('유저 회원 가입 테스트', () => {
    it('유저 id 와 email 을 입력받아 회원가입에 성공한 경우', async () => {
      const givenSinUpIn: UserSignUpIn = {
        userId: 'user_1234',
        email: 'test@email.com',
      };
      jest
        .spyOn(userRepositoryMocking, 'findOneOrNullByUserId')
        .mockResolvedValue(null);
      jest
        .spyOn(userRepositoryMocking, 'findOneOrNullByEmail')
        .mockResolvedValue(null);

      await sut.signUp(givenSinUpIn);
    });

    it('유저 id가 중복되어 회원가입이 실패한 경우', async () => {
      const givenSinUpIn: UserSignUpIn = {
        userId: 'user_1234',
        email: 'test@email.com',
      };
      const givenUser: User = {
        id: 1,
        userId: 'user_1234',
        email: 'test1234@email.com',
      };
      jest
        .spyOn(userRepositoryMocking, 'findOneOrNullByUserId')
        .mockResolvedValue(givenUser);

      await expect(async () => sut.signUp(givenSinUpIn)).rejects.toThrowError(
        new Error('중복된 userId가 존재합니다.'),
      );
    });

    it('유저 id는 중복 X email 의 중복이 존재하는 경우', async () => {
      const givenSinUpIn: UserSignUpIn = {
        userId: 'user_1234',
        email: 'test@email.com',
      };
      const givenUser: User = {
        id: 1,
        userId: 'user_123456',
        email: 'test@email.com',
      };
      jest
        .spyOn(userRepositoryMocking, 'findOneOrNullByUserId')
        .mockResolvedValue(null);

      jest
        .spyOn(userRepositoryMocking, 'findOneOrNullByEmail')
        .mockResolvedValue(givenUser);

      await expect(async () => sut.signUp(givenSinUpIn)).rejects.toThrowError(
        new Error('중복된 email 이 존재합니다.'),
      );
    });
  });
});

```
{% asset_img test_success.PNG %}
* 아직 repository 의 구현체가 존재하지 않아도 테스트를 실행해보면 위와같이 테스트가 수행되는 것을 확인할 수 있습니다.


### Infrastructure Layer
***
1.1 **controller 만들기**
```typescript
// /user/infrastructure/user.controller.ts
import { Body, Controller, Inject, Post } from '@nestjs/common';
import { UserSignUpDto } from './user.dto';
import { IUserService } from '../domain/user.service';

@Controller()
export class UserController {
  constructor(@Inject('IUserService') private userService: IUserService) {}
  @Post('/user/sign-up')
  async signUp(@Body() signUpDto: UserSignUpDto) {
    const { userId, email } = signUpDto;

    await this.userService.signUp({
      userId,
      email,
    });
    return true;
  }
}

```

1.2 **DTO (data transfer object) 만들기**
```typescript
// /user/infrastructure/user.dto.ts
export class UserSignUpDto {
  readonly userId: string;

  readonly email: string;
}
```
* Client의 요청에 필요한 Dto를 위와 같이 정의합니다. 

***

2. **repository TypeORM 구현체 만들기**
* TypeORM을 통한 구현체 만들기
* 먼저 TypeORM에 필요한 라이브러리를 설치합니다.
> npm install --save @nestjs/typeorm typeorm pg

* **Entity 만들기**
```typescript
// user/infrastructure/typeorm/user.typeorm.entity.ts
import { Entity, Column, PrimaryGeneratedColumn } from 'typeorm';

@Entity({
  name: 'user',
})
export class UserEntity {
  @PrimaryGeneratedColumn('increment')
  id: number;

  @Column({ name: 'user_id' })
  userId: string;

  @Column()
  email: string;
}
```
* 파일 명에 해당 외부 라이브러리를 나타내는 typeorm을 표기합니다.

* **Repository 구현체 만들기**
```typescript
// user/infrastructure/typeorm/user.typeorm.repository.ts
import { IUserRepository } from '../../domain/user.repository';
import { Injectable } from '@nestjs/common';
import { InjectRepository } from '@nestjs/typeorm';
import { Repository } from 'typeorm';
import { UserEntity } from './user.typeorm.entity';
import { UserSignUpOut } from '../../interface/user.out';
import { User } from '../../domain/user';

@Injectable()
export class UserTypeormRepository implements IUserRepository {
  constructor(
    @InjectRepository(UserEntity)
    private repository: Repository<UserEntity>,
  ) {}

  private convert(entity: UserEntity): User {
    return entity;
  }
  
  async findOneOrNullByEmail(email: User['email']): Promise<User | null> {
    const oneUser = await this.repository.findOneBy({ email });

    return this.convert(oneUser);
  }

  async findOneOrNullByUserId(userId: User['userId']): Promise<User | null> {
    const oneUser = await this.repository.findOneBy({ userId });

    return this.convert(oneUser);
  }

  async signUp(signUpOut: UserSignUpOut): Promise<void> {
    const { userId, email } = signUpOut;

    await this.repository.save({ userId: userId, email: email });
    return;
  }
}
```
* TypeORM 라이브러리를 통해 domain 에 작성했던 메서드들을 실제 구현합니다.

***

3. module 만들기
* **database module 만들기**
```typescript
// /database/database.module.ts
import { Module } from '@nestjs/common';
import { TypeOrmModule } from '@nestjs/typeorm';
import { UserEntity } from '../user/infrastructure/typeorm/user.typeorm.entity';

@Module({
  imports: [TypeOrmModule.forFeature([UserEntity])],
  controllers: [],
  providers: [],
  exports: [TypeOrmModule.forFeature([UserEntity])],
})
export class DatabaseModule {}

```
* database module을 따로 생성합니다.
  * TypeORM 이라는 세부적인 선택사항은 해당 모듈에만 의존하도록 합니다.
  * 다른 모듈에서는 database module만 import 하여 사용할 수 있도록 합니다.


* **user module 만들기**
```typescript
// /user/user.module.ts
import { Module } from '@nestjs/common';
import { UserController } from './infrastructure/user.controller';
import { UserService } from './application/user.service';
import { DatabaseModule } from '../database/database.module';
import { UserTypeormRepository } from './infrastructure/typeorm/user.typeorm.repository';

@Module({
  imports: [DatabaseModule],
  controllers: [UserController],
  providers: [
    {
      provide: 'IUserService',
      useClass: UserService,
    },
    {
      provide: 'IUserRepository',
      useClass: UserTypeormRepository,
    },
  ],
})
export class UserModule {}
```
* 모든 레이어에 의존성을 Module 파일에 위와 같의 정의합니다.
> * interface 를 통한 DI에 자세한 내용
> * [Custom providers](https://docs.nestjs.com/fundamentals/custom-providers)


* **app module 수정하기**
```typescript
// /app.module.ts
import { Module } from '@nestjs/common';
import { UserModule } from './user/user.module';
import { TypeOrmModule } from '@nestjs/typeorm';

@Module({
  imports: [
    UserModule,
    TypeOrmModule.forRoot({
      type: 'postgres',
      host: 'localhost',
      port: 5432,
      username: 'postgres',
      password: 'postgres',
      database: 'hexagonal',
      autoLoadEntities: true,
      synchronize: false,
      logging: true,
    }),
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

[예제 코드](https://github.com/FonDitbul/hexagonal-architecture.git)
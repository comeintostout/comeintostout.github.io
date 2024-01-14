---
layout: post
title:  "[Nestjs] 정적 모듈에서의 환경변수 사용"
author: Stouter
categories: [Trouble Shooting, Nestjs]
tags: [Trouble Shooting, Nestjs]
#image: assets/images/11.jpg
featured: false
hidden: false
comments: true
---

`NestJS`에서는 `dotenv` 등의 외부 라이브러리가 아닌 내장된 `ConfigModule`를 통해 환경 변수를 접근할 수 있도록 해주는데. `.env` 파일을 삽입해두면 하위 모듈에서 이를 `process.env` 식으로 접근하여 환경 변수를 사용할 수 있다.

여기에 `TypeORM`을 연동하려고 다음과 같이 `AppModule` 을 구성했다.

```tsx
// app.module.ts
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";
import { TypeOrmModule } from "@nestjs/typeorm";
import { typeORMConfig } from "./configs/typeorm.config";
import { User } from "./user/user.entity";
import { UserModule } from "./user/user.module";

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRoot(typeORMConfig),
    UserModule
  ],
  controllers: [],
  providers: []
})
export class AppModule {}
```

```tsx
// typeorm.config.ts
import { TypeOrmModuleOptions } from "@nestjs/typeorm";
import { User } from "../user/user.entity";

export const typeORMConfig: TypeOrmModuleOptions = {
  type: "postgres",
  host: "localhost",
  port: 5432,
  username: process.env.DB_USERNAME,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_ID,
  entities: [User],
  synchronize: true
};
```

그러나 이 때, 오류가 발생한다. 환경 변수 `DB_ID` 값을 이상한 값으로 가져와서 DB 연동에 실패하는 오류가 자꾸 생겼었다. 알아본 결과 그 이유는 다음과 같았다.

> 모듈은 정적으로 만들어진다. Nest에서의 configModule은 정적이다. 그래서 안 된다. 무슨 말이냐면, controller나 service처럼, Module들로 인해 모든 application이 만들어진 다음에야 configModule을 사용할 수 있다는 것이다. **module의 생성 시점에는 configModule을 사용할 수 없다.** (출처 : [Nest.js에 ConfigModule 설정](https://velog.io/@kakasoo/Nest%EC%97%90%EC%84%9C-ConfigModule-TypeORM-%EC%93%B0%EA%B8%B0))

<aside>
<b>💡 Module의 생성 시점에는 configModule을 사용할 수 없다.</b>
</aside>

<br>

# 모듈을 동적으로 만들자

---

아래와 같이 `ConfigModule`과 다른 `Module`들이 생성된 후에, 해당 환경 변수를 조회하도록 순서를 강제해주자.

```tsx
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";
import { TypeOrmModule } from "@nestjs/typeorm";
import { User } from "./user/user.entity";
import { UserModule } from "./user/user.module";

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    TypeOrmModule.forRootAsync({
      useFactory: () => ({
        type: "postgres",
        host: process.env.DB_HOST,
        port: 5432,
        username: process.env.DB_ID,
        password: process.env.DB_PASSWORD,
        database: process.env.DB_NAME,
        entities: [User],
        synchronize: true
      })
    }),
    UserModule
  ],
  controllers: [],
  providers: []
})
export class AppModule {}
```

<br>

# 모듈을 동적으로 만들자 2

아래의 방식은 `JwtModule`에 환경변수 `JWT_SECRET_KEY`를 주입하다가 알아낸 방식이다.

```tsx
// AppModule 에 JWTModule 추가하기.
import { Module } from "@nestjs/common";
import { ConfigModule } from "@nestjs/config";
import { JwtModule } from "@nestjs/jwt";
import { UserModule } from "./user/user.module";

@Module({
  imports: [
    ConfigModule.forRoot({ isGlobal: true }),
    JwtModule.registerAsync({
      inject: [ConfigService],
      useFactory: (config: ConfigService) => ({
        secret: "" + config.get<string>("JWT_SECRET_KEY"),
        signOptions: { expiresIn: "86400s" }
      })
    }),
    UserModule
  ],
  controllers: [],
  providers: []
})
export class AppModule {}
```

`JwtModule`의 등록 부분에서 `secret` 부분을 자세히 보면 환경 변수를 불러오고 빈 문자열`‘’`을 더해준다. 단순히 가져온 토큰을 `string` 형식으로 형 변환 시키기 위함이 아닌, 해당 부분에 강제로 연산을 부여해주어 해당 부분의 연산을 Module이 모두 생성된 이후로 미루게 하는 방법이다. 그렇게 된다면, 해당 시기엔`ConfigModule`이 생성된 이후의 시점이므로 에러가 발생하지 않고 환경 변수를 잘 불러온다.
매우 기발하다!

<br>

# 개선점

---

위와 같은 방식에서 가장 불편한(?) 점은 `Module` 부분에 `TypeOrmModuleOptions` 같은 부분이 일일이 적힌다는 것이다. 나중에 `module`에 내용이 많아진다면, 코드가 지저분해질 수 있다. 해당 부분을 해결할 순 없을까?

# 참조

[Nest.js에 ConfigModule 설정](https://velog.io/@kakasoo/Nest%EC%97%90%EC%84%9C-ConfigModule-TypeORM-%EC%93%B0%EA%B8%B0)

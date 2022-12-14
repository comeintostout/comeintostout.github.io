---
layout: post
title: ๐จ Nest.js ์ ์  ๋ชจ๋์์์ ํ๊ฒฝ๋ณ์ ์ฌ์ฉ
thumbnail-img: /assets/img/nestjs_icon.png
tags: [ErrorNote, Nestjs]
comments: true
---

`NestJS`์์๋ `dotenv` ๋ฑ์ ์ธ๋ถ ๋ผ์ด๋ธ๋ฌ๋ฆฌ๊ฐ ์๋ ๋ด์ฅ๋ `ConfigModule`๋ฅผ ํตํด ํ๊ฒฝ ๋ณ์๋ฅผ ์ ๊ทผํ  ์ ์๋๋ก ํด์ฃผ๋๋ฐ. `.env` ํ์ผ์ ์ฝ์ํด๋๋ฉด ํ์ ๋ชจ๋์์ ์ด๋ฅผ `process.env` ์์ผ๋ก ์ ๊ทผํ์ฌ ํ๊ฒฝ ๋ณ์๋ฅผ ์ฌ์ฉํ  ์ ์๋ค.

์ฌ๊ธฐ์ `TypeORM`์ ์ฐ๋ํ๋ ค๊ณ  ๋ค์๊ณผ ๊ฐ์ด `AppModule` ์ ๊ตฌ์ฑํ๋ค.

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
    UserModule,
  ],
  controllers: [],
  providers: [],
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
  synchronize: true,
};
```

๊ทธ๋ฌ๋ ์ด ๋, ์ค๋ฅ๊ฐ ๋ฐ์ํ๋ค. ํ๊ฒฝ ๋ณ์ `DB_ID` ๊ฐ์ ์ด์ํ ๊ฐ์ผ๋ก ๊ฐ์ ธ์์ DB ์ฐ๋์ ์คํจํ๋ ์ค๋ฅ๊ฐ ์๊พธ ์๊ฒผ์๋ค. ์์๋ณธ ๊ฒฐ๊ณผ ๊ทธ ์ด์ ๋ ๋ค์๊ณผ ๊ฐ์๋ค.

> ๋ชจ๋์ ์ ์ ์ผ๋ก ๋ง๋ค์ด์ง๋ค. Nest์์์ configModule์ ์ ์ ์ด๋ค. ๊ทธ๋์ ์ ๋๋ค. ๋ฌด์จ ๋ง์ด๋๋ฉด, controller๋ service์ฒ๋ผ, Module๋ค๋ก ์ธํด ๋ชจ๋  application์ด ๋ง๋ค์ด์ง ๋ค์์์ผ configModule์ ์ฌ์ฉํ  ์ ์๋ค๋ ๊ฒ์ด๋ค. **module์ ์์ฑ ์์ ์๋ configModule์ ์ฌ์ฉํ  ์ ์๋ค.** (์ถ์ฒ : [Nest.js์ ConfigModule ์ค์ ](https://velog.io/@kakasoo/Nest%EC%97%90%EC%84%9C-ConfigModule-TypeORM-%EC%93%B0%EA%B8%B0))

<aside>
<h3>๐ก Module์ ์์ฑ ์์ ์๋ configModule์ ์ฌ์ฉํ  ์ ์๋ค.</h3>
</aside>

<br>

# ๋ชจ๋์ ๋์ ์ผ๋ก ๋ง๋ค์

---

์๋์ ๊ฐ์ด `ConfigModule`๊ณผ ๋ค๋ฅธ `Module`๋ค์ด ์์ฑ๋ ํ์, ํด๋น ํ๊ฒฝ ๋ณ์๋ฅผ ์กฐํํ๋๋ก ์์๋ฅผ ๊ฐ์ ํด์ฃผ์.

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
        synchronize: true,
      }),
    }),
    UserModule,
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

<br>

# ๋ชจ๋์ ๋์ ์ผ๋ก ๋ง๋ค์ 2

์๋์ ๋ฐฉ์์ `JwtModule`์ ํ๊ฒฝ๋ณ์ `JWT_SECRET_KEY`๋ฅผ ์ฃผ์ํ๋ค๊ฐ ์์๋ธ ๋ฐฉ์์ด๋ค.

```tsx
// AppModule ์ JWTModule ์ถ๊ฐํ๊ธฐ.
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
        signOptions: { expiresIn: "86400s" },
      }),
    }),
    UserModule,
  ],
  controllers: [],
  providers: [],
})
export class AppModule {}
```

`JwtModule`์ ๋ฑ๋ก ๋ถ๋ถ์์ `secret` ๋ถ๋ถ์ ์์ธํ ๋ณด๋ฉด ํ๊ฒฝ ๋ณ์๋ฅผ ๋ถ๋ฌ์ค๊ณ  ๋น ๋ฌธ์์ด`โโ`์ ๋ํด์ค๋ค. ๋จ์ํ ๊ฐ์ ธ์จ ํ ํฐ์ `string` ํ์์ผ๋ก ํ ๋ณํ ์ํค๊ธฐ ์ํจ์ด ์๋, ํด๋น ๋ถ๋ถ์ ๊ฐ์ ๋ก ์ฐ์ฐ์ ๋ถ์ฌํด์ฃผ์ด ํด๋น ๋ถ๋ถ์ ์ฐ์ฐ์ Module์ด ๋ชจ๋ ์์ฑ๋ ์ดํ๋ก ๋ฏธ๋ฃจ๊ฒ ํ๋ ๋ฐฉ๋ฒ์ด๋ค. ๊ทธ๋ ๊ฒ ๋๋ค๋ฉด, ํด๋น ์๊ธฐ์`ConfigModule`์ด ์์ฑ๋ ์ดํ์ ์์ ์ด๋ฏ๋ก ์๋ฌ๊ฐ ๋ฐ์ํ์ง ์๊ณ  ํ๊ฒฝ ๋ณ์๋ฅผ ์ ๋ถ๋ฌ์จ๋ค.
๋งค์ฐ ๊ธฐ๋ฐํ๋ค!

<br>

# ๊ฐ์ ์ 

---

์์ ๊ฐ์ ๋ฐฉ์์์ ๊ฐ์ฅ ๋ถํธํ(?) ์ ์ `Module` ๋ถ๋ถ์ `TypeOrmModuleOptions` ๊ฐ์ ๋ถ๋ถ์ด ์ผ์ผ์ด ์ ํ๋ค๋ ๊ฒ์ด๋ค. ๋์ค์ `module`์ ๋ด์ฉ์ด ๋ง์์ง๋ค๋ฉด, ์ฝ๋๊ฐ ์ง์ ๋ถํด์ง ์ ์๋ค. ํด๋น ๋ถ๋ถ์ ํด๊ฒฐํ  ์ ์์๊น?

# ์ฐธ์กฐ

[Nest.js์ ConfigModule ์ค์ ](https://velog.io/@kakasoo/Nest%EC%97%90%EC%84%9C-ConfigModule-TypeORM-%EC%93%B0%EA%B8%B0)

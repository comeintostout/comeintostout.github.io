---
layout: post
title: ๐ย Nest.js Socket Server์์ disconnect ๊ฐ์งํ๊ธฐ
thumbnail-img: /assets/img/socket.io_icon.png
cover-img: /assets/img/socket.io_icon.png
gh-repo: boostcampwm-2022/web05-SleepyWoods
gh-badge: [star, follow]
tags: [Nestjs, socket]
comments: true
---

๋ถ์คํธ์บ ํ์์ ํ์๋ค๊ณผ `SleepyWoods`๋ผ๋ ์ค์๊ฐ WebSocket ์๋น์ค๋ฅผ ๋ง๋ค์์ ๋, `Socket.io`๋ฅผ ์ฌ์ฉํ์๋ค. ์ด๋ฒ ๊ธ์์๋ ๊ฐ๋จํ๊ฒ ํด๋ผ์ด์ธํธ์ disconnect๋ฅผ ๊ฐ์งํ๋ ๋ถ๋ถ์ ๊ตฌํํด๋ณด์.

# HandleDisconnect

---

Nest.js์์  Socket Server๋ฅผ ๊ตฌํํ  ๋, ์ฃผ๋ก ๋ค์๊ณผ ๊ฐ์ด `SocketGateWay`๋ฅผ ๊ตฌ์ฑํ๋ค.

```tsx
export class SocketGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  public handleConnection(client: Socket): void {
    // ํด๋ผ์ด์ธํธ๊ฐ ์ ์๋์์ ๋, ํ  ์ผ
  }

  public handleDisconnect(client: Socket): void {
    // ํด๋ผ์ด์ธํธ๊ฐ ์ ์ ํด์ ๋์์ ๋, ํ  ์ผ
  }
}
```

`SocketGateway`๋ `OnGatewayConnection`์ `OnGatewayDisconnect`๋ฅผ ์์ํจ์ผ๋ก์จ, ํ์์ ์ผ๋ก `handleConnection`๊ณผ `handleDisconnect`๋ฅผ ๋ฉ์๋๋ก ์ง๋๊ฒ ๋๋ค.

์ฌ๊ธฐ์ ํด๋ผ์ด์ธํธ๊ฐ ์ผ๋ฐ์ ์ธ ์ ์ ํด์ ๋ฅผ ํ  ๊ฒฝ์ฐ ๋๋ถ๋ถ์ `handleDisconnect`๋ก ์ด๋ฅผ ๊ฐ์งํ๊ณ , ์ฒ๋ฆฌํด์ค ์ ์๋ค.

<br>

# HandleDisconnect๋ ๋ชจ๋  disconnect์ ๊ฐ์งํ ๊น?

---

๊ฐ์ด Socket Server๋ฅผ ๊ตฌ์ฑํ๋ ํผ์ด์ ์คํํด๋ณธ ๊ฒฐ๊ณผ, `handleDisconnect`๋ ๋ชจ๋  disconnect๋ฅผ ๊ฐ์งํ์ง ๋ชปํ๋ค. ๋ค์๊ณผ ๊ฐ์ ์์ธ ์ํฉ์ด ์์ ์ ์๋ค.

**โ ๏ธย ํด๋ผ์ด์ธํธ๊ฐ ๋น์ ์์ ์ธ ์ข๋ฃ๋ฅผ ํ์ ๊ฒฝ์ฐ**

- ๋ธ๋ผ์ฐ์ ์ ๊ฐ์  ์ข๋ฃ๊ฐ ์ผ์ด๋ฌ์ ๋,
- ๋ธํธ๋ถ์ ๋ฐฐํฐ๋ฆฌ๊ฐ ๋ค ๋์ด ๋ธํธ๋ถ ์์ฒด๊ฐ ๊ฐ์  ์ข๋ฃ ๋์์ ๋,

์์ ์ํฉ ๋ง๊ณ ๋, ์ฐ๋ฆฌ๊ฐ ํ์คํธํด๋ณด์ง ๋ชปํ ๋ง์ ์์ธ ์ํฉ๋ค์ด ์์ ์ ์๋ค. ๋ง์ฝ disconnect๋ฅผ ๊ฐ์งํ์ง ์์๋ ๋๋ ์๋น์ค๋ผ๋ฉด ๋ชจ๋ฅด๊ฒ ์ง๋ง, ์ฐ๋ฆฌ ์๋น์ค์์  disconnect ์์ ํ์์ ์ธ ๋ก์ง์ด ์์ด ์ด ์ ์ด ์ค์ํ๋ค.

```tsx
public handleDisconnect(client: sleepySocket): void {
    // ๋ค๋ฅธ ์ ์ ๋ค์๊ฒ ํด๋น ์ ์ ๊ฐ ๋ก๊ทธ์์ ํ๋ค๋ ์ ๋ณด ์ ๋ฌ
    // ํด๋น ์ ์ ์ ๊ทธ ๋ ์ ํ๋๋์ ๊ธฐ๋ก
}
```

<br>

# PingInterval๊ณผ pingTimeout

---

Nest.js ์์ `SocketGateway`๋ฅผ ๊ตฌ์ฑํ ๋, `@WebSocketGateway` ๋ฐ์ฝ๋ ์ดํฐ๋ฅผ ํตํด, WebSocket์ option๊ฐ์ ๋ฃ์ด์ค ์ ์๋ค. ๋ค์๊ณผ ๊ฐ์ด `SocketGateway`๋ฅผ ์์ ํ๋ค.

```tsx
@WebSocketGateway({
  pingInterval: 5000,
  pingTimeout: 3000,
})
export class SocketGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  public handleConnection(client: Socket): void {
    // ํด๋ผ์ด์ธํธ๊ฐ ์ ์๋์์ ๋, ํ  ์ผ
  }

  public handleDisconnect(client: Socket): void {
    // ํด๋ผ์ด์ธํธ๊ฐ ์ ์ ํด์ ๋์์ ๋, ํ  ์ผ
  }
}
```

์์ ๋ ๊ฐ์ง option๋ค์ ํตํด heartbeat๋ฅผ ์ค์ ํ  ์ ์๋๋ฐ, ์ด๋ connection์ด ์ ํจํ ์ง ์ฃผ๊ธฐ์ ์ผ๋ก ํ์ธํ๋ ๊ณผ์ ์ด๋ผ ๋ณผ ์ ์๋ค.

์๋ฒ๋ `pingInterval`๋ง๋ค connection์ ํ์ธํ๊ณ , ๋ง์ง๋ง ์์ฒญ์ผ๋ก๋ถํฐ `pingTimeout` ์๊ฐ ๋์ ์๋ต์ด ์๋ค๋ฉด ํด๋ผ์ด์ธํธ๊ฐ ์ฃฝ์ ๊ฒ์ผ๋ก ํ๋จ ํด disconnect ์ํจ๋ค.
(์์ ์์์์๋ 5์ด๋ง๋ค ํ์ธ์ ํ๋ฉฐ, ๋ง์ง๋ง ์์ฒญ์์ 3์ด๊ฐ ์๋ต์ด ์์์ผ๋ฉด ์ฐ๊ฒฐ์ด ํด์ ๋์๋ค๊ณ  ํ๋จํ๋ค.)

์ด๋ฅผ ํตํด ํด๋ผ์ด์ธํธ๊ฐ ์๊ธฐ์น ๋ชปํ ์ ์ ์ข๋ฃ๋ฅผ ํ๋๋ผ๋, ์๋ฒ์์ ์ด๋ฅผ ์ก์๋ผ ์ ์๋ค.

์๋์ `Socket.io` ๋ํ๋จผํธ์์ ์กฐ๊ธ ๋ ์์ธํ ์ ๋ณด๋ฅผ ํ์ธํ  ์ ์๋ค.

<br>

# ์ฐธ์กฐ

---

[Server options](https://socket.io/docs/v4/server-options/)

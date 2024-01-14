---
layout: post
title:  "[Nestjs] Socket Server에서 disconnect 감지하기"
author: Stouter
categories: [Nestjs, Socket]
tags: [Nestjs, Socket]
#image: assets/images/11.jpg
description: "My review of Inception movie. Acting, plot and something else in this short description."
featured: false
hidden: false
#rating: 4.5
comments: true
---


부스트캠프에서 팀원들과 `SleepyWoods`라는 실시간 WebSocket 서비스를 만들었을 때, `Socket.io`를 사용했었다. 이번 글에서는 간단하게 클라이언트의 disconnect를 감지하는 부분을 구현해보자.

# HandleDisconnect

---

Nest.js에선 Socket Server를 구현할 때, 주로 다음과 같이 `SocketGateWay`를 구성한다.

```tsx
export class SocketGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  public handleConnection(client: Socket): void {
    // 클라이언트가 접속되었을 때, 할 일
  }

  public handleDisconnect(client: Socket): void {
    // 클라이언트가 접속 해제되었을 때, 할 일
  }
}
```

`SocketGateway`는 `OnGatewayConnection`와 `OnGatewayDisconnect`를 상속함으로써, 필수적으로 `handleConnection`과 `handleDisconnect`를 메소드로 지니게 된다.

여기서 클라이언트가 일반적인 접속 해제를 할 경우 대부분은 `handleDisconnect`로 이를 감지하고, 처리해줄 수 있다.

<br>

# HandleDisconnect는 모든 disconnect을 감지할까?

---

같이 Socket Server를 구성하던 피어와 실험해본 결과, `handleDisconnect`는 모든 disconnect를 감지하지 못한다. 다음과 같은 예외 상황이 있을 수 있다.

**⚠️ 클라이언트가 비정상적인 종료를 했을 경우**

- 브라우저의 강제 종료가 일어났을 때,
- 노트북의 배터리가 다 되어 노트북 자체가 강제 종료 되었을 때,

위의 상황 말고도, 우리가 테스트해보지 못한 많은 예외 상황들이 있을 수 있다. 만약 disconnect를 감지하지 않아도 되는 서비스라면 모르겠지만, 우리 서비스에선 disconnect 시에 필수적인 로직이 있어 이 점이 중요했다.

```tsx
public handleDisconnect(client: sleepySocket): void {
    // 다른 유저들에게 해당 유저가 로그아웃 했다는 정보 전달
    // 해당 유저의 그 날의 활동량을 기록
}
```

<br>

# PingInterval과 pingTimeout

---

Nest.js 에서 `SocketGateway`를 구성할때, `@WebSocketGateway` 데코레이터를 통해, WebSocket에 option값을 넣어줄 수 있다. 다음과 같이 `SocketGateway`를 수정했다.

```tsx
@WebSocketGateway({
  pingInterval: 5000,
  pingTimeout: 3000
})
export class SocketGateway implements OnGatewayConnection, OnGatewayDisconnect {
  @WebSocketServer()
  server: Server;

  public handleConnection(client: Socket): void {
    // 클라이언트가 접속되었을 때, 할 일
  }

  public handleDisconnect(client: Socket): void {
    // 클라이언트가 접속 해제되었을 때, 할 일
  }
}
```

위의 두 가지 option들을 통해 heartbeat를 설정할 수 있는데, 이는 connection이 유효한 지 주기적으로 확인하는 과정이라 볼 수 있다.

서버는 `pingInterval`마다 connection을 확인하고, 마지막 요청으로부터 `pingTimeout` 시간 동안 응답이 없다면 클라이언트가 죽은 것으로 판단 해 disconnect 시킨다.
(위의 예시에서는 5초마다 확인을 하며, 마지막 요청에서 3초간 응답이 없었으면 연결이 해제되었다고 판단한다.)

이를 통해 클라이언트가 예기치 못한 접속 종료를 하더라도, 서버에서 이를 잡아낼 수 있다.

아래의 `Socket.io` 도큐먼트에서 조금 더 자세한 정보를 확인할 수 있다.

<br>

# 참조

---

[Server options](https://socket.io/docs/v4/server-options/)

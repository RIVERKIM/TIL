# PM2

생성일: 2021년 8월 30일 오후 9:52

## Node.js 프로세스 매니저 PM2

Node.js 기본적으로 single thread 이다. 따라서 Node.js 애플리케이션은 단일 CPU 코어에서 실행되기 때문에 CPU 멀티코어 시스템은 사용할 수 없다. 

Node.js는 이런 문제를 해결하기 위해 클러스터(cluster) 모듈을 통해 단일 프로세스를 멀티 프로세스로 늘릴 방법을 제공.

따라서 클러스터 모듈을 사용해서 마스터 프로세스에서 CPU 코어 수만큼 워커 프로세스를 생성해서 모든 코어를 사용하도록 개발하면 된다.

애플리케이션 실행시 처음에는 마스터 프로세스만 있다. 이 때 CPU 개수만큼 워커 프로세스를 생성하고 마스터 프로세스와 워커 프로세스가 각각 수행해야 할 일들을 구현해야 한다. 

### PM2 설치

```jsx
npm install -g pm2@latest
```

### PM2 기본 사용 방법

```jsx
//app.js
const express = require('express')
const app = express()
const port = 3000
app.get('/', function (req, res) { 
  res.send('Hello World!')
})
app.listen(port, function () {
  console.log(`application is listening on port ${port}...`)
})
```

위 코드는 아래 명령어로 데몬화(daemonize)하고 모니터링 할 수 있다.

```jsx
pm2 start app.js

[PM2] Spawning PM2 daemon with pm2_home=C:\Users\garam\.pm2
[PM2] PM2 Successfully daemonized
[PM2] Starting C:\Users\garam\OneDrive\Desktop\projectPractice\pm2\app.js in fork_mode (1 instance)ance)
[PM2] Done.
┌────┬────────────────────┬──────────┬──────┬───────────┬──────────┬──────────┐
│ id │ name               │ mode     │ ↺    │ status    │ cpu      │ memory   │
├────┼────────────────────┼──────────┼──────┼───────────┼──────────┼──────────┤
│ 0  │ app                │ fork     │ 0    │ online    │ 0%       │ 35.4mb   │
└────┴────────────────────┴──────────┴──────┴───────────┴──────────┴──────────┘
```

아무 옵션없이 애플리케이션을 실행하면 PM2는 기본 모드인 포크(fork)모드로 애플리케이션을 실행한다. 

모든 CPU를 사용하기 위해서는 애플리케이션을 클러스터 모드로 실행해야 한다. 따라서 아래와 같은 설정파일을 만들어야 한다.

```jsx
//ecosystem.config.js
module.exports = {
  apps: [{
  name: 'app',
  script: './app.js',
  instances: 0, //cpu 코어 수 만큼 프로세스 생성.
  exec_mode: ‘cluster’ // 애플리케이션 실행 모드
  }]
}
```

```jsx
pm2 start pm2.config.js
```

![Untitled](PM2%20ad982e948186442ca60c03c44359954c/Untitled.png)

프로세스 개수를 늘리거나 줄여야 할 때 

```jsx
// 늘리고 싶을 때
pm2 scale app +4
// 줄이고 싶을 때
pm2 scale app 4
```

app.js를 수정한 뒤 수정 사항을 프로세스에 반영하고 싶다면 프로세스를 재시작 해야한다.

```jsx
pm2 reload app
//OR
pm2 start ecosystem.config.js --watch
```

기타 명령어들

```jsx
// 프로세스 상태 표시
pm2 list
// 프로세스 재시작
pm2 restart app
// 프로세스 중지
pm2 stop app
// 프로세스 제거
pm2 delete app

//logs
pm2 logs --lines 200
```

> [참고](https://pm2.keymetrics.io/docs/usage/quick-start/)

### 프로세스 재시작 과정

![Untitled](PM2%20ad982e948186442ca60c03c44359954c/Untitled%201.png)

프로세스가 10개 실행되고 있다고 가정할 때, pm reload 실행시 PM2는 기존 프로세스를 _old_0 프로세스로 옮겨두고 새로운 0번 프로세스를 생성한다.

새로운 프로세스는 요청을 처리할 준비가 되면 'ready' 이벤트를 보내고, 마스터 프로세스는 더 이상 필요없어진 _old_0 프로세스에게 SIGINT 시그널을 보내고 프로세스가 종료되기를 기다립니다. 만약 SIGINT 시그널을 보내고 난 후 일정 시간이 지났는데도 종료되지 않는다면, SIGKILL 시그널을 보내 프로세스를 강제로 종료합니다.  이 과정을 총 프로세스 개수만큼 반복하면 모든 프로세스의 재시작이 완료됩니다.

### 재시작 과정에서 서비스 중단이 발생하는 경우

- **새로 만들어진 프로세스가 실제로는 아직 요청을 받을 준비가 되지 않았는데 ready 이벤트를 보내는 경우.**

![Untitled](PM2%20ad982e948186442ca60c03c44359954c/Untitled%202.png)

프로세스를 생성한 뒤 앱 구동이 완료되기도 전에 마스터 프로세스에게 ready 이벤트를 보낸다면, 마스터 프로세스는 새로운 프로세스가 요청받을 준비가 완료됐다고 판단해버립니다. 이에 기존 프로세스는 더 이상 필요없다고 판단하고 SIGINT를 보내 프로세스에게 곧 종료될 것을 알립니다. 

만약 2번 과정에서 새로운 프로세스를 생성하고 요청 받을 준비를 하는데까지 일정시간(1600ms) 이상 걸리게 된다면 기존 프로세스는 이미 종료된 상태에서 새로운 프로세스는 사용자 요청이 유입돼도 처리할 수 없는 상황이 되어 버린다. 즉 서비스 중단이 발생.

**해결 방안:**

프로세스가 수행될 때 바로 ready 이벤트를 보내지 말고, 애플리케이션 로직에서 요청 받을 준비가 완료된 시점에 ready 이벤트를 보내도록 처리해야 한다. 그리고 마스터 프로세스가 ready 이벤트를 언제까지 기다리게 할 것인지도 설정파일에 명시해야 한다. 

```jsx
//ecosystem.config.js
module.exports ={
    app: [{
        name: 'app',
        scripts: './app.js',
        instances: 0,
        exec_mode: 'cluster',
        wait_ready: true, // 마스터 프로세스에게 ready 이벤트를 기다리도록 요청.
        listen_timeout: 50000//ready 이벤트를 기다릴 시간 값

    }]
}
```

```jsx
//app.js

const express = require('express')
const app = express()
const port = 3000
app.get('/', function (req, res) { 
  res.send('Hello World!')
})
app.listen(port, function () {
  process.send(‘ready’) //앱 실행시 ready 이벤트를 보냄.
  console.log(`application is listening on port ${port}...`)
})
```

![Untitled](PM2%20ad982e948186442ca60c03c44359954c/Untitled%203.png)

- **클라이언트 요청을 처리하는 도중에 프로세스가 죽어버리는 경우**

![Untitled](PM2%20ad982e948186442ca60c03c44359954c/Untitled%204.png)

reload 명령어를 실행할 때, 기존 0번 프로세스인 _old_0 프로세스는 프로세스가 종료되기 전까진 계속해서 사용자 요청을 받는다. 그런데 만약 SIGINT 시그널이 전달된 상태에서 사용자 요청을 받았고,  이 요청 시간이 1600ms를 넘는다면 사용자 요청을 처리하는 도중 SIGKILL 시그널을 받고 사용자에게 응답을 보내주지 못한 채 종료될 테고, 프로세스가 강제 종료되었기 때문에 클라이언트와 연결은 끊어지게 됩니다. 

**해결 방안**

SIGINT 시그널을 리스닝하다가 해당 시그널이 전달되면 app.close 명령어로 프로세스가 새로운 요청을 받는 것을 거절하고 기존 연결은 유지하게 처리합니다. 그리고 사용자 요청을 처리하기에 충분한 시간을 kill_timeout에 설정하고, 기존에 유지되고 있던 연결이 종료되면 프로세스가 종료되도록 처리.

```jsx
//ecosystem.config.js
module.exports = {
  apps: [{
  name: 'app',
  script: './app.js',
  instances: 0,
  exec_mode: ‘cluster’,
  wait_ready: true,
  listen_timeout: 50000,
  kill_timeout: 5000 //SIGINT 시그널을 보낸 후 프로세스가 종료되지 않았을 때 SIGKILL 시그널을 보내기까지 대기시간
  }]
}
```

```jsx
//app.js
const express = require('express')
const app = express()
const port = 3000
app.get('/', function (req, res) { 
  res.send('Hello World!')
})
app.listen(port, function () {
  process.send(‘ready’)
  console.log(`application is listening on port ${port}...`)
})
process.on(‘SIGINT’, function () {
  app.close(function () {
  console.log(‘server closed’)
  process.exit(0)
  })
})
```

> express app.close()는 동작하지 않는다. app.listen()에서 반환된 값에서 호출해야함.

app.close 를 이용해서 새로운 요청을 거절하고 이미 연결되어 있는 건 유지할 때, 만약 HTTP 1.1 Keep-Alive를 사용하고 있따면, 요청이 처리된 후에도 기존 연결이 계속 유지되기 때문에 앞의 방법으로 해결이 안됨.

![Untitled](PM2%20ad982e948186442ca60c03c44359954c/Untitled%205.png)

**해결 방안**

SIGINT 시그널을 받았을 때 특정 전역 플래그값에 따라 응답 헤더에 'Connection:close'를 설정해서 클라이언트 요청을 종료

```jsx
//코드12. 특정 전역 플래그값에 따른 응답 헤더 설정
//app.js
const express = require('express')
const app = express()
const port = 3000
let isDisableKeepAlive = false
app.use(function(req, res, next) {
  if (isDisableKeepAlive) {
    res.set(‘Connection’, ‘close’)
  }
  next()
})
app.get('/', function(req, res) { 
  res.send('Hello World!')
})
app.listen(port, function() {
  process.send(‘ready’)
  console.log(`application is listening on port ${port}...`)
})
process.on(‘SIGINT’, function () {
  isDisableKeepAlive = true
  app.close(function () {
  console.log(‘server closed’)
  process.exit(0)
  })
})
```

> [출처](https://engineering.linecorp.com/ko/blog/pm2-nodejs/)
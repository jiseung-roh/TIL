# 의식의 노드

O'Reilly Media에서 발행한 *Node Up and Running* 책을 읽고 깨달은 바를 정리한 것이다. 의식의 흐름으로 작성하였고, 추측도 함께 있어 오류가 많으며, 속어와 반어도 듬뿍! 담아내었으니 가급적 읽지 않기를 권장한다. (무턱대고 읽었다가 매우 피로해질 수 있음을 알리는 바.)

## 1. A Very Brief Introduction to Node.js

### 노드란 무엇인가
노드가 무엇인가 = 크롬의 _V8 자바스크립트 런타임_을 감싸고 있는 wrapper다. 왜 감싸고 있는가? 크롬 V8엔진은 매우 강려크한 퍼포먼스를 가지고 있지만 브라우저 내에서만 작동한다. (그것도 '크롬' 브라우저에서...) 노드느 이 V8엔진을 감싸면서 엔진이 브라우저 뿐만 아니라 다른 컨텍스트에서도 사용할 수 있게 다양한 API를 제공함(흔히 우리가 아는 `node_modules/`)

> 이게 무슨 말이냐? 이를테면 그런거지. 서버는 가끔 바이너리 데이터를 직접 가공해야 할 때가 있어. 하지만 멍청한 자바스크립트는 이를 할 수가 없지, which means that V8도 할 수 없다는 것. 그런데 노드가 `Buffer`라는 클래스?를 제공해서 자바스크립트로도 바이너리 데이터를 조작할 수 있게 도와줌. 그래서 노드는 웹 서버 환경뿐만 아니라, 라즈베리 파이같은 microcomputers, PC(노드로 런타임 돌리고 있잖아ㅋ) 등 많은 컨텍스트에서 두루두루 사용될 수 있다는 것. **한 마디로 노드는, 자바스크립틀르 매우 강려크하게 만들어 주는 녀석이다.**

### 노드에서의 이벤트
자바스크립트는 `event-driven` 언어다. 모든 것이 이벤트를 기준으로 일어나지. 노드도 이를 표방해서 `event-driven`이야. 노드에서는 특별히 이를 `event loop`라고 이야기하는데, 이런 설계 방식(architecture)을 통해 아주 scalable한 서버 제작을 가능하게 했다.

> 도대체 event loop이랑 scalable한 서버 제작이 가능하다는 것이 무슨 연관이 있는진 모르겠지만, 일단 넘어간다.

`event loop`는 추상적인 개념—아까 말했듯, 설계방식이니까—이고, 노드는 이를 구현하기 위해 `non-blocking libraries(=modules)`를 왕왕 제공한다. 

> 대체 그럼 `Non-blocking`은 또 뭐냐? 아까의 예를 다시 가져오자. 서버에서 바이너리 파일을 수정해야 해. "노드가 `blocking`하다는 것"은 노드가 직접 하드를 뒤져 파일을 하나하나 들쳐내고, 고치고... 뭐 그런다는 뜻이다. 하지만 노드는 노노노~ `Non-blocking`하다. 즉, 노드는 자신이 직접 그런 일들을 수행하는 게 아니라, `fs`같은 모듈에게 대신 시킨다. `fs`는 노드가 시킨 일을 마친 후 노드를 **두들긴다(=이벤트 발생)**. 그럼 노드는 그 결과값을 받아서 Next move로 나아가는 것. 노드가 알아야하는 것은 **1) 결과가 발생한 시점(이벤트 발생 시점)과 2) 결과**다.

### Node REPL
노드는 서버로 사용할 수도 있지만 자바스크립트 런타임으로도 사용가능하다. 이를 REFL이라고 하는 것 같더라. 

### 퍼포먼스를 위해서라면 물불 가리지 않는 노드
노드는 Scalable한 서버 환경 제공을 목표로 한다. 노드는 V8 런타임을 사용해서 자바스크립트를 실행시키기도 하지만, 더 파워풀한 퍼포먼스를 위해 다른 언어로 만들어진(?) 라이브러리도 적극적으로 모듈로 포함시켰다.

> 노드의 기본 제공 모듈인 `http` 모듈을 예로 들자. `http` 통신을 할 수 있는 서버를 만들어주는 등 다양한 역할을 하는 `http` 모듈은 자바스크립트로 만드는 것보다 C로 만드는 것이 더 퍼포먼스가 좋았다고 한다. 그럼 C로 짜즈아!

### 예시

```javascript
const http = require('http')
// http 존잘템을 긁어서 변수 http에 담았다.

http.createServer((req, res) => {
  // http.createServer는 팩토리 함수다.
  // 이 아이가 실행이 되면 HTTP 서버가 만들어지지. (인스턴스)
  // 얘는 전역을 떠도는 object이 될거야 (레퍼런스 지정 안해줌)
  // 그래서 chaining으로 바로 listen 시켜줌
  // 암튼, 그런데 얘가 서버를 만들기 위해선 반드시 필요한 argument가 있음. 바로 콜백함수
  // 이 콜백 함수는 서버 인스턴스의 Request event listener에 assign될거야.
  // 그래서 요청(request)이 발생하면 이 함수를 빠방 실행시키는 것이지 (=event loop)
  res.writeHead(200, {
    'Content-Type': 'text/plain'
  })
  // 응답(response)에는 반드시 HTTP Head를 작성해야 함.
  // 아님 브라우저에서 못 받음
  res.end('Hello World\n')
  // HTTP Body에 헬로월드 쓰고
  // 연결 종료 끄읕!
}).listen(8124)
```

## 2. Doing Interesting Stuff

```javascript
const net = require('net')

const chatServer =  net.createServer()

chatServer.on('connection', (client) => {
  client.write('Hi!\n')
  client.end()
})

chatServer.listen(8124)
```

```javascript
const net = require('net')

const chatServer = net.createServer()
const clientList = []

chatServer.on('connection', (client) => {
  client.name = client.requestPort
  clientList.push(client)

  client.on('data', (data) => {
    clientList.forEach((item, index) => {
      if (client !== item) {
        client.write(data)
      }
    })
  })
})

chatServer.listen(8124)
```

```javascript
const net = require('net')

const chatServer = net.createServer()
const clientList = []

const broadcast = (msg, currentClient) => {
  clientList.forEach((client, index) => {
    if (currentClient !== client) {
      client.write(data)
    }
  })
}

chatServer.on('connection', (client) => {
  client.name =  `${client.remoteAddress}:${client.remotePort}`
  clientList.push(client)

  client.on('data', (data) => {
    broadcast(data, client)
  })

  client.on('end', () => {
    clientList.splice(clientList.indexOf(client), 1)
  })
})
```

```javascript
const net = require('net')
const chatServer = net.createServer()
const clientList = []

const broadcast = (msg, currentClient) => {
  const cleanup = []
  clientList.forEach((client, index) => {
    if (!client.writable) {
      cleanup.push(client)
      client.destroy()
    } else {
      if (client !== currentClient) {
        client.write(msg)
      }
    }
  })

  cleanup.forEach((item, index) => {
    clientList.splice(clientList.indexOf(item), 1)
  })
}

chatServer.on('connection', (client) => {
  client.name = `${client.remoteAddress}:${client.remotePort}`
  clientList.push(client)

  client.on('data', (data) => {
    broadcast(data, client)
  }) 
})
```


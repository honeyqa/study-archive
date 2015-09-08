# zookeeper-node.js study

#소개

분산 시스템을 설계 하다보면, 가장 문제점 중의 하나가 분산된 시스템간의 정보를 어떻게 공유할것이고, 클러스터에 있는 서버들의 상태를 체크할 필요가 있으며 또한, 분산된 서버들간에 동기화를 위한 락(lock)을 처리하는 것들이 문제로 부딪힌다.

이러한 문제를 해결하는 시스템을 코디네이션 서비스 시스템 (coordination service)라고 하는데, Apache Zookeeper가 대표적이다. 이 코디네이션 서비스는 분산 시스템 내에서 중요한 상태 정보나 설정 정보등을 유지하기 때문에, 코디네이션 서비스의 장애는 전체 시스템의 장애를 유발하기 때문에, 이중화등을 통하여 고가용성을 제공해야 한다. ZooKeeper는 이러한 특성을 잘 제공하고 있는데, 그런 이유로 이미 유명한 분산 솔루션에 많이 사용되고 있다. NoSQL의 한종류인 Apache HBase, 대용량 분산 큐 시스템인 Kafka등이 그 대표적인 사례이다.
분산 시스템을 코디네이션 하는 용도로 디자인이 되었기 때문에, 데이타 억세스가 빨라야 하며, 자체적으로 장애에 대한 대응성을 가져야 한다. 그래서 Zookeeper는 자체적으로 클러스터링을 제공하며, 장애에도 데이타 유실 없이 fail over/fail back이 가능하다.

Apache Zookeeper의 기능은 사실상 별로 없다. 디렉토리 구조기반으로 znode라는 데이타 저장 객체를 제공하고, (key-value식). 이 객체에 데이타를 넣고 빼는 기능만을 제공한다. 일단 디렉토리 형식을 사용하기 때문에 데이타를 계층화된 구조로 저장하기 용이하다.

#주키퍼의 특징
네임서비스를 통한 부하분산 
하나의 클라이언트(하나의 서버)만 서비스를 수행하지 않고 알맞게 분산하여 각각의 클라이언트들이 동시 작업할 수 있도록 지원
분산락이나 동화 문제 해결 
하나의 서버에서 처리된 결과가 또 다른 서버들과 동기화하여 데이터 안정성 보장
장애상황 판단 및 복구 
액티브(일반서버)서버가 예기치 못한 상황으로 문제가 발생하여 서비스를 지속적으로 처리를 못 할 경우 스탠바이(일반서버)서버가 액티브 서버로 바뀌어서 기존에 액티브 서버가 서비스를 하던 일을 처리하게 된다.
환경설정 관리 
각각의 다른 서버들을 통합적으로 관리하여 환경설정을 따로 분산하지 않고 주키퍼 자체적으로 관리하게 된다.

#주키퍼는 왜 필요한가
분산처리 환경에서는 기본적으로 서버가 몇 대에서 수십 대, 수백 대까지도 갈수도 있습니다. 이런 분산처리 환경에서는 예상치 못하는 예외적인 부분이 많이 발생하게 되 는대 주로 네트워크장애, 일부 서비스/기능 예상치 못한 처리로 중지나 장애, 서비스 업그레이드, 서버 확장 등에 문제가 발생할 수 있습니다. 
쉽게 이해가 안되시면 프로그램적으로 생각하셔도 됩니다. 예로 싱글 쓰레드만 존재하는 프로그램에서 멀티 쓰레드 프로그램을 하게 될 경우 싱글 쓰레드에서 이상없던 동기성 문제등이 나타나기 시작합니다. 분산처리도 마찬가지입니다. 하나만 처리하던 싱글 서버에서는 문제가 되지 않으나 멀티 서버를 관리를 해야한다면 여러 문제점들이 발생 할 수 있습니다. 
 따라서 이런한 점들을 쉽게 해결 할 수 있는 시스템이 바로 주키퍼입니다.

#데이타 모델
<center>![ig](http://cfile4.uf.tistory.com/image/212C9A41552A787B2C67D4)</center><br>
데이타 모델은 간단하다. 디렉토리 구조의 각 노드에 데이타를 저장할 수 있다.

노드는 아래와 같이 기능에 따라 몇가지 종류로 나뉜다.

Persistent Node : 노드에 데이타를 저장하면 일부러 삭제하지 않는 이상 삭제되지 않고 영구히 저장된다.
Ephemeral Node : 노드를 생성한 클라이언트의 세션이 연결되어 있을 경우만 유효하다. 즉 클라이언트 연결이 끊어지는 순간 삭제 된다. 이를 통해서 클라이언트가 연결이 되어 있는지 아닌지를 판단하는데 사용할 수 있다. (클러스터를 구성할때 클러스터내에 서버가 들어오면, 이 Ephemeral Node로 등록하면 된다.)
Sequence Node : 노드를 생성할때 자동으로 sequence 번호가 붙는 노드이다. 주로 분산락을 구현하는데 이용된다.

#Watcher

Watch 기능은 ZooKeeper 클라이언트가 특정 znode에 watch를 걸어놓으면, 해당 znode가 변경이 되었을때, 클라이언트로 callback 호출을 날려서 클라이언트에 해당 znode가 변경이 되었음을 알려준다. 그리고 해당 watcher는 삭제 된다. 

ZooKeeper 활용 시나리오

Zookeeper는 단순히, 디렉토리 형태의 데이타 저장소이지만, 노드의 종류별 특성과, Watcher 기능들을 활용하면 다양한 시나리오에 사용할 수 있다. 

큐 : Watcher와 Sequence node를 이용하면, 큐를 구현할 수 있는데, Queue 라는 Node를 만든 후에, 이 노드의 Child node를 sequence node로 구성하면, 새롭게 생성되는 메세지들은 이 sequence node로 순차적으로 생성된다. 이 큐를 읽는 클라이언트가 이 큐 node를 watch 하도록 설정하면, 이 클라이언트는 새로운 메세지가 들어올 때 마다 call back을 받아서, 마치 메세지 Queue의 pub/sub 과 같은 형태의 효과를 낼 수 있다. 대용량 메세지나 애플리케이션 성격상의 메세지는 일반적인 큐 솔루션인 Rabbit MQ등을 활용하고, ZooKeeper는 클러스터간 통신용 큐로 활용하는 것을 고려해볼 수 있다.
서버 설정 정보 : 가장 일반적인 용도로는 클러스터 내의 각 서버들의 설정 정보(Configuration)를 저장하는 저장소로 쓸 수 있다. 정보가 안정적으로 저장이 될 뿐 아니라. Watch 기능을 이용하면, 설정 정보가 저장될 경우, 각 서버로 알려서 바로 반영을 할 수 있다.
클러스터 정보 : 현재 클러스터에서 기동중인 서버 목록을 유지할 수 있다. Ephemeral Node는 Zookeeper 클라이언트가 살아 있을 경우에만 유효하기 때문에, 클러스터내의 각 서버가 Ephemeral Node를 등록하도록 하면, 해당 서버가 죽으면 Ephemeral Node 가 삭제 되기 때문에 클러스터 내의 살아 있는 Node 리스트만 유지할 수 있다. 조금 더 발전하면, 클러스터가 master/slave 구조일때, master 서버가 죽었을 때, 다른 master 서버를 선출하는 election logic 도 구현이 가능하다. (https://zookeeper.apache.org/doc/r3.4.6/recipes.html#sc_recipes_Queues 참고)
글로벌 락 : Zookeeper를 이용하여 많이 사용되는 시나리오 중의 하나인데, 여러개의 서버로 구성된 분산 서버가 공유 자원을 접근하려고 했을때, 동시에 하나의 작업만이 발생해야 한다고 할때, 그 작업에 Lock을 걸고 작업을 할 수 있는 기능을 구현할때 사용한다. 자바 멀티 쓰레드 프로그램에서 synchronized를 생각하면 쉽게 이해가 가능할 것이다.
자세한 구현 방식과 시나리오에 대해서는 https://zookeeper.apache.org/doc/r3.4.6/recipes.html 를 참고하기 바란다.

[Node.js](http://nodejs.org) 관리를 위한 자바스크립트로 작성된 [ZooKeeper](http://zookeeper.apache.org) 클라이언드 모듈

ZooKeeper version 3.4.*과 호환됩니다

---

## app.js

Zookeeper clustering, Node.js 코디네이션 모듈

## 예제

1\. 다음과 같은 방법으로 node를 생성합니다:

```javascript
var zookeeper = require('node-zookeeper-client');

var client = zookeeper.createClient('localhost:2181');
var path = process.argv[2];

client.once('connected', function () {
    console.log('Connected to the server.');

    client.create(path, function (error) {
        if (error) {
            console.log('Failed to create node: %s due to: %s.', path, error);
        } else {
            console.log('Node: %s is successfully created.', path);
        }

        client.close();
    });
});

client.connect();
```

2\. node의 자식 프로세스를 리스트하고 감시합니다

```javascript
var zookeeper = require('node-zookeeper-client');

var client = zookeeper.createClient('localhost:2181');
var path = process.argv[2];

function listChildren(client, path) {
    client.getChildren(
        path,
        function (event) {
            console.log('Got watcher event: %s', event);
            listChildren(client, path);
        },
        function (error, children, stat) {
            if (error) {
                console.log(
                    'Failed to list children of %s due to: %s.',
                    path,
                    error
                );
                return;
            }

            console.log('Children of %s are: %j.', path, children);
            
        }
    );
}

client.once('connected', function () {
    console.log('Connected to ZooKeeper.');
    listChildren(client, path);
});

client.connect();
```

### 클라이언트 생성 - createClient(connectionString, [options])

Factory method를 이용하여 새로운new zookeeper [client](#client) instance를 생성합니다

---

### 클라이언트

ZooKeeper 클라이언트 모듈의 메인 클래스입니다. 앱은 반드시 [`createClient`] 메소드를 활용하여 클라이언트를 초기화합니다.
클라이언트로부터 서버까지의 연결이 수립된 이후, 세션 아이디가 클라이언트로 부여됩니다.
그 후, 클라이언트는 주기적으로 서버로 하트비트를 전송하기 시작하여 세션을 유지하도록 합니다.

만약 클라이언트가 서버로 긴 시간동안 하트비트를 보내지 못한다면, 서버는 세션을 파기합니다.
파기되면 클라이언트 객체는 더이상 사용되지 않습니다.

만약 ZooKeeper 서버가 클라이언트로부터 연결 실패되거나 응답이 없다면, 클라이언트는 자동으로 다른 서버로 연결해서 세션 타임아웃을 방지합니다. 성공적으로 연결되면, 애플리케이션은 클라이언트를 계속해서 사용합니다.

---

**예제**

```javascript
zookeeper.create(
    '/test/demo',
    new Buffer('data'),
    CreateMode.EPHEMERAL,
    function (error, path) {
        if (error) {
            console.log(error.stack);
            return;
        }

        console.log('Node: %s is created.', path);
    }
);
```

---
### Zookeeper 클러스터링
#### Zookeeper 서버, 클라이언트 실행파일 및 설정정보
주키퍼는 3개의 서버로 클러스터링 되어 실행됩니다.<br>
.
```
Cluster1: ./zoo/zoo_cluster/zookeeper1/zookeeper-3.4.6/bin/zkServer.sh
Cluster2: ./zoo/zoo_cluster/zookeeper2/zookeeper-3.4.6/bin/zkServer.sh
Cluster3: ./zoo/zoo_cluster/zookeeper3/zookeeper-3.4.6/bin/zkServer.sh
```
```

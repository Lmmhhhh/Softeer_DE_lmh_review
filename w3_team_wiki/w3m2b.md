# W3M2b - Understanding of Hadoop Configuration Files
1팀 팀원 - 이승연, 이민하, 박민서, 홍준성

### 이승연

### 중요하다고 생각하는 설정

- `core-site.xml`의 `fs.defaultFS`
- `yarn-site.xml`의 `yarn.resourcemanager.hostname`

### 용도

#### `fs.defaultFS`

클러스터의 기본 파일시스템 주소를 지정한다. `hdfs://namenode:9000` 형태로 설정하면 모든 노드가 이 주소를 기준으로 NameNode를 찾아서 통신한다.

#### `yarn.resourcemanager.hostname`

ResourceManager가 바인딩할 호스트 주소를 지정한다. 이 값을 기준으로 NodeManager들이 ResourceManager를 찾아서 자원 정보를 등록한다.

### 중요하다고 생각하는 이유

두 설정 모두 직접 겪었는데, `fs.defaultFS`가 `hadoop-master`로 되어 있으면 NameNode 자체가 실행되지 않아 클러스터가 동작하지 않았다. `yarn.resourcemanager.hostname`도 마찬가지로 `hadoop-master` 그대로 두면 ResourceManager가 `Unresolved address` 에러로 바로 종료되었다.

두 설정 모두 컨테이너의 실제 hostname과 일치해야 한다. 이 값이 잘못되면 다른 설정이 아무리 잘되어 있어도 클러스터가 동작하지 않는다는 점에서 가장 근본적인 설정이라고 생각한다.

---

### 이민하

### 중요하다고 생각하는 설정

- `core-site.xml`의 `fs.defaultFS`
- `hdfs-site.xml`의 `dfs.replication`
- `mapred-site.xml`의 `mapreduce.framework.name`

### 용도

#### `fs.defaultFS`

Hadoop 클러스터가 기본으로 사용할 파일 시스템의 주소를 지정한다.

`hdfs://namenode:9000`으로 설정하면 HDFS 명령과 MapReduce 작업이 해당 주소를 기준으로 NameNode에 접근한다.

#### `dfs.replication`

HDFS에 저장되는 데이터 블록의 기본 복제 개수를 지정한다.

값을 `2`로 설정하면 하나의 블록이 서로 다른 DataNode 두 곳에 저장되며, 하나의 DataNode에 장애가 발생해도 다른 복제본을 통해 데이터를 사용할 수 있다.

#### `mapreduce.framework.name`

MapReduce 작업을 어떤 실행 환경에서 수행할지 지정한다.

값을 `yarn`으로 설정하면 MapReduce 작업이 YARN의 자원 관리 아래에서 분산 실행되고, `local`로 설정하면 단일 JVM에서 로컬 작업으로 실행된다.

### 중요하다고 생각하는 이유

`fs.defaultFS`는 클라이언트와 Hadoop 구성 요소가 NameNode의 위치를 찾기 위해 사용하는 기본 주소이다. 이 값이 실제 컨테이너의 hostname과 일치하지 않으면 HDFS에 접근할 수 없어 클러스터의 기본적인 파일 저장과 조회가 불가능하다.

`dfs.replication`은 Hadoop의 핵심 특징인 장애 대응과 직접적으로 관련된 설정이다. 데이터가 하나의 노드에만 저장되어 있으면 해당 노드에 장애가 발생했을 때 데이터를 사용할 수 없지만, 복제본을 여러 노드에 저장하면 다른 노드의 데이터를 통해 복구할 수 있다.

`mapreduce.framework.name`은 MapReduce 작업이 실제 분산 환경에서 실행되는지를 결정한다. 이 값을 `yarn`으로 설정하지 않으면 YARN 클러스터가 정상적으로 구성되어 있어도 MapReduce 작업이 로컬 모드로 실행될 수 있다.

따라서 HDFS에 데이터를 분산 저장하고 YARN을 통해 작업을 분산 처리하기 위해서는 세 설정이 모두 올바르게 구성되어야 한다고 생각하였다.

---

### 박민서

### 1. `core-site.xml`

#### `fs.defaultFS`

##### 사용법

- Hadoop 클러스터가 기본으로 참조할 파일 시스템 주소 URI를 지정
- `hdfs://<namenode-host>:<port>` 형식
- 예: `hdfs://namenode:9000`
- 이 값을 기준으로 모든 HDFS 명령과 클라이언트 요청이 해당 NameNode로 라우팅됨
- 실제 컨테이너 또는 호스트명과 정확히 일치시켜야 하는 값

##### 중요한 이유

- Hadoop 전체가 참조하는 클러스터의 진입점이기 때문
- 이 값이 없으면 클라이언트가 어디로 접근해야 할지 모르고, 클러스터 전체가 파일 시스템 위에서 작동하지 않게 됨
- 값이 틀리면 클러스터 자체가 실행되지 않음
- `hadoop-master`를 `namenode`로 바꾸지 않아 초반에 계속 실패했던 실습 경험이 이를 직접 보여줌

#### 그 외 언급된 항목

- `hadoop.tmp.dir`: Hadoop 실행 중 생성되는 임시 파일 저장 경로. 안정적인 실행 환경을 위해 지정 필요
- `io.file.buffer.size`: 파일 읽기·쓰기 버퍼 크기를 기본값 `4096`의 32배인 `131072`, 즉 128KB로 늘려 대용량 데이터 처리 성능 향상

### 2. `hdfs-site.xml`

#### `dfs.replication`

##### 사용법

- 하나의 데이터 블록을 몇 개의 DataNode에 복제해 저장할지 지정하는 정수값
- DataNode 개수와 맞물려야 의미가 있어, 실습에서는 worker 2대에 맞춰 `2`로 설정

##### 중요한 이유

- 하나의 노드가 종료되어도 다른 복제본에서 복구할 수 있는, Hadoop의 가장 기본적이고 중요한 개념이기 때문
- 값을 `2` 이상으로 설정하면 장애로 파일 접근이 안 될 때 다른 노드의 복제본으로 접근할 수 있음
- 복제 수가 늘수록 안정성은 높아지지만 저장 공간을 더 사용하게 되는 트레이드오프가 있음
- 실습에서는 `worker1`과 `worker2`에 각각 복제본을 두는 방식으로 확인

#### `dfs.namenode.name.dir`

##### 사용법

- NameNode가 파일 시스템 메타데이터인 `fsimage`, `edits`를 저장하는 로컬 경로
- `hdfs namenode -format` 실행 시 이 경로에 메타데이터가 생성되며, 이후 클러스터의 원본 장부 역할을 함

##### 중요한 이유

- 이 경로의 파일이 손상되면 클러스터 전체에 문제가 발생할 수 있는 HDFS 운영의 필수 항목이기 때문
- 경로를 중간에 바꾸면 포맷을 다시 해야 하고, 그 과정에서 `clusterID` 불일치로 DataNode가 종료되는 문제를 실제로 겪어 함부로 바꾸면 안 되는 설정임을 체감

#### `dfs.blocksize`

##### 사용법

- 파일을 나누어 저장하는 블록 단위 크기 지정
- 일반적으로 128MB 사용

##### 중요한 이유

- 블록 크기는 대용량 파일의 저장 방식과 처리 성능에 직접적인 영향을 주기 때문

### 3. `mapred-site.xml`

#### `mapreduce.framework.name`

##### 사용법

- MapReduce 작업을 실행할 프레임워크 지정
- 설정값은 `yarn` 또는 `local`
- `yarn`으로 설정하면 자원 관리를 YARN에 위임하며 `yarn-site.xml` 설정값들과 연결되어 동작
- `local`은 단일 JVM에서 실행되는 개발·디버깅용 모드

##### 중요한 이유

- 이 값이 `yarn`이 아니면 YARN을 설정해도 MapReduce가 로컬에서 실행될 수 있어 분산 처리 여부를 결정하는 핵심 설정이기 때문
- `yarn`으로 설정하면 MapReduce 작업을 YARN 자원 관리자에게 맡기게 되므로 다른 `yarn-site.xml` 설정과 실질적으로 연결되는 지점이기 때문

#### `mapreduce.task.io.sort.mb`

##### 사용법

- Map 단계의 출력 데이터를 정렬할 때 사용하는 메모리 버퍼 크기 지정

##### 중요한 이유

- 값을 크게 설정하면 디스크로 spill되는 데이터가 줄어 디스크 I/O가 감소하고, Shuffle 단계의 속도가 빨라지는 등 작업 성능에 직접 영향을 주는 튜닝 항목이기 때문

#### 그 외 언급된 항목

- `mapreduce.jobhistory.address`: 완료된 작업의 실행 기록을 관리하는 JobHistory Server 주소로, 결과 확인과 디버깅에 활용

### 4. `yarn-site.xml`

#### `yarn.resourcemanager.address` / `yarn.resourcemanager.hostname`

##### 사용법

- 클러스터 전체 자원을 관리하는 ResourceManager가 위치한 호스트를 지정
- `hostname`은 호스트명만 지정하는 축약형
- `address`는 `host:port` 형태로 기본 포트 `8032`까지 명시한 클라이언트 IPC 엔드포인트

##### 중요한 이유

- 작업 제출과 상태 조회 시 클라이언트가 접근하는 주소이자 YARN 클러스터의 진입점이기 때문
- 모든 NodeManager가 이 서버와 통신해야 하기 때문
- 컨테이너 호스트명을 `hadoop-master`로 그대로 두면 ResourceManager가 실행되지 않는 문제를 실제로 겪어, `fs.defaultFS`와 마찬가지로 호스트명 일치가 중요함을 확인

#### `yarn.nodemanager.resource.memory-mb`

##### 사용법

- 각 NodeManager가 컨테이너 운영에 할당할 수 있는 메모리 총량 지정
- 클러스터 전체 자원 총량은 이 값에 worker 노드 수를 곱하여 계산

##### 중요한 이유

- 각 노드가 사용할 수 있는 자원의 총량을 정하고, 자원을 효율적으로 관리하는 기준값이기 때문

#### `yarn.scheduler.minimum-allocation-mb`

##### 사용법

- 컨테이너 하나에 할당할 수 있는 최소 메모리 크기 지정
- 다음 계산을 통해 노드당 동시에 실행할 수 있는 컨테이너 개수를 구할 수 있음

```text
yarn.nodemanager.resource.memory-mb
÷ yarn.scheduler.minimum-allocation-mb
= 노드당 동시 처리 가능한 컨테이너 개수
````

##### 중요한 이유

* 컨테이너의 최소 자원 할당 단위를 결정하며, 동시에 실행할 수 있는 컨테이너 개수와 자원 활용 효율에 직접적인 영향을 주기 때문

#### 그 외 언급된 항목

* `yarn.nodemanager.aux-services`: `mapreduce_shuffle`로 설정해야 MapReduce의 Shuffle 기능이 정상적으로 동작함

---

### 홍준성

### 중요하다고 생각하는 설정

* `hdfs-site.xml`의 `dfs.replication`
* `mapred-site.xml`의 `mapreduce.framework.name`

### 용도

#### `dfs.replication`

HDFS에 저장되는 데이터 블록을 몇 개의 DataNode에 복제할지 지정한다. 예를 들어 값을 `2`로 설정하면 동일한 블록이 서로 다른 두 DataNode에 저장된다.

#### `mapreduce.framework.name`

MapReduce 작업을 어떤 환경에서 실행할지 지정한다. 값을 `yarn`으로 설정하면 ResourceManager와 NodeManager가 작업에 필요한 자원을 할당하고 실행 과정을 관리한다.

### 중요하다고 생각하는 이유

`dfs.replication`은 데이터의 안정성과 저장 공간 사용량에 직접적인 영향을 주기 때문에 중요하다고 생각한다. 복제 수가 너무 적으면 DataNode에 문제가 발생했을 때 데이터를 잃을 수 있고, 반대로 너무 많으면 불필요하게 저장 공간을 많이 사용하게 된다.

특히 DataNode가 두 개인 실습 환경에서는 복제 수를 노드 수보다 크게 설정하지 않도록 주의해야 한다.

`mapreduce.framework.name`은 MapReduce 작업의 실행 방식을 결정하는 설정이다. 이 값이 `yarn`으로 설정되어 있지 않으면 구축한 YARN 클러스터의 자원을 활용하지 못하거나 작업이 예상한 방식으로 실행되지 않을 수 있다.

따라서, 이 설정들이 HDFS에 데이터를 저장하는 것에서 끝나지 않고 실제로 분산 처리 작업을 실행하기 위해 필요한 핵심 설정이라고 생각한다.

## 팀 내 의견 종합

팀원들의 의견을 종합한 결과, Hadoop 설정은 크게 클러스터 연결, ​데이터 안정성, ​분산 처리 실행, ​자원 관리의 네 가지 측면에서 중요하다고 정리할 수 있었다.

`fs.defaultFS`와 `yarn.resourcemanager.hostname`은 각 노드가 NameNode와 ResourceManager의 위치를 찾기 위한 가장 기본적인 설정이며, 실제 hostname과 일치하지 않으면 클러스터 자체가 정상적으로 실행되지 않는다. dfs.replication은 데이터를 여러 DataNode에 복제하여 장애 발생 시에도 데이터를 사용할 수 있도록 하는 설정이고, `mapreduce.framework.name`은 MapReduce 작업을 로컬이 아닌 YARN 환경에서 분산 실행하도록 결정한다.

또한 `yarn.nodemanager.resource.memory-mb`, `yarn.scheduler.minimum-allocation-mb`, `mapreduce.task.io.sort.mb`와 같은 설정은 클러스터가 정상적으로 실행된 이후 자원을 효율적으로 사용하고 작업 성능을 조절하는 데 중요한 역할을 한다.

결과적으로 Hadoop 설정에서는 먼저 각 서비스가 서로 올바르게 연결되도록 주소와 hostname을 정확하게 지정해야 하며, 이후 복제 수와 메모리 등의 설정을 클러스터 규모와 실행 환경에 맞게 조정하는 것이 중요하다는 의견으로 정리되었다.

## 1\. 자바 설치

엘라스틱서치는 자바 언어로 개발된 프로그램이므로 구동을 시키기 위해 자바 환경이 설치되어 있어야 합니다. 자바 설치 방법은 많은 곳에서 쉽게 찾아 볼 수 있으므로 본 포스트에서는 따로 다루지 않겠습니다.

## 2\. 엘라스틱서치 설치

엘라스틱서치는 공식 홈페이지에서 내려받을 수 있습니다. (아래 링크)

[다운로드 링크](https://www.elastic.co/kr/downloads/elasticsearch?baymax=KR-ES-getting-started&elektra=landing-page)

[설치 가이드 링크](https://www.elastic.co/guide/en/elasticsearch/reference/current/getting-started-install.html)

내려받은 파일의 압축을 풀면 bin 디렉토리에 elasticsearch 파일이 있습니다. 윈도우 환경이라면 elasticesarch.bat 파일이 있을 것입니다. 저는 MacOS 환경이므로 아래와 같이 명령어를 실행하면 간단하게 엘라스틱서치를 실행시킬 수 있습니다.

```
./elasticsearch
```

엘라스틱서치의 기본포트는 9200 이고, 웹브라우저에서 [http://localhost:9200](http://localhost:9200) 에 접속해보면 아래와 같이 엘라스틱서치의 클러스터 이름과 버전 정보가 나타날 것입니다.

[실행화면](./images/ex.png)

위 화면은 제가 실습을 위해 미리 설정 파일을 수정한 상태이므로 처음 설치 후 구동하였다면 출력되는 정보가 저와는 다르게 나올 것입니다.

## 엘라스틱서치 디렉토리 구조

#### bin

자바 실행 변수 파일과 엘라스틱서치 실행 파일, 플러그인 설치 프로그램이 있습니다.

#### config

엘라스틱서치 실행 환경 및 로그 설정 파일이 있습니다.

#### data

데이터 저장 공간입니다. 클러스터 별로 하위 디렉토리가 생성됩니다.

#### lib

엘라스틱서치 자바 라이브러리 파일이 있습니다.

#### logs

실행 로그와 슬로우 로그 파일이 있습니다.

---

## 주요 설정 항목

엘라스틱서치는 설치 디렉토리의 config 디렉토리에 있는 elasticsearch.yml 파일을 수정하여 변경할 수 있습니다.

#### cluster.name

클러스터로 여러 노드를 하나로 묶을 수 있는데, 여기서 클러스터명을 지정할 수 있습니다. (기본값은 "elasticsearch" 입니다.)

#### node.name

엘라스틱서치 노드의 이름을 설정합니다.

#### path.data

엘라스틱서치의 인덱스 경로를 지정합니다. 설정하지 않으면 기본적으로 에랄스틱서치 하위의 data 디렉토리에 인덱스가 생성됩니다.

#### path.logs

엘라스틱서치의 노드와 클러스터에서 생성되는 로그를 저장할 경로를 지정합니다. 기본 경로는 /path/to/logs 입니다.

#### path.repo

엘라스틱서치 인덱스를 백업하기 위한 스냅샷의 경로를 지정합니다.

#### network.host

특정 IP만 엘라스틱서치에 접근하도록 허용할 수 있습니다. 선택적으로 허용해야 할 경우 \[1.1.1.1, 2.2.2.2\]와 같이 지정하고, 모든 IP를 허용한다면 0.0.0.0을 지정하면 됩니다.

#### http.port

엘라스틱서치 서버에 접근할 수 있는 HTTP API 호출을 위한 포트를 지정합니다. 기본 값은 9200 입니다.

#### transport.tcp.port

엘라스틱서치 노드들 간에 통신을 하기 위해 접근할 수 있는 TCP 포트를 설정합니다. 기본 값은 9300 입니다.

#### node.master

마스터 후보 노드 여부를 설정합니다. false인 경우 이 노드는 마스터 노드로 선출이 불가능합니다. 모든 클러스터는 1개의 마스터 노드가 존재하며 마스터 노드가 다운되거나 끊어진 경우 남은 마스터 후보 노드들 중에서 새로운 마스터 노드를 선출합니다.

#### node.data

노드가 데이터를 저장하도록 지정합니다. false인 경우 이 노드는 데이터를 저장하지 않습니다.

#### bootstrap.memory\_lock

엘라스틱서치가 사용중인 힙메모리 영역이 다른 자바 프로그램이 간섭하지 못하도록 미리 점유하는 설정입니다. 항상 true로 놓고 사용하는 것을 권장합니다.

---

## 참고자료

[https://esbook.kimjmin.net/02-install/2.3-elasticsearch/2.3.2-elasticsearch.yml](https://esbook.kimjmin.net/02-install/2.3-elasticsearch/2.3.2-elasticsearch.yml)

[엘라스틱서치 실무 가이드](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791158391485&orderClick=LEa&Kc=) <<권택환, 김동우, 김흥래, 박진현, 최용호, 황희정 지음>>
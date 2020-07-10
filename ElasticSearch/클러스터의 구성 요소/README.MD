# 클러스터의 구성 요소

 엘라스틱서치는 기본적으로 클러스터라는 단위로 데이터를 제공합니다. 클러스터는 하나 이상의 물리적인 노드로 이뤄져 있으며 각 노드는 모두 데이터 색인 및 검색 기능을 제공하는 일종의 물리적인 서버이기도 합니다. **내부에는 루씬 라이브러리가 자리 잡고 있으며 이는 엘라스틱서치의 근간을 이루는 핵심 모듈입니다.**

## 클러스터 (Cluster)

 클러스터는 데이터를 실제로 가지고 있는 노드의 모음입니다. 엘라스틱서치에서는 관련된 모든 노드들을 논리적으로 확장해서 클러스터라고 부릅니다. 같은 클러스터 내부의 데이터만 서로 공유가 가능하기 때문에 연관된 노드들이 하나의 클러스터로 연결되도록 설정을 해주어야 합니다. 이를 위해 **같은 클러스터를 구성하는 노드들은 같은 클러스터 이름으로 설정해야 합니다.** 클러스터 내부의 노드들은 평소 데이터를 색인이나 검색 작업을 함께 수행하게 되고 장애가 발생했을 때 데이터 복구를 위한 다양한 작업도 서로 협력해서 함께 진행합니다.

## 노드 (Node)

 물리적으로 실행된 런타임 상태의 엘라스틱서치를 노드라고 부릅니다. 노드는 논리적인 클러스터를 이루는 구성원의 일부이며 실제 데이터를 물리적으로 가지고 있는 단일 서버이기도 합니다. 실행 시 노드는 클러스터에 의해 UUID(Universally Unique IDentifier)가 할당되고 클러스터 내에서는 할당된 UUID로 서소를 식별합니다. 기본값으로 부여되는 UUID를 원하지 않는다면 직접 원하는 이름을 설정할 수도 있습니다.

 같은 클러스터 내부에 존재하는 모든 노드는 서로 다른 노드와 수시로 정보를 주고받습니다. 노드는 목적과 역할에 따라 다음과 같은 형태로 실행할 수 있습니다.

-   마스터 노드(Master Node) : node.master 설정이 true로 설정된 노드이며 클러스터의 제어를 담당합니다.
-   데이터 노드(Data Node) : node.data 설정이 true로 설정된 노드입니다. 데이터를 저장하고, CRUD, 검색, 집계 등 데이터 관련 작업을 담당합니다.
-   인제스트 노드(Ingest Node) : node.ingest가 true로 설정된 노드입니다. 색인 전 전처리 작업을 담당합니다.
-   코디네이팅 노드(Coordinating Node) : 검색이나 집계 시 분산 처리만을 목적으로 설정된 노드입니다. 대량의 데이터를 처리할 경우에 효율적으로 사용할 수 있는 노드입니다.

## 인덱스 (Index)

 엘라스틱서치 인덱스는 유사한 특성을 가지고 있는 문서를 모아둔 문서들의 컬렉션입니다. **클러스터 내부에 생성되는 모든 인덱스는 클러스터 내에서 유니크한 인덱스명을 가져야하며, 모두 소문자로 설정되어야 합니다.**

## 문서 (Document)

 문서는 검색 대상이 되는 실제 물리적인 데이터를 뜻합니다. 문서는 인덱스를 생성할 수 있는 기본적인 정보 단위입니다. 인덱스에는 원하는 만큼의 많은 문서를 저장할 수 있으며 실제로는 물리적인 샤드 형태로 나눠져서 다수의 노드로 분산 저장됩니다.

## 샤드 (Shard)

 인덱스에는 매우 많은 양의 문서가 저장될 수 있습니다. 이러한 이유로 하드웨어의 물리적인 한계를 넘기 위해 샤드라는 개념을 도입하였습니다. 이를 이용하면 데이터를 분산 저장하는 방식으로 손쉬운 **수평 확장**이 가능해집니다. 샤드의 개수는 원하는 만큼 설정할 수 있습니다. 샤드는 전체 데이터의 부분집합이지만 각 샤드는 자신이 가지고 있는 데이터만으로도 독립적인 검색 서비스가 가능합니다.

 실제로 인덱스에 쿼리를 요청하면 인덱스가 가지고 있는 모든 샤드로 검색 요청이 전달되고 각 샤드에서 1차적으로 검색이 이뤄진 후 그 결과를 취합해서 최종 결과로 제공됩니다. 결과적으로 **여러 노드에서 샤드를 통해 분산 처리되므로 성능이나 처리량을 향상시킬 수 있습니다.**

## 레플리카 (Replica)

 샤드의 복제본을 레플리카라고 합니다. 엘라스틱서치에서는 인덱스를 생성할 때 기본적으로 1개의 레플리카 세트를 생성합니다. 검색 시 레플리카가 적극적으로 활용되기 때문에 이를 이용하면 읽기 성능에 유리해집니다.

 수평적으로 분산된 시스템을 운영할 때는 언제든지 장애가 발새할 수 있습니다. 엘라스틱서치는 특정 노드에 장애가 발생하여 다운되었을 때 레플리카를 이용하여 페일오버(Failover) 메커니즘으로 안정적인 클러스터 운영을 보장합니다. **레플리카는 기본적으로 원본 샤드가 존재하지 않는 노드에서 생성됩니다.**

 인덱스가 생성될 때 샤드 개수와 레플리카 개수를 자유롭게 설정할 수 있습니다. 하지만 인덱스가 생성된 이후에는 샤드 개수를 변경하는 것이 불가능하기 때문에 현재 운영하는 환경을 잘 고려하여야 합니다. 그와 반해 레플리카 개수는 인덱스를 생성한 이후에도 자유롭게 변경하는 것이 가능합니다. 그렇기 때문에 운영 중 트래픽 증가에 따른 유연한 대응이 가능해집니다.

## 세그먼트 (Segment)

 문서들은 빠른 검색에 유리하도록 설계된 특수한 자료구조로 저장됩니다. 샤드 내부에는 루씬 라이브러리를 포함하고 있는데, 이를 이용해 대부분의 검색 기능을 제공합니다. 루씬에 데이터가 색인되면 데이터는 최소한의 단위인 토큰으로 분리되고 특수한 형태의 자료구조로 저장되는데 이렇게 저장된 자료구조를 세그먼트라고 합니다.

 세그먼트는 읽기에 최적화된 자료구조로써 역색인이라는 특수한 형태로 변환되어 물리적인 디스크에 저장됩니다. 검색엔진의 특성상 쓰기 연산보다 읽기 연산의 비중이 비교적 높기 때문에 좀 더 효율적으로 검색될 수 있도록 고안된 자료구조가 바로 역색인 구조입니다.

## 루씬에 대해서..

 엘라스틱서치는 루씬 라이브러리를 샤드 내부에 가지고 있으며, 이 루씬 라이브러리가 핵심 모듈입니다. 루씬은 다수의 클래스로 구성되어 있는 검색 라이브러리이고, 이 중에서 가장 중요한 클래스가 IndexWriter와 IndexSearcher입니다. **IndexWriter는 데이터를 색인하는 클래스**이고, **IndexSearcher는 색인된 데이터를 검색 결과로 제공하는 클래스**입니다.

 IndexWriter와 IndexSearcher를 가지고 색인과 검색을 동시에 제공하는 루씬 인스턴스를 루씬 인덱스라고 하는데, 하나의 엘라스틱서치 샤드는 하나의 루씬 인덱스라고 볼 수 있습니다.

<table style="border-collapse: collapse; width: 100%; height: 76px;" border="1" data-ke-style="style4"><tbody><tr style="height: 19px;"><td style="width: 100%; height: 19px; text-align: center;" colspan="8">Elasticsearch Index</td></tr><tr style="height: 19px;"><td style="width: 25%; height: 19px; text-align: center;" colspan="2">Elasticsearch shard</td><td style="width: 25%; height: 19px; text-align: center;" colspan="2"><span style="color: #333333;">Elasticsearch shard</span></td><td style="width: 25%; height: 19px; text-align: center;" colspan="2"><span style="color: #333333;">Elasticsearch shard</span></td><td style="width: 25%; height: 19px; text-align: center;" colspan="2"><span style="color: #333333;">Elasticsearch shard</span></td></tr><tr style="height: 19px;"><td style="width: 25%; height: 19px; text-align: center;" colspan="2">Lucene index</td><td style="width: 25%; height: 19px; text-align: center;" colspan="2"><span style="color: #333333;">Lucene index</span></td><td style="width: 25%; height: 19px; text-align: center;" colspan="2"><span style="color: #333333;">Lucene index</span></td><td style="width: 25%; height: 19px; text-align: center;" colspan="2"><span style="color: #333333;">Lucene index</span></td></tr><tr style="height: 19px;"><td style="width: 12.5%; height: 19px; text-align: center;">Segment</td><td style="width: 12.5%; height: 19px; text-align: center;"><span style="color: #333333;">Segment</span></td><td style="width: 12.5%; height: 19px; text-align: center;"><span style="color: #333333;">Segment</span></td><td style="width: 12.5%; height: 19px; text-align: center;"><span style="color: #333333;">Segment</span></td><td style="width: 12.5%; height: 19px; text-align: center;"><span style="color: #333333;">Segment</span></td><td style="width: 12.5%; height: 19px; text-align: center;"><span style="color: #333333;">Segment</span></td><td style="width: 12.5%; height: 19px; text-align: center;"><span style="color: #333333;">Segment</span></td><td style="width: 12.5%; height: 19px; text-align: center;">&nbsp;</td></tr></tbody></table>

 루씬 인덱스 내부에는 세그먼트라는 특수한 자료구조가 다수 존재합니다. 루씬 인덱스는 세그먼트를 이용해 검색을 수행하는데, 세그먼트는 내부적으로 역색인 구조이기 때문에 이를 통해 빠른 검색 결과를 얻을 수 있습니다.

#### 세그먼트

-   루씬 내부에 존재하는 자료구조입니다.
-   역색인 구조로 생성되어 읽기에 최적화되어 있습니다.
-   하나의 루씬 내부에서만 존재할 수 있고 확장이 불가능합니다.

#### 루씬 인덱스

-   검색과 색인 기능을 가진 최소한의 검색엔진입니다.
-   IndexWriter로 색인 과정을 통해 세그먼트를 생성합니다.
-   IndexSearcher를 이용해 세그먼트를 검색합니다.
-   자신이 가진 세그먼트 내에서만 검색이 가능합니다.

#### 엘라스틱서치 샤드

-   엘라스틱서치에서 제공하는 가장 작은 단위의 검색엔진입니다.
-   내부적으로 루씬을 확장해서 검색엔진 역할을 수행합니다.
-   다수의 샤드가 협력해서 존재하는 모든 세그먼트를 검색할 수 있습니다.

---

## 참고자료

[엘라스틱서치 실무 가이드](http://www.kyobobook.co.kr/product/detailViewKor.laf?ejkGb=KOR&mallGb=KOR&barcode=9791158391485&orderClick=LAG&Kc=) <<권택환, 김동우, 김흥래, 박진현, 최용호, 황희정 지음>>
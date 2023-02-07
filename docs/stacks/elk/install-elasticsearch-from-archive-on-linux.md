---
layout: default
title: Install Elasticsearch from archive on Linux
grand_parent: Stacks
parent: ELK
---

{: .no_toc }
<details open markdown="block">
  <summary>
    목차
  </summary>
  {: .text-delta }
- TOC
{:toc}
</details>

## Host Setup

- Ubuntu 20.04.5
- Docker Engine version 20.10.23
- Docker Compose version 2.15.1

## 1. Introduction

Elasticsearch는 확장성이 뛰어난 분산형 오픈 소스 검색 및 분석 엔진으로, 대용량 데이터에 대한 실시간 검색 및 분석을 제공하기 위해 개발되었습니다.

Elasticsearch는 Apache Lucene 라이브러리를 기반으로 하며 대량의 데이터를 효율적이고 빠르게 처리하도록 구축되었습니다.

Elasticsearch는 다음과 같은 다양한 애플리케이션에서 유용하게 사용됩니다:

- **Search**: 텍스트, 숫자, 날짜 등 다양한 데이터 유형에 대해 빠르고 관련성 높은 검색 결과를 제공합니다.
- **Analytics**: 강력한 분석 기능을 제공하여 복잡한 데이터 분석과 시각화를 실시간으로 수행할 수 있습니다.
- **Log analysis**: 로그 분석에 널리 사용되며 로그 데이터를 저장, 검색, 분석할 수 있는 중앙 집중식 플랫폼을 제공합니다.
- **Business intelligence**: 비즈니스 인텔리전스 도구의 데이터 소스로 사용할 수 있으며, 데이터 분석 및 보고를 위한 유연하고 확장 가능한 플랫폼을 제공합니다.
- **Monitoring**: 시스템 로그, 네트워크 로그, 애플리케이션 로그와 같은 다양한 유형의 데이터를 모니터링하고 경고하는 데 사용할 수 있어 문제를 신속하게 탐색하고 해결하는 데 도움이
됩니다.

요약하자면, Elasticsearch는 검색, 분석, 로그 분석, 비즈니스 인텔리전스, 모니터링 등 다양한 애플리케이션에 널리 사용되는 확장성이 뛰어난 **분산형 검색 및 분석 엔진**입니다.

## 2. Host Server Requirements

### Hardware Requirements

- 2GB RAM (운영의 경우 4GB RAM 이상 권장)
- 2 vCPUs (운영의 경우 4 vCPUs 이상 권장)

### Software Requirements

- Java 11 이상(Java 8은 버전 7.10까지 지원)
- Linux, macOS, Windows 등 Java를 지원하는 모든 운영 체제

{: .note }
> Elasticsearch 7.0 버전 부터는 open-jdk가 포함되어 있어 별도로 java를 설치하지 않아도 됩니다.
> 다만, Host의 운영체제에 맞는 버전를 받아야 합니다. 운영체제에 따른 버전은 [Support Matrix](https://www.elastic.co/kr/support/matrix) 참조



## 3. Installing Elasticsearch

### 3.1 Installation

설치하고자 하는 서버의 운영체제와 버전을 확인하고 [Download Elasticsearch](https://www.elastic.co/kr/downloads/elasticsearch)에서 다운받습니다.

저의 경우에는 Ubuntu 배포판 Elasticsearch 7.17.8 버전으로 구성했습니다.

```bash
$ wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-7.17.8-linux-x86_64.tar.gz
```

### 3.2 Extract ZIP

다운받은 배포판의 압축을 풀고 사용하기 편하게 이름을 변경합니다.

```bash
$ tar xfz elasticsearch-7.17.8-linux-x86_64.tar.gz
$ mv elasticsearch-7.17.8-linux-x86_64 es-717
$ tree es-717 -L 1
├── bin
├── config
├── jdk
├── lib
├── LICENSE.txt
├── logs
├── modules
├── NOTICE.txt
├── plugins
└── README.asciidoc
```

이렇게만 하면 기본적인 Elasticsearch 설치는 끝입니다.

다음은 기본적으로 포함되어 있는 폴더들에 대한 간단한 설명입니다:

- **bin**: 서비스를 제어하고 관리하는데 사용되는 여러 실행 스크립트들이 포함되어 있습니다.
    - `elasticsearch`: 서비스를 시작, 중지하거나 상태를 확인할 수 있습니다.
    - `elasticsearch-plugin`: 플러그인을 설치, 조회 및 제거하는 데 사용됩니다.
    - `elasticsearch-keystore`: 기본 사용자의 비밀번호와 같은 민감한 설정에 대한 키 저장소를 관리하는 데 사용됩니다.
    - `elasticsearch-certutil`: 노드와 클라이언트 간의 보안 통신을 위해 인증 기관(CA)과 인증서를 관리하는 데 사용됩니다.
    - `elasticsearch-node`: 노드 이름을 변경하거나 샤드 할당을 비활성화하는 등 노드 수준 작업을 수행하는 데 사용됩니다.
    - `elasticsearch-shard`: 샤드 재배치 또는 분할과 같은 샤드 수준 작업을 수행하는 데 사용됩니다.
    - `elasticsearch-cluster`: 클러스터의 상태를 관리하거나 복제본 수를 변경하는 등 클러스터 수준 작업을 수행하는 데 사용됩니다.
- **config**: 서비스의 동작을 구성하는 데 사용되는 여러 구성 파일이 포함되어 있습니다.
    - `elasticsearch.yml`: Elasticsearch의 기본 구성 파일입니다. 노드 이름, 네트워크 설정, 데이터 색인, 검색, 저장과 관련된 기타 설정 등 Elasticsearch 클러스터와 관련된
    설정이 포함되어 있습니다.
    - `jvm.options`: JVM에 대한 구성 옵션이 포함되어 있습니다. 메모리 할당, 가비지 수집 및 기타 JVM 관련 설정과 관련된 옵션이 포함되어 있습니다.
    - `log4j2.properties`: 로깅 시스템에 대한 구성 옵션이 포함되어 있습니다. 로그 수준, 출력 형식 및 로그 파일 위치와 관련된 설정이 포함됩니다.
    - `role_mapping.yml`: 역할 기반 액세스 제어(RBAC) 매핑을 정의하는 데 사용되며, Elasticsearch 클러스터에서 서로 다른 사용자 및 역할에 할당된 기능을 결정합니다.

## 4. Starting the Elasticsearch Service

### 4.1 Running Basic

Elasticsearch는 자체적으로 실행할 수 있는 스크립 파일을 기본으로 제공합니다.

```bash
$ ./bin/elasticsearch
```

별도의 설정 없이도 위 명령어만으로도 간단하게 Elasticsearch를 실행할 수 있습니다.

서비스가 정상적으로 돌아가는지는 아래의 API에 요청해서 확인할 수 있습니다.

```bash
$ curl localhost:9200
{
    "name" : "user",
    "cluster_name" : "elasticsearch",
    "cluster_uuid" : "kongqtNMTdmg8L1DKrPs7g",
    "version" : {
        "number" : "7.17.8",
        "build_flavor" : "default",
        "build_type" : "tar",
        "build_hash" : "120eabe1c8a0cb2ae87cffc109a5b65d213e9df1",
        "build_date" : "2022-12-02T17:33:09.727072865Z",
        "build_snapshot" : false,
        "lucene_version" : "8.11.1",
        "minimum_wire_compatibility_version" : "6.8.0",
        "minimum_index_compatibility_version" : "6.0.0-beta1"
    },
    "tagline" : "You Know, for Search"
}
```

### 4.2 Running Backgroud

위의 Elastictsearch 설치 명령어에 `-d` 옵션을 주면 백그라운드로 실행됩니다.

```bash
$ ./bin/elasticsearch -d
$ ps -ef | grep elasticsearch
elk 44371 1 77 22:24 pts/3
elk 44452 42933 0 22:24 pts/3
```

작동중인 서비스의 로그는 `logs/elasticsearch.log`에서 확인할 수 있습니다.

```bash
$ tail -f logs/elasticsearch.log
[2023-02-06T22:24:27,176][INFO ][o.e.i.g.DatabaseNodeService] [user] retrieve geoip database [GeoLite2-City.mmdb] from
[.geoip_databases] to
[/tmp/elasticsearch-8270268597875853814/geoip-databases/kSoo1Gv8SFuZ1y7VuG3nXw/GeoLite2-City.mmdb.tmp.gz]
[2023-02-06T22:24:27,177][INFO ][o.e.i.g.DatabaseNodeService] [user] retrieve geoip database [GeoLite2-ASN.mmdb] from
[.geoip_databases] to
[/tmp/elasticsearch-8270268597875853814/geoip-databases/kSoo1Gv8SFuZ1y7VuG3nXw/GeoLite2-ASN.mmdb.tmp.gz]
[2023-02-06T22:24:27,188][INFO ][o.e.c.r.a.AllocationService] [user] Cluster health status changed from [RED] to [GREEN]
(reason: [shards started [[.ds-ilm-history-5-2023.02.07-000001][0]]]).
[2023-02-06T22:24:27,302][INFO ][o.e.i.g.DatabaseNodeService] [user] successfully reloaded changed geoip database file
[/tmp/elasticsearch-8270268597875853814/geoip-databases/kSoo1Gv8SFuZ1y7VuG3nXw/GeoLite2-Country.mmdb]
[2023-02-06T22:24:27,348][INFO ][o.e.i.g.DatabaseNodeService] [user] successfully reloaded changed geoip database file
[/tmp/elasticsearch-8270268597875853814/geoip-databases/kSoo1Gv8SFuZ1y7VuG3nXw/GeoLite2-ASN.mmdb]
[2023-02-06T22:24:27,811][INFO ][o.e.i.g.DatabaseNodeService] [user] successfully reloaded changed geoip database file
[/tmp/elasticsearch-8270268597875853814/geoip-databases/kSoo1Gv8SFuZ1y7VuG3nXw/GeoLite2-City.mmdb]
```

Elasticsearch 서비스를 백그라운드로 실행시킬때 함께 사용하기 좋은 `-p` 옵션도 있습니다.

이 옵션은 해당 서비스를 백그라운드로 실행 시키는 프로세스 아이디를 적은 파일을 생성해 주는데 아래와 같이 사용할 수 있습니다.

```bash
$ ./bin/elasticsearch -d -p pid
$ cat pid
45151
```

이렇게 PID 45151로 Elasticsearch가 작동중이고 이를 종료할 때는 아래와 같이 종료시킵니다.

```bash
$ pkill -F pid
```

`-p` 옵션으로 실행된 Elasticsearch는 종료될 시점에 생성한 PID가 적인 파일을 자동으로 삭제됩니다.

## 5. Configuring Elasticsearch

Elasticsearch를 구동하기 위한 대부분의 설정은 `config/elasticsearch.yml`에서 합니다.

다음은 `elasticsearch.yml`에서 구성할 수 있는 몇 가지 주요 설정입니다:

- **Cluster Settings**: Elasticsearch 클러스터의 이름, 서비스가 실행 중인 노드의 이름, 클러스터의 마스터 노드 수를 구성할 수 있습니다.
- **Network Settings**: 서비스가 수신 대기해야 하는 호스트 이름 또는 IP 주소, 사용해야 하는 포트 번호와 같은 Elasticsearch 서비스에 대한 네트워크 설정을 구성할 수 있습니다. SSL 및
TLS 인증서와 같은 보안 및 인증과 관련된 설정도 구성할 수 있습니다.
- **Indexing Settings**: 각 인덱스에 사용되는 샤드 및 복제본 수, 인덱스 새로 고침 간격, 인덱스에 저장할 수 있는 최대 문서 수 등 Elasticsearch에서 데이터를 색인하고 저장하는 것과 관련된
설정을 구성할 수 있습니다.
- **Search Settings**: 검색 스레드 수, 단일 검색 요청에서 반환할 수 있는 최대 결과 수 등 Elasticsearch에서 데이터를 검색 및 검색하는 것과 관련된 설정을 구성할 수 있습니다.
- **Memory and Performance Settings**: JVM에 할당된 메모리 양, 사용된 가비지 수집 알고리즘, 기타 성능 관련 설정 등 Elasticsearch를 실행하는 Java 가상 머신(JVM)과
관련된 설정을 구성할 수 있습니다.

## 6. Conclusion

이번 포스팅에서는 Elasticsearch를 작동시키는 가장 기본적인 과정에 대해 정리했습니다.

실제 운영환경에서는 클러스터링, 노드 보안, Realm 등 수 많은 설정들이 적용되어야 합니다.  

다음 포스팅 부터는 Elasticsearch를 운영 환경에 적용하기 위한 설정들을 하나 하나 정리하도록 하겠습니다.
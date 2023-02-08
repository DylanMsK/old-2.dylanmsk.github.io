---
layout: default
title: Running Kibana in background
nav_order: 1
grand_parent: ELK Stacks
parent: Kibana
---

{: .no_toc }
<details open markdown="block">
  <summary>
    목차
  </summary>
  {: .text-delta }
- TOC
{:toc}
---
</details>


## Host Setup

- Ubuntu 20.04.5
- 4GB RAM
- 4 vCPUs

## 1. Introduction

Kibana는 Elasticsearch를 실행할 때 사용하는 `-d` 옵션같은 백그라운드 실행을 제공하지 않습니다.

그렇기 때문에 Kibana 실행 명령어 뒤에 `&` 를 붙히거나 `nohup`으로 백그라운드 실행을 시키거나 systemd에 서비스를 등록해서 system이 시작될 때 실행시키는 방법을 많이 사용합니다.

이번 포스팅에서는 위 방법들 이외에 운영 환경에서 쓰기 좋은 Kibana를 백그라운드로 실행시키는 방법을 소개합니다.

사용법을 설명하기 앞서 Kibana의 작업 프로세스를 보면 node로 작동하고 있는것을 확인할 수 있습니다.

```bash
$ ps -ef | grep node
elk      48333   48271 30 22:35 pts/0    00:00:07 bin/../node/bin/node bin/../src/cli/dist
elk      48352   48271  0 22:36 pts/0    00:00:00 grep --color=auto node
```

실제로 `src/cli/cli.js` 가 Kibana의 실행 스크립으로, 해당 javascript 파일을 실행시키면 `bin/kibana` 로 실행시키는 것과 동일합니다.

단, 이 때 중요한 점은 ELK 스택은 버전 의존도가 높기 때문에  반드시 Kibana에 포함되어있는 node와 같은 버전으로 돌려야 합니다.

```bash
$ ./node/bin/node ./src/cli/cli.js
log   [22:49:04.089] [info][savedobjects-service] Waiting until all Elasticsearch nodes are compatible with Kibana before starting saved objects migrations...
log   [22:49:04.089] [info][savedobjects-service] Starting saved objects migrations
log   [22:49:04.128] [info][savedobjects-service] [.kibana_task_manager] INIT -> OUTDATED_DOCUMENTS_SEARCH_OPEN_PIT. took: 16ms.
log   [22:49:04.151] [info][savedobjects-service] [.kibana] INIT -> OUTDATED_DOCUMENTS_SEARCH_OPEN_PIT. took: 40ms.
log   [22:49:04.169] [info][savedobjects-service] [.kibana] OUTDATED_DOCUMENTS_SEARCH_OPEN_PIT -> OUTDATED_DOCUMENTS_SEARCH_READ. took: 18ms.
log   [22:49:04.170] [info][savedobjects-service] [.kibana_task_manager] OUTDATED_DOCUMENTS_SEARCH_OPEN_PIT -> OUTDATED_DOCUMENTS_SEARCH_READ. took: 42ms.
```

Node.js에는 [PM2](https://pm2.keymetrics.io/)라는 무중단 서비스를 위해 많이 사용되는 demon process manager가 있는데 이 패키지를  사용해서 Kibana를 띄워보도록 하겠습니다.

## 2. Runnning in Background

PM2를 설치하기 앞서 전역에 설치된 node의 버전이 Kibana의 node 버전과 맞는지 확인합니다.

Kibana의 node 버전은 `package.json` 을 확인하면 됩니다.

```bash
$ cat package.json | grep node
    "node": "16.18.1"
```

만약 전역에 Node.js가 설치되어있지 않던가, 버전이 일치하지 않다면 [nvm](https://github.com/nvm-sh/nvm) 을 이용해 설치하도록 합니다.

Kibana의 node와 동일한 버전을 받았다면 아래 명령어로 PM2를 설치합니다.

```bash
$ npm install pm2 -g
```

설치가 완료 됐다면 아래 명령어로 간단하게 Kibana를 데몬으로 실행시킬 수 있습니다.

```bash
$ pm2 start src/cli/cli.js --name kibana
```

실행중인 데몬을 중지시키는 방법은 아래와 같습니다.

```bash
$ pm2 stop kibana
$ # 또는
$ pm2 delete kibana
```

## 3. Conclusion

이번 포스팅에서는 PM2를 사용해 Kibana를 데몬으로 실행시키는 방법에 대해 정리했습니다.

PM2는 필요에 따라 서비스를 쉽게 모니터링이 가능하고, 서비스를 재시작, 중지, 삭제 등 편리하게 관리할 수 있습니다.

뿐만 아니라 서버의 리소스 모니터링, 실시간 로그 관리 등 다양한 기능을 제공하기 때문에 자세한 사용법은 [PM2 Document](https://pm2.keymetrics.io/docs/usage/quick-start/) 를 참고하시길 바랍니다.

---

## References

- [Start and stop Kibana](https://www.elastic.co/guide/en/kibana/master/start-stop.html)
- [PM2 Document](https://pm2.keymetrics.io/docs/usage/quick-start/)
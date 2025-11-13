# Docker

Docker의 핵심 개념(컨테이너, 이미지, 네트워크, Dockerfile 등)을 정리한 문서입니다.

## 학습 전략과 docker-lab 구조

* 주제별 실험 단위로 관리하는 랩 레포를 만든다.

```text
docker-lab/
├── README.md
└── topics/
    ├── 001-container-basics/
    │   ├── Dockerfile
    │   ├── notes-raw.md
    │   └── notes.md
    ├── 002-layer-and-build-cache/
    ├── 020-network-basics/
    ├── 030-volume-basics/
    ├── 040-compose-basics/
    ├── 050-prod-image-optimization/
    └── 060-cgroups-and-limits/
```

### notes-raw vs notes

* `notes-raw.md`: 실험 중 바로 적는 날것의 기록(명령, 로그, 느낌, 에러 등).
* `notes.md`: raw를 기반으로 정리된 최종 문서(목표, 실험 과정, 관찰, 개념 정리, 실무 의미).
* 패턴:

  * **본인은 실험 + raw 기록**
  * **LLM에게 정리(notes.md) 위임** → 재사용 가능한 지식 문서로 축적.

## 컨테이너의 본질: 컨테이너 = 프로세스

### PID namespace와 PID 1

* 컨테이너는 **독립된 PID namespace**를 가진다.

  * 컨테이너 내부: PID 1, 2, 3…
  * 호스트: 15324, 15325… 전혀 다른 번호.
* 컨테이너 내부의 **PID 1**은 특별한 역할:

  * 종료 시그널(SIGTERM 등)을 가장 먼저 받는다.
  * 자식 프로세스의 zombie 처리 책임을 가진다.
  * PID 1이 종료되면 컨테이너도 종료된다.

### 여러 프로세스가 보이는 이유

* ENTRYPOINT/CMD로 실행된 프로그램이 내부적으로 fork할 수 있다.

  * 예: nginx master + worker 프로세스들.
* ENTRYPOINT가 쉘 스크립트를 돌리는 경우:

  * PID 1 = `/bin/sh` 또는 `bash`,
  * 실제 앱 프로세스는 자식 프로세스가 된다.

## Dockerfile 문법과 쉘의 관계

### Dockerfile = Docker DSL + 쉘 명령

* Docker 고유 명령:

  * `FROM`, `RUN`, `COPY`, `ADD`, `CMD`, `ENTRYPOINT`, `ENV`, `WORKDIR` 등은 **Dockerfile DSL**.
* 쉘은 특정 구간에서만 동작:

  * `RUN`: 거의 항상 `/bin/sh -c "..."` 형태로 실행.
  * `CMD`, `ENTRYPOINT`의 shell form (`CMD java -jar app.jar`)일 때도 쉘을 거친다.

### exec form vs shell form

* exec form:

  ```dockerfile
  ENTRYPOINT ["java", "-jar", "app.jar"]
  ```

  * 쉘을 거치지 않고 바로 실행.
  * PID 1 = java 프로세스 → signal 처리, graceful shutdown 측면에서 권장.
* shell form:

  ```dockerfile
  ENTRYPOINT java -jar app.jar
  ```

  * 실제로는 `/bin/sh -c "java -jar app.jar"` 실행.
  * PID 1이 쉘이 되어 signal 전달 문제, zombie 처리 이슈가 생길 수 있다.

## 네트워크와 포트 포워딩의 내부 동작

### 네트워크 네임스페이스와 veth pair

* 컨테이너 생성 시:

  * **network namespace**가 생성되어 컨테이너만의 `eth0`, 라우팅 테이블, IP가 생긴다.
  * 가상 이더넷 인터페이스 쌍(veth pair)이 생성된다.

    * 컨테이너 내부: `eth0`
    * 호스트: `vethXXXX`
* 호스트의 `docker0` 브리지(가상 스위치)에 `vethXXXX`가 연결되고, 이 브리지를 통해 컨테이너들끼리 통신한다.

### 포트 포워딩 = iptables NAT

* 예: `docker run -p 8080:80 app`

  * Docker가 내부적으로 iptables NAT 규칙을 추가한다.
  * DNAT: 호스트 `:8080` → 컨테이너 `172.17.0.X:80`
  * SNAT/MASQUERADE: 컨테이너 outbound 트래픽이 호스트 IP에서 나가는 것처럼 보이게 처리.
* 결론:

  * **컨테이너가 직접 포트를 여는 것이 아니라, 리눅스 커널이 iptables NAT를 통해 포워딩을 수행한다.**

## 이미지 레이어와 dive

### 레이어 구조

* Docker 이미지는 레이어 스택으로 구성된다.

  * `FROM`, `RUN`, `COPY` 등 명령 하나가 레이어 하나를 만든다.
* 레이어는 불변이며, 위에 새로운 레이어가 쌓이는 형태로 변화가 기록된다.
* 이 구조 덕분에:

  * 빌드 캐시 활용 가능.
  * base 레이어를 여러 이미지에서 공유 가능.
  * 작은 변경만 새 레이어로 추가할 수 있다.

### dive의 역할

* `dive <이미지>`로 각 레이어에서:

  * 추가/수정/삭제된 파일 목록,
  * 레이어 크기,
  * 최적화 점수 등을 분석할 수 있다.
* 활용 포인트:

  * 불필요하게 비대해진 레이어 찾기.
  * 잘못된 `COPY`나 `RUN` 순서로 인한 캐시 깨짐 확인.
  * multi-stage build 전/후를 비교해 최적화 효과 검증.

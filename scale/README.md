# 🐳 Docker Compose로 Locust Worker 스케일링 시 컨테이너 이름 충돌 해결

## 개요

프로젝트의 성능 테스트 환경으로 Python 기반의 부하 테스트 도구인 Locust를 사용하였으며, Docker Compose를 이용해 Master-Worker 구조로 구성하고, --scale 옵션을 통해 Worker 다중 배포를 실현하고자 하였습니다.

## 문제 상황

초기 docker-compose.yml 설정에서 다음과 같이 locust-worker 서비스에 container_name을 명시한 상태에서 테스트를 수행했습니다:

```
  locust-worker:
    container_name: locust-worker
    image: locustio/locust
    ...
```

```
docker-compose up --scale locust-worker=3
```

## 에러 메시지:
```
Conflict. The container name "/locust-worker" is already in use by container ...
```

container_name을 명시할 경우, 해당 이름은 고유해야 하며 `동일 이름의 컨테이너`를 여러 개 생성할 수 없습니다.

--scale은 같은 서비스 정의를 여러 컨테이너로 복제하므로, 정적인 이름이 충돌을 일으킵니다.

## 원인 분석

Docker의 container_name은 단일 컨테이너에만 부여할 수 있는 고유 식별자입니다.

--scale locust-worker=3 명령은 locust-worker_1, locust-worker_2, locust-worker_3과 같이 자동으로 이름을 생성하려 하지만, 수동으로 container_name: locust-worker를 지정하면 `모든 인스턴스가 동일한 이름을 요구`하게 되어 충돌이 발생합니다.

## 해결 방법

container_name을 제거하여 Docker가 각 컨테이너에 자동으로 고유 이름을 부여하도록 

```
  locust-worker:
    image: locustio/locust
    depends_on:
      - locust-master
    volumes:
      - ./:/mnt/locust
    command: >
      -f /mnt/locust/locustfile-test.py
      --worker
      --master-host locust-master
```
```
docker-compose up --scale locust-worker=3
```

## 결과

* container_name 설정을 제거함으로써 Locust Worker 컨테이너의 `수평 확장`이 가능해졌습니다.

* 부하 테스트 시스템을 설계하면서 Master-Worker 구조의 통신 방식, 동기화 구조, 확장성까지 함께 고려한 경험을 통해 성능 최적화 및 테스트 자동화 기반을 마련했습니다.

* 컨테이너 이름 충돌은 스케일링 환경에서 자주 발생할 수 있는 실무 이슈이며, 이를 통해 Docker 내부 작동 방식에 대한 깊은 이해했습니다.


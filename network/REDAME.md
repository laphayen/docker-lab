# 🐳 Docker 네트워크 실습 포트폴리오

본 프로젝트는 Docker Compose를 활용하여 컨테이너 간 네트워크 구성을 테스트하고, 명시적 네트워크 설정 유무에 따른 통신 가능성 차이를 실증한 포트폴리오입니다. 이 실습을 통해 Docker 내부 DNS 해석, 컨테이너 간 통신 구조, 네트워크 전략 구성에 대한 실무 감각을 키울 수 있었습니다.

---

## 디렉토리 구조

```
.
├── backend/
│   └── Dockerfile                 # ping 명령어 포함한 백엔드 테스트 이미지
├── docker-compose.named-with-networks.yml
└── docker-compose.named-without-networks.yml
```

---

## 실습 1: 명시적 네트워크 사용

### docker-compose.named-with-networks.yml

```yaml
version: '3.7'

services:
  mysql:
    container_name: test_mysql_with_network
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=testdb
    ports:
      - "3307:3306"
    networks:
      - my_network

  backend:
    container_name: test_backend_with_network
    build:
      context: ./backend
      dockerfile: Dockerfile
    networks:
      - my_network

networks:
  my_network:
    driver: bridge
```

### 결과

- `backend` 컨테이너에서 `ping mysql` 성공
- Docker 내부 DNS를 통해 `mysql`이라는 이름을 해석
- 통신 로그 예시:

```
PING mysql (172.23.0.3) 56(84) bytes of data.
64 bytes from test_mysql_with_network.network_my_network: icmp_seq=1 ttl=64 time=0.463 ms
```

---

## 실습 2: 네트워크 명시 없이 default 네트워크 사용

### docker-compose.named-without-networks.yml

```yaml
version: '3.7'

services:
  mysql:
    container_name: test_mysql_without_network
    image: mysql:8.0
    environment:
      - MYSQL_ROOT_PASSWORD=root
      - MYSQL_DATABASE=testdb
    ports:
      - "3308:3306"

  backend:
    container_name: test_backend_without_network
    build:
      context: ./backend
      dockerfile: Dockerfile
    command: ["sh", "-c", "ping -c 4 mysql"]
```

### 결과

- 예상과 다르게 `ping mysql` 성공
- 이유: Docker Compose는 같은 파일 내의 서비스에 대해 `default` 네트워크를 자동으로 구성함
- 통신 로그 예시:

```
64 bytes from test_mysql_without_network.network_default (172.24.0.3): icmp_seq=2 ttl=64 time=0.081 ms
```

---

## backend/Dockerfile

```Dockerfile
FROM openjdk:17-jdk-slim

RUN apt update && apt install -y iputils-ping

CMD ["sh", "-c", "ping -c 4 mysql"]
```

---

## 실습을 통해 얻은 인사이트

- Docker Compose는 기본적으로 default 네트워크를 생성하여 컨테이너 간 통신을 지원
- 하지만 **명시적인 `networks:` 구성을 통해 다음과 같은 이점 확보**:
  - 네트워크 이름이 명확해짐 → 디버깅 시 유리
  - 다른 Compose 프로젝트와 충돌 방지
  - 외부 연동 및 프록시 구성 시 필수적
  - 유지보수성과 확장성 확보

---

## 실습 결론 정리

| 항목 | default 네트워크 | 명시적 네트워크 설정 |
|------|------------------|----------------------|
| 통신 가능 | ✅ | ✅ |
| 네트워크 명확성 | ❌ (default 이름) | ✅ (직접 지정) |
| 디버깅 편의성 | ❌ | ✅ |
| 실무 확장성 | 제한적 | 유연 |

---

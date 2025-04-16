# backend/Dockerfile
FROM openjdk:17-jdk-slim

# ping 명령어를 위한 설치
RUN apt update && apt install -y iputils-ping

# mysql 컨테이너로 ping 테스트
CMD ["sh", "-c", "ping -c 4 mysql"]
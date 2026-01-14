# WeKnora 도커 실행 및 설치 실패 원인 분석 보고서

이 문서는 WeKnora 프로젝트 설치 및 `docker compose up -d` 실행 시 발생한 주요 문제점과 그 원인, 해결 방안을 정리한 보고서입니다.

## 1. 주요 증상 및 확인된 에러
분석 결과, 다음과 같은 핵심 에러가 확인되었습니다:
- **Frontend 서버 로그**: `nginx: [emerg] host not found in upstream "app"`
- **서비스 상태**: 특정 컨테이너가 `Exited (1)` 상태로 종료되거나 계속 `starting` 상태에 머묾.
- **의존성 문제**: `app` 서비스가 `postgres`나 `docreader`의 헬스체크를 기다리다 타임아웃 발생 가능성.

---

## 2. 원인 분석

### 2.1 Nginx 업스트림 DNS 해석 실패 (가장 직접적인 원인)
Nginx는 기본적으로 기동 시 설정 파일(`nginx.conf`)에 정의된 모든 호스트 이름을 체크합니다. 
- **원인**: `frontend` 컨테이너가 시작될 때, 아직 `app` 컨테이너가 네트워크 상에서 준비(IP 할당 및 DNS 등록)되지 않았기 때문에 Nginx가 "app이라는 호스트를 찾을 수 없다"며 기동을 거부하고 종료됩니다.
- **로그 증거**: `host not found in upstream "app" in /etc/nginx/conf.d/default.conf:26`

### 2.2 시스템 리소스 부족 (WSL2/Docker Desktop)
WeKnora는 **ParadeDB(Postgres 기반)**, **DocReader(PaddleOCR 및 브라우저 엔진)** 등 리소스를 많이 점유하는 다수의 서비스를 동시에 구동합니다.
- **원인**: Windows의 Docker Desktop이나 WSL2의 기본 메모리 할당량(보통 2GB)을 초과하여 OOM(Out of Memory)이 발생, 특정 컨테이너가 강제 종료됩니다.
- **결과**: 서비스가 뜨다가 죽는 현상이 반복됩니다.

### 2.3 서비스 초기화 지연 및 헬스체크 타임아웃
- **원인**: 
    1. **DB 마이그레이션**: `app` 서비스 시작 시 데이터베이스 스키마 마이그레이션이 실행되는데, DB 서버가 느릴 경우 헬스체크가 실패한 것으로 간주될 수 있습니다.
    2. **DocReader 모델 로드**: `docreader`는 기동 시 OCR 모델 등을 로드하므로 초기 준비 시간이 길 수 있습니다.
- **결과**: `depends_on` 조건(service_healthy)을 만족하지 못해 상위 서비스(app, frontend)가 시작되지 못합니다.

---

## 3. 해결 방안 및 권장 조치

### 3.1 Nginx 설정 개선 (DNS 지연 해석)
Nginx가 실행 중에 DNS를 해석하도록 설정을 변경하면 `app` 서버가 나중에 뜨더라도 `frontend`가 죽지 않습니다.

**수정 제안 (`frontend/nginx.conf`):**
```nginx
location /api/ {
    resolver 127.0.0.11 valid=30s; # Docker 내장 DNS
    set $target http://app:8080;
    proxy_pass $target/api/;
    ...
}
```

### 3.2 도커 리소스 할당 증설
Docker Desktop 설정에서 다음 사양 이상으로 할당을 권장합니다.
- **Memory**: 8GB 이상
- **CPU**: 4 Core 이상

### 3.3 단계별 실행 (순차적 기동)
한 번에 모든 서비스를 띄우기보다 의존성 순서대로 실행하여 부하를 줄이고 상태를 확인합니다.

```bash
# 1. 인프라 서비스 먼저 실행
docker compose up -d postgres redis minio

# 2. 헬스체크 확인 후 나머지 실행
docker compose up -d docreader app

# 3. 마지막으로 프론트엔드 실행
docker compose up -d frontend
```

### 3.4 프로필(Profiles) 확인
일부 서비스(minio, qdrant 등)는 `--profile full` 옵션이 필요할 수 있습니다. 
`.env` 파일의 설정과 일치하는 프로필을 사용해 실행해야 합니다.

---

## 4. 결론
이번 실행 실패는 주로 **Nginx의 엄격한 호스트 체크**와 **초기 구동 시의 리소스 경합**으로 인해 발생했습니다. 위 해결 방안을 적용하면 안정적으로 서비스를 운영할 수 있습니다.

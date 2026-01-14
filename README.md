# 🧠 WeKnora Education Edition

![Python](https://img.shields.io/badge/Python-3.10%2B-blue) ![Docker](https://img.shields.io/badge/Docker-Enabled-2496ED) ![License](https://img.shields.io/badge/License-MIT-green)

## 📖 프로젝트 소개 (Introduction)

이 프로젝트는 **AI 융합 풀스택 개발자 과정** 실습을 위해 구성된 레포지토리입니다.
Tencent의 오픈소스인 **WeKnora**를 기반으로 하며, 기업 실무 환경에서의 **RAG(검색 증강 생성)** 구축 및 **지능형 문서 처리(IDP)** 기술을 학습하기 위해 커스터마이징되었습니다.

### 🎯 학습 목표 (Learning Objectives)

- **RAG 아키텍처 이해:** 대규모 문서를 처리하고 LLM과 연동하는 전체 파이프라인 학습
- **Docker 컨테이너 활용:** 복잡한 AI 서비스를 Docker Compose로 배포하고 관리하는 능력 배양
- **지식 그래프(Knowledge Graph):** 문서 간의 관계를 시각화하고 추론하는 방법 실습
- **Private LLM 연동:** 로컬 LLM(Ollama 등)을 연동하여 기업 내부형 AI 구축 실습

---

## 🛠️ 설치 및 실행 (Quick Start)

### 1. 프로젝트 복제 (Clone)

```bash
git clone https://github.com/lsy3709/WeKnora-.git
cd WeKnora-
```

### 2. 환경 변수 설정

```bash
cp .env.example .env
# .env 파일을 열어 필요한 API Key 및 DB 설정을 입력하세요.
```

### 3. 서비스 실행

```bash
docker-compose up -d
```

실행 후 브라우저에서 `http://localhost:[포트번호]`로 접속하여 대시보드를 확인합니다.

---

## 📂 프로젝트 구조 (Structure)

```text
📦 WeKnora-Education
 ┣ 📂 internal/cmd      # WeKnora 핵심 로직 (Backend)
 ┣ 📂 frontend          # 사용자 인터페이스 (Frontend)
 ┣ 📂 docs              # [실습] 학생들이 분석할 샘플 문서 및 가이드
 ┣ 📜 docker-compose.yml
 ┣ 📜 .env.example      # 환경변수 예시 파일
 ┗ 📜 LICENSE           # 라이선스 파일 (MIT)
```

---

## ⚖️ 라이선스 및 저작권 (License & Credits)

This project is a customized version of **WeKnora** for educational purposes.

- **Original Project:** [Tencent/WeKnora](https://github.com/Tencent/WeKnora)
- **License:** MIT License
- **Copyright:** Copyright (C) 2025 Tencent.

본 프로젝트는 원본 프로젝트의 **MIT License**를 준수합니다. 포함된 제3자 라이브러리의 라이선스 정보는 `LICENSE` 파일에서 확인할 수 있습니다.

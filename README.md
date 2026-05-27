# 혐오 탐지 프로젝트 (Hate text Detection)

## 📌 프로젝트 소개
본 프로젝트는 빅데이터를 활용하여 영상 및 텍스트 콘텐츠 내의 혐오 표현을 탐지하는 모델을 개발하는 프로젝트입니다. 

## 🚀 주요 기능
- **데이터 처리**: 텍스트 분석
- **모델 비교**: BERT와 RoBERTa를 활용한 혐오 표현 분류 성능 비교

## 📂 폴더 구조
```text
hatestext-bd-project/
├── notebooks/           # 분석 및 실험용 노트북 파일
│   ├── baseline/        # 초기 베이스라인 모델
│   ├── bert/            # BERT 모델 실험
│   └── roberta/         # RoBERTa 모델 실험
├── src/                 # 소스 코드 및 데이터 파일
│   └── KOCOH_v.2.csv    # 원천 데이터셋
├── .gitignore           # Git 추적 제외 설정
├── README.md            # 프로젝트 소개 및 가이드
└── requirements.txt     # 프로젝트 필수 라이브러리 목록

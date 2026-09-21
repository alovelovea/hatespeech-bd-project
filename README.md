# 혐오 표현 탐지 프로젝트 (KOCOH Hate Speech Detection)

한국어 온라인 댓글의 혐오 표현을 **맥락(context)**과 함께 탐지하는 모델을 만드는 프로젝트입니다. `context`(원문/게시글)와 `comment`(댓글)를 함께 입력했을 때, 댓글만 보는 방식보다 혐오 탐지 성능이 얼마나 좋아지는지를 KOCOH 데이터셋으로 검증합니다.

## 📌 프로젝트 소개

혐오 표현은 댓글 문장 하나만 놓고 보면 중의적인 경우가 많습니다. 예를 들어 "투자하라"는 댓글은 그 자체로는 혐오 표현이 아니지만, 원문 맥락과 함께 보면 비꼬는 의미의 혐오 표현일 수 있습니다.

본 프로젝트는 이러한 문제의식에서 출발하여,

- **Baseline**: 댓글(comment) 단독 입력으로 혐오 여부를 분류하는 모델
- **제안 모델**: `context + comment`를 함께 입력하여 분류하는 모델

두 방식의 성능을 BERT, RoBERTa 두 모델 기준으로 비교합니다.

## 🚀 주요 기능

- **데이터 전처리**: [KOCOH](https://github.com/AI-networking/K-Comment-Context-Hate) 원천 데이터를 학습/검증/테스트용으로 정제 및 분할
- **Context 기준 Group Split**: 같은 `context`가 train/valid/test에 동시에 섞여 성능이 과대평가되는 것을 막기 위해, context 단위로 그룹을 나눠 split
- **Baseline 모델 학습**: `klue/bert-base`, `klue/roberta-base`로 댓글 단독 분류 모델 학습
- **제안 모델 학습**: 위 두 모델에 `context + comment`를 문장쌍(`[CLS] context [SEP] comment [SEP]`) 형태로 입력하는 분류 모델 학습
- **성능 비교 및 시각화**: Accuracy / Precision / Recall / F1 기준으로 baseline과 제안 모델, 그리고 KOCOH 논문의 LLM 평균 성능까지 비교

## 📊 실험 결과

| 모델 | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| KOCOH 논문 LLM 평균 (GPT-4o 등 6개) | - | - | - | 0.6073 |
| BERT (comment only) | 0.5967 | 0.3472 | 0.6494 | 0.4525 |
| RoBERTa (comment only) | 0.6367 | 0.3710 | 0.5974 | 0.4578 |
| BERT (context + comment, group split) | 0.7722 | 0.5327 | 0.6786 | 0.5969 |
| **RoBERTa (context + comment, group split)** | **0.8284** | **0.6667** | **0.6190** | **0.6420** |

> `context`를 함께 입력한 제안 모델이 댓글 단독 모델 및 LLM 평균 성능(F1 0.6073)보다 높은 F1 점수를 기록했습니다. 자세한 수치는 [`result/`](./result) 폴더의 CSV와 비교 그래프(`all_metrics_comparison.png`)를 참고하세요.

## 🧠 사용 모델 및 학습 설정

| 항목 | 내용 |
|---|---|
| Pretrained 모델 | `klue/bert-base`, `klue/roberta-base` |
| 입력 형식 (제안 모델) | `[CLS] context [SEP] comment [SEP]` |
| Split 방식 | Context 기준 GroupShuffleSplit (train/valid/test) |
| MAX_LENGTH | 256 |
| Epochs | 5 |
| Learning Rate | 2e-5 |
| Class Weight | [1.0, 1.3] (혐오 클래스 가중치 부여) |
| Threshold | 0.5 |
| Framework | PyTorch, HuggingFace Transformers |

## 📂 폴더 구조

```text
hatespeech-bd-project/
├── notebooks/
│   ├── baseline/
│   │   └── kocoh_baseline.ipynb              # 댓글 단독 baseline (BERT/RoBERTa) 학습
│   ├── berta/
│   │   └── kocoh_proposal_bert.ipynb         # BERT 기반 context+comment 제안 모델
│   ├── roberta/
│   │   └── kocoh_proposal_roberta_context_split (1).ipynb  # RoBERTa 기반 context+comment 제안 모델
│   └── kocoh_baseline/                       # baseline 실험 중간 산출물(csv) 모음
├── src/
│   ├── KOCOH_v.2.csv                         # 원천 데이터셋
│   ├── team_train_context_split.csv          # 학습 데이터 (context 기준 split)
│   ├── team_valid_context_split.csv          # 검증 데이터
│   ├── team_test_context_split.csv           # 테스트 데이터
│   └── visualize_all.ipynb                   # 전체 결과 시각화 노트북
├── result/
│   ├── baseline_result.csv                   # baseline 모델 성능
│   ├── proposal_result_roberta_context_split.csv  # 제안 모델(RoBERTa) 성능
│   ├── final_compare_result.csv              # 전체 모델 성능 비교
│   └── all_metrics_comparison.png            # 성능 비교 그래프
├── requirements.txt
└── README.md
```

## 🗂 데이터셋 (KOCOH)

`src/KOCOH_v.2.csv`는 한국어 온라인 게시글(context)과 그에 달린 댓글(comment)에 혐오 표현 여부를 라벨링한 데이터셋입니다. 주요 컬럼은 다음과 같습니다.

| 컬럼 | 설명 |
|---|---|
| `Context` | 원문 게시글/기사 내용 |
| `Comment` | 댓글 내용 |
| `Hate speech` | 혐오 표현 여부 (0: 비혐오, 1: 혐오) |
| `Counter speech` | 반박/대응 표현 여부 |
| `Gender`, `Disability`, `Race/Nationality`, `Region (Korea)`, `Age` | 혐오 표현의 세부 대상 카테고리 |
| `Profanity` | 욕설 포함 여부 |

노트북에서는 이를 `context`, `comment`, `label`, `type` 컬럼 형식으로 정리해 사용합니다.

> ⚠️ 데이터셋에는 실제 혐오/차별 표현이 포함되어 있습니다. 연구 및 모델 개선 목적으로만 사용해 주세요.

## 🛠 실행 방법

노트북은 Google Colab 환경 기준으로 작성되었습니다.

1. 리포지토리를 clone 하거나 원하는 노트북을 Colab에서 엽니다.
   ```bash
   git clone https://github.com/alovelovea/hatespeech-bd-project.git
   ```
2. Colab 상단 메뉴에서 **런타임 → 런타임 유형 변경 → GPU(T4 권장)** 로 설정합니다.
3. 노트북 상단 안내에 따라 필요한 데이터 파일(`KOCOH_v.2.csv`, `team_train.csv`, `team_valid.csv`, `team_test.csv` 등)을 Colab 세션에 업로드합니다.
4. 셀을 순서대로 실행합니다.
   - Baseline 실험: `notebooks/baseline/kocoh_baseline.ipynb`
   - BERT 제안 모델: `notebooks/berta/kocoh_proposal_bert.ipynb`
   - RoBERTa 제안 모델: `notebooks/roberta/kocoh_proposal_roberta_context_split (1).ipynb`
5. 학습이 끝나면 `result/` 폴더에 성능 지표(csv)가 저장되고, `src/visualize_all.ipynb`로 전체 결과를 비교/시각화할 수 있습니다.

### 로컬에서 실행할 경우

```bash
pip install pandas scikit-learn transformers torch
```

## 📎 참고

- KOCOH 데이터셋 및 논문 성능(LLM 평균 F1 0.6073)은 `result/final_compare_result.csv`의 비교 기준으로 사용되었습니다.
- 프로젝트 진행 중 생성된 중간 실험 파일은 `notebooks/kocoh_baseline/` 폴더에 남아있습니다.

## 👥 팀

빅데이터 프로젝트 수업의 일환으로 진행되었습니다.

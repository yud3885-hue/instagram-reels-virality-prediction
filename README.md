# 📱 Instagram Reels 바이럴 예측 (Virality Prediction)

> **Data Science** 최종 팀프로젝트 · **Team 6** · Ewha Womans University (Dept. of AI)
> 5개 분류 모델을 비교하고, **"왜 바이럴 예측이 어려웠는가"** 를 데이터 관점에서 규명한 프로젝트

![Python](https://img.shields.io/badge/Python-3.10-blue)
![sklearn](https://img.shields.io/badge/scikit--learn-LR%20%7C%20DT%20%7C%20RF%20%7C%20AdaBoost-orange)
![XGBoost](https://img.shields.io/badge/XGBoost-tabular-red)
![Task](https://img.shields.io/badge/Task-Binary%20Classification-green)
![Result](https://img.shields.io/badge/ROC--AUC-%E2%89%88%200.5-lightgrey)

> 💡 **핵심 결론**: *"The limitation was not the model — it was the dataset."*
> 알고리즘을 바꿔도 성능이 오르지 않았고, 원인은 **데이터셋 자체의 한계**였음.

---

## 📖 프로젝트 소개

- **주제**: Instagram Reels가 **바이럴(viral)** 될지 여부를 예측하는 이진 분류
- **목표**: 다양한 분류 모델을 **공정하게 비교**하고, 예측이 어려웠던 **근본 원인 규명**
- **강좌**: Data Science (Instructor: Hyunsoo Cho) — Spring 2026 Final Project
- **접근의 차별점**: 단순 성능 경쟁이 아니라, **"모델이 실패한 이유"를 EDA로 역추적**한 분석형 프로젝트

---

## 🎯 문제 정의 & 데이터

- **문제 유형**: 이진 분류 (Viral `1` / Non-Viral `0`)
- **타겟 정의**: `virality_score >= 80` → Viral (1), 그 외 → Non-Viral (0) — Viral 비율 약 **20%**
- **데이터셋**: Kaggle [*What Makes an Instagram Reel Go Viral*](https://www.kaggle.com/datasets/deepeshkansotia/what-makes-an-instagram-reel-go-viral)
- **규모**: **100,000 samples × 26 features** (1행 = 1 Reel)

### 변수 카테고리

| 카테고리 | 예시 변수 |
| --- | --- |
| Creator Information | follower count |
| Content Characteristics | reel length, caption length, hashtag count |
| Viewer Behavior | average watch time, retention time, rewatch rate |
| Exposure Signals | non-follower reach ratio, trending audio, explore boost |

---

## 🛠 Tech Stack

### 🤖 Models

- **scikit-learn**: Logistic Regression, Decision Tree, Random Forest, AdaBoost
- **XGBoost**: Gradient Boosting (tabular)

### 📊 Data & Viz

- pandas / numpy
- matplotlib / seaborn

### ☁️ Environment

- Python 3.10+
- Google Colab / Jupyter Notebook

---

## 🧹 전처리 — 공통 파이프라인

> **왜 공통 전처리인가?** 모델마다 다른 전처리를 쓰면 성능 차이가 *알고리즘* 때문인지 *전처리* 때문인지 구분 불가.
> → 모든 모델에 **동일 데이터셋 · 동일 결측치 처리 · 동일 Split · 동일 seed(42)** 적용해 **공정 비교** 확보.

- **Data Leakage 변수 제거**: 업로드 *이후*에 생성되는 성과 지표 → 예측 시점에 알 수 없음
  `likes`, `comments`, `shares`, `saves`, `impressions`, `reach`, `engagement_rate`
- **ID 변수 제거**: `reel_id`, `creator_id` — 각 100,000개 고유값(High Cardinality), 예측 정보 없음 + One-Hot 시 메모리 폭증
- **결측치 처리**: 수치형 → `SimpleImputer(strategy='median')`
- **범주형**: ID 제거 후 남은 범주형 없음 → One-Hot 미수행
- **Split**: Stratified train/test split, `random_state=42`

---

## 🤖 모델별 결과 (Threshold = 80)

| 모델 | Accuracy | F1 | ROC-AUC | 주요 발견 |
| --- | --- | --- | --- | --- |
| Logistic Regression | 0.5026 | 0.2896 | 0.4997 | 선형 신호 거의 없음 |
| Decision Tree | 0.6767 | 0.2076 | 0.5008 | class weight로 Recall 개선 |
| AdaBoost | 0.2026 | 0.3365 | 0.4999 | Viral 과다 예측 (false positive) |
| Random Forest | 0.7976 | ≈ 0.00 | ≈ 0.50 | 다수 클래스로 편향 |
| XGBoost | Low | Low | 0.5110 | 명확한 경계 학습 실패 |

- **Logistic Regression Best**: `C=0.1`, `penalty=L1`, `class_weight=balanced` (5-fold Stratified CV)
- **Decision Tree Best**: `criterion=entropy`, `max_depth=7`, `min_samples_leaf=16`
- ⚠️ **공통 현상**: 모델 종류(선형·트리·부스팅)와 무관하게 **ROC-AUC ≈ 0.5** → 사실상 랜덤 추측 수준

---

## 🔬 추가 실험 — Threshold 조정 (80 → 60)

- 가설: *"진짜 문제는 심한 클래스 불균형 아닐까?"*
- `virality_score` 임계값을 80 → 60으로 낮춰 Viral 비율 **20% → 40%** 로 완화
- **결과**: 여러 모델에서 **F1은 개선**되었으나, **ROC-AUC는 여전히 ≈ 0.5**
- **해석**: 균형을 맞추면 점수는 오르지만 **판별력(discrimination)은 개선되지 않음** → 불균형이 근본 원인이 아님

---

## 📊 Advanced EDA — 왜 모델이 실패했나

데이터를 다시 파고들어 **성능 저조의 원인**을 규명함.

- **팔로워 규모**: 팔로워가 많아도 평균 Virality Score 차이는 미미 → `팔로워 수 ≠ 바이럴`
- **콘텐츠 특성**: 영상 길이·해시태그 수의 그룹 간 차이 무시할 수준
- **시청 행동**: Viral의 Retention이 높지만 **분포가 크게 중첩** → 단일 변수로 구분 불가
- **상관 분석**: 대부분 약한 상관, Virality와 강한 선형 관계를 갖는 변수 없음

### 🧩 근본 원인 3가지

1. **심한 클래스 불균형** (Viral 20% vs Non-Viral 80%)
2. **클래스 간 분포 중첩** (Viral/Non-Viral의 feature 범위가 유사)
3. **낮은 변수 분리력** (명확한 decision boundary 부재)

### 📉 Data Leakage 제거로 인한 정보 손실

- 제거된 변수(`likes`, `comments`, `shares`, `saves`, `reach`, `impressions`, `engagement_rate`)가
  사실은 **바이럴 예측의 가장 강한 신호**였음
- 즉, **누수를 막는 과정에서 예측력의 핵심 정보도 함께 사라짐** → 남은 메타데이터만으로는 설명 부족

---

## 💡 결론

| # | Finding |
| --- | --- |
| 1 | 대부분의 변수는 설명력이 낮았다 |
| 2 | Viral / Non-Viral 분포가 크게 중첩되었다 |
| 3 | 클래스 불균형이 예측을 더 어렵게 만들었다 |
| 4 | 중요한 **콘텐츠 수준 정보**가 데이터셋에 없었다 |

> **The limitation was not the model. The limitation was the dataset.**

---

## 🚀 향후 개선 방안

- **상호작용 피처**: `creator_followers × explore_page_boost → explore_effect_score`
- **콘텐츠 품질 지표**: `hook_strength + video_quality + editing_quality → content_quality_index`
- **시청 행동 통합 지표**: `retention + completion + rewatch → watch_behavior_score`
- **근본적으로**: 영상 내용·주제·감정·스토리텔링 등 **콘텐츠 자체 정보** 확보 필요

---

## 📂 프로젝트 구조

```
instagram-reels-virality-prediction/
├── README.md                      # 프로젝트 문서 (본 파일)
├── notebooks/
│   ├── ml_models_all.ipynb              # 5개 모델 통합 (LR·DT·RF·AdaBoost·XGBoost)
│   ├── logistic_regression.ipynb        # LR 상세 분석 (결과·그래프 포함)
│   └── logistic_regression_annotated.ipynb  # LR + 강의 개념 매핑판
├── docs/
│   ├── project_description.pdf     # 과제 안내
│   ├── proposal.pdf               # 프로젝트 제안서
│   ├── preprocessing_notes.pdf    # 공통 전처리 근거 정리
│   ├── presentation.pdf           # 최종 발표자료 (31 slides)
│   └── presentation_script.pdf    # 발표 스크립트 (Slides 8–18)
├── data/
│   └── README.md                  # 데이터 안내 (Kaggle, 원본 미포함)
├── requirements.txt
├── .gitignore
└── _posts/
    └── 2026-06-17-instagram-reels-virality-prediction.md   # 깃허브 블로그 포스트
```

---

## 👥 팀원 및 역할

> Team 6 · Data Science (Spring 2026)

| 팀원 | 담당 역할 |
| --- | --- |
| Dayeon Yu | < 데이터 전처리, Logistic Regression, Advanced EDA , 튜닝, 모델 비교, 발표 > |
| Jii Kang | < 데이터 전처리 , Random Forest , 모델 비교, 튜닝, 발표 > |
| Rian Kang | < 데이터 전처리,  AdaBoost , 모델 비교, Advanced EDA , 튜닝, ppt 제작 > |
| Afina Khairullina | < 데이터 전처리, XGBoost , 발표 > |
| Seohyon Lee | < Decision Tree, ppt 제작  > |



---

## 🏃 실행 방법

```bash
# 1. 저장소 클론
git clone https://github.com/DAYEON/instagram-reels-virality-prediction.git
cd instagram-reels-virality-prediction

# 2. 의존성 설치
pip install -r requirements.txt

# 3. 데이터 내려받기
#    Kaggle에서 데이터셋 다운로드 후 data/ 폴더에 배치 (data/README.md 참고)

# 4. 노트북 실행
jupyter notebook notebooks/ml_models_all.ipynb
```

> ⚠️ 노트북은 Google Colab 기준으로 작성되어 있습니다.
> 로컬 실행 시 상단 `drive.mount(...)` 및 데이터 경로를 로컬 경로로 수정하세요.

---

## 📄 산출물 & 문서

- 📓 [`notebooks/ml_models_all.ipynb`](./notebooks/ml_models_all.ipynb) — 5개 모델 전체 파이프라인
- 📈 [`notebooks/logistic_regression.ipynb`](./notebooks/logistic_regression.ipynb) — LR 상세 (결과·그래프)
- 📑 [`docs/presentation.pdf`](./docs/presentation.pdf) — 최종 발표자료
- 📋 [`docs/proposal.pdf`](./docs/proposal.pdf) — 제안서

---

<div align="center">

**Data Science · Team 6 · Ewha W. University · 2026**

</div>

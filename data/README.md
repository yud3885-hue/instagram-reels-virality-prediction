# 📁 데이터 안내

원본 데이터셋은 용량 문제로 **저장소에 포함되어 있지 않습니다.**
아래 Kaggle에서 직접 내려받아 이 폴더(`data/`)에 배치하세요.

## 데이터셋

- **이름**: What Makes an Instagram Reel Go Viral
- **출처**: [Kaggle — deepeshkansotia/what-makes-an-instagram-reel-go-viral](https://www.kaggle.com/datasets/deepeshkansotia/what-makes-an-instagram-reel-go-viral)
- **규모**: 100,000 rows × 26 columns
- **타겟 생성**: `virality_score >= 80` → Viral(1), 그 외 → Non-Viral(0)

## 다운로드 방법

### 방법 1 — 웹에서 직접

1. 위 Kaggle 링크 접속 → **Download** 클릭
2. 압축 해제 후 `.csv` 파일을 `data/` 폴더에 배치

### 방법 2 — Kaggle API

```bash
# Kaggle API 토큰(kaggle.json) 설정 후
pip install kaggle
kaggle datasets download -d deepeshkansotia/what-makes-an-instagram-reel-go-viral
unzip what-makes-an-instagram-reel-go-viral.zip -d data/
```

> ⚠️ **주의**: 예측 시점에 알 수 없는 성과 지표(`likes`, `comments`, `shares`, `saves`,
> `impressions`, `reach`, `engagement_rate`)는 **Data Leakage 방지를 위해 전처리에서 제거**됩니다.

# 💄 올리브영 리뷰 분석을 통한 피부 고민별 맞춤 화장품 추천 프로젝트
> **KoELECTRA 기반의 감성 분석 및 BERTopic 토픽 모델링 활용**

![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white) ![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-EE4C2C?logo=pytorch&logoColor=white) ![HuggingFace](https://img.shields.io/badge/Transformers-4.0%2B-FFD21E?logo=huggingface&logoColor=black) ![Selenium](https://img.shields.io/badge/Selenium-4.0%2B-43B02A?logo=selenium&logoColor=white) 

## 📌 1. 프로젝트 개요 (Overview)
* **프로젝트명**: 올리브영 리뷰 데이터 분석을 통한 맞춤형 스킨케어 인사이트 도출
* **대용량 데이터 저장소**: [Google Drive 링크](https://drive.google.com/drive/folders/16iDWNRGRV3p_oI1JLGJ9MXyGEnriKlk8?hl=ko)

### 🎯 기획 의도
기존의 단순 별점(1~5점) 시스템은 제품의 **구체적인 효능(속건조 해결, 좁쌀 여드름 진정 등)**과 **부작용**을 파악하기 어렵다는 한계가 있습니다.
본 프로젝트는 **약 13만 건의 리뷰 데이터**를 딥러닝 모델(**KoELECTRA**)로 분석하여, 사용자의 세밀한 피부 고민에 맞는 최적의 제품을 추천하고 시각화된 인사이트를 제공합니다.

---

## 🛠️ 2. 시스템 아키텍처 및 기술 스택 (System Architecture)

### 🏗️ Data Pipeline
1.  **Data Collection**: `Selenium`을 이용한 동적 웹 크롤링 (무한 스크롤 처리)
2.  **Preprocessing**: 정규표현식(Regex)을 이용한 특수문자 제거 및 `Pandas` 기반 데이터 정제
3.  **Labeling**: 생성형 AI(LLM) 초안 작성 + Human-in-the-loop 검수를 통한 고품질 데이터셋(2,500건) 구축
4.  **Modeling**: `KoELECTRA` Fine-tuning (Classification) & `BERTopic` (Clustering)

### 💻 Tech Stack
| 구분 | 기술 및 라이브러리 | 상세 내용 |
| :--- | :--- | :--- |
| **수집 (Crawling)** | `Selenium`, `BeautifulSoup` | 동적 페이지 크롤링, 리뷰/평점/피부타입 수집 |
| **전처리 (Preprocessing)** | `Pandas`, `NumPy`, `OpenPyXL` | 결측치 처리, 노이즈 제거, 엑셀/CSV 변환 |
| **모델링 (Model)** | **`KoELECTRA`** | `monologg/koelectra-base-v3-discriminator` |
| **분석 (Analysis)** | **`BERTopic`** | 토픽 모델링, 키워드 추출 (c-TF-IDF) |
| **시각화 (Vis)** | `Matplotlib`, `Seaborn`, `WordCloud` | 파이차트, 히트맵, 워드클라우드 시각화 |
| **환경 (Env)** | `Jupyter Notebook`, `CUDA 12.1` | NVIDIA RTX 4070 GPU 가속 활용 |

---

## 📊 3. 상세 분석 프로세스 (Analysis Process)

### ① 데이터 수집 및 전처리 (Data Collection)
* **Target**: 올리브영 '크림/젤' 카테고리 상위 제품
* **Volume**: Raw Data 약 **130,000건** 확보
* **Cleaning**: 특수문자, 이모티콘 제거 및 중복 리뷰 삭제

### ② 데이터 분포 (Data Distribution)
수집된 데이터를 분석한 결과, 소비자들이 제품 선택 시 가장 중요하게 고려하는 요소는 **'보습'**과 **'진정'** 키워드인 것으로 나타났습니다.

![Data Distribution](notebooks/results/distribution_pie_chart.png)

### ③ 모델 학습 (Model Training)
한국어 문맥 이해에 특화된 **KoELECTRA-Base-V3** 모델을 사용하여 5가지 카테고리(보습, 진정, 사용감, 자극, 기타)로 리뷰를 분류했습니다.

* **Hyperparameters**:
    * `Epochs`: 3
    * `Batch Size`: 16
    * `Learning Rate`: 2e-5
    * `Max Length`: 128
    * `Optimizer`: AdamW

| Epoch | Train Loss | Train Accuracy | Validation Loss | Validation Accuracy |
| :---: | :---: | :---: | :---: | :---: |
| 1 | 1.3339 | 43.55% | 1.0890 | 63.87% |
| 2 | 0.8286 | 74.85% | 0.7000 | 76.95% |
| **3** | **0.6122** | **79.05%** | **0.5684** | **81.05%** |

> **Result**: 3 Epoch 학습 후 Validation Accuracy **81.05%** 달성. 혼동 행렬(Confusion Matrix) 분석 결과, 특히 '보습'과 '진정' 클래스 간의 분류 성능이 우수함을 확인했습니다.

![Confusion Matrix](notebooks/results/confusion_matrix.png)

### ④ 토픽 모델링 (BERTopic)
분류된 리뷰 내에서 구체적인 소비자 언어를 파악하기 위해 **BERTopic**을 적용했습니다.

| **Topic: 진정/트러블 (Soothing)** | **Topic: 보습/계절 (Moisture)** |
| :---: | :---: |
| ![Topic 0](notebooks/results/wordcloud_topic_0.png) | ![Topic 1](notebooks/results/wordcloud_topic_1.png) |
| **Key**: `좁쌀`, `붉은기`, `마스크`, `뒤집어짐` | **Key**: `속건조`, `겨울`, `악건성`, `당김` |

---

## 🏆 4. 최종 분석 결과 (Final Insights)

단순 평점이 아닌, **모델이 예측한 리뷰의 긍정/부정 확률과 키워드 빈도**를 종합하여 피부 고민별 베스트 제품을 선정했습니다.

### 🌿 [진정/트러블 케어] BEST Product
* **1위: 닥터지 레드 블레미쉬 클리어 수딩크림**
    * *선정 사유*: '좁쌀', '진정' 키워드와의 연관도가 가장 높으며, 트러블 완화에 대한 긍정 리뷰 비율 압도적.

![Soothing Ranking](notebooks/results/ranking_soothing.png)

### 💧 [보습/수분 케어] BEST Product
* **1위: 피지오겔 레드 수딩 AI 크림**
    * *선정 사유*: '속건조', '겨울철' 키워드에서 가장 높은 점수를 획득, 악건성 피부 타입 사용자에게 강력 추천.

![Moisture Ranking](notebooks/results/ranking_moisture.png)

---

## 📂 5. 프로젝트 구조 (Directory Structure)

```bash
📦 Cosmetics_Project
 ┣ 📂 data
 ┃ ┣ 📂 01_raw             # Selenium으로 수집된 원본 데이터 (reviews_raw.csv)
 ┃ ┣ 📂 02_processed       # 전처리 및 라벨링 완료 데이터
 ┃ ┗ 📂 03_final           # 모델 학습용 최종 데이터셋 (train/val split)
 ┣ 📂 models
 ┃ ┗ 📂 koelectra_finetuned # Fine-tuning된 모델 가중치 (.pt)
 ┣ 📂 notebooks
 ┃ ┣ 📜 01_Crawling.ipynb       # 올리브영 리뷰 크롤링 코드
 ┃ ┣ 📜 02_Preprocessing.ipynb  # 데이터 정제 및 라벨링 전처리
 ┃ ┣ 📜 03_Model_KoELECTRA.ipynb # KoELECTRA 모델 학습 및 평가 (Main)
 ┃ ┣ 📜 04_Inference.ipynb      # 신규 데이터 추론 및 분석
 ┃ ┣ 📜 05_Topic_Modeling.ipynb # BERTopic 기반 토픽 추출
 ┃ ┗ 📜 06_Visualization.ipynb  # 결과 시각화 및 그래프 생성
 ┣ 📂 notebooks/results         # README에 사용된 시각화 이미지 리소스
 ┗ 📜 README.md                 # 프로젝트 문서

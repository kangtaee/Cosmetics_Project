# 💄 올리브영 리뷰 분석을 통한 피부 고민별 맞춤 화장품 추천 프로젝트

![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white) ![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-EE4C2C?logo=pytorch&logoColor=white) ![HuggingFace](https://img.shields.io/badge/Transformers-4.0%2B-FFD21E?logo=huggingface&logoColor=black) ![Selenium](https://img.shields.io/badge/Selenium-4.0%2B-43B02A?logo=selenium&logoColor=white)

## 1. 서론 (Introduction)
### 1.1. 프로젝트 배경 및 목적
기존의 화장품 쇼핑몰은 대부분 **단순 별점(1~5점)** 시스템에 의존하고 있다. 하지만 별점만으로는 제품의 **구체적인 효능(속건조 해결, 좁쌀 여드름 진정 등)**이나 사용자별 **부작용(따가움, 트러블)**을 파악하기 어렵다.
본 프로젝트는 **약 13만 건의 방대한 리뷰 데이터**를 딥러닝(KoELECTRA)으로 분석하여, 사용자의 세밀한 피부 고민에 맞는 최적의 제품을 추천하고 인사이트를 도출하는 데 목적이 있다.

### 1.2. 분석 범위
* **대상**: 올리브영 온라인몰 '크림/젤' 카테고리 상위 베스트셀러 제품
* **데이터**: 실제 구매자의 리뷰 텍스트, 평점, 피부 타입 등 비정형 데이터

---

## 2. 데이터 수집 (Data Collection)
### 2.1. 수집 방법
올리브영 리뷰 페이지는 동적 웹 페이지(Dynamic Web Page)로 구성되어 있어 정적 크롤링(BeautifulSoup)만으로는 수집이 불가능하다. 따라서 **Selenium**을 활용하여 웹 브라우저를 직접 제어하는 방식을 채택했다.

* **페이지네이션(Pagination) 처리**: 리뷰가 10개 단위로 로드되는 점을 고려하여, 페이지 버튼을 순차적으로 클릭하며 `WebDriverWait`를 통해 로딩을 대기하는 로직을 구현했다.
* **예외 처리**: 품절 상품이나 리뷰가 없는 경우 크롤러가 중단되지 않도록 `try-except` 구문을 적용했다.

### 2.2. 데이터 저장소
수집된 원본 데이터(Raw Data)는 용량 문제로 인해 별도의 구글 드라이브에 저장하였다.
* **대용량 데이터 저장소**: [Google Drive 링크](https://drive.google.com/drive/folders/16iDWNRGRV3p_oI1JLGJ9MXyGEnriKlk8?hl=ko)
* **크롤링 소스코드**: [`notebooks/01_Crawling.ipynb`](notebooks/01_Crawling.ipynb)

---

## 3. 데이터 라벨링 (Data Labeling)
### 3.1. 고품질 학습 데이터 구축
단순 키워드 매칭(Rule-based)의 한계를 넘기 위해, 문맥을 이해하는 AI 모델 학습용 데이터를 직접 구축했다. **Semi-supervised Learning** 방식을 도입하여 효율성을 높였다.

1.  **1차 분류 (LLM 활용)**: 생성형 AI를 활용하여 리뷰 텍스트의 주된 감성 및 의도를 5가지 카테고리로 1차 분류했다.
2.  **2차 검수 (Human-in-the-loop)**: AI가 분류한 데이터 중 신뢰도가 낮거나 모호한 데이터를 사람이 직접 검수하여 오분류를 수정했다.
3.  **최종 데이터셋**: 총 **2,500건**의 고품질 라벨링 데이터 확보 (Train 2,000 / Validation 500).

* **분류 기준 (Class)**: `보습`, `진정`, `사용감`, `자극`, `기타`

---

## 4. 탐색적 데이터 분석 (EDA & Preprocessing)
### 4.1. 데이터 분포 시각화
수집된 전체 리뷰를 분석한 결과, 소비자들이 제품 선택 시 가장 중요하게 고려하는 요소는 **'보습'**과 **'진정'** 키워드로 나타났다.

![Data Distribution](notebooks/results/distribution_pie_chart.png)

### 4.2. 데이터 전처리
* **노이즈 제거**: 정규표현식(Regex)을 사용하여 특수문자, 이모티콘, 의미 없는 자음/모음(ㅋㅋ, ㅠㅠ)을 제거하여 모델의 학습 효율을 높였다.
* **중복 제거**: 동일 사용자의 반복 리뷰 및 광고성 스팸 리뷰를 필터링했다.

### 4.3. 학습 데이터 추출 기준
전체 데이터 중 각 카테고리(Class)별 데이터 불균형(Imbalance)을 해소하기 위해, **층화 추출(Stratified Sampling)** 방식을 적용했다. 이를 통해 특정 클래스에 모델이 편향되는 것을 방지했다.

---

## 5. 학습 결과 (Model Training Results)
### 5.1. 모델 성능 평가
한국어 문맥 이해에 특화된 **KoELECTRA-Base-V3** 모델을 파인튜닝(Fine-tuning)했다.

* **Epoch별 정확도 추이**

| Epoch | Train Loss | Train Accuracy | Validation Loss | Validation Accuracy |
| :---: | :---: | :---: | :---: | :---: |
| 1 | 1.3339 | 43.55% | 1.0890 | 63.87% |
| 2 | 0.8286 | 74.85% | 0.7000 | 76.95% |
| **3** | **0.6122** | **79.05%** | **0.5684** | **81.05%** |

### 5.2. 결과 분석
최종적으로 **81.05%**의 검증 정확도를 달성했다. 아래 혼동 행렬(Confusion Matrix)을 보면, 특히 혼동하기 쉬운 **'보습'**과 **'진정'** 카테고리 간의 미세한 차이도 효과적으로 구분해냈음을 확인할 수 있다.

![Confusion Matrix](notebooks/results/confusion_matrix.png)

---

## 6. 결론 및 심화 분석 (Conclusion)
### 6.1. 토픽 모델링 (BERTopic) 분석
단순 분류를 넘어, 각 카테고리 내부의 세부 주제를 파악하기 위해 **BERTopic** 군집 분석을 수행했다.

| **Topic:  보습/계절 케어** | **Topic: 진정/트러블 케어** |
| :---: | :---: |
| ![Topic 0](notebooks/results/wordcloud_topic_0.png) | ![Topic 1](notebooks/results/wordcloud_topic_1.png) |
| **핵심 키워드**:  `속건조`, `겨울`, `악건성`  | **핵심 키워드**: `좁쌀`, `붉은기`, `마스크` |

* **분석**: '진정' 카테고리에서는 구체적인 트러블 양상(좁쌀, 붉은기)이 주로 언급되었으며, '보습' 카테고리에서는 피부 타입(악건성)과 계절(겨울)이 밀접하게 연관됨을 확인했다.

### 6.2. 최종 제품 추천 랭킹 (Ranking)
단순 별점순이 아닌, 모델이 분석한 **'리뷰 긍정 확률'**과 **'키워드 연관도'**를 종합하여 피부 고민별 BEST 제품을 선정했다.

* **[진정 케어] 1위: 닥터지 레드 블레미쉬 클리어 수딩크림**
    ![Soothing Ranking](notebooks/results/ranking_soothing.png)
    * **근거**: 트러블 진정 관련 키워드 비중이 압도적으로 높으며, 민감성 피부 사용자들의 긍정 평가가 주를 이룸.

* **[보습 케어] 1위: 피지오겔 레드 수딩 AI 크림**
    ![Moisture Ranking](notebooks/results/ranking_moisture.png)
    * **근거**: '속건조' 해결에 대한 만족도가 가장 높았으며, 겨울철 보습력 유지 항목에서 최고점을 기록함.

### 6.3. 프로젝트의 의의
본 프로젝트를 통해 리뷰 텍스트에 숨겨진 사용자들의 진짜 니즈를 데이터로 증명했다. 향후 이 모델을 활용한다면 쇼핑몰에서 **"내 피부 고민(좁쌀여드름) 필터"**와 같은 맞춤형 검색 기능을 구현하여 사용자 경험(UX)을 획기적으로 개선할 수 있을 것이다.

---

## 📂 폴더 구조 (Directory Structure)

```bash
📦 Cosmetics_Project
 ┣ 📂 data
 ┃ ┣ 📂 01_raw             # 원본 데이터 (Google Drive 저장)
 ┃ ┣ 📂 02_processed       # 전처리 완료 데이터
 ┃ ┗ 📂 03_final           # 학습용 최종 데이터
 ┣ 📂 models
 ┃ ┗ 📂 koelectra_finetuned # 학습 모델 가중치
 ┣ 📂 notebooks
 ┃ ┣ 📜 01_Crawling.ipynb       # 데이터 수집 코드
 ┃ ┣ 📜 02_Preprocessing.ipynb  # 전처리 및 라벨링
 ┃ ┣ 📜 03_Model_KoELECTRA.ipynb # 모델 학습 및 평가
 ┃ ┣ 📜 05_Topic_Modeling.ipynb # 토픽 모델링 (BERTopic)
 ┃ ┗ 📜 06_Visualization.ipynb  # EDA 및 시각화
 ┗ 📜 README.md                 # 프로젝트 보고서

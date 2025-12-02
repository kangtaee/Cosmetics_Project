# 💄 올리브영 리뷰 분석을 통한 피부 고민별 맞춤 화장품 추천 프로젝트
> **"별점 5점이라 샀는데 왜 트러블이 날까?"**
> 단순 평점이 아닌, **13만 건의 리뷰 텍스트**를 분석하여 **'진짜 보습/진정 효능'**을 찾아주는 AI 프로젝트다.

![Python](https://img.shields.io/badge/Python-3.8%2B-blue?logo=python&logoColor=white) ![PyTorch](https://img.shields.io/badge/PyTorch-2.0%2B-EE4C2C?logo=pytorch&logoColor=white) ![HuggingFace](https://img.shields.io/badge/Transformers-4.0%2B-FFD21E?logo=huggingface&logoColor=black) ![Selenium](https://img.shields.io/badge/Selenium-4.0%2B-43B02A?logo=selenium&logoColor=white)

## 1. 서론 (Introduction)
### 1.1. 프로젝트 배경 및 목적
현대 화장품 시장에서 소비자는 수많은 제품 홍수 속에 살고 있다. 구매 의사결정에 가장 큰 영향을 미치는 것은 '리뷰'와 '평점'이지만, 기존 이커머스 플랫폼의 **'별점(Star Rating)' 시스템은 구체적인 제품의 효능을 대변하지 못하는 한계**가 있다. 별점 5점짜리 제품이라도 "촉촉하지만 진정 효과는 없다"는 부정적인 텍스트가 섞여 있을 수 있기 때문이다.

본 프로젝트는 이러한 문제의식에서 출발하여, 올리브영의 방대한 텍스트 리뷰를 딥러닝(KoELECTRA)으로 분석하고, **단순 별점이 아닌 '실제 피부 고민 해결 능력'에 기반한 새로운 랭킹 시스템**을 제안하는 데 목적이 있다.

### 1.2. 분석 범위
* **대상**: 올리브영 온라인몰 '크림/젤' 카테고리 상위 베스트셀러 제품
* **데이터**: 총 **130,456건**의 텍스트 리뷰의 실제 구매자의 리뷰 텍스트, 평점, 피부 타입 등 비정형 데이터
* **핵심 분석 클래스**: 보습(Moisture) vs 진정(Soothing)

---

## 2. 데이터 수집 (Data Collection)
### 2.1 수집 방법 및 도구
올리브영 리뷰 페이지는 동적 웹 페이지(Dynamic Web Page)로 구성되어 있어 정적 크롤링(BeautifulSoup)만으로는 수집이 불가능하다. 따라서 **Selenium**을 활용하여 웹 브라우저를 직접 제어하는 방식을 채택했다.

* **소스코드**: `notebooks/01_Crawling.ipynb` (상세 코드 포함)
* **페이지네이션(Pagination) 처리**: 리뷰가 10개 단위로 로드되는 점을 고려하여, 페이지 버튼을 순차적으로 클릭하며 `WebDriverWait`를 통해 로딩을 대기하는 로직을 구현했다.
* **예외 처리**: 품절 상품이나 리뷰가 없는 경우 크롤러가 중단되지 않도록 `try-except` 구문을 적용했다.

### 2.2. 데이터 저장소
수집된 원본 데이터)는 용량 문제로 인해 별도의 구글 드라이브에 저장하였다.
* **대용량 데이터 저장소**: [Google Drive 링크](https://drive.google.com/drive/folders/16iDWNRGRV3p_oI1JLGJ9MXyGEnriKlk8?hl=ko)
* **크롤링 소스코드**: [`notebooks/01_Crawling.ipynb`](notebooks/01_Crawling.ipynb)

---

## 3. 데이터 라벨링 (Data Labeling)
### 3.1 라벨링 프로세스 상세
13만 건의 대용량 데이터를 효율적으로 처리하기 위해 **Semi-supervised Learning(준지도 학습)** 접근 방식을 채택했다.

1.  **1차 분류 (LLM 활용)**: 생성형 AI를 활용하여 리뷰 텍스트가 '보습' 위주인지 '진정' 위주인지 1차 태깅을 진행하여 시간과 비용을 획기적으로 절감했다.
2.  **신뢰도 필터링**: AI 모델의 분류 확신도(Confidence Score)가 낮은 데이터를 별도로 추출했다.
3.  **2차 검수 (Human-in-the-loop)**: 추출된 모호한 데이터 및 중복 데이터를 사람이 직접 검수하고 수정하여 최종 학습 데이터셋(Gold Standard)을 구축했다.
4.  **최종 데이터셋**: 총 **2,500건**의 고품질 라벨링 데이터 확보 (Train 2,000 / Validation 500).

---

## 4. 탐색적 데이터 분석 (EDA & Preprocessing)
### 4.1 데이터 분포 시각화
수집된 리뷰 데이터의 클래스 분포를 확인한 결과, '보습' 관련 리뷰가 '진정'보다 다소 많은 분포를 보였다. 이는 겨울철 크림 구매가 많은 계절적 요인이 반영된 것으로 해석된다.

![Data Distribution](notebooks/results/distribution_pie_chart.png)

### 4.2 데이터 전처리 및 학습 데이터 추출 기준
모델 성능을 극대화하기 위해 다음과 같은 전처리 과정을 거쳤다.
* **텍스트 정제**: 정규표현식(Regex)을 사용하여 한글, 영문, 숫자 외의 특수문자 및 이모티콘을 제거했다. (`[^가-힣a-zA-Z0-9]`)
* **중복 제거**: 동일 사용자의 반복 리뷰 및 광고성 스팸 리뷰를 필터링했다.

* **학습 데이터 추출 기준**: 클래스 불균형 문제를 해결하기 위해 학습(Train) 및 검증(Validation) 데이터 분할 시 **층화 추출(Stratified Sampling)** 기법을 적용하여 클래스 비율을 일정하게 유지했다.

---

## 5. 학습 결과 (Model Training Results)
### 5.1 Epoch별 학습 정확도 및 검증 정확도
한국어 문맥 이해에 특화된 **KoELECTRA-Base-V3** 모델을 파인튜닝(Fine-tuning)하였다. 아래 그래프는 Epoch 진행에 따른 모델의 학습 양상을 보여준다.

![학습 곡선](notebooks/results/training_curves.png)

* **분석**: 학습 손실(Train Loss)과 검증 손실(Validation Loss)이 안정적으로 감소하며 모델이 수렴하는 것을 확인할 수 있다. 특히 검증 정확도(Validation Accuracy)가 꾸준히 상승하여 과적합(Overfitting) 없이 학습이 잘 이루어졌다.
* 
* **Epoch별 정확도 추이**

| Epoch | Train Loss | Train Accuracy | Validation Loss | Validation Accuracy |
| :---: | :---: | :---: | :---: | :---: |
| 1 | 1.3339 | 43.55% | 1.0890 | 63.87% |
| 2 | 0.8286 | 74.85% | 0.7000 | 76.95% |
| **3** | **0.6122** | **79.05%** | **0.5684** | **81.05%** |

### 5.2 최종 성능 평가 (Confusion Matrix)
최종 검증 데이터셋에 대해 **정확도 81.05%**를 달성했다. 혼동 행렬을 통해 모델이 '보습'과 '진정'의 미세한 뉘앙스 차이를 효과적으로 구분하고 있음을 확인했다.

![혼동 행렬](notebooks/results/confusion_matrix.png)

---

## 6. 결론 및 심화 분석 (Conclusion)
### 6.1 토픽 모델링을 통한 심층 분석
단순 분류에서 나아가 **BERTopic**을 활용해 각 클래스 내부의 세부적인 주제를 도출했다.

| **Topic:  보습/계절 케어** | **Topic: 진정/트러블 케어** |
| :---: | :---: |
| ![Topic 0](notebooks/results/wordcloud_topic_0.png) | ![Topic 1](notebooks/results/wordcloud_topic_1.png) |
| **핵심 키워드**:  `속건조`, `겨울`, `악건성`  | **핵심 키워드**: `좁쌀`, `붉은기`, `마스크` |

* **분석**
- 진정(Soothing)**: '좁쌀', '여드름', '붉은기', '마스크' 등의 구체적 트러블 키워드가 군집화되었다.
- 보습(Moisture)**: '속건조', '악건성', '수분감' 등의 키워드가 주를 이루었다.

### 6.2 분류에 따른 시계열적 특성 (Time-Series Analysis)
화장품 도메인은 계절성이 매우 뚜렷하다. 본 프로젝트 데이터와 **네이버 데이터랩(DataLab)**의 검색 트렌드를 비교 분석한 결과, 뚜렷한 역의 상관관계를 발견했다.
* **겨울(11월~1월)**: '보습' 키워드 검색량 및 관련 리뷰 급증
* **여름(7월~8월)**: '진정' 키워드 검색량 및 관련 리뷰 급증
* **결론**: 본 모델을 서비스화할 때, 계절별로 랭킹 가중치를 다르게 적용(예: 겨울엔 보습 제품 우선 노출)함으로써 사용자 만족도를 극대화할 수 있다.
* 🟢보습
* 🔴진정
![시계열 트렌드](notebooks/results/스크린샷%202025-12-01%20204345.png)

### 6.3 최종 산출물: 효능 기반 맞춤 랭킹
모델의 예측 확률(Confidence Score)을 기반으로 제품별 **'효능 점수'**를 산출하여 최종 랭킹을 도출했다. 이는 단순 판매량 순위와는 다른, **"진짜 진정/보습 효과가 있는 제품"**을 찾는 소비자에게 실질적인 근거가 된다.

| **진정(Soothing) 랭킹** | **보습(Moisture) 랭킹** |
| :---: | :---: |
| ![진정 랭킹](notebooks/results/ranking_soothing.png) | ![보습 랭킹](notebooks/results/ranking_moisture.png) |

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

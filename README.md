# Opensource-FISH
# 🛡️ FISH: AI 기반 보이스피싱 및 스미싱 방지 시스템
> **"단순 화면 캡처 업로드"**만으로 피싱 문맥과 사칭 URL을 실시간 탐지하는 사회적 약자 배려형 AI 보안 솔루션

본 프로젝트는 지능화되는 타겟형 스미싱 범죄를 실시간 단계에서 차단하기 위해 비전 엔진(EasyOCR)의 이미지 전처리, 편집 거리(Levenshtein Distance) 추적, 그리고 **Dual AI Engine (TF-IDF MLP + KoBERT)** 가중치 결합을 통해 피싱 여부를 다각도로 교차 검증하는 시스템입니다.

---

## ✨ 1. 주요 기능 (Key Features)
* **📸 이미지 기반 피싱 위험도 산출 (%)**
  * 사용자가 의심스러운 문자 화면을 캡처하여 업로드하면, AI가 내포된 문맥을 분석하여 직관적인 위험도 수치(%)를 제공합니다.
* **🔗 텍스트 기반 사칭 URL 판별 및 증거 제시**
  * 문자열 내에 포함된 인터넷 주소(URL)를 정식 기관 도메인 DB와 비교·연산하여 사칭 주소 여부를 판별하고 경고 결과를 나타냅니다.
* **🖥️ 디지털 약자 맞춤형 Gradio 대시보드**
  * 복잡한 프로그램 설치 없이 작동하는 심플한 UI/UX를 통해 고령층 등 디지털 취약계층을 범죄로부터 안전하게 보호합니다.

---

## ⚙️ 2. 시스템 아키텍처 (System Pipeline)

시스템은 외부 AI API나 무거운 서버 인프라에 의존하지 않고, 오직 로컬 자원과 수학적 알고리즘을 기반으로 유기적인 파이프라인을 구축하여 동작합니다.

[캡처 이미지 업로드] ──► [EasyOCR 엔진] ──► [텍스트 영역 파싱] ──► [본문 & URL 분리]
│
┌───────────────────────────────────────────────────────────────────┘
▼                                       ▼
[Dual AI Engine Stage]                 [Deterministic Rule Stage]
① TF-IDF + Sequential MLP 신경망       ③ 화이트리스트 직접 매칭 (539개 도메인)

Char N-gram(2,4) 벡터화 연산         ④ Levenshtein Distance (편집 거리)

3-Layer Dense 신경망 가중치 추론     - 공식 도메인과의 수학적 철자 차이 계산
② KoBERT Sequence Classifier           - 교묘한 철자 변조 사칭 URL 차단

skt/kobert-base-v1 파인튜닝

한국어 특화 피싱 문맥 탐지 보완
│                                       │
└─────────────────┬─────────────────────┘
▼
[가중치 산정 알고리즘]
▼
[Gradio UI 최종 대시보드 출력]

---

## 🧱 3. 핵심 탐지 모델 소개 (Model Specifications)

### ① EasyOCR 기반 비정형 텍스트 추출 모듈
* 캡처 이미지의 해상도, 배경, 폰트에 따른 인식률 저하를 줄이기 위해 **Grayscale 변환 및 Threshold 적용 등 이미지 전처리** 기법을 거친 후 문자를 추출합니다.
* 추출된 데이터는 정규표현식(Regex) 및 URL 파싱 로직을 통하여 순수 본문 텍스트와 도메인 주소 영역으로 깔끔하게 정제 및 분리됩니다.

### ② Levenshtein Distance (편집 거리 알고리즘)
* 공식 도메인과 피싱 도메인의 철자 차이를 수학적으로 계산하는 문자열 매칭 알고리즘을 직접 구현했습니다.
* **539개의 정상 도메인 화이트리스트 DB**와 비교 연산하여, 미세하게 주소를 변조한 typo-squatting 사칭 행위를 완벽하게 잡아냅니다.

### ③ TF-IDF + Sequential MLP 신경망 모델
* 글자 단위의 조합 특성을 반영하기 위해 **Character N-gram(ngram_range=(2,4), max_features=500)** 방식을 적용하여 텍스트 데이터를 벡터로 바꿉니다.
* Keras `Sequential` 모델을 통해 은닉층 가중치를 배치 크기 8로 120 에포크 동안 조밀하게 학습시킨 신경망 모델(`Dense(16) -> Dense(8) -> Sigmoid`)로 피싱 문맥 확률을 안정적으로 계산합니다.

### ④ KoBERT 문맥 분류 Transformer 모델
* 한국어 특유의 피싱 문맥(지인 사칭, 청첩장 위장, 택배 배송 오류, 정부 지원금 빙자 등) 탐지 성능을 극대화하기 위해 `skt/kobert-base-v1` 사전학습 모델을 기반으로 파인튜닝을 진행했습니다.
* 데이터 파이프라인에서 정답 라벨 리스트를 `labels` 키값으로 강제 매핑하는 패치를 적용하여, Hugging Face `Trainer` 규격에서 손실값($\text{loss}$) 유실 없이 안정적으로 수렴하도록 구현했습니다.

---

## 📊 4. 모델 학습 및 평가 결과 (Model Evaluation)

순수 원본 데이터셋을 기반으로 로컬 빌드 및 학습을 진행한 결과, 데이터 손실이나 오버피팅 없이 우수한 수렴 성능과 정밀도를 확보했습니다.

### ① MLP 모델 학습 곡선 (Accuracy / Loss)
120 에포크 학습 진행 결과, Train 및 Validation 세트 모두에서 Loss가 안정적으로 하향 곡선을 그리며 Target 정확도에 도달했습니다.

<img width="1599" height="1211" alt="Image" src="https://github.com/user-attachments/assets/c59e978b-cfae-43b3-92b7-b9a71d4f098d" />

### ② MLP 피싱 탐지 혼동 행렬 (Confusion Matrix)
테스트 데이터셋 검증 결과, 피싱 위험(1) 클래스에 대해 26건을 정확히 찾아내고 단 1건의 오차만을 허용하며 높은 탐지 정밀도를 기록했습니다.

<img width="3420" height="1099" alt="Image" src="https://github.com/user-attachments/assets/51b9589b-4755-42fc-9b24-68fbd1c22596" />

---

## 🚀 5. 시작하기 (Getting Started)

### Prerequisites
프로젝트 실행을 위해 아래 라이브러리 설치가 필요합니다.
```bash
pip install transformers datasets accelerate evaluate scikit-learn pandas numpy matplotlib seaborn gradio easyocr torch tensorflow

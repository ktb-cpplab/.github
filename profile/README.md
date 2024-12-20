
## 👥 Team Members

## ☁️ Cloud
|      Jake(이태윤)    |       Navy(김소담)        |
| :-----: | :-----: |
| <img src="https://avatars.githubusercontent.com/u/127961622?v=4" width=120px alt="이태윤"/>  | <img src="https://avatars.githubusercontent.com/u/122079946?v=4" width=120px alt="김소담"/>  |
| [@rkfcl](https://github.com/rkfcl) | [@dadamji34](https://github.com/dadamji34) |

## 📚 Full-Stack
|      Cowee(이용우)    |       Katrina(이은경)        |
| :-----: | :-----: |
| <img src="https://avatars.githubusercontent.com/u/95459741?v=4" width=120px alt="이용우"/>  | <img src="https://avatars.githubusercontent.com/u/86763857?v=4" width=120px alt="이은경"/>  |
| [@softwareyong](https://github.com/softwareyong) | [@s1lv3rrud](https://github.com/s1lv3rrud) |

## 🤖 AI
|      Simon(김형민)      |          peter(심상훈)        |
| :-----: | :-----: |
| <img src="https://avatars.githubusercontent.com/u/74032445?v=4" width=120px alt="김형민"/>  | <img src="https://avatars.githubusercontent.com/u/69721000?v=4" width=120px alt="심상훈"/>  |
| [@hyeong8465](https://github.com/hyeong8465) | [@sanghoon416](https://github.com/sanghoon416)  


## 강의 추천

생성된 프로젝트에 도움이 되는 강의를 추천해주는 기능

## 데이터 수집

![셀레니움](https://github.com/user-attachments/assets/5b9d74e9-b66d-4bbb-993b-42afcac73fc3)

![포스트그레](https://github.com/user-attachments/assets/52d90a52-8281-4a69-bcd4-5c70070f005c)

인프런에서 강의 데이터를 수집하기 위해 **Selenium**을 활용한 웹 크롤러를 제작하였습니다.

### Selenium 사용 이유

일반적으로 **BeautifulSoup**를 활용한 크롤링 방법이 있지만, 인프런 페이지는 대부분 JavaScript로 렌더링되어 있어 단순한 HTML 파싱으로는 데이터 수집이 어려웠습니다. 이에 동적 웹 페이지 처리에 유리한 **Selenium**을 선택하였습니다.

### 수집 대상 및 범위

- 분야: 개발/프로그래밍, 게임 개발, 데이터 사이언스, 인공지능, 보안/네트워크 총 5개 분야
- 수집 강의 수: 약 120개 강의 * 5개 분야 → 중복 제외 총 535개 강의 데이터 확보

### 수집 데이터 항목

- **제목**
- **강의 내용**
- **강의 대상**
- **난이도**
- **사전 지식**

이러한 정보를 추출한 뒤 **PostgreSQL**에 저장하였습니다.

### PostgreSQL 사용 이유

최종적으로 강의 내용을 임베딩하여 벡터 형태로 저장하기 때문에 NoSQL 사용을 고민했습니다.

하지만 데이터의 양이 많지 않고 **PostgreSQL**에서 **pgvector** 확장을 통해 벡터 유사도 계산을 지원해서 백엔드와 인공지능 DB를 나누지 않고 **PostgreSQL** 하나만 사용했습니다.

### 아쉬운점

```python
# 강의 url 수집 
wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'css-y21pja')))

ul_elements = driver.find_elements(By.CLASS_NAME, 'css-y21pja')
```

최초의 데이터 수집 이후에 크롤러를 다시 실행했을 때 크롤러가 작동하지 않았습니다.

CLASS_NAME을 사용하여 하드 코딩했는 데 인프런 페이지의 CLASS_NAME이 매달 1일에 랜덤한 값으로 교체되어 다시 실행이 되지 않는 문제로 파악했습니다.

여유가 있다면 인프런 페이지의 구조를 어느정도 파악한 후 TAG_NAME을 위주로 구현하여 언제든 사용할 수 있도록 리팩토링할 예정입니다.

## 임베딩

---

서비스 시작 시 **콜드 스타트** 문제를 해결하기 위해 **콘텐츠 기반 필터링**을 채택하였습니다. 임베딩 기법으로는 **Sentence Embedding**과 **TF-IDF** 두 가지를 활용하였고, 초기에는 다양한 모델을 시도한 후 한국어 임베딩 모델인 **SRoBERTa**를 사용하기로 결정했습니다.

### 모델 선택 과정

- **MiniLM**: 추천 결과가 특정 분야(“게임”)에 편중되는 문제 발생
- **DistilBERT**: GPU 사용량 과다
- **SRoBERTa**: 한국어 임베딩 모델로 최종 선택

### 임베딩 전략

1. **Sentence Embedding**
    - 강의 내용(제목, 학습 대상, 학습 내용, 난이도, 기술 스택)을 문장 끝마다 띄워쓰기( ``)를 추가하여 하나의 문장으로 이어붙여 임베딩
    - 문장 끝마다 마침표(`.`)를 추가하여 문장 단위 의미를 유지
    - 강의 내용을 각각 임베딩 후 평균을 내어 하나의 벡터 생성
    
    BERT 기반 토크나이저에 맞게 데이터를 전처리한 후 768차원 벡터를 생성했습니다
    
2. **TF-IDF임베딩 벡터 활용**
    
    **문제점**: `TfidfVectorizer`를 바로 사용할 경우, 한국어 형태소 분석 및 명사 추출에 제한이 있어 정확한 임베딩이 어려움
    
    **해결방안**: KoNLPy 라이브러리로 형태소 분석기를 적용하여 명사 추출
    
    - 비교 형태소 분석기: Hannanum, Kkma, Mecab, Komoran, Okt
    - **Mecab** 선택 이유:
        - 한국어 명사 추출 성능 우수
        - 처리 속도 빠름
    
    단어 정제
    
    - **영어 명사**:
        - python, java, ai 등 중요 단어만 선별 후
        - 한국어(“파이썬”, “자바”, “인공지능”)로 치환
    - **한국어 명사**:
        - 1음절 단어 제외 (불필요 단어 최소화)
        - 네트워크 (네트+워크) , 자료구조 (자료+구조) 등 특정 단어를 토크나이저에서 제외
            - 의미 없는 단어 수 증가로 인한 벡터 차원 증가 방지
            - 중요한 단어의 필터링 방지
    
    단어 수에 따라 차원이 결정되므로 정제 과정이 중요했습니다
    
    정제된 명사 리스트를 바탕으로 1950차원의 벡터를 생성했습니다
    
3. **임베딩 벡터 활용**
    - TF-IDF 벡터를 통해 프로젝트와 관련된 상위 20개 강의를 추출
    - 추출된 강의에 대해서 SRoBERTa 임베딩으로 의미 유사도를 추가 반영하여 최종적으로 3개의 강의 추천

### 시간 단축 시도

강의 추천 시 약 7초 정도 걸려서 시간 단축의 필요성을 느꼈습니다

TF-IDF 특성상 벡터에 0이 많아서 이를 이용하여 차원수를 줄일 수 있지 않을까 라는 생각이 들었습니다

#### Truncated SVD 선택 이유

- 희소행렬 처리에 유리
- 직사각행렬에 사용 가능

<img width="564" alt="그래프좌" src="https://github.com/user-attachments/assets/ec43f5cb-0099-410a-beee-90a9642f8b2f" />

<img width="564" alt="그래프우" src="https://github.com/user-attachments/assets/c1965975-b797-4cd2-bb20-47875be374d3" />

누적 분산 합 그래프를 기반으로 에너지의 80~90퍼센트를 보존할 수 있는 차원인 10차원으로 축소했습니다

![explain](https://github.com/user-attachments/assets/0149a91b-8ab5-47ff-9337-5926d6ee1d28)

좌측 : 1950 차원

우측 : 10 차원

1950차원일 때와 10차원일 때 explain을 통한 cost에 차이가 없는 것을 확인했습니다

<img width="1478" alt="시간단축1" src="https://github.com/user-attachments/assets/1aa46862-f3af-4b73-bab7-91589a3bf0af" />

<img width="1484" alt="시간단축2" src="https://github.com/user-attachments/assets/9c233567-0282-48ca-8345-d09ac63bbe34" />

또한 로컬 테스트에서 svd를 적용한 이후 시간이 오히려 늘어난 것을 확인했습니다

프로젝트 마감까지 하루 남아서 더 이상의 개발은 못했지만 추후에 소요 시간을 단축할 예정입니다

### 최종 결과

<img width="1404" alt="최종결과좌1" src="https://github.com/user-attachments/assets/785671ca-0cec-4806-b6d2-6bb40485f9c4" />

<img width="668" alt="최종결과우1" src="https://github.com/user-attachments/assets/4b146857-5609-47a3-8a8e-4a6f099f6e10" />

<img width="1405" alt="최종결과좌2" src="https://github.com/user-attachments/assets/80b2151e-dc9c-4e30-baa4-c616419cd64e" />

<img width="667" alt="최종결과우2" src="https://github.com/user-attachments/assets/be391fd7-79e9-4a37-9b8f-c888b9a7f39e" />

좌측 : ETL 프로세스 자동화 프로젝트

우측 : 클라우드 기반 CI/CK 파이프라인 구축

생성된 프로젝트에 맞는 강의가 추천되는 것을 확인할 수 있습니다

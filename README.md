# KoreanRecipeGPT
> 안녕하세요, 성균관대학교에서 인공지능과 NLP를 공부하는 학생들로 이루어진 부추삼겹팀입니다. 저희는 자연어처리를 활용한 프로젝트를 계획 중에 냉장고 속 남은 재료를 어떻게 활용할 지 고민하다 본 프로젝트를 고안하게 되었습니다. 
<p align = "center"> <img src = https://github.com/skku-taehwan/KoreanRecipeGPT/blob/main/teamphoto/team%20introduction.png?raw=true width = 1000></p>


### 개요
<p align = "center"> <img src = https://github.com/skku-taehwan/KoreanRecipeGPT/blob/main/teamphoto/Project%20intro.png?raw=true width = 800></p>


> 저희 부추삼겹팀은 **‘냉장고를 부탁해’** 프로젝트를 통해 냉장고 속에 남은 음식들을 버리지 않고, 최대한 활용하고자 했다. 모델에 조리하고 싶은 식재료들과 음식명을 입력(input)하면, 입력된 재료들을 활용한 레시피를 출력(output)으로 생성하여 사용자에게 제공하는 것이 본 프로젝트의 목표이다. 앞서 언급한 식재료 기반의 레시피 생성(recipe-generation) 태스크를 수행하기 위해서는 식재료와 음식별 조리 방법에 대한 데이터를 수집해야 한다.
> 
> 마침 웹사이트나 공공기관의 레시피 데이터는 조리에 필요한 식재료들과 조리 방법을 제공하여, 스크래핑(scraping)을 활용하여 데이터를 수집할 수 있다. 또한, ‘해먹남녀’나 ‘만개의 레시피’ 등의 웹사이트는 요리에 필요한 식재료를 입력하면 이에 따른 레시피를 제공한다. 이를 활용하여 음식명과 사용할 식재료를 모델에 입력하면 쉽게 따라할 수 있는 레시피를 생성해주는 모델을 개발하여 앞서 언급된 문제점들을 해결하고자 한다.

### 모델 구조

<p align = "center"> <img src = https://github.com/skku-taehwan/KoreanRecipeGPT/blob/main/teamphoto/model%20structure.png?raw=true width = 800></p>

#### Input
- [해먹남녀](https://haemukja.com/), [만개의레시피](https://www.10000recipe.com/), [공공API](https://www.data.go.kr/), [메뉴판](https://www.menupan.com/)의 웹사이트에서 파이썬 모듈 beautifulsoup과 selenium을 이용하여 스크래핑한 식재료와 레시피 데이터를 전처리하여 모델에 넣어준다.

**데이터 전처리**
> 수집한 데이터에서 유의미한 단어만 선별하기 위하여 큰 의미가 없는 불용어를 제거하였다. 반복적으로 등장하는 불필요한 인삿말, 특수문자, url 등을 제거하여 꼭 필요한 부분만 남겼다. 또한 한국어 맞춤법을 교정하는 py-hanspell 모듈을 사용하여 각 레시피의 조리 순서의 맞춤법을 교정하였다. 마지막으로 레시피 조리 순서 문장의 띄어쓰기 교정을 위한 PyKoSpacing 모듈을 이용하였다.

#### [KoGPT2](https://github.com/SKT-AI/KoGPT2) 모델
> GPT-2는 머신러닝 알고리즘을 활용해 입력된 샘플 텍스트를 구문론적, 문법적 정보 등의 일관성을 갖춘 텍스트로 생성하는 자연어 처리 모델이다. 한국어로 학습된 오픈소스 기반 GPT-2 모델인 KoGPT-2는 질문에 대한 응답 생성, 문장 완성, 챗봇 등 한국어 해석이 필요한 여러 애플리케이션의 머신러닝 성능을 향상시킬 수 있다.

<p align = "center"> <img src = https://github.com/skku-taehwan/KoreanRecipeGPT/blob/main/teamphoto/KoGPT2%20%EB%A0%88%EC%8B%9C%ED%94%BC%20%EA%B2%80%EC%A6%9D%20%EA%B3%BC%EC%A0%95.png?raw=true width = 800></p>



**1. 인코딩 과정**
> 데이터프레임 형식으로 처리된 레시피 데이터를 하나의 레시피 당 하나의 라인으로 구성했다. 모델이 요리명, 식재료, 조리법의 경계를 알 수 있도록 각 구획 처음과 끝에 특정한 토큰을 삽입하고 식재료 사이에 특수문자를 삽입하여 식재료의 경계를 나타낼 수 있도록 처리하였다. 모델을 훈련하기 위해서 KoGPT2 모델에 적용되었던 토크나이저를 이용해 토큰화와 토큰을 숫자로 변환하는 인코딩(encoding)을 수행했다. 아래 그림은 데이터 변형 작업의 흐름을 나타낸다.

<p align = "center"> <img src = https://github.com/skku-taehwan/KoreanRecipeGPT/blob/main/teamphoto/Model%20encoding.png?raw=true width = 600></p>


**2. 디코딩 과정**
> 디코더의 경우 문장 생성과 더 직접적인 연관을 가진 평가 기준을 도입했다. 구체적으로, 생성된 레시피의 형태소를 분리하여 안에 포함된 식재료들이 입력값의 식재료와 얼마나 일치하는지 평가하고, 기존의 레시피와 얼마나 유사한지를 평가하였다. 이를 위해 F1 점수와 BLEU 점수를 적용했다.

**학습 과정**
> 인코딩된 레시피는 워드 임베딩, 포지셔널 인코딩(positional encoding)을 차례대로 거친 뒤 훈련한다. 타겟을 한 칸씩 옮겨가며 다음 단어를 추정하는데 배치 사이즈(batch size)만큼의 레시피로 결과값을 도출해내고 추정한 단어와 실제 단어를 비교해 손실을 구하게 된다. 마지막으로 Adam optimizer를 통해 모수의 값을 조정하며 학습이 진행된다.


### 결과
컴퓨터 사양 - **Colab Pro**

<p align = "center"> <img src = https://github.com/skku-taehwan/KoreanRecipeGPT/blob/main/teamphoto/Output.png?raw=true width = 800></p>


GridSearch를 적용한 결과 Max_Length=300, Epoch=5, Batch_Size=16일 때의 성능이 가장 좋았다.

#### 모델 예시 

<p align = "center"> <img src = https://github.com/skku-taehwan/KoreanRecipeGPT/blob/main/teamphoto/Result.png?raw=true width = 800></p>


### 한계점
> 생성된 지시문에서 나타나는 식재료의 용량이 부정확하다는 점, 입력한 식재료와 다른 식재료를 산출한다는 점, 입력한 요리 이름과 거리가 있는 지시문을 생성한다는 점 등은 본 프로젝트의 한계이다. 또한, 룰베이스 기반의 전처리를 시행하였기 때문에, 직접 정의한 불용어와 완벽히 일치하지 않는 불필요한 어절이나 문장은 제거할 수 없었다.



### 참조 문헌
[1] RecipiGPT, https://recipegpt.org/
[2] KoGPT2, https://github.com/SKT-AI/KoGPT2
[3] Ratsgo NLPBOOK, https://ratsgo.github.io/nlpbook/docs/generation/inference1/
[4] 해먹남녀, https://haemukja.com/
[5] 만개의레시피, https://www.10000recipe.com/
[6] 공공데이터포털, https://www.data.go.kr/
[7] 메뉴팟닷컴, https://www.menupan.com/


```ratsnlp를 활용하여 

자연어 처리 실습을 위한 패키지입니다. 구글 코랩(colab) 환경에서 동작할 수 있도록 작성하였습니다.

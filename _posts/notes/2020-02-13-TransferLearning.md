---
title: "What is Transfer Learning"
excerpt: "2020. 02. 13.  Transfer Learning 소개"
search: true
categories: 
  - notes
---

# 전이학습 뽀개기 
  Fairness 공부에 앞서, 인종분류기를 만드는 과제를 받았다. 따라서 오늘은 전이학습의 종류와 pytorch로 전이학습을 하는 방법에 대해서 알아볼 예정이다. 

일단 글을 쓰는 이유는 다음과 같다. 예전에 전이학습 및 앙상블을 진행해봤는데, 다시 어려움을 겪고 있기 때문이다. 

(1) 케라스로만 전이학습을 진행해봤다 -> 튜토리얼로 진행해 본것이기 때문에 정의가 확실히 서있지 않다. 
(2) pytorch가 익숙하지 않다. -> 전이학습도 부족한데...

따라서 이런 이유로 전이학습의 정의 및 유형부터 어떻게 할 수 있는지 자세히 적어보려고 한다. 


## What is transfer learning? (11시 30분 전까지 정리해서 커밋) 

전이학습은 주로 학습 데이터가 부족한 분야의 모델 구축을 위해 데이터가 풍부한 분야에서 훈련된 모델을 재사용하는 학습 기법이다. 즉, 데이터 세트가 유사한 분야에 학습치를 전이하여 fine tuning 기바의 신경망 학습 재사용 기법이다. 
(#데이터부족해소 #학습시간단축 #학습치재사용

### 전이학습의 매커니즘은 다음과 같이 분류할 수 있다. 

	1. Feature learning : Dataset 기반 학습 수행 
	2. Transfer Paraneter : 학습치 전이 
	3. Classifier Learning : Fine Tuning 기반의 미세 조정 

### 전이학습은 Target Data와 Source Data의 label 존재 유무에 따라 분류 할 수 있다. 

	1. Multi-task 학습 : 하나의 training set으로 여러개의 분류 모델을 처리하는 것
	2. Self-taught 학습 : 라벨링된 데이터 셋으로 Feature를 생성하고, 최종 classufuer을 변환
	3. Domain Adaptation : Feature 생성 후 Target Domain 구별을 차단. 풍부한 데이터를 바탕으로 훈련 시 도메인 구분 능력을 약하게 학습하여 Target Data를 분류 가능하도록 모델 구축
	4. Sample Select, Bias : 학습치 샘플을 선택하여, 해당 학습치만 전이한다. 

**추가적으로** 
	* 	Fine-tune Conv : 미리 학습된 Convnet 마지막에 Fully Connected Layer 만 변경하여 분류 실행
	* Pre-trained Model : 미리 학습된 모델의 가중치를 새로운 모델에 적용
	* Layer Re-use : 기존 모델의 일부 레이어를 재사용하여 부족한 데이터의 도메인 모델 구축에 활용한다. 


### 그렇다면 나는 어떤 접근 방식을 활용해야할까?

지금 내 상황을 보자, 서치한 모델은 총 3가지가 있고, 성별_나이_성별+나이 분류기를 만들어야한다. 이럴땐 어떻게 '사고'하는 것이 좋을지 생각해보자

#### 내가 해결해야할 문제 정의하기
나의 Task -> '성별_나이_성별+나이'를 분류하는 모델을 만들자 -> 다수의 task를 해야함.

#### 인종 먼저 분류해보기

먼저  pre-training된 모델을 이용하여 학습을 진행해봤다. 
	* facenet - vggface2 / pretrain model 사용 

##### Multi Task 해결하기

(1) Multi-Task Learning 
      학습하려는 여러 종류의 task가 동일한 도메인에 존재하는 경우
       Ex) 얼굴 데이터로 부터 얼굴 인식, 표정 인식, 성별 분류, 포즈 분류 모두 처리 

		* 	Hard Parameter Sharing 
		Hard Parameter Sharing 은 가장 흔히 사용되는 방법으로
		Hidden Layer 를 공유하고 Task 별로 일부 개별적인 Layer 를 
                가지고 가는 형태로 활용된다. 이러한 방법의 활용은 당연히            
		Over Fitting 을 방지하는데 효과가 있다.
		![](&&&SFLOCALFILEPATH&&&image.png)

		* 	Soft Parameter Sharing 
		Soft Parameter Sharing 는 조금 다르게 각각의 Task 별로 별도의 Layer 를 가지고 있다. 
		다만, 각각의 Layer 가 비슷해 질 수 있도록 L2 Distance 를 사용한다.

(2) Multi-Domain Learning 
     여러 종류의 task가 각각 다른 도메인에 존재하는 경우 
       Ex) 의료 이미지에서 병변 검출, 일상 이미지에서 카테고리 검출, 문자 이미지에서 문자열 검출
![](&&&SFLOCALFILEPATH&&&image.png)




#### 현재 search한 model 
	1. 	(tensorflow) gender and race model with faceNet 
	-> pre-trained 된 faceNet을 활용하여 multi-task 작업을 진행했다. 

	2. 해당 모델을 참고하여 진행하도록 결정

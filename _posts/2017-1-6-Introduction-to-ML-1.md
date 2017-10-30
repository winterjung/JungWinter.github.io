---
layout: post
title: 머신러닝 입문하기 강의를 듣고 - 1
tags: [머신러닝, 공부]
---

엄태웅 교수님의 [기계학습 (Machine Learning) 입문하기](http://t-robotics.blogspot.kr/2015/10/machine-learning.html)강의를 듣고 필기한 것들을 복습삼아 포스팅한다.

## Introduction to machine learning and deep learning
### Caution
다 얻으려고 하기 말기  
전체적인 것을 알아가기  
키워드를 잡아채서 더 알아보기  
무엇을 하고 원리는 무엇인가 알아가기

### 머신러닝
`경험 E`, `태스크 T`, `측정 P`  
데이터, 나의 목표, 점수  
e.g. T: win, P: goals, E: Movements, Evaluation  
e.g. 탁구치는 로봇

### Tasks
보통 분류/회귀/군집  
분류는 손글씨 숫자를 0부터 9까지 분류  
회귀는 특정데이터에 대한 결과를 예측하는 것  
군집은 라벨링 되어있지 않은 데이터들을 묶는 것

### Performance
잘됐다 안됐다 판단  
분류: 0-1(제로원) loss function : error 발생시 error += 1  
회귀: L2(거리제곱) loss function : 가까운건 높은 점수, distance로 점수  
군집: 중심과 모든 것들의 총합이 최소인 곳

### Experience
그냥 데이터들  
분류, 회귀 -> labeled  
군집 -> unlabeled

### Toy e.g.
키와 몸무게, fitting curve찾기  
잘 찾는 것, 여기서 잘 이란건 선과 점들의 거리 차이가 최소로  
모델, `y = ax + b` 에서 a, b찾기

#### 상세
Least square problem -> Ax = b인데 A가 square라면 x = A^-1b이다.  
근데 A가 square가 아니면 Pseudo inverse(의사역행렬?) B = (X^TX)^-1X^Ty  
(아놔 마크다운에는 수식이 없나? jupyter는 수식 기능이 있어서 편한데)  
클로즈폼 솔루션이 항상 있진않음 -> 최적화(optimize)해준다.

### LSM
선의 기울기를 바꿔봤는데 에러값이 줄음 -> 굿 뉴스, 그럼 이대로 계속 갈게  
언제까지? -> 그 변화가 줄어드는 양, error값이 점점 줄다가 더 이상 안줄어 들 때까지   
P를 측정하는 함수 -> objective function혹은 cost function  
코스트 함수가 가장 많이 줄어드는 방향으로  
기울기를 Gradient라고함 -> 그래서 Gradient Descent Method라고 함  
근데 항상 좋지는 않음 -> Local minima problem때문에  

### Regularization
무엇이 Correct Model이냐? -> 아깐 우리가 lineal하다고 가정함.  
키-몸무게 를 나이-운동능력이라 하면?  
-> 1차, 2차, 로그인지 어떻게 알아 -> Model Selection문제  
막 100차 함수가 최적이다 -> Overfitting -> 그래서 필요한게 Regularization  
우리의 주 목적 1. 현재를 잘 설명, 2. 미래를 잘 설명 -> 복잡한 모델 피하기
(y-yhat)^2 예측값과 실제값의 차이를 최소화  
||w||^2 이 weight자체도 작게하는게 목적  
-> 즉 잘 설명하면서 큰 weight를 갖지 않게  
라이클리후드? Posteriori?

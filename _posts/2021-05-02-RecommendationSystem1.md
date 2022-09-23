---
title: (Recommendation System) 행렬 분해를 이용한 잠재요인 협업 필터링, SGD
date: 2021-05-02 23:36 +0900
lastmod: 2021-05-02 23:36 +0900
categories: [Recommendation System]
tags: [Recommendation System]
mermaid: true
math: true
---

행렬 분해는 다차원 매트릭스를 저차원 매트릭스로 분해하는 기법으로 대표적으로 SVD(Singular Value Decomposition), NMF(Non-Negative Matrix Factorization) 등이 있다.

즉, M개의 사용자(User) 행과 N개의 아이템(Item) 열을 가진 평점 행렬 R은 M X N 차원으로 구성되며, 행렬 분해를 통해 사용자-K 차원의 잠재 요인 행렬 P (M X K 차원)와 K 차원의 잠재 요인ㅡ아이템 행렬 Q.T (K X N 차원)으로 분해될 수 있다.

 따라서, R = P * Q.T 이며, 아래 그림과 같이 나타낼 수 있다. R 행렬의 u행 사용자와 i열 아이템 위치에 있는 평점 데이터를 r(u,i)라고 하면, P행렬과 Q.T 행렬의 곱을 통해 평점데이터를 유추할 수 있다.

$r_{(u,i)}= p_u * q_i^t$

![Untitled](/assets/img/2021-05-02-RecommendationSystem1/Untitled.png)

# 확률적 경사하강법(SGD)을 이용한 행렬 분해

R 행렬을 P와 Q 행렬로 분해하기 위해서는 주로 SVD 방식을 이용하나, 이는 널(NaN) 값이 없는 행렬에만 적용할 수 있다. R 행렬은 대부분의 경우 널(NaN) 값이 많이 존재하는 희소행렬이기 때문에 일반적인 SVD 방식으로 분해할 수 없고, 확률적 경사 하강법(Stochastic Gradient Descent, SGD) 이나 ALS(Alternating Least Squares) 방식을 이용해 SVD 를 수행한다.

특히, 확률적 경사 하강법을 이용한 행렬 분해를 살펴보자면 P와 Q 행렬로 계산된 예측 R 행렬 값이 실제 R 행렬 값과 최소한의 오류를 가질 수 있도록 반복적으로 비용함수를 최적화함으로써 적합한 P와 Q 행렬을 유추하는 것이 알고리즘의 핵심이다.

확률적 경사 하강법을 이용한 행렬 분해의 전반적인 절차는 다음과 같다.

```markdown
1. P와 Q 행렬을 임의의 값을 가진 행렬로 초기화 한다.
2. P와 Q 전치행렬을 곱해 예측 R 행렬을 계산하고, 실제 R 행렬과의 차이를 계산한다.
3. 차이를 최소화할 수 있도록 P와 Q 행렬의 값을 적절한 값으로 각각 업데이트한다.
4. 특정임계치 아래로 수렴할 때까지 2, 3번 작업을 반복하면서 P와 Q 행렬을 업데이트해 근사화한다.
```

---

## 목적함수(비용함수)

실제값과 예측값의 차이 최소화와 L2 정규화(Regularization)를 고려한 비용 함수식은 다음과 같다.

(cf. 학습을 진행할 때, 학습 데이터에 따라 특정 weight의 값이 커지게 될 수있다. 이렇게 되면 과적합이 일어날 가능성이 아주 높은데 이를 방지하기 위해 L1, L2 regularization를 사용한다.)

$\min\sum {(r_{(u,i)}-p_uq_i^{T})^2 + \lambda({\lVert p_u \rVert}^2} + {\lVert q_i \rVert}^2)$

- $r_{(u,i)}$ : 실제 R 행렬의 u행, i열에 위치한 값
- $p_u$ : P 행렬의 사용자 u행 벡터
- $q_i^{T}$ : Q 행렬의 아이템 i행 전치벡터
- $\lambda$ : L2 정규화 계수

### **1) 오차 제곱 합**

목적 함수는 두 가지 텀으로 나뉘는데, 앞 부분은 우리가 많이 알고 있는 SE(Squared Error)와 같다. 여기서 오차란, 실제 평점값과 예측 평점 값의 차이이다. 주목할 점은 **학습 데이터에서 실제 평점이 있는 경우(observed)에 대해서만 오차를 계산한다**는 것이다. SVD는 행렬 분해를 위해 행렬 안에 값이 모두 차있어야 한다. 그러나 User-Item Rating 행렬의 경우, 사용자는 극히 일부의 아이템에만 평점을 남겨 행렬의 대부분이 결측값으로 존재한다. 즉, Sparsity가 매우 높다. SVD를 사용하려면 결측값을 0이나 평균 평점 등으로 대체해야 하는데, 이는 데이터를 왜곡시키므로 학습에 악영향을 미친다. 반면에, **MF는 결측치를 대체하지 않고도 학습이 가능하므로 Sparsity가 성능에 큰 영향을 주지 않는다는 장점이 있다.**

### **2) 정규화**

두 번째는 과적합을 방지하는 정규화 텀이다. **모델이 학습함에 따라 파라미터의 값(weight)이 점점 커지는데, 너무 큰 경우에는 학습 데이터에 과적합하게 된다.** 이를 방지하기 위하여, 학습 파라미터인 p와 q의 값이 너무 커지지 않도록 규제를 하는 것이 정규화이다. MF에서는 L2 정규화를 사용하였다. 파라미터 값이 커지면 p벡터의 제곱과 q벡터의 제곱 합도 커질 것이고 이는 목적 함수도 커지게 하므로 패널티를 주는 것이다. 람다는 목적 함수에 정규화 텀의 영향력을 어느 정도로 줄 것인지 정하는 하이퍼 파라미터이다. 람다가 너무 작으면 정규화 효과가 적고, 너무 크면 파라미터가 제대로 학습되지 않아서 언더피팅(Underfitting)이 발생한다.

### **3) Optimazation (SGD)**

MF에서는 목적함수를 최소화하는 최적화 기법으로 SGD(Stochastic Gradient Descent)를 활용하였다. 파라미터 업데이트를 위해 목적함수를 p와 q로 편미분한다. 다음은 MF의 목적함수를 p와 q로 미분하는 과정을 보여준다. 이렇게 편미분으로 도출된 값을 Gradient라고 한다.

위의 비용 함수를 최소화 하기 위해 새롭게 업데이트 되는 $p$와 $q$ 는 다음과 같이 계산된다(p와 q에 대한 편미분을 통해 유도). 

${\partial L \over \partial p_u} = {\partial (r_{u,i} - p_u^Tq_i)^2 \over \partial p_u} + {\partial \lambda {\lVert p_u \rVert}_2^2 \over \partial p_u} = -2(r_{u,i} - p_u^Tq_i)q_i + 2\lambda p_u = -2(e_{u,i}q_i - \lambda p_u)$

${\partial L \over \partial q_i} = {\partial (r_{u,i} - p_u^Tq_i)^2 \over \partial q_i} + {\partial \lambda {\lVert q_i \rVert}_2^2 \over \partial q_i} = -2(r_{u,i} - p_u^Tq_i)p_u + 2\lambda q_i = -2(e_{u,i}p_u - \lambda q_i)$

$p_{u} = p_u + \eta(e_{u,i} * q_i - \lambda * p_u)$

$q_{i} = q_i + \eta(e_{u,i} * p_u - \lambda * q_i)$

- $e_{(u,i)}$ : u행, i열에 위치한 실제 행렬 값과 예측 행렬 값의 차이
- $p_u$ : P 행렬의 사용자 u행 벡터
- $q_i^{T}$ : Q 행렬의 아이템 i행 벡터
- $\lambda$ : L2 정규화 계수
- $\eta$ : 학습률

SGD는 모든 사용자(u) 잠재 벡터와 모든 아이템(i) 잠재 벡터에 대해서 파라미터 업데이트를 하나씩 순차적으로 진행된다. 따라서 사용자와 아이템 수가 많아질수록 학습이 매우 오래 걸린다. 그리고 하나씩 순차적으로 학습해야 하다보니 병렬처리가 불가능하다. 이러한 단점을 보완하여 병렬 처리가 가능한 ALS라는 최적화 기법이 등장했다.

---

## **Adding Biases**

사용자나 아이템별로 평점에 편향이 있을 수 있다. 예를 들어, 사용자 A는 점수를 후하게 주고 반면에 사용자 B는 점수를 짜게 준다던지, 아니면 어떤 영화는 유명한 명작이라 사용자의 취향과 별개로 점수가 높고, 어떤 영화는 그렇지 않을 수 있다. 아래 식을 보면, Adding Bias에서 예측 평점은 학습 데이터 전체 평점의 평균(μ)에 사용자가 가진 편향(b_u), 아이템이 가진 편향(b_i), 그리고 두 잠재벡터의 내적들로 구성되어 있다. 모든 평점의 평균을 더해주는 이유는 대부분의 아이템이 평균적으로 μ 정도의 값은 받는다는 것을 감안해주기 위함이다.

$\hat r_{u,i} = \mu + b_u + b_i + p_u^Tq_i$

위의 예측값을 가지고 수정된 목적함수는 아래와 같다. 여기서 주의할 점은, μ는 학습 데이터로부터 도출되는 상수값인 반면에, Bias들은 모두 학습 대상인 파라미터라는 것이다. 따라서, 두번째 정규화 텀에도 Bias가 추가되어 있다. (*참고: p와 u는 벡터인 반면, bias들은 모두 스칼라값이므로 둘이 서로 표현법이 다르다.)

$\min\sum {(r_{(u,i)} {\color{Red} -\mu} {\color{Red} -b_u} {\color{Red} -b_i}-p_uq_i^{T})^2 + \lambda({\lVert p_u \rVert}^2} + {\lVert q_i \rVert}^2 {\color{red} +b_u^2} {\color{red} +b_i^2})$

위에서 설명한대로 SGD를 하기 위한 업데이트 공식은 아래와 같이 도출된다. 이번에는 파라미터가 네 종류이므로, 4개의 각각 파라미터로 목적함수를 편미분하여 Gradient를 구하고 이를 바탕으로 업데이트를 수행한다. 대체로 기본 MF보다 Adding Bias를 적용했을 때 성능이 더 좋게 나온다.

$b_u = b_u + \eta(e_{u,i} - \lambda * b_u)$

$b_i = b_i + \eta(e_{u,i} - \lambda * b_i)$

$p_{u} = p_u + \eta(e_{u,i} * q_i - \lambda * p_u)$

$q_{i} = q_i + \eta(e_{u,i} * p_u - \lambda * q_i)$

---

## **MF의 장점과 단점**

MF는 Data Sparsity에 큰 영향을 받지 않는다는 점에서 Content Base Collaborative Filtering(이하 CBCF)이나 SVD보다 성능이 훨씬 좋다. 사용자나 아이템이 늘어날수록 Sparsity가 증가하게 되는데, CBCF나 SVD는 결측값을 임의로 채워서 추천에 사용하다보니 성능이 떨어질 수 밖에 없다. 반면에, MF는 결측값을 아예 사용하지 않으므로 사용자와 아이템이 많아져도 기존 성능을 유지할 수 있다. **즉, Sparsity에 강하고 Scalibility가 좋다.** 그러나 SGD를 활용한 MF는 앞서 언급한대로 사용자와 아이템이 많아지면 학습 속도가 느리고, 병렬 처리도 불가능하다는 단점이 존재한다.

---

## **References**

[1] [https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf](https://datajobs.com/data-science-repo/Recommender-Systems-[Netflix].pdf)
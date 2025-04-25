---
title: "ORB descriptor"
layout: post
categories: "Computer Vision"
---

# ORB

태그: CV, paper

## Abstraction

- 현재의 feature matching 방법들은 높은 연산 복잡도를 갖고 있다.
- 이 논문은 BRIEF에 기반한 binary descriptor인 ORB를 제안한다.

## 1. Introduction

- SIFT keypoint detector와 descriptor는 다양한 분야에서 유용하지만 계산하기 복잡하다.
    - for realtime systems: visual odometry, low-power devices, …
- 그래서 SIFT의 높은 계산 복잡도를 해결하기 위한 방법들이 제안되었다.
    - SURF, using GPU
- 때문에 이 논문에서는 계산복잡도를 줄이고 노이즈에 영향을 덜 받는 방법을 제안한다.
- ORB(Orienteted FAST Rotated BRIEF)는 FAST keypoint detector와 BRIEF descriptor를 기반으로 한다.
- ORB의 main contribution
    
    ![스크린샷 2023-08-05 23.54.31.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-05_23.54.31.png)
    

## 2. Related Work

### *Keypoint vs. Descriptor*

- *Keypoint is a (locally) distinct location in an image*
- *The feature descriptor summarizes the local structure around the keypoint*
- ORB 한 줄 요약
    - *oFAST로 keypoint 찾고 → Harris로 Corner 찾고(Feature) → rBRIEF로 Description*

### 2.1. Keypoints

- FAST는 keypoint를 찾는 방법 중 하나
    - 효율적이고 합리적인 코너 키 포인트를 찾지만 scale을 위해 image pyramid를 사용
- ORB는 edge와 corner를 구분하기 위해 Harris를 사용한다.
    - FAST에서는 edge를 corner로 인식하는 문제가 있었다.
- SIFT와 SURF같이 많은 keypoint detector들이 orientation operator를 포함하지만 FAST는 그러지 않는다.
    - SIFT: gradient 계산의 히스토그램 → 계산하기 까다로움
    - SURF: block pattern에 의한 approximation → 부정확
- ORB는 Rosin의 centroid technique을 활용했다.
    - 하나의 keypoint에 다양한 결과가 나오는 SIFT orientation operator와 달리 centroid operator는 하나의 결과가 나온다.

### 2.2. Descriptor

- BRIEF는 smooted image patch에서 픽셀 사이의 간단한 이진 테스트를 이용한 feature descriptor다.
- SIFT와 비슷하지만 빛, 블러, perspective distortion에서 강인함.
    - 근데 in-plane rotation에 취약함(in-plane: roll, out-of-plane: pitch, yaw)
    - *SIFT는 계산 복잡도가 크다고 한다*
- BRIEF는 이진 테스트를 사용하여 classification trees를 훈련시키는 연구에서 기반한다.
    - 사실 잘 이해가 안됨..
    - *작은 2진 문자열을 통해 keypoint를 descript하기 때문에, 계산 복잡도가 낮다*
- Visual vocabulary 방식은 offline clustering을 통해 관련이 없는(uncorrelated) 예제들과 매칭에 사용할 수 있는 예제들을 찾는다.
    - 이 방법은 uncorrelated binary test를 찾는데 유용할 것

## 3. oFAST: FAST Keypoint Orientation

### 3.1. FAST Detector

- FAST는 하나의 parameter를 갖는다.
    - 중심 픽셀과 중심 픽셀 기준 원형 고리 사이의 intensity threshold
- ORB는 FAST-9을 사용한다.
    
    ![스크린샷 2023-08-06 02.31.39.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-06_02.31.39.png)
    
    - p: 중심픽셀
    - 중심 픽셀 값 + threshold 보다 크고 작은지를 통해 어두운지 밝은지 비슷한지 구분함
    - FAST-9은 9개의 픽셀이 연속적으로 밝거나 어두우면 코너 후보로 판단한다.
- ORB에서는 FAST keypoint를 정렬하기 위해 Harris corner measure(?)를 적용함
- FAST는 multi-scale features을 생성하지 않는다.
    - 이미지의 스케일 피라미드를 사용하고 피라미드의 각 수준에서 FAST features(Harris 필터링)을 생성한다.

### 3.2. Orientation by Intensity Centroid

- *강도의 무게 중심?*
- corner의 중심와 corner intensity의 centroid(무게중심) 사이의 벡터
- 이 순간의 패치를 다음과 같이 정의한다.
    - *Image Moment*
    
    $$
    m_{pq} = \sum_{x,y} x^p y^q I(x,y)
    $$
    
- 이 순간의 무게 중심은 다음과 같이 정의할 수 있다.
    - *Center of mass*
    
    $$
    C = \left( {m_{10} \over m_{00}}, {m_{01} \over m_{00}} \right)
    $$
    
- 그리고 코너의 중심을 $O$라고 할 때, Intensity Centroid는 $\vec{OC}$라고 할 수 있다.
- 이때의 방향은 다음과 같이 구할 수 있다.
    - *Orientation*
    
    $$
    \theta = atan2(m_{01}, m_{10})
    $$
    

![Untitled](/images/resorce_ORB_des/Untitled.png)

![Untitled](/images/resorce_ORB_des/Untitled%201.png)

### *Intensity Centroid를 사용한 이유*

- 대표적인 gradient 측정 방법 BIN, MAX와 비교, 둘 다 smoothed image에서 계산함
- MAX: keypoint patch에서 최대값을 고름
- BIN: 10도 간격으로 히스토그램을 만든 후 그 중 가장 높은 bin을 고름(SIFT와 비슷, SIFT는 단일 방향만 선택함)
    - bin: 큰 상자
    
    ![Untitled](/images/resorce_ORB_des/Untitled%202.png)
    
    - 노이즈에 따른 orientation error

## 4. rBRIEF: Rotation-Aware Breif

### 4.1. Efficient Rotation of the BRIEF Operator

### *Brief overview of BRIEF*

- BRIEF descriptor는 binary intensity test 세트로 구성된 이미지 patch에 대한 bit string description이다. (뭔소리여?)
    - *1. Keypoint 주변의 patch를 고른다*
    - *2. Patch 안에서 pixel 쌍의 집합을 고른다*
    - *3. 각각의 쌍에서 픽셀의 intesity를 비교한다. 아래 식 참고, p(x) < p(y)*
    - *4. 모든 binary test 결과(아래 식에서는 $\tau$)를 bit string으로 concatenate 한다 (1, 0, 1, 1 → “1011”)*
    - *이정도면 알잘딱 understanding*
- ***Additional*** :Sampling pair를 고르는 방법
    
    ![Untitled](/images/resorce_ORB_des/Untitled%203.png)
    
    - $G \, I$: Uniform random smapling
    - $G \, II$: Gaussian sampling
    - $G III$: $s_1$ Gaussian;  $s_2$ Gaussian centered around $s_1$
    - $G \, IV$: Discrete location from a coarse polar grid
    - $G \, V$: $s_1 = (0,0)$; $s_2$ are all location from a coarse polar grid
    - 뭘 사용해도 좋지만, G V는 성능이 좀 떨어지는 편
- **Smoothed** image patch를 $\bold{p}$라 하고, binary test $\tau$를 다음과 같이 정의한다.
    - *smoothed image를 사용하는 이유: noise*
    
    ![Untitled](/images/resorce_ORB_des/Untitled%204.png)
    
    - $\bold{p(x)}$는 점 $\bold{x}$에서 $\bold{p}$의 intensity를 의미하고, 이 함수는 n개의 이진 테스트의 벡터로 정의할 수 있다.
    
    ![Untitled](/images/resorce_ORB_des/Untitled%205.png)
    
    - *이거 그냥 쉽게 말해서 2진 테스트 이후 string concatenate한 결과(”1010001”)의 decimal.*
- *본문에서는 “we use one of the best performers, a Gaussian distribution around the center of the patch” 라고 되어 있는데,
아무래도 위의 Additional에서 G-3를 말하는 듯하다*
    - *또한 벡터 길이 256은 256개의 쌍을 이용했다는 거로 생각된다. 이건 BRIEF 논문에서 참고한 듯*
    - *그리고 smoothing을 위해 integral image를 사용했다고 함*
        - *5x5 sub-window patch, 31x31 pixel patch*

### *Steered BRIEF*

- BRIEF는 in-plane rotation에서 Matching performance가 급격하게 떨어진다.
    - 그래서 BRIEF의 저자는 각 patch에 대해 회전과 원근 왜곡 세트에 대한 BRIEF descriptor 연산을 제안했다.
    - 그러나 이 방법은 계산 복잡도가 높다.
    - 더 나은 방법은 keypoint의 방향에 따라 BRIEF descriptor를 회전시키는 것(steer)
- $(\bold{x}_i, \bold{y}_i)$에서 n 이진 테스트의 특징점 세트를 $2 \times n$  행렬로 정의할 수 있다.
    
    ![Untitled](/images/resorce_ORB_des/Untitled%206.png)
    
- 이때의 Intensity Centroid를 통해 계산된 Orientation을 행렬 S에 곱해준다.
    - 본문에서는 “Using the patch orientation $\theta$ and the corresponding rotation matrix $R_\theta$, we construct a “steered” version $\bold{S}_\theta$ of $\bold{S}$
    
    ![Untitled](/images/resorce_ORB_des/Untitled%207.png)
    
- 추가적으로 각도를 $2\pi / 30 (12 \degree)$로 이산화 했고, 미리 계산된 BRIEF pattern들을 이용하여 lookup table을 구성함~

### 4.2. Variance and Correlation

- *BRIEF descriptor binary pairs should be high variance and uncorrelated*
- BRIEF는 분산이 크고 평균이 0.5가 나온다는 특징이 있다. (*그래야만 한다*)
    - 평균이 0.5일 때 분산은 0.25가 된다.
    - *분산: 편차 제곱의 평균, BRIEF 이진 테스트의 결과가 0 또는 1이기 때문에 (1 - 0.5)^2 = 0.25*
    - *BRIEF 이진 테스트 쌍들의 평균의 0.5(분산 0.25)이어야 좋은 이유*
        - 그래야 “특징점”을 잘 구별할 수 있음.
        - 예를 들어 corner로 의심되는 keypoint 주변을 설명하는 BIREF 설명자가 flat한 부분하고 큰 차이가 없다면,
        correspondance를 찾기가 너무 어려워지기 때문?
    
    ![Untitled](/images/resorce_ORB_des/Untitled%208.png)
    
    - x 축은 평균(0.5)과의 거리를 의미하기 때문에, 256개의 쌍에서 표준 편차(?)의 히스토그램이라고 생각하면 될듯?
        - 특이한 점은, steered BRIEF를 사용하면 분산이 줄어든다는 점이다…
- 그리고 서로 연관성이 적어야 한다. (그래야 테스트 때마다 유의미한 결과를 얻을 수 있기 때문)
    
    ![Untitled](/images/resorce_ORB_des/Untitled%209.png)
    
    - PCA(principle Components Analysis)를 통해 계산한 eigenvalue들 중 가장 높은 40개를 뽑은 것이다.
    - *PCA 계산 결과로 나온 eigenvalue는 분산의 크기를 의미하고, eigenvector는 분산의 큰 방향을 의미한다.*
    - 결론적으로 Steered BRIEF는 분산이 작기 때문에, discriminative하지 못하다.
    
    ![Untitled](/images/resorce_ORB_des/Untitled%2010.png)
    
    - 실선: keypoint~inliers, 점선: keypoint~outliers
    - 여기서 주목할 점은, steered BRIEF가 왼쪽으로 치우쳐져서 다른 방법보다 inliers들과 겹치는 부분이 많다는 점이다.
        - 결론: Steered BRIEF는 not discriminative하다

### 4.3. Learning Good Binary Features

- steered BRIEF에서 분산의 손실을 회복하고 이진 테스트의 연관성을 줄이기 위해 좋은 이진 테스트를 고르는 learning method를 개발함
    - 가능한 방법으로는 PCA를 사용하거나, 다른 dimensionality-reduction method를 사용하는 것
    - 대규모 이진 테스트 집합에서 분산이 높고 연관성이 적은 256개의 feature를 식별하는 것
- 일단, PASCAL 2006에서 300k개의 keypoint로 트레이닝 세트를 구성함
- 31*31 patch로 가능한 모든 이진 테스트를 반복하였고, 각각의 테스트는 5*5 sub-windows를 사용함
- *Algorithm*
    
    ![Untitled](/images/resorce_ORB_des/Untitled%2011.png)
    
- 쉽게 말해서
    - greedy searching으로 가능한 모든 테스트 중 분산이 높고(high variance), 연관성이 적은(low correlation) 테스트 세트들을 골라냈다.
    
    ![Untitled](/images/resorce_ORB_des/Untitled%2012.png)
    
    - 좌 → learning 알고리즘 안돌림/ 우 → learning 알고리즘 돌림
    - 보면 learning 알고리즘을 돌린 경우가 더 분산이 크고 correlation이 적다.
    (보라색, 검정색에 가까울 수록 correlation이 적음)

### 4.4. Evaluation

- Evaluation of rBRIEF
    - 같은 평면 상에서 임의로 회전 시킴 + Gaussian noise
    - 위와 같이 처리를 하였을 때 inlier의 비율로 평가함
    
    ![스크린샷 2023-08-18 22.21.12.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_22.21.12.png)
    
    - SURF가 45°에서 떨어지는 이유는 Haar wavelet filter를 사용해서 quantization effect가 발생했기 때문이라고 한다.
        
        ![Untitled](/images/resorce_ORB_des/Untitled%2013.png)
        
- ORB(rBRIEF)는 Gaussian noise에 강인함(SIFT는 취약함)
    
    ![스크린샷 2023-08-18 22.23.23.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_22.23.23.png)
    
- ORB를 실제 이미지에서 평가
    - 이것도 쉽게 말해서, 걍 이미지를 homographic warp 시킨 후에 전후 사진의 inlier와 검출 포인트 개수를 비교
    
    ![스크린샷 2023-08-18 22.26.27.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_22.26.27.png)
    
    ![스크린샷 2023-08-18 22.26.01.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_22.26.01.png)
    
    - 결과(사실 결과 표만 봐서는 더 좋은지 잘 모르겠음…)
        
        ![스크린샷 2023-08-18 22.26.53.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_22.26.53.png)
        
    - 추가적으로 SIFT는 graffiti 이미지에서 강하다고 함?

## 5. Scalable Matching of Binary Features

---

- ORB가 nearest neighbor matching에서 다른 방법들(SIFT/SURF) 보다 좋다는걸 보임
- ORB의 중요한 부분 중 하나는 분산을 recovery하여 NN search를 더 효율적으로 만든다는 점임.

### 5.1. Locality Sensitive Hashing for rBRIEF

- Locality Sensitive Hashing(LSH)
    - Jaccard similarity가 높은 원소들을 같은 bucket에 넣음
        
        ![스크린샷 2023-08-18 23.04.46.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_23.04.46.png)
        
    
    ![스크린샷 2023-08-18 22.36.51.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_22.36.51.png)
    
- rBRIEF에서 이진 테스트를 위한 NN search를 수행할 때 사용

### 5.2. Correlation and Leveling

- 2개의 dataset으로 BRIEF, steered BRIEF, rBRIEF에서 고르는 이진 테스트가 서로 얼마나 correlation한 지 평가함
    
    ![스크린샷 2023-08-18 23.11.24.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_23.11.24.png)
    
    - rBRIEF가 bucket의 사이즈는 작고, 수는 많다.
    - LSH 방법은 Jaccard similarity가 높은 원소들을 같은 bucket에 넣기 때문에,
    bucket의 수가 많다는 것은 원소들이 서로 less similarity를 갖는다는 것이다.
    - 그리고 원소들이 낮은 연관성을 갖기 때문에 속도가 더 빠르다.
    - *해쉬맵의 “하나의” 인덱스에 여러 값들이 있는 것보다, 여러 값이 여러 인덱스를 갖고 있는 편이 더 빠르기 때문?*

### 5.3. Evaluation

- rBRIEF에서 LSH를 사용한 것과 SIFT 특징점들을 kd-tree로 정리한 뒤 FLANN으로 이미지 매칭의 속도와 정확도를 비교함
    
    ![스크린샷 2023-08-18 23.24.18.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_23.24.18.png)
    
    - rBRIEF LSH 최고!

## 6. Applications

---

### 6.1. Benchmarks

- 성능 좋음
    
    ![스크린샷 2023-08-18 23.33.54.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_23.33.54.png)
    
    ![스크린샷 2023-08-18 23.34.23.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_23.34.23.png)
    

### 6.2. Textured Object Detection

- dataset에 있는 물체들과 입력 이미지 안에 있는 물체에서 ORB를 수행하여 매칭시킴
- 61%의 정확도를 보인다고 함
    
    ![스크린샷 2023-08-18 23.41.22.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_23.41.22.png)
    

### 6.3. Embeded Real-time Feature Tracking

- 안드로이드 핸드폰으로 ORB를 돌림
    - 640*480 해상도에 7Hz, 1GHz ARM chip, 512MB 램
    - 하나의 이미지를 처리하는 시간
    
    ![스크린샷 2023-08-18 23.44.02.png](/images/resorce_ORB_des/%25E1%2584%2589%25E1%2585%25B3%25E1%2584%258F%25E1%2585%25B3%25E1%2584%2585%25E1%2585%25B5%25E1%2586%25AB%25E1%2584%2589%25E1%2585%25A3%25E1%2586%25BA_2023-08-18_23.44.02.png)
    

## 7. Conclusion

- 우리 개쩌는거 만들었다.
- GPU랑 SSE로 최적화해주면 더 좋아질듯
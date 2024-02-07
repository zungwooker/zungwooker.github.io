# Revisiting the Importance of Amplifying Bias for Debiasing

진행: 완료
3줄 요약: 1. Biased network가 충분히 편향되지 않아서 relative difficult score를 만들기 불충분
2. 보조 biased networks를 이용해서 biased network가 bias-aligned 샘플만 학습하게 만들자
3. Biased network가 더욱 편향되어 debiasing에 더욱 잘 응용 가능
Author: Jungsoo Lee, Jaegul Choo*
Field: Debiasing
Keyword: BiasEnsemble
Published: AAAI
URL: https://ojs.aaai.org/index.php/AAAI/article/view/26748
year: 2023
단점: • Bias-conflict 샘플이 없는 class에 대한 성능 향상은 기대할 수 없음
    ◦ Biased network가 학습하는 모든 샘플이 biased 되어있기 때문
장점: • Biased network을 이용하는 모든 method에 적용 가능
• 간단하고 biased network를 역이용하고자 하는 방법은 참신했음 그리고 성능 향상이 꽤 큼

# 3줄 요약

1. Biased network가 충분히 편향되지 않아서 relative difficult score를 만들기 불충분
2. 보조 biased networks를 이용해서 biased network가 bias-aligned 샘플만 학습하게 만들자
3. Biased network가 더욱 편향되어 debiasing에 더욱 잘 응용 가능

# Problem Statement

---

### Spurious correlation

훈련 데이터에 label과 intrinsic하지 않은 attribute 사이 강한 correlation을 bias라 칭하고, 이 bias가 학습하기 쉽기 때문에 모델의 추론이 target attribute가 아닌 bias에 의존하게 됩니다.

때문에 이러한 correlation이 없는 테스트셋에 대해서는 일반화 능력이 떨어지게 됩니다.

# Limitation of Previous Studies

---

### Utilizing Biased Network

**LfF**와 **DisEnt**는 bias-conflict 샘플에 대한 loss를 가중하여 훈련하는 방법으로 bias를 피합니다. 이 bias-conflict 샘플과 가중 정도는 biased network에 의해 결정됩니다. 즉, debiased network를 훈련하기 위해서는 biased network의 성능(bias-aligned 샘플에 대한) 또한 중요하게 작용합니다.

그런데, biased network는 bias-aligned 샘플 뿐 아니라 bias-conflict 샘플에 대해서도 일관되게 훈련합니다. Biased 되는 것이 목적인 biased network의 훈련에 bias-conflict 샘플을 사용하는 것을 오히려 방해가 될 수 있습니다.

# Observations & Motivations

---

### Observation: Poor Biased Network

![B.A: Bias-Aligned, B.C: Bias-Conflict](Revisiting%20the%20Importance%20of%20Amplifying%20Bias%20for%20D%20bc582e415f5647b4a671c716386108df/Untitled.png)

B.A: Bias-Aligned, B.C: Bias-Conflict

Biased되는 것이 목표인 biased-network는 bias-conflict에 대해서도 학습을 하기 때문에, 이 문제로 인하여 biased되는 것에 방해를 받는 것을 직접 확인할 수 있습니다. Bias label을 이용하여 훈련하였을 때보다

1. unbiased set에 대해 정확도가 높으며
2. Biased set에 대한 정확도가 낮고
3. bias-aligned 샘플에 대한 relative difficult score가 높고
4. bias-conflict 샘플에 대한 relative difficult score가 낮은 현상을 볼 수 있습니다.

마지막으로 bias label로 훈련한 biased network를 이용해서 debiasing할 때 보다 debiasing 성능이 떨어지게 됩니다. **즉, biased network의 bias attribute를 이용한 추론 성능이 부족하면 debiasing에도 영향을 주게 됩니다.**

**저자는 biased network가 더욱 biased되는 것을 방해하는 bias-conflict 샘플에 대해 biased network가 학습하지 못하게 제거하여 biased network의 훈련을 도와 궁극적으로 debiasing 성능을 올리고자 합니다.**

# Method

---

![Screenshot 2024-02-06 at 8.19.18 PM.png](Revisiting%20the%20Importance%20of%20Amplifying%20Bias%20for%20D%20bc582e415f5647b4a671c716386108df/Screenshot_2024-02-06_at_8.19.18_PM.png)

### BCD: Bias Conflicting Detector

Biased network 훈련에 사용할 bias-aligned 샘플을 골라내고, bias-conflict 샘플은 제거해야 합니다. 이를 위해서 **Bias Conflicting Detector**를 제안합니다.

**Bias Conflicting Detector(BCD)**는 보조 biased network로, 이 네트워크에 대해 낮은 confidence(softmax probability)를 기반하여 bias-conflict 샘플을 탐지하는 역할을 합니다. 약간의 iterations으로도 충분히 biased될 수 있기 때문에 작은 cost(약간의 iterations)로 훈련하여 사용합니다.

![$\tau$: Set threshold
0: Bias-conflict
1: Bias-aligned](Revisiting%20the%20Importance%20of%20Amplifying%20Bias%20for%20D%20bc582e415f5647b4a671c716386108df/Screenshot_2024-02-06_at_8.26.53_PM.png)

$\tau$: Set threshold
0: Bias-conflict
1: Bias-aligned

하지만, 하나의 BCD는 변동성이 매우 컸으며, 초기화 상태에 따라 결과가 달라지기 때문에 여러 개의 BCD를 사용하여 bias-conflict 샘플을 선택하게 됩니다.

![Visualization results of GradCAM applied to biased models with different initialization](Revisiting%20the%20Importance%20of%20Amplifying%20Bias%20for%20D%20bc582e415f5647b4a671c716386108df/Screenshot_2024-02-06_at_7.52.33_PM.png)

Visualization results of GradCAM applied to biased models with different initialization

여러 BCD 중, **confidence가 threshold보다 낮은 BCD가 과반수**일 경우, 해당 샘플을 **bias-conflict**로 판단합니다.

![PBA: Pseudo Bias Aligned
M: number of BCD
$f^{*}_{B_{i}}$: $i$th BCD](Revisiting%20the%20Importance%20of%20Amplifying%20Bias%20for%20D%20bc582e415f5647b4a671c716386108df/Screenshot_2024-02-06_at_8.33.09_PM.png)

PBA: Pseudo Bias Aligned
M: number of BCD
$f^{*}_{B_{i}}$: $i$th BCD

예를 들어, $M=5$이고 3개 이상 혹은 3개의 BCD가 bias-conflict라고 판단하면 이 샘플은 bias-conflict로 최종 판단합니다.

### Training

Biased network는 BCD가 판단한 샘플들에 대해서만 훈련하게 되고, debiased network는 LfF와 동일하게 weighted loss를 이용하여 훈련하게 됩니다.

이때 Biased network는 relative difficult score를 계산하기 위해 모든 샘플을 입력으로 받지만, BCD가 bias-aligned라고 판단한 샘플에 대해서만 update하게 됩니다.

# Experiments

---

![Untitled](Revisiting%20the%20Importance%20of%20Amplifying%20Bias%20for%20D%20bc582e415f5647b4a671c716386108df/Untitled%201.png)

# Pros & Cons

---

### Pros

- Biased network을 이용하는 모든 method에 적용 가능
- 간단하고 biased network를 역이용하고자 하는 방법은 참신했음 그리고 성능 향상이 꽤 큼

### Cons

- Bias-conflict 샘플이 없는 class에 대한 성능 향상은 기대할 수 없음
    - Biased network가 학습하는 모든 샘플이 biased 되어있기 때문

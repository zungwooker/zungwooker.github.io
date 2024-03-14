---
title: "[Paper Review] Revisiting the Importance of Amplifying Bias for Debiasing"
date: 2024-02-18
permalink: /posts/Revisiting-the-Importance-of-Amplifying-Bias-for-Debiasing/
categories:
  - debiasing
---

## 3줄 요약
1. Biased network가 충분히 편향되지 않아서 relative difficulty score를 만들기 불충분
2. 보조 biased networks를 이용해서 biased network가 bias-aligned 샘플만 학습하게 만들자
3. Biased network가 더욱 편향되어 debiasing에 더욱 잘 응용 가능

---

## Problem Statement

### Spurious correlation

훈련 데이터에 label과 intrinsic하지 않은 attribute 사이 강한 correlation을 bias라 칭하고, 이 bias가 학습하기 쉽기 때문에 모델의 추론이 target attribute가 아닌 bias에 의존하게 됩니다. 때문에 이러한 correlation이 없는 테스트셋에 대해서는 일반화 능력이 떨어지게 됩니다.

---

## Limitation of Previous Studies

### Utilizing Biased Network

**LfF**와 **DisEnt**는 bias-conflict 샘플에 대한 loss를 가중하여 훈련하는 방법으로 bias를 피합니다. 이 bias-conflict 샘플과 가중 정도는 biased network에 의해 결정됩니다. 즉, debiased network를 훈련하기 위해서는 biased network의 성능(bias-aligned 샘플에 대한) 또한 중요하게 작용합니다. 그런데, biased network는 bias-aligned 샘플 뿐 아니라 bias-conflict 샘플에 대해서도 일관되게 훈련합니다. Biased 되는 것이 목적인 biased network의 훈련에 bias-conflict 샘플을 사용하는 것을 오히려 방해가 될 수 있습니다.

---

## Observations & Motivations

### Observation: Poor Biased Network

![Untitled](https://github.com/academicpages/academicpages.github.io/assets/78302006/8ff1ac1c-5d62-42e3-8787-6648b09cbb47)
Biased되는 것이 목표인 biased-network는 bias-conflict에 대해서도 학습을 하기 때문에, 이 문제로 인하여 biased되는 것에 방해를 받는 것을 직접 확인할 수 있습니다. Bias label을 이용하여 훈련하였을 때보다
1. unbiased set에 대해 정확도가 높으며
2. Biased set에 대한 정확도가 낮고
3. bias-aligned 샘플에 대한 relative difficulty score가 높고
4. bias-conflict 샘플에 대한 relative difficulty score가 낮은 현상을 볼 수 있습니다.
마지막으로 bias label로 훈련한 biased network를 이용해서 debiasing할 때 보다 debiasing 성능이 떨어지게 됩니다. **즉, biased network의 bias attribute를 이용한 추론 성능이 부족하면 debiasing에도 영향을 주게 됩니다.**

저자는 **biased network가 더욱 biased되는 것을 방해하는 bias-conflict 샘플에 대해 biased network가 학습하지 못하게 제거하여 biased network의 훈련을 도와 궁극적으로 debiasing 성능을 올리고자 합니다**.

---

## Method

![0](https://github.com/academicpages/academicpages.github.io/assets/78302006/89211afe-548a-4a2d-a8b6-58fbd0d14bc4)

### BCD: Bias Conflicting Detector

Biased network 훈련에 사용할 bias-aligned 샘플을 골라내고, bias-conflict 샘플은 제거해야 합니다. 이를 위해서 **Bias Conflicting Detector**를 제안합니다.

**Bias Conflicting Detector(BCD)**는 보조 biased network로, 이 네트워크에 대해 낮은 confidence(softmax probability)를 기반하여 bias-conflict 샘플을 탐지하는 역할을 합니다. 약간의 iterations으로도 충분히 biased될 수 있기 때문에 작은 cost(약간의 iterations)로 훈련하여 사용합니다.

![1](https://github.com/academicpages/academicpages.github.io/assets/78302006/4f11ddec-59a4-4cad-82c7-b527beeb1c04)

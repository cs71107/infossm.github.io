---
layout: post
title:  "기하 문제를 스스로 해결하는 AI"
date:   2024-01-29
author: garam1732
tags: [mathematics, ai]
---

> 2024년 1월 17일, **Solving olympiad geometry without human demonstrations**이라는 논문이 Nature에 실렸습니다. 논문에 따르면, AlphaGeomtery라는 AI가 IMO(국제 수학 올림피아드)에 출제된 기하 문제 30개 중 무려 25개를 스스로 해결하였으며, 이는 IMO 금메달리스트의 성적 평균에 준하는 수준이라 합니다. 

해당 글에서는 논문의 내용을 간단히 정리한 후, AI의 실제 풀이를 구체적으로 살펴보며 AI의 성능을 분석해보려 합니다.

## Background

AI 연구에서 관심받던 주제 중 하나로, 스스로 증명을 구축하는 AI의 개발은 이전부터 다양한 연구가 진행되어 왔습니다. 해당 연구의 가장 큰 장벽은, 사람의 수학적 증명을 기계 친화적인 언어로 변화하는 것이 어렵다는 것입니다. 특히나 기하학의 경우 기본적인 증명 예시가 적으며, 다양한 기하학적 도구들을 표현하는 것이 어렵다고 합니다. 

해당 논문에서 오직 **순수 기하**, 즉 평면에서 다루는 유클리드 기하학 문제만을 다룹니다. 간혹 기하 부등식(길이나 각에 대한 부등식)이나 조합 기하 문제(PS에서 다루는 기하 유형)가 출제되는 경우도 있는데, 해당 연구에서 이런 문제는 배제되었습니다. 그 외에도 기계 친화적 언어로 변환이 힘든 몇몇 문제가 배제되었습니다. 

일반적으로 AI가 평면 기하를 해결하는 방법은 두 가지가 있습니다.

#### 1. Computer Algebra Method
주어진 점들을 좌표로, 직선이나 원을 방정식으로 표현한 후 연립을 통해 결론과 동일한 식이 도출되는지 검증하는 방식입니다. 가장 근원적인 증명 방법으로 볼 수 있습니다. 기본적으로 명제가 참인 이상, 전제식으로부터 결론식이 무조건 도출될 수 밖에 없기 때문에, **이론적으로** 무적 풀이법이기도 합니다. 그러나 이 방법은 그림이 조금만 복잡해지더라도 식이 기하급수적으로 복잡해집니다. 인간은 물론, AI 조차도 계산하기 어려운 수준이라 합니다.

#### 2. Search(Synthetic) Method
실질적으로 인간이 사용하는 증명 방식입니다. 참인 여러 개의 전제들로부터 여러 단계의 연역 추론을 거쳐 결론을 논리적으로 도출해내는 방식입니다. 대다수의 AI, 그리고 AlphaGeometry가 해당 방식을 사용합니다. 이때 이 추론 방식은 크게 두 가지 유형으로 나뉩니다. 먼저 기본적이라고 여겨지는 조건 명제들을 이용합니다. 전이 관계(Transitivity)를 이용한 자명한 추론부터, 일직선이나 공원점 등을 도출해내는 기본 추론을 사용합니다. 그러나 이러한 추론만으로는 도출해낼 수 없는 명제들도 있습니다. 

가장 간단한 예시를 생각해봅시다. "이등변삼각형에서 두 밑각의 크기는 같다"라는 명제를 증명해봅시다.   
중학교에서 배우는 간단한 명제입니다. 그러나 막상 시도해보면, 추론 규칙만으로는 증명이 어려울 것입니다. 이 문제를 증명하는 데에는 **보조점**이 필요합니다. 밑변의 중점이라는 새로운 점을 정의하면, 만들어지는 두 직각삼각형의 합동으로부터 결론을 도출해낼 수 있습니다. 

아무리 추론 규칙을 적용해봤자 주어진 그림을 벗어나지는 못하기 때문에, 때로는 새로운 점을 잡아 새로운 관찰을 하는 시도가 필요합니다. 실제로 대부분의 IMO 기하 문제들은 적어도 1개 이상의 보조점을 의도하는 경우가 많습니다. 문제에서 요구하는 보조점을 얼마나 잘 찾느냐에 따라 기하 실력이 나뉜다는 말도 있습니다. 그런데 이 보조점을 찾는 것이, 생각보다 어려운 작업입니다. 유한한 그림에서는 유한 개의 추론만 가능한 것에 비해, 보조점을 잡는 방법은 정말 무수히 많습니다. 특히나 AI의 입장에서 보조점을 찾는 작업은 무한한 가지를 뻗어나가는 것이라 볼 수 있습니다. 그렇기에 기하 문제를 푸는 AI에게 있어 가장 도전적인 과제가 바로 "적절한 보조점을 찾는 방법"입니다.

## Method

이제 본격적으로 AlphaGeometry에 적용된 여러가지 방법들에 대해 알아보겠습니다.

#### Training Mechanism

AlphaGeometry의 훈련 메커니즘을 간단히 정리하면 다음과 같습니다. 기본 전제들로 구성된 테이블에서 일부를 골라 consistent한 전제 집합 P를 생성합니다. 그 후 주어진 P의 전제들을 바탕으로 증명 알고리즘을 적용해 결론 N을 도출합니다. 마지막으로, 증명 과정에서 만들어진 DAG를 거슬러 올라가며 P에서 N이 도출된 경로 G(N)을 찾습니다. 이렇게 구성된 (P, N, G(N))이 하나의 훈련 데이터가 됩니다. 

위와 같이 훈련 데이터를 만들다보면, 결론 N과 독립적인 P의 부분집합이 발견될 때가 있습니다. 이때 해당 전제들은 결론에는 포함되지 않으나 증명 과정에서는 사용되는, **보조점**을 설계하는 과정으로 간주할 수 있습니다. AlphaGeometry는 이러한 방법으로 보조점을 찾는 방법을 능동적으로 찾습니다.

훈련 데이터를 구성하기 위해서는 두 가지 알고리즘이 필요합니다. 첫째, 전제들을 바탕으로 결론을 도출하는 알고리즘입니다. 해당 논문에서는 DD와 AR이라는 방법을 이용합니다. 둘째, 연역 경로 G(N)을 **최소화하는** 알고리즘이 필요합니다. 예를 들어 $a=b$와 $b=c$라는 연역 과정이 있는데, 만약 $a=c$를 직접적으로 연역할 수 있다면 굳이 이 과정을 거칠 필요가 없을 것입니다. 불필요한 연역 과정 및 전제는 풀이가 불필요하게 길어지게 만드므로, 이에 대한 최적화 알고리즘도 필요합니다.

#### Proving Mechanism

AlphaGeometry의 동작은 두 가지 단계로 구분됩니다. 먼저 Symbolic Deduction Engine이 주어진 그림에서 연역 추론을 반복해 Deduction Closure에 도달합니다. 그래도 결론이 도출되지 않았을 때, Language Model이 적합한 보조점을 선택합니다. 두 작업이 번갈아 이루어지며, 결론을 도출하거나 최대 탐색 횟수에 도달할 때까지 반복합니다. 

Symbolic Deduction Engine은 말 그대로 논리적 추론을 담당하는 DD(Deduction Database)와 대수적 계산을 담당하는 AR(Algebra Relation)로 나뉩니다. 구체적으로 DD는 definite Horn clause의 형태에 대한 추론 규칙에 따릅니다. AlphaGeometry는 그래프 자료 구조를 사용하여 등식이나 공점선, 공원점을 찾아낸다고 합니다. 반면 AR은 대수적인 분석을 담당하며, 각이나 길이 비율, 거리 등을 계산합니다. 주어진 각이나 길이 조건은 일반적으로 선형 일차 방정식 $a-b=c-d$의 형태로 표현이 가능합니다. 예를 들어 각이 같다는 조건은 두 각을 이루는 직선들의 기울기 차이가 같다는 식으로 표현 가능합니다. 길이 비율의 경우, 로그를 취하면 마찬가지로 차이가 같다는 식으로 표현됩니다. 이때 여러 개의 선형 일차식으로부터 도출되는 결론들은 가우스 소거법을 적용하여 쉽게 알아낼 수 있습니다.

#### Optimization

짧고 이해하기 쉬운 풀이를 만들기 위해서는, 최소한의 연역 과정을 뽑아내는 최적화 알고리즘이 필요합니다. 구체적으로, Symbolic Deduction Engine을 구성하는 각 알고리즘에 대한 역추적 알고리즘은 다음과 같습니다. 

먼저 DD의 경우, Transitive Relation이나 공선점, 공점원 등을 다루게 됩니다. 가장 간단한 Transitive Relation의 경우, 간단히 등호 관계를 변으로 표현한 그래프에서 너비 우선 탐색을 통해 최단 경로를 찾을 수 있을 것입니다. 공선점이나 공점원은 보다 복잡한데, 3개나 4개의 점이 하나의 관계를 이루기 때문에 hypergraph를 이용해야 합니다. 이때의 최적해는 hypergraph에서 MST를 찾는 문제가 됩니다. 논문에 따르면 이는 NP-Hard 문제이며, 해당 연구에서는 적절한 그리디 알고리즘을 적용하였다고 합니다.

AR의 경우에는 아까 가우스 소거법에서 사용된 행렬을 이용합니다. 행렬의 행이 각각의 식을 의미하는데, 이때 최소한의 식을 선택하는 문제는 해당 행렬에 대한 mixed integer linear programming problem으로 간주할 수 있습니다.

## Analysis

해당 논문을 검색해보면, AlphaGeometry의 성능 검증에 사용된 30개의 IMO 문제와 그에 대한 AI의 풀이를 직접 읽어보실 수 있습니다. 이 중 몇 개를 골라 살펴보면서 현재 AlphaGeometry가 가진 한계와 성장 방향을 논의해보려 합니다. 

IMO는 총 6문제이며, 이틀에 걸쳐 문제를 풀게 됩니다. IMO는 일반적으로 난이도 순으로 출제되므로, 1번과 4번이 Easy, 2번과 5번이 Medium, 3번과 6번이 Hard한 문제입니다. 연구에 사용된 30개의 문제들을 난이도에 따라 분류하여 성공율을 비교해보면 다음과 같습니다.

|Easy|Medium|Hard|Total|
|:---:|:---:|:---:|:---:|
|15 / 16|8 / 8|3 / 6|26 / 30|

다만, 여기서 조정되어야 할 점이 하나 있습니다. 문제 리스트를 살펴보면, 한 문제를 두 개의 문제로 나누어(A와 B) 따로 문제를 해결하도록 한 경우가 있습니다. 문제를 나눈 기준은 다음과 같았습니다:

1. 양방향 증명을 위해 각 방향을 따로 해결하도록 지시한 경우   
2. 다섯 개 이상의 점이 한 원 위에 있음을 보이기 위해 네 점씩 따로 증명을 지시한 경우   
3. 기계 친화적 언어로 표현하기 위해 본래는 추측해야 할 내용을 전제에 포함시켜 증명을 지시한 경우(A로만 표시되어 있음)   

이러한 경우들에 대해서, 우선 A와 B를 합쳐서 하나의 문제로 봐야함이 적절합니다. 또한, 하나라도 해결하지 못했을 경우 해결하지 못한 것으로 간주하는 것이 적절합니다. 3번 상황의 경우, 난이도를 결정짓는 추측 과정을 생략해버렸기 때문에, 해당 문제를 온전히 해결했다고 보긴 어렵습니다. 이에 따라 조정된 비율은 다음과 같습니다.

|Easy|Medium|Hard|Total|
|:---:|:---:|:---:|:---:|
|14 / 15|5~7 / 7|3 / 6|22~24 / 28|

난이도별로 문제를 몇 개씩 골라 AlphaGeometry의 풀이를 분석해보겠습니다.

> 원 논문에서는 일부 문제에 대해 AlphaGeomtery의 풀이와 Evan Chen님의 풀이를 비교하여 서술하였는데, 본 글에서는 제가 직접 풀어본 풀이를 사용하여 비교하려 합니다.

#### Easy Problem

가장 쉬운 문제들답게, 대부분의 문제가 풀렸습니다. 다만 풀이가 다소 비직관적으로 느껴졌던 문제들이 있어 이에 대해 논의해보려 합니다.

![](../assets/images/garam1732/AIGEO/2020P1.png)

이 문제는 2020년 1번으로 출제된 문제입니다. AlphaGeometry는 3개의 보조점을 잡아 35줄의 풀이를 완성하였습니다. 그런데 풀이를 읽어보면, 보조점의 직관성이 떨어질뿐더러 풀이가 지나치게 복잡한 감이 있습니다. 실제로 이 문제는 세 직선의 실제 교점으로 추정되는 $\triangle PAB$의 외심 $O$를 잡으면 바로 쉽게 풀립니다. AlphaGeometry가 이러한 접근을 찾지 못한 점에 대해 두 가지 요인이 있다고 생각됩니다.

첫째, 이 문제에서 요구되는 보조점은 단순히 증명의 중간 단계로 소모되는 도구가 아니라 실질적으로 결론에도 영향으로 미칩니다. AlphaGeometry가 학습하던 보조점은 결론과 독립적인 전제에 대한 이야기였습니다. 그런데 이 문제의 경우, 외심 $O$를 잡은 후 두 각의 이등분선이 각각 $O$를 지난다는 새로운 명제를 증명하게 됩니다. 즉, 보조점이 더 이상 결론과 독립적이지 않게 됩니다. 이런 경우에서 보조점의 역할은 기존의 결론 N에서 도출해내기 더 쉬운 새로운 결론 N'을 잡을 수 있게 해줍니다.  

다른 여러 문제들에서도 이러한 경향이 나타났습니다. 대표적으로, 2005년 5번 문제는 이전에 언급했던 AB 문제 유형 중 세 번째에 속하는 경우였습니다. 이 문제로 치자면 보조점 $O$의 존재를 미리 알려주고 풀이를 진행한 셈입니다. 이렇듯 AlphaGeometry가 보조점에 대해 아직 국소적인 관점을 가지고 있다 보니, 보다 직관적이고 휴리스틱한 보조점을 찾는데에는 어려움을 겪는 것으로 보입니다. 

둘째, AlphaGeometry는 기본 연역 규칙만을 이용하여 정보를 찾아내기 때문에, 귀류법과 같은 특수 연역 규칙을 사용하는 증명에는 취약한 것으로 보입니다. 이에 대한 대표적인 예시로 **동일법**이라는 기하 증명 방법이 있습니다. 이를 형식적으로 정리하면 다음과 같습니다.

1. 조건 $q$를 만족하는 점은 조건 $p$를 만족한다.
2. 조건 $p$를 만족하는 점이 **유일하게** 존재한다.
3. 따라서 조건 $p$를 만족하는 점은 조건 $q$를 만족한다.

한마디로, **유일성**을 이용하여 동치성을 보장하는 증명 방법입니다. 예를 들어, 다음 명제를 증명해봅시다.   
"$AB\ne AC$인 삼각형 $ABC$에서 변 $BC$의 수직이등분선과 각 $A$의 이등분선의 교점 $M$은 삼각형 $ABC$의 외접원 위에 있다."

이 문제를 동일법을 이용하여 증명해보면 다음과 같습니다.

1. 삼각형 $ABC$의 외접원에서 호 $BC$의 중점을 $M'$이라 하자.
2. 호의 중점의 성질에 따라 $M'$은 변 $BC$의 수직이등분선 위에 있으며 각 $A$의 이등분선 위에도 있다.
3. 두 직선의 교점은 **유일하게** 결정되므로, 그 점은 $M$이어야 한다.
4. 따라서 $M=M'$이며, 이는 $M$이 호 $BC$의 중점임을 의미한다.

제가 풀이에서 사용한 논리 또한 동일법을 사용하면 쉽게 증명이 가능합니다. 그러나 AlphaGeometry는 이런 방식의 추론을 학습하지 않았기 때문에, 억지로 기본 연역 규칙만으로 증명하는 과정에서 풀이가 지나치게 길어진 것으로 예상됩니다.

#### Hard Problem

절반은 증명에 성공하였고, 절반은 증명에 실패하였습니다. 그럼 AlphaGeometry는 어떤 문제를 상대적으로 쉽다고 생각하고, 어떤 문제를 상대적으로 어렵다고 생각할까요? 보통 어려운 기하 문제들은 특별한 아이디어를 요구하거나, 그렇지 않은 대신 여러 단계의 추론을 요구합니다. 기본적으로 AlphaGeometry는 매우 뛰어난 연산 수행 능력을 가지고 있기 때문에, 주어진 조건 내에서 수 초 이내에 deduction closure에 쉽게 도달할 수 있습니다. 따라서 후자와 같은 문제 유형은 인간보다 AI가 증명을 성공하기에 유리한 환경이라 볼 수 있습니다. 반면 전자의 유형은 인간에게나 AI에게나 상당히 어려운 문제에 속합니다. 이런 문제에서는 상당히 고급적인 기하 지식을 요구하며, 기하학적 구조에 대한 정확한 이해를 필요로 합니다. 이런 아이디어들은 단순한 연역 추론으로는 도출하기 어려운 경우가 많습니다.

![](../assets/images/garam1732/AIGEO/2008P6.png)

이 문제는 2008년 6번에 출제된 문제로, 위 사진은 실제 논문에 실린 부분을 가져온 것입니다. 30문제 중 가장 어려운 문제로 선정되었는데, 그 난이도에 비해 풀이는 짧은 편입니다. 이 문제에 사용된 핵심 아이디어는 세 원의 닮음 관계를 적극적으로 이용하는 것입니다. 두 원의 닮음의 중심을 잡았을 때 각 원의 대응점들과 일직선 관계에 있다는 사실을 이용합니다. 이런 성질을 Homothety라 부르는데, 이는 닮음의 기하학적 구조를 직접적으로 응용하는 방식으로 단순 연역으로는 흉내내기 어렵습니다. 이러한 분석은 실제 논문에서도 언급되어 있습니다.

## Conclusion

AlphaGeometry는 단순 연역 추론과 자체적인 보조점 알고리즘을 통해 상당수의 IMO 기하 문제를 해결하였습니다. 다만, 연역 수준 자체는 아직 초보적인 수준에 있어 증명의 가독성이나 구성에 아쉬운 점이 많았던 것 같습니다. 논문에서도 언급된 피드백이지만, 더 휴리시틱한 추론 규칙, 동일법과 같은 새로운 추론 규칙, 그리고 올림피아드에서 자주 사용된 기법들이 더 반영되면 충분히 기대할만한 결과가 나올 거라 생각합니다. 

기하학은 수학의 다양한 분야들 중 단순한 조건문의 형태로 표현하기 가장 쉬운 분야입니다(근본적으로 유한 개의 점들 사이의 유한한 관계로 표현됩니다). 어느 정도 기하학을 학습할 수 있는 AI의 개발은 앞으로 증명 AI의 연구에 있어 초석이 되는 연구라 할 수 있습니다. 극복해야 할 부분이 많겠지만, 앞으로 증명 AI의 발전이 기대되는 바입니다.

## Reference

[1] Trinh, T.H., Wu, Y., Le, Q.V. et al. Solving olympiad geometry without human demonstrations. Nature 625, 476–482 (2024).
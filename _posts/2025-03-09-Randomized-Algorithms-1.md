---
title: Introduction [Randomized Algorithms]
author: Mingeon Jeong
date: 2025-03-09 21:53:00 +0900
categories: [Randomized Algorithms]
tags: [algorithm]
---

# Introduction

## Brief summary

- RandQS
- Min-Cut
- Las Vegas and Monte Carlo
- Binary Planar Partitions
- Probabilistic Recurrence
- Analysis in terms of Computation & Complexity Theory

### RandQS
    
> 피벗을 랜덤으로 설정하는 퀵소트

- Notation : $S_{(i)}$ means the $i$-th smallest element in the set $S$
- $X_{ij}$를 $S_{(i)}$와 $S_{(j)}$를 비교했는지 여부로 두면
    
    $$
    \mathbf{E}[ \sum_{i=1}^n \sum_{j>i} X_{ij}] = \sum_{i=1}^n \sum_{j>i} \mathbf{E}[X_{ij}] = \sum_{i=1}^n \sum_{j>i} p_{ij}
    $$
    
    가 평균 시간복잡도
    
- 자르는 과정을 각 원소를 노드로 가지는 바이너리 트리로 만든다고 생각하면 $S_{(i)}$와 $S_{(j)}$가 조상-자손 관계(직계일 필요는 없음)일 때만 $X_{ij}=1$
    - 중위 순회 결과 = 정렬된 배열
- 이제 두 가지 관찰을 하자.
    - 레벨 순회(BFS) 결과를 순열 $\pi$라 하자. $S_{(i)}, S_{(i+1)}, \cdots, S_{(j)}$ 중에서 $\pi$에 가장 빠르게 나타나는 수가 $S_{(k)}$일 때, $k \in \{i, j\}$와 $X_{ij}=1$은 필요충분조건
    - $\pi$는 모든 순열이 가능하므로 위 조건이 만족될 확률은 $2/(j-i+1)$
- 따라서
    
    $$
    \sum_{i=1}^n \sum_{j>i} \frac{2}{j-i+1} \leq 2n \sum_{k=1}^n \frac{1}{k} = 2nH_n \sim 2n\log n + \Theta(n)
    $$
        
### Min-Cut
    
> 연결 그래프를 비연결 상태로 만들기 위해 잘라야 하는 간선 개수의 최솟값 찾기

- 랜덤하게 두 정점을 하나로 합치고, 이때 만들어지는 self-loop를 제거하는 “수축” 연산 이후에 min-cut의 크기는 감소하지 않는다.
- 고로, 두 개의 정점이 남을 때까지 “수축”을 하면 두 정점 사이에 연결된 간선의 수는 min-cut 이상이다.
- 그렇다면, 랜덤 간선을 골라서 위 과정을 실행할 때 남은 간선의 수가 min-cut이 될 확률은 얼마일까?
- Min-cut의 집합 $C$(여러 개 존재할 수 있지만 아무거나 하나 잡는다)의 원소가 지금까지 한 번도 선택되지 아니했을 때**(조건부확률)**, $i$번째 “수축”에서 선택되지 않을 확률 $\epsilon_i$라 하면
    - 남은 정점이 $n$개이고, Min-cut의 크기가 $k$일 때, 모든 정점의 차수가 $k$ 이상이어야 하므로 간선의 개수가 $kn/2$개 이상 (수축 연산 이후에도 self-loop는 없음을 상기하라)
    - 즉, $\epsilon_i \geq 1 - 2/(n-i+1)$, $i$번째 수축에서 정점 개수는 $n-i+1$개이므로
    - 결국, 남은 간선들이 모두 $C$의 원소일 확률은
        
        $$
        \prod_{i=1}^{n-2} \epsilon_i \geq \frac{2}{n(n-1)}
        $$
        
- 즉, 반복을 $O(n^2)$번 시행하면 min-cut을 찾지 못할 확률은
    
    $$
    \left( 1 - \frac{2}{n^2} \right)^{cn^2} < e^{-2c} \qquad \textrm{by definition of }e
    $$
    
    미만이 되므로 충분히 작게 할 수 있다.
        
### Las Vegas and Monte Carlo
- 랜덤을 활용한 알고리즘은 크게 두 가지로 분류할 수 있다
- Las Vegas : “정답”을 구하는 과정에서 랜덤을 사용하여 프로그램의 효율을 향상시키는 방법
- Monte Carlo : “정답”을 구할 확률이 있는 효율적인 랜덤 알고리즘을 사용하는 방법
    - 결정 문제에 대한 “답”을 낼 때, YES와 NO 모두 틀릴 가능성이 있다면 two-sided error를 가지고 있다고 하고, 하나만 틀릴 가능성이 있다면 one-sided error를 가지고 있다고 한다.
- **답에 대한 검증이 가능하다면**, 모든 Monte Carlo 알고리즘은 Las Vegas 알고리즘으로 환원할 수 있다.

---

어떤 문제에 대한 Monte Carlo 알고리즘의 해가 맞을 확률이 $\gamma(n)$이며, 실행 시간이 $T(n)$, 검증 시간이 $t(n)$일 때, 시간복잡도의 기댓값이

$$
T(n)+t(n) \over \gamma(n)
$$

인 Las Vegas 알고리즘을 만들 수 있다.

---
    
### Binary Planar Partitions
    
> 겹치지 않는 선분들이 주어질 때, 하나의 영역을 두 개로 가르는 선을 긋는 것을 반복해, 각 영역에 최대 하나의 선분 또는 선분의 일부가 포함되도록 자르는 방법 찾기

- 다음 알고리즘을 생각하자

---
주어진 선분 집합을 $S = \{ s_1, s_2, \cdots, s_n \}$이라 하자.

1. 길이 $n$의 순열 $\pi$을 하나 잡는다.
2. **(while)** 둘 이상의 선분을 포함하는 영역 $R$이 존재하면, 선분 $s_i$를 포함하는 선을 경계로 그어 $R$을 둘로 나눈다. 여기서 $i$는 $s_i$가 영역 $R$을 나누면서 $\pi$에서 가장 앞에 등장하는 숫자이다.

* 경계와 일치하는 선분은 위든 아래든 한 곳으로 정해서 포함시킨다 *
---

- 위와 같은 알고리즘을 고안하면, 선분 $v$는 다른 선분 $u$를 포함하는 반직선에 의해서만 잘릴 수 있다. 즉, 선분이 잘리는 횟수는 최악에 $O(n^2)$번이고, 영역의 개수도 최악에 $O(n^2)$개이다.
- 또한, 영역 개수의 기댓값은 $O(n \log n)$이다.
    - 위의 알고리즘을 시행한 후의 결과를 보면, 선분 $v$는 다른 선분 $u$를 포함하는 반직선에 의해서만 잘릴 수 있기 때문에, 다음 함수를 정의할 수 있다.
        
        $index(u, v)$를 $u$를 포함하는 반직선이 $v$를 몇 번째로 잘랐는지로 한다. 자르지 않았으면 $\infty$이다.
        
    - 이제, $u$가 $u_1, u_2, \cdots, u_{k-1}$을 자르고 $v$를 자를 확률을 생각해보자. $\pi$에서 가장 앞에 등장하는 숫자를 인덱스로 가지는 선분으로 자르기 때문에, $u$가 $u, u_1, u_2, \cdots, u_{k-1}, v$ 중에서 가장 앞에 등장해야 한다. 이 확률은 $1/(k+1)$이다.
    - 즉, $u$가 자르는 선분의 개수가 많아질수록, 선분을 하나 더 자를 확률은 줄어들게 된다.
        - $u$가 처음 두 개를 자를 때는 각각의 기댓값이 $1/2$이다. 하지만, 그 다음 두 개는 각각이 $1/3$이 되고, 이렇게 점점 감소한다.
        - 합을 낸다면, 기댓값은 아무리 커야 $2 [\frac{1}{2} + \frac{1}{3} + \cdots + \frac{1}{n/2}] \sim 2 \log n$이다.
    - 따라서, 선분의 개수를 곱하면 선분이 절단되는 횟수의 기댓값이 $O(n \log n)$임을 얻고, 이어서 영역 개수의 기댓값도 $O(n \log n)$임을 얻는다.

### Probabilistic Recurrence
### Analysis in terms of Computation & Complexity Theory
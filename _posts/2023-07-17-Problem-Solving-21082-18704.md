---
title:  "2023-07-17 Problem Solving"
mathjax: true
layout: post
---

2023-07-17 Problem Solving 일지

문제 번호: 21082(P1), 18704(D5)


## 문제 개요

- 문제 번호: 21082번 (https://www.acmicpc.net/problem/21082)
- 문제 제목: Assignment Problem
- 문제 출처: PTZ camp > 2021w > Day 5 A번 / Open cup > 2020/2021 > Stage 11 A번
- 난이도: P1
- 알고리즘: `#bruteforcing`
- 풀이 시간: 16분
- 문제 요약: $$n$$명의 후보자들 중 $$m$$명 골라 각 자리에 앉히려고 한다. 한 명의 후보자는 최대 한 자리만 선택될 수 있다. 우리의 목적은 각 $$m$$명을 선정하였을 때 얻는 이득을 최대로 만드는 것이다. 하지만 정확한 값은 알 수 없고 값의 대소만 알 수 있다. $$m$$개의 자리에 대한 값의 순위를 나타내는 길이 $$n$$의 순열(permutation) $$m$$개가 주어질 때, $$i$$번째 후보자가 선택될 수 있는 값이 존재하는 후보자들의 개수와 후보자를 오름차순으로 출력하라.
- 제약 조건
    - $$1≤n≤1000$$이다.
    - $$1≤m≤11$$이다.

## 만점 풀이

$$m$$의 제한에서 $$O(m!)$$과 유사한 시간복잡도를 가질 것이라는 것을 굉장히 쉽게 파악할 수 있었다. 그로부터 다음와 같은 명제를 얻어낼 수 있었고, 이 명제를 증명하고자 한다.

> 모든 행렬 $$A$$에 의한 결과는, $$m$$개의 자리를 적당한 순서로 우선순위를 매겨 얻은 결과와 같은 적당한 순서가 존재한다. 


### < 증명 >

&emsp;어떤 행렬 $$A$$에 의한 결과로 $$i$$번째 자리에 선택된 후보자의 번호를 나타낸 수열 $$a_1, a_2, \cdots, a_m$$을 생각하자. 만약 수열에 있는 모든 $$a_i$$가 $$i$$번째 자리에서 가장 이득이 최대가 되는 후보자가 아니라고 가정하자. 그러면 각 $$i$$에 대해 첫 번째 후보자가 수열 $$\{ a \}$$에 포함된다는 뜻이다. 그렇지 않으면 $$a_i$$ 대신 첫 번째 후보자를 넣으면 되기 때문이다.

&emsp;이제 일반성을 잃지 않고 $$a_2$$가 1번째 자리에서 가장 이득이 되는 후보자라고 하자. 2번째 자리에서 가장 이득이 되는 후보자는 $$a_2$$일 수 없으므로 $$a_1$$이거나 $$a_i (i≠1, 2)$$일 것이다. 만약 $$a_1$$이라면 $$a_1$$ 대신 $$a_2$$를, $$a_2$$ 대신 $$a_1$$을 자리에 앉혔을 때 더 이득이 커지므로 이득이 최대가 된다는 본래의 가정에 모순이다. 만약 $$a_i (i≠1, 2)$$라면 다시 위와 같은 방법으로 반복할 수 있다. 즉 결과적으로 모순이 발생하게 된다.

&emsp;따라서, $$a_i$$가 $$i$$번째 자리에서 가장 이득이 최대가 되는 후보자인 $$i$$가 존재하고, 이는 $$i$$번째 자리를 가장 높은 우선순위로 둔 경우이다. 이제 $$a_i$$를 제외하고 각 $$a_j (j≠i)$$가 몇 번째로 큰 이득을 주는 후보자인지 계산하자. 이렇게 되면 위와 같은 경우가 되므로 다시 가장 이득이 최대인 후보자를 앉힌 자리가 존재하고, 그 자리를 두 번째 우선순위로 둔 경우로 이야기할 수 있다. 따라서, 결론적으로 모든 행렬 $$A$$에 의한 결과는 $$m$$개의 자리를 적당한 순서로 우선순위를 매겨 얻은 결과 중 하나에 속한다.
    

이제 우리는 존재함을 보이기만 하면 된다. 존재함이 보이는 것은 매우 쉬운데, 정해진 우선순위에 대하여 우선순위가 높은 것부터 각 후보자들의 등수에 따른 $$A$$ 값의 차이가 더 크도록 하면, 전체 합이 제일 커지도록 고르기 때문에 똑같은 결과를 얻을 수 있다.

즉, 가능한 $$m!$$가지의 우선순위에 대하여 얻는 결과를 기록하여 한 번이라도 나온적 있으면 선택될 수 있는 것으로 볼 수 있다. 아래는 재귀함수를 이용하여 구현한 $$O(m \times m!)$$ 코드이다. 구현은 매우 쉽다. ~~개인적으로는 코드포스에 어울리는듯~~

```c++
#include <bits/stdc++.h>
using namespace std;
int m, n, arr[11][1010];
int ans[1010];

vector<int> pos;
int ch[11], use[1010];
void solve(int L) {
    if (L == m) {
        for (int i = 0; i < m; i++) ans[pos[i]] = 1;
        return;
    }
    for (int i = 0; i < m; i++) {
        if (ch[i] == 1) continue;
        ch[i] = 1;
        int j = 0;
        while (use[arr[i][j]] == 1) j++;
        use[arr[i][j]] = 1;
        pos.push_back(arr[i][j]);
        solve(L + 1);
        pos.pop_back();
        use[arr[i][j]] = 0;
        ch[i] = 0;
    }
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    cin >> n >> m;
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) cin >> arr[i][j];
    }

    solve(0);

    int cnt = 0;
    for (int i = 1; i <= n; i++) cnt += ans[i];
    cout << cnt << "\n";
    for (int i = 1; i <= n; i++) if (ans[i] == 1) cout << i << " ";

    return 0;
}
```

## 문제 개요

- 문제 번호: 18704번 (https://www.acmicpc.net/problem/18704)
- 문제 제목: Awesome Shawarma
- 문제 출처: ACPC 2018 A번
- 난이도: D5
- 알고리즘: `#centroid_decomposition #divide_and_conquer #tree`
- 풀이 시간: 44분
- 문제 요약: 정점의 개수가 $$N$$인 어떤 트리에서 어떤 간선 $$E$$를 제거했을 때 그래프가 연결 그래프가 아닌 간선 $$E$$를 bridge라고 하자. 임의의 두 정점 $$s$$와 $$e$$를 연결하여 간선을 추가할 때, bridge의 개수가 닫힌 구간 $$[L, R]$$에 속하는 정점쌍 $$(s, e)$$의 개수를 출력하라.
- 제약 조건
    - $$2≤N≤10^5$$이다.
    - $$0≤L≤R≤N-1$$이다.

## 만점 풀이

두 정점 $$s$$와 $$e$$를 연결하였을 때 bridge의 개수는 생긴 사이클에 포함되지 않는 간선의 개수와 같다. 즉, 전체 $$N$$개의 간선 중 $$dis(s, e)+1$$개의 간선은 bridge가 아니게 된다. 즉, 문제의 조건을 달리 말하면 $$L≤N-1-dis(s, e)≤R$$인 정점쌍 $$(s, e)$$의 개수를 세는 것이다.

즉, 길이가 특정 구간 내에 있는 정점쌍 $$(s, e)$$의 개수를 세는 문제로 환원되며, 이는 센트로이드를 이용한 분할 정복으로 $$O(n \log^2n)$$의 시간에 구할 수 있음이 알려져 있다. 이 코드를 짜면 AC를 받을 수 있으며, 시간은 2초정도 걸린다.

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;

int n, l, r;
vector<int> adj[101010];
int sz[101010], ch[101010], dis[101010];

void getsz(int v, int prev) {
    sz[v] = 1;
    for (int i = 0; i < adj[v].size(); i++) {
        int next = adj[v][i];
        if (prev == next) continue;
        if (ch[next]) continue;
        getsz(next, v);
        sz[v] += sz[next];
    }
}

int getct(int v, int prev, int S) {
    for (int i = 0; i < adj[v].size(); i++) {
        int next = adj[v][i];
        if (prev == next) continue;
        if (ch[next]) continue;
        if (sz[next] * 2 > S) return getct(next, v, S);
    }
    return v;
}

vector<int> value;
void getdis(int v, int prev) {
    value.push_back(dis[v]);
    for (int i = 0; i < adj[v].size(); i++) {
        int next = adj[v][i];
        if (prev == next) continue;
        if (ch[next]) continue;
        dis[next] = dis[v] + 1;
        getdis(next, v);
    }
}

ll _count(int v, bool isct) {
    ll cnt = 0;
    dis[v] = 0;
    getdis(v, 0);
    sort(value.begin(), value.end());
    if (!isct) for (int i = 0; i < value.size(); i++) value[i] += 1;
    for (int i = 0; i < value.size(); i++) {
        ll st = lower_bound(value.begin() + i + 1, value.end(), l - value[i]) - value.begin();
        ll ed = upper_bound(value.begin() + i + 1, value.end(), r - value[i]) - value.begin();
        cnt += ed - st;
    }
    value.clear();
    return cnt;
}

ll divide(int root, int S) {
    if (S == 1) return 0;
    ll ret = 0;
    getsz(root, 0);
    int ct = getct(root, 0, S);
    getsz(ct, 0);
    ch[ct] = 1;
    for (int i = 0; i < adj[ct].size(); i++) {
        int next = adj[ct][i];
        if (ch[next]) continue;
        ret += divide(next, sz[next]);
    }

    ret += _count(ct, true);
    for (int i = 0; i < adj[ct].size(); i++) {
        int next = adj[ct][i];
        if (ch[next]) continue;
        ret -= _count(next, false);
    }

    ch[ct] = 0;
    return ret;
}

void solve() {
    cin >> n >> l >> r;
    l = n - 1 - l;
    r = n - 1 - r;
    swap(l, r);
    for (int i = 1; i < n; i++) {
        int a, b;
        cin >> a >> b;
        adj[a].push_back(b);
        adj[b].push_back(a);
    }
    cout << divide(1, n) << "\n";

    for (int i = 1; i <= n; i++) adj[i].clear();
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    int t;
    cin >> t;
    while (t--) solve();

    return 0;
}
```
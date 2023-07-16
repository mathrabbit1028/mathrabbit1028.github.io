---
title:  "2023-07-16 Problem Solving"
mathjax: true
layout: post
---

2023-07-16 Problem Solving 일지
문제 번호: 18180(P2), 1839(D4)


## 문제 개요

- 문제 번호: 18180번 (https://www.acmicpc.net/problem/18180)
- 문제 제목: Saba1000kg
- 문제 출처: CERC 2019 S번
- 난이도: P2
- 알고리즘: `#disjoint_set`
- 풀이 시간: 102분
- 문제 요약: 정점 $$N$$개, 간선 $$E$$개인 그래프가 입력으로 주어질 때, 다음의 질의 $$P$$개를 해결하라:
    - 수 $$M$$개가 주어질 때, 그 $$M$$개의 정점만을 통해 갈 수 있는 정점들을 하나의 그룹으로 만들자. 이때 그룹의 개수를 출력하라.
- 제약 조건
    - $$1≤N≤10^5$$
    - $$0≤E≤10^5$$
    - $$1≤P≤10^5$$
    - $$1≤M≤N$$
    - $$\sum M ≤ 10^5$$

## 만점 풀이

기본적으로 분리 집합(disjoint set)을 사용해야 함을 쉽게 떠올릴 수 있다. 이제 문제는 시간 복잡되인데, 분리 집합을 사용하기 위해 $$M$$개의 정점들끼리의 연결 관계를 직접 구한다면 적어도 $$O(M^2)$$의 시간이 필요하므로 TLE가 나게 된다. 이를 해결할 수 있는 방법을 생각해보자.

먼저, $$M$$개의 정점 목록에 속하는 어떤 정점 $$V$$에 대해 나머지 $$M-1$$개의 정점이 연결되어 있는지 판단하는 알고리즘을 생각해보자. 그 전에, $$\vert V \vert$$는 정점 $$V$$와 연결된 정점의 수이다.

- 알고리즘 1: $$V$$와 연결된 정점을 `ch` 배열에 마킹한 후, 나머지 $$M-1$$개의 정점의 `ch` 값이 1인지 확인한다. 시간복잡도는 $$O(M+ \vert V \vert)$$이다.
- 알고리즘 2: $$V$$와 연결된 각각의 정점에 대해, 나머지 $$M-1$$개의 정점에 포함되어 있는지 이분탐색을 통해 확인한다. 이를 위해 $$M$$개의 정점을 번호 순으로 정렬한다. 시간복잡도는 $$O(\vert V \vert \log M)$$이다.

위 두 알고리즘 모두 장단점이 있다. 첫 번째 알고리즘은 $$\vert V \vert$$가 작은 경우에 더 빠르고, 두 번째 알고리즘은 $$M$$이 작은 경우에 더 빠르다. 이를 바탕으로 $$\vert V \vert$$와 $$M$$의 대소에 따라 알고리즘을 고르는 휴리스틱을 만들 수 있다.

하지만 이렇게 코드를 구성하더라도 TLE를 받는데, 그 이유는 $$M$$개의 정점을 번호 순으로 정렬되어 있기 때문에 1번 정점이 모든 질의에 포함되고 1번 정점과 연결된 정점의 수가 매우 많다면 두 알고리즘 모두 최악의 시간복잡도를 보이기 때문이다. 이를 해결하는 방법 중 하나는 $$M$$개의 정점을 정렬할 때 **각각의 정점과 연결된 정점의 수를 기준으로 오름차순으로 정렬**하는 것이다. 이렇게 하면 위의 유일한 worst case를 해결할 수 있다.

이외에도 풀이는 다양하다. sqrt decomposition을 사용할 수도 있고, 각 $$\vert V \vert$$의 값을 최대한 균일하게 맞추는 방법 등등

아래는 위 방법으로 구현한 코드이다.

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
int n, m, q, root[101010], cnt[101010], ch[101010], reord[101010], ans;
vector<int> _adj[101010], adj[101010];

int _find(int v) {
    if (v == root[v]) return v;
    else return root[v] = _find(root[v]);
}

void _union(int u, int v) {
    //cout << u << " " << v << "\n";
    v = _find(v);
    u = _find(u);
    if (v == u) return;
    ans--;
    if (cnt[v] > cnt[u]) {
        root[u] = v;
        cnt[v] += cnt[u];
    }
    else {
        root[v] = u;
        cnt[u] += cnt[v];
    }
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    cin >> n >> m >> q;
    vector< pair<int, int> > temp;
    for (int i = 1; i <= m; i++) {
        int a, b;
        cin >> a >> b;
        temp.push_back({a, b});
        _adj[a].push_back(b);
        _adj[b].push_back(a);
    }

    pair<int, int> v[101010];
    for (int i = 1; i <= n; i++) {
        v[i].first = _adj[i].size();
        v[i].second = i;
    }
    sort(v + 1, v + n + 1);
    for (int i = 1; i <= n; i++) reord[v[i].second] = i;

    for (int i = 0; i < m; i++) {
        int a = temp[i].first, b = temp[i].second;
        a = reord[a], b = reord[b];
        if (a > b) swap(a, b);
        adj[a].push_back(b);
    }

    while (q--) {
        int k; vector<int> arr;
        cin >> k;
        for (int i = 1; i <= k; i++) {
            int t;
            cin >> t;
            t = reord[t];
            arr.push_back(t);
            root[t] = t;
            cnt[t] = 0;
        }
        ans = k;
        sort(arr.begin(), arr.end());
        for (int i = 0; i < arr.size(); i++) {
            //cout << adj[arr[i]].size() << "\n";
            if (k - i < adj[arr[i]].size()) {
                for (int j = 0; j < adj[arr[i]].size(); j++) {
                    ch[adj[arr[i]][j]] = 1;
                }
                for (int j = i + 1; j < arr.size(); j++) {
                    if (ch[arr[j]] == 1) _union(arr[i], arr[j]);
                }
                for (int j = 0; j < adj[arr[i]].size(); j++) {
                    ch[adj[arr[i]][j]] = 0;
                }
            }
            else {
                for (int j = 0; j < adj[arr[i]].size(); j++) {
                    int v = adj[arr[i]][j];
                    auto it = lower_bound(arr.begin(), arr.end(), v);
                    if (it != arr.end() && *it == v) _union(arr[i], v);
                }
            }
        }
        cout << ans << "\n";
    }

    return 0;
}
```

## 문제 개요

- 문제 번호: 1839번 (https://www.acmicpc.net/problem/1839)
- 문제 제목: 트리 모델 만들기
- 문제 출처: POI 2003/2004 > Stage 1 2번
- 난이도: D4
- 알고리즘: `#dp_tree #parametric_search #greedy`
- 풀이 시간: 57분
- 문제 요약: 모든 정점 사이의 간격이 1인 트리를 노끈을 통해 구성하는 아래와 같은 모델을 따를 때, 주어진 트리를 만들기 위해 필요한 최소 노끈 개수와 그 때 가장 긴 노끈의 길이의 최솟값을 구하여라.
    - 규칙: 트리의 각 정점은 매듭으로, 이 매듭은 하나 또는 여러 개의 노끈의 어떤 부분을 묶은 것이다. 이 부분은 끝이어도 되고 중간이어도 됨에 유의하라.
- 제약 조건
    - $$2≤N≤10^4$$

## 만점 풀이

먼저 필요한 최소 개수는 굉장히 쉽게 계산할 수 있다. 어떤 한 리프로부터 노끈을 만들기 시작하다가 교점을 만났을 때 새롭게 나오는 분기선의 개수가 1개이면 새로운 노끈이 필요하지 않으며, 만약 그보다 많으면 남은 분기선을 2개씩 묶어 새로운 노끈으로 만드는 것이 최적이기 때문이다. 이를 이용하면 임의의 한 리프로부터 DFS를 돌리며 나오는 자식 노드의 개수를 이용해 계산할 수 있다.

하지만 DFS를 사용하지 않고 차수만으로 간단하게 구할 수 있는데, 루트를 제외하면 트리에서 자식 노드의 개수는 점의 차수 - 1이므로 모든 점의 차수 $$d_i$$에 대해 $$\lfloor { {d_i - 2} \over 2 } \rfloor$$의 합에 1을 더한 값으로도 구할 수 있다.

가장 긴 노끈의 길이의 최솟값을 구하는 아이디어는 [Parkovi (24472)](https://www.acmicpc.net/problem/24472) 문제에서 사용한 아이디어와 거의 유사하다. 노끈의 길이로 parametric search를 하면, 우리는 정해진 노끈 길이 $$d$$에서 필요한 노끈의 최소 개수만 계산해내면 되며, 이는 tree에서 greedy하게 계산할 수 있다.

일단 $$x$$를 현재까지 전부 사용한 노끈의 개수, $$k$$를 노끈을 현재까지 쓴 길이라고 하자. 정의상 $$0≤k<d$$를 만족함에 유의하라. 우리는 매 DFS 과정에서 각 정점 $$V$$의 자식 노드의 $$k$$ 값들을 이용해 $$x$$의 값을 갱신할 수 있다.

먼저 자식 노드로부터 받은 $$k$$의 값을 $$k_1, k_2, \cdots, k_m$$이라고 하자. 그리고 현재 노드로 오면서 길이 1을 더 썼을 것이므로 이 값들에 모두 1을 더한다.

- $$k_i + 1 = d$$인 경우: 이 경우에는 노끈을 더 늘릴 수 없으므로 정점 $$V$$에서 노끈이 끝난다. 이제 $$i$$를 **더 이상 고려하지 않고** $$x$$를 1 증가시킨다.
- $$(k_i +1) + (k_j + 1) ≤ d$$인 $$j$$가 존재하는 경우: 이 경우에는 두 방향에서 온 노끈을 하나의 노끈으로 합치면 된다. 따라서 이 $$i$$와 $$j$$값을 **더 이상 고려하지 않고** $$x$$를 1 증가시킨다.
- $$(k_i +1) + (k_j + 1) ≤ d$$인 $$j$$가 존재하지 않는 경우: 이 경우에는 이러한 $$i$$들 중에서 $$k_i+1$$이 제일 작은 경우를 골라 계속 이어나갈 노끈으로 잡는다. 그리고 나머지 노끈들의 경우 정점 $$V$$에서 노끈을 묶어지며 끝나므로 하나당 $$x$$를 1씩 증가시킨다.

위 과정에서의 핵심은 두 번째와 같이 쌍으로 묶이는 경우가 최대한 많도록 하는 것이다. 이는 $$k_1, k_2, \cdots, k_m$$를 내림차순으로 정렬한 후 가능한 최대한 큰 값과 매칭하면 된다. 이렇게 해야 최대한 큰 값들과 매칭시키면서 남는 값들을 작게 만들 수 있다.

이렇게 계산된 $$x$$와 리턴된 $$k$$ 값을 이용해 필요한 노끈의 최소 개수를 알아낼 수 있으므로 문제를 해결할 수 있다.

이제 시간복잡도를 논의해보자. 노끈을 매칭하는 과정의 시간복잡도가 자식 노드의 수 $$m$$에 대해 $$O(m^2)$$이므로 한번 DFS에 시간복잡도는 최악의 경우 $$O(n^2)$$이다. 따라서 최악 시간복잡도는 $$O(n^2 \log n)$$임을 알 수 있다. (parametric search에서 가능한 답의 범위가 $$O(n)$$ 스케일)

$$n=10^4$$에 대해 애매한 시간복잡도지만 각 한번의 반복의 연산량이 많지 않고, 매칭된 경우 넘어가기 때문에 실제 시간은 그렇게 오래 걸리지 않는다. 실제 $$n=10^4$$일 때 최악의 경우 실행 시간은 500ms 정도이므로 문제가 되지 않으며, 실제 데이터는 약해 20ms 이내의 시간으로 AC를 받을 수 있다.

```c++
#include <bits/stdc++.h>
using namespace std;
typedef long long ll;
int n, deg[10101];
vector<int> adj[10101];

int cnt = 0, ans = 0;
int DFS(int v, int prev, int d) {
    vector<int> ret;
    for (int i = 0; i < adj[v].size(); i++) {
        int next = adj[v][i];
        if (next == prev) continue;
        ret.push_back(DFS(next, v, d) + 1);
    }
    sort(ret.begin(), ret.end());
    int ch[10101], res = 1e9;
    for (int i = 0; i < ret.size(); i++) ch[i] = 0;
    for (int i = ret.size() - 1; i >= 0; i--) {
        if (ch[i] == 1) continue;
        if (ret[i] == d) {
            ch[i] = 1;
            cnt++;
            continue;
        }
        for (int j = i - 1; j >= 0; j--) {
            if (ch[j] == 1) continue;
            if (ret[i] + ret[j] <= d) {
                ch[i] = 1;
                ch[j] = 1;
                cnt++;
                break;
            }
        }
    }
    for (int i = 0; i < ret.size(); i++) {
        if (ch[i] == 0) {
            cnt++;
            res = min(res, ret[i]);
        }
    }
    if (res == 1e9) {
        return 0;
    }
    else {
        cnt--;
        return res;
    }
}

bool solve(int d) {
    cnt = 0;
    if (DFS(1, 0, d) > 0) cnt++;
    //cout << d << " " << cnt << "\n";
    return cnt == ans;
}

int main() {
    ios_base::sync_with_stdio(false);
    cin.tie(0);

    cin >> n;
    for (int i = 1; i < n; i++) {
        int a, b;
        cin >> a >> b;
        adj[a].push_back(b);
        adj[b].push_back(a);
        deg[a]++;
        deg[b]++;
    }

    for (int i = 1; i <= n; i++) ans += (deg[i] - 1) / 2;
    ans++;
    cout << ans << "\n";

    int st = 1, ed = n - 1;
    while (st < ed) {
        int mid = (st + ed) / 2;
        if (solve(mid)) ed = mid;
        else st = mid + 1;
    }
    cout << st << "\n";

    return 0;
}
```
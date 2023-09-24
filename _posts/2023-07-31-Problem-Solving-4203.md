---
title:  "2023-07-31 Problem Solving"
mathjax: true
layout: post
---

2023-07-31 Problem Solving 일지

문제 번호: 4203(D1)


## 문제 개요

- 문제 번호: 4203번 (https://www.acmicpc.net/problem/4203)
- 문제 제목: Asteroid Rangers
- 문제 출처: ACM-ICPC World Finals 2012 A번
- 난이도: D1
- 알고리즘: `#bfs #minimum_spanning_tree`
- 풀이 시간: 105분
- 문제 요약: 총 $$N$$개의 소행성 중 몇 개의 소행성 쌍을 연결하여 모든 소행성이 연결되게 하려고 한다. 하지만, 각 소행성 쌍을 연결하기 위해서는 두 소행성 사이의 거리만큼의 비용이 든다. 즉, 우리의 목적은 최소 비용으로 모든 소행성을 연결하는 것이다. 만약 소행성이 정지해있다면 쉬운 문제겠지만, 소행성은 움직이기 때문에 현재 가장 가까운 소행성이 미래에는 그렇지 않을 수도 있다. 처음 소행성의 위치 및 이동 속도가 주어질 때, 총 몇 번 소행성 연결 상태를 세팅하거나 바꾸어야 하는지 출력하는 프로그램을 작성하라. 단, $$t=0$$에 연결 상태를 설정하는 것도 한 번으로 센다.
- 제약 조건
    - $$T$$개의 테스트케이스가 있다.
    - $$1≤N≤50$$을 만족한다.
    - 각 소행성의 위치 $$(x, y, z)$$와 속도벡터 $$(v_x, v_y, v_z)$$는 $$-150 ≤ x, y, z, 150$$과 $$-100≤v_x, v_y, v_z≤100$$을 만족한다.
    - 초기에 가능한 최적 소행성 연결 상태는 유일하고, 임의의 실수 $$t≥0$$에 대해 최적 소행성 연결 상태는 $$t<s<t+10^{-6}$$인 모든 $$s$$에서 유일하게 같다.

## 만점 풀이

먼저 처음 상태의 `minimum spanning tree`를 구하자. 이 처음 상태로부터 최적 연결 상태가 변경되려면, 적어도 간선들의 가중치 사이의 대소관계가 바뀌어야 한다. 또한, 거리 변화는 연속적이므로 현재 순서에서 인접한 두 간선의 대소관계만 변화할 수 있음이 보장된다. 즉, 인접한 두 간선의 대소관계의 변화의 시각 중 가장 작은 것을 찾아 변경하면 된다. 만약 가장 작은 것이 여러 개라면 모두 동시에 변경하여야 한다. 또한 이 시각을 구할 때 이차방정식 여부, 근의 개수 여부 등에 따라 나누어 처리해야 한다. 

만약 naive하게 구현한다면 매번 업데이트에 소요되는 시간은 $$O(N^2)$$, 최적 연결 상태를 구하는데 걸리는 시간은 $$O(N^2)$$이고, 최대 $$O(N^4)$$번 업데이트가 일어날 수 있으므로 전체 시간복잡도는 $$O(N^6 T)$$이다. 이제 각 과정을 최적화해보자.

- 순서 바뀌는 간선 찾기: 인접한 간선의 순서가 바뀌는 시각을 `std::set`에 넣어 관리하자. 그러면 $$O(\log N)$$에 순서가 바뀌는 간선을 찾을 수 있다. 또한 순서가 바뀌는 경우에도 그 주변 세 개만 처리하면 되므로 업데이트도 $$O(\log N)$$에 할 수 있다. 이 과정에서 $$0.5 \times 10^{-6}$$ 이내에 바뀌는 간선은 한번에 처리해야 함에 유의하자.
- mst 변경 처리하기: $$i$$번째 간선과 $$i+1$$번째 간선의 순서가 바뀌었을 때 mst가 바뀔 조건은 $$i-1$$번째 간선까지 했을 때 $$i$$번째 간선이 연결하는 두 컴포넌트와 $$i+1$$번째 간선이 연결하는 두 컴포넌트가 동일할 조건이다. 따라서 mst를 구하는 과정에서 하나의 간선을 추가할 때마다 $$N$$개의 점의 루트를 저장하면 쉽게 판단할 수 있다. 또한, 순서가 바뀌더라도 $$i+1$$번째 간선 이후의 각 정점의 루트는 동일하므로 $$i$$번째 간선 이후와 $$i+1$$번째 간선 이후만 처리해도 된다. 즉, mst 변경을 $$O(1)$$에 처리할 수 있다.

고로 전체 과정을 시간복잡도 $$O(N^4 T \log N)$$의 시간에 처리할 수 있다. 아래는 구현 코드이다. 644ms에 AC를 받는다.

```cpp
#include <bits/stdc++.h>
using namespace std;
int t;
int n, ans = 0;
struct threeD {
    int x, y, z;
    int vx, vy, vz;
    threeD operator- (const threeD &other) const {
        return {x - other.x, y - other.y, z - other.z, vx - other.vx, vy - other.vy, vz - other.vz};
    }
    int two() {
        return vx * vx + vy * vy + vz * vz;
    }
    int one() {
        return 2 * x * vx + 2 * y * vy + 2 * z * vz;
    }
    int zero() {
        return x * x + y * y + z * z;
    }
} pos[55];
struct edges {
    int u, v; // always u < v
    bool operator< (const edges &other) const {
        if (v == other.v) return u < other.u;
        return v < other.v;
    }
};
vector<edges> e;
double last = 0;
set< pair<double, int> > s;
double dis(threeD p, threeD q) {
    double dx = (p.x + p.vx * last - q.x - q.vx * last);
    double dy = (p.y + p.vy * last - q.y - q.vy * last);
    double dz = (p.z + p.vz * last - q.z - q.vz * last);
    return dx * dx + dy * dy + dz * dz;
}
bool compare(edges a, edges b) {
    return dis(pos[a.u], pos[a.v]) < dis(pos[b.u], pos[b.v]);
}

int mst[2020][55];
int root[55];
int _find(int v) {
    if (root[v] == v) return v;
    else return root[v] = _find(root[v]);
}
bool _union(int v, int u) {
    v = _find(v);
    u = _find(u);
    if (v == u) return false;
    if (v < u) swap(v, u);
    root[v] = u;
    return true;
}
void getmst() {
    for (int i = 1; i <= n; i++) root[i] = i;
    for (int i = 0; i < (int)e.size(); i++) {
        for (int j = 1; j <= n; j++) mst[i][j] = _find(j);
        if (_union(e[i].u, e[i].v) == false);
    }
}
void update(int idx) {
    for (int j = 1; j <= n; j++) root[j] = mst[idx][j];
    _union(e[idx].u, e[idx].v);
    idx++;
    for (int j = 1; j <= n; j++) mst[idx][j] = _find(j);
    _union(e[idx].u, e[idx].v);
    idx++;
    for (int j = 1; j <= n; j++) mst[idx][j] = _find(j);
}

void addtime(int idx) {
    if (idx <= 0 || idx >= (int)e.size()) return;
    threeD d1 = pos[e[idx - 1].u] - pos[e[idx - 1].v], d2 = pos[e[idx].u] - pos[e[idx].v];
    double a = d1.two() - d2.two();
    double b = d1.one() - d2.one();
    double c = d1.zero() - d2.zero();
    if (a != 0) {
        if (b * b - 4 * a * c == 0 && a > 0) {
            s.insert({(-b+sqrt(b * b - 4 * a * c)) / 2 / a, idx});
        }
        if (b * b - 4 * a * c > 0) {
            s.insert({(-b+sqrt(b * b - 4 * a * c)) / 2 / a, idx});
        }
    }
    else {
        if (b > 0) {
            s.insert({-c/b, idx});
        }

    }
    while (s.size() > 0 && s.begin()->first < last) s.erase(s.begin());
}
void remtime(int idx) {
    if (idx <= 0 || idx >= (int)e.size()) return;
    threeD d1 = pos[e[idx - 1].v] - pos[e[idx - 1].u], d2 = pos[e[idx].v] - pos[e[idx].u];
    double a = d1.two() - d2.two();
    double b = d1.one() - d2.one();
    double c = d1.zero() - d2.zero();
    if (a != 0) {
        if (s.find({(-b+sqrt(b * b - 4 * a * c)) / 2 / a, idx}) != s.end())
            s.erase({(-b+sqrt(b * b - 4 * a * c)) / 2 / a, idx});
    }
    else {
        if (s.find({-c/b, idx}) != s.end())
            s.erase({-c/b, idx});
    }
}

void solve(int ith) {
    if (!(cin >> n)) exit(0);
    for (int i = 1; i <= n; i++)
        cin >> pos[i].x >> pos[i].y >> pos[i].z >> pos[i].vx >> pos[i].vy >> pos[i].vz;

    ans = 1;
    last = 0;
    e.clear();
    for (int i = 1; i <= n; i++) {
        for (int j = i + 1; j <= n; j++) {
            e.push_back({i, j});
        }
    }
    sort(e.begin(), e.end(), compare);
    for (int i = 1; i < (int)e.size(); i++) addtime(i);
    getmst();

    while (true) {
        last = s.begin()->first;
        int f = 0;
        while (s.size() > 0 && s.begin()->first <= last + 0.5e-6) {
            int idx = s.begin()->second;
            remtime(idx - 1);
            remtime(idx);
            remtime(idx + 1);
            swap(e[idx], e[idx - 1]);
            addtime(idx - 1);
            addtime(idx);
            addtime(idx + 1);
            int a = mst[idx - 1][e[idx].u], b = mst[idx - 1][e[idx].v];
            int c = mst[idx - 1][e[idx - 1].u], d = mst[idx - 1][e[idx - 1].v];
            if (a != b && c != d) {
                if (a == c && b == d) f = 1;
                if (a == d && b == c) f = 1;
            }
            update(idx - 1);
        }
        if (f) ans++;
        if (s.size() == 0) break;
    }

    cout << "Case " << ith << ": " << ans << "\n";
}

int main() {
    int t = 1;
    while (true) {
        solve(t);
        t++;
    }
}
```
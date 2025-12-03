# alg3
algorithm
#include <bits/stdc++.h>
using namespace std;

const int MAXN = 200005;
vector<pair<int,int>> adj[MAXN];

int parent[MAXN], depth[MAXN], heavy[MAXN];
int head[MAXN], pos[MAXN], sz[MAXN], baseArray[MAXN];
int curPos = 0;

struct SegTree {
    int n;
    vector<int> tree;
    SegTree(int n) : n(n) {
        tree.resize(4 * n);
    }
    void build(int node, int l, int r) {
        if (l == r) {
            tree[node] = baseArray[l];
            return;
        }
        int mid = (l + r) / 2;
        build(node*2, l, mid);
        build(node*2+1, mid+1, r);
        tree[node] = max(tree[node*2], tree[node*2+1]);
    }
    void update(int node, int l, int r, int idx, int val) {
        if (l == r) {
            tree[node] = val;
            return;
        }
        int mid = (l+r)/2;
        if (idx <= mid) update(node*2, l, mid, idx, val);
        else update(node*2+1, mid+1, r, idx, val);
        tree[node] = max(tree[node*2], tree[node*2+1]);
    }
    int query(int node, int l, int r, int ql, int qr) {
        if (ql > r || qr < l) return -1e9;
        if (ql <= l && r <= qr) return tree[node];
        int mid = (l+r)/2;
        return max(query(node*2, l, mid, ql, qr),
                   query(node*2+1, mid+1, r, ql, qr));
    }
};

int dfs(int u) {
    sz[u] = 1;
    int maxSubtree = 0;
    for (auto &[v, w] : adj[u]) {
        if (v != parent[u]) {
            parent[v] = u;
            depth[v] = depth[u] + 1;
            int subtreeSize = dfs(v);
            sz[u] += subtreeSize;
            if (subtreeSize > maxSubtree) {
                maxSubtree = subtreeSize;
                heavy[u] = v;
            }
        }
    }
    return sz[u];
}

void decompose(int u, int h, int edgeWeight) {
    head[u] = h;
    pos[u] = ++curPos;
    baseArray[curPos] = edgeWeight;

    if (heavy[u] != -1) {
        // heavy edge weight stored in baseArray
        for (auto &[v, w] : adj[u]) {
            if (v == heavy[u]) {
                decompose(v, h, w);
                break;
            }
        }
    }
    for (auto &[v, w] : adj[u]) {
        if (v != parent[u] && v != heavy[u]) {
            decompose(v, v, w);
        }
    }
}

int queryPath(SegTree &st, int u, int v) {
    int res = -1e9;
    while (head[u] != head[v]) {
        if (depth[head[u]] < depth[head[v]]) swap(u, v);
        res = max(res, st.query(1,1,curPos,pos[head[u]], pos[u]));
        u = parent[head[u]];
    }
    if (u == v) return res;
    if (depth[u] > depth[v]) swap(u, v);
    // now u is LCA, so skip its position
    res = max(res, st.query(1,1,curPos, pos[u]+1, pos[v]));
    return res;
}

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int n;
    cin >> n;

    for (int i=1; i<=n; i++) heavy[i] = -1;

    for (int i=0; i<n-1; i++) {
        int u, v, w;
        cin >> u >> v >> w;
        adj[u].push_back({v, w});
        adj[v].push_back({u, w});
    }

    parent[1] = -1;
    depth[1] = 0;
    dfs(1);
    decompose(1, 1, -1);

    SegTree st(curPos);
    st.build(1, 1, curPos);

    int q;
    cin >> q;
    while (q--) {
        string type;
        cin >> type;
        if (type == "QUERY") {
            int u, v;
            cin >> u >> v;
            cout << queryPath(st, u, v) << "\n";
        } else {
            int u, v, w;
            cin >> u >> v >> w;
            if (parent[u] == v) st.update(1, 1, curPos, pos[u], w);
            else st.update(1, 1, curPos, pos[v], w);
        }
    }
    return 0;
}

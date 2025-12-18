https://codeforces.com/blog/entry/44351
```cpp showLineNumbers title="getsz"
auto getsz = [&](auto self, int u, int fa) -> void {
	siz[u] = 1;  // every vertex has itself in its subtree
	for (auto v : adj[u])
		if (v != fa) {
			self(self, v, u);
			siz[u] += siz[v];  // add size of child u to its parent(v)
		}
};
```
2. easy to code and $\mathcal O(N\log N)$
```cpp showLineNumbers title="getsz"
vector<unique_ptr<vector<int>>> vec(n + 1);
vector<int>                     cnt(n + 1);
auto dfs = [&](auto&& self, int u, int fa, bool keep) -> void {
    int mx = -1, bigChild = -1;
    for (auto v : adj[u])
        if (v != fa && siz[v] > mx) mx = siz[v], bigChild = v;
    for (auto v : adj[u])
        if (v != fa && v != bigChild) self(self, v, u, 0);
    if (bigChild != -1)
        self(self, bigChild, u, 1), vec[u] = std::move(vec[bigChild]);
    else
        vec[u] = make_unique<vector<int>>();
    vec[u]->push_back(u);
    cnt[col[u]]++;
    for (auto v : adj[u])
        if (v != fa && v != bigChild)
            for (auto x : *vec[v]) {
                cnt[col[x]]++;
                vec[u]->push_back(x);
            }
    // now cnt[c] is the number of vertices in subtree of vertex v that has color c.
    //  note that in this step *vec[v] contains all of the subtree of vertex v.
    if (keep == 0)
        for (auto x : *vec[u]) {
            cnt[col[x]]--;
        }
};
```
3. heavy-light decomposition style $\mathcal O(N\log N)$
```cpp
int  cnt[maxn];
bool big[maxn];
auto add = [&](int u, int fa, int x) {
	cnt[col[u]] += x;
	for (auto v : adj[u])
		if (v != fa && !big[v]) add(v, u, x);
};
auto dfs = [&](auto&& self, int u, int fa, bool keep) {
	int mx = -1, bigChild = -1;
	for (auto v : adj[u])
		if (v != fa && siz[v] > mx) mx = siz[v], bigChild = v;
	for (auto v : adj[u])
		if (v != fa && v != bigChild) self(self, v, u, 0);  // run a dfs on small childs and clear them from cnt
	if (bigChild != -1)
		self(self, bigChild, u, 1), big[bigChild] = 1;  // bigChild marked as big and not cleared from cnt
	add(u, fa, 1);
	// now cnt[c] is the number of vertices in subtree of vertex v that has color c. You can answer the queries
	// easily.
	if (bigChild != -1) big[bigChild] = 0;
	if (keep == 0) add(u, fa, -1);
};
```
3. 
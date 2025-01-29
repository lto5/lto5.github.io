# Offline Incremental SCC

Dynamic Connectivity chắc hẳn là một lớp bài toán khá quen thuộc với dân CP, và được áp dụng rộng rãi trong nhiều bài toán khác nhau. Tuy nhiên, các lớp bài toán này chỉ khai thác trên đồ thị vô hướng, còn về vấn đề liên thông động trên đồ thị có hướng thì chưa được quan tâm nhiều. Bài viết này sẽ giới thiệu về một thuật toán để quản lý các thành phần liên thông mạnh trên đồ thị có hướng, và một vài áp dụng cơ bản của nó.

## Bài toán

Cho đồ thị $G$ gồm $n$ đỉnh và $m$ cạnh vô hướng được thêm vào $G$ lần lượt theo thứ tự đọc vào. Sau mỗi lần thêm cạnh, ta cần trả lời một số thông tin về các thành phần liên thông mạnh trong $G$. Ví dụ:

- tìm thời điểm đầu tiên mà $u$ và $v$ cùng SCC.
- tìm kích thước SCC chứa $u$ sau khi $i$ cạnh được thêm vào.

**Lưu ý**: ý tưởng chỉ có thể áp dụng được khi ta biết trước tất cả các truy vấn.

## Nhận xét

Gọi $S(u,i)$ là tập các đỉnh cùng SCC với $u$ khi đã thêm $i$ cạnh đầu tiên vào $G$.
	
Thuật toán được đưa ra dựa vào nhận xét khá dễ thấy:
> Với mọi cặp $(u,v)$ ta có $S(u,j) \neq S(v,j) \Rightarrow S(u,k) \neq S(v,k) \ \forall k \leq j$, nghĩa là nếu tại thời điểm $j$ mà $u$ và $v$ chưa cùng thuộc một SCC, thì tại các thời điểm $k \le j$, chúng cũng không thể thuộc cùng một SCC.

Như vậy, với mỗi cạnh $E_i = (u,v)$, ta gọi $t_i = \arg \min_j \left(S(u,j)=S(v,j) \right)$ (và bằng vô cùng nếu như không bao giờ $S(u)=S(v)$). Dễ thấy rằng sự xuất hiện của $E_i$ trong $G$ không ảnh hưởng đến bất kì một SCC nào cho đến khi $t_i$ cạnh đầu tiên được thêm vào.

Bên cạnh đó, khi ta tính được $t[.]$ cho tất cả các cạnh, thì việc xử lí các truy vấn tại thời điểm thứ $i$ cũng dễ dàng hơn rất nhiều, do ta chỉ cần xem các cung trong cùng một SCC là một **cạnh vô hướng**, từ đó có thể xử lí bằng các kĩ thuật trên đồ thị vô hướng (ví dụ như DSU).

Do đó, vấn đề của ta là đi tính tất cả các $t_i$. Tuy nhiên, nếu ta tính bình thường thì độ phức tạp là $O(n^2)$, dĩ nhiên không qua được nếu giới hạn lớn. Tuy nhiên, ta có thể áp dụng một thuật toán tương tự như chặt nhị phân song song để tính toàn bộ mảng $t[]$ trong $O(n \log n)$, và sẽ được trình bày ngay sau đây.

## Thuật toán

### Thuật toán chung

Ta sẽ thực hiện chia để trị trên đoạn thời gian các cạnh được thêm vào bằng cách cài đặt một hàm đệ quy $\text{rec}(l,r,\text{edge list})$ với $[l,r]$ là khoảng thời gian hiện tại chúng ta đang xét và $\text{edge list}$ là danh sách cạnh có thể làm ảnh hưởng cấu trúc SCC tại các thời điểm $x$ mà $l \leq x \leq r$, nghĩa là $l \leq t_i \leq r$.

- Đặt $\text{mid} = \lfloor \frac{l+r}{2} \rfloor$
- Ta tìm tất cả các SCC được hình thành từ các cạnh trong $\text{edge list}$ và có thời gian xuất hiện $\leq \text{mid}$ vào đồ thị được hình thành bởi các cạnh này
- Với mỗi cạnh $(u,v)$ có thời gian xuất hiện $\leq \text{mid}$, nếu $u$ và $v$ chung một SCC thì ta có thể kết luận $t_i \leq \text{mid}$, ngược lại $t_i >\text{mid}$. Chia các cạnh này thành hai nhóm tương ứng là $G_1$ và $G_2$.
- Lưu lại các thông tin cần sử dụng
- Gọi $\text{rec}(l,mid-1,G_1)$
- Từ nhận xét, ta biết chắc chắn các cạnh nối 2 đỉnh chung SCC (nghĩa là các cạnh trong $G_1$) sau thời điểm này sẽ vẫn nối 2 đỉnh chung SCC, do đó ta có thể nén toàn bộ các SCC được hình thành qua việc thêm các cạnh trong $G_1$
- Gọi $\text{rec}(\text{mid}+1,r,G_2)$. **Lưu ý**: việc gọi hàm này phải được thực hiện trên đồ thị mới được tạo ra sau khi đã nén các SCC được tạo ra bởi việc thêm các cạnh trong $G_1$

Lưu ý rằng khi ta thực hiện hàm $\text{rec}(l,r,\text{edge list})$, các thao tác tìm SCC hay kiểm tra hai đỉnh thuộc cùng một SCC **phải được thực hiện trên đồ thị nén khi đã nối các cạnh có $t_i<l$**, khi đó ta mới có đủ dữ kiện để tiếp tục nối các cạnh trong $\text{edge list}$ có thời gian xuất hiện $\leq \text{mid}$. Việc này có thể được thực hiện một cách dễ dàng bằng cách duy trì một đồ thị toàn cục $G$ và tiến hành đệ quy theo đúng thứ tự nêu trên.

Ở mỗi tầng đệ quy, ta đảm bảo mỗi cạnh chỉ được xét đến duy nhất một lần. Khi DFS, ta chỉ xét các đỉnh xuất hiện trong ít nhất một cạnh. Do đó, tổng độ phức tạp của việc chia để trị vẫn là $O(n \log n)$.

Mã giả của thuật toán:

```
G = đồ thị rỗng
def f(l, r, edge list):
	if l > r:
		return
	mid = (l + r) >> 1
	reset cấu trúc SCC
	// đảm bảo các cạnh có chỉ số < l đã được nối
	// vậy ta chỉ cần quan tâm các cạnh trong tập edge list và có chỉ số <= mid
	for [u, v] in edge list:
		if i <= mid:
			u = SCC chứa u trong G
			v = SCC chứa v trong G
			thêm cạnh u->v vào cấu trúc SCC
	tìm các SCC
	G1, G2 = danh sách cạnh rỗng
	for [u, v] in edge list:
		u = SCC chứa u trong G
		v = SCC chứa v trong G
		if i <= mid:
			if scc[u] == scc[v]:
				thêm u->v vào G1 // u->v có thể ảnh hưởng đến cấu trúc tại thời điểm trước đây
			else:
				thêm u->v vào G2 // u->v chưa ảnh hưởng gì
		else:
			thêm u->v vào G2 // u->v chưa xuất hiện nên rõ ràng không thay đổi gì cho cấu trúc
	f(l, mid-1, G1)
	// lúc này, trong G đã có đầy đủ thông tin tạo bởi các cạnh [1, mid - 1]
	// như vậy, ta chỉ cần nối thêm các cạnh trong G1 nữa, là đã duy trì đủ thông tin là các SCC tạo bởi các cạnh [1, mid]
	for các SCC tạo thành bởi G1:
		nén các đỉnh trong SCC này thành một đỉnh trong G
	sử dụng G để trả lời câu hỏi liên quan đến mid
	f(mid+1, r, G2)
		
f(1, m, edge list)
```

Độ phức tạp: mỗi lần ta chia $G$ thành hai list $G_1$ và $G_2$ không trùng nhau, vậy với mỗi tầng đệ quy ta chỉ xét mỗi cạnh duy nhất $1$ lần. Vậy độ phức tạp nhìn chung là $O(m \log m)$. Độ phức tạp cho việc trả lời truy vấn là khác nhau với từng bài toán, vậy nên nó sẽ không được đánh giá ở đây.

### Áp dụng

Do kĩ thuật này còn khá mới và chưa được áp dụng nhiều, nên cũng chưa có nhiều bài toán về vấn đề này. Hiện tại các bài toán liên quan đều thuộc một trong hai dạng chính sau:

#### Truy vấn trên cạnh

> Các truy vấn có dạng: tìm thông tin về các SCC sau khi thêm $i$ cạnh.

Ta có thể duy trì một số cấu trúc dữ liệu như DSU để trả lời các truy vấn, do khi merge các SCC lại thành một siêu đỉnh, thì ta có thể coi như các cung trong SCC đó là một cạnh vô hướng mà không ảnh hưởng đến bài toán. Nghĩa là, xét thời điểm $i$ và các cung $(u, v)$ có $t[.] = i$, ta sẽ xem cung $u \rightarrow v$ là một cạnh vô hướng $(u, v)$, và cạnh này sẽ được thêm vào đồ thị vô hướng mới tại thời điểm $i$. Như vậy, tại mỗi thời điểm, ta chỉ cần duy trì một CTDL nhất định (thường là DSU) để quản lý các TPLT trên đồ thị vô hướng.
	
#### Truy vấn trên đỉnh

> Các truy vấn có dạng: tìm $i$ nhỏ nhất sao cho $u$ và $v$ chung SCC sau khi đã thêm $i$ cạnh đầu tiên vào $G$.

Dựng đồ thị vô hướng $G'$ mới với:

- Tập đỉnh là tập đỉnh của đồ thị gốc.
- Tập cạnh là các cạnh $(u, v, t)$ với $u \rightarrow v$ là một cung trên đồ thị gốc, và $t$ là thời điểm đầu tiên mà $u$ và $v$ chung một SCC.

Như vậy, đáp án cho mỗi truy vấn $(s, t)$ là cạnh lớn nhất trên cây khung nhỏ nhất của $G'$ (dễ thấy dựa vào tính chất của cây khung nhỏ nhất).

## Ví dụ

### [QOJ 2214 - Link Cut Digraph](https://qoj.ac/contest/796/problem/2214)

#### Đề bài
Cho một đồ thị có hướng $n$ đỉnh, ban đầu không có cạnh nào. Bạn được cho $m$ truy vấn, mỗi truy vấn yêu cầu bạn thêm cạnh có hướng $u \rightarrow v$, sau đó in ra số cặp $(u,v)$ chung một thành phần liên thông mạnh.

#### Lời giải

Đây là bài toán được nêu ra ở phần Áp dụng. Trước hết, ta sử dụng thuật toán nêu trên để tính mảng $t[.]$ cho tất cả các cung. Sau đó, duyệt các thời điểm $i$ từ $1 \rightarrow m$, tại mỗi thời điểm, ta nối các cạnh $(u,v)$ có $t[.] = i$ trong DSU. Để trả lời truy vấn một cách nhanh chóng, trong DSU, ta lưu thêm một biến thể hiện số cặp $(u,v)$ trong cùng một TPLT; và cập nhật biến này mỗi lần nối cạnh. Do mỗi lần nối cạnh, chỉ có tối đa 2 TPLT bị thay đổi kích thước, nên ta dễ dàng cập nhật đáp án trong $O(1)$. Vậy tổng độ phức tạp là $O((n+m) \log m)$.

#### Code mẫu

```cpp
#include <bits/stdc++.h>

using namespace std;

/* DSU lưu thông tin về các TPLT. */
struct DSU {
  vector<int> par;
  /* Số cặp (u, v) với u != v và u, v chung một TPLT đến thời điểm hiện tại. */
  int64_t cur_ans;

  DSU(int n = 0) {
    par.assign(n + 5, -1);
    cur_ans = 0;
  }

  int root(int u) {
    if (par[u] < 0)
      return u;
    return par[u] = root(par[u]);
  }

  int size(int u) {
    return -par[root(u)];
  }

  bool join(int u, int v) {
    u = root(u);
    v = root(v);
    if (u == v)
      return false;
    /* Thêm vào đáp án các cặp (s, t) mà s thuộc TPLT gốc u, và t thuộc TPLT gốc v, do trước đó, chúng chưa đi được đến nhau, và sau thời điểm này, chúng sẽ đi được đến nhau. */
    cur_ans += 1ll * -par[u] * -par[v];
    if (par[u] > par[v])
      swap(u, v);
    par[u] += par[v];
    par[v] = u;
    return true;
  }
};

const int N = 2.5e5 + 5;

int n, m;
pair<int, int> ed[N];

DSU dsu(N);
int64_t ans[N];
vector<int> nodes;
vector<int> g[N];
int low[N], num[N], time_dfs, del[N];
int id_scc[N];
vector<int> stk;
vector<vector<int>> scc;
vector<int> T[N];

/* Thuật toán Tarjan tìm các TPLTM trong đồ thị hiện tại. */
void dfs(int u) {
  low[u] = num[u] = ++time_dfs;
  stk.push_back(u);
  for (int v : g[u]) {
    if (del[v]) {
      continue;
    }
    if (!num[v]) {
      dfs(v);
      low[u] = min(low[u], low[v]);
    } else {
      low[u] = min(low[u], num[v]);
    }
  }
  if (low[u] == num[u]) {
    scc.push_back({});
    int v;
    do {
      v = stk.back();
      scc.back().push_back(v);
      id_scc[v] = scc.size();
      del[v] = 1;
      stk.pop_back();
    } while (v != u);
  }
}

/* Thêm cung u -> v vào đồ thị tạm thời, để sau này tiến hành tìm SCC */
void add_edge(int u, int v) {
  /* Chỉ xét các đỉnh thuộc ít nhất một cạnh trong truy vấn hiện tại. Do ta cần DFS nlog lần, nhưng tổng kích thước các tập đỉnh ta xét mỗi lần DFS cũng không vượt quá nlog, nên việc duyệt từ 1 -> n để DFS sẽ có thể bị TLE, nhưng việc lưu lại như thế này lại không bị TLE. */
  nodes.push_back(u);
  nodes.push_back(v);
  g[u].push_back(v);
}

/* Reset dữ liệu cho các lần DFS sau. */
void reset() {
  for (int u : nodes) {
    low[u] = num[u] = del[u] = 0;
    id_scc[u] = 0;
    g[u].clear();
  }
  nodes.clear();
  stk.clear();
  scc.clear();
  time_dfs = 0;
}

/* Code để tính mảng t[.] */
void solve(int l, int r, vector<int> edges) {
  if (l > r) {
    return;
  }
  int mid = (l + r) >> 1;
  /* Xuất phát từ cơ sở là các đỉnh thể hiện các SCC được tạo bởi các cạnh có chỉ số < l, ta xét các cạnh có chỉ số trong đoạn [l, mid] và xem chúng có tạo ra các SCC mới không */
  reset();
  for (int i : edges) {
    if (i <= mid) {
      auto [x, y] = ed[i];
      x = dsu.root(x);
      y = dsu.root(y);
      add_edge(x, y);
    }
  }
  for (int u : nodes) {
    if (!num[u]) {
      dfs(u);
    }
  }
  /* Đây là phần tương tự chặt nhị phân song song. Nếu u và v cùng một SCC, chắc chắn t[.] <= mid, như vậy ta cần phải giảm cận dưới xuống cho chúng. Ngược lại, ta cần phải tăng cận trên lên. */
  vector<int> g1, g2;
  for (int i : edges) {
    if (i <= mid) {
      auto [u, v] = ed[i];
      u = dsu.root(u), v = dsu.root(v);
      if (id_scc[u] == id_scc[v]) {
        g1.push_back(i);
      } else {
        g2.push_back(i);
      }
    } else {
      g2.push_back(i);
    }
  }
  /* Xét các cạnh trong tập trái */
  solve(l, mid - 1, g1);
  /* Đến đây, ta được phép merge các đỉnh thuộc cùng một SCC thành một đỉnh to vào với nhau, do ta đã xét hết toàn bộ các giá trị <= mid */
  for (int i : g1) {
    dsu.join(ed[i].first, ed[i].second);
  }
  ans[mid] = dsu.cur_ans;
  /* Sau khi merge, ta tiếp tục xét các cạnh ở tập bên phải */
  solve(mid + 1, r, g2);
  edges.clear();
  g1.clear();
  g2.clear();
}

int32_t main() {
  ios_base::sync_with_stdio(0);
  cin.tie(0);
  cin >> n >> m;
  for (int i = 1; i <= m; i++) {
    auto& [u, v] = ed[i];
    cin >> u >> v;
  }
  vector<int> id(m);
  iota(id.begin(), id.end(), 1);
  solve(1, m, id);
  for (int i = 1; i <= m; i++) {
    cout << ans[i] << "\n";
  }
  return 0;
}
```

### [ONTAK 2009 - Godzilla](https://www.acmicpc.net/problem/8496)

#### Đề bài

Cho đồ thị có hướng $n$ đỉnh $m$ cạnh. Bạn cần xử lý $q$ truy vấn, mỗi truy vấn yêu cầu xoá một cạnh trong đồ thị hiện tại. Sau đó, bạn cần in ra số lượng thành phần liên thông mạnh có bậc vào bằng $0$.

#### Lời giải

Thay vì xử lí các truy vấn xoá cạnh, ta đảo ngược lại các truy vấn, và xem như ta có các truy vấn thêm cạnh. Để cho tiện, với những cạnh mà không bị truy vấn xoá nào tác động, ta cũng coi như là bị xoá sau thời điểm thứ $q$, và ta sẽ thêm từ từ các cạnh đó lại từ đầu.

Như vậy, bài toán quy về việc thêm cạnh và đếm số SCC có bậc vào bằng $0$. Trước hết, ta tính mảng $t[]$ theo định nghĩa ở trên. Sau đó, ta xét các cạnh từ $1 \rightarrow m$ theo thứ tự thêm vào:

- Giả sử cung đang xét là $(u,v)$. Đầu tiên, ta lưu lại đáp án cho đến thời điểm hiện tại, sau đó ta xử lí truy vấn thêm cung này.
- Trước hết, ta cần thêm cung $u\rightarrow v$ vào cấu trúc dữ liệu.
    - Với mỗi TPLT trong DSU (ở đây được xem như TPLTM), ta lưu thêm $\text{deg}$ là số SCC đi vào trong nó.
    - Khi thêm cung, ta tăng $\text{deg}[\text{root}(v)]$ lên $1$, với $\text{root}(v)$ là chỉ số của TPLT chứa $v$.
- Tuy nhiên, tại thời điểm hiện tại (ta tạm gọi là thời điểm $i$), cần phải xoá đi những cung $(u', v')$ có $t[.] = i$, do sau thời điểm này, chúng sẽ là một cạnh trong một SCC nào đó, và sẽ không đóng góp gì vào đáp án.
    - Xét các cung $(u',v')$ có $t[.]=i$.
    - Trước hết, ta cần trừ đi đóng góp của các cung này vào đáp án, rất đơn giản ta chỉ cần trừ $\text{deg}[\text{root}(v')]$ đi $1$.
    - Sau thời điểm này, cung $(u',v')$ sẽ là một cạnh trong một SCC nào đó, do đó ta coi như cạnh này là một cạnh vô hướng. Ta tiến hành nối cạnh $(u',v')$ trong DSU. Khi nối cạnh trong DSU, lưu ý cập nhật $\text{deg}$ một cách thích hợp.
- Lưu ý, cùng với việc duy trì mảng $\text{deg}$, ta cũng cần duy trì một biến đáp số, là số TPLT có $\text{deg}[.] = 0$ trong DSU. Với mỗi lần thay đổi, ta cần tăng giảm biến này một cách thích hợp.
- Độ phức tạp: $O((n+m) \log m)$

#### Code mẫu

```cpp
#include <bits/stdc++.h>

using namespace std;

const int N = 1e5 + 5;

/* Cấu trúc dữ liệu DSU, duy trì thông tin về các SCC */
struct DSU {
  vector<int> par;
  /* Số SCC có bán bậc vào bằng 0 */
  int cnt_zero_deg_in;
  /* Bán bậc vào của một SCC */
  vector<int> deg;

  DSU(int n = 0) {
    par.assign(n + 5, -1);
    deg.assign(n + 5, 0);
    /* Ban đầu, mỗi đỉnh con là một SCC */
    cnt_zero_deg_in = n;
  }

  int root(int u) {
    if (par[u] < 0)
      return u;
    return par[u] = root(par[u]);
  }

  int size(int u) {
    return -par[root(u)];
  }

  void add_deg(int v, int delta) {
    cnt_zero_deg_in -= !deg[v];
    deg[v] += delta;
    cnt_zero_deg_in += !deg[v];
  }

  /* Thêm cung u -> v vào đồ thị */
  void add_diedge(int u, int v) {
    add_deg(root(v), 1);
  }

  /* Xoá đóng góp của cung u -> v */
  void rem_diedge(int u, int v) {
    add_deg(root(v), -1);
  }

  bool join(int u, int v) {
    u = root(u);
    v = root(v);
    if (u == v)
      return false;
    if (par[u] > par[v])
      swap(u, v);
    /* Khi gộp 2 SCC, những cung đi vào SCC chứa v cũng là những cung đi vào SCC chứa u. Do đó, ta được phép gán deg[u] += deg[v], và huỷ đi đóng góp của đỉnh v. */
    add_deg(u, deg[v]);
    cnt_zero_deg_in -= !deg[v];
    par[u] += par[v];
    par[v] = u;
    return true;
  }

  bool fake_join(int u, int v) {
    u = root(u);
    v = root(v);
    if (u == v)
      return false;
    if (par[u] > par[v])
      swap(u, v);
    par[u] += par[v];
    par[v] = u;
    return true;
  }
};

struct Edge {
  int u, v, id;
  Edge(int u = 0, int v = 0, int id = 0) : u(u), v(v), id(id) {}
};

Edge edges[N];
Edge queries[N];
int ans[N];
bool deleted[N];
int min_time_same[N];
vector<int> edges_join_in_time[N];
int n, m, q;

DSU dsu(N);
vector<int> nodes;
vector<int> g[N];
int low[N], num[N], time_dfs, del[N];
int id_scc[N];
vector<int> stk;
vector<vector<int>> scc;
vector<int> T[N];

/* Thuật toán Tarjan để tìm các SCC */
void dfs(int u) {
  low[u] = num[u] = ++time_dfs;
  stk.push_back(u);
  for (int v : g[u]) {
    if (del[v]) {
      continue;
    }
    if (!num[v]) {
      dfs(v);
      low[u] = min(low[u], low[v]);
    } else {
      low[u] = min(low[u], num[v]);
    }
  }
  if (low[u] == num[u]) {
    scc.push_back({});
    int v;
    do {
      v = stk.back();
      scc.back().push_back(v);
      id_scc[v] = scc.size();
      del[v] = 1;
      stk.pop_back();
    } while (v != u);
  }
}

void add_edge(int u, int v) {
  nodes.push_back(u);
  nodes.push_back(v);
  g[u].push_back(v);
}

void reset() {
  for (int u : nodes) {
    low[u] = num[u] = del[u] = 0;
    id_scc[u] = 0;
    g[u].clear();
  }
  nodes.clear();
  stk.clear();
  scc.clear();
  time_dfs = 0;
}

/* Cài đặt để tìm mảng t[.] */
void solve(int l, int r, vector<int> indices) {
  if (l > r) {
    return;
  }
  int mid = (l + r) >> 1;
  reset();
  for (int i : indices) {
    if (i <= mid) {
      auto [s, t, _] = queries[i];
      s = dsu.root(s);
      t = dsu.root(t);
      add_edge(s, t);
    }
  }
  for (int u : nodes) {
    if (!num[u]) {
      dfs(u);
    }
  }
  vector<int> g1, g2;
  for (int i : indices) {
    if (i <= mid) {
      auto [u, v, _i] = queries[i];
      u = dsu.root(u), v = dsu.root(v);
      if (id_scc[u] == id_scc[v]) {
        min_time_same[i] = min(min_time_same[i], mid);
        g1.push_back(i);
      } else {
        g2.push_back(i);
      }
    } else {
      g2.push_back(i);
    }
  }
  solve(l, mid - 1, g1);
  for (int i : g1) {
    auto [u, v, _i] = queries[i];
    dsu.fake_join(u, v);
  }
  solve(mid + 1, r, g2);
  indices.clear();
  g1.clear();
  g2.clear();
}

int32_t main() {
  ios_base::sync_with_stdio(0);
  cin.tie(0);
#ifdef LOCAL
#define task "a"
#else
#define task ""
#endif
  if (fopen(task ".inp", "r")) {
    freopen(task ".inp", "r", stdin);
    freopen(task ".out", "w", stdout);
  }
  cin >> n >> m;
  for (int i = 1; i <= m; i++) {
    auto& [u, v, _i] = edges[i];
    cin >> u >> v;
    min_time_same[i] = 1e9;
    _i = i;
  }
  cin >> q;
  for (int i = 1; i <= q; i++) {
    int id;
    cin >> id;
    deleted[id] = true;
    queries[i] = edges[id];
    queries[i].id = i;
  }
  /* Coi như tất cả các cạnh đã bị xoá */
  int num_query = q;
  for (int i = 1; i <= m; i++) {
    if (!deleted[i]) {
      queries[++q] = edges[i];
      queries[q].id = q;
    }
  }
  reverse(queries + 1, queries + q + 1);
  vector<int> idx(q);
  iota(idx.begin(), idx.end(), 1);
  solve(1, q, idx);
  DSU dsu(n);
  for (int i = 1; i <= q; i++) {
    if (min_time_same[i] != 1e9) {
      edges_join_in_time[min_time_same[i]].push_back(i);
    }
  }
  /* Xét các truy vấn theo thứ tự ngược lại, nghĩa là thứ tự thêm cạnh vào thay vì xoá cạnh đi. */
  for (int i = 1; i <= q; i++) {
    auto [s, t, idx_query] = queries[i];
    /* Lưu lại đáp án cho truy vấn hiện tại */
    ans[idx_query] = dsu.cnt_zero_deg_in;
    /* Thêm cung s -> t */
    dsu.add_diedge(s, t);
    /* Xoá đóng góp của các cung u -> v chung SCC */
    for (auto idx_edge : edges_join_in_time[i]) {
      auto [u, v, _i] = queries[idx_edge];
      dsu.rem_diedge(u, v);
    }
    /* Sau đó, nén SCC lại thành một đỉnh duy nhất */
    for (auto idx_edge : edges_join_in_time[i]) {
      auto [u, v, _i] = queries[idx_edge];
      dsu.join(u, v);
    }
  }
  for (int i = 1; i <= num_query; i++) {
    cout << ans[i] << "\n";
  }
  return 0;
}
```

### [Codeforces - 1989F](https://codeforces.com/problemset/problem/1989/F)

#### Đề bài

Bạn được cho một ma trận kích thước $n \cdot m$. Bạn được thực hiện hai loại thao tác: tô một hàng màu đỏ, và tô một cột màu xanh. Trong một giây, bạn có thể thực hiện $k$ thao tác một lúc, và sẽ mất chi phí $k^2$ nếu $k>1$. Khi thực hiện một chuỗi thao tác, với mỗi ô bị ảnh hưởng bởi cả thao tác tô hàng và cột, ta có thể chọn một màu bất kì cho nó.

Bạn cần xử lí $q$ truy vấn. Ban đầu, các ô chưa được tô màu, và không có ràng buộc nào đối với màu của các ô. Tại thời điểm $i$, bạn có thêm một truy vấn dạng $x_i \ y_i \ c_i$ - yêu cầu ô $(x_i,y_i)$ phải được tô màu $c_i$. Sau mỗi truy vấn, bạn cần in ra chi phí ít nhất để tô màu bảng mà thoả mãn $i$ yêu cầu đầu tiên.

#### Lời giải

Xét một ràng buộc $x \ y \ c$. Nếu $c = R$, ta nên tô cột $y$ trước khi tô hàng $x$, nếu như ta thực hiện hai thao tác này ở hai chuỗi khác nhau. Tương tự, nếu $c = B$, ta nên tô hàng $x$ trước khi tô cột $y$.

Như vậy, ta nghĩ đến việc dựng một đồ thị gồm $n+m$ đỉnh, với một đỉnh đại diện cho một hàng hoặc cột của ma trận. Xét một ràng buộc $x \ y \ c$, ta sẽ:

- Thêm cung nối từ $x \rightarrow y+n$ nếu $c=B$, thể hiện ta phải tô hàng $x$ trước khi tô cột $y$.
- Thêm cung nối từ $y + n \rightarrow x$ nếu $c=R$, thể hiện ta phải tô cột $y$ trước khi tô hàng $x$.

Nếu đồ thị không có SCC nào kích thước lớn hơn $1$, thì đáp án là $0$, do tồn tại một thứ tự topo cho việc tô màu các hàng và cột, nên ta có thể thực hiện từng thao tác đơn lẻ. Ngược lại, nghĩa là ta luôn phải thực hiện một chuỗi thao tác đối với từng SCC, và thực hiện các chuỗi này theo thứ tự topo trên đồ thị sau khi đã nén các SCC thành một đỉnh, vậy đáp án sẽ là $\sum$ (kích thước các SCC)$^2$.

Đến đây, bài toán trở về giống bài Link Cut Digraph như đã giới thiệu ở trên, bạn đọc chỉ cần chỉnh sửa một chút cho phù hợp là được.

#### Code mẫu

```cpp
#include <bits/stdc++.h>

using namespace std;

const int N = 4e5 + 5;

struct DSU {
  vector<int> par;
  int64_t sum = 0;

  int64_t val(int x) {
    return x > 1 ? 1ll * x * x : 0;
  }

  DSU(int n = 0) {
    par.assign(n + 5, -1);
    sum = 0;
  }

  int root(int u) {
    if (par[u] < 0)
      return u;
    return par[u] = root(par[u]);
  }

  int size(int u) {
    return -par[root(u)];
  }

  bool join(int u, int v) {
    u = root(u);
    v = root(v);
    if (u == v)
      return false;
    if (par[u] > par[v])
      swap(u, v);
    sum -= val(-par[u]);
    sum -= val(-par[v]);
    par[u] += par[v];
    par[v] = u;
    sum += val(-par[u]);
    return true;
  }
};

struct Edge {
  int u, v, id;
  Edge(int u = 0, int v = 0, int id = 0) : u(u), v(v), id(id) {}
};

Edge edges[N];
int ans[N];
int min_time_same[N];
vector<int> edges_join_in_time[N];
int n, m, q;

DSU dsu(N);
vector<int> nodes;
vector<int> g[N];
int low[N], num[N], time_dfs, del[N];
int id_scc[N];
vector<int> stk;
vector<vector<int>> scc;
vector<int> T[N];

void dfs(int u) {
  low[u] = num[u] = ++time_dfs;
  stk.push_back(u);
  for (int v : g[u]) {
    if (del[v]) {
      continue;
    }
    if (!num[v]) {
      dfs(v);
      low[u] = min(low[u], low[v]);
    } else {
      low[u] = min(low[u], num[v]);
    }
  }
  if (low[u] == num[u]) {
    scc.push_back({});
    int v;
    do {
      v = stk.back();
      scc.back().push_back(v);
      id_scc[v] = scc.size();
      del[v] = 1;
      stk.pop_back();
    } while (v != u);
  }
}

void add_edge(int u, int v) {
  nodes.push_back(u);
  nodes.push_back(v);
  g[u].push_back(v);
}

void reset() {
  for (int u : nodes) {
    low[u] = num[u] = del[u] = 0;
    id_scc[u] = 0;
    g[u].clear();
  }
  nodes.clear();
  stk.clear();
  scc.clear();
  time_dfs = 0;
}

void solve(int l, int r, vector<int> indices) {
  if (l > r) {
    return;
  }
  int mid = (l + r) >> 1;
  reset();
  for (int i : indices) {
    if (i <= mid) {
      auto [s, t, _] = edges[i];
      s = dsu.root(s);
      t = dsu.root(t);
      add_edge(s, t);
    }
  }
  for (int u : nodes) {
    if (!num[u]) {
      dfs(u);
    }
  }
  vector<int> g1, g2;
  for (int i : indices) {
    if (i <= mid) {
      auto [u, v, _i] = edges[i];
      u = dsu.root(u), v = dsu.root(v);
      if (id_scc[u] == id_scc[v]) {
        min_time_same[i] = min(min_time_same[i], mid);
        g1.push_back(i);
      } else {
        g2.push_back(i);
      }
    } else {
      g2.push_back(i);
    }
  }
  solve(l, mid - 1, g1);
  for (int i : g1) {
    auto [u, v, _i] = edges[i];
    dsu.join(u, v);
  }
  solve(mid + 1, r, g2);
  indices.clear();
  g1.clear();
  g2.clear();
}

int32_t main() {
  ios_base::sync_with_stdio(0);
  cin.tie(0);
  cin >> n >> m >> q;
  for (int i = 1; i <= q; i++) {
    char type;
    int s, t;
    cin >> s >> t >> type;
    if (type == 'R') {
      edges[i] = Edge(t + n, s);
    } else {
      edges[i] = Edge(s, t + n);
    }
    min_time_same[i] = 1e9;
  }
  vector<int> idx(q);
  iota(idx.begin(), idx.end(), 1);
  solve(1, q, idx);
  for (int i = 1; i <= q; i++) {
    if (min_time_same[i] != 1e9) {
      edges_join_in_time[min_time_same[i]].push_back(i);
    }
  }
  DSU dsu(n + m);
  for (int i = 1; i <= q; i++) {
    for (int j : edges_join_in_time[i]) {
      auto [s, t, _] = edges[j];
      dsu.join(s, t);
    }
    cout << dsu.sum << "\n";
  }
  return 0;
}
```


## Tài liệu tham khảo

- [infossm](https://github.com/infossm/infossm.github.io/blob/master/_posts/2021-03-21-offline-incremental-SCC.md?plain=1) 
- [Codeforces](https://codeforces.com/blog/entry/91608)
# Dựng cây truy vấn

Thông thường khi giải quyết các bài tập yêu cầu: Đưa trạng thái hiện tại về trạng thái trước khi thực hiện truy vấn thứ $k$, mọi người thường hay dùng các [cấu trúc dữ liệu Persistent](https://vnoi.info/wiki/algo/data-structures/persistent-data-structures.md) để giải. Tuy nhiên, một số bài cho phép chúng ta xử lí offline, vậy ta có thể sử dụng phương pháp này để code cho đơn giản.

## Thuật toán

Đọc hết các truy vấn trước khi giải. Với mỗi truy vấn thứ $i$, nếu nó không là truy vấn đưa về trạng thái thứ $k$, nối nó đến đỉnh thứ $i-1$; ngược lại, nối nó đến đỉnh $k$. Khi này đồ thị chúng ta nhận được sẽ là một cây (do mỗi đỉnh chỉ nối đến một đỉnh duy nhất nằm ở đằng trước nó).

Sau khi đã đọc hết các truy vấn, ta sẽ DFS từ đỉnh số $0$ để giải bài toán.

``` 
void dfs(int u) {
  trả lời/thực hiện truy vấn u
  for (truy vấn v kề u) {
    dfs(v);
  }
  nếu truy vấn u ảnh hưởng đến dữ liệu, tiến hành 'rollback' thao tác vừa làm
}
```

Độ phức tạp: $O(q \times T)$, trong đó $T$ là thời gian thực hiện một truy vấn.

## Bài tập áp dụng
- [Codeforces 707D](https://codeforces.com/problemset/problem/707/D)

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>
	using namespace std;

	struct Query {
	  int t, i, j;
	  Query (int _t = 0, int _i = 0, int _j = 0) {
	    t = _t; i = _i; j = _j;
	  }
	};

	const int Q = 1e5 + 5;
	const int N = 1e3 + 5;

	int n, m, q;
	int a[N][N];
	int tot = 0;
	vector <int> adj[Q];
	int ans[Q];
	Query queries[Q];

	void dfs (int u) {
	  ans[u] = tot;
	  for (int v: adj[u]) {
	    if (queries[v].t == 1) {
	      bool changed = false;
	      int i = queries[v].i, j = queries[v].j;

	      if (a[i][j] == 0)
	        changed = true,
	        tot++,
	        a[i][j] = 1;

	      dfs(v);

	      if (changed)
	        a[i][j] = 0,
	        tot--;
	    }
	    else if (queries[v].t == 2) {
	      bool changed = false;
	      int i = queries[v].i, j = queries[v].j;

	      if (a[i][j] == 1)
	        changed = true,
	        tot--,
	        a[i][j] = 0;

	      dfs(v);

	      if (changed)
	        a[i][j] = 1,
	        tot++;
	    }
	    else if (queries[v].t == 3) {
	      int row = queries[v].i;
	      for (int col = 1; col <= m; col++)
	        tot -= a[row][col],
	        a[row][col] ^= 1,
	        tot += a[row][col];

	      dfs(v);

	      for (int col = 1; col <= m; col++)
	        tot -= a[row][col],
	        a[row][col] ^= 1,
	        tot += a[row][col];
	    }
	    else {
	      dfs(v);
	    }
	  }
	}

	int main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0); cout.tie(0);

	  cin >> n >> m >> q;

	  for (int i = 1; i <= q; i++) {
	    cin >> queries[i].t >> queries[i].i;
	    if (queries[i].t <= 2)
	      cin >> queries[i].j;
	    if (queries[i].t == 4)
	      adj[queries[i].i].push_back(i);
	    else
	      adj[i - 1].push_back(i);
	  }
	  
	  dfs(0);
	  for (int i = 1; i <= q; i++)
	    cout << ans[i] << "\n";

	  return 0;
	}
	```

- [TTRAVEL - SPOJ](https://vn.spoj.com/problems/TTRAVEL/)

??? abstract "Cài đặt"

    ```cpp linenums="1"
	#include <bits/stdc++.h>
	using namespace std;
	 
	const int N = 80005;
	 
	vector <int> adj[N];
	vector <int> cur;
	pair <char, int> queries[2*N];
	int ans[2*N];
	 
	void dfs (int u) {
	  for (int v: adj[u]) {
	    if (queries[v].first == 'a') {
	      cur.push_back(queries[v].second);
	      ans[v] = cur.back();
	      dfs(v);
	      cur.pop_back();
	    }
	    else if (queries[v].first == 's') {
	      int last = -1;
	      if (cur.size())
	        last = cur.back(), cur.pop_back();
	      if (cur.size()) ans[v] = cur.back();
	      else ans[v] = -1;
	      dfs(v);
	      if (last != -1)
	        cur.push_back(last);
	    }
	    else {
	      if (cur.size())
	        ans[v] = cur.back();
	      else ans[v] = -1;
	      dfs(v);
	    }
	  }
	}
	 
	int main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0); cout.tie(0);
	 
	  int n;
	  cin >> n;
	 
	  for (int i = 1; i <= n; i++) {
	    cin >> queries[i].first;
	    if (queries[i].first == 'a' || queries[i].first == 't')
	      cin >> queries[i].second;
	    if (queries[i].first == 't')
	      adj[queries[i].second - 1].push_back(i);
	    else
	      adj[i - 1].push_back(i);
	  }
	 
	  dfs(0);
	 
	  for (int i = 1; i <= n; i++) {
	    cout << ans[i] << "\n";
	  }
	 
	  return 0;
	}
	 
	```

??? success "Lời giải"

    Hai bài trên là một áp dụng cơ bản của dạng bài này, các bạn chỉ cần chép nguyên hàm DFS vào, mỗi truy vấn làm y như đề bảo là AC.

- [FIXSTR - PreVOI 2019](http://csloj.ddns.net/problem/1077)

??? success "Lời giải"

    Dựng cây Segment Tree; mỗi nút ở Segment Tree lưu thông tin: số ngoặc mở chưa được ghép với đứa nào trong đoạn $(i,j)$, tương tự với số ngoặc đóng.
    
    ```cpp linenums="1"
    struct Node {
        int open, close;
    };
    ```
    
    Như vậy khi ghép 2 nút con $L$, $R$ vào với nút cha ta sẽ làm như sau:
    
    ```cpp linenums="1"
    Node operator + (Node L, Node R) {
        Node ans = {L.open + R.open, L.close + R.close};
        int will_be_match = min(L.open, R.close);
        ans.open -= will_be_match;
        ans.close -= will_be_match;
        return ans;
    }
    ```
    
    Với các thông tin như vậy ta có thể dễ dàng suy ra trọng số của đoạn $[L,R]$. 

    Đến đây ta đã giải quyết được các truy vấn $1$ và $2$ trong $O(logN)$. Còn truy vấn $3$, bạn chỉ cần chép hàm DFS như trên là AC.
    
??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>
	using namespace std;

	const int N = 1e6 + 5;

	struct Node {
	    int not_matched_open, not_matched_close;

	    Node(int a = 0, int b = 0) {
	        not_matched_open = a;
	        not_matched_close = b;
	    }

	    Node operator+(const Node &rhs) const {
	        Node res;

	        int match = min(not_matched_open, rhs.not_matched_close);

	        res.not_matched_open = not_matched_open + rhs.not_matched_open - match;
	        res.not_matched_close = not_matched_close + rhs.not_matched_close - match;

	        return res;
	    }
	};

	struct Tree {
	    typedef Node T;
	    T f(T a, T b) { return a + b; }  // (any associative fn)
	    vector<T> s;
	    int n;
	    T unit = Node();
	    Tree(int n = 0, T def = Node()) : s(2 * n + 5, def), n(n) {}
	    void update(int pos, char val) {
	        for (s[pos += n] = Node(val == '(', val == ')'); pos /= 2;) {
	            s[pos] = f(s[pos * 2], s[pos * 2 + 1]);
	        }
	    }
	    T query(int b, int e) {  // query [b, e)
	        T ra = unit, rb = unit;
	        for (b += n, e += n; b < e; b /= 2, e /= 2) {
	            if (b % 2)
	                ra = f(ra, s[b++]);
	            if (e % 2)
	                rb = f(s[--e], rb);
	        }
	        return f(ra, rb);
	    }
	    int get_ans(int b, int e) {
	        e++;
	        T ans = query(b, e);
	        return ans.not_matched_close + ans.not_matched_open;
	    }
	};

	struct Query {
	    char type;
	    int i, j;

	    Query(char a = 0, int b = 0, int c = 0) {
	        type = a;
	        i = b;
	        j = c;
	    }

	    void read() {
	        cin >> type >> i;
	        if (type == 'Q')
	            cin >> j;
	    }
	} queries[N];

	char get_rev(char ch) { return ch == '(' ? ')' : '('; }

	vector<int> adj[N];
	Tree st;
	string s;
	int ans[N];
	int n, m;

	void dfs(int u) {
	    if (queries[u].type == 'Q') {
	        int i = queries[u].i, j = queries[u].j;
	        ans[u] = st.get_ans(i, j);
	    }

	    for (int v : adj[u]) {
	        if (queries[v].type == 'C') {
	            int idx = queries[v].i;
	            st.update(idx, get_rev(s[idx]));
	            s[idx] = get_rev(s[idx]);
	        }
	        dfs(v);
	        if (queries[v].type == 'C') {
	            int idx = queries[v].i;
	            st.update(idx, get_rev(s[idx]));
	            s[idx] = get_rev(s[idx]);
	        }
	    }
	}

	int main() {
	    ios_base::sync_with_stdio(0);
	    cin.tie(0);
	    cout.tie(0);

	    cin >> s;
	    n = s.size();
	    s = ' ' + s;

	    cin >> m;

	    st = Tree(n + 5);

	    for (int i = 1; i <= n; i++) st.update(i, s[i]);

	    for (int i = 1; i <= m; i++) {
	        queries[i].read();
	        if (queries[i].type != 'U')
	            adj[i - 1].push_back(i);
	        else
	            adj[queries[i].i - 1].push_back(i);
	    }

	    dfs(0);

	    for (int i = 1; i <= m; i++) {
	        if (queries[i].type == 'Q')
	            cout << ans[i] << "\n";
	    }

	    return 0;
	}
	```

* [Query-Max 4 - LQDOJ](https://lqdoj.edu.vn/problem/querymax4)
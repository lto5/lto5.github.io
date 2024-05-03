# Chia căn và các biến thể
*Bài được viết khi tác giả đang panic với môn TTHCM :fontawesome-regular-face-sad-tear:*

Trên VNOI đã có rất nhiều bài về chia căn (tới 2 blog và 2 contest Edu), tuy nhiên các kĩ thuật sử dụng trong contest Edu của VNOI lại chưa được trình bày cụ thể ở bài viết nào mà chỉ có thể thu được qua việc làm các bài tương tự. Vậy trong bài viết này chúng ta sẽ cùng tìm hiểu một số "dạng" chia căn thường gặp.

## Chia block

### Định nghĩa

Thông thường các bài toán chúng ta thường gặp sẽ là chia cấu trúc của ta thành $\sqrt{N}$ khối, mỗi khối chứa $\sqrt{N}$ phần tử. Với mỗi khối, ta lưu một vài thông tin có ích để có thể truy xuất dữ liệu trong độ phức tạp nhỏ. Với mỗi truy vấn, ta tìm các "khối" giao với đoạn cần quan tâm, và sau đó truy vấn từ các thông tin đã lưu trước đó để có thể truy xuất dữ liệu một cách nhanh chóng. 

### Ví dụ

[PQUERY - Siêu cúp 2023](https://oj.vnoi.info/problem/olp_sc23_pquery)

??? question "Đề bài"

    Cho dãy $A$ gồm các phần tử có giá trị $\leq 10^9$ và dãy $P$ là hoán vị của các số từ $1$ đến $N$. Xử lí 4 loại truy vấn:

    - Gán $A_{P_i}=A_{P_i}+v$ với $i \in [L, R]$
    - Tính tổng $A[L:R]$
    - Đổi chỗ $P_i, P_j$
    - Rollback về trước thời điểm thứ $T$

??? success "Lời giải"

    Để cho tiện, tất cả các chỉ số sử dụng trong bài ta sẽ chuyển về dạng $0-indexed$ thay vì $1$ như đề bài cho.
    Đầu tiên chúng ta sẽ tìm hiểu cách xử lí bài toán khi không có truy vấn rollback. Để xử lí truy vấn rollback, ta có thể dựng cây theo truy vấn và DFS, tham khảo tại [đây](https://lto5.github.io/old-blog/2022-08-18-query-tree/).
    Ta sẽ đồng thời chia mảng $A$ và $P$ thành $\sqrt{N}$ khối, và khối thứ $i$ sẽ quản lí đoạn $[i \times \sqrt{N}, (i + 1) \times \sqrt{N})$. Vậy mỗi truy vấn ta sẽ thực hiện như thế nào:

    - Truy vấn tăng: Với mỗi khối của $P$, ta lưu thêm một thông tin là $lazyP_i$ nghĩa là các chỉ số $P_j$ thuộc khối thứ $i$ của mảng $P$ đã bị tác động lên bao nhiêu lần.
        - Nếu $L,R$ thuộc cùng một block, ta thực hiện duyệt từ $L \Rightarrow R$ và tăng trâu như đề bài bảo.
        - Ngược lại, gọi $b_L$ và $b_R$ lần lượt là block chứa $L$ và chứa $R$. Với các block $i$ có chỉ số thuộc đoạn $[b_L+1,b_R)$, ta tiến hành gán $lazyP_i=lazyP_i+v$. Ngược lại, với các phần tử $P_i$ mà $i \in [L,R] \cap ([b_L \times N, (b_L+1) \times N) \cup [b_R \times N, b_{R+1} \times N))$, ta tiến hành tăng trâu như khi xử lí trường hợp trên.
        - Độ phức tạp cho phần này sẽ là $\sqrt{N}$:
            - Nếu $L$ và $R$ thuộc cùng một khối, thì $R-L+1 \leq \sqrt{N}$
            - Nếu $L$ và $R$ thuộc hai khối khác nhau:
                - Ta cần duyệt qua các khối, vậy nếu $L=1,R=N$ thì ta có tối đa $\frac{N}{\sqrt{N}}=\sqrt{N}$ khối
                - Độ dài của mỗi khối $b_L,b_R$ là $\sqrt{N}$. Vậy đoạn $[L,R]$ có độ dài như thế nào, thì phần giao của nó với $b_L,b_R$ cũng chỉ có độ dài tối đa là $\sqrt{N}$. Như vậy, phần tăng trâu này cũng chỉ duyệt tối đa là $2 \times \sqrt{N}$ phần tử.
            - Như vậy tổng số bước duyệt cho mỗi truy vấn tối đa là $3 \times \sqrt{N}$
    - Truy vấn tính tổng: Ta lưu một mảng $sumBlock_x$ thể hiện tổng các phần tử thuộc khối thứ $x$, nghĩa là tổng các phần tử $A_i$ với $i \in [x \times \sqrt{N}, (x+1) \times \sqrt{N})$
        - Nếu $L$ và $R$ thuộc chung một khối: duyệt $i$ từ $L$ đến $R$, và cộng đáp án với $A_i$. Tuy nhiên sẽ có trường hợp $A_i$ bị cập nhật ở một truy vấn tăng trước đó, nhưng truy vấn tăng này lại chưa cập nhật trực tiếp vào $A_i$. Vậy ta cần cộng đáp án với $lazyP_j$ với $j$ là chỉ số của block trên mảng $P$ và chứa chỉ số $i$, nghĩa là trong block thứ $j$ của mảng $P$ có tồn tại một giá trị $i$
        - Nếu $L$ và $R$ thuộc hai khối khác nhau:
            - Gọi $b_L$ và $b_R$ lần lượt là block chứa $L$ và chứa $R$. 
            - Với các block $i$ có chỉ số thuộc đoạn $[b_L+1,b_R)$, ta tiến hành tăng đáp án với $sumBlock_i$
            - Với các chỉ số $i \in [L,R] \cap ([b_L \times N, (b_L+1) \times N) \cup [b_R \times N, b_{R+1} \times N))$, ta tiến hành tăng trâu như khi xử lí trường hợp trên.
            - Tuy nhiên đáp án vẫn có thể sẽ bị sai với các phần tử thuộc các khối có chỉ số thuộc đoạn $[b_L+1,b_R)$, do các phần tử thuộc các khối này cũng có thể đã bị tác động bởi truy vấn tăng trước đó, tuy nhiên lại chưa được ghi nhận vào $sumBlock_i$
            - Một cách giải quyết đó là ta duyệt hết các khối $i$ thuộc mảng $P$, và cộng đáp án với $lazyP_i \times$ số phần tử $P_j$ với $j$ thuộc khối $i$ và $P_j$ thuộc khối $[b_L+1,b_R)$. Tuy nhiên ta không thể duyệt hết các phần tử $A$ thuộc khối trên được vì có thể sẽ lên đến $N$ phần tử. 
                - Nhận thấy các phần tử thuộc khối $[b_L+1,b_R)$ sẽ tạo thành một đoạn liên tiếp, như vậy ta có thể xác định được mút trái, mút phải của chúng. Với mỗi khối trên $P$ ta có thể lưu một fenwick tree độ dài $N$, với $P_{ij}=0/1$ thể hiện xem trong khối $P_i$ có phần tử nào bằng $j$ không. Như vậy với mỗi khối thuộc khối $P$ ta sẽ xử lí trong $\log(N)$, và có $\sqrt{N}$ block nên sẽ mất $\sqrt{N} \times \log(N)$ cho mỗi truy vấn, sẽ ăn được các subtask $N, Q \leq 70000$
                - Để giảm độ phức tạp ta có thể tính trước một mảng $intersect_{ij}$ là phần giao của khối thứ $i$ trên mảng $P$ và khối thứ $j$ trên mảng $A$. Với mỗi cặp $(i,j)$, ta duyệt các phần tử $P_x$ mà $x \in [i \times \sqrt{N}, (i+1) \times \sqrt{N})$ và cộng $1$ khi $j \times \sqrt{N} \leq P_x < (j + 1) \times \sqrt{N}$. Cùng với sự hỗ trợ của Prefix Sum, ta giảm được truy vấn xuống $\sqrt{N}$
        - Tương tự trường hợp trên, ta cũng dễ dàng chứng minh được độ phức tạp cho phần này chỉ là $O(\sqrt{N})$
        - Lưu ý mảng $sumBlock$ cũng phải được cập nhật khi thực hiện truy vấn update ở trên, việc update như thế nào xin được dành cho bạn đọc.
    - Truy vấn đổi chỗ:
        - Nếu $L$ và $R$ thuộc cùng một khối thì việc cần làm chỉ là $swap(P_L,P_R)$ do các thông tin ta lưu chỉ liên quan đến giá trị các phần tử xuất hiện trong khối, chứ không quan tâm đến thứ tự của chúng.
        - Ngược lại, gọi $b_L$ và $b_R$ lần lượt là khối chứa $L$ và $R$. 
            - Đầu tiên ta cần push các thay đổi trong $lazyP$ xuống các phần tử $a[.]$. Với mỗi khối $i \in \{b_L,b_R\}$, ta duyệt các $j$ nằm trong khối này và tiến hành thay đổi $A_{P_j}$ và $sumBlock_{\frac{P_j}{\sqrt{N}}}$. Sau đó gán $lazyP_i=0$
            - Sau đó ta cần thay đổi mảng $intersect$, và swap $P_L,P_R$. Phần update này xin dành cho bài đọc, gợi ý là bằng việc duyệt tất cả các phần tử cần quan tâm để cập nhật, độ phức tạp sẽ là $O(\sqrt{N})$

    Vậy tổng độ phức tạp là: $O((N+Q) \times \sqrt{N})$

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>

	using namespace std;

	#ifndef LOCAL
	#define cerr \
	  if (0) cerr
	#endif

	// #define int int64_t

	void debug_out() { cerr << '\n'; }
	template <typename Head, typename... Tail>
	void debug_out(Head H, Tail... T) {
	  cerr << " " << H;
	  debug_out(T...);
	}
	#define debug(...) cerr << "[" << #__VA_ARGS__ << "]:", debug_out(__VA_ARGS__)

	const int N = 150005;
	const int S = 490;

	int n, q;
	int64_t a[N];
	int p[N];
	int intersect[S][S];
	int64_t lazyP[S];
	array<int, 4> qr[N];
	int64_t ans[N];
	int64_t sumBlock[N];
	int belong[N];
	vector<int> g[N];

	// push all lazyP[.]
	void push_p(int block) {
	  // O(n) ?
	  // sumBlock[.]
	  // a[.]
	  // intersect[block, .]
	  for (int i = block * S; i <= min(n - 1, (block + 1) * S - 1); i++) {
	    a[p[i]] += lazyP[block];
	    sumBlock[p[i] / S] += lazyP[block];
	  }
	  lazyP[block] = 0;
	}

	void build(int i) {
	  for (int j = 0; j < S; j++) {
	    intersect[i][j] = 0;
	    int l = j * S, r = min(n - 1, (j + 1) * S - 1);
	    for (int ii = i * S; ii <= min(n - 1, (i + 1) * S - 1); ii++) {
	      intersect[i][j] += l <= p[ii] && p[ii] <= r;
	    }
	  }
	  for (int j = 1; j < S; j++) {
	    intersect[i][j] += intersect[i][j - 1];
	  }
	}

	void update(int l, int r, int v) {
	  int bL = l / S, bR = r / S;
	  if (bL == bR) {
	    for (int i = l; i <= r; i++) {
	      a[p[i]] += v;
	      sumBlock[p[i] / S] += v;
	    }
	    return;
	  }
	  for (int i = bL + 1; i <= bR - 1; i++) {
	    lazyP[i] += v;
	  }
	  for (int i = l; i < (bL + 1) * S; i++) {
	    a[p[i]] += v;
	    sumBlock[p[i] / S] += v;
	  }
	  for (int i = bR * S; i <= r; i++) {
	    a[p[i]] += v;
	    sumBlock[p[i] / S] += v;
	  }
	}

	int64_t cal(int l, int r) {
	  int bL = l / S, bR = r / S;
	  // debug(l, r);
	  int64_t ans = 0;
	  if (bL == bR) {
	    for (int i = l; i <= r; i++) {
	      // debug(a[i], belong[i], lazyP[belong[i]]);
	      ans += a[i] + lazyP[belong[i]];
	    }
	  } else {
	    for (int i = l; i < (bL + 1) * S; i++) {
	      ans += a[i] + lazyP[belong[i]];
	    }
	    for (int i = bL + 1; i <= bR - 1; i++) {
	      ans += sumBlock[i];
	    }
	    if (bL + 1 <= bR - 1) {
	      for (int i = 0; i < S; i++) {
	        ans += lazyP[i] * (intersect[i][bR - 1] - intersect[i][bL]);
	      }
	    }
	    for (int i = bR * S; i <= r; i++) {
	      ans += a[i] + lazyP[belong[i]];
	    }
	  }
	  return ans;
	}

	void revert(int blk) {
	  for (int j = S - 1; j >= 1; j--) {
	    intersect[blk][j] -= intersect[blk][j - 1];
	  }
	}

	void accum(int blk) {
	  for (int j = 1; j < S; j++) {
	    intersect[blk][j] += intersect[blk][j - 1];
	  }
	}

	void add(int block, int pos, int delta) {
	  int block_pos = p[pos] / S;
	  int lo = block_pos * S, hi = min(n - 1, (block_pos + 1) * S - 1);
	  intersect[block][block_pos] += delta * (lo <= p[pos] && p[pos] <= hi);
	}

	// p_l, p_r
	void swap_p(int l, int r) {
	  int bL = l / S, bR = r / S;
	  if (bL == bR) {
	    swap(p[l], p[r]);
	    return;
	  }
	  push_p(bL);
	  push_p(bR);
	  revert(bL);
	  revert(bR);
	  add(bL, l, -1);
	  add(bR, r, -1);
	  swap(p[l], p[r]);
	  belong[p[l]] = l / S;
	  belong[p[r]] = r / S;
	  add(bL, l, 1);
	  add(bR, r, 1);
	  // intersect[bL][block contains p[l]] -=
	  accum(bL);
	  accum(bR);
	}

	void dfs(int u) {
	  auto [t, l, r, v] = qr[u];
	  if (t == 2) ans[u] = cal(l, r);
	  for (int i : g[u]) {
	    auto [t, l, r, v] = qr[i];
	    if (t == 1) {
	      update(l, r, v);
	    } else if (t == 3) {
	      swap_p(l, r);
	    }
	    dfs(i);
	    if (t == 1) {
	      update(l, r, -v);
	    } else if (t == 3) {
	      swap_p(l, r);
	    }
	  }
	}

	int32_t main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);
	#ifdef LOCAL
	#define task "a"
	#else
	#define task "PQUERY"
	#endif
	  if (fopen(task ".inp", "r")) {
	    freopen(task ".inp", "r", stdin);
	    freopen(task ".out", "w", stdout);
	  }
	  cin >> n >> q;
	  for (int i = 0; i < n; i++) {
	    cin >> a[i];
	    sumBlock[i / S] += a[i];
	  }
	  for (int i = 0; i < n; i++) {
	    cin >> p[i];
	    --p[i];
	    belong[p[i]] = i / S;
	  }
	  for (int i = 0; i < S; i++) {
	    build(i);
	  }
	  for (int i = 1; i <= q; i++) {
	    cin >> qr[i][0];
	    if (qr[i][0] == 1) {
	      cin >> qr[i][1] >> qr[i][2] >> qr[i][3];
	      --qr[i][1];
	      --qr[i][2];
	    } else if (qr[i][0] == 2 || qr[i][0] == 3) {
	      cin >> qr[i][1] >> qr[i][2];
	      --qr[i][1];
	      --qr[i][2];
	    } else {
	      cin >> qr[i][1];
	      g[qr[i][1] - 1].push_back(i);
	    }
	    if (qr[i][0] != 4) {
	      g[i - 1].push_back(i);
	    }
	  }
	  dfs(0);
	  for (int i = 1; i <= q; i++)
	    if (qr[i][0] == 2) cout << ans[i] << "\n";
	  return 0;
	}
	```

### Bài tập áp dụng

- [Codeforces 13E](https://codeforces.com/problemset/problem/13/E)
- [XOROCC](https://oj.vnoi.info/problem/fc_fyt_day1_xorocc)
- [FNCS - Codechef](https://www.codechef.com/NOV14/problems/FNCS/)
- [TLEOJ Cup Final Day 2 - Tráo bài](https://tleoj.edu.vn/problem/26h)

## Chia theo truy vấn

### Giới thiệu

Tư tưởng chính của thuật toán sẽ là: Khi đọc được một truy vấn thay đổi, thay vì tác động trực tiếp lên các cấu trúc dữ liệu để cập nhật, ta sẽ "ghi nhớ" lại truy vấn này vào một mảng ``buffer``. Khi gặp một truy vấn hỏi, kết hợp với dữ liệu có sẵn, cùng mảng ``buffer``, sẽ cho ra được đáp án. Khi ``buffer`` đầy, ta thực hiện cập nhật các thao tác trong ``buffer`` lên cấu trúc sẵn có.

### Ví dụ

[Reversal and Sum - CSES](https://cses.fi/problemset/task/2074)

??? question "Đề bài"

    Cho mảng $a$ có $n$ phần tử. Xử lý $q$ truy vấn thuộc hai loại:

    - Đảo ngược đoạn $[l,r]$
    - Tính tổng đoạn $[l,r]$

??? success "Lời giải"

    Trước tiên, ta cũng chia mảng $a$ ra thành $\frac{n}{T}$ đoạn, mỗi đoạn có $T$ phần tử. Mỗi khối sẽ chứa các phần tử có chỉ số $i \times T$ đến $(i+1)\times T-1$

    Với mỗi truy vấn tìm tổng $[l,r]$; ta sẽ tìm block chứa $l$, và cắt nó ra làm hai phần: từ đầu block chứa $l$ đến $l-1$, và từ $l$ đến cuối block chứa $l$. Sau đó, ta tiếp tìm block chứa $r+1$, và cắt nó ra làm hai phần tương tự như trên.

    ![image](https://i.imgur.com/7zqlRWh.png)

    Lúc này, trong mảng chỉ gồm các khối nằm gọn trong đoạn $[l,r]$, ta tiến hành duyệt qua các khối này và ghi nhận tổng vào đáp án.

    ![image](https://i.imgur.com/hLeba3z.png)

    Còn với truy vấn đảo ngược đoạn $[l,r]$, ta cũng tiến hành cắt ghép như vậy. Với mỗi khối, ta lưu thêm một trường ``is_reversed``, nghĩa là trước đây khối này đã được thực hiện truy vấn đảo ngược chưa. Lưu ý, đảo ngược 2 lần có nghĩa là không đảo ngược. Vậy nên, ``is_reversed`` chỉ có hai trạng thái: $0/1$. Nhận thấy các khối nằm gọn trong đoạn $[l,r]$ thuộc một đoạn liên tiếp. Nên ta chỉ cần đảo ngược đoạn này cũng như gán ``is_reversed=!is_reversed`` với các khối này.

    Với mỗi truy vấn, ta tạo thêm 2 block mới. Vậy trường hợp xấu nhất ta sẽ tạo thêm $q$ block và phải duyệt qua toàn bộ các block này. Để tránh dính phải trường hợp này, sau mỗi $T$ truy vấn, ta tiến hành ``rebuild`` lại cấu trúc, nghĩa là trả lại mảng $a$ về giá trị của nó sau $T$ truy vấn, và chia lại nó thành $\frac{n}{T}$ block.

    Lưu ý còn một phần nữa là việc xử lí các chỉ số trong khi thực hiện đảo ngược khối, phần này xin dành cho bạn đọc.

    Suy ra với mỗi truy vấn, ta duyệt qua không quá $3T$ khối. Và cứ $T$ lần ta lại xây dựng lại cấu trúc, vậy ta có $\frac{q}{T}$ lần rebuild và mỗi lần rebuild mất $O(n)$. Vậy độ phức tạp cuối cùng là $q \times 3T + n \times \frac{q}{T}$. Ta có thể chọn $T=\sqrt{n}$ để đạt một độ phức tạp khá tốt và đủ để AC bài này.

    > Bài toán vẫn có thể giải được bằng phương pháp chia block giống như ở trên, tuy nhiên việc cài đặt sẽ phức tạp hơn. Vẫn có nhiều bài toán chỉ có thể giải được bằng phương pháp này.

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>
	 
	using namespace std;
	 
	 
	const int N = 2e5 + 5;
	const int S = 450;
	 
	struct Range {
	  int l, r;
	  int64_t sum;
	  bool is_reversed;
	  Range(int l = 0, int r = 0, int64_t sum = 0, bool is_reversed = 0) : l(l), r(r), sum(sum), is_reversed(is_reversed) {}
	};
	 
	int n, q;
	int a[N];
	int new_a[N];
	int64_t s[N];
	Range segments[N];
	int cnt_ranges;
	 
	// xây dựng lại các block trong a
	void rebuild() {
	  int cur = 0;
	  for (int i = 1; i <= cnt_ranges; i++) {
	    auto [l, r, sum, is_rev] = segments[i];
	    if (is_rev) {
	      for (int j = r; j >= l; j--) {
	        new_a[++cur] = a[j];
	      }
	    } else {
	      for (int j = l; j <= r; j++) {
	        new_a[++cur] = a[j];
	      }
	    }
	  }
	  for (int i = 1; i <= n; i++) {
	    a[i] = new_a[i];
	    s[i] = s[i - 1] + a[i];
	  }
	  cnt_ranges = 0;
	  for (int i = 1; i <= n; i += S) {
	    segments[++cnt_ranges] = Range(i, min(n, i + S - 1), s[min(i + S - 1, n)] - s[i - 1], 0);
	  }
	}
	 
	void insert_seg(int pos, Range rng) {
	  for (int i = cnt_ranges; i >= pos; i--) {
	    segments[i + 1] = segments[i];
	  }
	  segments[pos] = rng;
	  cnt_ranges++;
	}
	 
	// tách block chứa `pos` thành 2 phần: từ đầu block chứa `pos` -> `pos` - 1, và từ `pos` -> cuối block chứa `pos`
	// và trả về chỉ số của block chứa `pos`
	int split(int pos) {
	  int cur_low = -1, cur_high = 0;
	  for (int i = 1; i <= cnt_ranges; i++) {
	    auto [l, r, sum, is_rev] = segments[i];
	    int len = r - l + 1;
	    int next_low = cur_high + 1;
	    int next_high = next_low + len - 1;
	    cur_low = next_low;
	    cur_high = next_high;
	    // đang ở block bên trái `pos`
	    if (next_high < pos) continue;
	    if (next_low == pos) {
	      return i;
	    }
	    Range range_left, range_right;
	    int left_len = pos - next_low, right_len = next_high - pos + 1;
	    if (!is_rev) {
	      range_left = Range(l, l + left_len - 1, s[l + left_len - 1] - s[l - 1], 0);
	      range_right = Range(l + left_len, r, s[r] - s[l + left_len - 1], 0);
	    } else {
	      range_left = Range(r - left_len + 1, r, s[r] - s[r - left_len], 1);
	      range_right = Range(l, r - left_len, s[r - left_len] - s[l - 1], 1);
	    }
	    segments[i] = range_left;
	    insert_seg(i + 1, range_right);
	    return i + 1;
	  }
	  return cnt_ranges + 1;
	}
	 
	void do_reverse(int l, int r) {
	  // xuất phát từ điểm l
	  int block_l = split(l);
	  // kết thúc từ điểm r -> phải xoá điểm r+1
	  int block_r = split(r + 1);
	  for (int i = block_l; i < block_r; i++) {
	    segments[i].is_reversed ^= 1;
	  }
	  reverse(segments + block_l, segments + block_r);
	}
	 
	int64_t get_sum(int l, int r) {
	  int block_l = split(l);
	  int block_r = split(r + 1);
	  int64_t ans = 0;
	  for (int i = block_l; i < block_r; i++) {
	    ans += segments[i].sum;
	  }
	  return ans;
	}
	 
	int32_t main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);
	  cin >> n >> q;
	  for (int i = 1; i <= n; i++) {
	    cin >> a[i];
	    new_a[i] = a[i];
	  }
	  rebuild();
	  for (int i = 1; i <= q; i++) {
	    int t, l, r;
	    cin >> t >> l >> r;
	    if (t == 1) {
	      do_reverse(l, r);
	    } else {
	      cout << get_sum(l, r) << "\n";
	    }
	    // mỗi truy vấn tạo ra tối đa hai đoạn mới
	    // khi số đoạn thêm vào vượt quá sqrt(q), khởi tạo lại để đảm bảo số đoạn cần xét trong mỗi query không vượt quá 2 * sqrt(n)
	    // số lần khởi tạo lại là q / sqrt(q) = sqrt(q), mỗi lần khởi tạo mất O(n) -> n * sqrt(q)
	    if (i % S == 0) {
	      rebuild();
	    }
	  }
	  return 0;
	}
	```

??? note "Bonus"

    *Nếu có thêm truy vấn chèn/xoá phần tử, bạn có làm được không, và độ phức tạp tốt nhất là bao nhiêu?*

### Bài tập áp dụng

- [Edu SQRT Contest - Part 1](https://oj.vnoi.info/contest/sqrt)
- [Edu SQRT Contest - Part 2](https://oj.vnoi.info/contest/sqrt)
- [TopCoder 13206 - TreeColoring](https://vjudge.net/problem/TopCoder-13206)

## Chia 2 cách làm

### Định nghĩa

Giả sử ta có $N$ tập $A_1, A_2, \cdots, A_N$ và $|A_1| + |A_2| + \cdots + |A_N| = M$. Khi đó ta có một nhận xét: có không quá $\sqrt{M}$ tập $A_i$ có $|A_i|>\sqrt{M}$. Chứng minh rất dễ dàng bằng phản chứng: nếu có nhiều hơn $\sqrt{M}$ tập lớn như vậy, thì $|A_1| + |A_2| + \cdots + |A_N|>\sqrt{M} \times \sqrt{M}=M$, vô lí.

Như vậy, tư tưởng của các bài toán này sẽ là phân các tập hợp ra làm hai loại: một loại có ít hon $\sqrt{M}$ phần tử, và một loại có nhiều hơn $\sqrt{M}$ phần tử. Lợi dụng vào nhận xét trên, ta sẽ chọn cách làm thích hợp với từng tập hợp để độ phức tạp cho việc xử lí từng truy vấn không vượt quá $\sqrt{M}$

### Ví dụ

[398D Codeforces](https://codeforces.com/contest/398/problem/D)

??? question "Đề bài"

    Cho đồ thị $N$ đỉnh, $M$ cạnh. Xử lí 5 loại truy vấn:

    - Gán $F_u=1$
    - Gán $F_u=0$
    - Nối cạnh $(u,v)$
    - Xoá cạnh $(u,v)$
    - Đếm số đỉnh $v$ kề với $u$ và $F_v=1$

??? success "Lời giải"

    ??? tip "Thuật trâu 1"
        Sử dụng ``std::set`` lưu trữ danh sách kề của đồ thị. Với mỗi truy vấn thêm/xoá cạnh, ta xử lí trong $O(log)$ bằng các thao tác chèn xoá trên ``set``. Với mỗi truy vấn gán, ta gán $F$ tương ứng bằng $1/0$. Với truy vấn hỏi, ta duyệt toàn bộ danh sách các đỉnh kề với $u$ và đếm số đỉnh $v$ có $F_v=1$. Thuật toán này có độ phức tạp xấu nhất là $O(nq)$ khi ta có thể tạo test để làm các truy vấn hỏi chạy trong $O(n)$ bằng việc cho đồ thị $n$ đỉnh và $q$ truy vấn hỏi đỉnh $1$ như sau:

        ![image](https://i.imgur.com/nr5czAy.png)

    ??? tip "Thuật trâu 2"
        Ta lưu mảng $ans_u$ là đáp án cho đỉnh $u$. Với mỗi truy vấn cập nhật $F_u$, ta duyệt toàn bộ đỉnh $v$ kề $u$ và cập nhật mảng $ans_v$ tương ứng. Với mỗi truy vấn thêm xoá cạnh, ta dễ dàng cập nhật $ans_u$ và $ans_v$ trong $O(1)$. Còn với truy vấn hỏi, ta chỉ việc in ra $ans_u$. Tương tự thuật toán trên, ta cũng có thể chỉ ra được worst case của mỗi truy vấn cập nhật là $O(n)$

    ??? tip "Thuật toán chuẩn"
        Nhận thấy nhược điểm của cả hai thuật toán là đều chạy chậm khi một đỉnh bị kề với quá nhiều đỉnh khác. Ta sẽ thử kết hợp hai thuật toán trên lại với nhau:

        - Trước tiên, đọc vào tất cả các truy vấn. Xác định mảng $deg_u$ là bậc tối đa của đỉnh u, mảng này có thể được tính bằng cách add hết các cạnh trong đồ thị cũng như các truy vấn thêm cạnh
        - Gọi $u$ là đỉnh "lớn" nếu $deg_u>S$, và $u$ là đỉnh "bé" nếu ngược lại. Lưu mảng $ans_u$ là đáp án cho đỉnh $u$, lưu ý mảng $ans$ chỉ lưu cho các đỉnh $u$ là đỉnh "lớn".
        - Ta sẽ xử lí các truy vấn như sau:
            - Với truy vấn thay đổi $F_u$, ta thay đổi $F_u$ như bình thường. Sau đó, ta duyệt các đỉnh $v$ là đỉnh "lớn" và kề với $u$, và cập nhật $ans_v$
            - Với truy vấn thay đổi cạnh, ta thay đổi danh sách kề bằng ``std::set`` như thuật toán trâu. Nếu $u$ hoặc $v$ là đỉnh "lớn", thì ta cần cập nhật lại $ans_u$ và $ans_v$ tương ứng.
            - Với truy vấn hỏi, nếu $u$ là đỉnh lớn, thì ta truy vấn trong $ans_u$. Ngược lại, ta làm như thuật toán trâu thứ nhất để tính đáp án.

        Đánh giá độ phức tạp:

        - Đặt $D=\sum_{i=1}^{N}deg_i$
        - Khi truy vấn vào đỉnh bé, ta chỉ cần duyệt không quá $O(S)$
        - Khi truy vấn vào đỉnh lớn, ta cần duyệt không quá $O(\frac{D}{S})$
        - Vậy độ phức tạp cho $Q$ truy vấn tối đa là $O(Q(S+\frac{D}{S}))$. Ta cần tìm $S$ để biểu thức đạt giá trị nhỏ nhất. Áp dụng bất đẳng thức Cosi, ta có $S + \frac{D}{S} \geq 2 \times \sqrt{D}$. Dấu bằng xảy ra tại $S=\frac{D}{S} \Leftrightarrow S=\sqrt{D}$. 
        - Như vậy, có thể kết luận độ phức tạp tối đa là $O(Q\sqrt{M+Q})$

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>

	using namespace std;

	const int N = 50005;
	const int M = 150005;
	const int Q = 250005;
	const int S = 650;

	struct Query {
	  char type;
	  int u, v;
	  Query(char type = 0, int u = 0, int v = 0) : type(type), u(u), v(v) {}
	  friend istream& operator>>(istream& ist, Query& qr) {
	    ist >> qr.type >> qr.u;
	    if (qr.type == 'A' || qr.type == 'D') {
	      ist >> qr.v;
	    }
	    return ist;
	  }
	};

	vector<int> g[N]; // chỉ lưu danh sách kề đối với các đỉnh ``bé``
	int id_big[N];
	int cnt_big;
	bitset<N> adj[(M + Q) / S + 5]; // còn với các đỉnh lớn, ta có thể lưu dưới dạng ma trận kề, do chỉ có <= sqrt đỉnh lớn -> bộ nhớ N*sqrt
	int deg[N];  // bậc tối đa của một đỉnh
	Query qr[Q];
	int is_online[N];
	int answer_for_big_nodes[N];  // ta chỉ quan tâm đáp án của các đỉnh ``lớn``, vì có thể suy được đáp án của các đỉnh nhỏ trong O(sqrt)

	bool is_big_node(int u) {
	  return deg[u] >= S;
	}

	void turn_on(int u) {
	  is_online[u] = 1;
	  for (int i = 1; i <= cnt_big; i++) {
	    if (adj[i][u]) answer_for_big_nodes[i]++;
	  }
	}

	void turn_off(int u) {
	  is_online[u] = 0;
	  for (int i = 1; i <= cnt_big; i++) {
	    if (adj[i][u]) answer_for_big_nodes[i]--;
	  }
	}

	void add_edge(int u, int v) {
	  if (!is_big_node(u)) g[u].push_back(v);
	  if (!is_big_node(v)) g[v].push_back(u);
	  if (is_big_node(v)) {
	    adj[id_big[v]][u] = 1;
	    answer_for_big_nodes[id_big[v]] += is_online[u];
	  }
	  if (is_big_node(u)) {
	    adj[id_big[u]][v] = 1;
	    answer_for_big_nodes[id_big[u]] += is_online[v];
	  }
	}

	void remove_edge(int u, int v) {
	  // ta có thể erase trâu vì số lượng đỉnh kề với u tối đa là sqrt()
	  if (!is_big_node(u)) g[u].erase(find(g[u].begin(), g[u].end(), v));
	  if (!is_big_node(v)) g[v].erase(find(g[v].begin(), g[v].end(), u));
	  if (is_big_node(v)) {
	    adj[id_big[v]][u] = 0;
	    answer_for_big_nodes[id_big[v]] -= is_online[u];
	  }
	  if (is_big_node(u)) {
	    adj[id_big[u]][v] = 0;
	    answer_for_big_nodes[id_big[u]] -= is_online[v];
	  }
	}

	int query(int u) {
	  if (is_big_node(u)) {
	    return answer_for_big_nodes[id_big[u]];
	  }
	  int ans = 0;
	  for (int v : g[u]) {
	    ans += is_online[v];
	  }
	  return ans;
	}

	int32_t main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);
	  int n, m, q;
	  cin >> n >> m >> q;
	  int o;
	  cin >> o;
	  while (o--) {
	    int u;
	    cin >> u;
	    is_online[u] = 1;
	  }
	  for (int i = 1; i <= m; i++) {
	    int u, v;
	    cin >> u >> v;
	    ++deg[u];
	    ++deg[v];
	    g[u].push_back(v);
	    g[v].push_back(u);
	  }
	  for (int i = 1; i <= q; i++) {
	    cin >> qr[i];
	    if (qr[i].type == 'A') {
	      ++deg[qr[i].u];
	      ++deg[qr[i].v];
	    }
	  }
	  for (int u = 1; u <= n; u++) {
	    if (is_big_node(u)) {
	      id_big[u] = ++cnt_big;
	    }
	  }
	  for (int u = 1; u <= n; u++) {
	    if (is_big_node(u)) {
	      for (int v : g[u]) {
	        adj[id_big[u]][v] = 1;
	        answer_for_big_nodes[id_big[u]] += is_online[v];
	      }
	    }
	  }
	  for (int i = 1; i <= q; i++) {
	    auto [type, u, v] = qr[i];
	    if (type == 'O') {
	      turn_on(u);
	    } else if (type == 'F') {
	      turn_off(u);
	    } else if (type == 'A') {
	      add_edge(u, v);
	    } else if (type == 'D') {
	      remove_edge(u, v);
	    } else {
	      cout << query(u) << "\n";
	    }
	  }
	  return 0;
	}
	```

### Bài tập áp dụng

- [Educational SQRT Contest - Part 2](https://oj.vnoi.info/problem/sqrt2_g), hầu hết các bài tập trong phần này đều khai thác đến khía cạnh chia 2 cách làm cũng như chia theo truy vấn.
- [1921F - Codeforces](https://codeforces.com/contest/1921/problem/F)
- [ABC335 F](https://atcoder.jp/contests/abc335/tasks/abc335_f)
- [KTREEC](https://oj.vnoi.info/problem/ktreec)
- [FC100 - GOODARR](https://oj.vnoi.info/problem/fc100_goodarr)
- [PATULJCI](https://vn.spoj.com/problems/PATULJCI/)

## Thuật toán Mo

### Không cập nhật

Đây là một thuật toán khá kinh điển, bạn đọc có thể tham khảo về lý thuyết cơ bản của nó tại tại [VNOI Wiki](https://vnoi.info/wiki/algo/data-structures/mo-algorithm.md). 

#### Mo trên cây

##### Truy vấn cây con

Bạn đọc có thể tham khảo [bài viết của VNOI](https://vnoi.info/wiki/algo/graph-theory/euler-tour-on-tree.md#c%E1%BA%ADp-nh%E1%BA%ADt-truy-v%E1%BA%A5n-%C4%91%E1%BB%91i-v%E1%BB%9Bi-%C4%91%E1%BB%89nh-v%C3%A0-c%C3%A2y-con) đã nói về vấn đề này. Cụ thể, khi ta đánh số lại các đỉnh theo Euler tour, thì khi đó cây con gốc $u$ sẽ thuộc một dãy liên tiếp trên mảng. Khi đó, ta có thể sắp xếp lại các truy vấn theo thuật toán Mo và thực hiện truy vấn

##### Truy vấn đường đi

Bài viết về Euler tour trên [VNOI](https://vnoi.info/wiki/algo/graph-theory/euler-tour-on-tree.md#c%E1%BA%ADp-nh%E1%BA%ADt-%C4%91%E1%BB%89nh-truy-v%E1%BA%A5n-%C4%91%C6%B0%E1%BB%9Dng-%C4%91i) đã nêu đến 3 cách đánh số lại các đỉnh để thực hiện truy vấn. Ở đây, chúng ta quan tâm đến cách số 2.

Cụ thể, theo đường đi Euler, ta thêm đỉnh $u$ vào mảng $tour$ khi thăm đỉnh $u$ lần đầu tiên và lần cuối cùng.

Ví dụ:

![image](https://i.imgur.com/vItQSTz.png)

$tour = [1, 2, 3, 4, 4, 3, 5, 6, 6, 5, 2, 7, 8, 8, 9, 9, 7, 1]$

Tương tự, ta cũng gọi $st_u$ và $en_u$ lần lượt là thời điểm đầu tiên và cuối cùng gặp đỉnh $u$ trong mảng $tour$. Mỗi đỉnh $u$ xuất hiện đúng 2 lần trong mảng này.

Giả sử ta có một truy vấn hỏi một thông tin gì đó trên đường đi $(u,v)$. Vậy ta sẽ lợi dụng mảng $tour$ thế nào để đưa về truy vấn trên đoạn liên tiếp?

??? success "Spoiler"

    - Nếu $u$ là tổ tiên của $v$, lúc này ta có $st_u \leq st_v \leq en_v \leq en_u$
        - Ta không cần quan tâm đến các đỉnh có chỉ số không thuộc đoạn $[st_u, st_v]$, do những điểm thuộc đường đi từ $u$ đến $v$ chắc chắn thuộc đoạn này.
        - Nếu $p$ thuộc đường đi từ $u$ đến $v$, nghĩa là $st_u \leq st_p \leq st_v < en_v$, nghĩa là $p$ xuất hiện duy nhất một lần trong đoạn cần xét.
        - Ngược lại, nghĩa là $p$ không phải là tổ tiên của $v$, vậy ta có $st_u<st_p<en_p<st_v$, nghĩa là $p$ xuất hiện hai lần trên đoạn.
        - Vậy bài toán có thể chuyển về thành quản lý các đỉnh xuất hiện đúng 1 lần trong đoạn $[st_u,st_v]$
    - Nếu $u$ và $v$ thuộc hai cây con khác nhau
        - Gọi $p=lca(u,v)$
        - Lúc này, đoạn chúng ta cần quan tâm là $[en_u, st_v] \subset [st_p,st_p]$
        - Các đỉnh $v'$ thuộc subtree khác gốc $p$ mà nằm trong đoạn $[en_u,st_v]$ thì có $st_p<en_u<st_{p'}<en_{p'}<st_v$ hoặc $en_u<st_v<st_{v'}$, vậy nên chúng chỉ xuất hiện 0 hoặc 2 lần trong dãy đang xét
        - Còn các đỉnh $p'$ là "cha" của $u$ và $v$ và nằm trên đường đi từ $u$ đến $v$, thì khi duyệt theo order từ $en_u \rightarrow st_v$, hoặc là ta thêm điểm xuất hiện lần đầu của $p'$, hoặc là thêm điểm cuối cùng xuất hiện $p'$, vậy nên ta cũng chỉ xét các đỉnh này đúng một lần.
        - Đỉnh $p$ có thể chưa được thêm vào cấu trúc, do ta chỉ xét các đỉnh có $st_p<en_u<st_v<en_p$. Như vậy, ta cần thêm đoạn $[st_p,st_p]$
        - Kết luận, ta chỉ cần xét những đỉnh xuất hiện duy nhất một lần trong đoạn cần quan tâm.

##### Bài toán áp dụng

- [COT2 - SPOJ](http://www.spoj.com/problems/COT2/) - [triển khai mẫu](https://ideone.com/6NVoPD)
- [Frank Sinatra - Prob.F](https://codeforces.com/gym/100962/attachments/download/4255/20152016-petrozavodsk-winter-training-camp-moscow-su-trinity-contest-en.pdf)
- [Vasya and Little Bear - Codechef](https://www.codechef.com/ALKH2016/problems/VLB)
- [Dating](https://codeforces.com/contest/852/problem/I)

#### Một số thủ thuật tối ưu

##### Một cách sắp xếp truy vấn khác

Đây là một cách sắp xếp các truy vấn để giảm độ phức tạp từ $O((n+q) \times \sqrt{n})$ xuống $O(n \times \sqrt{q})$, sử dụng Hilbert Curve. Các bạn có thể tham khảo bài viết [sau](https://codeforces.com/b\log/entry/61203) để hiểu rõ hơn về tư tưởng.

##### Khử $\log$

Xét bài toán: Cho mảng $A$ gồm $n$ phần tử và $q$ truy vấn, mỗi truy vấn yêu cầu tìm $mex(a_l, \cdots, a_r)$. Bài toán này có thể giải offline trong $O(q\log(n))$ hoặc online với các cấu trúc persistent, tuy nhiên những cách giải này sẽ không được xét đến trong phạm vi blog này, mà ta sẽ xét cách làm sử dụng thuật toán Mo:

- Sắp xếp các truy vấn theo thuật toán Mo
- Duy trì một set $S$ thể hiện các phần tử chưa xuất hiện trong mảng. Ban đầu $S$ chứa các phần tử từ $0 \rightarrow n$ do giá trị mex tối đa của ta chỉ là $n$ nên ta cũng chỉ quan tâm đến đó
- Bằng set $S$ này, với mỗi truy vấn ta có thể thêm/xoá phần tử trong $O(\log(n))$ và tìm min trong $O(\log(n))$

Độ phức tạp của cách làm này là $O(n \sqrt{n} \log(n))$, quá chậm và chậm hơn nữa khi ta sử dụng ``std::set``. Có cách làm khác sử dụng hash table, tuy nhiên constant của nó cũng khá cao và không phù hợp với các bài toán cần truy xuất một lượng lớn các truy vấn thêm/xoá phần tử như bài toán này.

Ta xét một cấu trúc dữ liệu cho phép:

- Chèn/xoá phần tử trong $O(1)$
- Tìm mex trong $O(\sqrt{n})$

*Sau tất cả, mình lại trở về với nhau...*

Đúng vậy, ta lại quay về việc chia block như ban đầu. Nhận thấy ta có thể lưu mảng $cnt_x$ là các phần tử có giá trị bằng $x$. Với mỗi truy vấn tìm $mex$, ta duyệt $x$ từ $0$ đến $n$, và dừng lại khi gặp $cnt_x=0$. Tuy nhiên việc làm này quá lâu, ta sẽ tối ưu lại bằng phương pháp chia block như sau:

- Chia mảng $cnt$ thành $\sqrt{n}$ khối. Với mỗi khối ta lưu $cntB_i$ là số lượng phần tử phân biệt xuất hiện trên khối thứ $i$
- Với mỗi truy vấn thêm $x$, ta tăng $cnt_x$ lên $1$ và $cntB_{\frac{x}{\sqrt{N}}}$ lên 1 nếu $cnt_x=1$. Tương tự với truy vấn xoá.
- Với mỗi truy vấn tìm mex, ta duyệt các khối và tìm khối $i$ đầu tiên có $cntB_i \neq \sqrt{n}$. Sau đó ta duyệt các phần tử $j$ thuộc khối $i$, và tìm $j$ đầu tiên có $cnt_j=0$. Ta có $\sqrt{n}$ khối, mỗi khối có $\sqrt{n}$ phần tử. Vậy độ phức tạp cho phần này là $O(\sqrt{n})$

Như vậy là ta giải quyết xong bài toán này trong $O((n+q) \times \sqrt{n})$

Bạn đọc có thể cài đặt thử tại link: [1000F - Codeforces](https://codeforces.com/contest/1000/problem/F)


### Có cập nhật

#### Ví dụ

[940F Codeforces](https://codeforces.com/contest/940/problem/F)

??? question "Đề bài"

    Xử lí 2 loại truy vấn:

    - Gán $a_i=x$
    - Cho 2 số $l,r$. Gọi $cnt_i$ là số lượng số có giá trị bằng $i$ trong đoạn $[l,r]$. Tìm $mex(cnt_0, \cdots, cnt_{10^9})$

??? success "Lời giải"

    Ta sẽ tách riêng hai loại truy vấn gán và hỏi. Với mỗi truy vấn hỏi, ta biểu diễn nó dưới dạng $(t,l,r)$ với $t$ là số lượng truy vấn gán trước nó. Ngược lại, ta biểu diễn dưới dạng $(i,x)$ - như đề bài đã cho.

    Đặt $P=n^{\frac{2}{3}}$. Lúc này, ta có thể chia mảng $a$ thành $\frac{n}{P}=n^{\frac{1}{3}}$ đoạn. Vậy mỗi cặp $(l,r)$ có thể thuộc tối đa $P$ cặp đoạn (do mỗi biến $l$ và $r$ có thể thuộc $\frac{n}{P}$ đoạn riêng biệt)

    Ta xét tất cả các cặp khối $(b_l,b_r)$ có thể xảy ra. Với mỗi cặp khối:
		
    - Ta duyệt các bộ $(t,l,r)$ với $\frac{l}{P}=b_l$ và $\frac{r}{P}=b_r$ theo thứ tự tăng dần của $t$. 
    - Cũng như thuật toán Mo, ta duy trì hai biến $cur_l,cur_r$ thể hiện hai biên hiện tại tính đến truy vấn đang xét. Với mỗi truy vấn xét đến, ta tiến hành tăng giảm $cur_l,cur_r$ và cập nhật $a_{cur_l},a_{cur_r}$ vào cấu trúc dữ liệu, tương tự như khi thực hiện thuật toán Mo thông thường. 
    - Tuy nhiên, ta vẫn chưa xét các thao tác update. Ta lưu thêm một biến $cur_t$ thể hiện xem ta đã duyệt đến truy vấn update thứ bao nhiêu. Trong khi $cur_t<t$, ta tiến hành cập nhật vào mảng $a$ nghĩa là gán $a_i=x$, và thay đổi cấu trúc dữ liệu nếu $cur_l \leq i \leq cur_r$

    Vì $q \approx n$, nên khi đánh giá độ phức tạp, ta coi như $q=n$.

    Ta có $(\frac{n}{P})^2=n^{\frac{2}{3}}$ cặp $(b_l,b_r)$ cần xét. Với các biên $(cur_l,cur_r)$, ta chỉ di chuyền chúng trong khối $b_l$ và $b_r$, vậy số bước di chuyển tối đa là $n^{\frac{2}{3}}$, và ta có $n$ truy vấn, nên số bước di chuyển là $n \times P=n^{\frac{5}{3}}$. Biến $t$ thay đổi tối đa $n$ lần với mỗi cặp $b$, nên có $n \times (\frac{n}{P})^2 = n^{\frac{5}{3}}$ lần duyệt $t$. Vậy độ phức tạp tổng có thể xem là $O(n^{\frac{5}{3}})$

    Đến đây việc còn lại chỉ là tìm mex. Bằng việc sử dụng thủ thuật được nêu ở phần trên, ta có thể tìm mex trong $O(\sqrt{n})$, do đáp án tối đa là $n$. Tuy nhiên, ta cũng có thể rút ra một cận chặt hơn là đáp án của chúng ta $\leq 2\sqrt{n}$. Do để mex bằng $c$ thì ít nhất ta phải có ít nhất một số xuất hiện $1$ lần, một số xuất hiện $2$ lần, $\cdots$, một số xuất hiện $c-1$ lần. Vậy ta cần có ít nhất $1+2+\cdots+c-1=\frac{c(c-1)}{2}$ số trong đoạn đang xét. Mà đoạn dài nhất tối đa là $n$ $\rightarrow \frac{c(c-1)}{2} \leq n \rightarrow c \leq 2\sqrt{n}$. Như vậy muốn tìm mex ta chỉ cần duyệt trâu là AC, vì tối đa chỉ có $q$ truy vấn.

??? abstract "Cài đặt"

    [https://codeforces.com/contest/940/submission/122177453](https://codeforces.com/contest/940/submission/122177453)

#### Bài tập áp dụng

- [Primitive Queries](https://www.codechef.com/FEB17/problems/DISTNUM3)
- [Candy Park](http://uoj.ac/problem/58?locale=en)

## Xâu

### Giới thiệu

Cho dãy $a$ có $n$ phần tử nguyên không âm và $a_1+a_2+\cdots+a_n=m$. Nhận xét: số lượng phần tử phân biệt của dãy $a$ $\leq \sqrt{m}$ 

??? success "Chứng minh"

    - Gọi $b_0,\cdots,b_{k-1}$ là dãy $a$ sau khi được sort unique
    - Vì $b_i$ nguyên không âm $\Rightarrow b_i \geq i$ $\Rightarrow \sum_{i=0}^{k-1}b_i \geq \sum_{i=0}^{k-1}i=\frac{k(k-1)}{2}$
    - Lại có $b_0+\cdots+b_{k-1} \leq m \Rightarrow \frac{k(k-1)}{2} \leq m \Rightarrow k \leq \sqrt{m}$

### Ví dụ
[Olympic Sinh Viên 2022 - Siêu cúp - Tìm xâu đẹp](https://oj.vnoi.info/problem/olp_sc22_bstr)

??? question "Đề bài"

    Xử lí các truy vấn

    - Thêm xâu $S$ vào tập
    - Xoá xâu $S$ khỏi tập
    - Đếm số xâu trong tập chứa $S$ như một xâu con

    Đặt $T =$ tổng độ dài các xâu $S$. Ta có $T \leq 5 \times 10^5$ 

??? success "Lời giải"

    Bằng một số thuật toán Automation hoặc chia hai cách làm ta cũng có thể giải được bài này. Tuy nhiên từ nhận xét trên ta có thể rút ra thuật toán sau đây:

    - Chỉ có $\sqrt{T}$ độ dài phân biệt của $S$. Lưu các độ dài này vào một mảng $L$
    - Với mỗi truy vấn thêm hoặc xoá xâu $S$, ta duyệt mọi độ dài $len$ trong $L$, và sử dụng một cấu trúc dữ liệu để ghi nhận số lần xuất hiện của mọi xâu con phân biệt độ dài $len$ của $S$ 
    - Với mỗi truy vấn hỏi, ta truy xuất trong cấu trúc dữ liệu để ra được đáp án
    - Ta có thể dùng Hash kết hợp với chặt nhị phân để làm việc này. Cụ thể, ta lưu mã Hash của toàn bộ truy vấn xâu $S$ vào một mảng $H$. Sau đó, ta sắp xếp mảng $H$ này lại. Khi thêm/bớt một xâu, ta chặt nhị phân mã hash của nó trên $H$. Nếu tồn tại, giả sử vị trí của nó trên $H$ là $P$, ta sử dụng mảng đếm $cnt$ và tiến hành truy xuất $cnt_P$.
    - Ngoài cách sử dụng Hash, ta cũng có thể sử dụng cấu trúc dữ liệu Trie để lưu trữ các xâu.

    Độ phức tạp sẽ là $O(Q\sqrt{T})$. Dù độ phức tạp rất lớn nhưng trên thực tế không có nhiều độ dài đến vậy, nên vẫn có thể AC.

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>

	using namespace std;

	const int N = 5e5 + 5;
	const int BASE = 311;

	struct Query {
	  string type, s;
	  Query(string t = "", string s = "") : type(t), s(s) {}
	};

	template <typename T>
	void sort_and_unique(vector<T>& v) {
	  sort(v.begin(), v.end());
	  v.resize(unique(v.begin(), v.end()) - v.begin());
	}

	int64_t power[N];
	int64_t cur_hash[N];
	int cnt[N];
	int vis[N];

	int64_t get_hash(int i, int j) {
	  return cur_hash[j] - cur_hash[i - 1] * power[j - i + 1];
	}

	int32_t main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);
	  power[0] = 1;
	  for (int i = 1; i < N; i++) {
	    power[i] = power[i - 1] * BASE;
	  }
	  vector<Query> qr;
	  int n;
	  cin >> n;
	  for (int i = 1; i <= n; i++) {
	    string s;
	    cin >> s;
	    qr.push_back({"i", s});
	  }
	  int q;
	  cin >> q;
	  vector<int> len_query;
	  vector<int64_t> hash_query;
	  for (int i = 1; i <= q; i++) {
	    string ask, s;
	    cin >> ask >> s;
	    qr.push_back({ask, s});
	    if (ask == "q") {
	      len_query.push_back(s.size());
	      hash_query.push_back(0);
	      for (char c : s) {
	        hash_query.back() = hash_query.back() * BASE + c;
	      }
	    }
	  }
	  sort_and_unique(len_query);
	  sort_and_unique(hash_query);
	  int timer = 0;
	  auto add = [&](string s, int delta) {
	    for (int i = 1; i <= int(s.size()); i++) {
	      cur_hash[i] = cur_hash[i - 1] * BASE + s[i - 1];
	    }
	    for (int len : len_query) {
	      ++timer;
	      for (int i = 1; i + len - 1 <= int(s.size()); i++) {
	        int64_t cur_val = get_hash(i, i + len - 1);
	        int pos = lower_bound(hash_query.begin(), hash_query.end(), cur_val) - hash_query.begin();
	        if (pos != int(hash_query.size()) && hash_query[pos] == cur_val && vis[pos] != timer) {
	          vis[pos] = timer;
	          cnt[pos] += delta;
	        }
	      }
	    }
	  };
	  for (auto [type, s] : qr) {
	    if (type == "q") {
	      int64_t val = 0;
	      for (char c : s) {
	        val = val * BASE + c;
	      }
	      cout << cnt[lower_bound(hash_query.begin(), hash_query.end(), val) - hash_query.begin()] << "\n";
	    } else if (type == "i") {
	      add(s, 1);
	    } else {
	      add(s, -1);
	    }
	  }
	  return 0;
	}
	```

[Bedao R16 - Domino](https://oj.vnoi.info/problem/bedao_r16_d)

??? question "Đề bài"

    Cho $n$ domino có mặt trên là $a_i$, mặt dưới là $b_i$. Tìm cách flip một số domino sao cho chênh lệch giữa tổng mặt trên và tổng mặt dưới là nhỏ nhất có thể. $n \leq 10^5; a_i, b_i \leq 6$

??? success "Lời giải"

    Nhận xét mỗi domino ta chỉ tác động tối đa 1 lần. Và khi biết được tổng mặt trên, ta dễ dàng suy ra được tổng mặt dưới bằng cách lấy tổng $a_i+b_i$ rồi trừ đi.

    Đến đây ta có hướng đi dp knapsack: Gọi $F(i,s)=0/1$ khi xét đến vị trí $i$ và ta tạo được tổng $s$. Dễ thấy công thức sẽ là $F(i,s)=F(i-1,s-a_i)||F(i-1,s-b_i)$. Tuy nhiên làm như vậy độ phức tạp là $O(n^2)$ do $max(s)=n \times 6$

    Nhìn lại bài toán. Giả sử ban đầu ta có $a_i \leq b_i \ \forall i$ (nếu không có điều kiện này thì ta có thể flip luôn từ khi đọc input). Ban đầu chênh lệch của ta là $|\sum_{i=1}^{n} a_i-b_i|$. Khi tác động lên domino thứ $i$, ta được cộng vào đáp án một lượng là $b_i-a_i$. Vậy ta có thể coi bài toán như là có $n$ vật giá trị $b_i-a_i$, ta cần quy hoạch động tìm một dãy có tổng xác định để chênh lệch đạt giá trị nhỏ nhất.

    Vì $b_i-a_i \leq 6$. Ta có thể duy trì mảng $cnt_i$ là số vật có giá trị $b_i-a_i$. Sau đó ta có thể quy hoạch động $F(i,s)$ là xét đến vật có giá trị $i$ và đạt được tổng $s$. Dễ thu nhận được $F(i,s)=F(i-1,s-c \times i) \ \forall c \in [0,cnt_i]$. Tuy nhiên làm như vậy độ phức tạp vẫn là $O(n^2)$

    Ta sẽ dùng một trick như sau: 

    - Giả sử có các vật với giá trị $i$ và số lượng $cnt_i$
    - Gọi $k$ là số lớn nhất thoả mãn: $2^0+2^1+⋯+2^k=2^{k+1}-1 \leq cnt_i (1)$. Dễ thấy $k \leq \log_2(cnt_i)$
    - Khi đó ta chia thành các vật nhỏ hơn với giá trị là $2^0,\cdots,2^k$; thêm một vật nữa nếu như $cnt_i-2^{k+1}+1>0$. Ta sẽ chứng minh bằng cách chọn một tập con của các vật mới tách ra này, ta luôn có thể tạo ra các số nguyên thuộc $[0,cnt_i]$
    - Bằng cách chọn một tập con của $[0,k]$, ta có thể tạo được các số thuộc $[0,2^{k+1})$ 
    - Còn thừa lại $x=cnt_i-2^{k+1}+1$, ta cần chứng minh rằng có thể tạo được tập $[2^{k+1},cnt_i]$
    - Dễ thấy với các số thuộc $[0,2^{k+1})$ , khi ta cộng thêm $x$, sẽ tạo được các số thuộc $[x,x+2^{k+1}-1]$
    - Vì $k$ là số lớn nhất thoả mãn $(1)$ nên $x \leq 2^{k+1}-1$
    - $x \leq 2^{k+1}-1 \leq cnt_i = x+2^{k+1}-1→[2^{k+1},cnt_i ] \subset [x,x+2^{k+1}-1]$
    - Như vậy ta có thể tạo được các số thuộc $[0,cnt_i]$
        
    Bằng nhận xét ở phần giới thiệu, ta cũng có thể rút ra được số lượng phần tử phân biệt trong dãy mới là $\leq \sqrt{6 \times n}$. Việc quy hoạch động trên dãy mới cho ta thuật toán với độ phức tạp là $O(n\sqrt{n})$. Phần truy vết xin dành cho bạn đọc.

??? abstract "Cài đặt"

	```cpp linenums="1"

	#include <bits/stdc++.h>

	using namespace std;

	const int MAX_N = 1e5 + 5;
	const int MAX_TYPE = 15;

	int n;
	int A[MAX_N];
	int B[MAX_N];
	int a[MAX_N];
	int cnt[MAX_N];
	int F[6 * MAX_N];
	vector<int> bucket[MAX_TYPE];
	int ansCnt[MAX_N];
	bool flipped[MAX_N];

	int main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);
	#define task "a"
	  if (fopen(task ".inp", "r")) {
	    freopen(task ".inp", "r", stdin);
	    freopen(task ".out", "w", stdout);
	  }
	  cin >> n;
	  for (int i = 1; i <= n; i++) {
	    cin >> A[i];
	  }
	  for (int i = 1; i <= n; i++) {
	    cin >> B[i];
	  }
	  for (int i = 1; i <= n; i++) {
	    if (A[i] > B[i]) {
	      flipped[i] = true;
	      swap(A[i], B[i]);
	    }
	  }
	  for (int i = 1; i <= n; i++) {
	    a[i] = abs(A[i] - B[i]);
	    bucket[a[i]].emplace_back(i);
	  }
	  vector<pair<int, int>> items;
	  for (int i = 1; i <= n; i++) ++cnt[a[i]];
	  for (int i = 0; i <= 6; i++) {
	    if (!cnt[i]) continue;
	    for (int l = 0; cnt[i] - (1 << l) >= 0; l++) {
	      cnt[i] -= 1 << l;
	      items.emplace_back((1 << l) * i, i);
	    }
	    if (cnt[i]) items.emplace_back(cnt[i] * i, i);
	  }
	  memset(F, 0x3f, sizeof(F));
	  F[0] = 0;
	  int maxSum = 6 * n;
	  int N = items.size();
	  for (int i = 1; i <= maxSum; i++) {
	    for (int j = 1; j <= N; j++) {
	      if (items[j - 1].first <= i && F[i - items[j - 1].first] < j) {
	        F[i] = j;
	        break;
	      }
	    }
	  }
	  auto trace = [&](int sum) {
	    while (sum > 0) {
	      int i = F[sum] - 1;
	      ansCnt[items[i].second] += items[i].first / items[i].second;
	      sum -= items[i].first;
	    }
	  };
	  int ansSum = -1;
	  int best = 2e9;
	  int S1 = 0, S = 0;
	  for (int i = 1; i <= n; i++) S1 += A[i], S += A[i] + B[i];
	  for (int sum = 0; sum <= maxSum; sum++) {
	    if (F[sum] != 0x3f3f3f3f) {
	      int cur = abs(2 * S1 - S + 2 * sum);
	      if (best > cur) {
	        best = cur, ansSum = sum;
	      }
	    }
	  }
	  assert(ansSum != -1);
	  trace(ansSum);
	  cout << best << "\n";
	  for (int i = 0; i <= 12; i++) {
	    while (ansCnt[i]--) {
	      flipped[bucket[i].back()] ^= 1;
	      bucket[i].pop_back();
	    }
	  }
	  cout << count(flipped + 1, flipped + n + 1, 1) << " ";
	  for (int i = 1; i <= n; i++)
	    if (flipped[i]) cout << i << " ";
	  return 0;
	}
	```

### Bài tập áp dụng
- [Strongest Friendship Group - USACO Gold 2022](http://www.usaco.org/index.php?page=viewproblem2&cpid=1259&lang=en)


## Baby step giant step

### Giới thiệu

Xét phương trình: $a^x \equiv b \ (\mod m)$, với giả thiết $\gcd(a,m)=1$. Ta cần tìm một nghiệm $x$ thoả mãn phương trình trên.

- Nhận xét: sau tối đa $m$ bước thì sẽ gặp lại một giá trị $a^i$ đã thăm trước đó. Vì theo nguyên lí Dirichlet, nếu ta có $m$ số giá trị $<m$, thì ít nhất có 2 số có giá trị bằng nhau. Vì vậy thử các giá trị $x>m$ là vô nghĩa.
![image](https://i.imgur.com/WaHGRvq.png)
- Vậy cách giải trâu ở đây sẽ là duyệt $x$ từ $0$ đến $m$, và kiểm tra $a^x$ có thoả mãn điều kiện đề bài không. Nhưng như vậy không đủ để giải với $m$ lớn.
- Đặt $x=Tp+q$, với $T$ là một hằng số xác định. Cách chọn $T$ sẽ được trình bày sau. Ở đây $p$ được gọi là "giant step", còn $q$ được gọi là "baby step"
- Khi đó phương trình trở thành $a^{Tp+q} \equiv b (\mod m)$
- Phương trình tương đương $a^{Tp} \equiv a^{-q} \times b (\mod m)$
- Ta có thể tính $a^{Tp}$ với mọi giá trị $p \in [0,\lfloor \frac{m}{T}\rfloor]$ và $a^{-q} \times b$ với mọi giá trị $q<T$. Sau đó, ta có thể sử dụng chặt nhị phân để đếm số cặp giá trị bằng nhau trên hai mảng trên.

Độ phức tạp phụ thuộc vào $\frac{m}{T} + T$. Như đã đề cập ở các phần trước, biểu thức này đạt giá trị nhỏ nhất khi $T=\sqrt{m}$.

Thuật toán trên không chỉ giới hạn ở mức giải phương trình, mà ta có thể áp dụng cho bất cứ cyclic group độ dài $n$ nào mà tồn tại inverse của chúng.

### Ví dụ

[1840G1 - Codeforces](https://codeforces.com/contest/1840/problem/G1)

??? success "Lời giải"

    - Để tìm $n$ cách đơn giản nhất là duyệt $10^6$ lần từ vị trí xuất phát được cho ban đầu đến khi gặp lại số đã đi qua. Độ dài chu trình đó chính là $n$. Tuy nhiên làm vậy sẽ bị quá số câu hỏi.
    - Chọn một số $K$ bất kì. Hỏi $K$ câu hỏi $+1$ để tìm $K$ số đầu tiên. Nếu $N<K$, sẽ có 2 số trùng nhau trong $K$ số đầu tiên được thăm. Vậy $N$ sẽ là khoảng cách giữa 2 số trùng nhau đó.
    - Ngược lại, $N \geq K$. Ta chia vòng tròn làm $\frac{N}{K}$ cung nhỏ. Bắt đầu từ vị trí hiện tại, ta nhảy $K$ bước một qua từng cung, nghĩa là hỏi các câu hỏi $+K$ để tìm số ở vị trí $2K,3K,...$. Nếu như quay lại cung chứa số đã được đánh dấu $\Rightarrow$ số đó đã được đánh dấu ở $K$ bước đầu $\Rightarrow$ tìm lại được $N$.
    - Độ phức tạp: $O(K+\frac{N}{K})$. Để tối ưu số câu hỏi ta cần tìm $K$ sao cho $K+\frac{N}{K}$ min. Theo Cosi ta có $K+\frac{N}{K} \geq 2 \times \sqrt{K \times \frac{N}{K}} = 2 \sqrt {N}$. Dấu bằng xảy ra khi $K=\frac{N}{K} \Leftrightarrow K = \sqrt{N}$. Vậy số câu hỏi cần dùng tối đa là $2000$ câu.

??? abstract "Cài đặt"
    ```cpp linenums="1"
	#include <bits/stdc++.h>

	using namespace std;

	int marked[1000005];

	int main() {
	  memset(marked, -1, sizeof(marked));
	  int x;
	  cin >> x;
	  marked[x] = 0;
	  for (int i = 1; i <= 1000; i++) {
	    cout << "+ 1" << endl;
	    int p;
	    cin >> p;
	    if (marked[p] != -1) {
	      cout << "! " << i - marked[p] << endl;
	      return 0;
	    }
	    marked[p] = i;
	  }
	  for (int i = 2000; i <= 1001000; i += 1000) {
	    cout << "+ 1000" << endl;
	    int p;
	    cin >> p;
	    if (marked[p] != -1) {
	      cout << "! " << abs(i - marked[p]) << endl;
	      return 0;
	    }
	  }
	  assert(false);
	  return 0;
	}
	```

### Bài tập áp dụng

- [SPOJ - MOD](https://www.spoj.com/problems/MOD/)
- [TopCoder - SplitingFoxes3](https://community.topcoder.com/stat?c=problem_statement&pm=14386&rd=16801)
- [Codechef - INVXOR](https://www.codechef.com/problems/INVXOR/)
- [Codeforces - Hard equation](https://codeforces.com/gym/101853/problem/G) - bài toán yêu cầu giải $a^x \equiv b$ nhưng không với điều kiện $gcd(a,m)=1$. Gợi ý: ta có thể đổi biến $a'=ga,b'=gb,m'=gm$ với $g=gcd(a,m)$.
- [Codechef - CHEFMOD](https://www.codechef.com/problems/CHEFMOD)

## Tài liệu tham khảo

- Toàn bộ các hình ảnh trong bài viết: [imgur](https://imgur.com/a/IbeUDPM)
- [MIPT Spring Camp 2016](http://acm.math.spbu.ru/~sk1/mm/lections/mipt2016-sqrt/mipt-2016-burunduk1-sqrt.en.pdf)
- [Discrete log - CP-Algorithm](https://cp-algorithms.com/algebra/discrete-log.html)
- [Chia căn - VNOI Wiki](https://vnoi.info/wiki/algo/data-structures/sqrt-decomposition.md)
- [Baby step giant step - Wikiwand](https://www.wikiwand.com/en/Baby-step_giant-step)
- [Collection of little techniques - Codeforces](https://codeforces.com/blog/entry/100910)
- [An alternative sorting order for Mo's algorithm - Codeforces](https://codeforces.com/blog/entry/61203)
- [Sqrt decomposition - Errichto](https://codeforces.com/blog/entry/96713)
- [SQRT Decomposition Problems - Codeforces](https://mamnoonsiam.github.io/dump/sqrt-decomp.html)
- [USACO - Sqrt Decomposition](https://usaco.guide/plat/sqrt?lang=cpp#set-a)
- [Mo's Algorithm on Trees [Tutorial] - Codeforces](https://codeforces.com/blog/entry/43230)
- [Mo's Algorithm (with update and without update, now you can understand both) - Codeforces](https://codeforces.com/blog/entry/72690)
- [ei1333's blog](https://ei1333.hateblo.jp/entry/2017/09/11/211011)
- [kazuma8128's blog](https://kazuma8128.hatenablog.com/entry/2018/03/16/225926)
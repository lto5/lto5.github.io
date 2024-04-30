# Quy hoạch động chữ số

Đây là một kĩ thuật tương đối phổ biến, mới được du nhập vào Việt Nam trong những năm gần đây. Tuy nhiên những tài liệu viết về kĩ thuật này bằng tiếng Việt còn khá ít. Vậy nên trong bài viết này mình sẽ cố gắng trình bày đầy đủ về dp digit cơ bản cũng như một số kĩ thuật tối ưu. 

## Định nghĩa

Bài toán đưa ra thường sẽ là: đếm số lượng các số nguyên có giá trị trên một miền $[L,R]$ xác định và thoả mãn một tính chất nào đó.

Nếu giới hạn $R-L$ nhỏ, ta có thể duyệt như sau:

???+ note "Pseudo Code" 
    ```
    for i = L -> R:
        if (ok(i)): ans++
    print(ans)
    ```
   
    Trong đó hàm ``ok(i)`` là hàm xác định xem $i$ có thoả mãn tính chất đề bài không. Độ phức tạp của cách làm này là $O((R-L+1) \times ...)$, có thể không phù hợp với các bài toán giới hạn lớn hơn.

Ta sẽ nhìn bài toán theo một hướng khác. Gọi $G(x)$ là số lượng các số nguyên như vậy trong phạm vi từ $0$ đến $x$, thì đáp án của bải toán là $G(R)-G(L-1)$. Như vậy, bài toán quy về việc tính hàm $G(x)$.

Thay vì duyệt từng số một để kiểm tra, ta sẽ đi xây dựng từng chữ số một của một số thoả mãn. Giả sử số $X$ có dạng $a_{n-1} a_{n-2} ... a_{0}$ với $n$ là số chữ số tối đa.

- Gọi số hiện đang xây dựng là số $Y=a'_{n-1}...a'_0$
- Giả sử chúng ta xây dựng được chữ số $a'_{n-1} a'_{n-2}... a'_{i+1}$. Bây giờ chúng ta cần xây dựng chữ số $a'_{i}$
- Điều kiện của chúng ta là $Y \leq X$. Nếu trước $i$ tồn tại một chỉ số $j$ mà $a'_j<a_j$, thì vị trí thứ $i$ ta có thể điền một chữ số bất kì trong khoảng cho phép, vì ta đã chắc chắn rằng $Y<X$. Ngược lại, ta chỉ có thể điền các chữ số $\leq a_i$ vào $a'_i$, vì ta có $a'_{n-1} ... a'_{i+1} = a_{n-1}...a_{i+1}$. Nếu ta điền một chữ số $>a_i$ vào vị trí thứ $i$, thì có thể khẳng định $Y>X$, không thoả mãn với yêu cầu hiện giờ.

Ta sẽ có một hàm đệ quy: $dp(i, ...)$ là hàm quay lui để thử các khả năng của chữ số thứ $i$. Với mỗi giá trị có thể của $a'_{i}$, ta sẽ gọi đệ quy đến $dp(i-1,...)$. Bằng cách gọi $dp(n-1,...)$, ta sẽ sinh ra các số $k$ trong phạm vi từ $0$ đến $x$. Với mỗi số sinh ra, ta kiểm tra nó có thoả mãn tính chất đề bài yêu cầu hay không, nếu có thì tăng kết quả thêm $1$. 

Tuy nhiên nếu làm như này thì không khác gì cách duyệt như trên? Tương tự quy hoạch động, nếu gặp một trạng thái $dp(i...)$ đã được tính rồi, ta không tính lại nữa mà sẽ truy xuất đến kết quả đã được tính trước đó. Đây còn được gọi là kĩ thuật đệ quy có nhớ.

Như vậy ta có một mẫu chung cho các bài toán dạng này như sau:

???+ note "Pseudo Code" 
    ```
    // hàm quy hoạch động xây dựng đến chữ số thứ i, biến smaller = true/false dùng để kiểm tra xem: số đang xây dựng đã chắc chắn nhỏ hơn cận trên chưa, cond là các điều kiện để xác định xem một số có thoả mãn hay không.
    dp(i, smaller, cond):
        if i < 0:
            return 1 nếu điều kiện thoả mãn, 0 nếu ngược lại
        if (f[i, smaller, cond] != -1): // đã được tính
            return f[i, smaller, cond]
        ans=0
        limit = smaller ? 9 : a[i]
        for d = 0 -> limit:
            // new_cond là điều kiện mới sau khi điền chữ số d vào vị trí thứ i
            ans += dp(i - 1, smaller \|\| (d < a[i]), new_cond)
        return f[i, smaller, cond] = ans
    
    fill(all(f), -1);
    dp(n - 1, 0, ...); 
    ```

Tuy nhiên không chỉ có việc đếm một số $X$, mà việc đếm một bộ số $\{X_1...X_{N}\}$ cũng có thể được thực hiện bằng quy hoạch động chữ số. Chi tiết chúng ta sẽ đi qua phần ví dụ.

## Ví dụ

### [SPOJ - GONE](https://www.spoj.com/problems/GONE/)

??? question "Đề bài"

    Có bao nhiêu số từ $A$ đến $B$ mà tổng các chữ số của nó là số nguyên tố?

??? success "Lời giải"

    - Gọi $F(i,smaller,sum)$ là số cấu hình khi xét đến vị trí $i$, số hiện tại đã nhỏ hơn cận trên chưa, và tổng các chữ số của số đang xét hiện tại là bao nhiêu?
    - Nếu $i < 0$ ta trả về $1$ nếu $sum$ là số nguyên tố
    - Ngược lại, ta duyệt mọi ứng cử viên của chữ số thứ $i$, và cập nhật biến $smaller,sum$ tương ứng
    - Với giới hạn $A,B \leq 10^8$, cùng $100$ test, ta có độ phức tạp là $O(T \times \|B\| \times \|B\| \times 9 \times 10) = O(T \times 8^2 \times 9 \times 10)$ (số trạng thái nhân với chi phí chuyển trạng thái)

??? abstract "Cài đặt"

    ```cpp linenums="1"
    #include <bits/stdc++.h>
    using namespace std;

    const int MAX_DIGITS = 10;
    const int MAX_SUM = 9 * MAX_DIGITS;

    int num[MAX_DIGITS];
    int F[MAX_DIGITS][2][MAX_SUM];
    bool p[MAX_SUM];

    int dp(int i, bool f, int cur_sum) {
        if (i < 0)
            return !p[cur_sum];

        if (F[i][f][cur_sum] != -1)
            return F[i][f][cur_sum];

        int res = 0;
        int max_digit = (f ? num[i] : 9);

        for (int c = 0; c <= max_digit; c++) {
            bool new_f = (f == true && c == max_digit);
            res += dp(i - 1, new_f, cur_sum + c);
        }

        return F[i][f][cur_sum] = res;
    }

    int solve(int x) {
        memset(F, -1, sizeof(F));

        int n = 0;
        do {
            num[n++] = x % 10;
            x /= 10;
        } while (x > 0);

        return dp(n - 1, true, 0);
    }

    int main() {
        ios_base::sync_with_stdio(0);
        cin.tie(0);
        cout.tie(0);

        p[0] = p[1] = true;
        for (int i = 2; i * i < MAX_SUM; i++)
            if (!p[i])
            for (int j = i * i; j < MAX_SUM; j += i)
                p[j] = true;

        int tc;
        cin >> tc;
        while (tc--) {
            int a, b;
            cin >> a >> b;
            cout << solve(b) - solve(a - 1) << "\n";
        }
    
        return 0;
    }
    ```

### [GIRLNUM - Girl Numbers](http://lequydon.ntucoder.net/Problem/Details/5841)

??? question "Đề bài"

    Đếm số số $X$ thoả mãn $X_1<X_2>X_3<X_4...$ hoặc $X_1>X_2<X_3>X_4...$

??? success "Lời giải"

    - Khi điền chữ số thứ $i$, ta cần biết thêm thông tin là vị trí ngay trước nó đã được điền chữ số nào, cũng như "dấu" hiện tại là cái gì (nghĩa là vị trí này ta đang cần điền chữ số nhỏ hơn, hay lớn hơn chữ số trước đó)
    - Với bài trên, ta có thể điền các chữ số 0 vào đầu thoải mái, mà không làm ảnh hưởng đến kết quả tính được. Tuy nhiên, ở bài này ta cần quản lí việc đó. Bởi vì khi ta chưa bắt đầu một số (nghĩa là prefix vẫn là $00...0$), thì ta chưa cần quan tâm đến chữ số đằng trước hay dấu như thế nào, mà có thể điền bất kì
    - Vậy hàm quy hoạch động của chúng ta sẽ là $F(i,smaller,zero,last,sign)$, với ý nghĩa:
        - Ta đang điền đến vị trí thứ $i$
        - $smaller=0/1$: Số hiện tại đã nhỏ hơn cận trên chưa?
        - $zero=0/1$: Ta đã "chạm" đến số hiện tại đang xét chưa, hay là vẫn đang nằm ở prefix $000..000xxx$?
        - $last=0...9$: Chữ số được điền ngay trước nó, tại thời điểm đạt được trạng thái $zero=0$
        - $sign=0/1$: Hiện tại ta đang cần điền chữ số nhỏ hơn $(1)$, hay lớn hơn $(0)$ chữ số đứng trước nó
    - Với mỗi trạng thái $(i,smaller,zero,last,sign)$ và hiện tại đang điền chữ số $j$. Ta chuyển sang trạng thái $(i-1,next\_smaller,next\_zero, next\_last,next\_sign)$ như sau
        - Nếu chữ số $j$ không điền được vào vị trí $i$ thì không xét $(smaller=true$ nhưng $j>$ chữ số thứ $i$ của cận trên, $sign=1$ nhưng $j>last$ hoặc $sign=0$ nhưng $j<last)$
        - Nếu $j=0$, $next\_zero=true$ nếu $zero=true,i \geq 1$. Ngược lại $next\_zero=false$ $(j>0$ hoặc các điều kiện trên không thoả mãn$)$
        - Nếu $next\_zero=true$ thì $next\_last=last,next\_sign=sign$. Ngược lại $next\_last=j,next\_sign=2-sign$
        - $next\_smaller=smaller$ $OR$ $j<$ chữ số thứ $i$ của cận trên

    Đáp án sẽ là $dp(n - 1, false, -1, true, true) + dp(n - 1, false, 10, true, false)$ với $n$ là độ dài số đang xét (ta đánh số từ $n-1$ về $0$). Tuy nhiên nếu như làm như vậy sẽ bị đếm thừa các số có $1$ chữ số bởi vì các số này thoả mãn cả 2 trường hợp xuất phát từ $sign=0/1$. Vậy ta cần trừ đáp án cho (số lượng số có $1$ chữ số và nhỏ hơn hoặc bằng cận trên).

    Tuy nhiên với giới hạn $A \leq 10^{10^5}$ ta không thể tính $A-1$ như bình thường. Đến đây ta có hai cách: duyệt từng chữ số của $A$ để tính $A-1$, hoặc tính $G(B)-G(A)$ rồi $+1$ nếu như $A$ thoả mãn đề bài.

    Độ phức tạp: $O(|B| \times 9 \times 2^3 \times 9) =O(10^5 \times 9 \times 2^3 \times 9)$. Tuy nhiên vì có một số ràng buộc nên số trạng thái sẽ nhỏ hơn.

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>
	using namespace std;

	const int N = 1e5 + 5;
	const int D = 15;
	const int MOD = 1e9 + 7;
	typedef long long ll;

	int num[N];
	ll f[N][2][D][2][2];

	ll dp(int i, bool limit, int last, bool zero, bool inc) {
	  if (i < 0)
	    return 1;

	  if (last != -1 && last != 10 && f[i][limit][last][zero][inc] != -1)
	    return f[i][limit][last][zero][inc];

	  ll res = 0;
	  int mx = limit ? num[i] : 9;

	  //<
	  if (inc == true) {
	    for (int c = last + 1; c <= mx; c++) {
	      bool new_limit = limit && c == mx;
	      bool new_zero = i >= 1 && c == 0 && zero;

	      int new_last = (new_zero == true ? -1 : c);
	      bool new_inc = (new_zero == true ? inc : !inc);

	      res = (res + dp(i - 1, new_limit, new_last, new_zero, new_inc)) % MOD;
	    }
	  }
	  //>
	  else {
	    for (int c = 0; c <= min(last - 1, mx); c++) {
	      bool new_limit = limit && c == mx;
	      bool new_zero = i >= 1 && c == 0 && zero;

	      int new_last = (new_zero == true ? 10 : c);
	      bool new_inc = (new_zero == true ? inc : !inc);

	      res = (res + dp(i - 1, new_limit, new_last, new_zero, new_inc)) % MOD;
	    }
	  }

	  return f[i][limit][last][zero][inc] = res;
	}

	ll solve(string s) {
	  int n = s.size();

	  if (n == 1)
	    return s[0] - '0' + 1;

	  for (int i = 0; i < n; i++)
	    num[i] = s[n - i - 1] - '0';

	  memset(f, -1, sizeof(f));
	  return (dp(n - 1, true, -1, true, true) + dp(n - 1, true, 10, true, false) - 10 + MOD) % MOD;
	}

	bool check(string s) {
	  if (s.size() == 1)
	    return true;

	  bool f1 = true, f2 = true;

	  bool cur = true;  // a[i] < a[i + 1];
	  for (int i = 1; i < s.size(); i++) {
	    if (s[i - 1] == s[i])
	      return false;
	    bool st = s[i - 1] < s[i];
	    if (st != cur) {
	      f1 = false;
	      break;
	    }
	    cur = !cur;
	  }

	  cur = true;  // a[i] > a[i + 1];
	  for (int i = 1; i < s.size(); i++) {
	    bool st = s[i - 1] > s[i];
	    if (st != cur) {
	      f2 = false;
	      break;
	    }
	    cur = !cur;
	  }

	  return f1 || f2;
	}

	int main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);

	  string L, R;
	  cin >> L >> R >> L >> R;

	  cout << (solve(R) - solve(L) + check(L) + MOD) % MOD;

	  return 0;
	}
	```

### [Baltic OI '13 P2 - Palindrome-Free Numbers](https://dmoj.ca/problem/btoi13p2)

??? question "Đề bài"

    Đếm số số $X$ không chứa một xâu con độ dài $>1$ nào là palindrome.

??? success "Lời giải"

    - Nhận xét nếu bất kì một xâu palin độ dài $>1$ đều có tâm là palin độ dài $2$ hoặc $3$. Vậy ta chỉ cần đảm bảo $X$ không chứa $palin$ độ dài $2$ hoặc $3$ là sẽ thoả mãn đề bài
        - Điều kiện không tồn tại palin độ dài $2$: $X_i \neq X_{i-1}$ với mọi $i>1$
        - Điều kiện không tồn tại palin độ dài $3$: $X_i \neq X_{i-2}$ với mọi $i>2$
    - Vậy tương tự bài trên, ta cần lưu một biến $zero$ để kiểm soát xem ta đang nằm ở prefix $000..000$ hay đã chạm đến số cần xét, cũng như hai chữ số được điền ngay trước $i$ (nếu như có ít hơn 2 chữ số thì ta cứ xem chữ số nào bị tràn ra ngoài là $-1$). Lưu ý khi $zero=true$ thì ta không xét điều kiện gì mà điền bất kì.

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>
	using namespace std;

	const int N = 20;

	int a[N];
	int n;
	int64_t f[N][2][2][11][11];

	int64_t dp(int i, bool smaller, bool is_zero, int last2, int last1) {
	  if (i < 0) {
	    return 1;
	  }

	  int64_t &ans = f[i][smaller][is_zero][last2 + 1][last1 + 1];
	  if (ans != -1) {
	    return ans;
	  }

	  ans = 0;
	  int lim = smaller ? 9 : a[i];
	  for (int d = 0; d <= lim; d++) {
	    bool new_smaller = smaller || d < a[i];
	    bool new_zero = i >= 1 && is_zero && d == 0;
	    int new_last_2 = last1;
	    int new_last_1 = d;

	    if (!new_zero) {
	      if (d == last1 || d == last2)
	        continue;
	    } else {
	      new_last_2 = new_last_1 = -1;
	    }

	    ans += dp(i - 1, new_smaller, new_zero, new_last_2, new_last_1);
	  }

	  return ans;
	}

	int64_t solve(int64_t x) {
	  n = 0;
	  do {
	    a[n++] = x % 10;
	    x /= 10;
	  } while (x > 0);
	  memset(f, -1, sizeof(f));
	  return dp(n - 1, 0, 1, -1, -1);
	}

	int main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);

	  int64_t l, r;
	  cin >> l >> r;
	  cout << solve(r) - solve(l - 1);
	  return 0;
	}

	```

### [ADBRACK - VNOI](https://oj.vnoi.info/problem/adbrack)

??? success "Lời giải"

    Gọi $dp(i, smaller, deg)$ là số dãy ngoặc khi:

    - Đã xét đến vị trí $i$
    - $smaller$ thể hiện dãy ngoặc hiện tại đang xây dựng đã chắc chắn nhỏ hơn dãy ngoặc gốc hay chưa
    - $deg$ thể hiện số ngoặc mở chưa ghép được với ngoặc đóng nào trong dãy ngoặc hiện tại. Dãy ngoặc đúng có bậc không quá $k$ khi $min(deg) \geq 0$ và $max(deg) \leq k$

    Với mỗi vị trí $i$:

    - Nếu ta điền một ngoặc mở $c$ bất kì: chuyển sang trạng thái $dp(i + 1, smaller \\|\\| c < s[i], deg + 1)$
    - Nếu ta điền ngoặc đóng:
        - Nếu $smaller=true$: ta chỉ có duy nhất một cách điền, đó chính là điền một dấu ngoặc đóng "khớp" với dấu ngoặc mở gần nhất đã điền trước đó (giống thuật toán Stack). Ta chuyển sang trạng thái $dp(i + 1, smaller, deg - 1)$
        - Ngược lại, gọi $c$ là dấu ngoặc đóng "khớp" với dấu ngoặc mở gần nhất chưa được ghép với ai trước $i$ (để tìm ta dùng stack). Nếu $c \leq s_i$, ta chuyển sang trạng thái $dp(i + 1, smaller \\|\\| c < s[i], deg - 1)$
    - Lưu ý phải kiểm soát trạng thái nhỏ hơn khi điền, tương tự các bài trên

    Kết quả là kết quả của lời gọi hàm $dp(0,0,0)$. Độ phức tạp: $O(n^2 \times bignum)$

??? abstract "Cài đặt"

    ```cpp linenums="1"
	BigInt f[105][105];
	bool done[105][105];
	int n, k;
	char match[105];
	string s;
	 
	BigInt dp(int i, bool smaller, int deg) {
	  if (deg < 0 || deg > k) {
	    return 0;
	  }
	  if (i >= n) {
	    return deg == 0;
	  }
	  if (smaller && done[i][deg]) {
	    return f[i][deg];
	  }
	  BigInt ans = 0;
	  for (char c : OPEN) {
	    if (!smaller && c > s[i]) {
	      continue;
	    }
	    ans += dp(i + 1, smaller || c < s[i], deg + 1);
	  }
	  if (smaller) {
	    ans += dp(i + 1, smaller, deg - 1);
	  } else {
	    if (match[i] <= s[i]) {
	      ans += dp(i + 1, smaller || match[i] < s[i], deg - 1);
	    }
	  }
	  if (smaller) {
	    done[i][deg] = 1;
	    f[i][deg] = ans;
	  }
	  return ans;
	}
	 
	char rev(char c) {
	  if (c == '(') return ')';
	  if (c == '[') return ']';
	  return '}'; // '{'
	}
	 
	int32_t main(){
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);
	  cin >> n >> k >> s;
	  stack<int> st;
	  for (int i = 0; i < n; i++) {
	    if (!st.empty()) {
	      match[i] = rev(s[st.top()]);
	    }
	    if (OPEN.find(s[i]) != string::npos) {
	      st.push(i);
	    } else {
	      st.pop();
	    }
	  }
	  cout << dp(0, 0, 0);
	  return 0;
	}
	```

### [Bedao OI Contest 1 - Bất phương trình tuyến tính](https://oj.vnoi.info/problem/bedao_oi1_c)

??? success "Lời giải"

    Để khử điều kiện nghiệm nguyên dương ta có thể tiến hành trừ cả 2 vế của bất phương trình cho $a_1+ \cdots +a_n$ khi đó ta có bất phương trình

    $a_1 \times (x_1-1) +  \cdots  + a_n \times (x_n-1) \leq m - a_1 -  \cdots  -a_n (1)$

    Đặt $t_i=x_i-1$ khi đó bài toán chuyển về việc đếm số bộ nghiệm $(t_1 \cdots t_n)$ không âm.

    Để khử điều kiện $<$ thì ta đặt thêm một biến $t_0$ với hệ số $a_0=1$ thể hiện hiệu giữa vế phải và vế trái của $(1)$. Lúc này bài toán trở thành đếm số nghiệm nguyên không âm của phương trình

    $t_0 + a_1 \times t_1 + \cdots + a_n \times t_n = m-a_1- \cdots -a_n$

    Vì lí do $n \leq 10$ và $m \leq 10^9$ nên các số $t_0,t_1, \cdots, t_n$ cũng chỉ có khoảng $30$ bit. Vậy thay vì xây dựng từng số $t$ một thì ta sẽ xây dựng từng bit của cả $n$ số một lần.

    Từ đây ta coi $m=m-a_1- \cdots - a_n$

    Gọi $f(i,rem)$ là số bộ nghiệm thoả mãn khi:

    - xét đến bit thứ $i$
    - $rem$ thể hiện phép nhớ của vế trái hiện tại

    Với mỗi bit thứ $i$:

    - duyệt toàn bộ $mask$ từ $0$ đến $2^n -1$ với $mask_j=0/1$ thể hiện xem ở ta sẽ điền vào bit thứ $i$ số thứ $j$ là $0$ hoặc $1$
    - tổng hiện tại cho bit thứ $i$ sẽ là ($rem+$tổng các $a_j$ với $mask_j=1$) mod $2$. Nếu trùng với bit thứ $i$ của $m$ thì ta tính $rem'$ mới và chuyển sang bit thứ $i+1$

    ??? tip "Lưu ý"
        Đôi khi không phải lúc nào ta cũng đổi về hệ cơ số $2$, hãy biến đổi linh động cho từng bài toán. Và khi đề yêu cầu mối quan hệ giữa các số trong dãy (tăng/giảm dần ...), thì ta cần thêm một số chiều vào hàm quy hoạch động để cho phù hợp.

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>

	using namespace std;

	const int MOD = 1e9 + 7;

	void add(int& a, int b) {
	  if ((a += b) >= MOD) a -= MOD;
	}

	int f[31][205];
	int n, m;
	int lim;
	int a[11];
	int sum[1 << 11];

	int dp(int i, int rem) {
	  if (i > lim) return !rem;
	  int& ans = f[i][rem];
	  if (ans != -1) {
	    return ans;
	  }
	  ans = 0;
	  for (int mask = 0; mask < 1 << n; mask++) {
	    int cur = (rem + sum[mask]) & 1;
	    if (cur != (m >> i & 1)) continue;
	    add(ans, dp(i + 1, (rem + sum[mask]) >> 1));
	  }
	  return ans;
	}

	int main() {
	  freopen("nequation.inp", "r", stdin);
	  freopen("nequation.out", "w", stdout);
	  cin >> n >> m;
	  a[0] = 1;
	  for (int i = 1; i <= n; i++) {
	    cin >> a[i];
	    m -= a[i];
	  }
	  n++;
	  if (m < 0) return cout << 0, 0;
	  for (int mask = 0; mask < 1 << n; mask++) {
	    int& cur = sum[mask];
	    for (int i = 0; i < n; i++) {
	      if (mask >> i & 1) {
	        cur += a[i];
	      }
	    }
	  }
	  memset(f, -1, sizeof(f));
	  lim = __lg(m + 5);
	  cout << dp(0, 0);
	}
	```

## Các kĩ thuật tối ưu

### Có cần phải memset lại không?

Đây là một phương pháp tối ưu hoá cơ bản nhất, thường gặp với các bài multitest.

Thay vì mỗi test phải memset lại toàn bộ mảng $dp[state]$, ta lưu thêm một mảng $pos[state]$ và một biến $time$ chỉ thời gian tác động đến trạng thái $state$ cũng như thời gian hiện tại. Với mỗi test, ta tăng biến $time$ lên một đơn vị. Sau đó trong hàm dp, ta kiểm tra xem: nếu $pos[state] = time$, nghĩa là trạng thái này đã được tính rồi, thì trả về $dp[state]$, ngược lại, ta gán $pos[state] = time$ và tính $dp[state]$.

???+ note "Pseudo Code" 

    ```
    Int dp(state...) {
        if pos[state] == time:
            return dp[state]
        pos[state] = time
        calculate dp[state]...
        return dp[state]
    }
    ```

Kĩ thuật này cũng thường được sử dụng trong các bài toán khác, ví dụ như thuật toán Kuhn để tìm cặp ghép cực đại.

Áp dụng: [LIDS](https://toph.co/p/lids)

??? success "Lời giải"

    - Do $x \leq y \leq 10^9$, nên giá trị dãy con tăng dài nhất tối đa chỉ là $9$
    - Gọi $dp(i, limit, zero, last, len)$ là số số:
        - có giá trị nhỏ hơn cận trên $R$
        - số này đã chắc chắn nhỏ hơn cận trên chưa
        - số này đã chắc chắn khác $0$ chưa
        - chữ số cuối của một dãy con tăng bất kì là $last$
        - độ dài dãy con tăng đã xây dựng được là $len$
    - Với mỗi chữ số $c$ ta có $2$ lựa chọn: hoặc thêm vào dãy con tăng đang xây dựng hiện tại, hoặc không. Từ đó xây dựng cách chuyển trạng thái thích hợp. 
    - Để tránh trùng thì ta duyệt ngược độ dài dãy con tăng từ $9$ về $1$, sau đó đếm nếu có kết quả $>0$ thì in kết quả luôn.
    - Với mỗi test thay vì memset mảng $dp$ thì ta có thể tạo mảng $pos$ như hướng dẫn để pass. Dĩ nhiên bài này còn có cách tối ưu nhanh hơn, sẽ được trình bày tại phần sau.

??? abstract "Cài đặt"

    ```cpp linenums="1"
	#include<bits/stdc++.h>
	using namespace std;

	const int N = 15;
	const int MAX_D = 15;

	long long f[N][2][2][MAX_D][N];
	int vis[N][2][2][MAX_D][N], curState = 1;
	int num[N], curLen;

	long long dp(int i, bool limit, bool zero, int last, int len){
	  if(i < 0)
	    return len >= curLen;

	  if(vis[i][limit][zero][last + 1][len] == curState)
	    return f[i][limit][zero][last + 1][len];

	  vis[i][limit][zero][last + 1][len] = curState;

	  int mx = limit ? num[i] : 9;
	  long long res = 0;

	  for(int c = 0; c <= mx; c++){
	    bool newLimit = limit && c == mx;
	    bool newZero = i >= 1 && c == 0 && zero;

	    if(newZero == true){
	      res += dp(i - 1, newLimit, newZero, last, len);
	      continue;
	    }

	    res += dp(i - 1, newLimit, newZero, last, len);
	    if(c > last && newZero == false)
	      res += dp(i - 1, newLimit, newZero, c, len + 1);
	  }

	  return f[i][limit][zero][last + 1][len] = res;
	}

	void solve(){
	  int _x, _y;
	  cin >> _x >> _y;

	  for(curLen = 9; curLen >= 1; curLen--){
	    ++curState;
	    long long curSub = 0;
	    int x = _x - 1, y = _y;

	    int n = 0;
	    do{
	      num[n++] = y % 10;
	      y /= 10;
	    } while(y > 0);

	    curSub += dp(n - 1, true, true, -1, 0);
	    n = 0;

	    ++curState;
	    
	    do{
	      num[n++] = x % 10;
	      x /= 10;
	    } while(x > 0);

	    curSub -= dp(n - 1, true, true, -1, 0);

	    if(curSub > 0){
	      cout << curLen << " " << curSub;
	      return;
	    }
	  }

	  cout << "0 1";
	}

	int main(){
	  ios_base::sync_with_stdio(0);
	  cin.tie(0); cout.tie(0);

	  int t;
	  cin >> t;

	  for(int i = 1; i <= t; i++){
	    cout << "Case " << i << ": ";
	    solve();
	    cout << '\n';
	  }

	  return 0;
	}
	```

### Cần quản lý trạng thái biên không?

Trong một số bài toán quy hoạch động chữ số, ta sẽ cần phải kiểm soát xem: hiện tại số đang xây dựng đã chắc chắn nhỏ hơn một cận trên $R$ nào hay chưa (và có thể là lớn hơn một cận dưới $L$ nào đó nữa). Trong các bài toán như vậy, thông thường cách xử lý là lưu thêm một hoặc hai chiều:

???+ note "Pseudo Code" 

    ```
    // Giả sử đang xét bài toán đếm số số thoả mãn một tính chất nào đó và có giá trị nằm trong đoạn [L, R]. Biến lo và hi được dùng để kiểm soát xem: số hiện tại đang xây dựng đã chắc chắn >=L và <= R chưa.
    Int dp(int pos, int lo, int hi, ...) {
        if dp[pos, lo, hi, ...] != -1:
            return dp[pos, lo, hi...]
        calculate...
    }
    ```

Làm như vậy có các nhược điểm:

- Không tận dụng được các bài toán được tính trước ở các cận $L$ $R$ khác nhau, dẫn đến việc phải reset cũng như tính lại toàn bộ mảng $dp$, rất phí.
- Làm tăng số lượng trạng thái phải cache, gây chậm chương trình.
Vì vậy, có một giải pháp là sẽ không lưu các trạng thái dp với $lo = false$ hoặc $hi = false$. Và ta chỉ cần khởi tạo mảng $dp$ một lần trong suốt chương trình (kể cả bài có nhiều test case).

*Nhưng với nhiều test case, $L$ với $R$ khác nhau thì sao?*

??? success "Trả lời"

    Lưu ý, ta đang quy hoạch động xây dựng các cấu hình với một prefix nào đó, nên nếu prefix đó đảm bảo chắc chắn số xây dựng đã thoả mãn điều kiện các cận rồi, thì phần đằng sau có thể điền một cách tự do. Và vì vậy các phần tự do đó chỉ cần xét một lần duy nhất trong toàn bộ chương trình. Bên cạnh đó, số trạng thái có $lo=false$ hoặc $hi=false$ thì chỉ là độ dài số $L$ hoặc $R$, vì để số $x$ trùng với một tiền tố độ dài $i$ của $R$ thì tất cả các chữ số của tiền tố độ dài $i$ của $x$ phải trùng với $L$ hoặc $R$. Như vậy số prefix ta cần phải xét chỉ là độ dài của số $L$ hoặc $R$, và độ phức tạp cho phần này tương đương vói việc đọc số $L$ hoặc $R$. Đây cũng là ý tưởng của việc code dp digit đếm số cấu hình bằng vòng for.

Để hiểu rõ hơn bạn đọc có thể tham khảo code bài LIDS ở ví dụ trên sau khi đã tối ưu:

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>
	using namespace std;

	const int N = 15;
	const int MAX_D = 15;

	int64_t f[N][MAX_D][N][N];
	int num[N];

	int64_t dp(int i, bool limit, bool zero, int last, int len, int cur_len) {
	  if (i < 0)
	    return len >= cur_len;

	  if (!limit && !zero && f[i][last + 1][len][cur_len] != -1)
	    return f[i][last + 1][len][cur_len];

	  int mx = limit ? num[i] : 9;
	  int64_t res = 0;

	  for (int c = 0; c <= mx; c++) {
	    bool new_limit = limit && c == mx;
	    bool new_zero = i >= 1 && c == 0 && zero;

	    if (new_zero == true) {
	      res += dp(i - 1, new_limit, new_zero, last, len, cur_len);
	      continue;
	    }

	    res += dp(i - 1, new_limit, new_zero, last, len, cur_len);
	    if (c > last && new_zero == false)
	      res += dp(i - 1, new_limit, new_zero, c, len + 1, cur_len);
	  }

	  return !limit && !zero ? f[i][last + 1][len][cur_len] = res : res;
	}

	void solve() {
	  int _x, _y;
	  cin >> _x >> _y;

	  for (int cur_len = 9; cur_len >= 1; cur_len--) {
	    int64_t cur_sub = 0;
	    int x = _x - 1, y = _y;

	    int n = 0;
	    do {
	      num[n++] = y % 10;
	      y /= 10;
	    } while (y > 0);

	    cur_sub += dp(n - 1, true, true, -1, 0, cur_len);
	    n = 0;

	    do {
	      num[n++] = x % 10;
	      x /= 10;
	    } while (x > 0);

	    cur_sub -= dp(n - 1, true, true, -1, 0, cur_len);

	    if (cur_sub > 0) {
	      cout << cur_len << " " << cur_sub;
	      return;
	    }
	  }
	}

	int main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);

	  memset(f, -1, sizeof(f));

	  int t;
	  cin >> t;

	  for (int i = 1; i <= t; i++) {
	    solve();
	    cout << '\n';
	  }

	  return 0;
	}
	```

### Lợi dụng các tính chất số học

Ta sẽ sử dụng một số tính chất số học (ví dụ như liên quan đến phép đồng dư) để đếm số số thoả mãn một điều kiện nhất định. Để chi tiết hơn, ta cùng xét các ví dụ sau đây.

#### [Chef and special numbers](https://www.codechef.com/problems/WORKCHEF)

??? success "Lời giải"

    ??? note "Nhận xét"

        - Ta sẽ không cần kiểm tra xem chữ số $0$ có xuất hiện hay không vì không có số nào chia được cho $0$
        - Thay vì lưu mod của số hiện tại với $2 \rightarrow 9$, ta chỉ cần lưu mod của số hiện tại với $lcm(2,...,9)=2520$

    ??? tip "Thuật toán"

        Vậy trạng thái dp của chúng ta sẽ là $dp(i,s,z,mask,mod)$ khi:

        - xét đến chữ số thứ $i$
        - đã nhỏ hơn cận trên chưa
        - đã có chữ số đầu khác $0$ chưa
        - các chữ số đã xuất hiện là $mask$
        - số hiện tại mod $2520$ có giá trị là $mod$

        Do bài có nhiều test case nên ta có thể cache thêm một trạng thái là $k$ và sử dụng phần tối ưu giảm chiều ở trên để AC.

#### [VM 09 Bài 04 - Self-divisible Number](https://oj.vnoi.info/problem/selfdiv)

??? success "Lời giải"

    Đây là một bài tương tự bài trên tuy nhiên cần tối ưu rất nhiều mới có thể AC

    - Từ đề bài dễ thấy số cần tìm không được có số $0$
    - Với $n \leq 500$ ta có thể làm tương tự bài trên:
        - Gọi $dp(i,mask,mod)$ là số số thoả mãn khi xét đến vị trí $i$, $mask$ thể hiện các chữ số đã xuất hiện, và $mod$ là số dư của số hiện tại cho $lcm(2, \cdots, 9)=2520$
        - Tuy nhiên như vậy vẫn là chưa đủ. Ta nhận thấy, bằng việc xác định $3$ chữ số cuối cùng của kết quả ta có thể xác định được xem số đó có chia hết cho $2,4,8,5$ hay không. Vậy ta chỉ $dp$ đến $n-4$ cũng như lưu mask chỉ đến $2^8$ và $mod$ giảm còn lưu số dư khi chia cho $lcm(3,7,9)=63$. Còn việc xác định số dư cho $6$ có thể đưa về việc xác định chia hết cho $2,3$
        - Khi tính kết quả ta duyệt mọi $mask,mod$ có thể, trong đó duyệt mọi bộ $3$ chữ số cuối cùng và tính lại $mask,mod$ mới; sau đó kiểm tra điều kiện bài toán
        - Vậy độ phức tạp chỉ là $n \times 2^8 \times 63 + 2^8 \times 63 \times 1000$, đủ qua subtask này
    - Tuy nhiên với $n \leq 10^6$ ta không thể làm cách trên. Ta cần chỉnh sửa cách làm một chút dựa trên tư tưởng trên
        - Ta tính $f(i,mask,mod)$ là số cách điền $i$ chữ số đầu tiên, chỉ được dùng các chữ số là **tập con** của $mask$, và số dư cho $63$ là $mod$
        - Ta không thể tính $f(i,...)$ trong $O(n \times ...)$. Nhưng ta có một trick để giảm độ phức tạp, các bạn có thể tham khảo tại [blog của DeMen100ns](https://hackmd.io/@DeMen100ms/DeMenBlog3).
            - Cụ thể: với $f(i1, mask, mod)$ và $f(i2, mask', mod')$ thì ta có thể merge chúng lại thành $f(i1+i2, mask\|mask', (mod \times 10^{i2} + mod') \%63)$
            - Độ phức tạp cho việc merge 2 state $(mask,mod)$ và $(mask',mod)$ là $(2^8 \times 63)^2$, quá lâu. Vậy thay vì gom cả $mask$ vào trạng thái, ta duyệt $mask$, tính $f(n-4,mask,\cdots)$ theo cách trên
        - Đặt $F(mask,mod)=f(n-4,mask,mod)$. 
        - Để tính số cách dùng đúng các chữ số trong $mask$, không có số nào chỉ dùng các chữ số trong tập con của $mask$ ta dùng DP SOS.
        - Cuối cùng ta duyệt $3$ chữ số cuối cùng như subtask 1.

    Tuy nhiên bài này còn rất nhiều cách làm khác như việc sử dụng các thuật toán nhân đa thức hoặc có thể duyệt $lcm$ và sử dụng bao hàm loại trừ... các bạn có thể đọc ở các submissions.

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>

	using namespace std;

	template <int MOD_>
	struct modint {
	  static constexpr int MOD = MOD_;
	  static_assert(MOD_ > 0, "MOD must be positive");

	 private:
	  using ll = long long;
	  static int minv(int a, int m) {
	    a %= m;
	    assert(a);
	    return a == 1 ? 1 : int(m - ll(minv(m, a)) * ll(m) / a);
	  }

	 public:
	  int v;
	  modint() : v(0) {}
	  modint(ll v_) : v(int(v_ % MOD)) {
	    if (v < 0) v += MOD;
	  }
	  explicit operator int() const { return v; }
	  friend ostream& operator<<(ostream& out, const modint& n) { return out << int(n); }
	  friend istream& operator>>(istream& in, modint& n) {
	    ll v_;
	    in >> v_;
	    n = modint(v_);
	    return in;
	  }
	  friend bool operator==(const modint& a, const modint& b) { return a.v == b.v; }
	  friend bool operator!=(const modint& a, const modint& b) { return a.v != b.v; }
	  modint inv() const {
	    modint res;
	    res.v = minv(v, MOD);
	    return res;
	  }
	  friend modint inv(const modint& m) { return m.inv(); }
	  modint neg() const {
	    modint res;
	    res.v = v ? MOD - v : 0;
	    return res;
	  }
	  friend modint neg(const modint& m) { return m.neg(); }
	  modint operator-() const { return neg(); }
	  modint operator+() const { return modint(*this); }
	  modint& operator++() {
	    v++;
	    if (v == MOD) v = 0;
	    return *this;
	  }
	  modint& operator--() {
	    if (v == 0) v = MOD;
	    v--;
	    return *this;
	  }
	  modint& operator+=(const modint& o) {
	    v += o.v;
	    if (v >= MOD) v -= MOD;
	    return *this;
	  }
	  modint& operator-=(const modint& o) {
	    v -= o.v;
	    if (v < 0) v += MOD;
	    return *this;
	  }
	  modint& operator*=(const modint& o) {
	    v = int(ll(v) * ll(o.v) % MOD);
	    return *this;
	  }
	  modint& operator/=(const modint& o) { return *this *= o.inv(); }
	  friend modint operator++(modint& a, int) {
	    modint r = a;
	    ++a;
	    return r;
	  }
	  friend modint operator--(modint& a, int) {
	    modint r = a;
	    --a;
	    return r;
	  }
	  friend modint operator+(const modint& a, const modint& b) { return modint(a) += b; }
	  friend modint operator-(const modint& a, const modint& b) { return modint(a) -= b; }
	  friend modint operator*(const modint& a, const modint& b) { return modint(a) *= b; }
	  friend modint operator/(const modint& a, const modint& b) { return modint(a) /= b; }
	};

	void debug_out() { cerr << '\n'; }
	template <typename Head, typename... Tail>
	void debug_out(Head H, Tail... T) {
	  cerr << " " << H;
	  debug_out(T...);
	}
	#define debug(...) cerr << "[" << #__VA_ARGS__ << "]:", debug_out(__VA_ARGS__)

	const int N = 1e6 + 5;
	const int MOD = 10007;
	using mint = modint<MOD>;

	int n;

	namespace solve_small {
	void solve() {
	  int ans = 0;
	  for (int i = 1; i <= 1e5; i++) {
	    if (int(log10(i)) + 1 != n) {
	      continue;
	    }
	    bool ok = 1;
	    for (auto d : to_string(i)) {
	      d -= '0';
	      if (d == 0 || i % d) {
	        ok = 0;
	        break;
	      }
	    }
	    ans += ok;
	  }
	  cout << ans;
	}
	}  // namespace solve_small

	int pw[N];
	mint ans[63];
	mint cur[63];
	mint nex[63];
	mint ans_for_mask[1 << 10][63];

	void mul(mint a[63], mint b[63], int sz) {
	  for (int i = 0; i < 63; i++) nex[i] = 0;
	  for (int i = 0; i < 63; i++) {
	    if (a[i] == 0) continue;
	    for (int j = 0; j < 63; j++) {
	      if (b[j] == 0) {
	        continue;
	      }
	      nex[(i * pw[sz] + j) % 63] += a[i] * b[j];
	    }
	  }
	}

	int __lcm(int a, int b) {
	  return a / __gcd(a, b) * b;
	}

	void cal_dp(int mask) {
	  for (int i = 0; i < 63; i++) {
	    cur[i] = ans[i] = nex[i] = 0;
	  }
	  for (int digit = 1; digit <= 9; digit++) {
	    if (mask >> (digit - 1) & 1) {
	      ans[digit] = cur[digit] = 1;
	    } else {
	      ans[digit] = cur[digit] = 0;
	    }
	  }
	  int power = n - 4;
	  int lg = n - 4 <= 0 ? -1 : __lg(n - 4);
	  for (int bit = 0; bit <= lg; bit++) {
	    if (power >> bit & 1) {
	      mul(ans, cur, 1 << bit);
	      for (int i = 0; i < 63; i++) ans[i] = nex[i];
	    }
	    mul(cur, cur, 1 << bit);
	    for (int i = 0; i < 63; i++) cur[i] = nex[i];
	  }
	  for (int i = 0; i < 63; i++) {
	    ans_for_mask[mask][i] += ans[i];
	  }
	}

	int32_t main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);
	#define task "a"
	  if (fopen(task ".inp", "r")) {
	    freopen(task ".inp", "r", stdin);
	    freopen(task ".out", "w", stdout);
	    freopen(task ".log", "w", stderr);
	  }
	  cin >> n;
	  if (n <= 3) {
	    solve_small::solve();
	    return 0;
	  }
	  pw[0] = 1;
	  for (int i = 1; i <= n; i++) pw[i] = pw[i - 1] * 10 % 63;
	  for (int mask = 0; mask < 1 << 9; mask++) {
	    cal_dp(mask);
	  }
	  for (int mod = 0; mod < 63; mod++) {
	    for (int i = 0; i < 9; i++) {
	      for (int mask = 0; mask < 1 << 9; mask++) {
	        if (mask >> i & 1)
	          ans_for_mask[mask][mod] -= ans_for_mask[mask ^ (1 << i)][mod];
	      }
	    }
	  }
	  mint ret = 0;
	  for (int mask = 0; mask < 1 << 9; mask++) {
	    for (int mod = 0; mod < 63; mod++) {
	      for (int d1 = 1; d1 <= 9; d1++) {
	        for (int d2 = 1; d2 <= 9; d2++) {
	          for (int d3 = 1; d3 <= 9; d3++) {
	            int cur_num = d1 * 100 + d2 * 10 + d3;
	            bool ok = 1;
	            int nmask = mask | (1 << (d1 - 1)) | (1 << (d2 - 1)) | (1 << (d3 - 1));
	            for (int modolo : {2, 4, 8, 5}) {
	              if (((nmask >> (modolo - 1)) & 1) && (cur_num % modolo)) {
	                ok = false;
	                break;
	              }
	            }
	            for (int modolo : {3, 7, 9}) {
	              int big_num = (mod * pw[3] + cur_num) % 63;
	              // 6
	              if (modolo == 3) {
	                if (nmask >> 5 & 1) {
	                  if (big_num % 3 != 0 || cur_num % 2 != 0) {
	                    ok = false;
	                    break;
	                  }
	                }
	              }
	              if (((nmask >> (modolo - 1)) & 1) && (big_num % modolo)) {
	                ok = false;
	                break;
	              }
	            }
	            if (ok) {
	              ret += ans_for_mask[mask][mod];
	            }
	          }
	        }
	      }
	    }
	  }
	  cout << ret;
	  return 0;
	}
	```

### Một cách kiểm soát biên khác

Giả sử đang tính các số có giá trị $\leq R$, và đang xây dựng số $x$.

Gọi $lo$ là vị trí $i$ đầu tiên có $x_i<R_i$, $hi$ là vị trí $i$ đầu tiên có $x_i>R_i$ ($lo$ hoặc $hi$ bằng $\infty$ nếu không tồn tại). Khi đó có thể kết luận $x<R$ nếu $lo<hi$.

Tính chất này có thể được lợi dụng với các bài không thể xây dựng các chữ số theo thứ tự từ trái sang phải.

Áp dụng:

#### [LOJ 1205 - Palindromic Number](https://vjudge.net/problem/LightOJ-1205)

??? success "Lời giải"

    ??? tip "Đếm bằng toán học"
        - Bằng việc lợi dụng tính chất $a_i=a_{n-i-1}$ với $a[0:n-1]$ là số palindrome hiện tại đang xét, ta có thể duyệt để đếm số số palindrome có $1 \rightarrow n-1$ chữ số.
        - Để đếm số số palindrome có đúng $n$ chữ số, ta chỉ việc cố định các prefix, và tính số cách để điền phần còn lại
        ??? abstract "Cài đặt"

            ```cpp linenums="1"
			#include <bits/stdc++.h>
			using namespace std;

			const int N = 20;

			int64_t f[N];
			int64_t power[N];
			int num[N];
			int a[N];
			int n;

			bool is_palin(int64_t x) {
			  string s = to_string(x);
			  string ss = s;
			  reverse(ss.begin(), ss.end());
			  return s == ss;
			}

			bool is_palin(int *arr, int n) {
			  for (int i = 0; i < n; i++) {
			    if (arr[i] != arr[n - i - 1]) {
			      return false;
			    }
			  }
			  return true;
			}

			int count_palin(int64_t x) {
			  if (x <= 0) {
			    return 0;
			  }
			  if (x <= 9) {
			    return x;
			  }
			  n = 0;
			  int64_t xx = x;
			  do {
			    num[n++] = xx % 10;
			    xx /= 10;
			  } while (xx > 0);

			  reverse(num, num + n);

			  int64_t ans = 0;
			  // get rid of trailing zeros
			  for (int len = 1; len < n; len++) {
			    int half = (len + 1) / 2;
			    ans += power[half] / 10 * 9;
			  }

			  // now length is exactly n
			  // prefix of answer < prefix of current number
			  int64_t half = 0;
			  for (int i = 0; i < (n + 1) / 2; i++) {
			    half = half * 10 + num[i];
			  }
			  ans += max<int64_t>(0, half - power[(n + 1) / 2 - 1]);

			  // equal prefix

			  for (int i = 0; i < n / 2; i++) {
			    num[n - i - 1] = num[i];
			  }

			  if (is_palin(num, n)) {
			    int64_t value = 0;
			    for (int i = 0; i < n; i++) {
			      value = value * 10 + num[i];
			    }
			    ans += value <= x;
			  }

			  return ans;
			}

			int main() {
			  ios_base::sync_with_stdio(0);
			  cin.tie(0);
			  cout.tie(0);

			  memset(f, -1, sizeof(f));

			  power[0] = 1;
			  for (int i = 1; i <= 18; i++) {
			    power[i] = power[i - 1] * 10;
			  }

			  int q;
			  cin >> q;

			  for (int i = 1; i <= q; i++) {
			    int64_t l, r;
			    cin >> l >> r;
			    if (l > r) swap(l, r);
			    cout << "Case " << i << ": " << count_palin(r) - count_palin(l - 1) + (l <= 0 && 0 <= r) << "\n";
			  }

			  return 0;
			}
			```

    ??? tip "Đếm bằng quy hoạch động"

        - Gọi $n$ là độ dài cận trên
        - Gọi $dp(i,lo,hi,ze)$ là số số palindrome khi điền đến chữ số thứ $i$, biến $lo,hi$ ý nghĩa như trên, và $ze$ là số chữ số $0$ đứng đầu số hiện tại đang xây dựng
        - Do ta có tính chất $a_i=a_{n-i-1}$ khi $a[0:n-1]$ là số đối xứng, nên khi điền $a_i$ ta sẽ xác định luôn được $a_{j=n-i-1}$. Vậy ta chỉ cần điền một nửa số sẽ suy được cả cấu hình
        - Tuy nhiên vấn đề là ta không tính các chữ số $0$ không hợp lệ ở đầu số cần điền, vậy công thức xác định $j$ sẽ khác một chút dựa vào biến $ze$
        - Chi tiết có thể tham khảo code (nguồn Codeforces):
        
        ??? abstract "Cài đặt"

            ```cpp linenums="1"
            // i - position, l - leftmostlower, h - leftmosthigher, ze - numbers of leading zeros
            ll dp(int i, int l, int h, int ze) {
                // imagine it as n - i - 1, and plus the offset of leading zeros
                // i is already offset by leading zeros
                int j = n - i - 1 + ze;
                
                if(i > j) return l <= h;
                if(vis[i][l][h][ze] == cur) return mem[i][l][h][ze];
                vis[i][l][h][ze] = cur;
                
                ll res = 0;
                for(int d = 0; d <= 9; d++) {
                    int nl = l;
                    int nh = h;
                    
                    if(d < v[i] && i < nl) nl = i;
                    if(d < v[j] && j < nl) nl = j;
                    if(d > v[i] && i < nh) nh = i;
                    if(d > v[j] && j < nh) nh = j;
                    
                    res += dp(i + 1, nl, nh, ze + (i == ze && d == 0));
                }
                
                return mem[i][l][h][ze] = res;
            }
            ```

#### [Chữ số đẹp - PDIGIT](https://oj.vnoi.info/problem/olp_sc22_pdigit)

??? success "Lời giải"

    - Thay vì xây dựng lần lượt từng vị trí của số đã cho, ta sẽ xây dựng lần lượt các "tập" vị trí của từng chữ số từ $0$ đến $9$
    - Gọi $dp(mask,d,lo,hi)$ là số số thoả mãn:
        - đã điền các vị trí trong $mask$
        - hiện tại đang điền đến chữ số thứ $d$
        - $lo,hi$ có ý nghĩa như đã nói ở trên
    - Với mỗi trạng thái, ta duyệt các tập con của $(2^{11} - 1) \oplus mask$ (do giới hạn cận trên $\leq 10^{10}$) và kiểm tra các điều kiện, rồi chuyển sang trạng thái $(mask',d+1,lo',hi')$. 
    - Lưu ý trường hợp điền số $0$ thì số chữ số $0$ nhận được nếu tập các vị trí được điền thuộc $mask$ là số bit $1$ nằm sau bit $0$ có vị trí nhỏ nhất.

??? abstract "Cài đặt"

	```cpp linenums="1"
	#include <bits/stdc++.h>

	using namespace std;

	bool ex[15];
	int64_t f[1 << 11][12][12][10];
	bool can_use[1 << 11][2];
	int cnt[1 << 11][2];
	int a[15];

	// filled pos, left most lower, left most higher, current digit
	int64_t dp(int filled, int l, int h, int d) {
	  if (filled == (1 << 11) - 1) {
	    return l <= h;
	  }
	  if (d >= 10) {
	    return 0;
	  }
	  int64_t &ans = f[filled][l][h][d];
	  if (ans != -1) {
	    return ans;
	  }
	  ans = dp(filled, l, h, d + 1);
	  int rem = ((1 << 11) - 1) ^ filled;
	  for (int fill_d = rem; fill_d > 0; fill_d = (fill_d - 1) & rem) {
	    if (!can_use[fill_d][bool(d)]) {
	      continue;
	    }
	    if (!ex[cnt[fill_d][bool(d)]]) {
	      continue;
	    }
	    int nl = l;
	    int nh = h;
	    for (int i = 0; i < 11; i++) {
	      if (fill_d >> i & 1) {
	        if (d < a[i]) nl = min(nl, i);
	        if (d > a[i]) nh = min(nh, i);
	      }
	    }
	    ans += dp(filled | fill_d, nl, nh, d + 1);
	  }
	  return ans;
	}

	int64_t solve(int64_t x) {
	  memset(a, 0, sizeof(a));
	  memset(f, -1, sizeof(f));
	  int n = 0;
	  do {
	    a[n++] = x % 10;
	    x /= 10;
	  } while (x > 0);
	  reverse(a, a + 11);
	  return dp(0, 11, 11, 0);
	}

	int main() {
	  ios_base::sync_with_stdio(0);
	  cin.tie(0);
	  cout.tie(0);
	#define task "a"
	  if (fopen(task ".inp", "r")) {
	    freopen(task ".inp", "r", stdin);
	    freopen(task ".out", "w", stdout);
	  }
	  int n, k;
	  int64_t l, r;
	  cin >> n >> k >> l >> r;
	  for (int i = 1; i <= n; i++) {
	    int x;
	    cin >> x;
	    ex[x] = 1;
	  }
	  ex[0] = 1;
	  for (int mask = 0; mask < 1 << 11; mask++) {
	    int m[2];
	    {
	      m[1] = mask;
	      int first_off = __builtin_ctz(mask ^ ((1 << 11) - 1));
	      m[0] = mask & ~((1 << first_off) - 1);
	    }
	    for (int i = 0; i < 2; i++) {
	      cnt[mask][i] = __builtin_popcount(m[i]);
	      int last = -1;
	      bool &ok = can_use[mask][i];
	      ok = 1;
	      for (int j = 0; j < 11; j++) {
	        if (m[i] >> j & 1) {
	          if (last == -1) {
	          } else {
	            if (j - last < k) {
	              ok = 0;
	              break;
	            }
	          }
	          last = j;
	        }
	      }
	    }
	  }
	  cout << solve(r) - solve(l - 1);
	  return 0;
	}

	```

## Tài liệu tham khảo

- [Digit DP - Codeforces](https://codeforces.com/blog/entry/53960)
- [Digit DP "tricks" - Codeforces](https://codeforces.com/blog/entry/95488)
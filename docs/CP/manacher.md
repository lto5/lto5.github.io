# Thuật toán Manacher

Dịch từ bài viết gốc của [CP-Algo](https://cp-algorithms.com/string/manacher.html).

## Nhận xét

Nhận thấy nếu xâu độ dài $L$ đối xứng có tâm tại $i$, thì xâu độ dài $L-2,L-4,\cdots$ có tâm đặt tại $i$ cũng sẽ đối xứng.

## Định nghĩa

- Các kí tự được đánh số từ $1$.
- $d_{1i}$ là độ dài của xâu đối xứng dài nhất đặt tại $i$ (không tính $i$). Ví dụ xâu $s = abababc$ có $d_{1}[4]=3$:

$$a\ \overbrace{b\ a\ \underbrace{b}_{s_4}\ a\ b}^{d_{1}[4]=3} c$$

- $d_{2i}$ là độ dài của xâu đối xứng dài nhất có tâm chẵn tại $i$, nghĩa là xâu có tâm là 2 kí tự $s_i$ và $s_{i+1}$. Ví dụ xâu $s = cbaabd$ có $d_{2}[4]=2$:

$$c\ \overbrace{b\ a\ \underbrace{a}_{s_4}\ b}^{d_{2}[4]=2} d$$

- Với mỗi vị trí $i$, gọi $l, r$ là vị trí của xâu palindrome gần với vị trí $i$ nhất (bên phải nhất) tính đến thời điểm hiện tại. Ban đầu $l = 0, r = -1$.

## Giải thuật

### Tính mảng $d_1$

Với mỗi vị trí $i$, ta xét các trường hợp sau:

- Nếu $i>r$, chạy thuật toán trâu:

    ```
    while (i - d1[i] >= 1 and i + d1[i] <= n and s[i - d1[i]] = s[i + d1[i]]) 
        d1[i] += 1
    ```
    
- Ngược lại, xét trường hợp $i \leq r$, nghĩa là $i$ thuộc vào đoạn palin gần nhất đang xét. Nhận thấy ở đoạn $[l,r]$, vị trí đối xứng với vị trí $i$ qua tâm $\frac{l+r}{2}$ là vị trí $l+(r-i)$. Đặt $j=l+(r-i)$. Ở đây ta tiếp tục có $2$ trường hợp con:
    
    - Đoạn $[j-d_{1j}+1,j+d_{1j}-1]$ nằm trong đoạn $[l,r]$. Vì $s_j$ đối xứng với $s_i$ mà đoạn to $[l,r]$ đối xứng qua tâm $\Rightarrow$ đoạn $[j-d_{1j}+1,j+d_{1j}-1]$ và $[i-d_{1j}+1,i+d_{1j}-1]$  đối xứng. Như vậy ta gán $d_{1i} = d_{1j}$.
    
    $$
    \ldots\ 
    \overbrace{
        s_{l+1}\ \ldots\ 
        \underbrace{
            s_{j-d_{odd}[j]+1}\ \ldots\ s_j\ \ldots\ s_{j+d_{odd}[j]-1}\ 
        }_\text{palindrome}\ 
        \ldots\ 
        \underbrace{
            s_{i-d_{odd}[j]+1}\ \ldots\ s_i\ \ldots\ s_{i+d_{odd}[j]-1}\ 
        }_\text{palindrome}\ 
        \ldots\ s_{r-1}\ 
    }^\text{palindrome}\ 
    \ldots
    $$
    
    - Đoạn $[j-d_{1j}+1,j+d_{1j}-1]$ chỉ giao 1 phần với đoạn $[l,r]$. Lúc này ta không thể kết luận được $d_{1i} = d_{1j}$, bởi vì mình không quản lý cái phần nằm ngoài đoạn $[l,r]$, nên chưa đủ cơ sở để khẳng định $d_{1i} = d_{1j}$. Khi đó ta gán $d_{1i}=r-i$, và tiếp tục chạy thuật trâu để mở rộng tâm ra.
    
    $$
    \ldots\ 
    \overbrace{
        \underbrace{
            s_{l+1}\ \ldots\ s_j\ \ldots\ s_{j+(j-l)-1}\ 
        }_\text{palindrome}\ 
        \ldots\ 
        \underbrace{
            s_{i-(r-i)+1}\ \ldots\ s_i\ \ldots\ s_{r-1}
        }_\text{palindrome}\ 
    }^\text{palindrome}\ 
    \underbrace{
        \ldots \ldots \ldots \ldots \ldots
    }_\text{try moving here}
    $$

* Lưu ý, sau khi xét tất cả TH trên xong, ta cần cập nhật lại vị trí của biên $l$ và $r$

    ```
    if (i + d1[i] > r) l = i - d1[i], r = i + d1[i];
    ```

### Tính mảng $d_2$

Thêm các kí tự `DUMMY` vào đầu, cuối, và giữa các kí tự xâu, nghĩa là các kí tự mà không bao giờ xuất hiện trong xâu. Ví dụ xâu ``orz`` trở thành ``.o.r.z.`` với `DUMMY = .`.

Chạy thuật toán Manacher với tâm lẻ trên xâu mới, tạm gọi mảng này là mảng $d[]$. Lúc này nhận thấy các xâu đối xứng tâm chẵn trên xâu gốc đã trở thành xâu đối xứng tâm lẻ với tâm đặt tại các kí tự DUMMY trên xâu mới. Như vậy ta có $d_{2i}=\dfrac{d_{2 \times i + 1}}{2}$ (vì thêm các kí tự DUMMY làm số kí tự gấp đôi lên)

## Độ phức tạp

Nhận xét mỗi lần lặp $r$ sẽ được tăng lên ít nhất $1$ đơn vị, và $r$ không thể nào bị giảm đi, nên độ phức tạp của thuật toán sẽ là $O(n)$.

Các bạn có thể tham khảo chi tiết chứng minh tại link ở đầu bài.

## Cài đặt mẫu

??? success "Cài đặt của tác giả"

    ```cpp linenums="1"
    // AC submission for https://cses.fi/problemset/task/1111/

    #include<bits/stdc++.h>
    using namespace std;
        
    const int N = 1e6 + 5;
    const char DUMMY = '.';
        
    // 1-indexed
    int odd_center[N];
    int even_center[N];
    int d[N * 2];
        
    void odd_manacher(string s, int *p) {
        int n = s.size();
        s = DUMMY + s + DUMMY;
        int l = 0, r = -1;
        
        int dem = 0;
        
        for (int i = 1; i <= n; i++) {
            if (i > r)
                p[i] = 0;
            else
                p[i] = min(r - i, p[l + (r - i)]);
            while (i - p[i] >= 1 && i + p[i] <= n && s[i - p[i]] == s[i + p[i]])
                p[i]++;
            p[i]--;
            
            if (i + p[i] > r)
                l = i - p[i],
                r = i + p[i];
        }
    }
        
    void even_manacher(string s, int *p) {
        int n = s.size();
        string t = "";
        for (char ch: s) {
            t += '.';
            t += ch;
        }
        t += '.';
        odd_manacher(t, d);
        for (int i = 1; i <= n; i++) {
            p[i] = d[2 * i - 1] / 2;
        }
    }
        
    int main(){
        ios_base::sync_with_stdio(0);
        cin.tie(0);
        cout.tie(0);
        
        string s;
        cin >> s;
        int n = s.size();
        
        odd_manacher(s, odd_center);
        even_manacher(s, even_center); 
        
        int best_l = -1, best_r = -1;
        s = ' ' + s;
        
        for (int i = 1; i <= n; i++) {
            int l = i - odd_center[i], r = i + odd_center[i];
            if (r - l + 1 > best_r - best_l + 1)
                best_l = l, best_r = r;
            l = i - even_center[i], r = i + even_center[i] - 1;
            if (l >= 1 && r <= n && l <= r && r - l + 1 > best_r - best_l + 1)
                best_l = l, best_r = r;
        }
        
        cout << s.substr(best_l, best_r - best_l + 1);
        return 0;
    }
    ```

??? success "Cài đặt của [kc97ble](https://sites.google.com/site/kc97ble/1-3-d%C3%A3y-s%E1%BB%91-v%C3%A0-x%C3%A2u/manacher-cpp?authuser=0)"

    ```cpp linenums="1"
    #include <stdio.h>
    #include <string.h>
    #include <algorithm>
    #include <iostream>
    #include <string>

    using namespace std;

    bool maximize(int &a, int b) {
        if (a < b)
            a = b;
        else
            return false;
        return true;
    }

    void prepare(char a[], char b[]) {
        int cnt = 0;
        b[++cnt] = '#';
        for (int i = 1; a[i]; i++) {
            b[++cnt] = a[i];
            b[++cnt] = '#';
        }
        b[++cnt] = 0;
        b[0] = '^';
    }

    int manacher(char b[]) {
        int C = 1, R = 1, n = strlen(b + 1);
        int *P = new int[n + 2], r = 0;
        for (int i = 2; i < n; i++) {
            int i_mirror = 2 * C - i;
            P[i] = (R > i) ? min(R - i, P[i_mirror]) : 0;
            while (b[i - 1 - P[i]] == b[i + 1 + P[i]]) P[i]++;
            maximize(r, P[i]);
            if (i + P[i] > R) {
                C = i;
                R = i + P[i];
            }
        }
        delete[] P;
        return r;
    }

    #define N 50004
    char a[N], b[2 * N];

    int main() {
        ios::sync_with_stdio(false);
        int n;
        cin >> n >> (a + 1);
        prepare(a, b);
        cout << manacher(b) << endl;
    }
    ```

## Bài tập áp dụng
- [CSES - Longest Palindrome](https://cses.fi/problemset/task/1111/)
- [AtCoder - Palindrome Construction](https://atcoder.jp/contests/abc349/tasks/abc349_g) 
- [Library Checker - Enumerate Palindromes](https://judge.yosupo.jp/problem/enumerate_palindromes)
- [Longest Palindrome](https://cses.fi/problemset/task/1111)
- [UVA 11475 - Extend to Palindrome](https://onlinejudge.org/index.php?option=com_onlinejudge&Itemid=8&category=26&page=show_problem&problem=2470)
- [GYM - (Q) QueryreuQ](https://codeforces.com/gym/101806/problem/Q)
- [CF - Prefix-Suffix Palindrome](https://codeforces.com/contest/1326/problem/D2)
- [SPOJ - Number of Palindromes](https://www.spoj.com/problems/NUMOFPAL/)
- [Kattis - Palindromes](https://open.kattis.com/problems/palindromes)
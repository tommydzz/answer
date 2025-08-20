

---

# 1) 题目一算法设计（伪代码）

设 $A\in \mathbb{R}^{n\times n}$ 满足

$$
A_{i,j}=a_{n-1+i-j},\qquad 0\le i,j\le n-1,
$$

其中压缩表示 $\mathbf a=(a_0,a_1,\dots,a_{2n-2})$（长度 $2n-1$）。给定向量 $\mathbf v=(v_0,\dots,v_{n-1})$。

## 关键等式（把乘法写成卷积）

对任意 $i$,

$$
(A\mathbf v)_i=\sum_{j=0}^{n-1} A_{i,j}v_j
=\sum_{j=0}^{n-1} a_{\,n-1+i-j}\,v_j.
$$

令“反转向量” $\mathbf w$ 满足 $w_j=v_{n-1-j}$（$0\le j\le n-1$），则上式等价于

$$
(A\mathbf v)_i
=\sum_{j=0}^{n-1} a_{\,n-1+i-(n-1-j)}\, w_j
=\sum_{j=0}^{n-1} a_{\,i+j}\, w_j
=\bigl(\mathbf a * \mathbf w\bigr)_{\,n-1+i},
$$

其中 $*$ 为**线性卷积**。于是只要做一次 $\mathbf a$ 与 $\mathbf w$ 的线性卷积，再取出中间那一段即可。

## 伪代码（FFT 版本，$O(n\log n)$）

```
Input: n, a[0..2n-2], v[0..n-1]
Output: y[0..n-1] = A * v

1. // 反转 v，得到 w
2. for j = 0..n-1:
3.     w[j] = v[n-1-j]

4. // 线性卷积长度为 (2n-1) + n - 1 = 3n - 2
5. L = 3*n - 2
6. m = next_power_of_two(L)

7. // 零填充到长度 m
8. A_pad = zero_array(m);  for k = 0..2n-2: A_pad[k] = a[k]
9. W_pad = zero_array(m);  for k = 0..n-1:  W_pad[k] = w[k]

10. // FFT
11. FA = FFT(A_pad)       // 复杂度 O(m log m)
12. FW = FFT(W_pad)

13. // 点乘
14. FC[k] = FA[k] * FW[k] for k = 0..m-1

15. // IFFT 得到线性卷积
16. C = IFFT(FC)          // C[0..m-1] 的前 L 项为 a*w 的线性卷积

17. // 取出卷积的中间一段作为结果
18. for i = 0..n-1:
19.     y[i] = Re( C[n-1 + i] )

20. return y
```



---

# 2) 复杂度证明

* 卷积长度为 $L=3n-2=\Theta(n)$。取 $m$ 为不小于 $L$ 的 2 的幂，则 $m=\Theta(n)$。
* 两次 FFT 与一次 IFFT 各需 $O(m\log m)=O(n\log n)$；点乘是 $O(m)=O(n)$；其余线性操作均为 $O(n)$。
* 因此总时间复杂度为

$$
T(n)=O(n\log n).
$$

* 空间复杂度：需要存放长度 $m=\Theta(n)$ 的若干数组（含复数系数），总体为 $O(n)$ 辅助空间（若计入复数常数因子，可写为 $O(n)$）。

**正确性说明**
由上面“关键等式”

$$
(A\mathbf v)_i=(\mathbf a*\mathbf w)_{\,n-1+i}
$$

可知从线性卷积结果 $\mathbf a*\mathbf w$ 中截取索引 $n-1$ 到 $2n-2$ 的这段，恰好就是目标向量 $A\mathbf v$ 的各分量。这证明了算法输出与 $A\mathbf v$ 一致。



---

# 题目 2：用 FFT 求带 `?` 通配符的正则（单字符通配）匹配

## 思路与等式

给定主串 $S[0..n-1]$ 与模式串 $P[0..m-1]$，其中 `?` 可匹配任意**单个**字符。
记 $K=\#\{j\in[0,m-1]\mid P[j]\neq ?\}$ 为模式中**非 `?`** 的位置数。

对每个出现于 **$P$** 的字符 $c$（或者固定小字母表的每个字符）构造指示数组

$$
S_c[t]=\mathbf 1_{\{S[t]=c\}},\quad 0\le t<n; \qquad
P_c[j]=\mathbf 1_{\{P[j]=c\}},\quad 0\le j<m.
$$

将 $P_c$ 反转为 $\widetilde{P}_c[j]=P_c[m-1-j]$。做**线性卷积**

$$
C_c = S_c * \widetilde{P}_c.
$$

则当模式左端与主串位置 $i$ 对齐时（$0\le i\le n-m$），这个对齐的**匹配字符数**为

$$
M[i]=\sum_{c} C_c[i+m-1].
$$

因为 `?` 可匹配任意字符，只有非 `?` 处需要“刚好相同”。于是

$$
\text{位置 } i \text{ 为匹配}\ \Longleftrightarrow\ M[i]=K.
$$


## 算法（伪代码，$O(n\log n)$）

假设字母表有限且常数阶（如大小写英文字母）；若不是，则只对 **出现在 $P$** 的不同字符做通道（见后述复杂度与优化）。

```
Input: S[0..n-1], P[0..m-1] with '?' wildcard
Output: all match positions i in [0..n-m]

1. // 收集需要处理的字符集合 ΣP
2. ΣP = unique characters of P excluding '?'

3. // 为每个 c ∈ ΣP 构造指示数组
4. for each c in ΣP:
5.     Sc = zero array length n
6.     Pc = zero array length m
7.     for t = 0..n-1: if S[t] == c: Sc[t] = 1
8.     for j = 0..m-1: if P[j] == c: Pc[j] = 1
9.     reverse Pc in-place  // 得到 ~P_c

10. // 卷积长度 L = n + m - 1，取 mfft = next_power_of_two(L)
11. mfft = next_power_of_two(n + m - 1)

12. // 逐字符做 FFT 卷积，并累加到总计数组 Mconv
13. Mconv = zero array length (n + m - 1)
14. for each c in ΣP:
15.     A = pad Sc to length mfft with zeros
16.     B = pad Pc(reversed) to length mfft with zeros
17.     FA = FFT(A)
18.     FB = FFT(B)
19.     FC = FA .* FB     // 点乘
20.     C  = IFFT(FC)     // 线性卷积，长度 n+m-1
21.     Mconv += round(real(C))   // 累加到总匹配计数

22. // 统计 K = 非 '?' 的模式位数
23. K = count of j where P[j] != '?'

24. // 取出每个对齐 i 的匹配计数 M[i] = Mconv[i + m - 1]
25. matches = []
26. for i = 0..n-m:
27.     if Mconv[i + m - 1] == K:
28.         matches.push(i)

29. return matches
```

## 复杂度分析

* 设 $|\Sigma_P|$ 为 **模式串中不同的非 `?` 字符**的个数。每个字符需要 1 次长度 $\Theta(n)$ 的 FFT 与 IFFT（严格是 $\Theta(n+m)$，但 $n\gg m$）。
* 故总复杂度为

  $$
  T(n,m)=O\bigl(|\Sigma_P|\cdot(n\log n)\bigr).
  $$
* 若字母表为常数规模（如英文字母），或把计算限制在 $\Sigma_P$（最坏 $|\Sigma_P|\le m$），则整体即为

  $$
  T(n,m)=O(n\log n).
  $$
* 额外空间为若干长度 $\Theta(n)$ 的复数组，整体 $O(n)$。

---

## 题目 2 的可选实现变体（仍为 FFT，供参考）

有时希望把多字符的多通道卷积**合并/压缩**，减少 FFT 次数。以下是常见优化思路（第 3 问）：

### （A）仅对 $\Sigma_P$ 做通道（减少通道数）

* 已在算法中采用：只为 **出现在 $P$** 的字符建通道，通道数 $|\Sigma_P|\le m$。
* 当模式字符种类远小于全字母表时，FFT 次数显著减少。
* 复杂度：$O(|\Sigma_P|\,n\log n)$。

### （B）复数打包：一次 FFT 同时处理两条实序列

* 将两种字符的指示序列打包到复数实部/虚部：
  $X = X^{(1)} + i X^{(2)},\ Y = Y^{(1)} + i Y^{(2)}$。
  一次 FFT 与 IFFT 即可得到两种字符的卷积（需用分离公式从 $(X*Y)$ 中拆解）。
* 可把通道数减半，从而把 FFT 次数减半。








Kaguya gives you two arrays $[a_1,a_2...a_n]$ and $[b_1,b_2...b_m]$, and you need to select one subsequences $[a_{c_1},a_{c_2}...a_{c_p}]$ that satisfy the following conditions: - $c_1\le k$; - $\forall i \ge 2, c_i-c_{i-1}\le k$; - $c_p=n$. Now she asks you what is the largest $k, k \le M$ no matter how you choose $[c_1,c_2...c_p]$, the array $[b_{d_1},b_{d_2}...b_{d_q}]$ is a subarray of $[a_{c_1},a_{c_2}...a_{c_p}]$.

$1\le M \le 100\quad 0 \le n, m \le1e6$

# Options Calculator
├── Backend
│   ├── __init__.py
│   ├── Option.py
├── Frontend
│   ├── about.py
│   ├── img
│   │   ├── background_2.png
│   │   ├── background_3.png
│   │   ├── background.png
│   │   ├── hint.png
│   │   └── logo2.png
│   ├── input.py
│   ├── list.py
│   ├── main.py
│   ├── page.py
│   ├── quit.py
│   ├── result.py
│   ├── style.qss
│   └── welcome.py
├── img
│   ├── about.png
│   ├── input.png
│   ├── list.png
│   ├── quit.png
│   ├── result.png
│   └── welcome.png
├── README.md
└── requirements.txt


class Option:
  
    def __init__(self, european, kind, s0, k, t, r, sigma, dv):
        self.european = european
        self.kind = kind
        self.s0 = s0
        self.k = k
        self.t = t /365
        self.sigma = sigma
        self.r = r
        self.dv = dv
        self.bsprice = None
        self.mcprice = None
        self.btprice = None

    def bs(self):
        if self.european or self.kind == 1:
            d_1 = (np.log(self.s0 / self.k) + (
                    self.r - self.dv + .5 * self.sigma ** 2) * self.t) / self.sigma / np.sqrt(
                self.t)
            d_2 = d_1 - self.sigma * np.sqrt(self.t)
            self.bsprice = self.kind * self.s0 * np.exp(-self.dv * self.t) * sps.norm.cdf(
                self.kind * d_1) - self.kind * self.k * np.exp(-self.r * self.t) * sps.norm.cdf(self.kind * d_2)
        else:
            self.bsprice = "美式看跌期权不适合这种计算方法"

$$
d_1 = \frac{ln\frac{S0}{k} + (r+ 0.5 \cdot \sigma^2 - dv)t}{\sigma \cdot \sqrt{t}}
$$

$$
d_2 = d_1 - \sigma \sqrt{t}
$$


$$
P = S_0 \cdot e^{-dv \cdot t} \cdot N(d_1) - k \cdot e^{-rt}N(d_2)
$$

$$
P = ke^{-rt}[1-N(d_2)] - S_0[1-N(d_1)]
$$

$$
N(d) = 1 - N(-d)
$$
    def mc(self, iteration):
        if self.european or self.kind == 1:
            zt = np.random.normal(0, 1, iteration)
            st = self.s0 * np.exp((self.r - self.dv - .5 * self.sigma ** 2) * self.t + self.sigma * self.t ** .5 * zt)
            st = np.maximum(self.kind * (st - self.k), 0)
            self.mcprice = np.average(st) * np.exp(-self.r * self.t)
        else:
            self.mcprice = "美式看跌期权不适合这种计算方法"
```


```python
    def bt(self, iteration):
        if iteration % 2 != 0:
            iteration += 1
        delta = self.t / iteration
        u = np.exp(self.sigma * np.sqrt(delta))
        d = 1 / u
        p = (np.exp((self.r - self.dv) * delta) - d) / (u - d)
        tree = []
        for j in range(int(iteration / 2) + 1):
            i = j * 2
            temp = self.s0 * np.power(u, iteration - i)
            temp = np.max([(temp - self.k) * self.kind, 0])
            tree.append(temp)
        for j in range(1, int(iteration / 2) + 1):
            i = j * 2
            temp = self.s0 * np.power(d, i)
            temp = np.max([(temp - self.k) * self.kind, 0])
            tree.append(temp)
        for j in range(0, iteration):
            newtree = []
            for i in (range(len(tree) - 1)):
                temp = tree[i] * p + (1 - p) * tree[i + 1]
                temp = temp * np.exp(-self.r * delta)
                if not self.european:
                    # 每一层的最高幂次
                    k = iteration - j - 1
                    if i < (k + 1) / 2:
                        power = k - i * 2
                        compare = self.s0 * np.power(u, power)
                    else:
                        power = i * 2 - k
                        compare = self.s0 * np.power(d, power)
                    temp = np.max([temp, (compare - self.k) * self.kind])
                newtree.append(temp)
            tree = newtree
        self.btprice = tree[0]
```


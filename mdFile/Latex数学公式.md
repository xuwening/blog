# Latex数学公式


用到数学符号总是从其他地方截图，一是看起来不美观，二是总依赖一堆图，还需要网络上去查找，一旦发生变更就更麻烦，索性就看了看手写数学公式的方式，发现也不复杂。于是做个笔记，方便以后查找。

## 开始和结束符号

数学公式使用`$`符号包裹，块级公式使用两个`$$`符号，行内元素使用一个`$`符号。

示例：

```
$$a^2 + b^2 = c^2$$
这是$a^2 + b^2 = c^2$行内公式

```

效果：

$$a^2 + b^2 = c^2$$

这是$a^2 + b^2 = c^2$行内公式

## 上下标

数学公式中上下标使用比较频繁，上标使用`^`表示，下标使用`_`表示：

```
$$A^N_K$$
```

$$A^N_K$$

## 群组

当上下标需要多个元素时，就需要使用`{}`包裹起来：

```
$$a^{2+1}$$
```

$$a^{2+1}$$

## 不等式

输入`不等式`时，可以使用专用符号代替，注意需要加前缀`\`：

```
$$a\geq b$$
$$a\leq b$$
$$a\neq b$$
```
$$ a\geq b$$
$$a\leq b$$
$$a\neq b$$

## 平方根

```
$$ \sqrt[n]	{x} $$
$$\sqrt{1+\sqrt[^p\!]{1+a^2}}$$
```
$$ \sqrt[n]	{x} $$
$$\sqrt{1+\sqrt[^p\!]{1+a^2}}$$

## 希腊字母

```
$$ \lambda \xi \pi \mu \Phi \omega \alpha \beta  \gamma $$
```

$$ \lambda \	 \xi \	\pi \	\mu \	\Phi \	\omega \	\alpha \	\beta  \	\gamma $$

## 水平线

```
$$ \overline{m+n} $$
$$ \overbrace{m+n} $$
$$ \underbrace{a+b+\cdots+z} $$
```

$$ \overline{m+n} $$
$$ \overbrace{m+n} $$
$$ \underbrace{a+b+\cdots+z} $$


## 点与省略号

```
$$ \cdot $$
$$ \cdots$$
```

$$ \cdot $$
$$ \cdots$$

```
$$a+b+\cdots+n$$
```

$$a+b+\cdots+n$$


## 分子、分母

分子分母表达式使用`\frac{分子}{分母}`:

```
$$\frac{x+y}{2}$$
$$\frac{1}{1+\frac{1}{2}}$$
```
$$\frac{x+y}{2}$$
$$\frac{1}{1+\frac{1}{2}}$$


## 向量

```
$$ \overrightarrow{AB} $$
```

$$ \overrightarrow{AB} $$



## 求和符号

```
$$\sum_{k=1}^{n}\frac{1}{k}$$
```

$$\sum_{k=1}^{n}\frac{1}{k}$$


## 极限

```
$$                 
\lim_{m \to \infty}
\sum_{k=1}^n \frac{1}{k^2}= \frac{\pi^2}{6}
$$
```

$$                 
\lim_{m \to \infty}
\sum_{k=1}^n \frac{1}{k^2}= \frac{\pi^2}{6}
$$


## 微积分符号

```
$$\int_a^b f(x)dx$$
```

$$\int_a^b f(x)dx$$


## 少于 10 列的矩阵

```
$$\begin{matrix}1 & 2\\3 &4\end{matrix}$$
$$\begin{pmatrix}1 & 2\\3 &4\end{pmatrix}$$
$$\begin{bmatrix}1 & 2\\3 &4\end{bmatrix}$$
$$\begin{Bmatrix}1 & 2\\3 &4\end{Bmatrix}$$
$$\begin{vmatrix}1 & 2\\3 &4\end{vmatrix}$$
$$\begin{Vmatrix}1 & 2\\3 &4\end{Vmatrix}$$
```

$$\begin{matrix}1 & 2\\3 &4\end{matrix}$$
$$\begin{pmatrix}1 & 2\\3 &4\end{pmatrix}$$
$$\begin{bmatrix}1 & 2\\3 &4\end{bmatrix}$$
$$\begin{Bmatrix}1 & 2\\3 &4\end{Bmatrix}$$
$$\begin{vmatrix}1 & 2\\3 &4\end{vmatrix}$$
$$\begin{Vmatrix}1 & 2\\3 &4\end{Vmatrix}$$


## 大于 10 列的矩阵

```
$$
\mathbf{X} =
\left( \begin{array}{ccc}
x_{11} & x_{12} & \ldots \\
x_{21} & x_{22} & \ldots \\
\vdots & \vdots & \ddots
\end{array} \right)
$$
```

$$
\mathbf{X} =
\left( \begin{array}{ccc}
x_{11} & x_{12} & \ldots \\
x_{21} & x_{22} & \ldots \\
\vdots & \vdots & \ddots
\end{array} \right)
$$

## 矩阵分割线

```
$$
\left(\begin{array}{c|c}
1 & 2 \\
\hline
3 & 4
\end{array}\right)
$$
```

$$
\left(\begin{array}{c|c}
1 & 2 \\
\hline
3 & 4
\end{array}\right)
$$

## 附录

Greek letters

$$
\alpha A
\nu N
\beta B
\xi\Xi
\gamma \Gamma
\delta \Delta
\pi \Pi
\epsilon \varepsilon E
\rho\varrho P
\zeta Z
\sigma \Sigma
\eta H
\tau T
\theta \vartheta \Theta
\upsilon \Upsilon
\iota I
\phi \varphi \Phi
\kappa K
\chi X
\lambda \Lambda
\psi \Psi
\mu M
\omega \Omega
$$

Arrows

$$
\leftarrow
\Leftarrow
\rightarrow
\Rightarrow
\leftrightarrow
\rightleftharpoons
\uparrow
\downarrow
\Uparrow
\Downarrow
\Leftrightarrow
\Updownarrow
\mapsto
\longmapsto
\nearrow
\searrow
\swarrow
\nwarrow
\leftharpoonup
\rightharpoonup
\leftharpoondown
\rightharpoondown
$$

Miscellaneous symbols

$$
\infty
\forall
\Re
\Im
\nabla
\exists
\partial
\nexists
\emptyset
\varnothing
\wp
\complement
\neg
\cdots
\square
\surd
\blacksquare
\triangle
$$

Binary Operation/Relation Symbols

$$
\times
\otimes
\div
\cap
\cup
\neq
\leq
\geq
\in
\perp
\notin
\subset
\simeq
\approx
\wedge
\vee
\oplus
\otimes
\Box
\boxtimes
\equiv
\cong
$$


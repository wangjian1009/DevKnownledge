# Markdown书写公式
by 顾轶康

[TOC]

## 1.如何插入公式

- 行中公式(放在文中与其它文字混编)可以用如下方法表示：`$ 数学公式 $`

- 独立公式可以用如下方法表示：`$$ 数学公式 $$`

- 自动编号的公式可以用如下方法表示：

  ```bash
  \begin{equation}
  数学公式
  \label{eq:当前公式名}
  \end{equation}
  自动编号后的公式可在全文任意处使用 \eqref{eq:公式名} 语句引用。
  ```

例子：

```bash
$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，行内公式示例} $
```

显示：

$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，行内公式示例} $

例子：

```
$$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，独立公式示例} $$
```

显示：

$$ J_\alpha(x) = \sum_{m=0}^\infty \frac{(-1)^m}{m! \Gamma (m + \alpha + 1)} {\left({ \frac{x}{2} }\right)}^{2m + \alpha} \text {，独立公式示例} $$



## 2.如何输入上下标

`^`表示上标, `_` 表示下标。如果上下标的内容多于一个字符，需要用 `{}`将这些内容括成一个整体。上下标可以嵌套，也可以同时使用。
例子：
`$$ x^{y^z}=(1+{\rm e}^x)^{-2xy^w} $$`
显示：

$$ x^{y^z}=(1+{\rm e}^x)^{-2xy^w} $$

另外，如果要在左右两边都有上下标，可以用`\sideset` 命令

- 例子：
  `$$ \sideset{^1_2}{^3_4}\bigotimes $$`
- 显示：

$$ \sideset{^1_2}{^3_4}\bigotimes $$



## 3．如何输入括号和分隔符

`()`、`[]`和`|`表示符号本身，使用 `\{\}` 来表示 `{}`。当要显示大号的括号或分隔符时，要用 `\left` 和 `\right` 命令。
一些特殊的括号：

| 输入                       | 显示             |
| -------------------------- | ---------------- |
| `$$\langle表达式\rangle$$` | ⟨表达式⟩⟨表达式⟩ |
| `$$\lceil表达式\rceil$$`   | ⌈表达式⌉⌈表达式⌉ |
| `$$\lfloor表达式\rfloor$$` | ⌊表达式⌋⌊表达式⌋ |
| `$$\lbrace表达式\rbrace$$` | {表达式}{表达式} |

例子：
`$$ f(x,y,z) = 3y^2z \left( 3+\frac{7x+5}{1+y^2} \right) $$`
显示：

$$ f(x,y,z) = 3y^2z \left( 3+\frac{7x+5}{1+y^2} \right) $$

有时候要用`\left.`或`\right.`进行匹配而不显示本身。

例子：`$$\left. \frac{ {\rm d}u}{ {\rm d}x} \right| _{x=0}$$`

显示：$$\left. \frac{ {\rm d}u}{ {\rm d}x} \right| _{x=0}$$



## 4．如何输入分数

通常使用 `\frac {分子} {分母}`命令产生一个分数\frac {分子} {分母}，分数可嵌套。
便捷情况可直接输入 `\frac ab`来快速生成一个\frac ab。
如果分式很复杂，亦可使用 分子 \over 分母 命令，此时分数仅有一层。

例子：

`$$\frac{a-1}{b-1} \quad and \quad {a+1\over b+1}$$`

显示：

$$\frac{a-1}{b-1} \quad and \quad {a+1\over b+1}$$



## 5．如何输入开方

使用 `\sqrt [根指数，省略时为2] {被开方数}`命令输入开方。

例子：

`$$\sqrt{2} \quad and \quad \sqrt[n]{3}$$`

显示：

$$\sqrt{2} \quad and \quad \sqrt[n]{3}$$



## 6．如何输入省略号

数学公式中常见的省略号有两种，\ldots 表示与文本底线对齐的省略号，\cdots 表示与文本中线对齐的省略号。

例子：

`$$f(x_1,x_2,\underbrace{\ldots}_{\rm ldots} ,x_n) = x_1^2 + x_2^2 + \underbrace{\cdots}_{\rm cdots} + x_n^2$$`

显示：

$$f(x_1,x_2,\underbrace{\ldots}_{\rm ldots} ,x_n) = x_1^2 + x_2^2 + \underbrace{\cdots}_{\rm cdots} + x_n^2$$



## 7．如何输入矢量

使用 `\vec{矢量}`来自动产生一个矢量。也可以使用 `\overrightarrow`等命令自定义字母上方的符号。

例子：

`$$\vec{a} \cdot \vec{b}=0$$`

显示：

$$\vec{a} \cdot \vec{b}=0$$

例子：

`$$\overleftarrow{xy} \quad and \quad \overleftrightarrow{xy} \quad and \quad \overrightarrow{xy}$$`

显示：

$$\overleftarrow{xy} \quad and \quad \overleftrightarrow{xy} \quad and \quad \overrightarrow{xy}$$



## 8．如何输入积分

使用 \int_积分下限^积分上限 {被积表达式} 来输入一个积分。

例子：

`$$\int_0^1 {x^2} \,{\rm d}x$$`

显示：

$$\int_0^1 {x^2} \,{\rm d}x$$



## 9．如何偏导

使用`\partial` 来输入一个偏导。

例子：

`$$\frac{\partial^{2}y}{\partial x^{2}}$$`

显示：

$$\frac{\partial^{2}y}{\partial x^{2}}$$



## 10．如何输入极限运算

使用`\lim_{变量 \to 表达式} 表达式` 来输入一个极限。如有需求，可以更改 `\to` 符号至任意符号。

例子：

`$$ \lim_{n \to +\infty} \frac{1}{n(n+1)} \quad and \quad \lim_{x\leftarrow{示例}} \frac{1}{n(n+1)} $$`

显示：

$$ \lim_{n \to +\infty} \frac{1}{n(n+1)} \quad and \quad \lim_{x\leftarrow{示例}} \frac{1}{n(n+1)} $$



## 11．如何输入累加、累乘运算

使用 `\sum_{下标表达式}^{上标表达式} {累加表达式}`来输入一个累加。
与之类似，使用 `\prod \bigcup \bigcap`来分别输入累乘、并集和交集。
此类符号在行内显示时上下标表达式将会移至右上角和右下角。

例子：

`$$\sum_{i=1}^n \frac{1}{i^2} \quad and \quad \prod_{i=1}^n \frac{1}{i^2} \quad and \quad \bigcup_{i=1}^{2} R$$`

显示：

$$\sum_{i=1}^n \frac{1}{i^2} \quad and \quad \prod_{i=1}^n \frac{1}{i^2} \quad and \quad \bigcup_{i=1}^{2} R$$



## 12．如何输入希腊字母

输入 `\小写希腊字母英文全称`和`\首字母大写希腊字母英文全称`来分别输入小写和大写希腊字母。
对于大写希腊字母与现有字母相同的，直接输入大写字母即可。

| 输入         | 显示 | 输入         | 显示 |
| ------------ | ---- | ------------ | ---- |
| `$\alpha$`   | α    | `$A$`        | A    |
| `$\beta$`    | β    | `$B$`        | B    |
| `$\gamma$`   | γ    | `$\Gamma$`   | Γ    |
| `$\delta$`   | δ    | `$\Delta$`   | Δ    |
| `$\epsilon$` | ϵ    | `$E$`        | E    |
| `$\zeta$`    | ζ    | `$Z$`        | Z    |
| `$\eta$`     | η    | `$H$`        | H    |
| `$\theta$`   | θ    | `$\Theta$`   | Θ    |
| `$\iota$`    | ι    | `$I$`        | I    |
| `$\kappa$`   | κ    | `$K$`        | K    |
| `$\lambda$`  | λ    | `$\Lambda$`  | Λ    |
| `$\nu$`      | ν    | `$N$`        | N    |
| `$\mu$`      | μ    | `$M$`        | M    |
| `$\xi$`      | ξ    | `$\Xi$`      | Ξ    |
| `$o$`        | o    | `$O$`        | O    |
| `$\pi$`      | π    | `$\Pi$`      | Π    |
| `$\rho$`     | ρ    | `$P$`        | P    |
| `$\sigma$`   | σ    | `$\Sigma$`   | Σ    |
| `$\tau$`     | τ    | `$T$`        | T    |
| `$\upsilon$` | υ    | `$\Upsilon$` | Υ    |
| `$\phi$`     | ϕ    | `$\Phi$`     | Φ    |
| `$\chi$`     | χ    | `$X$`        | X    |
| `$\psi$`     | ψ    | `$\Psi$`     | Ψ    |
| `$\omega$`   | ω    | `$\Omega$`   | Ω    |

## 13.大括号和行标的使用

使用 `\left`和 `\right`来创建自动匹配高度的 (圆括号)，[方括号] 和 {花括号} 。
在每个公式末尾前使用`\tag{行标}`来实现行标。
例子：

```
$$
f\left(
   \left[ 
     \frac{
       1+\left\{x,y\right\}
     }{
       \left(
          \frac{x}{y}+\frac{y}{x}
       \right)
       \left(u+1\right)
     }+a
   \right]^{3/2}
\right)
\tag{行标}
$$
```

显示：
$$
f\left(
   \left[ 
     \frac{
       1+\left\{x,y\right\}
     }{
       \left(
          \frac{x}{y}+\frac{y}{x}
       \right)
       \left(u+1\right)
     }+a
   \right]^{3/2}
\right)
\tag{行标}
$$


## 14. 运算符：

| 关系运算符 | markdown语言   | 集合运算符 | markdown语言  | 对数运算符 | markdown语言 | 戴帽符号 | markdown语言  |
| ---------- | -------------- | ---------- | ------------- | ---------- | ------------ | -------- | ------------- |
| ±          | `$\pm$`        | ∅          | `$\emptyset$` | log        | `$\log$`     | y^       | `$\hat{y}$`   |
| ×          | `$\times$`     | ∈          | `$\in$`       | lg         | `$\lg$`      | yˇ       | `$\check{y}$` |
| ÷          | `$\div$`       | ∉          | `$\notin$`    | ln         | `$\ln$`      | y˘       | `$\breve{y}$` |
| ∣          | `$\mid$`       | ⊂          | `$\subset$`   |            |              |          |               |
| ∤          | `$\nmid$`      | ⊃          | `$\supset$`   |            |              |          |               |
| ⋅          | `$\cdot$`      | ⊆          | `$\subseteq$` |            |              |          |               |
| ∘          | `$\circ$`      | ⊇          | `$\supseteq$` |            |              |          |               |
| ∗          | `$\ast$`       | ⋂          | `$\bigcap$`   |            |              |          |               |
| ⨀          | `$\bigodot$`   | ⋃          | `$\bigcup$`   |            |              |          |               |
| ⨂          | `$\bigotimes$` | ⋁          | `$\bigvee$`   |            |              |          |               |
| ⨁          | `$\bigoplus$`  | ⋁          | `$\bigvee$`   |            |              |          |               |
| ≤          | `$\leq$`       | ⋀          | `$\bigwedge$` |            |              |          |               |
| ≥          | `$\geq$`       | ⨄          | `$\biguplus$` |            |              |          |               |
| ≠          | `$\neq$`       | ⨆          | `$\bigsqcup$` |            |              |          |               |
| ≈          | `$\approx$`    |            |               |            |              |          |               |
| ≡          | `$\equiv$`     |            |               |            |              |          |               |
| ∑          | `$\sum$`       |            |               |            |              |          |               |
| ∏          | `$\prod$`      |            |               |            |              |          |               |
| ∐          | `$\coprod$`    |            |               |            |              |          |               |

| 三角运算符 | markdown语言 | 微积分运算符 | markdown语言 | 逻辑运算符 | markdown语言    |
| ---------- | ------------ | ------------ | ------------ | ---------- | --------------- |
| ⊥          | `$\bot$`     | ′            | `$\prime$`   | ∵          | `$\because$`    |
| ∠          | `$\angle$`   | ∫            | `$\int$`     | ∴          | `$\therefore$`  |
| 30∘        | `$30^\circ$` | ∬            | `$\iint$`    | ∀          | `$\forall$`     |
| sin        | `$\sin$`     | ∭            | `$\iiint$`   | ∃          | `$\exists$`     |
| cos        | `$\cos$`     | ∬∬           | `$\iiiint$`  | ≠          | `$\not=$`       |
| tan        | `$\tan$`     | ∮            | `$\oint$`    | ≯          | `$\not>$`       |
| cot        | `$\cot$`     | lim          | `$\lim$`     | ⊄          | `$\not\subset$` |
| sec        | `$\sec$`     | ∞            | `$\infty$`   |            |                 |
| csc        | `$\csc$`     | ∇            | `$\nabla$`   |            |                 |

| 箭头符号 | markdown语言        |
| -------- | ------------------- |
| ↑        | `$\uparrow$`        |
| ↓        | `$\downarrow$`      |
| ⇑        | `$\Uparrow$`        |
| ⇓        | `$\Downarrow$`      |
| →        | `$\rightarrow$`     |
| ←        | `$\leftarrow$`      |
| ⇒        | `$\Rightarrow$`     |
| ⇐        | `$\Leftarrow$`      |
| ⟶        | `$\longrightarrow$` |
| ⟵        | `$\longleftarrow$`  |
| ⟹        | `$\Longrightarrow$` |
| ⟸        | `$\Longleftarrow$`  |


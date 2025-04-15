### The Laplace Transform Approach for Solving Constant-Coefficient Linear ODEs

## 3.1 The Laplace Transform

### Definition
The **Laplace transform** of a function $f(t)$, denoted $\mathcal{L}\{f(t)\}$, is defined by:
$$
F(s) = \mathcal{L}\{f(t)\} = \int_0^{\infty} e^{-st}f(t)\,dt
$$
- Converts a function of time $t$ into a function of the complex variable $s.
- Especially useful for solving ODEs with constant coefficients.

---

## 3.2 Two Fundamental Properties of the Laplace Transform

### 1. Linearity
$$
\mathcal{L}\{a f(t) + b g(t)\} = a \mathcal{L}\{f(t)\} + b \mathcal{L}\{g(t)\}
$$

### 2. Derivative Property
$$
\mathcal{L}\{f'(t)\} = sF(s) - f(0)
$$
This is crucial for transforming ODEs into algebraic equations.

---

## 3.3 The Inverse Laplace Transform

- To recover $f(t)$ from $F(s)$, apply the **inverse Laplace transform**, denoted:
$$
f(t) = \mathcal{L}^{-1}\{F(s)\}
$$
- In practice, use standard tables or partial fraction decomposition to find the inverse.

---

## 3.4 Using Laplace Transforms to Solve IVPs

### General Steps
1. **Take the Laplace transform** of both sides of the ODE.
2. **Substitute initial conditions** (thanks to the derivative property).
3. **Solve for \(F(s)\)** (an algebraic equation).
4. **Find \(f(t)\)** by taking the inverse Laplace transform.

### Example (First-Order ODE)
For:
$$
y' + ay = f(t), \quad y(0) = y_0
$$
Take Laplace transform:
$$
sY(s) - y_0 + aY(s) = F(s)
$$
Solve for \(Y(s)\):
$$
Y(s) = \frac{F(s) + y_0}{s + a}
$$
Apply inverse transform to get \(y(t)\).

---

## 3.5 Other Properties of the Laplace Transform

### Time Shift
$$
\mathcal{L}\{u(t - T)f(t - T)\} = e^{-sT}F(s)
$$

### Frequency Shift
$$
\mathcal{L}\{e^{at}f(t)\} = F(s - a)
$$

---

## 3.7 Summary of Important Results and Key Terms

- **Laplace Transform:** Converts time-domain functions into \(s\)-domain.
- **Inverse Laplace Transform:** Converts back into time-domain.
- **Linearity & Derivative Properties:** Key tools for handling ODEs.
- **Initial-Value Problem:** IVP solved by transforming ODE into algebraic form.
- **Laplace Table:** Essential for solving inverse transforms.

---

## 3.8 Table of Common Laplace Transforms

| Time domain, $f(t)$                                         | $s$-domain, $F(s) = \mathcal{L}\{f(t)\}$ |
| ----------------------------------------------------------- | ---------------------------------------- |
| $u_{\text{step}}(t)$                                        | $\frac{1}{s}$                            |
| $u_{\text{ramp}}(t)$ (i.e., $f(t) = t\,u_{\text{step}}(t)$) | $\frac{1}{s^2}$                          |
| $e^{at}\,u_{\text{step}}(t)$                                | $\frac{1}{s - a}$                        |
| $\sin(at)\,u_{\text{step}}(t)$                              | $\frac{a}{s^2 + a^2}$                    |
| $\cos(at)\,u_{\text{step}}(t)$                              | $\frac{s}{s^2 + a^2}$                    |
| $\delta(t)$                                                 | $1$                                      |
| $t^n u_{\text{step}}(t)$ (for $n \geq 1$)                   | $\frac{n!}{s^{n+1}}$                     |
| $\sin^2(at)\,u_{\text{step}}(t)$                            | $\frac{2a^2}{s(s^2 + 4a^2)}$             |
| $\cos^2(at)\,u_{\text{step}}(t)$                            | $\frac{s^2 + 2a^2}{s(s^2 + 4a^2)}$       |
| $t\,\sin(at)\,u_{\text{step}}(t)$                           | $\frac{2as}{(s^2 + a^2)^2}$              |
| $t\,\cos(at)\,u_{\text{step}}(t)$                           | $\frac{s^2 - a^2}{(s^2 + a^2)^2}$        |
| $[\sin(at) - at\cos(at)]\,u_{\text{step}}(t)$               | $\frac{2a^3}{(s^2 + a^2)^2}$             |
| $e^{bt}\sin(at)\,u_{\text{step}}(t)$                        | $\frac{a}{(s - b)^2 + a^2}$              |
| $e^{bt}\cos(at)\,u_{\text{step}}(t)$                        | $\frac{s - b}{(s - b)^2 + a^2}$          |
| $te^{at}\,u_{\text{step}}(t)$                               | $\frac{1}{(s - a)^2}$                    |
| $t^2 e^{at}\,u_{\text{step}}(t)$                            | $\frac{2}{(s - a)^3}$                    |
| $t^n e^{at}\,u_{\text{step}}(t)$ (for $n \geq 1$)           | $\frac{n!}{(s - a)^{n+1}}$               |

---

## Techniques for Solving First-Order ODEs using Laplace Transforms

1. **Transform both sides of the equation.**
2. **Apply initial condition at \(t = 0\).**
3. **Solve for \(Y(s)\).**
4. **Look up or calculate inverse Laplace transform to find \(y(t)\).**

### Example
Solve:
$$
y' + 3y = 2e^{-t}, \quad y(0) = 5
$$

#### Step 1: Take Laplace transform
$$
sY(s) - 5 + 3Y(s) = \frac{2}{s+1}
$$

#### Step 2: Rearrange
$$
Y(s) = \frac{5 + \frac{2}{s+1}}{s + 3} = \frac{5}{s+3} + \frac{2}{(s+1)(s+3)}
$$

#### Step 3: Partial fraction decomposition
$$
\frac{2}{(s+1)(s+3)} = \frac{A}{s+1} + \frac{B}{s+3}
$$
Solve for \(A\) and \(B\), then inverse transform.

#### Final Step: Apply inverse Laplace transform
Using the table above, match terms to find \(y(t)\).

---

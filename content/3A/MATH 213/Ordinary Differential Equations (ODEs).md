## 2.1 What is Meant by a Solution to an ODE?

### 2.1.1 Definition of a Solution to an ODE
A **solution** to an ordinary differential equation (ODE) is a function \(y(t)\) that satisfies the equation when substituted into it.  
For example, for:
$$
\frac{dy}{dt} = f(t, y)
$$
\(y(t)\) is a solution if it satisfies this equation on some interval.

---

### 2.1.2 Existence and Uniqueness of Solutions to an ODE
- **Existence:** Determines whether a solution exists.
- **Uniqueness:** Determines whether the solution is unique.

**Picard-Lindelöf Theorem (simplified):**  
If \(f(t, y)\) is continuous and **Lipschitz continuous** in \(y\), the initial value problem has a unique solution.

---

### 2.1.3 The Initial-Value Problem (IVP)
An **initial-value problem (IVP)** specifies:
- The ODE
- An initial condition

Example:
$$
\frac{dy}{dt} = f(t, y), \quad y(t_0) = y_0
$$
The goal is to find \(y(t)\) satisfying both the ODE and \(y(t_0) = y_0\).

---

### 2.1.4 Factors that Affect the Solution to an IVP
- Continuity of \(f(t, y)\)
- Lipschitz continuity in \(y\)
- Location of initial condition \((t_0, y_0)\)
- Length of the desired interval for the solution

---

## 2.2 Several Approaches for Solving Certain First-Order ODEs

### 2.2.1 Approach 1: Phase Portrait Sketch
- **Qualitative method** to understand behavior without solving explicitly.
- Sketch trajectories, equilibria, and flow in the \(ty\) or \(y\)-\(y'\) plane.
- Useful for **autonomous ODEs**: 
$$
\frac{dy}{dt} = f(y)
$$

---

### 2.2.2 Approach 2: Separation of Variables
- Used when the ODE is separable:
$$
\frac{dy}{dt} = g(t)h(y)
$$
- Rearranged into:
$$
\frac{dy}{h(y)} = g(t) \, dt
$$
- Integrate both sides to solve for \(y(t)\).

---

### 2.2.3 Approach 3: Exact Differential Approach
- Applies to **exact equations** of the form:
$$
M(t, y) \, dt + N(t, y) \, dy = 0
$$
- Check the **exactness condition**:
$$
\frac{\partial M}{\partial y} = \frac{\partial N}{\partial t}
$$
- This means that we can take the partial derivative from both sides and if equal, we can find the ODE
- If exact, there is a potential function \(F(t, y)\) such that:
$$
\frac{\partial F}{\partial t} = M,\ \frac{\partial F}{\partial y} = N
$$
- The solution is:
$$
F(t, y) = C
$$
This is an **implicit solution**.

---

## 2.3 Solving Constant-Coefficient Linear ODEs
- First-order form:
$$
\frac{dy}{dt} + ay = f(t)
$$
- **Homogeneous case** (\(f(t) = 0\)):
$$
y(t) = Ce^{-at}
$$
- **Non-homogeneous case** (using an integrating factor):
$$
\mu(t) = e^{\int a \, dt}
$$
- Solution:
$$
y(t) = \frac{1}{\mu(t)} \int \mu(t)f(t)\,dt
$$
- Higher-order linear ODEs solved using **characteristic polynomials**.

---

## 2.4 Summary of Important Results and Key Terms

- **Solution:** Function satisfying an ODE.
- **IVP:** ODE with an initial condition.
- **Existence & Uniqueness Theorem:** Specifies when solutions exist and are unique.
- **Phase Portrait:** Sketch showing qualitative behavior of solutions.
- **Separation of Variables:** Technique for solving separable ODEs.
- **Exact Equation:** ODE derived from a potential function.
- **Linear ODE:** ODE where \(y\) and its derivatives appear linearly.
- **Integrating Factor:** Technique for solving first-order linear ODEs.

---

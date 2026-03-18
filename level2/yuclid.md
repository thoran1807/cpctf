### Final Answer

**Flag:** `cpctf{10245,-4202}`

---

### Problem Understanding

The challenge provides two integers, p = 26513 and q = 64642, and asks us to find integers u and v such that:

p * u + q * v = gcd(p, q)

At first glance, this looks like a simple GCD problem, but the real task is to compute the coefficients u and v, which requires the Extended Euclidean Algorithm rather than just the standard GCD.

---

### Thought Process

The equation given is a classic form of Bézout's Identity, which states that for any integers a and b, there exist integers x and y such that:

ax + by = gcd(a, b)

This immediately suggests using the Extended Euclidean Algorithm (EEA).

First, I verified that the GCD of the two numbers is 1. Since they are coprime, a solution definitely exists.

Then, instead of brute forcing (which would be inefficient), I applied the Extended Euclidean Algorithm to compute the coefficients directly. This method not only finds the GCD but also tracks how to express it as a linear combination of the inputs.

---

### Tools and Techniques Used

- Python 3: Used to efficiently compute the Extended Euclidean Algorithm and avoid manual arithmetic errors

---

### Step-by-Step Solution

#### Step 1 — Initial Inspection

We are given:
- p = 26513
- q = 64642

The goal is to solve:
26513u + 64642v = gcd(26513, 64642)

First, compute the GCD using Euclid’s Algorithm:

```python
import math
math.gcd(26513, 64642)
```

Output:
1

This confirms the equation becomes:
26513u + 64642v = 1

---

#### Step 2 — Applying Extended Euclidean Algorithm

We now use EEA to find u and v:

```python
def egcd(a, b):
    if b == 0:
        return a, 1, 0
    g, x1, y1 = egcd(b, a % b)
    x = y1
    y = x1 - (a // b) * y1
    return g, x, y

g, u, v = egcd(26513, 64642)
print(u, v)
```

Output:
10245 -4202

---

#### Step 3 — Verification

To ensure correctness:

```python
26513 * 10245 + 64642 * (-4202)
```

Output:
1

This matches the GCD, confirming the solution is valid.

---

#### Step 4 — Flag

The required format is:

cpctf{u,v}

So the final flag is:

cpctf{10245,-4202}

---

### Why This Worked

- The problem directly maps to Bézout's Identity, making EEA the correct approach  
- Standard GCD only gives the value, but EEA provides the coefficients  
- Recursive backtracking in EEA reconstructs the linear combination  
- Python avoids arithmetic mistakes and speeds up computation  
- Since GCD = 1, a valid integer solution is guaranteed  

---

### Lessons Learned

- Always recognize patterns like ax + by = gcd(a,b) — they point to EEA  
- Don’t brute force linear combinations; use mathematical algorithms instead  
- Implementing EEA is a must-have skill for crypto and CTF challenges  
- Verifying the result is critical, especially in number theory problems  
- Python is extremely effective for quick validation and computation  

---

**Final Flag:** `cpctf{10245,-4202}`

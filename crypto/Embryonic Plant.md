## Skills involved: Basic math

This problem is simple but educational.

<details>
  <summary>Hint 1: </summary>
  
  Write down how the PRNG numbers are generated. What can you do?
</details>

<details>
  <summary> <b>Solution:</b> </summary>
  <br/>
  This is a striking example of how bad random number generators (RNGs) using linear recurrence relations can be.
  Only 5 RNG outputs are needed to complete gain insight to its inner workings.
  
  Consider the following systems of equations (variable names according to output.txt)

  ``` py
  s[1] = (s[0]*p+q) % r   # (1)
  s[2] = (s[1]*p+q) % r   # (2)
  s[3] = (s[2]*p+q) % r   # (3)
  s[4] = (s[3]*p+q) % r   # (4)
  ```

  We are going to eliminate q by subtracting `(1)` from `(2)`, etc.
  This is a very common technique in dealing with recurrence relations, sum of sequences, etc.
  Also for sake of convenience I'll substitute `s[i+1]-s[0]` with `d[i]`

  ``` py
  d[1] = (d[0]*p) % r   # (1a)
  d[2] = (d[1]*p) % r   # (2a)
  d[3] = (d[2]*p) % r   # (3a)
  ```

  We can now eliminate p:

  ``` py
  d[2]*(d[0]*p) = (d[1]*p)*d[1] % r   # (1a)*(2a)
  d[2]*d[0] = d[1]**2 % r             # p!=0 % r
  d[3]*d[1] = d[2]**2 % r             # similarly
  ```

  r can be calculated with `gcd(d[2]*d[0]-d[1]**2, d[3]*d[1]-d[2]**2)`. p and q can then be found trivially.
  
  **Key takeaway**: always use true random numbers if you aren't sure.
</details>

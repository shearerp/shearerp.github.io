<link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.css" integrity="sha384-AfEj0r4/OFrOo5t7NnNe46zW/tFgW6x/bCJG8FqQCEo3+Aro6EYUG4+cU+KJWu/X" crossorigin="anonymous">
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/katex.min.js" integrity="sha384-g7c+Jr9ZivxKLnZTDUhnkOnsh30B4H0rpLUpJ4jAIKs4fnJI+sEnkvrMWph2EDg4" crossorigin="anonymous"></script>
<script defer src="https://cdn.jsdelivr.net/npm/katex@0.12.0/dist/contrib/auto-render.min.js" integrity="sha384-mll67QQFJfxn0IYznZYonOWZ644AWYC+Pt2cHqMaRhXVrursRwvLnLaebdGIlYNa" crossorigin="anonymous"
    onload="renderMathInElement(document.body);"></script>

# The Maximum Subarray Problem & Kadane's Algorithm: a Mathematician's Perspective

The *maximum subarray problem* is a famous computer science problem with a delightful $$O(n)$$ dynamic programming solution known as *Kadane's algorithm.* It's one of those algorithms where the steps are very easy to follow, but the reason those steps work is a bit harder to understand. There are many blog posts that explain the algorithm, but none of the ones I found convinced me that the algorithm was *correct*. After thinking about it on and off for a few days, I derived the algorithm and proved its correctness with the argument I'll be sharing here.

**Maximum subarray problem:** given an array $$[a_1, a_2, \ldots, a_n]$$, find the largest possible sum that a contiguous subarray $$[a_{i+1}, \ldots, a_j]$$ can have. Writing the sum as $$s_{ij} = \sum_{k = i+1}^j a_k$$, we can write the problem as

$$s^* = \max_{i, j} s_{ij} = \max_{i, j} \sum_{k = i+1}^j a_k$$

Note that the empty array, with a sum of zero, is considered a valid subarray. We'll use the convention that the subarray is empty if the endpoints of the subarray are improperly ordered. In particular, $$s_{jj} = 0$$ is the sum over an empty subarray.

A standard trick in multivariable optimization problems is to restate them in a nested form, as an inner and outer problem:

$$ \max_{i,j} s_{ij} = \max_j \max_i s_{ij}$$

In the inner problem, the right endpoint $$j$$ is fixed, and we just need to find the left endpoint $$i$$ that maximizes the sum. If we can solve this problem for any given $$j$$, we just take the max over those solutions and that's the solution to our original problem. Let's define $$s^*_j = \max_i s_{ij}$$ as the solution to the inner problem.

How do we find $$s^*_j$$ for any given $$j$$? It feels like induction might work here, since the arrays for $$j - 1$$ and $$j$$ are very similar, differing only in the one element $$a_j$$. To use induction we need to do two things:

***Step 1:** Find the best subarray when $$j = 0$$.* The only option is the empty array, so $$s^*_0 = 0$$.

***Step 2:** Given the best subarray ending in $$j - 1$$, find the best subarray ending in $$j$$.* Consider two cases: the $$i$$ for this subarray is either (1) $$i = j$$, yielding the empty array, or (2) $$i < j$$, a nonempty array. If $$i < j$$, then $$a_j$$ is in the best subarray, and we need to make the best possible choice of elements to the left of $$a_j$$, that is, the best possible subarray ending in $$a_{j-1}$$. But because we're doing induction, we already have that subarray --- it's the one that yields the sum $$s^*_{j-1}$$, which is nonnegative since the smallest sum it could have is zero, for the empty array. Combining the $$s^*_{j-1}$$ subarray with $$a_j$$, we have $$s^*_j = s^*_{j-1} + a_j$$ in case 2. The max of the two cases is the best overall, yielding

$$s^*_j = \max(0, s^*_{j-1} + a_j)$$

Running the induction to its end, we obtain all the $$s^*_j$$. The max over these is the solution to the original problem:

$$
\begin{aligned}
s^*_0 &= 0 \\
s^*_j &= \max(0, s^*_{j-1} + a_j) \,\, \text{for $j = 1, 2, \ldots, n$} \\
s^*   &= \max_j s^*_j
\end{aligned}
$$

...which is precisely Kadane's algorithm.

## Other Versions

With minor adjustments, the argument above continues to work when faced with other versions of the problem.

### Non-Empty Array Version

What if the array cannot be empty? Then the base case of the induction is $$a_1$$, and the inductive step is slightly different: since the array ends in $$j$$ and must be non-empty, we can either choose $$a_j$$ by itself or concatenate it with the best subarray ending in $$j - 1$$. This argument leads to a slightly different algorithm:

$$
\begin{aligned}
s^*_1 &= a_1 \\
s^*_j &= \max(a_j, s^*_{j-1} + a_j) \,\, \text{for $j = 2, \ldots, n$} \\
s^*   &= \max_j s^*_j
\end{aligned}
$$

### Maximum Product Version

What if we want to find the non-empty subarray with the maximum *product*? The base case is once again $$a_1$$, but the inductive step presents a new wrinkle, as the best choice depends on the sign of $$a_j$$:

*Case 1: $$a_j > 0$$.* The best choice is to multiply $$a_j$$ with the maximum product subarray ending in $$j - 1$$.

*Case 2: $$a_j < 0$$.* The best choice is to multiply $$a_j$$ with the *minimum* (most negative) product subarray ending in $$j - 1$$. 

*Case 3: $$a_j = 0$$.* The product is zero no matter what we do, so we can just combine it into case 1 and make that case $$a_j \geq 0$$.

We must therefore keep track of both the maximum and minimum products of subarrays ending in $$j$$, which we denote $$p^+_j$$ and $$p^-_j$$.

$$
\begin{aligned}
p^+_1 &= p^-_1 = a_1 \\
p^+_j &= \max(a_j, a_j p^+_{j-1}) \,\,\text{if $a_j \geq 0$}\,\, \\
p^-_j &= \min(a_j, a_j p^-_{j-1}) \,\,\text{if $a_j \geq 0$}\,\, \\
p^+_j &= \max(a_j, a_j p^-_{j-1}) \,\,\text{if $a_j < 0$}\,\, \\
p^-_j &= \min(a_j, a_j p^+_{j-1}) \,\,\text{if $a_j < 0$}\,\, \\
\end{aligned}
$$

Then $$p^+ = \max_j p^+_j$$ is the maximum product.

**Exercise:** What happens if the maximum product array is allowed to be empty? (The empty product is conventionally defined as 1, the multiplicative identity, just as the empty sum is conventionally 0, the additive identity.)

**Exercise:** Adapt the argument to solve the [maximum profit problem](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/).

## Discussion

After writing this, I found out that Wikipedia's [page](https://en.wikipedia.org/wiki/Maximum_subarray_problem#Empty_subarrays_admitted) on the maximum (sum) subarray problem covers substantially the same argument as this post. But it's terse and reads more like an informal explanation for computer scientists than a mathematical proof. The approach taken here makes the core argument clearer, and variant problems become straightforward.

That said, this is a [dynamic programming](https://en.wikipedia.org/wiki/Dynamic_programming) problem, and computer science has some very helpful advice for knowing if you're dealing with such a problem. When we noticed that the $$j-1$$ and $$j$$ problems were very similar, we were essentially noticing that the problem seemed likely to have [overlapping subproblems](https://en.wikipedia.org/wiki/Overlapping_subproblems) --- a telltale sign of a fast dynamic programming solution. For best results when solving these problems, consider approaching them from both the computer science perspective of dynamic programming and overlapping subproblems, as well as the mathematical perspective of optimization and induction.

<!---


# The Brute Force Solution

Let's forget about $O(n)$ for the moment and try to come up with the fastest possible brute force solution. If we check all possibilities $i \leq j$, we'll be solving a lot of *overlapping subproblems*, for example, summing $a_2 + a_3 + a_4$ and $a_3 + a_4 + a_5$, a lot of redundant work. We can fix that by computing the prefix sums $s_{0i}$, so that any subarray can be summed in $O(1)$ time:

$$ s_{ij} = s_{0j} - s_{0i} $$

We cam get those prefix sums with an $O(n)$ cumulative sum, but we still have to search over $O(n^2)$ possibilities. More tricks will be needed for an $O(n)$ solution.
-->

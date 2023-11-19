



# Algo  ideology

## 算法思想 - 分治算法

> 分治算法的基本思想是将一个规模为N的问题分解为K个规模较小的子问题，这些子问题相互独立且与原问题性质相同。求出子问题的解，就可得到原问题的解。@pdai

[分治相关题目]()[给表达式加括号]()

[#](#分治相关题目) 分治相关题目

### [#](#给表达式加括号) 给表达式加括号

[241. Different Ways to Add Parentheses (Medium)在新窗口打开](https://leetcode.com/problems/different-ways-to-add-parentheses/description/)

```java
Input: "2-1-1".

((2-1)-1) = 0
(2-(1-1)) = 2

Output : [0, 2]
public List<Integer> diffWaysToCompute(String input) {
    List<Integer> ways = new ArrayList<>();
    for (int i = 0; i < input.length(); i++) {
        char c = input.charAt(i);
        if (c == '+' || c == '-' || c == '*') {
            List<Integer> left = diffWaysToCompute(input.substring(0, i));
            List<Integer> right = diffWaysToCompute(input.substring(i + 1));
            for (int l : left) {
                for (int r : right) {
                    switch (c) {
                        case '+':
                            ways.add(l + r);
                            break;
                        case '-':
                            ways.add(l - r);
                            break;
                        case '*':
                            ways.add(l * r);
                            break;
                    }
                }
            }
        }
    }
    if (ways.size() == 0) {
        ways.add(Integer.valueOf(input));
    }
    return ways;
}
```

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/algorithm/alg-core-divide-and-conquer.html





## 算法思想 - 动态规划算法

> 动态规划算法通常用于求解具有某种最优性质的问题。在这类问题中，可能会有许多可行解。每一个解都对应于一个值，我们希望找到具有最优值的解。动态规划算法与分治法类似，其基本思想也是将待求解问题分解成若干个子问题，先求解子问题，然后从这些子问题的解得到原问题的解。动态规划算法在算法思想中是极为重要的，需要重点掌握。@pdai

[动态规划相关题目]()[斐波那契数列]()[矩阵路径]()[数组区间]()[分割整数]()[最长递增子序列]()[最长公共子序列]()[0-1 背包]()[股票交易]()[字符串编辑]()

[#](#动态规划相关题目) 动态规划相关题目

递归和动态规划都是将原问题拆成多个子问题然后求解，他们之间最本质的区别是，动态规划保存了子问题的解，避免重复计算。

### [#](#斐波那契数列) 斐波那契数列

#### [#](#爬楼梯) 爬楼梯

[70. Climbing Stairs (Easy)在新窗口打开](https://leetcode.com/problems/climbing-stairs/description/)

题目描述: 有 N 阶楼梯，每次可以上一阶或者两阶，求有多少种上楼梯的方法。

定义一个数组 dp 存储上楼梯的方法数(为了方便讨论，数组下标从 1 开始)，dp[i] 表示走到第 i 个楼梯的方法数目。

第 i 个楼梯可以从第 i-1 和 i-2 个楼梯再走一步到达，走到第 i 个楼梯的方法数为走到第 i-1 和第 i-2 个楼梯的方法数之和。

![img](https://latex.codecogs.com/gif.latex?dp[i]=dp[i-1]+dp[i-2])

考虑到 dp[i] 只与 dp[i - 1] 和 dp[i - 2] 有关，因此可以只用两个变量来存储 dp[i - 1] 和 dp[i - 2]，使得原来的 O(N) 空间复杂度优化为 O(1) 复杂度。

```java
public int climbStairs(int n) {
    if (n <= 2) {
        return n;
    }
    int pre2 = 1, pre1 = 2;
    for (int i = 2; i < n; i++) {
        int cur = pre1 + pre2;
        pre2 = pre1;
        pre1 = cur;
    }
    return pre1;
}
```

#### [#](#强盗抢劫) 强盗抢劫

[198. House Robber (Easy)在新窗口打开](https://leetcode.com/problems/house-robber/description/)

题目描述: 抢劫一排住户，但是不能抢邻近的住户，求最大抢劫量。

定义 dp 数组用来存储最大的抢劫量，其中 dp[i] 表示抢到第 i 个住户时的最大抢劫量。

由于不能抢劫邻近住户，因此如果抢劫了第 i 个住户那么只能抢劫 i - 2 或者 i - 3 的住户，所以

![img](https://latex.codecogs.com/gif.latex?dp[i]=max(dp[i-2],dp[i-3])+nums[i])

```java
public int rob(int[] nums) {
    int n = nums.length;
    if (n == 0) {
        return 0;
    }
    if (n == 1) {
        return nums[0];
    }
    int pre3 = 0, pre2 = 0, pre1 = 0;
    for (int i = 0; i < n; i++) {
        int cur = Math.max(pre2, pre3) + nums[i];
        pre3 = pre2;
        pre2 = pre1;
        pre1 = cur;
    }
    return Math.max(pre1, pre2);
}
```

#### [#](#强盗在环形街区抢劫) 强盗在环形街区抢劫

[213. House Robber II (Medium)在新窗口打开](https://leetcode.com/problems/house-robber-ii/description/)

```java
public int rob(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    int n = nums.length;
    if (n == 1) {
        return nums[0];
    }
    return Math.max(rob(nums, 0, n - 2), rob(nums, 1, n - 1));
}

private int rob(int[] nums, int first, int last) {
    int pre3 = 0, pre2 = 0, pre1 = 0;
    for (int i = first; i <= last; i++) {
        int cur = Math.max(pre3, pre2) + nums[i];
        pre3 = pre2;
        pre2 = pre1;
        pre1 = cur;
    }
    return Math.max(pre2, pre1);
}
```

#### [#](#信件错排) 信件错排

题目描述: 有 N 个 信 和 信封，它们被打乱，求错误装信方式的数量。

定义一个数组 dp 存储错误方式数量，dp[i] 表示前 i 个信和信封的错误方式数量。假设第 i 个信装到第 j 个信封里面，而第 j 个信装到第 k 个信封里面。根据 i 和 k 是否相等，有两种情况:

- i==k，交换 i 和 k 的信后，它们的信和信封在正确的位置，但是其余 i-2 封信有 dp[i-2] 种错误装信的方式。由于 j 有 i-1 种取值，因此共有 (i-1)*dp[i-2] 种错误装信方式。
- i != k，交换 i 和 j 的信后，第 i 个信和信封在正确的位置，其余 i-1 封信有 dp[i-1] 种错误装信方式。由于 j 有 i-1 种取值，因此共有 (i-1)*dp[i-1] 种错误装信方式。

综上所述，错误装信数量方式数量为:

![img](https://latex.codecogs.com/gif.latex?dp[i]=(i-1)*dp[i-2]+(i-1)*dp[i-1])

#### [#](#母牛生产) 母牛生产

题目描述: 假设农场中成熟的母牛每年都会生 1 头小母牛，并且永远不会死。第一年有 1 只小母牛，从第二年开始，母牛开始生小母牛。每只小母牛 3 年之后成熟又可以生小母牛。给定整数 N，求 N 年后牛的数量。

第 i 年成熟的牛的数量为:

![img](https://latex.codecogs.com/gif.latex?dp[i]=dp[i-1]+dp[i-3])

### [#](#矩阵路径) 矩阵路径

#### [#](#矩阵的最小路径和) 矩阵的最小路径和

[64. Minimum Path Sum (Medium)在新窗口打开](https://leetcode.com/problems/minimum-path-sum/description/)

```html
[[1,3,1],
 [1,5,1],
 [4,2,1]]
Given the above grid map, return 7. Because the path 1→3→1→1→1 minimizes the sum.
```

题目描述: 求从矩阵的左上角到右下角的最小路径和，每次只能向右和向下移动。

```java
public int minPathSum(int[][] grid) {
    if (grid.length == 0 || grid[0].length == 0) {
        return 0;
    }
    int m = grid.length, n = grid[0].length;
    int[] dp = new int[n];
    for (int i = 0; i < m; i++) {
        for (int j = 0; j < n; j++) {
            if (i == 0) {
                dp[j] = dp[j - 1];
            } else {
                dp[j] = Math.min(dp[j - 1], dp[j]);
            }
            dp[j] += grid[i][j];
        }
    }
    return dp[n - 1];
}
```

#### [#](#矩阵的总路径数) 矩阵的总路径数

[62. Unique Paths (Medium)在新窗口打开](https://leetcode.com/problems/unique-paths/description/)

题目描述: 统计从矩阵左上角到右下角的路径总数，每次只能向右或者向下移动。

![image](/images/pics/7c98e1b6-c446-4cde-8513-5c11b9f52aea.jpg)

```java
public int uniquePaths(int m, int n) {
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    for (int i = 1; i < m; i++) {
        for (int j = 1; j < n; j++) {
            dp[j] = dp[j] + dp[j - 1];
        }
    }
    return dp[n - 1];
}
```

也可以直接用数学公式求解，这是一个组合问题。机器人总共移动的次数 S=m+n-2，向下移动的次数 D=m-1，那么问题可以看成从 S 从取出 D 个位置的组合数量，这个问题的解为 C(S, D)。

```java
public int uniquePaths(int m, int n) {
    int S = m + n - 2;  // 总共的移动次数
    int D = m - 1;      // 向下的移动次数
    long ret = 1;
    for (int i = 1; i <= D; i++) {
        ret = ret * (S - D + i) / i;
    }
    return (int) ret;
}
```

### [#](#数组区间) 数组区间

#### [#](#数组区间和) 数组区间和

[303. Range Sum Query - Immutable (Easy)在新窗口打开](https://leetcode.com/problems/range-sum-query-immutable/description/)

```html
Given nums = [-2, 0, 3, -5, 2, -1]

sumRange(0, 2) -> 1
sumRange(2, 5) -> -1
sumRange(0, 5) -> -3
```

求区间 i ~ j 的和，可以转换为 sum[j] - sum[i-1]，其中 sum[i] 为 0 ~ i 的和。

```java
class NumArray {

    private int[] sums;

    public NumArray(int[] nums) {
        sums = new int[nums.length + 1];
        for (int i = 1; i <= nums.length; i++) {
            sums[i] = sums[i - 1] + nums[i - 1];
        }
    }

    public int sumRange(int i, int j) {
        return sums[j + 1] - sums[i];
    }
}
```

#### [#](#子数组最大的和) 子数组最大的和

[53. Maximum Subarray (Easy)在新窗口打开](https://leetcode.com/problems/maximum-subarray/description/)

```html
For example, given the array [-2,1,-3,4,-1,2,1,-5,4],
the contiguous subarray [4,-1,2,1] has the largest sum = 6.
public int maxSubArray(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    int preSum = nums[0];
    int maxSum = preSum;
    for (int i = 1; i < nums.length; i++) {
        preSum = preSum > 0 ? preSum + nums[i] : nums[i];
        maxSum = Math.max(maxSum, preSum);
    }
    return maxSum;
}
```

#### [#](#数组中等差递增子区间的个数) 数组中等差递增子区间的个数

[413. Arithmetic Slices (Medium)在新窗口打开](https://leetcode.com/problems/arithmetic-slices/description/)

```html
A = [1, 2, 3, 4]
return: 3, for 3 arithmetic slices in A: [1, 2, 3], [2, 3, 4] and [1, 2, 3, 4] itself.
```

dp[i] 表示以 A[i] 为结尾的等差递增子区间的个数。

在 A[i] - A[i - 1] == A[i - 1] - A[i - 2] 的条件下，{A[i - 2], A[i - 1], A[i]} 是一个等差递增子区间。如果 {A[i - 3], A[i - 2], A[i - 1]} 是一个等差递增子区间，那么 {A[i - 3], A[i - 2], A[i - 1], A[i]} 也是等差递增子区间，dp[i] = dp[i-1] + 1。

```java
public int numberOfArithmeticSlices(int[] A) {
    if (A == null || A.length == 0) {
        return 0;
    }
    int n = A.length;
    int[] dp = new int[n];
    for (int i = 2; i < n; i++) {
        if (A[i] - A[i - 1] == A[i - 1] - A[i - 2]) {
            dp[i] = dp[i - 1] + 1;
        }
    }
    int total = 0;
    for (int cnt : dp) {
        total += cnt;
    }
    return total;
}
```

### [#](#分割整数) 分割整数

#### [#](#分割整数的最大乘积) 分割整数的最大乘积

[343. Integer Break (Medim)在新窗口打开](https://leetcode.com/problems/integer-break/description/)

题目描述: For example, given n = 2, return 1 (2 = 1 + 1); given n = 10, return 36 (10 = 3 + 3 + 4).

```java
public int integerBreak(int n) {
    int[] dp = new int[n + 1];
    dp[1] = 1;
    for (int i = 2; i <= n; i++) {
        for (int j = 1; j <= i - 1; j++) {
            dp[i] = Math.max(dp[i], Math.max(j * dp[i - j], j * (i - j)));
        }
    }
    return dp[n];
}
```

#### [#](#按平方数来分割整数) 按平方数来分割整数

[279. Perfect Squares(Medium)在新窗口打开](https://leetcode.com/problems/perfect-squares/description/)

题目描述: For example, given n = 12, return 3 because 12 = 4 + 4 + 4; given n = 13, return 2 because 13 = 4 + 9.

```java
public int numSquares(int n) {
    List<Integer> squareList = generateSquareList(n);
    int[] dp = new int[n + 1];
    for (int i = 1; i <= n; i++) {
        int min = Integer.MAX_VALUE;
        for (int square : squareList) {
            if (square > i) {
                break;
            }
            min = Math.min(min, dp[i - square] + 1);
        }
        dp[i] = min;
    }
    return dp[n];
}

private List<Integer> generateSquareList(int n) {
    List<Integer> squareList = new ArrayList<>();
    int diff = 3;
    int square = 1;
    while (square <= n) {
        squareList.add(square);
        square += diff;
        diff += 2;
    }
    return squareList;
}
```

#### [#](#分割整数构成字母字符串) 分割整数构成字母字符串

[91. Decode Ways (Medium)在新窗口打开](https://leetcode.com/problems/decode-ways/description/)

题目描述: Given encoded message "12", it could be decoded as "AB" (1 2) or "L" (12).

```java
public int numDecodings(String s) {
    if (s == null || s.length() == 0) {
        return 0;
    }
    int n = s.length();
    int[] dp = new int[n + 1];
    dp[0] = 1;
    dp[1] = s.charAt(0) == '0' ? 0 : 1;
    for (int i = 2; i <= n; i++) {
        int one = Integer.valueOf(s.substring(i - 1, i));
        if (one != 0) {
            dp[i] += dp[i - 1];
        }
        if (s.charAt(i - 2) == '0') {
            continue;
        }
        int two = Integer.valueOf(s.substring(i - 2, i));
        if (two <= 26) {
            dp[i] += dp[i - 2];
        }
    }
    return dp[n];
}
```

### [#](#最长递增子序列) 最长递增子序列

已知一个序列 {S1, S2,...,Sn}，取出若干数组成新的序列 {Si1, Si2,..., Sim}，其中 i1、i2 ... im 保持递增，即新序列中各个数仍然保持原数列中的先后顺序，称新序列为原序列的一个 子序列 。

如果在子序列中，当下标 ix > iy 时，Six > Siy，称子序列为原序列的一个 递增子序列 。

定义一个数组 dp 存储最长递增子序列的长度，dp[n] 表示以 Sn 结尾的序列的最长递增子序列长度。对于一个递增子序列 {Si1, Si2,...,Sim}，如果 im < n 并且 Sim < Sn，此时 {Si1, Si2,..., Sim, Sn} 为一个递增子序列，递增子序列的长度增加 1。满足上述条件的递增子序列中，长度最长的那个递增子序列就是要找的，在长度最长的递增子序列上加上 Sn 就构成了以 Sn 为结尾的最长递增子序列。因此 dp[n] = max{ dp[i]+1 | Si < Sn && i < n} 。

因为在求 dp[n] 时可能无法找到一个满足条件的递增子序列，此时 {Sn} 就构成了递增子序列，需要对前面的求解方程做修改，令 dp[n] 最小为 1，即:

![img](https://latex.codecogs.com/gif.latex?dp[n]=max{1,dp[i]+1|S_i<S_n&&i<n})

对于一个长度为 N 的序列，最长递增子序列并不一定会以 SN 为结尾，因此 dp[N] 不是序列的最长递增子序列的长度，需要遍历 dp 数组找出最大值才是所要的结果，max{ dp[i] | 1 <= i <= N} 即为所求。

#### [#](#最长递增子序列-1) 最长递增子序列

[300. Longest Increasing Subsequence (Medium)在新窗口打开](https://leetcode.com/problems/longest-increasing-subsequence/description/)

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] dp = new int[n];
    for (int i = 0; i < n; i++) {
        int max = 1;
        for (int j = 0; j < i; j++) {
            if (nums[i] > nums[j]) {
                max = Math.max(max, dp[j] + 1);
            }
        }
        dp[i] = max;
    }
    return Arrays.stream(dp).max().orElse(0);
}
```

使用 Stream 求最大值会导致运行时间过长，可以改成以下形式:

```java
int ret = 0;
for (int i = 0; i < n; i++) {
    ret = Math.max(ret, dp[i]);
}
return ret;
```

以上解法的时间复杂度为 O(N2)，可以使用二分查找将时间复杂度降低为 O(NlogN)。

定义一个 tails 数组，其中 tails[i] 存储长度为 i + 1 的最长递增子序列的最后一个元素。对于一个元素 x，

- 如果它大于 tails 数组所有的值，那么把它添加到 tails 后面，表示最长递增子序列长度加 1；
- 如果 tails[i-1] < x <= tails[i]，那么更新 tails[i-1] = x。

例如对于数组 [4,3,6,5]，有:

```html
tails      len      num
[]         0        4
[4]        1        3
[3]        1        6
[3,6]      2        5
[3,5]      2        null
```

可以看出 tails 数组保持有序，因此在查找 Si 位于 tails 数组的位置时就可以使用二分查找。

```java
public int lengthOfLIS(int[] nums) {
    int n = nums.length;
    int[] tails = new int[n];
    int len = 0;
    for (int num : nums) {
        int index = binarySearch(tails, len, num);
        tails[index] = num;
        if (index == len) {
            len++;
        }
    }
    return len;
}

private int binarySearch(int[] tails, int len, int key) {
    int l = 0, h = len;
    while (l < h) {
        int mid = l + (h - l) / 2;
        if (tails[mid] == key) {
            return mid;
        } else if (tails[mid] > key) {
            h = mid;
        } else {
            l = mid + 1;
        }
    }
    return l;
}
```

#### [#](#一组整数对能够构成的最长链) 一组整数对能够构成的最长链

[646. Maximum Length of Pair Chain (Medium)在新窗口打开](https://leetcode.com/problems/maximum-length-of-pair-chain/description/)

```html
Input: [[1,2], [2,3], [3,4]]
Output: 2
Explanation: The longest chain is [1,2] -> [3,4]
```

题目描述: 对于 (a, b) 和 (c, d) ，如果 b < c，则它们可以构成一条链。

```java
public int findLongestChain(int[][] pairs) {
    if (pairs == null || pairs.length == 0) {
        return 0;
    }
    Arrays.sort(pairs, (a, b) -> (a[0] - b[0]));
    int n = pairs.length;
    int[] dp = new int[n];
    Arrays.fill(dp, 1);
    for (int i = 1; i < n; i++) {
        for (int j = 0; j < i; j++) {
            if (pairs[j][1] < pairs[i][0]) {
                dp[i] = Math.max(dp[i], dp[j] + 1);
            }
        }
    }
    return Arrays.stream(dp).max().orElse(0);
}
```

#### [#](#最长摆动子序列) 最长摆动子序列

[376. Wiggle Subsequence (Medium)在新窗口打开](https://leetcode.com/problems/wiggle-subsequence/description/)

```html
Input: [1,7,4,9,2,5]
Output: 6
The entire sequence is a wiggle sequence.

Input: [1,17,5,10,13,15,10,5,16,8]
Output: 7
There are several subsequences that achieve this length. One is [1,17,10,13,10,16,8].

Input: [1,2,3,4,5,6,7,8,9]
Output: 2
```

要求: 使用 O(N) 时间复杂度求解。

```java
public int wiggleMaxLength(int[] nums) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    int up = 1, down = 1;
    for (int i = 1; i < nums.length; i++) {
        if (nums[i] > nums[i - 1]) {
            up = down + 1;
        } else if (nums[i] < nums[i - 1]) {
            down = up + 1;
        }
    }
    return Math.max(up, down);
}
```

### [#](#最长公共子序列) 最长公共子序列

对于两个子序列 S1 和 S2，找出它们最长的公共子序列。

定义一个二维数组 dp 用来存储最长公共子序列的长度，其中 dp[i][j] 表示 S1 的前 i 个字符与 S2 的前 j 个字符最长公共子序列的长度。考虑 S1i 与 S2j 值是否相等，分为两种情况:

- 当 S1i==S2j 时，那么就能在 S1 的前 i-1 个字符与 S2 的前 j-1 个字符最长公共子序列的基础上再加上 S1i 这个值，最长公共子序列长度加 1，即 dp[i][j] = dp[i-1][j-1] + 1。
- 当 S1i != S2j 时，此时最长公共子序列为 S1 的前 i-1 个字符和 S2 的前 j 个字符最长公共子序列，或者 S1 的前 i 个字符和 S2 的前 j-1 个字符最长公共子序列，取它们的最大者，即 dp[i][j] = max{ dp[i-1][j], dp[i][j-1] }。

综上，最长公共子序列的状态转移方程为:

![img](https://latex.codecogs.com/gif.latex?dp[i][j]=\left{\begin{array}{rcl}dp[i-1][j-1]&&{S1_i==S2_j}\max(dp[i-1][j],dp[i][j-1])&&{S1_i<>S2_j}\end{array}\right.)

对于长度为 N 的序列 S1 和长度为 M 的序列 S2，dp[N][M] 就是序列 S1 和序列 S2 的最长公共子序列长度。

与最长递增子序列相比，最长公共子序列有以下不同点:

- 针对的是两个序列，求它们的最长公共子序列。
- 在最长递增子序列中，dp[i] 表示以 Si 为结尾的最长递增子序列长度，子序列必须包含 Si ；在最长公共子序列中，dp[i][j] 表示 S1 中前 i 个字符与 S2 中前 j 个字符的最长公共子序列长度，不一定包含 S1i 和 S2j。
- 在求最终解时，最长公共子序列中 dp[N][M] 就是最终解，而最长递增子序列中 dp[N] 不是最终解，因为以 SN 为结尾的最长递增子序列不一定是整个序列最长递增子序列，需要遍历一遍 dp 数组找到最大者。

```java
public int lengthOfLCS(int[] nums1, int[] nums2) {
    int n1 = nums1.length, n2 = nums2.length;
    int[][] dp = new int[n1 + 1][n2 + 1];
    for (int i = 1; i <= n1; i++) {
        for (int j = 1; j <= n2; j++) {
            if (nums1[i - 1] == nums2[j - 1]) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i][j - 1]);
            }
        }
    }
    return dp[n1][n2];
}
```

### [#](#_0-1-背包) 0-1 背包

有一个容量为 N 的背包，要用这个背包装下物品的价值最大，这些物品有两个属性: 体积 w 和价值 v。

定义一个二维数组 dp 存储最大价值，其中 dp[i][j] 表示前 i 件物品体积不超过 j 的情况下能达到的最大价值。设第 i 件物品体积为 w，价值为 v，根据第 i 件物品是否添加到背包中，可以分两种情况讨论:

- 第 i 件物品没添加到背包，总体积不超过 j 的前 i 件物品的最大价值就是总体积不超过 j 的前 i-1 件物品的最大价值，dp[i][j] = dp[i-1][j]。
- 第 i 件物品添加到背包中，dp[i][j] = dp[i-1][j-w] + v。

第 i 件物品可添加也可以不添加，取决于哪种情况下最大价值更大。因此，0-1 背包的状态转移方程为:

![img](https://latex.codecogs.com/gif.latex?dp[i][j]=max(dp[i-1][j],dp[i-1][j-w]+v))

```java
public int knapsack(int W, int N, int[] weights, int[] values) {
    int[][] dp = new int[N + 1][W + 1];
    for (int i = 1; i <= N; i++) {
        int w = weights[i - 1], v = values[i - 1];
        for (int j = 1; j <= W; j++) {
            if (j >= w) {
                dp[i][j] = Math.max(dp[i - 1][j], dp[i - 1][j - w] + v);
            } else {
                dp[i][j] = dp[i - 1][j];
            }
        }
    }
    return dp[N][W];
}
```

#### [#](#空间优化) 空间优化

在程序实现时可以对 0-1 背包做优化。观察状态转移方程可以知道，前 i 件物品的状态仅与前 i-1 件物品的状态有关，因此可以将 dp 定义为一维数组，其中 dp[j] 既可以表示 dp[i-1][j] 也可以表示 dp[i][j]。此时，

![img](https://latex.codecogs.com/gif.latex?dp[j]=max(dp[j],dp[j-w]+v))

因为 dp[j-w] 表示 dp[i-1][j-w]，因此不能先求 dp[i][j-w]，以防将 dp[i-1][j-w] 覆盖。也就是说要先计算 dp[i][j] 再计算 dp[i][j-w]，在程序实现时需要按倒序来循环求解。

```java
public int knapsack(int W, int N, int[] weights, int[] values) {
    int[] dp = new int[W + 1];
    for (int i = 1; i <= N; i++) {
        int w = weights[i - 1], v = values[i - 1];
        for (int j = W; j >= 1; j--) {
            if (j >= w) {
                dp[j] = Math.max(dp[j], dp[j - w] + v);
            }
        }
    }
    return dp[W];
}
```

无法使用贪心算法的解释

0-1 背包问题无法使用贪心算法来求解，也就是说不能按照先添加性价比最高的物品来达到最优，这是因为这种方式可能造成背包空间的浪费，从而无法达到最优。考虑下面的物品和一个容量为 5 的背包，如果先添加物品 0 再添加物品 1，那么只能存放的价值为 16，浪费了大小为 2 的空间。最优的方式是存放物品 1 和物品 2，价值为 22.

| id   | w    | v    | v/w  |
| ---- | ---- | ---- | ---- |
| 0    | 1    | 6    | 6    |
| 1    | 2    | 10   | 5    |
| 2    | 3    | 12   | 4    |

变种

- 完全背包: 物品数量为无限个
- 多重背包: 物品数量有限制
- 多维费用背包: 物品不仅有重量，还有体积，同时考虑这两种限制
- 其它: 物品之间相互约束或者依赖

#### [#](#划分数组为和相等的两部分) 划分数组为和相等的两部分

[416. Partition Equal Subset Sum (Medium)在新窗口打开](https://leetcode.com/problems/partition-equal-subset-sum/description/)

```html
Input: [1, 5, 11, 5]

Output: true

Explanation: The array can be partitioned as [1, 5, 5] and [11].
```

可以看成一个背包大小为 sum/2 的 0-1 背包问题。

```java
public boolean canPartition(int[] nums) {
    int sum = computeArraySum(nums);
    if (sum % 2 != 0) {
        return false;
    }
    int W = sum / 2;
    boolean[] dp = new boolean[W + 1];
    dp[0] = true;
    Arrays.sort(nums);
    for (int num : nums) {                 // 0-1 背包一个物品只能用一次
        for (int i = W; i >= num; i--) {   // 从后往前，先计算 dp[i] 再计算 dp[i-num]
            dp[i] = dp[i] || dp[i - num];
        }
    }
    return dp[W];
}

private int computeArraySum(int[] nums) {
    int sum = 0;
    for (int num : nums) {
        sum += num;
    }
    return sum;
}
```

#### [#](#改变一组数的正负号使得它们的和为一给定数) 改变一组数的正负号使得它们的和为一给定数

[494. Target Sum (Medium)在新窗口打开](https://leetcode.com/problems/target-sum/description/)

```html
Input: nums is [1, 1, 1, 1, 1], S is 3.
Output: 5
Explanation:

-1+1+1+1+1 = 3
+1-1+1+1+1 = 3
+1+1-1+1+1 = 3
+1+1+1-1+1 = 3
+1+1+1+1-1 = 3

There are 5 ways to assign symbols to make the sum of nums be target 3.
```

该问题可以转换为 Subset Sum 问题，从而使用 0-1 背包的方法来求解。

可以将这组数看成两部分，P 和 N，其中 P 使用正号，N 使用负号，有以下推导:

```html
                  sum(P) - sum(N) = target
sum(P) + sum(N) + sum(P) - sum(N) = target + sum(P) + sum(N)
                       2 * sum(P) = target + sum(nums)
```

因此只要找到一个子集，令它们都取正号，并且和等于 (target + sum(nums))/2，就证明存在解。

```java
public int findTargetSumWays(int[] nums, int S) {
    int sum = computeArraySum(nums);
    if (sum < S || (sum + S) % 2 == 1) {
        return 0;
    }
    int W = (sum + S) / 2;
    int[] dp = new int[W + 1];
    dp[0] = 1;
    Arrays.sort(nums);
    for (int num : nums) {
        for (int i = W; i >= num; i--) {
            dp[i] = dp[i] + dp[i - num];
        }
    }
    return dp[W];
}

private int computeArraySum(int[] nums) {
    int sum = 0;
    for (int num : nums) {
        sum += num;
    }
    return sum;
}
```

DFS 解法:

```java
public int findTargetSumWays(int[] nums, int S) {
    return findTargetSumWays(nums, 0, S);
}

private int findTargetSumWays(int[] nums, int start, int S) {
    if (start == nums.length) {
        return S == 0 ? 1 : 0;
    }
    return findTargetSumWays(nums, start + 1, S + nums[start])
            + findTargetSumWays(nums, start + 1, S - nums[start]);
}
```

#### [#](#字符串按单词列表分割) 字符串按单词列表分割

[139. Word Break (Medium)在新窗口打开](https://leetcode.com/problems/word-break/description/)

```html
s = "leetcode",
dict = ["leet", "code"].
Return true because "leetcode" can be segmented as "leet code".
```

dict 中的单词没有使用次数的限制，因此这是一个完全背包问题。

0-1 背包和完全背包在实现上的不同之处是，0-1 背包对物品的迭代是在最外层，而完全背包对物品的迭代是在最里层。

```java
public boolean wordBreak(String s, List<String> wordDict) {
    int n = s.length();
    boolean[] dp = new boolean[n + 1];
    dp[0] = true;
    for (int i = 1; i <= n; i++) {
        for (String word : wordDict) {   // 完全一个物品可以使用多次
            int len = word.length();
            if (len <= i && word.equals(s.substring(i - len, i))) {
                dp[i] = dp[i] || dp[i - len];
            }
        }
    }
    return dp[n];
}
```

#### [#](#_01-字符构成最多的字符串) 01 字符构成最多的字符串

[474. Ones and Zeroes (Medium)在新窗口打开](https://leetcode.com/problems/ones-and-zeroes/description/)

```html
Input: Array = {"10", "0001", "111001", "1", "0"}, m = 5, n = 3
Output: 4

Explanation: There are totally 4 strings can be formed by the using of 5 0s and 3 1s, which are "10","0001","1","0"
```

这是一个多维费用的 0-1 背包问题，有两个背包大小，0 的数量和 1 的数量。

```java
public int findMaxForm(String[] strs, int m, int n) {
    if (strs == null || strs.length == 0) {
        return 0;
    }
    int[][] dp = new int[m + 1][n + 1];
    for (String s : strs) {    // 每个字符串只能用一次
        int ones = 0, zeros = 0;
        for (char c : s.toCharArray()) {
            if (c == '0') {
                zeros++;
            } else {
                ones++;
            }
        }
        for (int i = m; i >= zeros; i--) {
            for (int j = n; j >= ones; j--) {
                dp[i][j] = Math.max(dp[i][j], dp[i - zeros][j - ones] + 1);
            }
        }
    }
    return dp[m][n];
}
```

#### [#](#找零钱的最少硬币数) 找零钱的最少硬币数

[322. Coin Change (Medium)在新窗口打开](https://leetcode.com/problems/coin-change/description/)

```html
Example 1:
coins = [1, 2, 5], amount = 11
return 3 (11 = 5 + 5 + 1)

Example 2:
coins = [2], amount = 3
return -1.
```

题目描述: 给一些面额的硬币，要求用这些硬币来组成给定面额的钱数，并且使得硬币数量最少。硬币可以重复使用。

- 物品: 硬币
- 物品大小: 面额
- 物品价值: 数量

因为硬币可以重复使用，因此这是一个完全背包问题。

```java
public int coinChange(int[] coins, int amount) {
    if (coins == null || coins.length == 0) {
        return 0;
    }
    int[] minimum = new int[amount + 1];
    Arrays.fill(minimum, amount + 1);
    minimum[0] = 0;
    Arrays.sort(coins);
    for (int i = 1; i <= amount; i++) {
        for (int j = 0; j < coins.length && coins[j] <= i; j++) {
            minimum[i] = Math.min(minimum[i], minimum[i - coins[j]] + 1);
        }
    }
    return minimum[amount] > amount ? -1 : minimum[amount];
}
```

#### [#](#组合总和) 组合总和

[377. Combination Sum IV (Medium)在新窗口打开](https://leetcode.com/problems/combination-sum-iv/description/)

```html
nums = [1, 2, 3]
target = 4

The possible combination ways are:
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)

Note that different sequences are counted as different combinations.

Therefore the output is 7.
```

#### [#](#完全背包。) 完全背包。

```java
public int combinationSum4(int[] nums, int target) {
    if (nums == null || nums.length == 0) {
        return 0;
    }
    int[] maximum = new int[target + 1];
    maximum[0] = 1;
    Arrays.sort(nums);
    for (int i = 1; i <= target; i++) {
        for (int j = 0; j < nums.length && nums[j] <= i; j++) {
            maximum[i] += maximum[i - nums[j]];
        }
    }
    return maximum[target];
}
```

### [#](#股票交易) 股票交易

#### [#](#需要冷却期的股票交易) 需要冷却期的股票交易

[309. Best Time to Buy and Sell Stock with Cooldown(Medium)在新窗口打开](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/description/)

题目描述: 交易之后需要有一天的冷却时间。

![image](/images/pics/a3da4342-078b-43e2-b748-7e71bec50dc4.png)

```java
public int maxProfit(int[] prices) {
    if (prices == null || prices.length == 0) {
        return 0;
    }
    int N = prices.length;
    int[] buy = new int[N];
    int[] s1 = new int[N];
    int[] sell = new int[N];
    int[] s2 = new int[N];
    s1[0] = buy[0] = -prices[0];
    sell[0] = s2[0] = 0;
    for (int i = 1; i < N; i++) {
        buy[i] = s2[i - 1] - prices[i];
        s1[i] = Math.max(buy[i - 1], s1[i - 1]);
        sell[i] = Math.max(buy[i - 1], s1[i - 1]) + prices[i];
        s2[i] = Math.max(s2[i - 1], sell[i - 1]);
    }
    return Math.max(sell[N - 1], s2[N - 1]);
}
```

#### [#](#需要交易费用的股票交易) 需要交易费用的股票交易

[714. Best Time to Buy and Sell Stock with Transaction Fee (Medium)在新窗口打开](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-with-transaction-fee/description/)

```html
Input: prices = [1, 3, 2, 8, 4, 9], fee = 2
Output: 8
Explanation: The maximum profit can be achieved by:
Buying at prices[0] = 1
Selling at prices[3] = 8
Buying at prices[4] = 4
Selling at prices[5] = 9
The total profit is ((8 - 1) - 2) + ((9 - 4) - 2) = 8.
```

题目描述: 每交易一次，都要支付一定的费用。

![image](/images/pics/61942711-45a0-4e11-bbc9-434e31436f33.png)

```java
public int maxProfit(int[] prices, int fee) {
    int N = prices.length;
    int[] buy = new int[N];
    int[] s1 = new int[N];
    int[] sell = new int[N];
    int[] s2 = new int[N];
    s1[0] = buy[0] = -prices[0];
    sell[0] = s2[0] = 0;
    for (int i = 1; i < N; i++) {
        buy[i] = Math.max(sell[i - 1], s2[i - 1]) - prices[i];
        s1[i] = Math.max(buy[i - 1], s1[i - 1]);
        sell[i] = Math.max(buy[i - 1], s1[i - 1]) - fee + prices[i];
        s2[i] = Math.max(s2[i - 1], sell[i - 1]);
    }
    return Math.max(sell[N - 1], s2[N - 1]);
}
```

#### [#](#买入和售出股票最大的收益) 买入和售出股票最大的收益

[121. Best Time to Buy and Sell Stock (Easy)在新窗口打开](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/description/)

题目描述: 只进行一次交易。

只要记录前面的最小价格，将这个最小价格作为买入价格，然后将当前的价格作为售出价格，查看当前收益是不是最大收益。

```java
public int maxProfit(int[] prices) {
    int n = prices.length;
    if (n == 0) return 0;
    int soFarMin = prices[0];
    int max = 0;
    for (int i = 1; i < n; i++) {
        if (soFarMin > prices[i]) soFarMin = prices[i];
        else max = Math.max(max, prices[i] - soFarMin);
    }
    return max;
}
```

#### [#](#只能进行两次的股票交易) 只能进行两次的股票交易

[123. Best Time to Buy and Sell Stock III (Hard)在新窗口打开](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iii/description/)

```java
public int maxProfit(int[] prices) {
    int firstBuy = Integer.MIN_VALUE, firstSell = 0;
    int secondBuy = Integer.MIN_VALUE, secondSell = 0;
    for (int curPrice : prices) {
        if (firstBuy < -curPrice) {
            firstBuy = -curPrice;
        }
        if (firstSell < firstBuy + curPrice) {
            firstSell = firstBuy + curPrice;
        }
        if (secondBuy < firstSell - curPrice) {
            secondBuy = firstSell - curPrice;
        }
        if (secondSell < secondBuy + curPrice) {
            secondSell = secondBuy + curPrice;
        }
    }
    return secondSell;
}
```

#### [#](#只能进行-k-次的股票交易) 只能进行 k 次的股票交易

[188. Best Time to Buy and Sell Stock IV (Hard)在新窗口打开](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-iv/description/)

```java
public int maxProfit(int k, int[] prices) {
    int n = prices.length;
    if (k >= n / 2) {   // 这种情况下该问题退化为普通的股票交易问题
        int maxProfit = 0;
        for (int i = 1; i < n; i++) {
            if (prices[i] > prices[i - 1]) {
                maxProfit += prices[i] - prices[i - 1];
            }
        }
        return maxProfit;
    }
    int[][] maxProfit = new int[k + 1][n];
    for (int i = 1; i <= k; i++) {
        int localMax = maxProfit[i - 1][0] - prices[0];
        for (int j = 1; j < n; j++) {
            maxProfit[i][j] = Math.max(maxProfit[i][j - 1], prices[j] + localMax);
            localMax = Math.max(localMax, maxProfit[i - 1][j] - prices[j]);
        }
    }
    return maxProfit[k][n - 1];
}
```

### [#](#字符串编辑) 字符串编辑

#### [#](#删除两个字符串的字符使它们相等) 删除两个字符串的字符使它们相等

[583. Delete Operation for Two Strings (Medium)在新窗口打开](https://leetcode.com/problems/delete-operation-for-two-strings/description/)

```html
Input: "sea", "eat"
Output: 2
Explanation: You need one step to make "sea" to "ea" and another step to make "eat" to "ea".
```

可以转换为求两个字符串的最长公共子序列问题。

```java
public int minDistance(String word1, String word2) {
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1] + 1;
            } else {
                dp[i][j] = Math.max(dp[i][j - 1], dp[i - 1][j]);
            }
        }
    }
    return m + n - 2 * dp[m][n];
}
```

#### [#](#编辑距离) 编辑距离

[72. Edit Distance (Hard)在新窗口打开](https://leetcode.com/problems/edit-distance/description/)

```html
Example 1:

Input: word1 = "horse", word2 = "ros"
Output: 3
Explanation:
horse -> rorse (replace 'h' with 'r')
rorse -> rose (remove 'r')
rose -> ros (remove 'e')
Example 2:

Input: word1 = "intention", word2 = "execution"
Output: 5
Explanation:
intention -> inention (remove 't')
inention -> enention (replace 'i' with 'e')
enention -> exention (replace 'n' with 'x')
exention -> exection (replace 'n' with 'c')
exection -> execution (insert 'u')
```

题目描述: 修改一个字符串成为另一个字符串，使得修改次数最少。一次修改操作包括: 插入一个字符、删除一个字符、替换一个字符。

```java
public int minDistance(String word1, String word2) {
    if (word1 == null || word2 == null) {
        return 0;
    }
    int m = word1.length(), n = word2.length();
    int[][] dp = new int[m + 1][n + 1];
    for (int i = 1; i <= m; i++) {
        dp[i][0] = i;
    }
    for (int i = 1; i <= n; i++) {
        dp[0][i] = i;
    }
    for (int i = 1; i <= m; i++) {
        for (int j = 1; j <= n; j++) {
            if (word1.charAt(i - 1) == word2.charAt(j - 1)) {
                dp[i][j] = dp[i - 1][j - 1];
            } else {
                dp[i][j] = Math.min(dp[i - 1][j - 1], Math.min(dp[i][j - 1], dp[i - 1][j])) + 1;
            }
        }
    }
    return dp[m][n];
}
```

#### [#](#复制粘贴字符) 复制粘贴字符

[650. 2 Keys Keyboard (Medium)在新窗口打开](https://leetcode.com/problems/2-keys-keyboard/description/)

题目描述: 最开始只有一个字符 A，问需要多少次操作能够得到 n 个字符 A，每次操作可以复制当前所有的字符，或者粘贴。

```text
Input: 3
Output: 3
Explanation:
Intitally, we have one character 'A'.
In step 1, we use Copy All operation.
In step 2, we use Paste operation to get 'AA'.
In step 3, we use Paste operation to get 'AAA'.
public int minSteps(int n) {
    if (n == 1) return 0;
    for (int i = 2; i <= Math.sqrt(n); i++) {
        if (n % i == 0) return i + minSteps(n / i);
    }
    return n;
}
public int minSteps(int n) {
    int[] dp = new int[n + 1];
    int h = (int) Math.sqrt(n);
    for (int i = 2; i <= n; i++) {
        dp[i] = i;
        for (int j = 2; j <= h; j++) {
            if (i % j == 0) {
                dp[i] = dp[j] + dp[i / j];
                break;
            }
        }
    }
    return dp[n];
}
```

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/algorithm/alg-core-dynamic.html



## 算法思想 - 贪心算法

> 本文主要介绍算法中贪心算法的思想: 保证每次操作都是局部最优的，并且最后得到的结果是全局最优的。@pdai

[贪心思想相关题目]()[分配饼干]()[不重叠的区间个数]()[投飞镖刺破气球]()[根据身高和序号重组队列]()[分隔字符串使同种字符出现在一起]()[种植花朵]()[判断是否为子序列]()[修改一个数成为非递减数组]()[股票的最大收益]()

[#](#贪心思想相关题目) 贪心思想相关题目

### [#](#分配饼干) 分配饼干

[455. Assign Cookies (Easy)在新窗口打开](https://leetcode.com/problems/assign-cookies/description/)

```html
Input: [1,2], [1,2,3]
Output: 2

Explanation: You have 2 children and 3 cookies. The greed factors of 2 children are 1, 2.
You have 3 cookies and their sizes are big enough to gratify all of the children,
You need to output 2.
```

题目描述: 每个孩子都有一个满足度，每个饼干都有一个大小，只有饼干的大小大于等于一个孩子的满足度，该孩子才会获得满足。求解最多可以获得满足的孩子数量。

给一个孩子的饼干应当尽量小又能满足该孩子，这样大饼干就能拿来给满足度比较大的孩子。因为最小的孩子最容易得到满足，所以先满足最小的孩子。

证明: 假设在某次选择中，贪心策略选择给当前满足度最小的孩子分配第 m 个饼干，第 m 个饼干为可以满足该孩子的最小饼干。假设存在一种最优策略，给该孩子分配第 n 个饼干，并且 m < n。我们可以发现，经过这一轮分配，贪心策略分配后剩下的饼干一定有一个比最优策略来得大。因此在后续的分配中，贪心策略一定能满足更多的孩子。也就是说不存在比贪心策略更优的策略，即贪心策略就是最优策略。

```java
public int findContentChildren(int[] g, int[] s) {
    Arrays.sort(g);
    Arrays.sort(s);
    int gi = 0, si = 0;
    while (gi < g.length && si < s.length) {
        if (g[gi] <= s[si]) {
            gi++;
        }
        si++;
    }
    return gi;
}
```

### [#](#不重叠的区间个数) 不重叠的区间个数

[435. Non-overlapping Intervals (Medium)在新窗口打开](https://leetcode.com/problems/non-overlapping-intervals/description/)

```html
Input: [ [1,2], [1,2], [1,2] ]

Output: 2

Explanation: You need to remove two [1,2] to make the rest of intervals non-overlapping.
Input: [ [1,2], [2,3] ]

Output: 0

Explanation: You don't need to remove any of the intervals since they're already non-overlapping.
```

题目描述: 计算让一组区间不重叠所需要移除的区间个数。

计算最多能组成的不重叠区间个数，然后用区间总个数减去不重叠区间的个数。

在每次选择中，区间的结尾最为重要，选择的区间结尾越小，留给后面的区间的空间越大，那么后面能够选择的区间个数也就越大。

按区间的结尾进行排序，每次选择结尾最小，并且和前一个区间不重叠的区间。

```java
public int eraseOverlapIntervals(Interval[] intervals) {
    if (intervals.length == 0) {
        return 0;
    }
    Arrays.sort(intervals, Comparator.comparingInt(o -> o.end));
    int cnt = 1;
    int end = intervals[0].end;
    for (int i = 1; i < intervals.length; i++) {
        if (intervals[i].start < end) {
            continue;
        }
        end = intervals[i].end;
        cnt++;
    }
    return intervals.length - cnt;
}
```

使用 lambda 表示式创建 Comparator 会导致算法运行时间过长，如果注重运行时间，可以修改为普通创建 Comparator 语句:

```java
Arrays.sort(intervals, new Comparator<Interval>() {
    @Override
    public int compare(Interval o1, Interval o2) {
        return o1.end - o2.end;
    }
});
```

### [#](#投飞镖刺破气球) 投飞镖刺破气球

[452. Minimum Number of Arrows to Burst Balloons (Medium)在新窗口打开](https://leetcode.com/problems/minimum-number-of-arrows-to-burst-balloons/description/)

```text
Input:
[[10,16], [2,8], [1,6], [7,12]]

Output:
2
```

题目描述: 气球在一个水平数轴上摆放，可以重叠，飞镖垂直投向坐标轴，使得路径上的气球都会刺破。求解最小的投飞镖次数使所有气球都被刺破。

也是计算不重叠的区间个数，不过和 Non-overlapping Intervals 的区别在于，[1, 2] 和 [2, 3] 在本题中算是重叠区间。

```java
public int findMinArrowShots(int[][] points) {
    if (points.length == 0) {
        return 0;
    }
    Arrays.sort(points, Comparator.comparingInt(o -> o[1]));
    int cnt = 1, end = points[0][1];
    for (int i = 1; i < points.length; i++) {
        if (points[i][0] <= end) {
            continue;
        }
        cnt++;
        end = points[i][1];
    }
    return cnt;
}
```

### [#](#根据身高和序号重组队列) 根据身高和序号重组队列

[406. Queue Reconstruction by Height(Medium)在新窗口打开](https://leetcode.com/problems/queue-reconstruction-by-height/description/)

```html
Input:
[[7,0], [4,4], [7,1], [5,0], [6,1], [5,2]]

Output:
[[5,0], [7,0], [5,2], [6,1], [4,4], [7,1]]
```

题目描述: 一个学生用两个分量 (h, k) 描述，h 表示身高，k 表示排在前面的有 k 个学生的身高比他高或者和他一样高。

为了在每次插入操作时不影响后续的操作，身高较高的学生应该先做插入操作，否则身高较小的学生原先正确插入第 k 个位置可能会变成第 k+1 个位置。

身高降序、k 值升序，然后按排好序的顺序插入队列的第 k 个位置中。

```java
public int[][] reconstructQueue(int[][] people) {
    if (people == null || people.length == 0 || people[0].length == 0) {
        return new int[0][0];
    }
    Arrays.sort(people, (a, b) -> (a[0] == b[0] ? a[1] - b[1] : b[0] - a[0]));
    List<int[]> queue = new ArrayList<>();
    for (int[] p : people) {
        queue.add(p[1], p);
    }
    return queue.toArray(new int[queue.size()][]);
}
```

### [#](#分隔字符串使同种字符出现在一起) 分隔字符串使同种字符出现在一起

[763. Partition Labels (Medium)在新窗口打开](https://leetcode.com/problems/partition-labels/description/)

```html
Input: S = "ababcbacadefegdehijhklij"
Output: [9,7,8]
Explanation:
The partition is "ababcbaca", "defegde", "hijhklij".
This is a partition so that each letter appears in at most one part.
A partition like "ababcbacadefegde", "hijhklij" is incorrect, because it splits S into less parts.
public List<Integer> partitionLabels(String S) {
    int[] lastIndexsOfChar = new int[26];
    for (int i = 0; i < S.length(); i++) {
        lastIndexsOfChar[char2Index(S.charAt(i))] = i;
    }
    List<Integer> partitions = new ArrayList<>();
    int firstIndex = 0;
    while (firstIndex < S.length()) {
        int lastIndex = firstIndex;
        for (int i = firstIndex; i < S.length() && i <= lastIndex; i++) {
            int index = lastIndexsOfChar[char2Index(S.charAt(i))];
            if (index > lastIndex) {
                lastIndex = index;
            }
        }
        partitions.add(lastIndex - firstIndex + 1);
        firstIndex = lastIndex + 1;
    }
    return partitions;
}

private int char2Index(char c) {
    return c - 'a';
}
```

### [#](#种植花朵) 种植花朵

[605. Can Place Flowers (Easy)在新窗口打开](https://leetcode.com/problems/can-place-flowers/description/)

```html
Input: flowerbed = [1,0,0,0,1], n = 1
Output: True
```

题目描述: 花朵之间至少需要一个单位的间隔，求解是否能种下 n 朵花。

```java
public boolean canPlaceFlowers(int[] flowerbed, int n) {
    int len = flowerbed.length;
    int cnt = 0;
    for (int i = 0; i < len && cnt < n; i++) {
        if (flowerbed[i] == 1) {
            continue;
        }
        int pre = i == 0 ? 0 : flowerbed[i - 1];
        int next = i == len - 1 ? 0 : flowerbed[i + 1];
        if (pre == 0 && next == 0) {
            cnt++;
            flowerbed[i] = 1;
        }
    }
    return cnt >= n;
}
```

### [#](#判断是否为子序列) 判断是否为子序列

[392. Is Subsequence (Medium)在新窗口打开](https://leetcode.com/problems/is-subsequence/description/)

```html
s = "abc", t = "ahbgdc"
Return true.
public boolean isSubsequence(String s, String t) {
    int index = -1;
    for (char c : s.toCharArray()) {
        index = t.indexOf(c, index + 1);
        if (index == -1) {
            return false;
        }
    }
    return true;
}
```

### [#](#修改一个数成为非递减数组) 修改一个数成为非递减数组

[665. Non-decreasing Array (Easy)在新窗口打开](https://leetcode.com/problems/non-decreasing-array/description/)

```html
Input: [4,2,3]
Output: True
Explanation: You could modify the first 4 to 1 to get a non-decreasing array.
```

题目描述: 判断一个数组能不能只修改一个数就成为非递减数组。

在出现 nums[i] < nums[i - 1] 时，需要考虑的是应该修改数组的哪个数，使得本次修改能使 i 之前的数组成为非递减数组，并且 **不影响后续的操作** 。优先考虑令 nums[i - 1] = nums[i]，因为如果修改 nums[i] = nums[i - 1] 的话，那么 nums[i] 这个数会变大，就有可能比 nums[i + 1] 大，从而影响了后续操作。还有一个比较特别的情况就是 nums[i] < nums[i - 2]，只修改 nums[i - 1] = nums[i] 不能使数组成为非递减数组，只能修改 nums[i] = nums[i - 1]。

```java
public boolean checkPossibility(int[] nums) {
    int cnt = 0;
    for (int i = 1; i < nums.length && cnt < 2; i++) {
        if (nums[i] >= nums[i - 1]) {
            continue;
        }
        cnt++;
        if (i - 2 >= 0 && nums[i - 2] > nums[i]) {
            nums[i] = nums[i - 1];
        } else {
            nums[i - 1] = nums[i];
        }
    }
    return cnt <= 1;
}
```

### [#](#股票的最大收益) 股票的最大收益

[122. Best Time to Buy and Sell Stock II (Easy)在新窗口打开](https://leetcode.com/problems/best-time-to-buy-and-sell-stock-ii/description/)

题目描述: 一次股票交易包含买入和卖出，多个交易之间不能交叉进行。

对于 [a, b, c, d]，如果有 a <= b <= c <= d ，那么最大收益为 d - a。而 d - a = (d - c) + (c - b) + (b - a) ，因此当访问到一个 prices[i] 且 prices[i] - prices[i-1] > 0，那么就把 prices[i] - prices[i-1] 添加到收益中，从而在局部最优的情况下也保证全局最优。

```java
public int maxProfit(int[] prices) {
    int profit = 0;
    for (int i = 1; i < prices.length; i++) {
        if (prices[i] > prices[i - 1]) {
            profit += (prices[i] - prices[i - 1]);
        }
    }
    return profit;
}
```

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/algorithm/alg-core-greedy.html



## 算法思想 - 二分法

> 本文主要介绍算法思想中分治算法重要的二分法，比如二分查找；二分查找也称折半查找（Binary Search），它是一种效率较高的查找方法。但是，折半查找要求线性表必须采用顺序存储结构，而且表中元素按关键字有序排列。@pdai

[二分查找]()[正常实现]()[时间复杂度]()[二分查找变种]()

[#](#二分查找) 二分查找

### [#](#正常实现) 正常实现

```java
public int binarySearch(int[] nums, int key) {
    int l = 0, h = nums.length - 1;
    while (l <= h) {
        int m = l + (h - l) / 2;
        if (nums[m] == key) {
            return m;
        } else if (nums[m] > key) {
            h = m - 1;
        } else {
            l = m + 1;
        }
    }
    return -1;
}
```

### [#](#时间复杂度) 时间复杂度

二分查找也称为折半查找，每次都能将查找区间减半，这种折半特性的算法时间复杂度都为 O(logN)。

**m 计算**

有两种计算中值 m 的方式:

- m = (l + h) / 2
- m = l + (h - l) / 2

l + h 可能出现加法溢出，最好使用第二种方式。

**返回值**

循环退出时如果仍然没有查找到 key，那么表示查找失败。可以有两种返回值:

- -1: 以一个错误码表示没有查找到 key
- l: 将 key 插入到 nums 中的正确位置

### [#](#二分查找变种) 二分查找变种

二分查找可以有很多变种，变种实现要注意边界值的判断。例如在一个有重复元素的数组中查找 key 的最左位置的实现如下:

```java
public int binarySearch(int[] nums, int key) {
    int l = 0, h = nums.length - 1;
    while (l < h) {
        int m = l + (h - l) / 2;
        if (nums[m] >= key) {
            h = m;
        } else {
            l = m + 1;
        }
    }
    return l;
}
```

该实现和正常实现有以下不同:

- 循环条件为 l < h
- h 的赋值表达式为 h = m
- 最后返回 l 而不是 -1

在 nums[m] >= key 的情况下，可以推导出最左 key 位于 [l, m] 区间中，这是一个闭区间。h 的赋值表达式为 h = m，因为 m 位置也可能是解。

在 h 的赋值表达式为 h = mid 的情况下，如果循环条件为 l <= h，那么会出现循环无法退出的情况，因此循环条件只能是 l < h。以下演示了循环条件为 l <= h 时循环无法退出的情况:

```bash
nums = {0, 1, 2}, key = 1
l   m   h
0   1   2  nums[m] >= key
0   0   1  nums[m] < key
1   1   1  nums[m] >= key
1   1   1  nums[m] >= key
...
```

当循环体退出时，不表示没有查找到 key，因此最后返回的结果不应该为 -1。为了验证有没有查找到，需要在调用端判断一下返回位置上的值和 key 是否相等。

#### [#](#求开方) 求开方

[69. Sqrt(x) (Easy)在新窗口打开](https://leetcode.com/problems/sqrtx/description/)

```html
Input: 4
Output: 2

Input: 8
Output: 2
Explanation: The square root of 8 is 2.82842..., and since we want to return an integer, the decimal part will be truncated.
```

一个数 x 的开方 sqrt 一定在 0 ~ x 之间，并且满足 sqrt == x / sqrt。可以利用二分查找在 0 ~ x 之间查找 sqrt。

对于 x = 8，它的开方是 2.82842...，最后应该返回 2 而不是 3。在循环条件为 l <= h 并且循环退出时，h 总是比 l 小 1，也就是说 h = 2，l = 3，因此最后的返回值应该为 h 而不是 l。

```java
public int mySqrt(int x) {
    if (x <= 1) {
        return x;
    }
    int l = 1, h = x;
    while (l <= h) {
        int mid = l + (h - l) / 2;
        int sqrt = x / mid;
        if (sqrt == mid) {
            return mid;
        } else if (mid > sqrt) {
            h = mid - 1;
        } else {
            l = mid + 1;
        }
    }
    return h;
}
```

#### [#](#大于给定元素的最小元素) 大于给定元素的最小元素

[744. Find Smallest Letter Greater Than Target (Easy)在新窗口打开](https://leetcode.com/problems/find-smallest-letter-greater-than-target/description/)

```html
Input:
letters = ["c", "f", "j"]
target = "d"
Output: "f"

Input:
letters = ["c", "f", "j"]
target = "k"
Output: "c"
```

题目描述: 给定一个有序的字符数组 letters 和一个字符 target，要求找出 letters 中大于 target 的最小字符，如果找不到就返回第 1 个字符。

```java
public char nextGreatestLetter(char[] letters, char target) {
    int n = letters.length;
    int l = 0, h = n - 1;
    while (l <= h) {
        int m = l + (h - l) / 2;
        if (letters[m] <= target) {
            l = m + 1;
        } else {
            h = m - 1;
        }
    }
    return l < n ? letters[l] : letters[0];
}
```

#### [#](#有序数组的-single-element) 有序数组的 Single Element

[540. Single Element in a Sorted Array (Medium)在新窗口打开](https://leetcode.com/problems/single-element-in-a-sorted-array/description/)

```html
Input: [1,1,2,3,3,4,4,8,8]
Output: 2
```

题目描述: 一个有序数组只有一个数不出现两次，找出这个数。要求以 O(logN) 时间复杂度进行求解。

令 index 为 Single Element 在数组中的位置。如果 m 为偶数，并且 m + 1 < index，那么 nums[m] == nums[m + 1]；m + 1 >= index，那么 nums[m] != nums[m + 1]。

从上面的规律可以知道，如果 nums[m] == nums[m + 1]，那么 index 所在的数组位置为 [m + 2, h]，此时令 l = m + 2；如果 nums[m] != nums[m + 1]，那么 index 所在的数组位置为 [l, m]，此时令 h = m。

因为 h 的赋值表达式为 h = m，那么循环条件也就只能使用 l < h 这种形式。

```java
public int singleNonDuplicate(int[] nums) {
    int l = 0, h = nums.length - 1;
    while (l < h) {
        int m = l + (h - l) / 2;
        if (m % 2 == 1) {
            m--;   // 保证 l/h/m 都在偶数位，使得查找区间大小一直都是奇数
        }
        if (nums[m] == nums[m + 1]) {
            l = m + 2;
        } else {
            h = m;
        }
    }
    return nums[l];
}
```

#### [#](#第一个错误的版本) 第一个错误的版本

[278. First Bad Version (Easy)在新窗口打开](https://leetcode.com/problems/first-bad-version/description/)

题目描述: 给定一个元素 n 代表有 [1, 2, ..., n] 版本，可以调用 isBadVersion(int x) 知道某个版本是否错误，要求找到第一个错误的版本。

如果第 m 个版本出错，则表示第一个错误的版本在 [l, m] 之间，令 h = m；否则第一个错误的版本在 [m + 1, h] 之间，令 l = m + 1。

因为 h 的赋值表达式为 h = m，因此循环条件为 l < h。

```java
public int firstBadVersion(int n) {
    int l = 1, h = n;
    while (l < h) {
        int mid = l + (h - l) / 2;
        if (isBadVersion(mid)) {
            h = mid;
        } else {
            l = mid + 1;
        }
    }
    return l;
}
```

#### [#](#旋转数组的最小数字) 旋转数组的最小数字

[153. Find Minimum in Rotated Sorted Array (Medium)在新窗口打开](https://leetcode.com/problems/find-minimum-in-rotated-sorted-array/description/)

```html
Input: [3,4,5,1,2],
Output: 1
public int findMin(int[] nums) {
    int l = 0, h = nums.length - 1;
    while (l < h) {
        int m = l + (h - l) / 2;
        if (nums[m] <= nums[h]) {
            h = m;
        } else {
            l = m + 1;
        }
    }
    return nums[l];
}
```

#### [#](#查找区间) 查找区间

[34. Search for a Range (Medium)在新窗口打开](https://leetcode.com/problems/search-for-a-range/description/)

```html
Input: nums = [5,7,7,8,8,10], target = 8
Output: [3,4]

Input: nums = [5,7,7,8,8,10], target = 6
Output: [-1,-1]
public int[] searchRange(int[] nums, int target) {
    int first = binarySearch(nums, target);
    int last = binarySearch(nums, target + 1) - 1;
    if (first == nums.length || nums[first] != target) {
        return new int[]{-1, -1};
    } else {
        return new int[]{first, Math.max(first, last)};
    }
}

private int binarySearch(int[] nums, int target) {
    int l = 0, h = nums.length; // 注意 h 的初始值
    while (l < h) {
        int m = l + (h - l) / 2;
        if (nums[m] >= target) {
            h = m;
        } else {
            l = m + 1;
        }
    }
    return l;
}
```

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/algorithm/alg-core-devide-two.html



# Algo  Sort

## Quicksort

> 快速排序

1. 定基准             ![](https://img-blog.csdnimg.cn/20201121183621106.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpb25nQmVuMTk5Mw==,size_16,color_FFFFFF,t_70#pic_center)

2. 分治            ![](https://img-blog.csdnimg.cn/20201121192051815.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3hpb25nQmVuMTk5Mw==,size_16,color_FFFFFF,t_70#pic_center)



### 快速排序时间复杂度和稳定性

[#](#快速排序稳定性) 快速排序稳定性

快速排序是不稳定的算法，它不满足稳定算法的定义。

`算法稳定性` -- 假设在数列中存在a[i]=a[j]，若在排序之前，a[i]在a[j]前面；并且排序之后，a[i]仍然在a[j]前面。则这个排序算法是稳定的！

#### [#](#快速排序时间复杂度) 快速排序时间复杂度

> 快速排序的时间复杂度在最坏情况下是O(N2)，平均的时间复杂度是O(N*lgN)。

这句话很好理解: 假设被排序的数列中有N个数。遍历一次的时间复杂度是O(N)，需要遍历多少次呢? 至少lg(N+1)次，最多N次。

- 为什么最少是lg(N+1)次? 快速排序是采用的分治法进行遍历的，我们将它看作一棵二叉树，它需要遍历的次数就是二叉树的深度，而根据完全二叉树的定义，它的深度至少是lg(N+1)。因此，快速排序的遍历次数最少是lg(N+1)次。
- 为什么最多是N次? 这个应该非常简单，还是将快速排序看作一棵二叉树，它的深度最大是N。因此，快读排序的遍历次数最多是N次。

## [#](#代码实现) 代码实现

------

著作权归@pdai所有 原文链接：https://pdai.tech/md/algorithm/alg-sort-x-fast.html

## Bubble Sort



### 复杂度和稳定性

#### [#](#冒泡排序时间复杂度) 冒泡排序时间复杂度

冒泡排序的时间复杂度是O(N2)。 假设被排序的数列中有N个数。遍历一趟的时间复杂度是O(N)，需要遍历多少次呢? N-1次！因此，冒泡排序的时间复杂度是O(N2)。

#### [#](#冒泡排序稳定性) 冒泡排序稳定性

冒泡排序是稳定的算法，它满足稳定算法的定义。 算法稳定性 -- 假设在数列中存在a[i]=a[j]，若在排序之前，a[i]在a[j]前面；并且排序之后，a[i]仍然在a[j]前面。则这个排序算法是稳定的！

### 代码实现

```Java
/**
 * 冒泡排序: Java
 *
 * @author skywang
 * @date 2014/03/11
 */

public class BubbleSort {

    /*
     * 冒泡排序
     *
     * 参数说明: 
     *     a -- 待排序的数组
     *     n -- 数组的长度
     */
    public static void bubbleSort1(int[] a, int n) {
        int i,j;

        for (i=n-1; i>0; i--) {
            // 将a[0...i]中最大的数据放在末尾
            for (j=0; j<i; j++) {

                if (a[j] > a[j+1]) {
                    // 交换a[j]和a[j+1]
                    int tmp = a[j];
                    a[j] = a[j+1];
                    a[j+1] = tmp;
                }
            }
        }
    }

    /*
     * 冒泡排序(改进版)
     *
     * 参数说明: 
     *     a -- 待排序的数组
     *     n -- 数组的长度
     */
    public static void bubbleSort2(int[] a, int n) {
        int i,j;
        int flag;                 // 标记

        for (i=n-1; i>0; i--) {

            flag = 0;            // 初始化标记为0
            // 将a[0...i]中最大的数据放在末尾
            for (j=0; j<i; j++) {
                if (a[j] > a[j+1]) {
                    // 交换a[j]和a[j+1]
                    int tmp = a[j];
                    a[j] = a[j+1];
                    a[j+1] = tmp;

                    flag = 1;    // 若发生交换，则设标记为1
                }
            }

            if (flag==0)
                break;            // 若没发生交换，则说明数列已有序。
        }
    }

    public static void main(String[] args) {
        int i;
        int[] a = {20,40,30,10,60,50};

        System.out.printf("before sort:");
        for (i=0; i<a.length; i++)
            System.out.printf("%d ", a[i]);
        System.out.printf("\n");

        bubbleSort1(a, a.length);
        //bubbleSort2(a, a.length);

        System.out.printf("after  sort:");
        for (i=0; i<a.length; i++)
            System.out.printf("%d ", a[i]);
        System.out.printf("\n");
    }
}

```



# Algo     Array

## Array

### 轮转数组

**描述**：给定一个数组 `nums`，再给定一个数字 `k`。

**要求**：将数组中的元素向右移动 `k` 个位置。




### 加1

**描述**：给定一个非负整数数组，数组每一位对应整数的一位数字。

**要求**：计算整数加 `1` 后的结果。

```
def plusOne(self, digits: List[int]) -> List[int]:
    digits = [0] + digits
    digits[len(digits) - 1] += 1
    for i in range(len(digits)-1, 0, -1):
        if digits[i] != 10:
            break
        else:
            digits[i] = 0
            digits[i - 1] += 1
        
    if digits[0] == 0:
        return digits[1:] 
    else:
        return digits
```





### 中心下标

**描述**：给定一个数组 `nums`。

**要求**：找到「左侧元素和」与「右侧元素和相等」的位置，若找不到，则返回 `-1`。

> 两次遍历，第一次遍历先求出数组全部元素和。第二次遍历找到左侧元素和恰好为全部元素和一半的位置

```python
class Solution:
    def pivotIndex(self, nums: List[int]) -> int:
        sum = 0
        for i in range(len(nums)):
            sum += nums[i]
        curr_sum = 0
        for i in range(len(nums)):
            if curr_sum * 2 + nums[i] == sum:
                return i
            curr_sum += nums[i]
        return -1
```

![image-20230421182132482](https://article.biliimg.com/bfs/article/07628ae51900e72595ef2787fcd78bf5e0cad6a6.png)

### 最大连续 1 的个数

给定一个二进制数组，数组中只包含 0 和 1，计算其中最大连续 1 的个数。****

> 使用两个变量 sum 和 ans。sum 用于存储当前连续 1 的个数，ans 用于存储最大连续 1 的个数。然后进行一次遍历，统计当前连续 1 的个数，并更新最大的连续 1 个数。

```python
class Solution:
    def findMaxConsecutiveOnes(self, nums: List[int]) -> int:
        ans = 0
        sum = 0
        for num in nums:
            if num == 1:
                sum += 1
                ans = max(ans, sum)
            else:
                sum = 0
        return ans
```



### nums[i] 之外的其他所有元素乘积。

要求不能使用除法，且在 O(n) 时间复杂度、常数空间复杂度内解决问题

> 构造一个答案数组 res，长度和数组 nums 长度一致。先从左到右遍历一遍 nums 数组，将 nums[i] 左侧的元素乘积累积起来，存储到 res 数组中。再从右到左遍历一遍，将 nums[i] 右侧的元素乘积累积起来，再乘以原本 res[i] 的值，即为 nums 中除了 nums[i] 之外的其他所有元素乘积。

```

class Solution:
    def productExceptSelf(self, nums: List[int]) -> List[int]:
        size = len(nums)
        res = [1 for _ in range(size)]

        left = 1
        for i in range(size):
            res[i] *= left
            left *= nums[i]

        right = 1
        for i in range(size-1, -1, -1):
            res[i] *= right
            right *= nums[i]
        return res
```

### 对角线遍历 (二↓ )

![img](https://assets.leetcode.com/uploads/2021/04/10/diag1-grid.jpg)

找规律：

1. 当「行号 + 列号」为偶数时，遍历方向为从左下到右上。可以记为右上方向 `(-1, +1)`，即行号减 `1`，列号加 `1`。
2. 当「行号 + 列号」为奇数时，遍历方向为从右上到左下。可以记为左下方向 `(+1, -1)`，即行号加 `1`，列号减 `1`。

边界情况：

1. 向右上方向移动时：
   1. 如果在最后一列，则向下方移动，即 `x += 1`。
   2. 如果在第一行，则向右方移动，即 `y += 1`。
   3. 其余情况想右上方向移动，即 `x -= 1`、`y += 1`。
2. 向左下方向移动时：
   1. 如果在最后一行，则向右方移动，即 `y += 1`。
   2. 如果在第一列，则向下方移动，即 `x += 1`。
   3. 其余情况向左下方向移动，即 `x += 1`、`y -= 1`。

```python
class Solution:
    def findDiagonalOrder(self, mat: List[List[int]]) -> List[int]:
        rows = len(mat)
        cols = len(mat[0])
        count = rows * cols
        x, y = 0, 0
        ans = []

        for i in range(count):
            ans.append(mat[x][y])

            if (x + y) % 2 == 0:
                # 最后一列
                if y == cols - 1:
                    x += 1
                # 第一行
                elif x == 0:
                    y += 1
                # 右上方向
                else:
                    x -= 1
                    y += 1
            else:
                # 最后一行
                if x == rows - 1:
                    y += 1
                # 第一列
                elif y == 0:
                    x += 1
                # 左下方向
                else:
                    x += 1
                    y -= 1
        return ans
```



### 旋转 90°

不能使用额外的数组空间

![img](https://article.biliimg.com/bfs/article/909611016f48da823bed5ebfffb3a1862acf7093.jpg)

> 「水平翻转」+「主对角线翻转」
>
> ![image-20230422124832629](https://article.biliimg.com/bfs/article/9674be8b748d1c62978eef89567ecc655368b4a4.png)

```python
def rotate(self, matrix: List[List[int]]) -> None:
    n = len(matrix)
    
    for i in range(n // 2): //i=0
        for j in range(n): //n=3
            matrix[i][j], matrix[n - i - 1][j] = matrix[n - i - 1][j], matrix[i][j]
    
    for i in range(n): //n=3
        for j in range(i):
            matrix[i][j], matrix[j][i] = matrix[j][i], matrix[i][j]
```



> 对于矩阵中第 `i` 行的第 `j` 个元素，在旋转后，它出现在倒数第 `i` 列的第 `j` 个位置。即 `matrixnew[j][n − i − 1] = matrix[i][j]`。
>
> 而 `matrixnew[j][n - i - 1]` 的点经过旋转移动到了 `matrix[n − i − 1][n − j − 1]` 的位置。
>
> ![img](https://article.biliimg.com/bfs/article/b45be892e52b9adf082e3c31903b9fca06a38e8b.jpg)

```
class Solution:
    def rotate(self, matrix: List[List[int]]) -> None:
        n = len(matrix)

        for i in range(n // 2):
            for j in range((n + 1) // 2):
                matrix[i][j], 
                matrix[n - j - 1][i], 
                matrix[n - i - 1][n - j - 1],
                matrix[j][n - i - 1] = 
                
                matrix[n - j - 1][i], matrix[n - i - 1][n - j - 1], matrix[j][n - i - 1], matrix[i][j]
```



### 矩阵置0

给定一个 `m * n` 大小的矩阵。

要求：如果一个元素为 `0`，则将其所在行和列所有元素都置为 `0`。要求使用原地算法，并使用常量空间解决。

> 使用标记数组

```
class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        m, n = len(matrix), len(matrix[0])
        row, col = [False] * m, [False] * n

        for i in range(m):
            for j in range(n):
                if matrix[i][j] == 0:
                    row[i] = col[j] = True
        
        for i in range(m):
            for j in range(n):
                if row[i] or col[j]:
                    matrix[i][j] = 0

```



> 我们可以用矩阵的第一行和第一列代替方法一中的两个标记数组，以达到 O(1) 的额外空间。但这样会导致原数组的第一行和第一列被修改，无法记录它们是否原本包含 
> 0。因此我们需要额外使用两个标记变量分别记录第一行和第一列是否原本包含 0

```
class Solution:
    def setZeroes(self, matrix: List[List[int]]) -> None:
        m, n = len(matrix), len(matrix[0])
        flag_col0 = any(matrix[i][0] == 0 for i in range(m))
        flag_row0 = any(matrix[0][j] == 0 for j in range(n))
        
        for i in range(1, m):
            for j in range(1, n):
                if matrix[i][j] == 0:
                    matrix[i][0] = matrix[0][j] = 0
        
        for i in range(1, m):
            for j in range(1, n):
                if matrix[i][0] == 0 or matrix[0][j] == 0:
                    matrix[i][j] = 0
        
        if flag_col0:
            for i in range(m):
                matrix[i][0] = 0
        
        if flag_row0:
            for j in range(n):
                matrix[0][j] = 0

```

### 螺旋矩阵

**描述**：给定一个 `m * n` 大小的二维矩阵 `matrix`。

**要求**：按照顺时针旋转的顺序，返回矩阵中的所有元素。

> 

```python
class Solution:
    def spiralOrder(self, matrix: List[List[int]]) -> List[int]:
        up, down, left, right = 0, len(matrix)-1, 0, len(matrix[0])-1
        ans = []
        while True:
            for i in range(left, right + 1):
                ans.append(matrix[up][i])
            up += 1
            if up > down:
                break
            for i in range(up, down + 1):
                ans.append(matrix[i][right])
            right -= 1
            if right < left:
                break
            for i in range(right, left - 1, -1):
                ans.append(matrix[down][i])
            down -= 1
            if down < up:
                break
            for i in range(down, up - 1, -1):
                ans.append(matrix[i][left])
            left += 1
            if left > right:
                break
        return ans
```



## Array___排序

### 冒泡排序

![img](https://article.biliimg.com/bfs/article/1fda9f88f76bfc4b14385fd4461202a5ad27bb91.gif)

```python
class Solution:
    def bubbleSort(self, arr):
        # 第 i 趟排序
        for i in range(len(arr)):
            # 从序列中前 n - i + 1 个元素的第 1 个元素开始，相邻两个元素进行比较
            for j in range(len(arr) - i - 1):
                # 相邻两个元素进行比较，如果前者大于后者，则交换位置
                if arr[j] > arr[j + 1]:
                    arr[j], arr[j + 1] = arr[j + 1], arr[j]

        return arr

    def sortArray(self, nums: List[int]) -> List[int]:
        return self.bubbleSort(nums)
```

- **最佳时间复杂度**：�(�)。最好的情况下（初始时序列已经是升序排列），则只需经过 `1` 趟排序，总共经过 `n - 1` 次元素之间的比较，并且不移动元素，算法就可结束排序。因此，冒泡排序算法的最佳时间复杂度为 �(�)。
- **最坏时间复杂度**：�(�2)。最差的情况下（初始时序列已经是降序排列，或者最小值元素处在序列的最后），则需要进行 `n - 1` 趟排序，总共进行 ∑�=2�(�−1)=�(�−1)2 次元素之间的比较，因此，冒泡排序算法的最坏时间复杂度为 �(�2)。
- **冒泡排序适用情况**：冒泡排序方法在排序过程中需要移动较多次数的元素，并且排序时间效率比较低。因此，冒泡排序方法比较适合于参加排序序列的数据量较小的情况，尤其是当序列的初始状态为基本有序的情况。
- **排序稳定性**：由于元素交换是在相邻元素之间进行的，不会改变值相同元素的相对位置，因此，冒泡排序法是一种 **稳定排序算法**。

### 选择排序（Min 不稳定）

> 简单来说，「选择排序算法」是在每一趟排序中，从未排序部分中选出一个值最小的元素，与未排序部分第 `1` 个元素交换位置，从而将该元素划分到已排序部分。

![img](https://article.biliimg.com/bfs/article/a88892fe30c826f7ac58c2f9854652ea99358c76.gif)

- **时间复杂度**：�(�2)。排序法所进行的元素之间的比较次数与序列的原始状态无关，时间复杂度总是 �(�2)。
  - 这是因为无论序列中元素的初始排列状态如何，第 `i` 趟排序要找出值最小元素都需要进行 `n − i` 次元素之间的比较。因此，整个排序过程需要进行的元素之间的比较次数都相同，为 ∑�=2�(�−1)=�(�−1)2 次。
- **选择排序适用情况**：选择排序方法在排序过程中需要移动较多次数的元素，并且排序时间效率比较低。因此，选择排序方法比较适合于参加排序序列的数据量较小的情况。选择排序的主要优点是仅需要原地操作无需占用其他空间就可以完成排序，因此在空间复杂度要求较高时，可以考虑选择排序。
- **排序稳定性**：由于值最小元素与未排序部分第 `1` 个元素的交换动作是在不相邻的元素之间进行的，因此很有可能会改变值相同元素的前后位置，因此，选择排序法是一种 **不稳定排序算法**。

```
class Solution:
    def selectionSort(self, arr):
        for i in range(len(arr) - 1):
            # 记录未排序部分中最小值的位置
            min_i = i
            for j in range(i + 1, len(arr)):
                if arr[j] < arr[min_i]:
                    min_i = j
            # 如果找到最小值的位置，将 i 位置上元素与最小值位置上的元素进行交换
            if i != min_i:
                arr[i], arr[min_i] = arr[min_i], arr[i]
        return arr

    def sortArray(self, nums: List[int]) -> List[int]:
        return self.selectionSort(nums)
```



### 插入排序（逐渐有序 1 ...）

> 将整个序列分为两部分：前面 `i` 个元素为有序序列，后面 `n - i` 个元素为无序序列。每一次排序，将无序序列的第 `1` 个元素，在有序序列中找到相应的位置并插入

![img](https://article.biliimg.com/bfs/article/83268ada64508699ceb8582a05d15694eb23804d.gif)

```
class Solution:
    def insertionSort(self, arr):
        # 遍历无序序列
        for i in range(1, len(arr)):
            temp = arr[i]
            j = i
            # 从右至左遍历有序序列
            while j > 0 and arr[j - 1] > temp:
                # 将有序序列中插入位置右侧的元素依次右移一位
                arr[j] = arr[j - 1]
                j -= 1
            # 将该元素插入到适当位置
            arr[j] = temp

        return arr

    def sortArray(self, nums: List[int]) -> List[int]:
        return self.insertionSort(nums)
```

- **最佳时间复杂度**：�(�)。最好的情况下（初始时序列已经是升序排列），对应的每个 `i` 值只进行一次元素之间的比较，因而总的比较次数最少，为 ∑�=2�1=�−1，并不需要移动元素（记录），这是最好的情况。
- **最差时间复杂度**：�(�2)。最差的情况下（初始时序列已经是降序排列），对应的每个 `i` 值都要进行 `i - 1` 次元素之间的比较，总的元素之间的比较次数达到最大值，为 ∑�=2�(�−1)=�(�−1)2。
- **平均时间复杂度**：�(�2)。如果序列的初始情况是随机的，即参加排序的序列中元素可能出现的各种排列的概率相同，则可取上述最小值和最大值的平均值作为插入排序时所进行的元素之间的比较次数，约为 �24。由此得知，插入排序算法的时间复杂度 �(�2)。
- **排序稳定性**：插入排序方法是一种 **稳定排序算法**。

### 希尔排序(划分)【插入排序】   NO

> 将整个序列切按照一定的间隔取值划分为若干个子序列，每个子序列分别进行插入排序。然后逐渐缩小间隔进行下一轮划分子序列和对子序列进行插入排序。直至最后一轮排序间隔为 `1`，对整个序列进行插入排序。

![img](https://article.biliimg.com/bfs/article/50867026817723f40add170a33c11323b39747b9.png)

- **时间复杂度**：介于 �(�×log2⁡�) 与 �(�2) 之间。
  - 希尔排序方法的速度是一系列间隔数 ���� 的函数，而比较次数与 ���� 之间的依赖关系比较复杂，不太容易给出完整的数学分析。
  - 由于采用 ����=⌊����−1/2⌋ 的方法缩小间隔数，对于具有 � 个元素的序列，若 ���1=⌊�/2⌋，则经过 �=⌊log2⁡�⌋ 趟排序后就有 ����=1，因此，希尔排序方法的排序总躺数为 ⌊log2⁡�⌋。
  - 从算法中也可以看到，最外层的 `while` 循环为 log2⁡� 数量级，中间层 `do-while` 循环为 `n` 数量级。当子序列分得越多时，子序列内的元素就越少，最内层的 `for` 循环的次数也就越少；反之，当所分的子序列个数减少时，子序列内的元素也随之增多，但整个序列也逐步接近有序，而循环次数却不会随之增加。因此，希尔排序算法的时间复杂度在 �(�×log2⁡�) 与 �(�2) 之间。
- **排序稳定性**：希尔排序方法是一种 **不稳定排序算法**。





### 归并排序（分治 递归）

> 采用经典的分治策略，先递归地将当前序列平均分成两半。然后将有序序列两两合并，最终合并成一个有序序列。

![img](https://article.biliimg.com/bfs/article/2a8172e000c55455e11555dbdbcdb0404e4b1921.gif)

- **时间复杂度**：�(�×log2⁡�)。归并排序算法的时间复杂度等于归并趟数与每一趟归并的时间复杂度乘积。子算法 `merge(left_arr, right_arr):` 的时间复杂度是 �(�)，因此，归并排序算法总的时间复杂度为 �(�×log2⁡�)。

- **空间复杂度**：�(�)。归并排序方法需要用到与参加排序的序列同样大小的辅助空间。因此算法的空间复杂度为 �(�)。

- 排序稳定性

  ：归并排序算法是一种

   

  稳定排序算法

  。

  - 因为在两个有序子序列的归并过程中，如果两个有序序列中出现相同元素，`merge(left_arr, right_arr):` 算法能够使前一个序列中那个相同元素先被复制，从而确保这两个元素的相对次序不发生改变。

```python
class Solution:
    def merge(self, left_arr, right_arr):           # 归并过程
        arr = []
        left_i, right_i = 0, 0
        while left_i < len(left_arr) and right_i < len(right_arr):
            # 将两个有序子序列中较小元素依次插入到结果数组中
            if left_arr[left_i] < right_arr[right_i]:
                arr.append(left_arr[left_i])
                left_i += 1
            else:
                arr.append(right_arr[right_i])
                right_i += 1
        
        while left_i < len(left_arr):
            # 如果左子序列有剩余元素，则将其插入到结果数组中
            arr.append(left_arr[left_i])
            left_i += 1
            
        while right_i < len(right_arr):
            # 如果右子序列有剩余元素，则将其插入到结果数组中
            arr.append(right_arr[right_i])
            right_i += 1
        
        return arr                                  # 返回排好序的结果数组

    def mergeSort(self, arr):                       # 分割过程
        if len(arr) <= 1:                           # 数组元素个数小于等于 1 时，直接返回原数组
            return arr
        
        mid = len(arr) // 2                         # 将数组从中间位置分为左右两个数组。
        left_arr = self.mergeSort(arr[0: mid])      # 递归将左子序列进行分割和排序
        right_arr =  self.mergeSort(arr[mid:])      # 递归将右子序列进行分割和排序
        return self.merge(left_arr, right_arr)      # 把当前序列组中有序子序列逐层向上，进行两两合并。

    def sortArray(self, nums: List[int]) -> List[int]:
        return self.mergeSort(nums)
```

### 快速排序（ | 递归）

> 通过一趟排序将无序序列分为独立的两个序列，第一个序列的值均比第二个序列的值小。然后*<u>**递归**</u>*地排列两个子序列，以达到整个序列有序。

![img](https://article.biliimg.com/bfs/article/cdc60009a22797909baf9ad55369dd8176038066.gif)

```
import random

class Solution:
    # 从 arr[low: high + 1] 中随机挑选一个基准数，并进行移动排序
    def randomPartition(self, arr: [int], low: int, high: int):
        # 随机挑选一个基准数
        i = random.randint(low, high)
        # 将基准数与最低位互换
        arr[i], arr[low] = arr[low], arr[i]
        # 以最低位为基准数，然后将序列中比基准数大的元素移动到基准数右侧，比他小的元素移动到基准数左侧。最后将基准数放到正确位置上
        return self.partition(arr, low, high)
    
    # 以最低位为基准数，然后将序列中比基准数大的元素移动到基准数右侧，比他小的元素移动到基准数左侧。最后将基准数放到正确位置上
    def partition(self, arr: [int], low: int, high: int):
        pivot = arr[low]            # 以第 1 为为基准数
        i = low + 1                 # 从基准数后 1 位开始遍历，保证位置 i 之前的元素都小于基准数
        
        for j in range(i, high + 1):
            # 发现一个小于基准数的元素
            if arr[j] < pivot:
                # 将小于基准数的元素 arr[j] 与当前 arr[i] 进行换位，保证位置 i 之前的元素都小于基准数
                arr[i], arr[j] = arr[j], arr[i]
                # i 之前的元素都小于基准数，所以 i 向右移动一位
                i += 1
        # 将基准节点放到正确位置上
        arr[i - 1], arr[low] = arr[low], arr[i - 1]
        # 返回基准数位置
        return i - 1

    def quickSort(self, arr, low, high):
        if low < high:
            # 按照基准数的位置，将序列划分为左右两个子序列
            pi = self.randomPartition(arr, low, high)
            # 对左右两个子序列分别进行递归快速排序
            self.quickSort(arr, low, pi - 1)
            self.quickSort(arr, pi + 1, high)

        return arr

    def sortArray(self, nums: List[int]) -> List[int]:
        return self.quickSort(nums, 0, len(nums) - 1)
```



### 堆排序

> 借用「堆结构」所设计的排序算法。将数组转化为大顶堆，重复从大顶堆中取出数值最大的节点，并让剩余的堆结构继续维持大顶堆性质。

- 时间复杂度

  ：

  �(�×log2⁡�)

  。

  - 堆积排序的时间主要花费在两个方面：「建立初始堆」和「调整堆」。

  - 设原始序列所对应的完全二叉树深度为

     

    �

    ，算法由两个独立的循环组成：

    1. 在第 1 个循环构造初始堆积时，从 �=�−1 层开始，到 �=1 层为止，对每个分支节点都要调用一次调整堆算法，而一次调整堆算法，对于第 � 层一个节点到第 � 层上建立的子堆积，所有节点可能移动的最大距离为该子堆积根节点移动到最后一层（第 � 层） 的距离，即 �−�。而第 � 层上节点最多有 2�−1 个，所以每一次调用调整堆算法的最大移动距离为 2�−1∗(�−�)。因此，堆积排序算法的第 1 个循环所需时间应该是各层上的节点数与该层上节点可移动的最大距离之积的总和，即：∑�=�−112�−1(�−�)=∑�=1�−12�−�−1×�=∑�=1�−12�−1×�2�≤�∑�=1�−1�2�<2�。这一部分的时间花费为 �(�)。
    2. 在第 2 个循环中，每次调用调整堆算法一次，节点移动的最大距离为这棵完全二叉树的深度 �=⌊log2⁡(�)⌋+1，一共调用了 �−1 次调整堆算法，所以，第 2 个循环的时间花费为 (�−1)(⌊log2⁡(�)⌋+1)=�(�×log2⁡�)。

  - 因此，堆积排序的时间复杂度为 �(�×log2⁡�)。

- **空间复杂度**：�(1)。由于在堆积排序中只需要一个记录大小的辅助空间，因此，堆积排序的空间复杂度为：�(1)。

- **排序稳定性**：堆排序是一种 **不稳定排序算法**。

```python
class Solution:
    # 调整为大顶堆
    def heapify(self, arr: [int], index: int, end: int):
        # 根节点为 index，左节点为 2 * index + 1， 右节点为 2 * index + 2
        left = index * 2 + 1
        right = left + 1
        while left <= end:
            # 当前节点为非叶子结点
            max_index = index
            if arr[left] > arr[max_index]:
                max_index = left
            if right <= end and arr[right] > arr[max_index]:
                max_index = right
            if index == max_index:
                # 如果不用交换，则说明已经交换结束
                break
            arr[index], arr[max_index] = arr[max_index], arr[index]
            # 继续调整子树
            index = max_index
            left = index * 2 + 1
            right = left + 1

    # 初始化大顶堆
    def buildMaxHeap(self, arr: [int]):
        size = len(arr)
        # (size - 2) // 2 是最后一个非叶节点，叶节点不用调整
        for i in range((size - 2) // 2, -1, -1):
            self.heapify(arr, i, size - 1)
        return arr

    # 升序堆排序，思路如下：
    # 1. 先建立大顶堆
    # 2. 让堆顶最大元素与最后一个交换，然后调整第一个元素到倒数第二个元素，这一步获取最大值
    # 3. 再交换堆顶元素与倒数第二个元素，然后调整第一个元素到倒数第三个元素，这一步获取第二大值
    # 4. 以此类推，直到最后一个元素交换之后完毕。
    def maxHeapSort(self, arr: [int]):
        self.buildMaxHeap(arr)
        size = len(arr)
        for i in range(size):
            arr[0], arr[size - i - 1] = arr[size - i - 1], arr[0]
            self.heapify(arr, 0, size - i - 2)
        return arr

    def sortArray(self, nums: List[int]) -> List[int]:
        return self.maxHeapSort(nums)
```

# algoLeetcode

## 贪心  局部最优

### 45  跳跃游戏2

> 题解分析

本题与[55. 跳跃游戏]这题不同，不能使用相同的贪心思路来解决。这里需要求解的问题是最小的跳跃次数，而要求解最小的跳跃次数需要每次在跳跃时选择尽可能跨度大的格子。

**换句话说，只需要判断哪一个选择最具有「潜力」即可**：

[![image](https://img2023.cnblogs.com/blog/1758873/202302/1758873-20230219224124199-1000875107.png)](https://img2023.cnblogs.com/blog/1758873/202302/1758873-20230219224124199-1000875107.png)

```java
class Solution {
    public int jump(int[] nums) {
        int right = 0, maxStep = 0, len = nums.length;
        int steps = 0;
        for (int i=0; i<len-1; i++) {
            maxStep = Math.max(maxStep, nums[i] + i);
            if (i == right) {
                right = maxStep;
                steps++;
            }
        }
        return steps;
    }
}
```





## 分治  转换小问题 递归

### 定义表达式

>   第一条特征是绝大多数问题都可以满足的，因为问题的计算复杂性一般随着问题规模的增加而增加。
>
>    第二条特征是应用分治法的前提，此特征反映了递归思想的应用。
>
>    第三条是关键，能否利用分治法完全取决于问题是否具有第三条特症，如果具备了第一条和第二条特征，而不具备第三条特征，可以考虑使用贪心法或者动态规划。
>
>    第四条特征涉及到分支法的效率。

```java
public List<Integer> diffWaysToCompute(String input) {
    List<Integer> ways = new ArrayList<>();
    for (int i = 0; i < input.length(); i++) {
        char c = input.charAt(i);
        if (c == '+' || c == '-' || c == '*') {
            List<Integer> left = diffWaysToCompute(input.substring(0, i));
            List<Integer> right = diffWaysToCompute(input.substring(i + 1));
            for (int l : left) {
                for (int r : right) {
                    switch (c) {
                        case '+':
                            ways.add(l + r);
                            break;
                        case '-':
                            ways.add(l - r);
                            break;
                        case '*':
                            ways.add(l * r);
                            break;
                    }
                }
            }
        }
    }
    if (ways.size() == 0) {
        ways.add(Integer.valueOf(input));
    }
    return ways;
}
------
著作权归@pdai所有
原文链接：https://pdai.tech/md/algorithm/alg-core-divide-and-conquer.html
```



## 动态规划  转换小问题  保存

### 概念

动态规划(dynamic programming)是运筹学的一个分支，是求解决策过程(decision process)最优化的数学方法。
动态规划算法通常基于一个**递推公式**及一个或多个**初始状态**。 当前**子问题的解将由上一次子问题的解推出**。

### 基本思想

要解决一个给定的问题，我们需要解决其不同部分（即**解决子问题**），再合并子问题的解以得出原问题的解。
通常许多子问题非常相似，为此动态规划法试图**只解决每个子问题一次**，从而减少计算量。
一旦某个给定子问题的解已经算出，则将其**记忆化存储**，以便下次需要同一个子问题解之时直接查表。
这种做法在重复子问题的数目关于输入的规模呈指数增长时特别有用。
动态规划有三个核心元素：
1.最优子结构
2.边界
3.状态转移方程

我们来看一到题目

### 爬楼梯

> 有一座高度是10级台阶的楼梯，从下往上走，每跨一步只能向上1级或者2级台阶。求出一共有多少种走法。

比如，每次走1级台阶，一共走10步，这是其中一种走法。
再比如，每次走2级台阶，一共走5步，这是另一种走法。

但是这样一个个算太麻烦了，我们可以只去思考最后一步怎么走，如下图
![图片描述](https://segmentfault.com/img/bVbe2Vg?w=1216&h=968)

这样走到第十个楼梯的走法 = 走到第八个楼梯 + 走到第九个楼梯
我们用f(n)来表示 走到第n个楼梯的走法，所以就有了f(10) = f(9) + f(8)
然后f(9) = f(8) + f(7), f(8) = f(7) + f(6)......

这样我们就得出来一个**递归式**：
**f(n) = f(n-1) + f(n-2);**
还有两个**初始状态**：
**f(1) = 1;**
**f(2) = 2;**

这样就得出了第一种解法

#### 方法一：递归求解

> ```scss
> function getWays(n) {
> 
>     if (n < 1) return 0;
>     if (n == 1) return 1;
>     if (n == 2) return 2;
> 
>     return getWays(n-1) + getWays(n-2); 
> }
> ```

这种方法的时间复杂度为**O(2^n)**
![图片描述](https://segmentfault.com/img/bVbe2Yo?w=1270&h=692)

可以看到这是一颗二叉树，数的节点个数就是我们递归方程需要计算的次数，
数的高度为N，节点个数近似于2^n
所以时间复杂度近似于O(2^n)

但是这种方法能不能优化呢？
我们会发现有些值被重复计算，如下图
![图片描述](https://segmentfault.com/img/bVbe2ZF?w=1272&h=682)
相同颜色代表着重复的部分，那么我们可不可以把这些重复计算的值**记录**下来呢？
这样的优化就有了第二种方法

#### 方法二：备忘录算法

> ```kotlin
> const map = new Map(); 
> function getWays(n) {
> 
>     if (n < 1) return 0;
>     if (n == 1) return 1;
>     if (n == 2) return 2;
> 
>     if (map.has(n)) {
>         return map.get(n);
>     }
>     const value = getWays(n-1) + getWays(n-2);
>     map.set(n, value);
>     return value; 
> }
> ```

因为map里最终会存放n-2个键值对，所以空间复杂度为**O(n)**,时间复杂度也为**O(n)**

继续想一想这就是最优的解决方案了吗？

我们回到一开始的思路，我们是假定前面的楼梯已经走完，只考虑最后一步，所以才得出来f(n) = f(n-1) + f(n-2)的递归式，这是一个置顶向下求解的式子
一般来说，按照正常的思路应该是一步一步往上走，应该是自底向上去求解才比较符合正常人的思维，我们来看看行不行的通

![图片描述](https://segmentfault.com/img/bVbe23m?w=1258&h=226)
这是一开始走的一个和两个楼梯的走法数,即之前说的**初始状态**

![图片描述](https://segmentfault.com/img/bVbe23E?w=1262&h=226)
这是进行了一次迭代得出了3个楼梯的走法，f(3)只依赖于f(1) 和 f(2)

继续看下一步
![图片描述](https://segmentfault.com/img/bVbe23Z?w=1270&h=226)
这里又进行了一次迭代得出了4个楼梯的走法，f(4)只依赖于f(2) 和 f(3)

我们发现每次迭代只需要前两次迭代的数据，不用像备忘录一样去保存所有子状态的数据

#### 方法三：动态规划求解

> ```javascript
> function getWays(n) {
> 
>     if (n < 1) return 0;
>     if (n == 1) return 1;
>     if (n == 2) return 2;
> 
>     // a保存倒数第二个子状态数据，b保存倒数第一个子状态数据， temp 保存当前状态的数据
>     let a = 1, b = 2;
>     let temp = a + b;
>     for (let i = 3; i <= n; i++) {
>         temp = a + b;
>         a = b;
>         b = temp; 
>     }
>     return temp; 
> }
> ```

这是我们可以再看看当前的时间复杂度和空间复杂度
当前时间复杂度仍为**O(n)**，但空间复杂度降为**O(1)**
这就是理想的结果

#### 总结

这只是动态规划里最简单的题目之一，因为它只有一个变化维度
当变化维度变成两个、三个甚至更多时，会更加复杂，背包问题就是比较典型的多维度问题，有兴趣的可以去网上看看《背包九讲》

### 打劫

我们定义**状态**：设dp[i]表示在前i间房子中可以偷得的最多钱数

**转移方程**为：

 dp[i] = max(dp[i - 2] + nums[i],dp[i - 1])

**初始化**：

 dp[0] = nums[0];

 dp[1] = max(nums[0] , nums[1]);

#### 代码

```java
public static int rob(int[] nums) {
    int len = nums.length;
    if (len == 0) return 0;
    if (len == 1) return nums[0];
    int[] dp = new int[len];
    //初始化
    dp[0] = nums[0];
    dp[1] = Math.max(nums[0],nums[1]);
    for (int i = 2; i < len; i++) {
        dp[i] = Math.max(dp[i - 2] + nums[i], dp[i - 1]);
    }
    return dp[len - 1];
}
```

通过分析，我们发现，实际上在整个代码的运行过程中，关键的循环部分每次都是只使用到了两个变量来进行变量的存储，那么上面的程序可以优化为：

```java
public static int rob(int[] nums) {
    int n = nums.length;
    if (n == 0) return 0;
    if (n == 1) return nums[0];
    int first = nums[0], second = nums[1];
    for (int i = 0; i < n; i++) {
        int temp = second;
        second = Math.max(first + nums[i], second);
        first = second;
    }
    return second;
}
```

### **打家劫舍-II**

#### **题目描述：**

你是一个专业的小偷，计划偷窃沿街的房屋，每间房内都藏有一定的现金。这个地方所有的房屋都 围成一圈 ，这意味着第一个房屋和最后一个房屋是紧挨着的。同时，相邻的房屋装有相互连通的防盗系统，如果两间相邻的房屋在同一晚上被小偷闯入，系统会自动报警 。

给定一个代表每个房屋存放金额的非负整数数组，计算你 在不触动警报装置的情况下 ，今晚能够偷窃到的最高金额。

![img](https://pic4.zhimg.com/80/v2-926a684e1321737d66319df5fcf9b2a3_1440w.webp)

示例 1：

输入：nums = [2,3,2]

输出：3

解释：你不能先偷窃 1 号房屋（金额 = 2），然后偷窃 3 号房屋（金额 = 2）, 因为他们是相邻的。

示例 2：

输入：nums = [1,2,3,1]

输出：4

解释：你可以先偷窃 1 号房屋（金额 = 1），然后偷窃 3 号房屋（金额 = 3）。 偷窃到的最高金额 = 1 + 3 = 4 。

示例 3：

输入：nums = [0]

输出：0

提示：

 1 <= nums.length <= 100 

0 <= nums[i] <= 1000

#### **题目分析：**

本题与第一题的最主要的区别在于：这里的房间变成了首尾相连的房子，这就意味着，**第一间房子和最后一间房子变成了相邻的两间房**,**偷了第一间房就不能再偷最后一间房，反之，偷了最后一间房就不能再偷第一间房。**

那么我们是不是能够单独计算从**第一间房到倒数第二间房**的部分和**第二间房到最后一间房**的部分中可以偷得的最高金额，然后通过比较返回最高金额

![img](https://pic3.zhimg.com/80/v2-989ab0c35550e9799ee712757407423a_1440w.webp)

直接看代码吧！

#### **代码：**

```java
public int rob(int[] nums) {
    int n = nums.length;
    if (n < 1) return n;
    if (n == 1) return nums[0];
    if (n == 2) return Math.max(nums[0], nums[1]);
    return Math.max(robRange(0, n - 2, nums), robRange(1, n - 1, nums));

}
public int robRange(int start,int end ,int[] nums){
    int first = nums[start], second = Math.max(nums[start], nums[start + 1]);
    for (int i = start + 2; i <= end; i++) {
        int temp = second;
        second = Math.max(first + nums[i], second);
        first = temp;
    }
    return second;
}
```







# algoLagou

## 数组：实现整数的数字反转 7

题目来源：Leetcode 7：https://leetcode-cn.com/problems/reverse-integer

## 暴力解法：逆序输出

解法一：暴力解法 

1. 整数转字符串，再转字符数组 
2. 反向遍历字符数组，并将元素存储到新数 组中

 3. 将新数组转成字符串，再转成整数输出

```java
public int reverse(int x) {
if(x == Integer.MAX_VALUE) {
// 整数类型最小值的绝对值 比 最大值的绝对值 大1
return 0; // 反转必然溢出，返回0
}
int sign = x > 0 ? 1 : -1; // 符号
x = x < 0 ? x * -1 : x; // 无论正负，都当成正数
// 1.整数转字符串，再转字符数组
String str = String.valueOf(x);
char[] chars = str.toCharArray();
// 2.交换首位(start)和末位(end)数字
// 3.循环操作：依次交换第二(start++)和倒数第二个(end--)
int start = 0，end = chars.length - 1;
while (start < end) { // 反转完成的标志：start >= end
// 交换两端等距离的元素
char temp = chars[start];
chars[start] = chars[end];
chars[end] = temp;
start++;
end--;
}
// 4.将原数组转成字符串，再转成整数输出
long value = Long.valueOf(String.valueOf(chars));
boolean b = value > Integer.MAX_VALUE || value < Integer.MIN_VALUE;
int result = b ? 0 : (int)value;
return result * sign;
}
```





# Algo  2


---
title: LeetCodea每日一题
date: 2021-04-05 21:06:35
tags:
---

## 20210426

```tex
题目：1011在D天内送达包裹的能力
-----------------------------
传送带上的包裹必须在 D 天内从一个港口运送到另一个港口。
传送带上的第 i 个包裹的重量为 weights[i]。每一天，我们都会按给出重量的顺序往传送带上装载包裹。我们装载的重量不会超过船的最大运载重量。
返回能在 D 天内将传送带上的所有包裹送达的船的最低运载能力
-----------------------------
输入：weights = [1,2,3,4,5,6,7,8,9,10], D = 5
输出：15
解释：
船舶最低载重 15 就能够在 5 天内送达所有包裹，如下所示：
第 1 天：1, 2, 3, 4, 5
第 2 天：6, 7
第 3 天：8
第 4 天：9
第 5 天：10
----------------------------
输入：weights = [3,2,2,4,1,4], D = 3
输出：6
解释：
船舶最低载重 6 就能够在 3 天内送达所有包裹，如下所示：
第 1 天：3, 2
第 2 天：2, 4
第 3 天：1, 4
----------------------------
输入：weights = [1,2,3,1,1], D = 4
输出：3
解释：
第 1 天：1
第 2 天：2
第 3 天：3
第 4 天：1, 1
```

<!-- more --> 

```java
class Solution{
    public int shipWithinDay(int[] weights, int D){
        int left = Arrays.stream(weights).max().getAsInt(), right = Arrays.stream(weights).sum();
        while(left < right){
            int mid = ((right - left) >> 1) + left;
            int day = 1, sum = 0;
            for(int w : weights){
                if(w + sum > mid){
                    day++;
                    sum = 0;
                }
                sum += w;
            }
            if(day <= D){
                right = mid;
            }else{
                left = mid + 1;
            }
        }
        return left;
    }
}
```

```tex
解题思路：采用二分查找的方法。先找到左边界，由于船舶的最低运载量必须大于等于货物中最大的值，这样才能保证能够将货物运送。右边界为船舶一天将所有的货物运算完的运载量。
```

## 20210425

```tex
题目：897递增顺序搜索树
---------------------
给你一棵二叉搜索树，请你 按中序遍历 将其重新排列为一棵递增顺序搜索树，使树中最左边的节点成为树的根节点，并且每个节点没有左子节点，只有一个右子节点。
---------------------
输入：root = [5,3,6,2,4,null,8,1,null,null,null,7,9]
输出：[1,null,2,null,3,null,4,null,5,null,6,null,7,null,8,null,9]
----------------------
输入：root = [5,1,7]
输出：[1,null,5,null,7]
```

```java
class Solution{
    private TreeNode res;
    
    public TreeNode increasingBST(TreeNode root){
        TreeNode dummy = new TreeNode(-1);
        res = dummy;
        inorder(root);
        return dummy.right;
    }
    
    private void inorder(TreeNode root){
        if(root == null){
            return;
        }
        inorder(root.left);
        res.right = root;
        root.left = null;
        res = root;
        
        inorder(root.right);
    }
}
```

```tex
解题思路：采用二叉树的中序遍历，并修改其二叉树的结构。
```

## 20210424

```txt
题目：377组合总和IV
-------------------
给你一个由 不同 整数组成的数组 nums ，和一个目标整数 target 。请你从 nums 中找出并返回总和为 target 的元素组合的个数。
-------------------
输入：nums = [1,2,3], target = 4
输出：7
解释：
所有可能的组合为：
(1, 1, 1, 1)
(1, 1, 2)
(1, 2, 1)
(1, 3)
(2, 1, 1)
(2, 2)
(3, 1)
请注意，顺序不同的序列被视作不同的组合。
---------------------
输入：nums = [9], target = 3
输出：0
```

```java
class Solution{
    public int combinationSum4(int[] nums, int target){
        int[] dp = new int[target+1];
        dp[0] = 1;
        for(int i = 1; i <= target; i++){
            for(int n : nums){
                if(n <= i){
                    dp[i] += dp[i-n];
                }
            }
        }
        return dp[target];
    }
}
```

## 20210423

```txt
题目：368最大整除子集
--------------------
给你一个由 无重复 正整数组成的集合 nums ，请你找出并返回其中最大的整除子集 answer ，子集中每一元素对 (answer[i], answer[j]) 都应当满足：
answer[i] % answer[j] == 0 ，或
answer[j] % answer[i] == 0
如果存在多个有效解子集，返回其中任何一个均可。
----------------------
输入：nums = [1,2,3]
输出：[1,2]
解释：[1,3] 也会被视为正确答案。
----------------------
输入：nums = [1,2,4,8]
输出：[1,2,4,8]
```

```java
class Solution{
    public List<Integer> largestDivisibleSubset(int[] nums) {
        int N = nums.length;
        if(N == 0){
            return new ArrayLis();
        }
        Arrays.sort(nums);
        int maxIndex = 0;
        List<Integer>[] res = new ArrayList[N];
        for(int i = 0; i < N; i++){
            int max = i;
            int maxLen = 0;
            for(int j = 0; j < i; j++){
                if(nums[i] % nums[j] == 0 && maxLen < res[j].size()){
                    max = j;
                    maxLen = res[j].size();
                }
            }
            res[i] = new ArrayList();
            res[i].addAll(res[max]);
            res[i].add(nums[i]);
            if(res[maxIndex].size() < res[i].size()){
                maxIndex = i;
            }
        }
        return res[maxIndex];
    }
}
```

## 20210422

```txt
题目：363矩形区域不超过 K 的最大数值和
----------------------------------
给你一个 m x n 的矩阵 matrix 和一个整数 k ，找出并返回矩阵内部矩形区域的不超过 k 的最大数值和。
题目数据保证总会存在一个数值和不超过 k 的矩形区域。
----------------------------------
输入：matrix = [[1,0,1],[0,-2,3]], k = 2
输出：2
解释：蓝色边框圈出来的矩形区域 [[0, 1], [-2, 3]] 的数值和是 2，且 2 是不超过 k 的最大数字（k = 2）。
------------------------------------
输入：matrix = [[2,2,-1]], k = 3
输出：3
```

 ```java
class Solution {
    public int maxSumSubmatrix(int[][] matrix, int k) {
        
        int row = matrix.length, col = matrix[0].length;
        int[][] sum = new int[row+1][col+1];
        for(int i = 1; i < row+1; i++){
            for(int j = 1; j < col+1; j++){
                sum[i][j] = sum[i-1][j] + sum[i][j-1] - sum[i-1][j-1] + matrix[i-1][j-1];
            }
        }

        int ans = Integer.MIN_VALUE;
        for(int i = 1; i < row+1; i++){
            for(int j = 1; j < col+1; j++){
                for(int p = i; p < row+1; p++){
                    for(int q = j; q < col+1; q++){
                        int cur = sum[p][q] - sum[i-1][q] - sum[p][j-1] + sum[i-1][j-1];
                        if(cur <= k){
                            ans = Math.max(ans, cur);
                        }
                    }
                }
            }
        }
        return ans;
    }
}
 ```

## 20210421

```txt
题目：91解码方法
---------------------
一条包含字母A-Z的消息通过以下映射进行了编码 ：
'A' -> 1
'B' -> 2
...
'Z' -> 26
要解码已编码的消息，所有数字必须基于上述映射的方法，反向映射回字母（可能有多种方法）。例如，"11106" 可以映射为：
"AAJF" ，将消息分组为 (1 1 10 6)
"KJF" ，将消息分组为 (11 10 6)
注意，消息不能分组为(1 11 06) ，因为"06"不能映射为"F"，这是由于"6"和"06"在映射中并不等价。
--------------------------
输入：s = "12"
输出：2
解释：它可以解码为 "AB"（1 2）或者 "L"（12）。
--------------------------
输入：s = "226"
输出：3
解释：它可以解码为 "BZ" (2 26), "VF" (22 6), 或者 "BBF" (2 2 6) 。
--------------------------
输入：s = "0"
输出：0
解释：没有字符映射到以 0 开头的数字。
含有 0 的有效映射是 'J' -> "10" 和 'T'-> "20" 。
由于没有字符，因此没有有效的方法对此进行解码，因为所有数字都需要映射。
---------------------------
输入：s = "06"
输出：0
解释："06" 不能映射到 "F" ，因为字符串含有前导 0（"6" 和 "06" 在映射中并不等价）。
```

```java
class Solution {
    public int numDecodings(String s) {
        if(s.charAt(0) == '0'){
            return 0;
        }
        int N = s.length();
        int[] dp = new int[N];
        dp[0] = 1;
        for(int i = 1; i < N; i++){
            if(s.charAt(i) != '0'){
                dp[i] += dp[i-1];
            }
            if(s.charAt(i-1) == '1' || s.charAt(i-1) == '2' && s.charAt(i) <= '6'){
                if(i > 1){
                    dp[i] += dp[i-2];
                }else{
                    dp[i]++;
                }
            }
        }
        return dp[N-1];
    }
}
```

```txt
解题思路：采用动态规划的方法。若第一个字符为'0'则直接输出0；不然从第二个字符开始遍历，若该字符不为'0'，则它就可以被解码成一个字母。则我们可以写出状态转移方程：dp[i] += dp[i-1]
第二种情况，我们使用了两个字符，当满足前一个字符==1或者前一个字符==2并且当前字符<=6则条件成立。此时可以写出状态转移方程：dp[i] += dp[i-2]，注意i需要大于1才可。
```

## 20210404

```txt
题目：781森林中的兔子
------------------------
描述：森林中，每个兔子都有颜色。其中一些兔子(可能是全部)告诉你还有其他的兔子和自己有相同的颜色，我们将这些回答放在answers数组中。返回森林中兔子的最少数量。
------------------------
输入：answers=[1,1,2]
输出：5
解释：两只回答了“1”的兔子可能有相同的颜色，设为红色。之后回答了“2”的兔子不会是红色，否则它们的回答会相互矛盾。设回答了“2”的兔子为蓝色。此外，森林中还应有另外2只蓝色兔子的回答没有包含在数组中。因此森林中兔子的最少数量是5：3只回答的和2只没有回答。
--------------------------
输入：answers = [10,10,10]
输出：11
--------------------------
输入：answers = []
输出：0
```

```java
class Solution{
    public int numRabbits(int[] answers){
        Map<Integer, Integer> count = new HashMap<Integer, Integer>();
        for(int a : answers){
            count.put(a, count.getOrDefault(a, 0) + 1);
        }
        int res = 0;
        for(Map.Entry<Integer, Integer> entry : count.entrySet()){
            int x = entry.getKey(), y = entry.getValue();
            res += (x+y)/(x+1)*(x+1);
        }
        return res;
    }
}
```

```txt
解题思路：两只相同颜色的兔子看到的其他同色兔子数必然是相同的。反之，若两只兔子看到的其他同色兔子数不同，那么这两只颜色也不同。因此，将answers中值相同的元素分为一组，对于每一组，计算出兔子的最少数量，然后后将所有的计算结果类加，就是最终答案。
一般地，如果回答的x的兔子有y只，则至少有(x+y)/(x+1)种不同的颜色，且每种颜色都有(x+1)只兔子。
```

## 20210405

```txt
题目：88合并两个有序数组
----------------------
描述：给你两个有序整数数组nums1和nums2，请你将nums2合并到nums1中，使nums1成为一个有序数组。初始化nums1和nums2的元素数量分别为m和n。你可以假设nums1的空间大小等于m+n，这样它就有足够的空间保存来自nums2的元素。
----------------------
输入：nums1=[1,2,3,0,0,0],m=3,nums2=[2,5,6],n=3
输出：[1,2,2,3,5,6]
----------------------
输入：nums1=[1],m=1,nums2=[],n=0
输出：[1]
```

```java
class Solution{
    public void merge(int[] nums1, int m, int[] nums2, int n){
        if(n == 0){
            return;
        }
        int i = m - 1, j = n - 1, k = m + n - 1;
        while(j >= 0){
            if(i >= 0 && nums1[i] > nums2[j]){
                nums1[k--] = nums1[i--];
            }else{
                nums1[k--] = nums2[j--];
            }
        }
    }
}
```

```txt
解题思路：逆向双指针法！在nums1的后半部分是空的，可以直接覆盖不受影响。可将指针k设置为从后向前遍历，每次取值为i，j指针中较大的值；i指针和j指针分别从nums1的最大数值和nums2的最大数值开始；若当i指针<0时说明nums剩下的数都可以直接赋给k指针指向剩下的数组；若当j指针<0时，nums1中数组已经有序，合并结束。
```

## 20210406

```txt
题目：80删除有序数组中的重复项II
------------------------------
描述：给你一个有序数组nums,请你原地删除重复出现的元素，使每个元素最多出现两次，返回删除后数组的新长度。不要使用额外的空间，你必须在原地修改输入数组并在使用O(1)额外空间的条件下完成。
------------------------------
输入：nums=[1,1,1,2,2,3]
输出：5，nums=[1,1,2,2,3]
------------------------------
输入：nums=[0,0,1,1,1,1,2,3,3]
输出：nums=[0,0,1,1,2,3,3]
```

```java
class Solution{
    public int removeDuplicates(int[] nums){
        if(nums.length <= 2){
            return nums.length;
        }
        int i = 2;
        for(int j = 2; j < nums.length; j++){
            if(nums[i-2] != nums[j]){
                nums[i] = nums[j];
                i++;
            }
        }
        return i;
    }
}
```

```txt
解题思路：给定的是有序数组，若元素相等则元素必会连续。我们使用双指针来解决本题，遍历数组中的每一个元素并检查其是否应该被保留。定义两个快慢指针i,j；慢指针i用来表名数组中的元素个数，快指针j用来检查元素是否应该加入新数组中。本题要求相同元素最多出现两次，所以我们应该检查上上个保留的元素nums[i-2]是否与当前检查的元素nums[j]相等。若nums[i-2]=nums[j]则，检查元素不应该放入新数组中，让其检查后续元素；若nums[i-2]!=nums[j]，则将当前元素保存下来，慢指针向后移动。当快指针全部遍历完毕时结束。
时间复杂度：O(n);空间复杂度O(1).
```



## 20210407

```txt
题目：81搜索旋转排序数组II
-------------------------
描述：已知存在一个按非降序排列的整数数组nums，数组中的值不必互不相同。在传递给函数之前，nums在预先未知的某个下标k上进行了旋转，使数组变为[nuk[k], nums[k+1]],...,nums[n-1], nums[0], nums[l], ...,nums[k-1]]。例如，[0,1,2,4,4,4,5,6,6,7]在下标5处经旋转后可能变为[4,5,6,6,7,0,1,2,4,4].给你旋转后的数组nums和一个整数target，请你编写一个函数来判断给定的目标是否存在于数组中。如果nums中存在这个目标值target,则返回true，否则返回false。
--------------------------
输入：nums=[2,5,6,0,0,1,2],target=0
输出：true
--------------------------
输入：nums[2,5,6,0,0,1,2],target=3
输出：false
```

```java
class Solution{
    public boolean search(int[] nums, int target){
        if(nums.length == 0){
            return false;
        }
        if(nums.length == 1){
            return nums[0] == target;
        }
        int i = 0, j = nums.length - 1;
        while(i <= j){
            int mid = ((j - i) >> 1) + i;
            if(nums[mid] == target){
                return true;
            }
            if(nums[i] == nums[mid] && nums[mid] == nums[j]){
                i++;
                j--;
            }else if(nums[i] <= nums[mid]){
                if(nums[i] <= target && target < nums[mid]){
                    j = mid - 1;
                }else{
                    i = mid + 1;
                }
            }else{
                if(nums[mid] < target && target <= nums[j]){
                    i = mid + 1;
                }else{
                    j = mid - 1;
                }
            }
        }
        return false;
    }
}
```

```txt
我们使用二分查找法来解决该问题。但是二分查找可能会出现a[i]=a[mid]=a[j]，此时无法判断左右两部分哪边是有序的。对于上述的问题我们可以当前二分区域的左边界+1，右边界-1，然后在新的分区上搜索。二分查找的思路可以详见33题。
时间复杂度：O(n);空间复杂度：O(1)。
```

## 20210408

```txt
题目：153寻找旋转排序数组中的最小值
--------------------------------
描述：已知一个长度为n的数组，预先按照升序排序，经由1到n此旋转后，得到输入数组。例如，原数组nums=[0,1,2,4,5,6,7]在变化后可能得到：
若旋转4次，则可以得到[4,5,6,7,0,1,2]
若旋转7次，则可以得到[0,1,2,4,5,6,7]
给你一个元素互不相同的数组nums，它原来是一个升序排序的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的最小元素。
-----------------------------------
输入：nums=[3,4,5,1,2]
输出：1
-----------------------------------
输入：nums=[4,5,6,7,0,1,2]
输出：0
------------------------------------
输入：nums=[11,13,15,17]
输出：11
```

```java
class Solution{
    public int findMin(int[] nums){
        int i = 0, j = nums.length;
        while(i < j){
            int mid = ((j - i) >> 1) + i;
            if(nums[mid] < nums[j]){
                j = mid;
            }else{
                i = mid + 1;
            }
        }
        return nums[j];
    }
}
```

```txt
我们使用二分法来查找最小数，原数组为一个升序排序数组，经过旋转后，通过数组中间的数值将这个数组分为两个部分，其中必有一部分为有序数组。我们用中间值与数组最后的值做比较，若中间值严格小于最后的值，那么数组的最小值应该在[左边界，中间值所在索引]之间，否则数组的最小值应该在[中间值索引+1，有边界]之间。当区间长度为1时，停止二分查找。
时间复杂度为：O(logn);空间复杂度为：O(1)
```

## 20210409

```txt
题目：154寻找旋转排序数组中的最小值II
---------------------------------
描述：已知一个长度为n的数组，预先按照升序排序，经由1到n此旋转后，得到输入数组。例如，原数组nums=[0,1,2,4,5,6,7]在变化后可能得到：
若旋转4次，则可以得到[4,5,6,7,0,1,2]
若旋转7次，则可以得到[0,1,2,4,5,6,7]
给你一个可能存在重复元素的数组nums，它原来是一个升序排序的数组，并按上述情形进行了多次旋转。请你找出并返回数组中的最小元素。
---------------------------------
输入：nums=[1,3,5]
输出：1
--------------------------------
输入：nums=[2,2,2,0,1]
输出：0
```

```java
class Solution{
    public int findMin(int nums){
        int i = 0, j = nums.length - 1;
        while(i < j){
            int mid = ((j - i) >> 1) + i;
            if(nums[i] == nums[mid] && nums[mid] == nums[j]){
                i++;
                j--;
            }else if(nums[mid] < nums[j]){
                j = mid;
            }else{
                i = mid + 1;
            }
        }
        return nums[j];
    }
}
```

```txt
我们使用二分查找。该数组经过旋转后，从中间切开，可以将数组分为两部分。由于数组中可能存在相等的数即nums[左边界]=nums[mid]=nums[有边界]，我们就让左边界+1，右边界-1.我们将比较nums[中间索引]和nums[有边界索引]，若nums[mid]严格小于nums[右边界]则让有边界赋值为mid，否则将左边界赋值为mid+1。
```

## 20210410

```txt
题目：263丑数
-----------------------------------------
描述：给你一个整数n，请你判断n是否为丑数，如果是，返回true；否则，返回false。
丑数就是只包含质因数2，3或5的正整数。
------------------------------------------
输入：n = 6
输出：true
解释：6 = 2 × 3
-----------------------------------------
输入：n = 8
输出：true
解释：8 = 2 × 2 × 2
------------------------------------------
输入：n = 14
输出：false
解释：14 不是丑数，因为它包含了另外一个质因数 7 。
------------------------------------------
输入：n = 1
输出：true
解释：1 通常被视为丑数。
```

```java
class Solution{
    public boolean isUgly(int n){
        if(n <= 0){
            return false;
        }
        if(n == 1){
            return true;
        }
        if(n % 2 == 0){
            return isUgly(n / 2);
        }
        if(n % 3 == 0){
            return isUgly(n / 3);
        }
        if(n % 5 == 0){
            return isUgly(n / 5);
        }       
        return false;
    }
}
```

```txt
采用递归的方法。若n能被2，3，5整除，将其先整除再继续迭代。递归的出口是若n==1则为丑数，否则不为丑数。
```

## 20210411

```txt
题目：264丑数II
---------------------------------
给你一个整数n，请你找出并返回第n个丑数。
丑数就是只包含质因数2、3或5的正整数。
---------------------------------
输入：n = 10
输出：12
解释：[1, 2, 3, 4, 5, 6, 8, 9, 10, 12] 是由前 10 个丑数组成的序列。
-----------------------------------
输入：n = 1
输出：1
解释：1 通常被视为丑数。
```

```java
class Solution{
    public int nthUglyNumber(int n){
        int[] res = new int[n+1];
        res[1] = 1;
        int p1 = 1, p2 = 1; p3 = 1;
        for(int i = 2; i <= n; i++){
            int num1 = res[p1] * 2, num2 = res[p2] * 3, num3 = res[p3] * 5;
            res[i] = Math.min(Math.min(num1, num2), num3);
            if(res[i] == num1){
                p1++;
            }
            if(res[i] == num2){
                p2++;
            }
            if(res[i] == num3){
                p3++;
            }
        }
        return res[n];
    }
}
```

```txt
采用三指针法，首先最小的丑数是1，再res中直接放入即res[1]=1；再通过三个指针来得到下一个丑数，让当前的丑数值乘以2，3，5得到对应的三个数，然后取其中最小的作为下一个丑数。
然后分别比较得到的丑数值是否等于这三个指针得到三个数，若相同则指针向前移。
```

## 20210412

```txt
题目：179最大数
-------------------
给定一组非负整数 nums，重新排列每个数的顺序（每个数不可拆分）使之组成一个最大的整数。
--------------------
输入：nums = [10,2]
输出："210"
--------------------
输入：nums = [3,30,34,5,9]
输出："9534330"
---------------------
输入：nums = [1]
输出："1"
---------------------
输入：nums = [10]
输出："10"
```

```java
class Solution {
    public String largestNumber(int[] nums) {
        if(nums.length == 1){
            return nums[0] + "";
        }

        String[] str = new String[nums.length];
        for(int i = 0; i < nums.length; i++){
            str[i] = nums[i] + "";
        }

        Arrays.sort(str, (a, b) -> {
            return (b+a).compareTo(a+b);
        });

        StringBuilder sb = new StringBuilder();
        for(String s : str){
            sb.append(s);
        }

        return sb.charAt(0) == '0' ? "0" : sb.toString();
    }
}
```

## 20210413

```txt
题目：783二叉搜索树节点最小距离
-----------------------------
给你一个二叉搜索树的根节点 root ，返回 树中任意两不同节点值之间的最小差值 。
-----------------------------
输入：root = [4,2,6,1,3]
输出：1
-----------------------------
输入：root = [1,0,48,null,null,12,49]
输出：1
```

```java
/**
 * Definition for a binary tree node.
 * public class TreeNode {
 *     int val;
 *     TreeNode left;
 *     TreeNode right;
 *     TreeNode() {}
 *     TreeNode(int val) { this.val = val; }
 *     TreeNode(int val, TreeNode left, TreeNode right) {
 *         this.val = val;
 *         this.left = left;
 *         this.right = right;
 *     }
 * }
 */
class Solution {
    int pre;
    int ans;

    public int minDiffInBST(TreeNode root) {
        pre = -1;
        ans = Integer.MAX_VALUE;
        dfs(root);
        return ans;
    }

    private void dfs(TreeNode root){
        if(root == null){
            return;
        }

        dfs(root.left);
        if(pre == -1){
            pre = root.val;
        }else{
            ans = Math.min(ans, root.val - pre);
            pre = root.val;
        }
        dfs(root.right);
    }
}
```

```txt
题目：208实现Trie
-------------------
Trie（发音类似 "try"）或者说 前缀树 是一种树形数据结构，用于高效地存储和检索字符串数据集中的键。这一数据结构有相当多的应用情景，例如自动补完和拼写检查。
请你实现 Trie 类：
Trie() 初始化前缀树对象。
void insert(String word) 向前缀树中插入字符串 word 。
boolean search(String word) 如果字符串 word 在前缀树中，返回 true（即，在检索之前已经插入）；否则，返回 false 。
boolean startsWith(String prefix) 如果之前已经插入的字符串 word 的前缀之一为 prefix ，返回 true ；否则，返回 false 。
--------------------
输入
["Trie", "insert", "search", "search", "startsWith", "insert", "search"]
[[], ["apple"], ["apple"], ["app"], ["app"], ["app"], ["app"]]
输出
[null, null, true, false, true, null, true]
```

```java
class Trie {

    Trie[] child;
    boolean isEnd;

    /** Initialize your data structure here. */
    public Trie() {
        child = new Trie[26];
        isEnd = false;
    }
    
    /** Inserts a word into the trie. */
    public void insert(String word) {
        Trie node = this;
        for(int i = 0 ; i < word.length(); i++){
            char ch = word.charAt(i);
            int index = ch - 'a';
            if(node.child[index] == null){
                node.child[index] = new Trie();
            }
            node = node.child[index];
        }
        node.isEnd = true;
    }
    
    /** Returns if the word is in the trie. */
    public boolean search(String word) {
        Trie node = this;
        for(int i = 0 ; i < word.length(); i++){
            char ch = word.charAt(i);
            int index = ch - 'a';
            if(node.child[index] == null){
                return false;
            }
            node = node.child[index];
        }
        return node.isEnd;
    }
    
    /** Returns if there is any word in the trie that starts with the given prefix. */
    public boolean startsWith(String prefix) {
        Trie node = this;
        for(int i = 0 ; i < prefix.length(); i++){
            char ch = prefix.charAt(i);
            int index = ch - 'a';
            if(node.child[index] == null){
                return false;
            }
            node = node.child[index];
        }
        return true;
    }
}
```




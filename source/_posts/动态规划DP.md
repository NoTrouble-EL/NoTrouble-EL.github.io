---
title: 动态规划DP
date: 2021-07-10 22:07:36
tags:
- DataStructure
- DP
- LeetCode
categories: LeetCode
mathjax: true
---

## 62.不同路径

```java
class Solution {
    public int uniquePaths(int m, int n) {
        //创建一个dp二维数组
        int[][] dp = new int[m][n];

        //dp[0][0] = 1，起点所在只有一种路径可达
        dp[0][0] = 1;
        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                //若路径不是在上边界和左边界上，则路径为上方加左方
                //若路径在左边界上，则路径为上方
                //若路径在上边界上，则路径为左方
                if(i > 0 && j > 0){
                    dp[i][j] = dp[i-1][j] + dp[i][j-1];
                }else if(i > 0){
                    dp[i][j] = dp[i-1][j];
                }else if(j > 0){
                    dp[i][j] = dp[i][j-1];
                }        
            }
        }

        return dp[m-1][n-1];
    }
}
```

 <!-- more --> 

## 63.不同路径II

```java
class Solution {
    public int uniquePathsWithObstacles(int[][] obstacleGrid) {
        //获取二维数组大小
        int m = obstacleGrid.length, n = obstacleGrid[0].length;

        //创建二维dp数组
        int[][] dp = new int[m][n];
        //起始点的路径取决于起始点是否有障碍
        dp[0][0] = obstacleGrid[0][0] == 1 ? 0 : 1;

        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                //检查该路径上是否有障碍
                if(obstacleGrid[i][j] != 1){
                    //思路和不同路径一样
                    if(i > 0 && j > 0){
                        dp[i][j] = dp[i-1][j] + dp[i][j-1];
                    }else if(i > 0){
                        dp[i][j] = dp[i-1][j];
                    }else if(j > 0){
                        dp[i][j] = dp[i][j-1];
                    }
                }
            }
        }

        return dp[m-1][n-1];
    }
}
```

## 64.最小路径和

```java
class Solution {
    public int minPathSum(int[][] grid) {
        //获取二维数组大小
        int m = grid.length, n = grid[0].length;

        //创建dp数组
        int dp[][] = new int[m][n];
        //起点路径的权值为当前的权值
        dp[0][0] = grid[0][0];

        for(int i = 0; i < m; i++){
            for(int j = 0; j < n; j++){
                //获取当前路径的权值
                int val = grid[i][j];

                //若路径不是上边界和左边界，则当前路径权重为其上和左最小值+当前权值
                //若路径为左边界上，则当前路径的权值为其上+当前权值
                //若路径为上边界，则当前路径的权值为其左+当前权值
                if(i > 0 && j > 0){
                    dp[i][j] = Math.min(dp[i-1][j], dp[i][j-1]) + val;
                }else if(i > 0){
                    dp[i][j] = dp[i-1][j] + val;
                }else if(j > 0){
                    dp[i][j] = dp[i][j-1] + val;
                }
            }
        }

        return dp[m-1][n-1];
    }
}
```


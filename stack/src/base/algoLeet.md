# Leetcode

### 跳跃游戏I

<img src="D:\docum\picture\image-20230920174957266.png" alt="image-20230920174957266" style="zoom:50%;" />

```shell
1. 控制代码：for if
2. 运算代码：贪心  动态规划
```

```java
class Solution {
    public boolean canJump(int[] nums) {
        int k=0;// 定义最大跳跃位置
        for(int i=0;i<nums.length;i++){
            if(i>k) return false;
            k=Math.max(k,i+nums[i]);
        }
        return true;
    }
}
```





### 跳跃游戏II

<img src="D:\docum\picture\image-20230920182506741.png" alt="image-20230920182506741" style="zoom:50%;" />

```java
class Solution {
    public int jump(int[] nums) {
        int k=0,step=0;//最大位置 ，步长
        int end=0;//贪心步长
        for(int i=0;i<nums.length-1;i++){
            k= Math.max(k,i+nums[i]);
            //k =        end   i
            //(0,2) 2     2    0
            //(2,4) 4      2     1
            //(4,3) 4       4     2
            //(4,5) 5       4    3
            //(5,8)  8       8   4
            //(8,7)  8       8   5
            //(8,9)  9       8   6
            //找到贪心位置，移动步长
            if(i==end){
                end =k;
                step++;
            }
        }
        return step;
    }
}
```


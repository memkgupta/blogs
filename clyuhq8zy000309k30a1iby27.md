---
title: "Find Valid Matrix Given Row and Column Sums"
seoTitle: "Find Valid Matrix Given Row and Column Sums"
datePublished: Sat Jul 20 2024 18:56:48 GMT+0000 (Coordinated Universal Time)
cuid: clyuhq8zy000309k30a1iby27
slug: find-valid-matrix-given-row-and-column-sums
tags: matrix, dsa, leetcode, problem-of-the-day

---

In this article i am going to explain the approach and solution for this leetcode problem

In the given problem we are given with two arrays one is rowSum and other is colSum  
basically what they are nothing but arrays of the sum of ith row and jth column of the matrix and we have to find the valid matrix.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721499212623/9f3056d8-d521-4655-aa82-f7c3f002f162.png align="center")

### **So what will be our approach ?**

It is simple what we will be doing is taking out maximum valid element possible for that position of matrix by selecting the minimum of that particular position's row and column and then updating that particular position' row and column sum by subtracting the choosen value from both row sum and column sum.  
Let's understand by the example <mark>rowSum = [5,7,10], colSum = [8,6,8]</mark>

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721499839354/96d6aef5-125c-4c08-848a-2070a2f617cc.png align="center")

Now if either the `rowSum[i`\] or `colSum[j]` becomes 0 we can say, that particular row or column is locked as the condition for that row is satisfied so we will move our j or i pointer according to that.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721499957792/ce214e03-4408-45bc-9fdd-350284361a2a.png align="center")

And since our rowSum\[i\] becomes zero we can move our i pointer or row to next

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721500255006/89e7b8a6-9569-4a3e-8e80-7977f0732ac1.png align="center")

Now for this row we have colSum allowed is 3 and rowSum allowed is 7 hence we will choose min i.e 3 and by putting 3 our colSum\[0\] will also become 0

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721500373293/42287d96-03b6-475c-9d94-64d40f18aef7.png align="center")

We will move our column pointer to the next . and will repeat the same procedure

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721500627268/810755bb-a4ea-4095-ab03-28387d2a78db.png align="center")

rowSum\[1\] becomes zero we move our row pointer to next.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721500816191/a927ecdf-a8b0-476e-ba2f-2b94bb06b707.png align="center")

Now here the minimum is 2 hence we choose 2 and subtract it from both rowSum\[2\] and colSum\[1\] , after this operation our colSum\[1\] becomes 0 hence we will move the col pointer.

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721501101759/6e273f86-5e0a-44b7-a64c-a9859daece86.png align="center")

After moving the pointer to the next column we choose 8 and subtract from both our both rowSum\[2\] and colSum\[2\] becomes 0 and also reaches to end hence no further operations.

### **Here is how our matrix will look for now**

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721501216085/860508d3-6faa-48fc-ab59-d7c235eec9fe.png align="center")

Now the remaining another positions will be filled with zero which is the default value.

Now if we have a final look at our matrix our matrix satisfies the condition and is our answer

![](https://cdn.hashnode.com/res/hashnode/image/upload/v1721501497454/8073bb79-2e45-4974-8277-404631e0d930.png align="center")

### Now I hope that I was able to explain the approach so **here is the code**

```java
class Solution {
    public int[][] restoreMatrix(int[] rowSum, int[] colSum) {
        int m = rowSum.length;
        int n = colSum.length;
        int i =0;
        int j = 0;
        int[][] mat= new int[m][n];
        while(i<m&&j<n){
            int el = Math.min(rowSum[i],colSum[j]);
            mat[i][j] = el;
            rowSum[i]-=el;
            colSum[j]-=el;
            if(rowSum[i]==0){
                i++;
            }
            if(colSum[j]==0){
                j++;
            }
        }
 return mat;
    }
}
```

The time complexity of this solution is O(m\*n) where m is the number of rows and n is the number of columns in the matrix. This is because we iterate through each element in the matrix once to fill in the values based on the rowSum and colSum arrays. The space complexity is O(m\*n) as well, as we are creating a new matrix of size m\*n to store the restored matrix values.

Thank you for reading will see you next time bye 🫡
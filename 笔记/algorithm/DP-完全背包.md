题目：

![](32.png)
![](33.png)


分析：

![](34.png)
![](35.png)


代码实现：

```javascript
/**  
 * @param {number[][]} items - [weight, value]  
 * @param {number} maxHeavy  
 */  
function resolve(items, maxHeavy) {  
  const len = items.length  
  const dp = new Array(len + 1)  
    .fill(null)  
    .map(() => new Array(maxHeavy + 1).fill(0))  
  
  for (let i = 0; i < len; i++) {  
    for (let j = 0; j <= maxHeavy; j++) {  
      if (j < items[i][0]) {  
        dp[i + 1][j] = dp[i][j] // 不拿这一个  
      } else {  
        dp[i + 1][j] = Math.max(  
          dp[i][j], // 不拿的情况  
  
          /**  
           * dp[i+1][j - items[i][1]]  代表上一次这个重量的 max 价值  
           * 加上这次拿上的价值计算 max  
           */          dp[i + 1][j - items[i][0]] + items[i][1],  
        )  
      }  
    }  
  }  
  
  dp.forEach((i) => console.log(i.join(', ')))  
  
  return dp[len][maxHeavy]  
}  
  
console.log(  
  resolve(  
    [  
      [3, 4],  
      [4, 5],  
      [2, 3],  
    ],  
    7,  
  ),  
)
```
  
题目  

![](29.png)

分析

![](30.png)
![](31.png)


代码实现

```javascript
/**  
 * @param {string} str1  
 * @param {string} str2  
 * @return {number}  
 */  
function resolve(str1, str2) {  
  const len1 = str1.length  
  const len2 = str2.length  
  const dp = new Array(len1 + 1)  
    .fill(null)  
    .map(() => new Array(len2 + 1).fill(0))  
  
  for (let i = 0; i < len1; i++) {  
    for (let j = 0; j < len2; j++) {  
      if (str1[i] === str2[j]) {  
        dp[i + 1][j + 1] = dp[i][j] + 1  
      } else {  
        dp[i + 1][j + 1] = Math.max(dp[i][j + 1], dp[i + 1][j])  
      }  
    }  
  }  
  
  return dp[len1][len2]  
}  
  
console.log(resolve('abcd', 'becd'))
```
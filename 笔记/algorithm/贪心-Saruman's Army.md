
![](笔记/algorithm/images/21.png)
![](笔记/algorithm/images/22.png)

思路：

![](笔记/algorithm/images/23.png)

代码实现：

```javascript
/**  
 * @param {number[]} points - Index of each point.  
 * @param range - The range  
 * @return {number} - The minimum num of point need to marked.  
 */function resolve(points, range) {  
  points.sort((a, b) => a - b)  
  let i = 0  
  const len = points.length  
  let result = 0  
  
  while (i < len) {  
    let s = points[i++] // 起始点  
  
    while (i < len && points[i] <= s + range) i++ // 找到需要标记的点  
  
    let p = points[i - 1] // 找到 range 之内的最后一个点坐标  
  
    while (i < len && points[i] <= p + range) i++ // 找到需要标记的点，也是下一次循环的起始点  
  
    result++  
  }  
  
  return result  
}  
  
console.log(resolve([1, 7, 15, 20, 30, 50], 10))
```
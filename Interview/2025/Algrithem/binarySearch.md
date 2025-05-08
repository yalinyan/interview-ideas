### **二分法（Binary Search）原理与操作模式**

二分法是一种基于**分治思想**的高效搜索算法，通过**不断缩小搜索范围**来快速定位目标值。其核心前提是：**数据必须有序**（单调性）。

---

## **核心原理**
1. **有序性依赖**  
   - 数据必须按升序或降序排列，确保可以通过中间值判断目标所在的半区。

2. **分治策略**  
   - 每次将搜索区间分为两部分，通过比较中间值与目标值，排除不可能的一半。

3. **终止条件**  
   - 找到目标值，或区间缩小到空（`left > right`）。

---

## **基本操作模式（框架）**
```python
def binary_search(nums, target):
    left, right = 0, len(nums) - 1  # 初始化搜索区间
    
    while left <= right:  # 终止条件：区间合法
        mid = left + (right - left) // 2  # 防溢出写法
        if nums[mid] == target:
            return mid  # 找到目标
        elif nums[mid] < target:
            left = mid + 1  # 目标在右半区
        else:
            right = mid - 1  # 目标在左半区
    
    return -1  # 未找到
```

### **关键点**
- **计算 `mid`**：推荐 `left + (right - left) // 2` 避免 `(left + right) // 2` 的溢出风险。
- **边界更新**：  
  - `left = mid + 1` 或 `right = mid - 1`，确保区间严格缩小，避免死循环。
- **终止条件**：  
  - `left <= right`（闭区间），若用 `left < right` 需额外处理剩余元素。

---

## **常见变体与场景**
### 1. **查找精确值**
- **场景**：有序数组中定位目标值。  
- **示例**：  
  ```python
  arr = [1, 3, 5, 7, 9]
  binary_search(arr, 5)  # 返回索引2
  ```

### 2. **查找左边界/右边界**
- **场景**：含重复元素时，找到第一个或最后一个等于目标的值。  
- **示例**（左边界）：  
  ```python
  def left_bound(nums, target):
      left, right = 0, len(nums) - 1
      while left <= right:
          mid = left + (right - left) // 2
          if nums[mid] < target:
              left = mid + 1
          else:
              right = mid - 1  # 即使等于也收缩右边界
      return left if left < len(nums) and nums[left] == target else -1
  ```

### 3. **寻找峰值或转折点**
- **场景**：部分有序或山脉数组（如 `[1,3,5,4,2]`）。  
- **示例**（峰值）：  
  ```python
  def find_peak(nums):
      left, right = 0, len(nums) - 1
      while left < right:
          mid = left + (right - left) // 2
          if nums[mid] < nums[mid + 1]:
              left = mid + 1  # 峰值在右侧
          else:
              right = mid  # 峰值在左侧或当前
      return left
  ```

### 4. **数值逼近问题**
- **场景**：求平方根、最小满足条件的值等。  
- **示例**（求平方根）：  
  ```python
  def sqrt(x):
      left, right = 0, x
      while left <= right:
          mid = left + (right - left) // 2
          if mid * mid <= x < (mid + 1) * (mid + 1):
              return mid
          elif mid * mid > x:
              right = mid - 1
          else:
              left = mid + 1
  ```

---

## **适用场景总结**
1. **有序数据检索**  
   - 数组、矩阵（如每行有序）、时间序列等。
2. **边界查找**  
   - 如插入位置（LeetCode 35）、重复元素的起止位置。
3. **数学逼近问题**  
   - 平方根、对数、方程解等。
4. **优化问题**  
   - 最小值最大化（如分配问题）、最大值最小化（如负载均衡）。

---

## **与暴力法的对比**
| **维度**       | **二分法**             | **暴力法**           |
|----------------|-----------------------|---------------------|
| 时间复杂度     | O(log n)              | O(n)                |
| 空间复杂度     | O(1)                  | O(1)                |
| 适用条件       | 必须有序              | 无要求              |
| 适用场景       | 大规模有序数据        | 小规模或无序数据    |

---

## **注意事项**
1. **确保有序性**：若数据无序，需先排序（O(n log n)），否则结果错误。
2. **避免死循环**：更新边界时需严格缩小范围（如 `mid ± 1`）。
3. **处理重复值**：需明确是否需要左/右边界。

二分法通过**对数级时间复杂度**显著提升效率，是处理有序数据问题的首选算法。
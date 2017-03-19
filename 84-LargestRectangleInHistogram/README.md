### 题目分析  
**中文描述**  
n 个非负整数表示柱状图中每一条的高度，每条的宽度是 1，找出柱状图中最大的矩形。  
![1](http://www.leetcode.com/wp-content/uploads/2012/04/histogram_area.png)  
`heights = [2,1,5,6,2,3]`.  
最大的矩形面积 `area = 2 * 5`.  

### 题解
#### 解题思路（1）
最直观的想法是暴力枚举所有的矩形，找出面积最大的矩形，具体的实现在 `solution1.cpp` 中。  
以柱状图的每一条作为矩形的高 `heights[i]`，以这一条为中心，向两边扫描，直到两侧的高度小于  `heights[i]` 为止。  
以上图为例，以 `heights[0]` 为高度的矩形，两侧的高度都小于2，因此以 `heights[0]` 为高度的矩形的宽度 `width = 1`，面积 `area = heights[0] * width = 2 * 1 = 2`；  
以 `heights[1]` 为高度的矩形，直方图其他条的高度都大于 1，向两侧扫描直到柱状图的边界，因此以 `heights[1]` 为高度的矩形的宽度 `width = 6`，面积 `area = heights[1] * width = 1 * 6 = 6`。  
**算法正确性及复杂度**  
通过暴力的方法，我们枚举了所有的柱状条作为矩形的高，并向两侧扩展，相当于枚举所有的面积可能最大的矩形，不会遗漏使面积最大的矩形。  
由于每一个柱状条都需要向两侧扫描，因此 算法复杂度会达到 O(n^2)，submit 之后的结果是 `Time Limit Exceed（超时）`。  

####解题思路（2）
思路（1）的暴力枚举，没有考虑任何柱状图的性质。现在我们仔细地观察柱状图，可以看到一种`断崖`现象，比如上图的 `heights[1]=1`，它就是一个断崖点，它左侧的柱状条的高度大于或者等于它的高度，形式化地定义一下断崖点：  
若 `i > 0`, 且 `heights[i-1] >= heights[i]`，则 `i` 为一个断崖点。  
**断崖点的性质**  
- 断崖点会隔断它左侧的所有高度高于或者等于它的柱状条对后面（右侧）的影响：  
比如，上图的 `heights[4] = 2` 是一个断崖点，其左侧的 `heights[2] 和 heights[3]` 两个柱状条的高度作为矩形的高度时，因为断崖点 `heights[4]` 的存在矩形的范围都无法越过断崖点扩展到断崖点右侧。  

**利用断崖点减少计算**  
由于断崖点的隔断特性，在遇到断崖点的时候，我们就可以计算断崖点左侧高度大于或者等于它的柱状条所代表的矩形的面积了。*至于如何计算我们稍后再说*。  
既然遇到断崖点我们就会计算那些高度高于或者等于断崖点的柱状条，那么还没有计算的柱状条的高度就会组成一个递增序列，那么递增序列的性质和意义是什么：  
1. 以递增序列中的某一柱状条作为矩形高度扩展矩形时，向右可以扩展到递增序列中该柱状条右侧的柱状条对应的位置或柱状图开始；  
2. 以递增序列中的某一柱状条作为矩形高度扩展矩形时，向左可以扩展到当前断崖点或者柱状图结束。

*证明：*  
递增序列的某一柱状条在 `heights` 数组中对应的位置是 `i`;在递增序列中，该柱状条左侧的柱状条在 `heights` 数组中对应的位置是 `k`；在递增序列中，该柱状条右侧的柱状条在 `heights` 数组中对应的位置是 `j`。性质 1 的意思是 `heights[k+1...i-1] >= heights[i] && heights[i] < heights[i+1...j-1]`。  
反证法证明 `[k+1...i-1]` 之间的柱状条高于或等于 `heights[i]`：
- 若 `[k+1...i-1]` 之间的某一柱状条 `m` 小于 `heights[i]`，分两种情况讨论：  
如果 `heights[m] > heights[k]`,那么 `heights[m]` 在递增序列内或者 `heights[k] 或 heights[i]` 不在递增序列内，不管怎么样，`heights[k] 和 heights[i]`都无法保持当前相邻的位置和大小关系。  
如果 `heights[m] <= heights[k]`,`heights[m]` 是断崖点，`heights[k]` 不会再递增序列内。  
所以 `[k+1...i-1]` 之间的柱状条高于或者等于 `heights[i]`。  

同理证明 `[i+1...j-1]` 之间的柱状条高于 `heights[i]`：
- 若 `[i+1...j-1]` 之间的某一柱状条 `m` 小于或者等于 `heights[i]`，`heights[m]` 是断崖点，`heights[i]` 不会再递增序列内。  
所以 `[i+1...j-1]` 之间的柱状条高于 `heights[i]`。  

综上所述，由于递增序列的性质，我们可以得到一个有效的方法来计算断崖点左侧高度大于或等于它的柱状条所代表的矩形的面积。  
**计算断崖点左侧高度大于或等于它的柱状条所代表的矩形的面积**  
*算法 2.1*  
我们要维护一个栈结构 `index_stack` 来记录递增序列，具体地，栈中的内容是柱状条在 `heights` 数组中的索引。  
假设在当前柱状条 `j` 是断崖点，栈中已经记录了柱状条高度递增序列：  
1. 得到栈顶索引 `i` ，若当前索引 `i` 对应的高度小于断崖点高度，说明断崖点左侧高度大于或等于它的柱状条所代表的矩形的面积都已经计算完了，算法结束；  
2. 若当前索引 `i` 对应柱状条的高度大于或者等于断崖点高度，栈顶索引 `i` 出栈；
3. 取出索引 `i` 后，我们还要知道索引 `i` 对应的柱状条向右可以扩展多远，因此用变量 `k` 来记录最远可以扩展到的位置：若栈不为空，得到当前栈顶索引 `x`（但不从栈中取出索引 `x`），`k = x + 1`，若栈为空，`k = 0`。  
则该索引对应的柱状条所代表的矩形面积为 `area = heights[i] * (j - k)`。
4. 若栈不为空，回到 `1` 继续处理。

至此，我们已经有了找最大矩形的一切理论基础，在下面我将写出算法的整体流程。  
算法的具体实现在 `solution2.cpp` 中。  
**算法整体流程**  
*算法 2.2*  
从左向右依次扫描 `heights` 数组中的柱状条，维护一个栈结构 `index_stack`，对于每个柱状条执行以下操作：
- 如果栈为空，将柱状条的索引 `i` 压入栈中，继续扫描；
- 如果当前柱状条的高度大于栈顶索引对应的柱状条的高度，将当前柱状条的索引压入栈中，继续扫描；
- 如果栈顶索引对应的柱状条的高度大于或者等于当前柱状条的高度，当前柱状条是断崖点，使用 `算法 2.1` 计算断崖点左侧高度大于或等于它的柱状条所代表的矩形的面积。  
如果这个面积大于当前最大的面积，更新最大面积；
- 继续扫描 `heights` 数组中的下一个柱状条。

每个柱状条扫描之后，如果栈未空，说明还有柱状条没有处理，从栈顶开始对每个柱状条使用 `算法 2.1` 计算面积并更新最大面积。  
**算法复杂度**  
算法依次扫描 `heights` 数组中的柱状条，如果这个柱状条不是断崖点，直接入栈，如果是断崖点，栈内元素出栈并计算面积。  
每个柱状条只会入栈和出栈各一次，出栈时计算面积的时间复杂度是 `O(1)`。  
因此算法的整体时间复杂度为 `O(2 * n) = O(n)`。  
空间复杂度也是 `O(n)`.  

#### 解题思路（3）
思路（2）中，扫描 `heights` 数组中的柱状条后，如果栈不为空，还要处理栈中剩余的柱状条。  
为了避免处理未空的栈，可以在 `heights` 数组的最后加上一个高度为 `0` 的“哨兵” `heights.emplace_back(0)`，这样在扫描到哨兵的时候，由于栈内索引对应的柱状条高度都大于或者等于 `0`,这个哨兵就是一个断崖点，它会让栈中所有索引出栈，同时计算索引对应的柱状条所代表矩形的面积。  
具体的实现在 `solution3.cpp` 中。  

**简单举个例子**  
就用开头的例子来走一遍算法的流程：`heights = [2,1,5,6,2,3]` 。  
在 `heights` 数组的最后加上一个高度为 `0` 的“哨兵”，`heights = [2,1,5,6,2,3,0]`；栈结构 ` index_stack = []`,左侧为栈底，右侧为栈顶；当前最大面积 `max_area = 0`。  
- 扫描 `heights[0] = 2`，当前栈为空，直接入栈；  

`index_stack = [0]`  
- 扫描 `heights[1] = 1`，栈顶索引对应的柱状条高度大于当前柱状条 `heights[0] > heights[1]`（当前柱状条为断崖点）:  
取出栈顶索引 `0`，由于取出索引 `0` 后栈为空，以 `heights[0] = 2` 为高度的矩形向右可以扩展到位置 `k = 0`。  
计算面积 `area = heights[0] * (1 - 0) = 2`,`max_area = max(max_area, area) = max(0, 2) = 2`。  
当前索引点 `1` 入栈；

`index_stack = [1]`  
- 扫描 `heights[2] = 5`，当前柱状条高度大于栈顶索引对应的柱状条高度 `heights[2] > heights[1]`，直接入栈；

`index_stack = [1,2]`  
- 扫描 `heights[3] = 6`，当前柱状条高度大于栈顶索引对应的柱状条高度 `heights[3] > heights[2]`，直接入栈；

`index_stack = [1,2,3]`  
- 扫描 `heights[4] = 2`，栈顶索引对应的柱状条高度大于当前柱状条 `heights[3] > heights[4]`（当前柱状条为断崖点）:  
取出栈顶索引 `3`，取出索引 `3` 后栈顶索引为 `2`,则以 `heights[3] = 6` 为高度的矩形向右可以扩展到位置 `k = 2 + 1 = 3`。  
计算面积 `area = heights[3] * (4 - 3) = 6`,`max_area = max(2, 6) = 6`。  

`index_stack = [1,2]`
- 栈顶索引对应的柱状条高度仍然大于当前柱状条 `heights[2] > heights[4]`:  
取出栈顶索引 `2`，取出索引 `2` 后栈顶索引为 `1`,则以 `heights[2] = 5` 为高度的矩形向右可以扩展到位置 `k = 1 + 1 = 2`。  
计算面积 `area = heights[1] * (4 - 2) = 10`,`max_area = max(6, 10) = 10`。  

`index_stack = [1]`
- 当前柱状条高度大于栈顶索引对应的柱状条高度 `heights[4] > heights[1]`，当前索引 `4` 入栈；  

`index_stack = [1,4]`  
- 扫描 `heights[5] = 3`，当前柱状条高度大于栈顶索引对应的柱状条高度 `heights[5] > heights[4]`，直接入栈；  

`index_stack = [1,4,5]`  
- 扫描 `heights[6] = 0`，栈顶索引对应的柱状条高度大于当前柱状条 `heights[5] > heights[6]`（当前柱状条为断崖点）:  
取出栈顶索引 `5`，取出索引 `5` 后栈顶索引为 `4`,则以 `heights[5] = 3` 为高度的矩形向右可以扩展到位置 `k = 4 + 1 = 5`。  
计算面积 `area = heights[5] * (6 - 5) = 3`,`max_area = max(10, 3) = 10`。  

`index_stack = [1,4]`
- 栈顶索引对应的柱状条高度仍然大于当前柱状条 `heights[4] > heights[6]`:  
取出栈顶索引 `4`，取出索引 `4` 后栈顶索引为 `1`,则以 `heights[4] = 2` 为高度的矩形向右可以扩展到位置 `k = 1 + 1 = 2`。  
计算面积 `area = heights[4] * (6 - 2) = 8`,`max_area = max(10, 8) = 10`。  

`index_stack = [1]`
- 栈顶索引对应的柱状条高度仍然大于当前柱状条 `heights[1] > heights[6]`:  
取出栈顶索引 `1`，取出索引 `1` 后栈为空,则以 `heights[1] = 1` 为高度的矩形向右可以扩展到位置 `k = 0`。  
计算面积 `area = heights[1] * (6 - 0) = 6`,`max_area = max(10, 6) = 10`。  
当前索引点 `6` 入栈；

`index_stack = [6]`  
所有柱状条扫描完毕，因为索引 `6` 对应的柱状条是哨兵，所以无须处理，算法结束。
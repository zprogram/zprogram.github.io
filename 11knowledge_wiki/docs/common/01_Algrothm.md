## Leetcode

### Python 实现-简单题目

#### [1、两数之和](https://leetcode-cn.com/problems/two-sum/)

给定一个整数数组 nums 和一个目标值 target，请你在该数组中找出和为目标值的那 两个 整数，并返回他们的数组下标。

你可以假设每种输入只会对应一个答案。但是，你不能重复利用这个数组中同样的元素。

```
给定 nums = [2, 7, 11, 15], target = 9

因为 nums[0] + nums[1] = 2 + 7 = 9
所以返回 [0, 1]
```

##### 题解：

1、声明字典结果集，判断num[i]中的值在不在字典里
2、如果不在字典结果集里，就把要组合的目标数字-当前值的结果和索引值存储起来，buff_dict[target - nums[i]] = i， 等价于buff_dict[目标数字-当前值] = 当前索引值
3、如果在字典里就把得到的索引值和当前索引值返回 [dict[target - nums[i]],i]

```
class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        if len(nums) <= 1:
            return False
        buff_dict = {}
        for i in range(len(nums)):
            if nums[i] in buff_dict:
                return [buff_dict[nums[i]], i]
            else:
                buff_dict[target - nums[i]] = i
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10540256.html
```

#### [7、整数反转](https://leetcode-cn.com/problems/reverse-integer) 

给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。

示例 1:

输入: 123
输出: 321
 示例 2:

输入: -123
输出: -321
示例 3:

输入: 120
输出: 21
注意:

假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−231,  231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

##### 题解：

取绝对值，取反原数字，然后根据符号值返回。但是要判断溢出问题。

逻辑：

取绝对值。每次取最后一位，然后拼接。在返回值加判断条件，如果超出范围就返回0，否则就返回原来的符号。

数学：

不借助数组和堆栈的情况下，可以借助数学解决问题。

```
class Solution:
    def reverse(self, x: int) -> int:
        """
        :type x: int
        :rtype: int
        """
        num = 0
        
        a = abs(x)
        
        while(a!=0):
            temp = a % 10
            num = num * 10 + temp
            a = int(a/10)
        
        if x > 0 and  num <= 2**31-1:
            return num
        elif x < 0 and num <= 2**31:
            return -num
        else:
            return 0
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10591372.html
```

#### [9、回文数](https://leetcode-cn.com/problems/palindrome-number)

判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

示例 1:
```
输入: 121
输出: true
```
示例 2:
```
输入: -121
输出: false
解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。
```
示例 3:
```
输入: 10
输出: false
解释: 从右向左读, 为 01 。因此它不是一个回文数。
```

##### 题解：

```
class Solution:
    def isPalindrome(self, x: int) -> bool:
        num=0
        a=abs(x)
        
        while(a!=0):
            temp = a % 10
            num =  num * 10 + temp
            a = a//10
            
        
        if x >= 0 and x == num:
            return True
        else:
            return False
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10663891.html
```

#### [13、罗马数字转整数](https://leetcode-cn.com/problems/roman-to-integer)

罗马数字包含以下七种字符: I， V， X， L，C，D 和 M。

字符          数值
I             1
V             5
X             10
L             50
C             100
D             500
M             1000
例如， 罗马数字 2 写做 II ，即为两个并列的 1。12 写做 XII ，即为 X + II 。 27 写做  XXVII, 即为 XX + V + II 。

通常情况下，罗马数字中小的数字在大的数字的右边。但也存在特例，例如 4 不写做 IIII，而是 IV。数字 1 在数字 5 的左边，所表示的数等于大数 5 减小数 1 得到的数值 4 。同样地，数字 9 表示为 IX。这个特殊的规则只适用于以下六种情况：

I 可以放在 V (5) 和 X (10) 的左边，来表示 4 和 9。
X 可以放在 L (50) 和 C (100) 的左边，来表示 40 和 90。 
C 可以放在 D (500) 和 M (1000) 的左边，来表示 400 和 900。

给定一个罗马数字，将其转换成整数。输入确保在 1 到 3999 的范围内。

示例 1:
```
输入: "III"
输出: 3
```
示例 2:
```
输入: "IV"
输出: 4
```
示例 3:
```
输入: "IX"
输出: 9
```
示例 4:
```
输入: "LVIII"
输出: 58
解释: L = 50, V= 5, III = 3.
```
示例 5:
```
输入: "MCMXCIV"
输出: 1994
解释: M = 1000, CM = 900, XC = 90, IV = 4.
```

##### 题解：

```
class Solution(object):
    def romanToInt(self, s):
        """
        :type s: str
        :rtype: int
        """
        numral_map ={ "I":1,"V":5, "X":10,  "L":50, "C":100, "D":500, "M":1000 }
        result = 0

        for i in range(len(s)):
            if i > 0 and numral_map[s[i]] > numral_map[s[i-1]]:
                result += numral_map[s[i]] - 2 * numral_map[s[i-1]]
            else:
                result += numral_map[s[i]]

        return result
```

##### 博文地址：





#### [14、最长公共前缀](https://leetcode-cn.com/problems/longest-common-prefix)

编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 ""。

示例 1:
```
输入: ["flower","flow","flight"]
输出: "fl"
```
示例 2:
```
输入: ["dog","racecar","car"]
输出: ""
解释: 输入不存在公共前缀。
```
说明:
```
所有输入只包含小写字母 a-z 。
```

##### 题解：

```
class Solution:
    def longestCommonPrefix(self, strs):
        result = ""
        i = 0
        while True:
            try:
                sets = set(string[i] for string in strs)
                if len(sets) == 1:
                    result += sets.pop()
                    i+=1
                else:
                    break
            except Exception as e:
                break

        return result
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10703837.html
```



#### [20、有效的括号](https://leetcode-cn.com/problems/valid-parentheses)

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
注意空字符串可被认为是有效字符串。

示例 1:
```
输入: "()"
输出: true
```
示例 2:
```
输入: "()[]{}"
输出: true
```
示例 3:
```
输入: "(]"
输出: false
```
示例 4:
```
输入: "([)]"
输出: false
```
示例 5:
```
输入: "{[]}"
输出: true
```

##### 题解

```
class Solution:
    def isValid(self, s: str) -> bool:
        stack = []
        lookup = {"(":")","{":"}","[":"]"}
        for parentheses in s:
            if parentheses in lookup:
                stack.append(parentheses)
            elif len(stack)== 0 or lookup[stack.pop()]!=parentheses:
                return False
        return len(stack)==0
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10747898.html
```

#### [21、合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists)

将两个有序链表合并为一个新的有序链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 

示例：

输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4

##### 题解：

```
# Definition for singly-linked list.
# class ListNode:
#     def __init__(self, x):
#         self.val = x
#         self.next = None

class Solution:
    def mergeTwoLists(self, l1: ListNode, l2: ListNode) -> ListNode:
        curr = dumy = ListNode(0)
        
        while l1 and l2:
            if l1.val < l2.val:
                curr.next = l1
                l1 = l1.next
            else:
                curr.next =l2
                l2 = l2.next
            curr = curr.next
        curr.next = l1 or l2
        
        return dumy.next
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10772079.html
```

#### [26、删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array)

给定一个排序数组，你需要在原地删除重复出现的元素，使得每个元素只出现一次，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

示例 1:
```
给定数组 nums = [1,1,2], 

函数应该返回新的长度 2, 并且原数组 nums 的前两个元素被修改为 1, 2。 

你不需要考虑数组中超出新长度后面的元素。
```
示例 2:
```
给定 nums = [0,0,1,1,1,2,2,3,3,4],

函数应该返回新的长度 5, 并且原数组 nums 的前五个元素被修改为 0, 1, 2, 3, 4。

你不需要考虑数组中超出新长度后面的元素。
```
说明:

为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以“引用”方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:
```
// nums 是以“引用”方式传递的。也就是说，不对实参做任何拷贝
int len = removeDuplicates(nums);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中该长度范围内的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}

```

##### 题解：

```
class Solution:
    def removeDuplicates(self, nums: List[int]) -> int:
        if not nums:
            return 0
        
        count = 0
        for i in range(len(nums)):
            if nums[count]!=nums[i]:
                count += 1
                nums[count] = nums[i]
            
        return count+1
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10814731.html
```





#### [27、移除元素](https://leetcode-cn.com/problems/remove-element)

给定一个数组 nums 和一个值 val，你需要原地移除所有数值等于 val 的元素，返回移除后数组的新长度。

不要使用额外的数组空间，你必须在原地修改输入数组并在使用 O(1) 额外空间的条件下完成。

元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素。

示例 1:
```
给定 nums = [3,2,2,3], val = 3,

函数应该返回新的长度 2, 并且 nums 中的前两个元素均为 2。

你不需要考虑数组中超出新长度后面的元素。
```
示例 2:
```
给定 nums = [0,1,2,2,3,0,4,2], val = 2,

函数应该返回新的长度 5, 并且 nums 中的前五个元素为 0, 1, 3, 0, 4。

注意这五个元素可为任意顺序。

你不需要考虑数组中超出新长度后面的元素。
说明:
```
为什么返回数值是整数，但输出的答案是数组呢?

请注意，输入数组是以“引用”方式传递的，这意味着在函数里修改输入数组对于调用者是可见的。

你可以想象内部操作如下:
```
// nums 是以“引用”方式传递的。也就是说，不对实参作任何拷贝
int len = removeElement(nums, val);

// 在函数里修改输入数组对于调用者是可见的。
// 根据你的函数返回的长度, 它会打印出数组中该长度范围内的所有元素。
for (int i = 0; i < len; i++) {
    print(nums[i]);
}
```

##### 题解：

```
class Solution:
    def removeElement(self, nums: List[int], val: int) -> int:
        i,last = 0,len(nums)-1
        
        while i <= last:
            if nums[i] == val:
                nums[i],nums[last] = nums[last],nums[i]
                last -= 1
            else:
                i += 1
                
        return last+1
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10814744.html
```

#### [28、实现 strStr()](https://leetcode-cn.com/problems/implement-strstr) 

给定一个 haystack 字符串和一个 needle 字符串，在 haystack 字符串中找出 needle 字符串出现的第一个位置 (从0开始)。如果不存在，则返回  -1。

示例 1:
```
输入: haystack = "hello", needle = "ll"
输出: 2
```
示例 2:
```
输入: haystack = "aaaaa", needle = "bba"
输出: -1
```
说明:

当 needle 是空字符串时，我们应当返回什么值呢？这是一个在面试中很好的问题。

对于本题而言，当 needle 是空字符串时我们应当返回 0 。这与C语言的 strstr() 以及 Java的 indexOf() 定义相符。

##### 题解：

1、获取指定要查询字符串的长度
2、切片从0开始，查询字符串的长度之间的字符
3、如果相等就返回，不相等就返回-1

```
class Solution(object):
    def strStr(self, haystack, needle):
        """
        :type haystack: str
        :type needle: str
        :rtype: int
        """
        for i in range(len(haystack) - len(needle)+1):
            if haystack[i:i+len(needle)]==needle:
                return i
        return -1
```

##### 博文地址：

```
https://www.cnblogs.com/17bdw/p/10832101.html
```





#### [35、搜索插入位置](https://leetcode-cn.com/problems/search-insert-position)

给定一个排序数组和一个目标值，在数组中找到目标值，并返回其索引。如果目标值不存在于数组中，返回它将会被按顺序插入的位置。

你可以假设数组中无重复元素。

示例 1:
```
输入: [1,3,5,6], 5
输出: 2
```
示例 2:
```
输入: [1,3,5,6], 2
输出: 1
```
示例 3:
```
输入: [1,3,5,6], 7
输出: 4
```
示例 4:
```
输入: [1,3,5,6], 0
输出: 0
```

##### 题解

1、比较目标数字是否比列表最后一位数大，如果是就返回。因为如果值没有是要插入到列表的。

2、循环比较目标数字的关系

```
if target > nums[len(nums)-1]:
    return len(nums)
for i in range(len(nums)):
    if nums[i] >= target:
        return i
```

##### 博文地址

https://www.cnblogs.com/17bdw/p/10901656.html



#### [38、报数](https://leetcode-cn.com/problems/count-and-say/)

报数序列是一个整数序列，按照其中的整数的顺序进行报数，得到下一个数。其前五项如下：

```
1.     1
2.     11
3.     21
4.     1211
5.     111221
```

1 被读作  "one 1"  ("一个一") , 即 11。
11 被读作 "two 1s" ("两个一"）, 即 21。
21 被读作 "one 2",  "one 1" （"一个二" ,  "一个一") , 即 1211。

给定一个正整数 n（1 ≤ n ≤ 30），输出报数序列的第 n 项。

注意：整数顺序将表示为一个字符串。

示例 1:

```
输入: 1
输出: "1"
```
示例 2:
```
输入: 4
输出: "1211"
```

为了便于理解，选择每次都执行输入值的方法

然后逐行对比，每个字符做一次计数。返回当前行数所有数字计算后的统计值。

##### 题解

```
class Solution(object):
    def countAndSay(self, n):
        """
        :type n: int
        :rtype: str
        """
        seq = "1"
        for i in range(n-1):
            seq = self.getNext(seq)
        return seq
    
    def getNext(self,seq):
        i,next_seq = 0,""
        while i < len(seq):
            count = 1
            while i < len(seq) - 1 and seq[i] == seq[i+1]:
                count +=1
                i += 1
            next_seq += str(count) + seq[i]
            i += 1
        return next_seq
```

##### 博文地址

https://www.cnblogs.com/17bdw/p/10965804.html

#### [53、最大子序和](https://leetcode-cn.com/problems/maximum-subarray/)


给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:

```
输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。
```


进阶:

如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。


##### 题解

```
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        if max(nums) < 0 :
            return max(nums)
        
        local_max,global_max = 0,0
        
        for num in nums:
            local_max = max(0,local_max+num)
            global_max = max(local_max,global_max)
            
        return global_max
```


##### 博文地址
https://www.cnblogs.com/17bdw/p/11009250.html



#### [58. 最后一个单词的长度](https://leetcode-cn.com/problems/length-of-last-word/)

给定一个仅包含大小写字母和空格 ' ' 的字符串，返回其最后一个单词的长度。

如果不存在最后一个单词，请返回 0 。

说明：一个单词是指由字母组成，但不包含任何空格的字符串。

示例:

```
输入: "Hello World"
输出: 5
```

##### 题解：

1、遍历字符串

2、当遇到空格时，就把计数置零

3、否则计数器加1

```
class Solution(object):
    def lengthOfLastWord(self, s):
        """
        :type s: str
        :rtype: int
        """
        count = 0
        local_count = 0
        
        for i in range(len(s)):
            if s[i] == ' ':
                local_count = 0
            else :
                local_count += 1
                count = local_count
        return count
```



##### 博文地址：

https://www.cnblogs.com/17bdw/p/11037760.html

#### [66. 加一](https://leetcode-cn.com/problems/plus-one/)

给定一个由整数组成的非空数组所表示的非负整数，在该数的基础上加一。

最高位数字存放在数组的首位， 数组中每个元素只存储单个数字。

你可以假设除了整数 0 之外，这个整数不会以零开头。

示例 1:

```
输入: [1,2,3]
输出: [1,2,4]
解释: 输入数组表示数字 123。
```

示例 2:

```
输入: [4,3,2,1]
输出: [4,3,2,2]
解释: 输入数组表示数字 4321。
```

##### 题解

```
class Solution(object):
    def plusOne(self, digits):
        """
        :type digits: List[int]
        :rtype: List[int]
        """
        for i in reversed(range(len(digits))):
            if digits[i] == 9:
                digits[i] = 0
            else:
                digits[i] += 1
                return digits
        digits[0] = 1
        digits.append(0)
        return digits

if __name__ == '__main__':
    c = Solution()
    print(c.plusOne([1,2,3,4]))
    print(c.plusOne([1,0,9]))
```

##### 博文地址
https://www.cnblogs.com/17bdw/p/11067058.html



#### [67. 二进制求和](https://leetcode-cn.com/problems/add-binary/)



给定两个二进制字符串，返回他们的和（用二进制表示）。

输入为**非空**字符串且只包含数字 `1` 和 `0`。

**示例 1:**

```
输入: a = "11", b = "1"
输出: "100"
```

**示例 2:**

```
输入: a = "1010", b = "1011"
输出: "10101"
```

##### 题解

```
result , carry,val = "",0,0
for i in range(max(len(a),len(b))):
    val = carry
    if i < len(a):
        val += int(a[-(i+1)])
     


```

##### 博文地址

https://www.cnblogs.com/17bdw/p/11111549.html




### Hash Table

#### Jewels and Stones

https://leetcode.com/problems/jewels-and-stones/

```
题解：https://www.cnblogs.com/17bdw/p/9977996.html#_label0_0
```


#### Single Number

https://leetcode.com/problems/single-number/

```
题解：https://www.cnblogs.com/17bdw/p/10092655.html#_label1_0
```

### string

#### Unique Email Addresses

https://leetcode.com/problems/unique-email-addresses/

```
题解：https://www.cnblogs.com/17bdw/p/10018357.html#_label0_0
```

#### To Lower Case

https://leetcode.com/problems/to-lower-case/

```
题解：https://www.cnblogs.com/17bdw/p/10055054.html#_label1_0
```


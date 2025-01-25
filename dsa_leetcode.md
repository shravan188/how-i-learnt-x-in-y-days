## DSA Learnings from Leetcode and Striver A2Z Sheet

## Day 1
* Logic is
    * Sorted array : Will have one instance where a number is lesser than preceding/previous number (including the comparison of last element with first element)
    * Unsorted array :  Will have more than one instance where number is lesser than preceding number
    (including the comparison of last element with first element)

```
### Check if array is sorted and rotated
# https://leetcode.com/problems/check-if-array-is-sorted-and-rotated/
# Approach 1
class Solution(object):
    def check(self, nums):
        """
        :type nums: List[int]
        :rtype: bool
        """
        c = 0
        l = len(nums)
        for i in range(l-1):
            if nums[i] > nums[i+1]:
                c += 1
        if nums[l-1] > nums[0]:
            c += 1
        if c > 1:
            return False
        else:
            return True

# Approach 2 (try doing in a single line using negative index)

```

## Day 2
### Duration : 1 hour

### Learnings

* Solved Move Zeros to End problem using 2 Pointer approach.

* Solved Find missing number in an array problem using hash map

```
### Move zeros to end
# https://leetcode.com/problems/move-zeroes/
class Solution(object):
    def moveZeroes(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        if len(nums) == 1:
            return nums
        n = len(nums)

        p1 = 0
        p2 = 1
        while p2 < n:
            if nums[p1] == 0 and nums[p2] != 0:
                temp = nums[p1]
                nums[p1] = nums[p2]
                nums[p2] = temp
                p1 += 1
                p2 += 1
            elif nums[p1] == 0 and nums[p2] == 0:
                p2 += 1
            elif nums[p1] != 0 and nums[p2] == 0:
                p1 += 1
            elif nums[p1] != 0 and nums[p2] != 0:
                p1 += 1
                p2 += 1

        return nums
# Examples : [1,2,3,0,4], [0,1]


### Find missing number in an array with consequtive numbers except one
# https://leetcode.com/problems/missing-number/
class Solution(object):
    def missingNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        n = len(nums)
        nums_set = set(range(n+1))
        for num in nums:
            nums_set.remove(num)

        return nums_set.pop()

```

### References

1. https://leetcode.com/problems/missing-number/solutions/4754285/faster-lesser-4-methods-sorting-mathematical-formula-bit-manipulation-hash-table/




## Day 3
### Duration : 40 mins

### Learnings
* Solved Single Occurence Number problem using 3 approaches
    * Brute force (O(N^2) time complexity)
    * Sorting + 2 pointer (O(NlogN) + O(N) time complexity)
    * Dictionary/Hashmap (O(N) time + O(N) space complexity)

* The most efficient approach is using XOR operator, based on following logic
```
1 xor 2 xor 3 xor 1 xor 2 xor 3 xor 4 
= (1 xor 1) xor (2 xor 2) xor (3 xor 3) xor 4 // commutative & associative property
= 0 xor 0 xor 0 xor 4
= 0 xor 4
= 4
```
Note that although we are showing decimal numbers xor happens at bit level i.e. 1 xor 2 is b'01 xor b'10 which is b'11 = 3. 3 xor 1 = b'11 xor b'01 = b'10 = 2
Thus xor between all numbers can help identify the only number with single occurence in O(N) time complexity with O(1) space complexity

* XOR is a dissmilarity indicator i.e. it gives 1 if both input bits are different and 0 if both input bits are same. It can also be thought of an addition between input bit without considering the carry (hence used in circuits to add bits)


```
### Find the number that appears once, and other numbers twice.
### https://leetcode.com/problems/single-number/
# Approach 1 : Using dictionary/hashmap to store number count
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        num_count = {}
        for num in nums:
            if num in num_count:
                num_count[num] += 1
            else:
                num_count[num] = 1
        
        singleNum = [k for k,v in num_count.items() if v==1][0]

        return singleNum

# Approach 2: Sort array, then use 2 pointer approach
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        nums.sort()
        p1 = 0
        p2 = 1
        length = len(nums)

        while p1 < length:
            if p2 == length:
                return nums[p1]
            if nums[p1] == nums[p2]:
                p1 += 2
                p2 += 2
            else:
                return nums[p1]

# Approach 3 (Brute Force)
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        length = len(nums)
        for i in range(length):
            isSingle = True
            for j in range(length):
                if nums[i] == nums[j] and i != j:
                    isSingle = False
                    break
            if isSingle:
                return nums[i] 

# Approach 4 : Using xor operator
class Solution(object):
    def singleNumber(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        xor = 0
        for num in nums:
            xor = xor ^ num

        return xor


```
### Doubts
1. Can we find key corresponding to a given value in O(1) time in Python dictionary? Because `[k for k,v in num_count.items() if v==1][0]` is O(N) time complexity.


## Day 4
### Duration : 30 mins
### Learnings

* 2 Sum problem : Check if a pair of numbers exist in an array such that their sum equals to a target number

* Solved 2 Sum problem using hash table. 
```
### 2 Sum problem
### https://leetcode.com/problems/two-sum/

# Approach 1 : Using hash table (not optimal)

class Solution(object):
    def twoSum(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """
        num_position = {i:num for i,num in enumerate(nums)}
        for i, num in enumerate(nums):
            diff = target - num
            if diff in num_position.values():
                index = [k for k,v in num_position.items() if v==diff and k != i]
                if len(index) > 0:
                    return [i, index[0]]
```


## Day 5

* Problem : Sort an array containing 0's, 1's and 2's

* Logic I used is to search for all zeros, for each zero, swap them with number at beginning using a pointer, then move pointer to next position. Do until all zeros at beginning. Repeat for 1 and 2
```
# Approach 1 : Brute Force : 
class Solution(object):
    def sortColors(self, nums):
        p1 = 0
        length = len(nums)
        if length == 1:
            return

        for i in range(length):
            if nums[i] == 0:
                temp = nums[p1]
                nums[p1] = nums[i]
                nums[i] = temp
                p1 += 1

        for i in range(p1, length):
            if nums[i] == 1:
                temp = nums[p1]
                nums[p1] = nums[i]
                nums[i] = temp
                p1 += 1
        
        for i in range(p1, length):
            if nums[i] == 2:
                temp = nums[p1]
                nums[p1] = nums[i]
                nums[i] = temp
                p1 += 1
        
        return


class Solution(object):
    def sortColors(self, nums):
        """
        :type nums: List[int]
        :rtype: None Do not return anything, modify nums in-place instead.
        """
        p1 = 0
        length = len(nums)
        if length == 1:
            return

        for color in [0, 1, 2]:
            for i in range(p1, length):
                if nums[i] == color:
                    temp = nums[p1]
                    nums[p1] = nums[i]
                    nums[i] = temp
                    p1 += 1

               
        return

## TO DO : Solve problem using Dutch National Flag algorithm
```

* Dutch National Flag algorithm : Have 3 pointers low, mid, high. low and mid start from beginning, high from end. End goal is to arrange in such a way that arr[0:low] has 0, arr[low:mid] has 1 and arr[high+1:n] has 2s, and iterate until mid and high pointers meet


### Doubts
1. How to extend Dutch National Flag algorithm to 4 numbers and beyond?

## Day 6

* Learnt about Moore's voting algorithm

* Moore's Voting algorithm: 
```
## Approach 1
class Solution(object):
    def majorityElement(self, nums):
        
        length = len(nums)
        num_count = {}
        for num in nums:
            if num in num_count:
                num_count[num] += 1
            else:
                num_count[num] = 1

        for num, count in num_count.items():
            if count > length / 2:
                return num


## Approach 2 : Using Moore's voting alogorithm
class Solution(object):
    def majorityElement(self, nums):
        el = None
        count = 0
        for num in nums:
            if count == 0:
                el = num
                #count += 1

            if el == num:
                count += 1
            else:
                count -= 1
        
        return el
            


```

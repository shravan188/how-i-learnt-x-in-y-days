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
        # [1,2,3,0,4], [0,1]
        while p2 < n:
            if nums[p1] == 0 and nums[p2] != 0:
                temp = nums[p1]


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

1. https://leetcode.com/problems/missing-number/solutions/4754285/faster-lesser-4-methods-sorting-mathematical-formula-bit-manipulation-hash-table/


## Day 3

```
### Find maximum consecutive ones in an array
# https://leetcode.com/problems/max-consecutive-ones/
class Solution(object):
    def findMaxConsecutiveOnes(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        count = 0
        max_count = 0
        for num in nums:
            if num == 1:
                count = count + 1
            else:
                if count > max_count:
                    max_count = count
                count = 0
        
        # in the edge case scenario where the longest sequence includes the last number
        if count > max_count:
            max_count = count
        
        return max_count

class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        length = len(nums)
        count = 0
        for i in range(length):
            if nums[i] == k:
                count += 1
            elif nums[i] > k:
                continue
            else:
                sum_ = nums[i]
                j = i + 1
                while sum_ < k and j < length:
                    sum_ += nums[j]
                    j = j + 1
                if sum_ == k:
                    count += 1

        return count


class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        length = len(nums)
        count = 0
        for i in range(length):
            if nums[i] == k:
                count += 1
            sum_ = nums[i]
            j = i + 1
            o = False
            while sum_ != k and j < length:
                sum_ += nums[j]
                j = j + 1
                o = True


            if sum_ == k and o:
                count += 1

        return count

class Solution(object):
    def subarraySum(self, nums, k):
        """
        :type nums: List[int]
        :type k: int
        :rtype: int
        """
        length = len(nums)
        count = 0
        for i in range(length):
            if nums[i] == k:
                count += 1
            sum_ = nums[i]
            j = i + 1
            while j < length:
                sum_ += nums[j]
                if sum_ == k:
                    count += 1
                j = j + 1


            

        return count





https://leetcode.com/problems/subarray-sum-equals-k/solutions/5334031/python-100-beat-100-efficient-optimal-solution-easy-to-understand/
[1,-1,0]

```
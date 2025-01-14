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


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

## Day 7
### Duration : 45 mins


* Solved 1 problem
    * Maximum subarray problem 

* Kadane's algorithm :  The intuition behind Kadane's algorithm is that:
A subarray with a sum less than 0 will always reduce our answer and so this type of subarray cannot be a part of the subarray with maximum sum. Hence whenever we encounter a subarray with sum less than zero, we just restart the count. If the sum upto a given point is negative, we start from 0 as it will not contribute positively going forward, but if it is greater than zero, we keep it as it will contribute positively going forward. 

* Important thing to note is we do not make current sum zero the moment we encounter a negative number, because as long as the cumulative sum upto that point is positive, it will contribute positively to the next number. Only when the cumulative sum is negative, we make the current sum zero as it will not contribute positively to the next number (e.g. [1,4,2,-2,4] although -2 is negative, the cumulative sum until that point is 5, so it will contribute positively to next number i.e. 4)
```
# Find subarray with maximum sum in a given array
# https://leetcode.com/problems/maximum-subarray/
## Approach 1 : Timeout for a few testcases (as complexity of O(N^2))
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        max_sum = -100000
        length = len(nums)
        
        for i in range(length):
            curr_sum = nums[i]
            if curr_sum > max_sum:
                max_sum = curr_sum
            for j in range(i+1, length):
                curr_sum += nums[j]
                if curr_sum > max_sum:
                    max_sum = curr_sum

        return max_sum


# Approach 2: Using Kadane's algorithm 
class Solution(object):
    def maxSubArray(self, nums):
        curr_sum = 0
        max_sum = -sys.maxsize

        length = len(nums)

        for i in range(0, length):
            curr_sum += nums[i]
            if curr_sum > max_sum:
                max_sum = curr_sum

            if curr_sum < 0:
                curr_sum = 0

        return max_sum
```

* Be careful of edge cases, always take a few examples to validate your code. 

```
# Will not work for [-2,1], will give max sum as -1 instead of 1
# Because it adds second number before checking if first number is -ve or not
class Solution(object):
    def maxSubArray(self, nums):
        """
        :type nums: List[int]
        :rtype: int
        """
        curr_sum = nums[0]
        max_sum = nums[0]
        
        length = len(nums)

        for i in range(1, length):
            curr_sum += nums[i]
            if curr_sum > max_sum:
                max_sum = curr_sum

            if curr_sum < 0:
                curr_sum = 0

        return max_sum

```

### Doubts
1. Why do we not make sum zero the moment we encounter a negative number, and make it zero only if the cumulative sum upto that point is negative?

## Day 8
### Duration : 15 mins
### Learnings
* Solved 1 problem:
    * Best time to Buy and Sell stock

* Min upto that point logic : In this problem we only want minumum value upto that point/index and not the global minima, hence while linearly traverling, by just using 1 variable, we can store the minimum value until that index. 

* Min value till that point vs global minimum logic : Identify whether problem needs global minimum or minimum until that point. If only until that point, then we can use the above logic and optimize the code

```
class Solution(object):
    def maxProfit(self, prices):
        profit = 0
        min_price = prices[0]
        max_profit = 0
        for price in prices: 
            # get min price upto that point/index      
            if price < min_price :
                min_price = price
            
            profit = price - min_price
            
            if profit > max_profit:
                max_profit = profit

        return max_profit

```

IndexError: list index out of range
    if nums[p2] < 0 and new_nums[-1] > 0:

### Doubts
1. How is the Stock Buy and Sell problem related to Kadane's algorithm? 


## Day 9
### Duration : 15 mins

### Learnings
* Solved 1 problem:
    * Rearrange +ve and -ve numbers alternatively


* Missed that could use 2 pointer on result array instead of original array            


```
# [39,-8,-28,46,-16,-21,-12,49,14,-46,22,9]
## Approach 1 : Using 2 pointer approach on source array

class Solution(object):
    def rearrangeArray(self, nums):
        """
        :type nums: List[int]
        :rtype: List[int]
        """
        new_nums = []
        p1 = p2 = 0
        length = len(nums)
        while p1 < length or p2 < length:
            if p1 < length:
                if nums[p1] > 0 and (len(new_nums) == 0 or new_nums[-1] < 0):
                    new_nums.append(nums[p1])
                    p1 += 1
                elif nums[p1] < 0:
                    p1 +=1

            if len(new_nums) == 0:
                continue

            if p2 < length:
                if nums[p2] < 0 and new_nums[-1] > 0:
                    new_nums.append(nums[p2])
                    p2 += 1
                elif nums[p2] > 0:
                    p2 += 1

        return new_nums
        



## Approach 2 : Optimized Solution, using 2 pointer on destination array
class Solution(object):
    def rearrangeArray(self, nums):
        length = len(nums)
        newNums = [0] * length
        p1 = 0
        p2 = 1
        for num in nums:
            if num > 0:
                newNums[p1] = num
                p1 += 2

            if num < 0:
                newNums[p2] = num
                p2 += 2
        
        return newNums


```

## Day 10
### Duration : 1.25 hours

### Learnings

* Solved problems:
    * Find the next permutation for given array of numbers (eg. [1,3,2] -> [2,1,3])
    * Simple binary search on sorted array
    * Remove outermost parentheses from a string

* Key point : The largest number which can be formed using a given set of digits is by placing them in descending order

* Using the point above, we try to locate the break point i.e. first index i from the back of the given array where arr[i] becomes smaller than arr[i+1]. Because we know that number formed from breakpoint onwards is not the largest number that can be formed using given set of digits.

* Once we do that, we try to find smallest number which is larger than the break point number, and put that in place of break point, and sort all the remaining numbers post the breakpoint

* Missed 2 edge case scenarios initially:
    1. When we have reached the largest possible combination for entire array (eg. [6,4,2]) (solved by adding i=-1 condition)
    2. When we have two same digits side by side (had to change while condition from nums[i] > nums[i+1] to nums[i] >= nums[i+1])


* Idea:  If we can track how many open and close parentheses we've seen using a counter that counts opening parenthesis, we can figure out which parentheses are the outermost ones and skip them. For outermost parentheses counter will be 1 if opening parenthesis and counter will be 0 if closing parenthesis

```
# https://leetcode.com/problems/next-permutation/
# passed 231/266 test cases, failed for [5,1,1]
class Solution(object):
    def nextPermutation(self, nums):
        length = len(nums)
        i = length - 2
        while nums[i] > nums[i+1]:
            i = i - 1
        
        nums[i+1:] = sorted(nums[i+1:])

        for j in range(i+1, length):
            if nums[j] > nums[i]:
                temp = nums[i]
                nums[i] = nums[j]
                nums[j] = temp
                break
        


# Approach 1
class Solution(object):
    def nextPermutation(self, nums):
        
        length = len(nums)
        i = length - 2
        while nums[i] >= nums[i+1] and i >= 0:
            i = i - 1      
        
        nums[i+1:] = sorted(nums[i+1:])

        if i == -1:
            return

        for j in range(i+1, length):
            if nums[j] > nums[i]:
                temp = nums[i]
                nums[i] = nums[j]
                nums[j] = temp
                break
        

# https://leetcode.com/problems/binary-search/
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        left = 0
        right = len(nums) - 1

        while left <= right:
            mid = (right + left) // 2
            if nums[mid] > target:
                right = mid - 1
            elif nums[mid] < target:
                left = mid + 1                
            elif nums[mid] == target:
                return mid

        return -1


# https://leetcode.com/problems/remove-outermost-parentheses/
class Solution(object):
    def removeOuterParentheses(self, s):
        count = 0
        result = ""
        for c in s:
            if c == "(":
                count = count + 1

            
            if c == ")":
                count = count - 1

            if c == "(" and count > 1:
                result = result + str(c)
            
            if c == ")" and count > 0:
                result = result + str(c)

           
        
        return result
        
```
### Doubts
1. A small question: When you search the next greater element in the right half, which one is better: Binary Search or simple iteration?
2. How do we generate all possible permutations of a given set of numbers?
3. In binary search is it (right + left)/2 or (right - left)/2? 

### References
1. https://stackoverflow.com/questions/2272819/sort-a-part-of-a-list-in-place
2. https://stackoverflow.com/questions/4435169/how-do-i-append-one-string-to-another-in-python
3. https://stackoverflow.com/questions/538346/iterating-each-character-in-a-string-using-python

## Day 11

### Duration : 1.5 hours

### Learnings
* Solved problems:
    * Search insert position for new element in sorted array
    * Find first and last position of element in sorted array
    * Binary search in rotated sorted array


* Insertion problem is literally binary search with 2 additional conditions at the end if target not present in the array


* In the second problem, missed edge case scenario where target is present until end like [0,0,1,1] and target=1. Also wrongly added edge condition to while loop expression instead of adding it within if statement (look below)

```
## wrong approach, will still give index out of range error
while nums[right] == target and right == length:
    right += 1
  
right -= 1

## right approach
while nums[right] == target:
    right += 1
    if right == length:
        break

right -= 1

```

* Key idea (binary search sorted) : Key idea is to compare the middle element with only the left element (not both left and right), to see which half of the array is sorted. Then we check if the number is between the two extremes of the sorted half (we can do that because we know that that half is sorted, we cannot do it for the unsorted half). Based on whether the number is in between the 2 extremes, we go ahead and shift either pointer accordingly. Note that left and right pointers change after each iteration

* If an array is sorted then just looking at the first and last number gives us a good idea of the entire array

*  The key point I missed is we have to look for the sorted half, and then search for the element in the sorted half using the 2 end values. Also I forgot that left and right pointer keep moving and thought they get stuck at the ends of the array

```
# https://leetcode.com/problems/search-insert-position/
class Solution(object):
    def searchInsert(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        l = 0
        r = len(nums) - 1

        while l <= r:
            mid = (l + r) // 2
            if nums[mid]  == target:
                return mid
            elif nums[mid] > target:
                r = mid - 1
            elif nums[mid] < target:
                l = mid + 1

        if nums[mid] < target:
            return mid + 1
        else:
            return mid

# https://leetcode.com/problems/find-first-and-last-position-of-element-in-sorted-array/
class Solution(object):
    def searchRange(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: List[int]
        """

        l = 0
        r = len(nums) - 1
        length = len(nums)

        if len(nums) == 0:
            return [-1, -1]

        while l <= r:
            mid = (l + r) // 2
            if nums[mid]  == target:
                break
            elif nums[mid] > target:
                r = mid - 1
            elif nums[mid] < target:
                l = mid + 1

        if l <= r:
            left = mid
            right = mid
            while nums[right] == target:
                right += 1
                if right == length:
                    break
            right -= 1
            while nums[left] == target:
                left -= 1
                if left == -1:
                    break
            left += 1

            return [left, right]
        
        else:
            return [-1, -1]

### https://leetcode.com/problems/search-in-rotated-sorted-array/
# wrong code
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        left = 0
        right = len(nums) - 1

        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                return mid
            if nums[mid] != target:
                if target == nums[left]:
                    return left
                if target == nums[right]:
                    return right
                if target > nums[left]:
                    right = mid - 1
                if target < nums[right]:
                    left = mid + 1
        
        return -1

# wrong code
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        length = len(nums)
        left = 0
        right = len(nums) - 1

        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                return mid
            if nums[mid] > nums[left] and nums[mid] > target:
                right = mid - 1
            if nums[mid] > nums[left] and nums[mid] < target:
                left =  mid + 1
            if nums[mid] < nums[right] and nums[mid] < target:
                left = mid + 1
            if nums[mid] < nums[right] and nums[mid] > target:
                right = mid - 1

# Correct approach : Find element in rotated sorted array
class Solution(object):
    def search(self, nums, target):
        """
        :type nums: List[int]
        :type target: int
        :rtype: int
        """
        length = len(nums)
        left = 0
        right = len(nums) - 1

        while left <= right:
            mid = (left + right) // 2
            if nums[mid] == target:
                return mid
            
            if nums[left] <= nums[mid]:
                if nums[left] <= target < nums[mid]:
                    right = mid - 1
                else:
                    left = mid + 1
            else:
                if nums[mid] < target <= nums[right]:
                    left = mid + 1
                else:
                    right = mid - 1
                    

    
        return -1
```
### Doubts
1. In rotated sorted array, should we compare array[mid] with border values or target with border values?
2. Is the last number always less than first number in a roated sorted array?



## Day N (To be cleaned up later)

### Learnings

### Find maximum consecutive ones in an array
```
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

# Approach 1
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

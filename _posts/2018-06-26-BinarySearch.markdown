---
layout:     post
title:      "BinarySearch"
subtitle:   "LeetCode Review 3"
date:       2018-06-26 12:00:00
author:     "Farrell"
header-img: "img/bg3.jpg"
catalog: true
tags:
    - Leetcode
---

> \# 436 also has a faster DP solution; do \# 81 again

## 1. BinarySearch
Use time cost O(logn) to get a position in a list that is sorted, or sequenced according to a certain pattern (applies for local pattern, see \# 162). 
Most "BinarySearch tagged" leetcode problems could be solved with other tools more efficiently such as Sorting, HashMap and TwoPointers.

---
## 2. Methods
### <span id="StandardBinarySearch">2.1 Standard Binary Search</span>
Recursive:
```java
public int BS(int[] A, int target) {
    return helper(A, 0, A.length, target);
}

public int helper(int[] A, int l, int r, int target) {
    if (l == r) {
    	if (A[l] == target) return l;
    	else return -1;
    }
    int mid = l + (r-l)/2;
    if (A[mid] == target) return mid;
    else if (A[mid] <= target) return helper(A, mid+1, r);
    else return helper(A, l ,mid);
}
```
Non-Recursive:
```java
public int BS(int[] A, int target){
	int l = 0, r = A.length, mid = 0;
	while (true) {
		if (l == r) {
			if (A[l] == target) return l;
			else return -1;
		}
		mid = l + (r-l)/2;
		if (A[mid] == target) return mid;
		else if (A[mid] <= target) l = mid+1;
		else r = mid;
	}
}
```

### <span id="BinarySearchInsertion">2.2 Binary Search Insertion</span>
```java
public ArrayList<Integer> BSI(ArrayList<Integer> A, target) {
	int ind = helper(A, 0, A.size(), target);
	A.add(ind, target);
	return A;
}

public int helper(ArrayList A, int l, int r, int target) {
    if (l == r) {
    	if (A[l] > target) return l;
    	else return l+1;
    }
    int mid = l + (r-l)/2;
    if (A[mid] == target) return mid;
    else if (A[mid] <= target) return helper(A, mid+1, r);
    else return helper(A, l ,mid);
}
```

### <span id="ValueBinarySearch">2.3 Value Binary Search</span>
```java
public int kthSmallest(int[][] matrix, int k) {
    int n = matrix.length;
    int low = matrix[0][0], high = matrix[n-1][n-1];
    while(low < high) {
        int mid = low + (high-low)/2;
        int cnt = getLessEqual(matrix, mid);
        if (cnt < k) low = mid+1;
        else high = mid;
    }
    return low;
}

private int getLessEqual(int[][] matrix, int val) {
    int res = 0;
    int n = matrix.length, i = n - 1, j = 0;
    while (i >= 0 && j < n) {
        if (matrix[i][j] > val) i--;
        else {
            res += i + 1;
            j++;
        }
    }
    return res;
}
```

<span id="MatrixValueRank">getLess Equal</span> can find target value rank in sorted matrix with O(m+n) timecost.

### <span id="RotatedArray"> 2.4 Rotated Array</span>
Compare middle with tail to see which seg mid is on.
If containing duplicates, set mid to dup left.
```java
public int findMin(int[] nums) {
    int l=0, r=nums.length-1;
    while (true) {
        if (l == r) return nums[l];
        int mid = l + (r-l)/2;
        if (nums[mid] > nums[r]) l = mid+1;		// compare mid to tail
        else r = mid;
    }
}
```

### <span id="LocalPattern"> 2.5 Local Pattern</span>
Binary Search doesn't requires the input array to be wholy sorted. It could be applied to array with local patterns.
See \# 162, the goal is to search for a local peak, thus when shortening candidates, the only requirement is to find a increasing direction.

---
## 3. Solutions

### 33 Search in Rotated Sorted Array

Param|Value
:---:|:---
Input|rotated array with no duplicates; target
Output|target position in rotated array
Solution|same as \# 153
TimeCost|O(logn)
Percent|83%
CornorCase|None
Tricks|[LocalPattern](#LocalPattern); [RotatedArray](#RotatedArray)

### 34 Search for a Range

Param|Value
:---:|:---
Input|array with duplicates, target
Output|start and end index of target in array
Solution|standard bisearch to find start, iterate over
TimeCost|O(logn+n)
Percent|75
CornorCase|None
Tricks|[StandardBinarySearch](#StandardBinarySearch);

### 35 Search Insert Position

Param|Value
:---:|:---
Input|array, elem to insert
Output|insertion position
Solution|binary search insertion
TimeCost|O(logn)
Percent|100
CornorCase|None
Tricks|[BinarySearchInsertion](#BinarySearchInsertion)

### 50 Pow(x, n)

Param|Value
:---:|:---
Input|base x and exponential n
Output|pow(x,n)
Solution|recursive, each time do x*x
TimeCost|O(logn)
Percent|85
CornorCase|n==0 || n==1; input
Tricks|None

### 69 Sqrt(x)

Param|Value
:---:|:---
Input|num x
Output|sqrt(x)
Solution|bisearching x root
TimeCost|O(logx)
Percent|48
CornorCase|None
Tricks|[StandardBinarySearch](#StandardBinarySearch); standard bisearch solution is behind the optimized version (on bisearch decision) by 50% for one more multiplication.

### 74 Search a 2D Matrix

Param|Value
:---:|:---
Input|sorted matrix, start of each line bigger than end of previous line; target number
Output|whether target number is in sorted matrix
Solution|bs along rows, bs along cols
TimeCost|O(logn)
Percent|95
CornorCase|first bs, find the first line head smaller than target
Tricks|[StandardBinarySearch](#StandardBinarySearch)

### 81 Search in Rotated Sorted Array II

Param|Value
:---:|:---
Input|rotated array with no duplicates; target
Output|target position in rotated array
Solution|same as \# 154
TimeCost|O(logn)
Percent|83%
CornorCase|None
Tricks|[LocalPattern](#LocalPattern); [RotatedArray](#RotatedArray)

### 153 Find Minimum in Rotated Sorted Array

Param|Value
:---:|:---
Input|rotated array without duplicates
Output|minimum number
Solution|modified binary search: compare nums[mid] with nums[r]
TimeCost|O(logn)
Percent|83
CornorCase|None
Tricks|[LocalPattern](#LocalPattern); [RotatedArray](#RotatedArray)

### 154 Find Minimum in Rotated Sorted Array II

Param|Value
:---:|:---
Input|rotated array with duplicates
Output|minimum number
Solution|modified binary search: compare nums[mid] with nums[r], if nums[mid] == nums[r], r-- until nums[r] != nums[mid]
TimeCost|O(logn)
Percent|100
CornorCase|None
Tricks|[LocalPattern](#LocalPattern); [RotatedArray](#RotatedArray)

### 162 Find Peak Element

Param|Value
:---:|:---
Input|array
Output|any local peak
Solution|modified binary search, always find an increasing direction, since if head > head+1 it's also a peak
TimeCost|O(logn)
Percent|N/A
CornorCase|None
Tricks|[LocalPattern](#LocalPattern)

### 167 Two Sum II - Input array is sorted

This is a two-pointer problem.

### 209 Minimum Size Subarray Sum

This is a two-pointer problem.

### 230 Kth Smallest Element in a BST

This is a Tree Problem. (Iteration)

### 240 Search a 2D Matrix II

Param|Value
:---:|:---
Input|array of sorted array; value k
Output|whether k is in matrix
Solution|value search in sorted matrix
TimeCost|O(m+n)
Percent|79 (should be best, since best solution in leetcode takes O(mlog(n)))
CornorCase|empty input matrix, empty input matrix[0]
Tricks|[MatrixValueRank](#MatrixValueRank)

### 278 First Bad Version

Omitted

### 287 Find the Duplicate Number

Param|Value
:---:|:---
Input|array
Output|duplicate number
Solution|1. sort then iterate; 2. linked list \# 142 (this solution is a bit too oblique...)
TimeCost|1. O(nlog(n)+n);  2. O(n)
Percent|1. 32; 2. 100
CornorCase|None
Tricks|None

### 300 Longest Increasing Subsequence

Param|Value
:---:|:---
Input|array
Output|length of longest increasing subsequence
Solution|dp building a longest increasing subsequence, each elem in it is a minimum choice.
TimeCost|O(nlogn)
Percent|99
CornorCase|null/empty input array
Tricks|[BinarySearchInsertion](#BinarySearchInsertion)

### 349 Intersection of Two Arrays

This is a HashMap/HashSet problem.

### 350 Intersection of Two Arrays II (sorted arrays)

This is a HashMap/HashSet/TwoPointer problem.

### 354 Russian Doll Envelopes

This is a DP problem.

### 367 Valid Perfect Square

Param|Value
:---:|:---
Input|a number
Output|whether it is a square
Solution|modified binary sort, search among 0...n
TimeCost|O(logn)
Percent|84
CornorCase|None
Tricks|[StandardBinarySearch](#StandardBinarySearch)

### 374 Guess Number Higher or Lower

[StandardBinarySearch](#StandardBinarySearch), omitted

### 378* Kth Smallest Element in a Sorted Matrix

Param|Value
:---:|:---
Input|array of sorted array; k
Output|kth smallest elemetn
Solution|1. value binary search;  2. Priority Queue
TimeCost|1. O(nlog(max-min));  2. O((n+k)log(n))
Percent|1. 100; 2. 54
CornorCase|None
Tricks|[ValueBinarySearch](#ValueBinarySearch); [MatrixValueRank](#MatrixValueRank)

### 392 Is Subsequence

This is a two-pointer problem.

### 436* Find Right Interval

There is also a faster DP solution.

Param|Value
:---:|:---
Input|array of intervals
Output|number of lines that are fully occupied
Solution|sort by start, for each interval i, binary search for first elem whose start >= i.end
TimeCost|O(nlogn)
Percent|82
CornorCase|None
Tricks|[StandardBinarySearch](#StandardBinarySearch)

### 441 Arranging Coins

Param|Value
:---:|:---
Input|coin number, coins are to be put into lines, where there are i coins at line i
Output|number of lines that are fully occupied
Solution|modified binary sort, search among 0...n, each time comparing n & mid*(mid+1)/2
TimeCost|O(logn)
Percent|92
CornorCase|None
Tricks|[StandardBinarySearch](#StandardBinarySearch)

### 454 4Sum II

This is a HashMap problem.

### 475 Heaters

Param|Value
:---:|:---
Input|heater m positions, house n positions
Output|minimum heater radius
Solution|1. sort heaters, for each house search for insertion point, get a radius; 2. sort heaters and houses, 2 ptrs iterate
TimeCost|1. O((m+n)logm); 2. O(mlogm+nlogn+(m+n))
Percent|58
CornorCase|1. None
Tricks|[StandardBinarySearch](#StandardBinarySearch)

### 668 Kth Smallest Number in Multiplication Table

Same as \# 378

### 718 Maximum Length of Repeated Subarray

This is a DP problem.

### 744 Find Smallest Letter Greater Than Target

Param|Value
:---:|:---
Input|a list of sorted characters; a target k
Output|first letter bigger than k (array wraps around)
Solution|modified binary sort
TimeCost|O(logn)
Percent|99
CornorCase|None
Tricks|[StandardBinarySearch](#StandardBinarySearch)

### 778 Swim in Rising Water

This is a BFS problem.

### 852 Peak Index in a Mountain Array

Param|Value
:---:|:---
Input|list: A[0] < A[1] < ... A[i-1] < A[i] > A[i+1] > ... > A[A.length - 1]
Output|i
Solution|modified binary search
TimeCost|O(logn)
Percent|N/A
CornorCase|None
Tricks|[StandardBinarySearch](#StandardBinarySearch)

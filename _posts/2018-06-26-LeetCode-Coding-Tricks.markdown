---
layout:     post
title:      "Java Algo Coding Tricks"
subtitle:   "LeetCode Review sub1"
date:       2018-06-25 12:00:00
author:     "Farrell"
header-img: "img/bg3.jpg"
catalog: true
tags:
    - Leetcode
---

> to be polished...

## Coding Tricks

### <span id="loopNoRecursion">1 Loop better than Recursion</span>
Loops costs less than recursion.

### <span id="miniCodeBlock">2 Minimize Code Block</span>
Remember to minimize basic code blocks, could make huge difference on time cost without modyfing logic. The 1st one below costs 10% more than the second one.
```java
if (l1.val <= l2.val) {
    l3.next = l1;
    l1 = l1.next;
    l3 = l3.next;
} else {
    l3.next = l2;
    l2 = l2.next;
	l3 = l3.next;
}
```
```java
if (l1.val <= l2.val) {
    l3.next = l1;
    l1 = l1.next;
} else {
    l3.next = l2;
    l2 = l2.next;
}
l3 = l3.next;
```

### <span id="stackedLoop">3 Stacked Loops</span>
Stacked loops cost less than one big loops that handles multiple scenarios.
```java
for(int i=0; i<k; i++, mod--){
    do_something();
    for(int j=0; some_bar(); j++) root = root.next;
    do_something()
}
```

### <span id="insertionSortModification">4 Insertion Sort Modification</span>
One-line code speed up insertion sort:
```java
while(leftPtr<nums.length){
	if (nums[leftPtr] > nums[rightPtr]) leftPtr = 0;	// speed uuup
	find_insertion_pos();
	insert();
}
```

### <span id="MINMAX_VALUE"> 5 Integer MIN MAX cost</span>
`Integer.MIN_VALUE` & `MAX_VALUE` cost 3ms. So when initializing numbers use 0 unless necessary.

### <span id="ComparatorCompare"> 6 Comparator Compare</span>
in `Comparator::compare()`, returning -1/0/1 would be faster than returning integer diffs


### <span id="useArrayNotMap"> 7 Array Instead of HashMap</span>
When counting fixe-sized elements such as characters, arrays are much faster than hashmap.

### <span id="StringBuilderNotString"> 8 StringBuilder is faster than String</span>
much faster

### <span id="CharArrayNotSB"> 8.1 CharArray is faster than StringBuilder</span>
much faster again...

### <span id="arrayLengthCost"> 9 ArrayList length func cost</span>
it costs a lot

### <span id="addOverflowsInteger"> 10 Integer Overflow caused by Adding</span>
`l + r` may cause overflow, use l + (r-l)/2 instead.

### <span id="..."> 11 cause not sure </span>
why is the following first faster than following second ?
```java
TreeNode l = helper(nums, start, cur-1);
root.left = l;
```
```java
root.left = helper(nums, start, end);
```

### <span id="classMemberEqualsRef"> 12 class member equals reference</span>
The following tow functions both successfully record tree features by updating array.
```java
class Solution {
    ArrayList<Integer> arr = new ArrayList<Integer>();
    
    public List<Integer> largestValues(TreeNode root) {
        helper(root, 0);
        ArrayList<Integer> arr = new ArrayList<Integer>();
        for (int cur: this.arr) arr.add(cur);
        return arr;
    }
    
    public void helper(TreeNode root, int level){
        if (root == null) return;
        if(level >= this.arr.size()) this.arr.add(root.val);
        if (root.val > this.arr.get(level)) this.arr.set(level, root.val);
        helper(root.left, level+1);
        helper(root.right, level+1);
    }
}
```

```java
class Solution {
    public List<Integer> largestValues(TreeNode root) {
        List<Integer> ret = new ArrayList<>();
        findMax(root, 1, ret);
        return ret;
    }
    
    private void findMax(TreeNode n, int level, List<Integer> ret) {
        if (n == null) return;
        if (ret.size() < level) ret.add(Integer.MIN_VALUE);
        if (ret.get(level - 1) < n.val) ret.set(level - 1, n.val);
        findMax(n.left, level + 1, ret);
        findMax(n.right, level + 1, ret);
    }
}
```

### <span id="arrayRefFasterThanPrimePassing"> 13 array reference is faster than prime type passing</span>
```java
int[] a = {1};
do_something(a);
```

slower than following:
```java
int a=1;
a = do_something(a);
```

### <span id="ArrayAsMap">14 Array as Map</span>
When dealing with fix-sized maps, we could use array instead, which is much faster than `HashMap` class. The following example is when we need a map for english lower case characters.
```java
int[] map = new int[26];
for(int i=0; i<S.length(); i++) 
    map[(int)(S.charAt(i)-'a')]++;
```

### <span id="">15 == much faster than \></span>
as it is
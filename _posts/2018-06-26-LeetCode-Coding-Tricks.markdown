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

### <span id="arrayLengthCost"> 9 ArrayList length func cost</span>
it costs a lot
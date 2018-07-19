---
layout:     post
title:      "Linked List"
subtitle:   "LeetCode Review 1"
date:       2018-06-22 12:00:00
author:     "Farrell"
header-img: "img/bg3.jpg"
catalog: true
tags:
    - Leetcode
---

> Starred Problems: 23, 19

## 1. Data Structure
A linked list is a linear collection of data elements, whose order is not given by their physical placement in memory. Instead, each element points to the next.  
```java
 public class ListNode {
     int val;
     ListNode next;
     ListNode(int x) { val = x; }
 }
```
A key feature of linked list is that with linked list, we access data by relative positions. Relative positions prevents us from knowing in advance the accurate position of an element, but it better preserves the correlation between data pieces (thus linked list problems also have few cornor cases). When dealing with linked list in which there are complex node correlations, our best chance is that we preserve the original linked list and add/remove/seperate elements from it.

---
## 2. Methods
### <span id="reversing">2.1 Reversing</span>
```java
public ListNode reverseList(ListNode head) {
    if (head == null) return null;
    ListNode tail = head;
    head = head.next;
    tail.next = null;
    while (head != null) {
        ListNode tmp = head;
        head = head.next;
        tmp.next = tail;
        tail = tmp;
    }
    return tail;
}
```

### <span id="deleting">2.2 Deleting</span>
1. When pointer points to prev:  
	- `prev = prev.next.next`;
2. When pointer points to the node to delete: 
	- `while (cur.next != null) {cur.val = cur.next.val; cur = cur.next}`

### <span id="insertion"> 2.3 Insertion</span>
```java
public boolean insertion(ListNode insert_prev, ListNode target_prev) {
    ListNode tmp = target_prev.next.next;
    target_prev.next.next = insert_prev.next;
    insert_prev.next = target_prev.next;
    target_prev.next = tmp;
}
```

### <span id="findCycle">2.4 Cycle Problems</span>
to find out whether there is a cycle: fast casing slow, timecost O(n)
```java
public boolean hasCycle(ListNode head) {
    if(head == null || head.next == null) return false;
    ListNode one = head;
    ListNode two = head.next;
    ListNode mark = head;
    boolean start = true;
    while(start == true || !one.equals(mark)){
        if (one == null) return false;
        if (one.equals(two)) return true;
        one = one.next;
        if (two == null || two.next == null) return false;
        two = two.next.next;
    }
    return false;
}
```
to index the cycle starting node, use another slow from head after the first slow meets fast

### <span id="concatenation">2.5 Concatenation</span>
Here it make two concatenations: new1 = l1+l2, new2 = l2+l1. So if there exists a node where l1 and l2 intersects, number of nodes before the node would be equal.  
![](/img/in-post/2018-06-23-Linked-List/1.png)

```java
public ListNode getIntersectionNode(ListNode headA, ListNode headB) {
    
    if (headA == null || headB == null) return null;
    ListNode headab = headA;
    ListNode headba = headB;
    while(headab != headba){
        headab = headab == null ? headB : headab.next;
        headba = headba == null ? headA : headba.next;
    }
    return headab;
}
```

### <span id="swapping"> 2.6 Swap</span>
```java
public void swap(ListNode head){
	// here I just swap the first two nodes:
	ListNode one = head.next;
	ListNode two = one.next;
	ListNode thr = two.next;
	head.next = two;
	two.next = one;
	one.next = thr;
}
```

### <span id="indexing"> 2.7 Indexing</span>
Two pointers are used for indexing. The most frequent usage is find middle (as follows);  
```java
public ListNode findMiddle(ListNode head, int k) {
    ListNode fast = head;
    ListNode slow = head;
    while (fast != null && fast.next != null) {
        fast = fast.next.next;
        slow = slow.next;
    }
    return slow;
}
```
and indexing from the end:
```java
public ListNode findKthFromEnd(ListNode head, int k) {
    ListNode fast = head;
    ListNode slow = head;
    for(int i=0; i<k; i++) fast = fast.next;
    while (fast != null) {
        fast = fast.next;
        slow = slow.next;
    }
    return slow;
}
```

### <span id="sorting"> 2.8 Linked List Sorting</span>

insertion sort
merge sort

### <span id="useListSelf"> 2.9 Use Linked List Itself</span>
Perform actions on the structure of the linked list to reach your purpose.

---
## 3. Solutions

### 2 Add Two Numbers
Same as \# 445

### 19 Remove Nth Node From End of List

Param|Value
:---:|:---
Input|linked list, n
Output|linked list with the nth node from behind removed
Solution|2 pointers, one faster than another by n steps, loop over linked list so slow pointer would point to the node to delete
TimeCost|O(n), n is size of input linked list
Percent|100
CornorCase|None
Tricks|[deleting](#deleting); [indexing](#indexing)

### 21 Merge Two Sorted Lists

Param|Value
:---:|:---
Input|two sorted lists
Output|one merged sorted list
Solution|two pointers iteration
TimeCost|O(m+n), m,n are sizes of two input lists.
Percent|95%
CornorCase|None
Tricks|[miniCodeBlock](#miniCodeBlock): improved 20% after extracting a same sentence from if...else...

### 23 Merge k Sorted Lists

Param|Value
:---:|:---
Input|n sorted lists
Output|one merged & sorted linked list
Solution|1. PriorityQueue; 2. MergeSort
TimeCost|1. 2nlogn; 2. nlogn
Percent|1. 67
CornorCase|empty ListNode[]; null whithin ListNode[]
Tricks|[ComparatorCompare](#ComparatorCompare)

### 24 Swap Nodes in Pairs

Param|Value
:---:|:---
Input|linked list
Output|swapped linked list
Solution|loop over linked list, swap next 2 nodes, then move 2 steps forward
TimeCost|O(n), n is size of input linked list
Percent|86
CornorCase|None
Tricks|[swapping](#swapping)

### 25 Reverse Nodes in k-Group

Param|Value
:---:|:---
Input|linked list
Output|k-grouped reversed list
Solution|take k steps at a time, add reversed sublist to result
TimeCost|O(n)
Percent|99
CornorCase|None
Tricks|[swapping](#swapping)

### 61 Rotate List

Param|Value
:---:|:---
Input|linked lists, rotating point k
Output|rotated linked list
Solution|1st iteration count list length; calc new start pos, link tail to start; 2nd iteration find new start, add null, return
TimeCost|2n
Percent|96
CornorCase|k==0; head==null; k>list.length; huge k
Tricks|None

### 82 Remove Duplicates from Sorted List II

Param|Value
:---:|:---
Input|linked list
Output|linked list with all duplicate numbers removed
Solution|loop over, when locating duplicates use inner loop to jump over
TimeCost|O(n), n is size of input linked list
Percent|100
CornorCase|None
Tricks|[stackedLoop](#stackedLoop)

### 83 Remove Duplicates from Sorted List

Param|Value
:---:|:---
Input|linked list
Output|linked list without duplicates
Solution|one-time loop with hashmap
TimeCost|O(n), n is size of input linked list
Percent|95%
CornorCase|empty linked list
Tricks|None

### 86 Partition List

Param|Value
:---:|:---
Input|linked list, parting number
Output|linked list parted by number, lesser number in the left, bigger number in the right with original order preserved
Solution|as insertion sort
TimeCost|O(n)
Percent|83
CornorCase|None
Tricks|[insertion](#insertion)

### 92 Reverse Linked List II

Param|Value
:---:|:---
Input|linked list, m, n
Output|linked list with the mth-nth nodes reversed
Solution|tag m-1 th node, move m-nth node to m pos
TimeCost|O(n)
Percent|97
CornorCase|None
Tricks|[insertion](#insertion)

### 109 Convert Sorted List to Binary Search Tree

Param|Value
:---:|:---
Input|sorted linked list
Output|BST
Solution|recursively building BST, find middle by 2 ptrs
TimeCost|O(nlogn)
Percent|99%
CornorCase|empty linked list
Tricks|[indexing](#indexing)

### 138 Copy List with Random Pointer

Param|Value
:---:|:---
Input|linked list with random pointers
Output|copied linked list
Solution|1st iteration: copy nodes, link them after their prototype; 2nd iteration: adjust random links of copied nodes; 3rd iteration: seperate copied list from original list
TimeCost|3n
Percent|99%
CornorCase|None
Tricks|[useListSelf](#useListSelf)

### 141 Linked List Cycle

Param|Value
:---:|:---
Input|linked list
Output|boolean: cycled or not
Solution|two pointers, one move 1 step at a time; the other move 2 steps at a time. If in (size) iterations they never met, no cycle exists
TimeCost|O(n), n is size of input linked list
Percent|99%
CornorCase|None
Tricks|[findCycle](#findCycle)

### 142 Linked List Cycle II

Param|Value
:---:|:---
Input|Linked List with Loop
Output|The Node where the loop begins
Solution|first use \# 141 to find meeting point, set as ptr1; set ptr2 at head; proceed ptr1 and ptr2 each by 1 step each time, their meeting point is the begin of the loop
TimeCost|O(n)
Percent|100
CornorCase|None
Tricks| Math :-)

### 143 Reorder List

Param|Value
:---:|:---
Input|linked list
Output|l1 -> ln -> l2 -> ln-1 -> l3 -> ln-2 -> ...
Solution|1st iteration find middle, then reverse latter half; 2nd half iteration insert latter half into front half
TimeCost|1.5n
Percent|96%
CornorCase|empty linked list
Tricks|[insertion](#insertion); [indexing](#indexing)

### 147 Insertion Sort List

Param|Value
:---:|:---
Input|linked list
Output|sorted linked list
Solution|pay attention to the modification to insertion sort algorithm
TimeCost|O(n^2)
Percent|97
CornorCase|None
Tricks|[stackedLoop](#stackedLoop); [insertionSortModification](#insertionSortModification); [insertion](#insertion)

### 148 Sort List

Param|Value
:---:|:---
Input|linked list
Output|sorted linked list
Solution|merge sort, since it builds sorted list from bottom to top. use one iteration to find middle each time, costs O(n)
TimeCost|O(nlogn)
Percent|91
CornorCase|None
Tricks|None

### 160 Intersection of Two Linked Lists

Param|Value
:---:|:---
Input|two linked lists
Output|intersection
Solution|construct two new linked lists: l1+l2 and l2+l1, if intersection exists, it would show in these two linked lists.
TimeCost|O(m+n), m,n are sizes of two input linked list
Percent|96
CornorCase|empty input linked list
Tricks|[concatenation](#concatenation)

### 203 Remove Linked List Elements

same as \# 82, \# 83

### 206 Reversed Linked List

Param|Value
:---:|:---
Input|linked list
Output|reversed linked list
Solution|repeatedly move current head to tail list head. 
TimeCost|O(n), n is size of input linked list; 
Percent|100% for non-recursive, 30% for recursive.
CornorCase|empty linked list
Tricks|[reversing](#reversing); [loopNoRecursion](#loopNoRecursion)

### 234 Palindrome Linked List

Param|Value
:---:|:---
Input|linked list
Output|boolean, whether linked list is palindrome
Solution|reverse first half of linked list, compare it to second half
TimeCost|2n, n is size of input linked list; first iteration find length; second iteration reverse first half + compare to 2nd half
Percent|92%
CornorCase|empty linked list
Tricks|[reversing](#reversing)

### 237 Delete Node in a Linked List

Param|Value
:---:|:---
Input|Node in a linked list
Output|Delete current node
Solution|loop over the remaining nodes, shifting all values backward by one position.
TimeCost|O(n), n is size of input linked list; 
Percent|100%
CornorCase|None
Tricks|[deleting](#deleting)

### 328 Odd Even Linked List

Param|Value
:---:|:---
Input|linked list
Output|1th-3th-5th-...-2th-4th-6th-...
Solution|move odd indexed nodes to front
TimeCost|O(n), n is size of input linked list
Percent|85
CornorCase|None
Tricks|[insertion](#insertion)

### 445 Add Two Numbers II

Param|Value
:---:|:---
Input|two linked lists
Output|add numbers presented by inputs, return a linked list representing the sum
Solution|put input linked list units into stack, then perform adding
TimeCost|O(m+n), m,n are sizes of input linked lists
Percent|86
CornorCase|None
Tricks|Stack: adding begins at tail

### 725 Split Linked List in Parts

Param|Value
:---:|:---
Input|linked list, number of parts
Output|ListNode[]
Solution|1st iteration calc list length & seg length; 2nd iteration split list
TimeCost|O(n), n is size of input linked list
Percent|100
CornorCase|None
Tricks|[stackedLoop](#stackedLoop)

### 817 Linked List Components

Param|Value
:---:|:---
Input|linked list, selected nodes
Output|connected component number
Solution|loop over linked list, component++ when `curNode in map && curNode.next not in map`
TimeCost|O(n)
Percent|NA
CornorCase|None
Tricks|HashMap; [stackedLoop](#stackedLoop)
## 🚀 AT&T DSA Fast-Track (Python)

### **Arrays & Two Pointers**

1. **Find Minimum/Maximum in Array** - [GeeksforGeeks](https://www.geeksforgeeks.org/problems/find-minimum-and-maximum-element-in-an-array4428/1) (Basic)
2. **Reverse an Array** - [GeeksforGeeks](https://www.geeksforgeeks.org/problems/reverse-an-array/1) (Basic)
3. **Move Zeroes to End** - [LeetCode](https://leetcode.com/problems/move-zeroes/) (Easy)
4. **Two Sum (Sorted)** - [LeetCode](https://leetcode.com/problems/two-sum-ii-input-array-is-sorted/) (Easy)
5. **Remove Duplicates from Sorted Array** - [LeetCode](https://leetcode.com/problems/remove-duplicates-from-sorted-array/) (Easy)
6. **Container With Most Water** - [LeetCode](https://leetcode.com/problems/container-with-most-water/) (Medium)
7. **Best Time to Buy and Sell Stock** - [LeetCode](https://leetcode.com/problems/best-time-to-buy-and-sell-stock/) (Easy)

### **Strings & Sliding Window**

8. **Palindrome Check** - [GeeksforGeeks](https://www.geeksforgeeks.org/problems/palindrome-string0817/1) (Basic)
9. **Valid Anagram** - [LeetCode](https://leetcode.com/problems/valid-anagram/) (Easy)
10. **Reverse Words in a String** - [LeetCode](https://leetcode.com/problems/reverse-words-in-a-string/) (Medium)
11. **Longest Substring Without Repeating Characters** - [LeetCode](https://leetcode.com/problems/longest-substring-without-repeating-characters/) (Medium)
12. **Maximum Sum Subarray of size K** - [GeeksforGeeks](https://www.geeksforgeeks.org/problems/max-sum-subarray-of-size-k5313/1) (Easy)

### **HashMap (Dictionaries in Python)**

13. **Two Sum** - [LeetCode](https://leetcode.com/problems/two-sum/) (Easy)
14. **First Unique Character in a String** - [LeetCode](https://leetcode.com/problems/first-unique-character-in-a-string/) (Easy)
15. **Group Anagrams** - [LeetCode](https://leetcode.com/problems/group-anagrams/) (Medium)
16. **Find All Duplicates in an Array** - [LeetCode](https://leetcode.com/problems/find-all-duplicates-in-an-array/) (Medium)

### **Linked List**

17. **Middle of the Linked List** - [LeetCode](https://leetcode.com/problems/middle-of-the-linked-list/) (Easy)
18. **Reverse a Linked List** - [LeetCode](https://leetcode.com/problems/reverse-linked-list/) (Easy)
19. **Detect Cycle in Linked List** - [LeetCode](https://leetcode.com/problems/linked-list-cycle/) (Easy)
20. **Merge Two Sorted Lists** - [LeetCode](https://leetcode.com/problems/merge-two-sorted-lists/) (Easy)
21. **Remove Nth Node From End** - [LeetCode](https://leetcode.com/problems/remove-nth-node-from-end-of-list/) (Medium)

### **Stacks & Queues**

22. **Valid Parentheses** - [LeetCode](https://leetcode.com/problems/valid-parentheses/) (Easy)
23. **Implement Queue using Stacks** - [LeetCode](https://leetcode.com/problems/implement-queue-using-stacks/) (Easy)
24. **Min Stack** - [LeetCode](https://leetcode.com/problems/min-stack/) (Medium)
25. **Next Greater Element** - [GeeksforGeeks](https://www.geeksforgeeks.org/problems/next-larger-element-1587115620/1) (Medium)

### **Recursion**

26. **Factorial & Fibonacci** - [GeeksforGeeks](https://www.geeksforgeeks.org/problems/nth-fibonacci-number1335/1) (Basic)
27. **Binary Search (Recursive)** - [LeetCode](https://leetcode.com/problems/binary-search/) (Easy)
28. **Subset Generation** - [LeetCode](https://leetcode.com/problems/subsets/) (Medium)
29. **Letter Combinations of a Phone Number** - [LeetCode](https://leetcode.com/problems/letter-combinations-of-a-phone-number/) (Medium)
30. **Climbing Stairs** - [LeetCode](https://leetcode.com/problems/climbing-stairs/) (Easy)

---

## 🛠️ Recovery Techniques: What to do if Stuck

If your friend hits a wall during the live coding, tell him to use these "Safety Valves":

1. **The "Brute Force" Confession:** Say, *"I’m currently thinking of a brute force way using nested loops which would be O(N²). Should I implement that first, or would you like me to try and optimize it to O(N) immediately?"* This buys time and shows you have a solution.
2. **Talk through the Constraints:** If the input size is small (N < 1000), O(N²) might be okay. Discussing this shows maturity.
3. **Input/Output Tracing:** Literally draw the array on the screen/whiteboard. Move your cursor like a "pointer" to show the interviewer your logic. Often, seeing the movement helps the brain find the missing `if` condition.
4. **Ask for a "Sanity Check":** Ask, *"Am I headed in the right direction with a HashMap approach, or is there a simpler way you'd like me to consider?"*

---

## ⚠️ Python-Specific OOPs Traps

Interviewers love to catch Python "beginners" with these specific quirks:

* **The `self` Trap:** Many beginners forget to put `self` as the first argument in class methods or forget to use `self.variable_name` inside the class.
* **Mutable Default Arguments:** Never use a list or dict as a default argument (e.g., `def func(a=[])`). In Python, that list is created **once** and shared across all calls. Use `None` instead.
* **`__init__` is NOT a Constructor:** It’s an initializer. The object is already created when `__init__` is called.
* **Multiple Inheritance (MRO):** Python supports multiple inheritance. If asked how it handles conflicts, the answer is **MRO (Method Resolution Order)** using the C3 Linearization algorithm.
* **Private Variables:** Python doesn't have "true" private variables. We use a single underscore `_var` (convention) or double underscore `__var` (name mangling).

---

## 📘 Essential Theoretical Topics

* **OOPs:** Classes, 4 Pillars (Inheritance, Polymorphism, Encapsulation, Abstraction), and the use of `@staticmethod` vs `@classmethod`.
* **DBMS:** SQL Joins, Normalization, and ACID properties (Atomicity, Consistency, Isolation, Durability).
* **Computer Networks:** OSI Layers (know 1, 3, 4, 7 perfectly), TCP vs UDP, and DNS.
* **Operating Systems:** Process vs Thread, Paging, and Virtual Memory.

---

### **Preparation Strategy**

* **Dry Run:** Always trace code on paper first.
* **Explain the "Why":** AT&T wants to see the thought process.
* **Video Guide:** [AT&T Technical Interview Process & Tips](https://www.youtube.com/watch?v=RCfjiVSw1KE)

**This guide covers the logic, the coding, and the "soft" technical skills needed to pass. Go ahead with perfection.**

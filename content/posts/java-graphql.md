---
author: "Phong Nguyen"
title: "Complexity"
date: "2026-01-27"
description: "Complexity of an Algorithm"
tags: ["cs"]   #tags search
FAcategories: ["syntax"]    #The category of the post, similar to tags but usually for broader classification.
FAseries: ["Themes Guide"]    #indicates that this post is part of a series of related posts
aliases: ["migrate-from-jekyl"]    #Alternative URLs or paths that can be used to access this post, useful for redirects from old posts or similar content.
ShowToc: true    # Determines whether to display the Table of Contents (TOC) for the post.
TocOpen: true    # Controls whether the TOC is expanded when the post is loaded. 
weight: 13    # The order in which the post appears in a list of posts. Lower numbers make the post appear earlier.
---

References:
[How to Find the Complexity of an Algorithm](https://www.baeldung.com/cs/algorithm-complexity-analysis)

## 1. Introduction
- Algorithmic complexity is a measure of the resources and algorithm requires about its input size.
- The two main types are:
  - `Time complexity`: number of operations performed by an algorithm
  - `Space complexity`: the amount of memory consumed

## 2. Time Complexity
- The time complexity of an algorithm quantifies the amount of time (operations) taken by an algorithm to run as a function of the length of the input
- It's commonly expressed by *Big-O notation*
- Hierarchy of Time Complexities (Fastest to Slowest): 
    \(O(1)\) - Constant: Fastest; performance is independent of input size.
    \(O(\log n)\) - Logarithmic: Very fast, typical for binary searches.
    \(O(n)\) - Linear: Efficient, grows proportionally to input.
    \(O(n\log n)\) - Linearithmic: Common for efficient sorting, like QuickSort.
    \(O(n^{2})\) - Quadratic: Acceptable for small input, common in nested loops.
    \(O(2^{n})\) or \(O(n!)\) - Exponential/Factorial: Generally only acceptable for very small inputs. 
### 2.1. Constant Time - \(O(1)\)
- It takes the same amount of time regardless[rɪˈɡɑːrd.ləs] of input size.

### 2.2 Linear Time - \(O(n)\)
[ˈlɪn.i.ɚ]
- It analyze each input element once, but can become inefficient with bigger data sets.

### 2.3. Polynomial Time - \(O(n^m)\)
[ˌpɑː.liˈnoʊ.mi.əl]
- It's caused by nested loop. Each loop processes the input, increasing execution time as the input size grows.

### 2.4. Logarithmic Time - \(O(log n)\)
- It reduce the size of the problem with each step, making them suitable for large datasets.

## 3. Evaluating Time Complexity
- How long it takes to execute as the size of the input increases.
### 3.1. Loop
- The time complexity for a single loop that iterates n times is \(O(n)\)
=> Finding the number of nesting layers and observing how the number of iterations varies with the size of the input are crucial steps in identifying loop-related complexities.
- E.g.
```
def cubic_loop(arr):
    for i in arr:  
        for j in arr:  
            for k in arr: 
                print(i, j, k)
=> O(n^3)
```
### 3.2. Recursive Calls
-  The complexity of divide and conquer algorithms, such as merge sort, which split and recombine data, is determined by recurrence relations and frequently yields \(O(n log n)\)
- E.g.
```
def binary_search(arr, target, low, high):
    if low > high:
        return -1
    mid = (low + high)
    if arr[mid] == target:
        return mid
    elif arr[mid] < target:
        return binary_search(arr, target, mid + 1, high)  
    else:
        return binary_search(arr, target, low, mid - 1)
=> O(logn)
```
### 3.3. Constant Time Operations
- In small operations, they dominate the complexity, but in big algorithms, they are usually insignificant when compared to loops or recursion, is \(O(1)\)
- E.g.
```
def constant_time_operations(arr):
    print(arr[0])
    print(len(arr))
    print(arr[0] + arr[1]) 
=> O(1)
```

## 4. Space Complexity
- The space complexity measures how much memory an algorithm requires for the size of its input.
### 4.1. Constant Time - \(O(1)\)
- It requires a give amount of memory, regardless of input size.
- It's common in algorithms that keep or process every element of the input.
### 4.2. Logarithmic Space
- It's frequently in recursive techniques, where the recursion depth determines memory usage.
### 4.3. Quadratic Space
- It's frequently in recursive techniques, where the recursion depth determines memory usage.

## 5. Evaluating Space Complexity
TBD
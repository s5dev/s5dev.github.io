---
layout: post
title: Binary Search and Hidden Overflow 🪲
categories: [security, programming, overflow]
tags: [security, programming]
description: Interesting post on integer overflow while performing a basic binary search
comments: true
---

Recently I was playing with overflow vulnerabilities help of `exploit.education` exercise which mostly covers basic heap, buffer overflow,
use-after-free vulnerability patterns in a contained `qemu` based environment. However, I was searching for Integer overflow patterns and articles around it "how to succesfully convert a integer overflow into a remote code execution". While reading through the vulnerability reports, I started exploring code snippets relevant to integer overflow and this blog post [`Nearly All Binary Searches and Mergesorts are Broken`](https://ai.googleblog.com/2006/06/extra-extra-read-all-about-it-nearly.html) caught my eyes. 

On the other day, I was practicing [binary search algorithm](https://leetcode.com/explore/learn/card/binary-search/) in leetcode. This particular line `pivot := (high + low) / 2` caught my eyes which was the culprit that triggered this overflow 🪲. There is no significant difference between `pivot = left + (right - left) / 2;` vs `pivot := (high + low) / 2` until (high + low) starts overflow beyond int32/int64. In this article, we'll take a quick look at the results of binary search when overflow happens.

### Compliant Code

```golang
pivot = left + (right - left) / 2;
```

### Vulnerable Code

```golang
pivot := (high + low) / 2
```

For example, I have intentionally created a overflow here with the help of `int8` data type in golang which can hold `-128 to 127` range of values. You can clearly see the error message with out-of-bounds exception in runtime when `search(126)` function is invoked. I have purposefully used `int8` for making this overflow obvious to make it easier to understand, However, this may happen even with `int64` (depends on system arch). Demo [here](https://go.dev/play/p/qGftxCMfnRW)

```golang
func search(nums []int8, target int8) int8 {
	low, high := int8(0), int8(len(nums)-1)

	for low <= high {
		fmt.Println(high + low)
		mid := int8((high + low) / 2)
		if nums[mid] < target {
			low = mid + 1
		} else if nums[mid] > target {
			high = mid - 1
		} else {
			return mid
		}
	}

	return -1
}
```
```
125
-68
panic: runtime error: index out of range [-34]

goroutine 1 [running]:
main.search({0xc000074ee2, 0x7e, 0x7f2a745b7108?}, 0x7e)
	/tmp/sandbox1190866906/prog.go:18 +0x116
main.main()
	/tmp/sandbox1190866906/prog.go:9 +0x16a

Program exited.
```

Whereas when you replace the overflow line `(high + low) / 2` with `low + ((high - low) / 2)`, this would respond with correct index. 
Demo [here](https://go.dev/play/p/9HBvUPepjSM)

```golang
func search(nums []int8, target int8) int8 {
	low, high := int8(0), int8(len(nums)-1)

	for low <= high {
		mid := int8(low + ((high - low) / 2))
		fmt.Println(mid)
		if nums[mid] < target {
			low = mid + 1
		} else if nums[mid] > target {
			high = mid - 1
		} else {
			return mid
		}
	}

	return -1
}
```

### Source and Reference:

1. Google Research Blog: [Nearly All Binary Searches and Mergesorts are Broken](https://ai.googleblog.com/2006/06/extra-extra-read-all-about-it-nearly.html)
2. [CVE-2013-5619 - binary searches use overflow-prone arithmetic](https://bugzilla.mozilla.org/show_bug.cgi?id=917841)

### Closing Note:

Though the impact of this overflow remains unknown or I (personally) donot know a way to exploit this overflow, However the impact is huge on application logic side as binary search has lot of usecases and these APIs are used widely across the systems.

I hope this post is helpful for vulnerability researcher 🔍 & code reviewers, For bugs,hugs & discussion, DM in [Twitter](https://twitter.com/sshivasurya). Opinions are my own and not the views of my employer.

+++
author = "James D Hughes"
categories = ["algorithms", "golang"]
date = 2018-11-13T10:47:10Z
description = ""
draft = false
slug = "brainteasers-theyre-important"
tags = ["algorithms", "golang"]
title = "Brainteasers - they're important!"

+++


48 days ago I signed up for [https://www.dailycodingproblem.com/](https://www.dailycodingproblem.com/)

I haven't completed all 48 problems however the ones that I have completed, I've had a blast with.  It bring back a challenge that's been missing - not that my work and personal life don't have their own challenges.  It brings me back to basics and gets me thinking again on algorithm design and optimization of code.

Much of my development work (which is less and less these days) is done using a framework of some sort which puts a layer of 'magic' on top of the coding experience.  I've actually recently gotten a little miffed at just how much 'magic' is built into a toolset and have determined that too much 'magic' is killing the next generation of developers - but that is a topic for a different day.

Daily coding problem - a delightful brain teaser sent to my email every day for me to think about and hopefully solve! The beauty of these is the 'get thinking in an algorithmic way' that has shrunk since my degree completed.  An algorithm _really_ has three parts:

1. Initialization
2. Logic / Processing
3. Termination

The point of an algorithm is to show that you can go from an initial state, through some logic and processing to a terminated / finished state with the correct answer.  It's almost frustrating how this simple concept is lost on so many people once they finish school.

I completed one this weekend that I was impressed with myself on.  Not impressed because of the elegance of my solution, not impressed with the performance of the solution, but impressed with how quickly I was able to go from _reading_ the problem to a successful implementation of a solution.

# The Problem:

This problem was asked by Amazon.

Given a string, find the longest palindromic contiguous substring. If there are more than one with the maximum length, return any one.

For example, the longest palindromic substring of "aabcdcb" is "bcdcb". The longest palindromic substring of "bananas" is "anana".

# The Plan

## 1) Initialization

I'll need a string to hold the largest witnessed palindrome, a recursive function to step through my array in a top-down manner, and maybe a helper function to determine if a substring is a palindrome or not

## 2) Logic

I'll check the full string first. If it's a palindrome - I'm done! Next we'll check the substring from 0-(n-1) and the substring from 1- (n). The result will be the largest palindrome of each of those two substrings.  We'll recurse over them until we find a palindrome and filter our way back up.

## 3) Termination

If the length of the string reaches 1 (single character) or the string is a palindrome, return the string. Else, continue through with our logic.

That's it.  Three simple steps to get to our answer. Now we just need to implement it.

```go
package main

import (
	"fmt"
)

func main() {
	fmt.Println(FindPalindrome("baabdaad"))
}

func FindPalindrome(s string) string {
	return PalindromeRecursive(s, len(s))
}

func IsPalindrome(s string) bool {
	var i, j int
	j = len(s) - 1
	for i = 0; i < j; i++ {
		if s[i] != s[j] {
			return false
		}
		j--
	}
	return true
}

func PalindromeRecursive(s string, n int) string {
	if n == 1 || IsPalindrome(s) {
		return s
	}
	pLeft := PalindromeRecursive(s[1:], len(s)-1)
	pRight := PalindromeRecursive(s[:len(s)-1], len(s)-1)
	if len(pLeft) > len(pRight) {
		return pLeft
	}
	return pRight
}
```

Then some tests to make sure it works

```go

package main

import "testing"

func TestShortPalindrome(t *testing.T) {
	s := "aba"
	res := FindPalindrome("aba")
	if res != s {
		t.Errorf("Expected %s got %s", s, res)
	}
}

func TestNoPalindrome(t *testing.T) {
	s := "abcdefg"
	res := FindPalindrome(s)
	if len(res) > 1 {
		t.Errorf("Expected no palindrome. Found %s", res)
	}
}

func TestPalindromeIsFullString(t *testing.T) {
	s := "tattarrattat"
	res := FindPalindrome(s)
	if res != s {
		t.Errorf("Expected %s got %s", s, res)
	}
}

func TestLongWordNoPalindrome(t *testing.T) {
	s := "abcdefghijklmnopqrstuvwxyz"
	res := FindPalindrome(s)
	if len(res) > 1 {
		t.Errorf("Expected no palindrome. Found %s", res)
	}
}

func testEvenPalindrome(t *testing.T) {
	s := "abba"
	res := FindPalindrome(s)
	if res != s {
		t.Errorf("Expected %s got %s", "abba", res)
	}
}

func TestOddPalindrome(t *testing.T) {
	s := "aabaa"
	res := FindPalindrome(s)
	if res != s {
		t.Errorf("Expected %s got %s", "aabaa", res)
	}
}

func TestEvenPalindromeMidString(t *testing.T) {
	s := "abcdefgabbaabcdefg"
	res := FindPalindrome(s)
	if res != "abba" {
		t.Errorf("Expected %s got %s", "abba", res)
	}
}

func TestOddPalindromeMidString(t *testing.T) {
	s := "abcdefgabcbaaxcdefg"
	res := FindPalindrome(s)
	if res != "abcba" {
		t.Errorf("Expected %s got %s", "abcba", res)
	}
}
```

They all pass - so at least the algorithm is pretty resilient.

This is simply an ode to the brainteaser.  If you find your mental agility getting a little slow, I'd suggest signing up for something such as this as it will help sharpen up that wit a touch.  This is a programmer-specific example but I'm sure it applies to many other industries. I just don't have a service for you to play with :)

My github has my progress on these and I'm writing them all in go. Check it out at https://github.com/daimonos/dcp

Hopefully I'll be able to keep up with them a little better than I have been.

Have fun!




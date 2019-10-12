+++
tags = ["algorithm", "csharp", "dynamic programming", "subset sum problem"]
categories = ["Algorithms"]
title = "Solving a payment without change problem"
math = true
markup = "mmark"
date = 2016-07-30T23:21:00+03:00
draft = false
+++

I have come across an interesting problem. A customer is standing at the checkout of a grocery store with his purchases and is asked to pay the exact amount without creating any change. There are notes of various denominations  in his pockets in random order (note denomination is not tied to a real bank notes, the only boundary is that it is an integer greater than 0). The objective is to pick up the necessary sum or indicate that it is not possible. If several solutions are possible then any of them is acceptable. So, let's solve it using C# language.
<!--more-->

This problem is a special case of the knapsack problem (given a set of items, each with a weight and a value, determine the number of each item to include in a collection so that the total weight is less than or equal to a given limit and the total value is as large as possible) and is equivalent to a subset sum problem (finding a non-empty subset whose sum is zero) with an extra condition that each element of the set is strictly greater than zero. As it is an NP-complete problem I can think of 2 possible ways to solve it. Let's first discuss the recursive approach and then improve it using dynamic programming.

## Recursive Approach

This is the most naive algorithm. For every element in the set there are two options, either we will include that element in the subset or we won’t include it. Then cycle through all these subsets and, for every one of them, check if the subset sum equals the right number. This solution is quite similar to _generating all strings of n bits_ and its time complexity is as high as_ O(n\*2n)_, since there are _2n_ subsets and for each subset we need to sum _n_ elements.

Let's improve it a bit. First optimization comes from the idea that we do not really need to calculate all the possible subsets and calculate all the sums afterward. Instead we can merge these two steps in a single operation and have a means to restore the set that produced the right sum. Having a set _A_ of elements _{a1, a2, a3 ... an}_ let's build a tree of all possible sums until we find a target sum _S_. Let's start with adding 0 to the tree and then adding each element of _A_ to all the elements in this tree.
<table style="width: 548px;">
<tbody>
<tr style="height: 24px;">
<td style="text-align: center; width: 38px; height: 24px;"><em><strong> </strong></em></td>
<td style="text-align: center; width: 52px; height: 24px;"><em><strong>a<sub>1</sub></strong></em></td>
<td style="text-align: center; width: 75px; height: 24px;"><em><strong>a<sub>2</sub></strong></em></td>
<td style="text-align: center; width: 114px; height: 24px;"><em><strong>a<sub>3</sub></strong></em></td>
<td style="text-align: center; width: 42px; height: 24px;"><em><strong>...</strong></em></td>
<td style="text-align: center; width: 191px; height: 24px;"><em><strong>a<sub>n</sub></strong></em></td>
</tr>
<tr style="height: 24px;">
<td style="text-align: center; width: 38px; height: 24px;"><em>0</em></td>
<td style="text-align: center; width: 52px; height: 24px;"><em>0+a<sub>1</sub></em></td>
<td style="text-align: center; width: 75px; height: 24px;"><em>0+a<sub>2</sub></em></td>
<td style="text-align: center; width: 114px; height: 24px;"><em>0+a<sub>3</sub></em></td>
<td style="text-align: center; width: 42px; height: 24px;"><em>...</em></td>
<td style="text-align: center; width: 191px; height: 24px;"><em>0+a<sub>n</sub></em></td>
</tr>
<tr style="height: 24px;">
<td style="text-align: center; width: 38px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 52px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 75px; height: 24px;"><em>(0+a<sub>1</sub>)+a<sub>2</sub></em></td>
<td style="text-align: center; width: 114px; height: 24px;"><em>(0+a<sub>1</sub>)+a<sub>3</sub></em></td>
<td style="text-align: center; width: 42px; height: 24px;"><em>... </em></td>
<td style="text-align: center; width: 191px; height: 24px;"><em>(0+a<sub>1</sub>)+a<sub>n</sub></em></td>
</tr>
<tr style="height: 24px;">
<td style="text-align: center; width: 38px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 52px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 75px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 114px; height: 24px;"><em>(0+a<sub>2</sub>)+a<sub>3</sub></em></td>
<td style="text-align: center; width: 42px; height: 24px;"><em>... </em></td>
<td style="text-align: center; width: 191px; height: 24px;"><em>(0+a<sub>2</sub>)+a<sub>n</sub></em></td>
</tr>
<tr style="height: 24px;">
<td style="text-align: center; width: 38px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 52px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 75px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 114px; height: 24px;"><em>((0+a<sub>1</sub>)+a<sub>2</sub>)+a<sub>3</sub></em></td>
<td style="text-align: center; width: 42px; height: 24px;"><em>...</em></td>
<td style="text-align: center; width: 191px; height: 24px;"><em>((0+a<sub>1</sub>)+a<sub>2</sub>)+a<sub>n</sub></em></td>
</tr>
<tr style="height: 24px;">
<td style="text-align: center; width: 38px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 52px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 75px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 114px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 42px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 191px; height: 24px;"><em>…</em></td>
</tr>
<tr style="height: 24px;">
<td style="text-align: center; width: 38px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 52px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 75px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 114px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 42px; height: 24px;"><em> </em></td>
<td style="text-align: center; width: 191px; height: 24px;"><em>(((0+a<sub>1</sub>)+a<sub>2</sub>)+a<sub>3</sub>)+…+a<sub>n</sub></em></td>
</tr>
</tbody>
</table>

As you can see now, no extra step is needed to sum each subset, instead each level of the tree uses sums from the previous tiers. So, there is 1 sum for the 1st element, 2 sums for the 2nd, 4 sums for the 3rd, 8 sums for the 4th, etc. This forms a geometrical progression with the total of $$\frac {1(1-2^n)}{1-2} = 2^n-1$$ elements. Now this method's complexity is _O(2n)_. 

Secondly, we can utilize the boundary condition. As we know that all the elements in the set are positive, it is obvious that adding a new element to the set only increases the current sum. So, if the current sum of the node is higher than the target sum this solution becomes rejected and is not used in further computations. This decreases the complexity to _O(2n) in the worst case_. Also we can benefit from sorting the set in the descending order, because this will help to reject more solutions in the early stage. Here is the code:
```csharp
public static List<int> FindRecursive(int[] set, int targetSum, int currentSum, int currentIndex)
{
	for (var i = currentIndex; i < set.Length; i++)
	{
		var newSum = currentSum + set[i];
		if (newSum > targetSum)
		{
			continue;
		}

		if (newSum == targetSum)
		{
			return new List<int> { set[i] };
		}

		var result = FindRecursive(set, targetSum, newSum, i + 1);
		if (result == null)
		{
			continue;
		}

		result.Add(set[i]);
		return result;
	}

	return null;
}

public static void Main(string[] args)
{
	const int sum = 47;
	var set = new[] { 100, 40, 5, 1, 1, 1, 1 };
	
	var result = FindRecursive(set, sum, 0, 0);
	Console.WriteLine(
		result?.Select(x => x.ToString()).Aggregate((x, y) => x + "," + y) ?? "null");
}
```

## Dynamic programming

This problem can be solved in pseudo-polynomial time using dynamic programming (a method for solving a complex problem by breaking it down into a collection of simpler subproblems). Define the boolean-valued function _Q(n, s)_ so that:

*   Q(n, 0) = true (return empty set)
*   Q(0, s) = false, if s > 0 (you can't pick a positive sum with an empty set)

Then all other cases are: _Q(n, s) = Q(n-1, s) || Q(n-1, s-aₙ)_. The only thing left is to fill the array of values of Q(i, s) for 1 ≤ i ≤ n using a simple recursion. The complexity of this method is _O(n*s)_ which is linear. 

Let's see how it works in the example: Given a set A {1, 2, 5, 7}, is there subset whose sum is 9?

1. Sum of 0 can be picked for any n with an empty set, the only sum that can be picked with an empty set is 0.
<table width="681">
<tbody>
<tr>
<td style="width: 33px;"></td>
<td style="text-align: center; width: 57px;">0</td>
<td style="text-align: center; width: 57px;">1</td>
<td style="text-align: center; width: 58px;">2</td>
<td style="text-align: center; width: 58px;">3</td>
<td style="text-align: center; width: 58px;">4</td>
<td style="text-align: center; width: 58px;">5</td>
<td style="text-align: center; width: 58px;">6</td>
<td style="text-align: center; width: 58px;">7</td>
<td style="text-align: center; width: 58px;">8</td>
<td style="text-align: center; width: 58px;">9</td>
</tr>
<tr>
<td style="width: 33px;">0</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">1</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
<tr>
<td style="width: 33px;">2</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
<tr>
<td style="width: 33px;">5</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
<tr>
<td style="width: 33px;">7</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
</tbody>
</table>

2. Adding 1 to the set we can now pick a sum of 1.
<table width="681">
<tbody>
<tr>
<td style="width: 33px;"></td>
<td style="text-align: center; width: 57px;">0</td>
<td style="text-align: center; width: 57px;">1</td>
<td style="text-align: center; width: 58px;">2</td>
<td style="text-align: center; width: 58px;">3</td>
<td style="text-align: center; width: 58px;">4</td>
<td style="text-align: center; width: 58px;">5</td>
<td style="text-align: center; width: 58px;">6</td>
<td style="text-align: center; width: 58px;">7</td>
<td style="text-align: center; width: 58px;">8</td>
<td style="text-align: center; width: 58px;">9</td>
</tr>
<tr>
<td style="width: 33px;">0</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">1</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">2</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
<tr>
<td style="width: 33px;">5</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
<tr>
<td style="width: 33px;">7</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
</tbody>
</table>

3. Adding 2 to the set we can now pick a sum of 2 and 3 and we already can pick 1.
<table width="681">
<tbody>
<tr>
<td style="width: 33px;"></td>
<td style="text-align: center; width: 57px;">0</td>
<td style="text-align: center; width: 57px;">1</td>
<td style="text-align: center; width: 58px;">2</td>
<td style="text-align: center; width: 58px;">3</td>
<td style="text-align: center; width: 58px;">4</td>
<td style="text-align: center; width: 58px;">5</td>
<td style="text-align: center; width: 58px;">6</td>
<td style="text-align: center; width: 58px;">7</td>
<td style="text-align: center; width: 58px;">8</td>
<td style="text-align: center; width: 58px;">9</td>
</tr>
<tr>
<td style="width: 33px;">0</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">1</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">2</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"> true</td>
<td style="text-align: center; width: 58px;"> true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;"> false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">5</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
<tr>
<td style="width: 33px;">7</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
<td style="text-align: center; width: 58px;"></td>
</tr>
</tbody>
</table>

4. Complete the table. Here is the answer shown in red.
<table width="681">
<tbody>
<tr>
<td style="width: 33px;"></td>
<td style="text-align: center; width: 57px;">0</td>
<td style="text-align: center; width: 57px;">1</td>
<td style="text-align: center; width: 58px;">2</td>
<td style="text-align: center; width: 58px;">3</td>
<td style="text-align: center; width: 58px;">4</td>
<td style="text-align: center; width: 58px;">5</td>
<td style="text-align: center; width: 58px;">6</td>
<td style="text-align: center; width: 58px;">7</td>
<td style="text-align: center; width: 58px;">8</td>
<td style="text-align: center; width: 58px;">9</td>
</tr>
<tr>
<td style="width: 33px;">0</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">1</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">2</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">5</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">7</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="width: 58px; text-align: center; background-color: #ed8585;">true</td>
</tr>
</tbody>
</table>

5. How to restore the set. We got the correct answer after adding 7 to the set, so let's remove it: 9-7=2. We got a sum of 2 after adding 2 to the set, so let's remove it. Now we are at the sum of 0, so the restoration is over. The track is shown in yellow.
<table width="681">
<tbody>
<tr>
<td style="width: 33px;"></td>
<td style="text-align: center; width: 57px;">0</td>
<td style="text-align: center; width: 57px;">1</td>
<td style="text-align: center; width: 58px;">2</td>
<td style="text-align: center; width: 58px;">3</td>
<td style="text-align: center; width: 58px;">4</td>
<td style="text-align: center; width: 58px;">5</td>
<td style="text-align: center; width: 58px;">6</td>
<td style="text-align: center; width: 58px;">7</td>
<td style="text-align: center; width: 58px;">8</td>
<td style="text-align: center; width: 58px;">9</td>
</tr>
<tr>
<td style="width: 33px;">0</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">1</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">2</td>
<td style="width: 57px; text-align: center; background-color: #f0e5a8;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="width: 58px; text-align: center; background-color: #f0e5a8;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">5</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="width: 58px; text-align: center; background-color: #f0e5a8;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
</tr>
<tr>
<td style="width: 33px;">7</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="text-align: center; width: 57px;">true</td>
<td style="width: 58px; text-align: center; background-color: #f0e5a8;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">false</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="text-align: center; width: 58px;">true</td>
<td style="width: 58px; text-align: center; background-color: #f0e5a8;">true</td>
</tr>
</tbody>
</table>

And finally, here is the code:
```csharp
public static List<int> FindDP(int[] set, int sum)
{
	var solution = new bool[set.Length + 1, sum + 1];
	for (var i = 0; i <= set.Length; i++)
	{
		solution[i, 0] = true;
	}

	for (var i = 1; i <= set.Length; i++)
	{
		for (var j = 1; j <= sum; j++)
		{
			solution[i, j] = solution[i - 1, j];

			if (!solution[i, j] && j >= set[i - 1])
			{
				solution[i, j] = solution[i, j] || solution[i - 1, j - set[i - 1]];
			}
		}

		if (!solution[i, sum])
		{
			continue;
		}

		var result = new List<int>();
		var q = sum;
		for (var p = i - 1; p >= 0; p--)
		{
			if (solution[p, q])
			{
				continue;
			}

			var s = set[p];
			result.Add(s);
			q -= s;
		}

		return result;
	}

	return null;
}
```

## PS

Solving this task I've started thinking about making a github repo for such algorithms, so that one can easily find an implementation of a needed algorithm in c# or simply use it without needing to reinvent the wheel.
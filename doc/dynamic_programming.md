# Dynamic Programming
Dynamic programming is mostly just a matter of taking a recursive algorithm
and finding the overlapping subproblems (that is, the repeated calls.) You
then cache those results for future recursive calls.

Some people call top-down dynamic programming "memorization" and only use
"dynamic programming" to refer to bottom-up work.

## Classic Example: Fibonacci
The recursive version
``` ruby
def fib(n)
  return 0 if n == 0
  return 1 if n == 1
  return fib(i - 1) + fib(i - 2)
end
```
### Top-Down Dynamic Approach
There are a lot repeated calls in the fib recursion. In fact, we should only
compute O(n) calls and rest should be cached. This is what memorization is
all about.

``` ruby
# Write a function, fibs(num) which returns the first n elements from
# the Fibonacci sequence, given n.
# Current time complexity is ~ 2.4^n power, exponentially
require 'benchmark'
def fibs(n)
  return [] if n <= 0
  return [1] if n == 1
  return [1, 1] if n == 2
  fibs(n-1) + [fibs(n-1).last + fibs(n-2).last]
end

$record = Hash.new
def fibs_dynamic(n)
  return [] if n <= 0
  return [1] if n == 1
  return [1, 1] if n == 2
  return $record[n] if $record[n]
  $record[n] = fibs_dynamic(n-1) + [fibs_dynamic(n-1).last + fibs_dynamic(n-2).last]
  $record[n]
end

puts Benchmark.measure { fibs_dynamic(22) }
puts Benchmark.measure { fibs(22)}
```

### Bottom-Up Dynamic Approach
Think about doing the same thing as the recursive memorized approach, but
in reverse. All we need is the first two base cases and we can construct
any Fibonacci number out of them.
``` ruby
def fibonacci(n)
  return 0 if n == 0
  a, b = 0, 1
  i = 2
  while (i < n)
    c = a + b
    a = b
    b = c
    i += 1
  end
  a + b
end
```

## LCS: Longest Common Subsequence

Write a function, longest_common_subsequence(str1, str2) that takes two
strings and returns the longest common subsequence.
This is known as the LCS problem. (NP-hard)

For example, we have "acbcefg" and "abcdaefg", the longest common subsequence
is abcfg. Notice that the letters do not need to appear consecutively, as long
as they are in the right order.

### Naive Approach
Generate all subsequences for a given string using recursion
``` ruby
def subseqs(str)
  return [str] if str.length == 1
  subseqs(str[0...str.length - 1]).map {|el| el + str[str.length - 1]} +
  subseqs(str[0...str.length - 1]) +
  [str[-1]]
end
# Example
# 123 => [1, 12, 2, 13, 123, 23, 3]
# Let's break it down
# [1, 12, 2] comes from seqs(12)
# [13, 123, 23] comes from "1" + "3", "12" + "3", "2" + "3", addng 3 to each element
# [3] comes from "123"[-1]
```
We generate subsequences for string 1 and string 2 and then compare them
``` ruby
def brute_force_lcs(str1, str2)
  subseqs1 = subseqs(str1)
  subseqs2 = subseqs(str2)
  longest_subseq = ""
  subseqs1.each do |sub1|
    subseqs2.each do |sub2|
      if sub1 == sub2 && sub1.length > longest_subseq.length
        longest_subseq = sub1
      end
    end
  end
  longest_subseq
end
```
This will run in exponential time due to the part where we generate all
subsequences for a given string by running recursions many many times.

### Dynamic Approach
We will use a matrix to perform memorization, using abcdaefg & acbcefg as
example.
* Create a str1.length + 1 by str2.length + 1 matrix, for this example,
it's 8 by 9 matrix.
* __i__ indicates str1[0...i], if i = 0, then str1[0...0] = "", if i = 1, then
str1[0...1] = "a".
* __j__ indicates str2[0...j], just like above
* Every cell in the matrix records length of the current common longest
subsequence
* For example, matrix[2][3] comapares __"ac"__ to __"abc"__ which gives us 2,
because the current lcs is __"ac"__
* Another example, matrix[3][5] compares __"acb"__ to __"abcda"__, once
again, it gives us 2, because the current lcs is __"ac"__ or __"ab"__

|   | _ | a | b | c | d | a | e | f | g |  
|---|---|---|---|---|---|---|---|---|---|
| _ | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| a | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 |
| c | 0 | 1 | 1 | 2 | 2 | 2 | 2 | 2 | 2 |
| b | 0 | 1 | 2 | 2 | 2 | 2 | 2 | 2 | 2 |
| c | 0 | 1 | 1 | 3 | 3 | 3 | 3 | 3 | 3 |
| e | 0 | 1 | 1 | 3 | 3 | 3 | 4 | 4 | 4 |
| f | 0 | 1 | 1 | 3 | 3 | 3 | 4 | 5 | 5 |
| g | 0 | 1 | 1 | 3 | 3 | 3 | 4 | 5 | 6 |

Answer can be found in the bottom-right cell, which is 6

#### The algorithm
1. Initialize the matrix and fill in 0 for __i__ ==  0 or __j__ == 0
2. Iterate through the matrix and do the following
```
  * If str1[i - 1] == str2[j - 1]
    * matrix[i][j] = matrix[i - 1][j - 1] + 1
  * Else
    * matrix[i][j] = Maximum of (matrix[i-1][j], matrix[i][j-1])
```

``` ruby
def dynamic_lcs(str1, str2)
  c = []
  # Initialize matrix and set default value to zero with empty string
  0.upto(str1.length) do |i|
    row = []
    0.upto(str2.length) do |j|
      row << {value: 0, subseq: ""}
    end
    c << row
  end
  # Iterate through the matrix and fill in values based on algorithm described above
  0.upto(str1.length) do |i|
    0.upto(str2.length) do |j|
      unless i == 0 || j == 0
        if str1[i-1] != nil && str2[j-1] != nil && str1[i-1] == str2[j-1]
          c[i][j][:value] = c[i-1][j-1][:value] + 1
          c[i][j][:subseq] = c[i-1][j-1][:subseq] + str1[i-1]
        elsif c[i][j-1][:value] > c[i-1][j][:value]
          c[i][j][:value] = c[i][j-1][:value]
          c[i][j][:subseq] = c[i][j-1][:subseq]
        else
          c[i][j][:value] = c[i-1][j][:value]
          c[i][j][:subseq] = c[i-1][j][:subseq]
        end
      end
    end
  end
  printMatrix(c)
  c[str1.length][str2.length]
end
```

Use an easier example to demonstrate the idea, let's say
```
str1 = "abcd"
str2 = "acgf"
We can tell that lcs of the two strings is "ac", just by looking at it
```
Then let's try to fill in the matrix

|   | _    | a     | b     | c                        | d      |
|---|------|-------|-------|--------------------------|--------|
| _ | 0 "" | 0 ""  | 0 ""  | 0 ""                     | 0 ""   |
| a | 0 "" | 1 "a" | 1 "a" | 1 "a"                    | 1 "a"  |
| c | 0 "" | 1 "a" | 1 "a" | 1 + 1 "a" + "c" = 2 "ac" | 2 "ac" |
| g | 0 "" | 1 "a" | 1 "a" | 2 "ac"                   | 2 "ac" |
| f | 0 "" | 1 "a" | 1 "a" | 2 "ac"                   | 2 "ac" |

Now we can confirm that LCS is "ac" and length is 2 
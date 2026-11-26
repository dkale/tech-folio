# Programming Logic Problems

This document is a cleaned and structured reference for common programming logic and interview problems, with sample C# solutions and discussion points.

## Table of Contents

- [Quick Arithmetic Notes](#quick-arithmetic-notes)
- [Complexity Basics](#complexity-basics)
- [String Problems](#string-problems)
  - [Reverse a String](#reverse-a-string)
  - [Palindrome Check](#palindrome-check)
  - [Pangram Check](#pangram-check)
  - [Anagram Check](#anagram-check)
  - [Least Occurring Character](#least-occurring-character)
  - [Rearrange Characters to Form a Palindrome](#rearrange-characters-to-form-a-palindrome)
  - [Count Vowels in a String](#count-vowels-in-a-string)
- [Number Problems](#number-problems)
  - [Reverse Digits in a Number](#reverse-digits-in-a-number)
  - [Fibonacci Series](#fibonacci-series)
  - [Factorial of a Number](#factorial-of-a-number)
  - [Convert Decimal to Binary](#convert-decimal-to-binary)
  - [Sum of Digits](#sum-of-digits)
- [Array and List Problems](#array-and-list-problems)
    - [Minimum Coin Reversals for Alternating Sequence](#minimum-coin-reversals-for-alternating-sequence)
  - [Find Min and Max from a List](#find-min-and-max-from-a-list)
  - [Bubble Sort](#bubble-sort)
  - [Move Even Numbers Left and Odd Numbers Right](#move-even-numbers-left-and-odd-numbers-right)
  - [Find the Second Largest Number](#find-the-second-largest-number)
  - [Find the Nth Largest Number](#find-the-nth-largest-number)
  - [Remove All Occurrences of a Number](#remove-all-occurrences-of-a-number)
    - [Find the Missing Number Between Two Arrays](#find-the-missing-number-between-two-arrays)
    - [Find Duplicates in an Array of 1 to 100](#find-duplicates-in-an-array-of-1-to-100)
- [General Logic Problems](#general-logic-problems)
    - [Swap Two Variables Without a Temporary Variable](#swap-two-variables-without-a-temporary-variable)
    - [Find the Peak Point from a List of Points](#find-the-peak-point-from-a-list-of-points)
- [Linked List Problems](#linked-list-problems)
    - [Swap the kth Node from Start and End in a Linked List](#swap-the-kth-node-from-start-and-end-in-a-linked-list)
- [Search and Combination Problems](#search-and-combination-problems)
  - [Find Prime Numbers](#find-prime-numbers)
    - [Count Valid Digital Clock Times from Four Digits](#count-valid-digital-clock-times-from-four-digits)
  - [Find Number Combinations That Add Up to a Target](#find-number-combinations-that-add-up-to-a-target)
- [Object and Collection Problems](#object-and-collection-problems)
  - [Neighboring Salaries in an Employee List](#neighboring-salaries-in-an-employee-list)
  - [Sort a Large Collection Efficiently](#sort-a-large-collection-efficiently)
  - [Sort a Collection Without LINQ](#sort-a-collection-without-linq)
- [Formatting and Validation Problems](#formatting-and-validation-problems)
  - [Remove and Reinstate Dashes in an ID](#remove-and-reinstate-dashes-in-an-id)
- [Design and Language Questions](#design-and-language-questions)
  - [Design a Simple Dependency Injection Container](#design-a-simple-dependency-injection-container)
  - [Generics and a Stack Example](#generics-and-a-stack-example)

## Quick Arithmetic Notes

Useful reminders for integer and decimal math:

- `1234 / 10 = 123`
- `1234 % 10 = 4`
- `1234 / 2 = 617`
- `13m / 2 = 6.5`
- `(decimal)13 / 2 = 6.5`

## Complexity Basics

Time complexity and space complexity describe how an algorithm scales as input gets bigger.

### Time Complexity

Time complexity tells us how the running time grows when the number of inputs grows.

Simple idea:

- If input size doubles, how much extra work does your code do?

Analogy:

- Searching for a name in a notebook.
- Time complexity is how many pages you may need to check.

### Space Complexity

Space complexity tells us how much extra memory the algorithm needs as input size grows.

Simple idea:

- Does your code use just a few variables, or does it allocate another big array/list?

Analogy:

- Space complexity is how many extra sticky notes you need while searching the notebook.

### Common Examples

1. Check whether a value exists in an unsorted list
    - Method: scan one by one.
    - Time: `O(n)`
    - Space: `O(1)`

2. Binary search in a sorted list
    - Method: repeatedly check the middle and cut the range in half.
    - Time: `O(log n)`
    - Space: `O(1)` (iterative version)

3. Sum all numbers in a list
    - Method: one full pass.
    - Time: `O(n)`
    - Space: `O(1)`

4. Copy all elements to a new list
    - Method: create another list of the same size.
    - Time: `O(n)`
    - Space: `O(n)`

5. Compare every pair using nested loops
    - Method: loop inside another loop over the same list.
    - Time: `O(n^2)`
    - Space: usually `O(1)`

### Quick Rule of Thumb

- `O(1)`: constant work
- `O(log n)`: grows slowly and is very efficient
- `O(n)`: linear growth, often acceptable
- `O(n log n)`: common for efficient sorting
- `O(n^2)` or higher: can become slow for large inputs

Why this matters:

- Small inputs can make most algorithms look fast.
- Large inputs reveal whether the algorithm scales well.
- Complexity helps you choose better solutions early.

## String Problems

### Reverse a String

#### Iterative

```csharp
static string ReverseString_Iterative(string inputStr)
{
    if (string.IsNullOrEmpty(inputStr))
        return inputStr;

    char[] strArray = inputStr.ToCharArray();

    for (int i = 0; i < strArray.Length / 2; i++)
    {
        char temp = strArray[i];
        strArray[i] = strArray[strArray.Length - 1 - i];
        strArray[strArray.Length - 1 - i] = temp;
    }

    return new string(strArray);
}
```

#### Recursive

```csharp
static string ReverseString_Recursive(string input)
{
    if (input.Length < 1)
        return input;

    return input[input.Length - 1] +
           ReverseString_Recursive(input.Substring(0, input.Length - 1));
}
```

### Palindrome Check

#### Iterative

```csharp
static bool IsPalindrome_Iterative(string input)
{
    if (string.IsNullOrEmpty(input))
        return false;

    char[] strArray = input.ToCharArray();

    for (int i = 0; i < strArray.Length / 2; i++)
    {
        if (strArray[i] != strArray[strArray.Length - 1 - i])
            return false;
    }

    return true;
}
```

#### Two-pointer version

```csharp
static bool IsPalindrome(string input)
{
    input = input.ToLower();

    int left = 0;
    int right = input.Length - 1;

    while (left < right)
    {
        if (input[left] != input[right])
            return false;

        left++;
        right--;
    }

    return true;
}
```

#### Recursive

```csharp
static bool IsPalindrome_Recursive(string input, int index)
{
    if (index > input.Length / 2 - 1)
        return true;

    if (input[index] == input[input.Length - 1 - index])
        return IsPalindrome_Recursive(input, index + 1);

    return false;
}
```

### Pangram Check

```csharp
static bool IsPangram(string input)
{
    return input.ToLower()
                .Where(c => char.IsLetter(c))
                .GroupBy(c => c)
                .Count() == 26;
}
```

### Anagram Check

```csharp
static bool IsAnagram(string str1, string str2)
{
    if (string.IsNullOrEmpty(str1) || string.IsNullOrEmpty(str2))
        return false;

    var str1Dictionary = CalculateLetterOccurrence(str1);
    var str2Dictionary = CalculateLetterOccurrence(str2);

    if (str1Dictionary.Count != str2Dictionary.Count)
        return false;

    foreach (var item in str1Dictionary)
    {
        if (!str2Dictionary.ContainsKey(item.Key))
            return false;

        if (str2Dictionary[item.Key] != item.Value)
            return false;
    }

    return true;
}

static Dictionary<char, int> CalculateLetterOccurrence(string input)
{
    var letterOccurrence = input.ToLower()
                                .Where(c => char.IsLetter(c))
                                .GroupBy(c => c);

    var letterDictionary = new Dictionary<char, int>();

    foreach (var item in letterOccurrence)
        letterDictionary.Add(item.Key, item.Count());

    return letterDictionary;
}
```

### Least Occurring Character

```csharp
static char GetLeastOccurring(string inputString)
{
    return inputString.ToCharArray()
                      .Where(c => char.IsLetter(c))
                      .GroupBy(c => c)
                      .OrderBy(c => c.Count())
                      .First()
                      .Key;
}
```

Dictionary-based alternative:

```csharp
static char GetLeastOccurring_Dictionary(string inputString)
{
    char[] someArray = inputString.ToCharArray();
    Dictionary<char, int> charDictionary = new Dictionary<char, int>();

    foreach (char c in someArray)
    {
        if (charDictionary.ContainsKey(c))
            charDictionary[c] += 1;
        else
            charDictionary.Add(c, 1);
    }

    char minChar = '\0';
    int minCount = int.MaxValue;

    foreach (KeyValuePair<char, int> kv in charDictionary)
    {
        if (kv.Value < minCount)
        {
            minChar = kv.Key;
            minCount = kv.Value;
        }
    }

    return minChar;
}
```

### Rearrange Characters to Form a Palindrome

Example:

- `ETTESTS` -> `true`
- `ETRTESTS` -> `false`

```csharp
static bool CanFormPalindrome(string str)
{
    Dictionary<char, int> charCount = new Dictionary<char, int>();

    foreach (char c in str)
    {
        if (charCount.ContainsKey(c))
            charCount[c]++;
        else
            charCount[c] = 1;
    }

    int oddCount = 0;

    foreach (int count in charCount.Values)
    {
        if (count % 2 != 0)
            oddCount++;
    }

    return oddCount <= 1;
}
```

### Count Vowels in a String

Prompt guidance:

- The candidate should make reasonable assumptions.
- Follow-up questions should be avoided unless they unblock the implementation.
- Case sensitivity is an easy detail to miss.

Basic version:

```csharp
public int CountVowels(string input)
{
    int count = 0;

    foreach (char c in input)
    {
        if ("aeiouAEIOU".Contains(c))
            count++;
    }

    return count;
}
```

Possible follow-up:

How would you optimize `CountVowels` for better performance?

Common discussion points:

- Use a `HashSet<char>` for constant-time lookup.
- Normalize casing once with `ToLowerInvariant()`.
- Prefer a simple indexed `for` loop for very hot paths.
- Consider parallelism only when the input size and runtime context justify it.

Optimized version:

```csharp
public int CountVowels_Optimized(string input)
{
    HashSet<char> vowels = new HashSet<char> { 'a', 'e', 'i', 'o', 'u' };

    int count = 0;
    string lowerInput = input.ToLowerInvariant();

    for (int i = 0; i < lowerInput.Length; i++)
    {
        if (vowels.Contains(lowerInput[i]))
            count++;
    }

    return count;
}
```

Why use a `HashSet<char>`?

- It provides average `O(1)` lookup time.
- It stores unique vowels without duplicates.
- It keeps the intent clear and direct.

## Number Problems

### Reverse Digits in a Number

#### Iterative

```csharp
static int NumberReverse_Iterative(int number)
{
    if (number == 0)
        return number;

    int result = 0;

    while (number > 0)
    {
        result = number % 10 + result * 10;
        number = number / 10;
    }

    return result;
}
```

#### Recursive

```csharp
static int result = 0;

static void NumberReverse_Recursive(int number)
{
    if (number <= 0)
        return;

    result = number % 10 + result * 10;
    NumberReverse_Recursive(number / 10);
}
```

### Fibonacci Series

#### Iterative

```csharp
static void Fibonacci_Iterative(int count)
{
    int num1 = 1, num2 = 1;

    for (int i = 1; i <= count; i++)
    {
        int temp = num2;
        num2 = num2 + num1;
        num1 = temp;

        Console.WriteLine(num2);
    }
}
```

#### Recursive

```csharp
static int i = 1;

static void Fibonacci_Recursive(int count, int a = 1, int b = 1)
{
    Console.WriteLine(a);

    if (i < count)
    {
        i++;
        Fibonacci_Recursive(count, b, a + b);
    }
}
```

### Factorial of a Number

#### Iterative

```csharp
static int Factorial_Iterative(int number)
{
    if (number == 0)
        return number;

    int result = 1;

    while (number > 1)
        result = result * number--;

    return result;
}
```

#### Recursive

```csharp
static int Factorial_Recursive(int number)
{
    if (number <= 1)
        return number;

    return number * Factorial_Recursive(--number);
}
```

### Convert Decimal to Binary

#### Iterative

```csharp
static void DecimalToBinary_Iterative(int decimalNumber)
{
    if (decimalNumber == 0)
        return;

    int[] binaryArray = new int[16];
    int i = 15, quotient = decimalNumber;

    while (i >= 0)
    {
        quotient = decimalNumber / 2;
        binaryArray[i] = quotient == 1 ? 0 : decimalNumber % 2;
        decimalNumber = quotient;
        i--;
    }
}
```

#### Recursive

```csharp
static string DecimalToBinary_Recursive(int number)
{
    if (number == 1)
        return "1";

    return DecimalToBinary_Recursive(number / 2) + number % 2;
}
```

### Sum of Digits

#### Iterative

```csharp
static int SumOfDigits(int number)
{
    int sum = 0;

    while (number > 0)
    {
        sum += number % 10;
        number = number / 10;
    }

    return sum;
}
```

#### Recursive

```csharp
static int SumOfDigits_Recursive(int number)
{
    if (number == 0)
        return 0;

    return SumOfDigits_Recursive(number / 10) + number % 10;
}
```

## Array and List Problems

### Minimum Coin Reversals for Alternating Sequence

Problem statement:

There are N coins, each showing either heads or tails. We want all coins to form a sequence of alternating heads and tails. Find the minimum number of coins that must be reversed.

Write a function:

```java
class Solution {
    public int solution(int[] A);
}
```

Given an array A of N integers where:

- `0` represents heads
- `1` represents tails

Consecutive elements in A represent consecutive coins.

Examples:

1. `A = [1, 0, 1, 0, 1, 1]` -> `1`
   - Reverse the sixth coin to get `[1, 0, 1, 0, 1, 0]`.
2. `A = [1, 1, 0, 1, 1]` -> `2`
   - Reverse the first and fifth coins to get `[0, 1, 0, 1, 0]`.
3. `A = [0, 1, 0]` -> `0`
   - Already alternating.
4. `A = [0, 1, 1, 0]` -> `2`
   - Reverse the first and second coins to get `[1, 0, 1, 0]`.

Assume:

- `N` is in the range `[1..100]`
- Each element of `A` is either `0` or `1`

Approach:

There are only two possible alternating patterns for any length N:

- Start with `0`: `0, 1, 0, 1, ...`
- Start with `1`: `1, 0, 1, 0, ...`

Count mismatches with both patterns and return the smaller count.

```java
class Solution {
    public int solution(int[] A) {
        int flipsStartWith0 = 0;
        int flipsStartWith1 = 0;

        for (int i = 0; i < A.length; i++) {
            int expectedIfStart0 = i % 2;
            int expectedIfStart1 = 1 - (i % 2);

            if (A[i] != expectedIfStart0) {
                flipsStartWith0++;
            }

            if (A[i] != expectedIfStart1) {
                flipsStartWith1++;
            }
        }

        return Math.min(flipsStartWith0, flipsStartWith1);
    }
}
```

### Find Min and Max from a List

```csharp
List<int> numberList = new List<int>() { 23, 54, 12, 78, 67, 43, 91, 35 };
int min = int.MaxValue, max = int.MinValue;

foreach (int num in numberList)
{
    if (num < min) min = num;
    if (num > max) max = num;
}
```

### Bubble Sort

```csharp
static List<int> BubbleSort(List<int> unsortedList)
{
    int count = unsortedList.Count;

    for (int i = 0; i < count; i++)
    {
        for (int j = 0; j < count - 1; j++)
        {
            if (unsortedList[j] > unsortedList[j + 1])
            {
                int temp = unsortedList[j];
                unsortedList[j] = unsortedList[j + 1];
                unsortedList[j + 1] = temp;
            }
        }
    }

    return unsortedList;
}
```

### Move Even Numbers Left and Odd Numbers Right

Problem statement: given an array of positive integers, move the even numbers to the left and the odd numbers to the right without using an extra array.

```csharp
static void EvenOddArrayRearrange(int[] inputArray)
{
    int i = 0;

    for (int j = 0; j < inputArray.Length; j++)
    {
        if (inputArray[j] % 2 == 0)
        {
            if (j == i)
                continue;

            int temp = inputArray[j];
            inputArray[j] = inputArray[i];
            inputArray[i] = temp;
            i++;
        }
    }
}
```

Interview variant:

```csharp
public static void MoveEvenToLeft(int[] array)
{
    int i = 0;

    for (int j = 0; j < array.Length; j++)
    {
        if (array[j] % 2 == 0)
        {
            Swap(array, i, j);
            i++;
        }
    }
}

static void Swap(int[] arr, int i, int j)
{
    int temp = arr[i];
    arr[i] = arr[j];
    arr[j] = temp;
}
```

### Find the Second Largest Number

```csharp
static int SecondLargestNumber(int[] array)
{
    int first = 0, second = 0;

    foreach (int num in array)
    {
        if (second > num && num > first)
        {
            second = first;
            first = num;
        }
        else
        {
            second = num;
        }
    }

    return second;
}
```

List-based variant:

```csharp
public static int GetSecondLargest(List<int> lst)
{
    int first = int.MinValue;
    int second = int.MinValue;

    foreach (int i in lst)
    {
        if (i > first && i > second)
        {
            second = first;
            first = i;
        }
        else if (i > second)
        {
            second = i;
        }
    }

    return second;
}
```

### Find the Nth Largest Number

```csharp
static int FindNthLargestNumber(int[] array, int nVal)
{
    return array.OrderByDescending(n => n)
                .Take(nVal)
                .ElementAt(nVal - 1);
}
```

### Remove All Occurrences of a Number

Interview constraint: solve without `Except`, `RemoveAll`, or allocating a new array.

Existing sample for reference:

```csharp
static int[] RemoveDigit(int[] array, int intToRemove)
{
    return array.Except(new int[] { intToRemove }).ToArray();

    var list = new List<int>(array);
    list.RemoveAll(n => n == intToRemove);
    return list.ToArray();
}
```

### Find the Missing Number Between Two Arrays

Question: given two arrays such as `1, 2, 3, 4, 5` and `2, 3, 1, 0, 5`, find which number from the first array is not present in the second array.

Approach:

Compare the elements of the first array against the second array and identify the value that does not exist in the second array.

For this example, the missing number from the first array is `4`.

### Find Duplicates in an Array of 1 to 100

Question: in an array containing values from `1` to `100`, many numbers are duplicated. How do you find them?

Approach:

Use a hash set or frequency array:

1. Traverse the array.
2. Track each number as it appears.
3. If a number is seen again, report it as a duplicate.

## General Logic Problems

### Swap Two Variables Without a Temporary Variable

```csharp
a = a + b;
b = a - b;
a = a - b;
```

Example if `a = 10` and `b = 20`:

```text
a = a + b   => a = 30, b = 20
b = a - b   => a = 30, b = 10
a = a - b   => a = 20, b = 10
```

### Find the Peak Point from a List of Points

Question: given a list of `Point(x, y)` values in a graph, find the peak point.

Answer: compare the `y` coordinates and select the point with the highest `y` value.

## Linked List Problems

### Swap the kth Node from Start and End in a Linked List

Question: swap the kth element from the beginning and the kth element from the end of a linked list.

Approach:

1. Find the length of the linked list.
2. Locate the kth node from the beginning.
3. Locate the kth node from the end using `length - k + 1`.
4. Swap the node values or adjust the node links.

## Search and Combination Problems

### Find Prime Numbers

Reference links:

- http://mathforum.org/isaac/problems/prime2.html
- http://www.dotnetperls.com/prime
- http://stackoverflow.com/questions/15743192/check-if-number-is-prime-number

### Count Valid Digital Clock Times from Four Digits

Problem statement:

Given four digits, count how many valid times can be displayed on a digital clock in 24-hour format using those digits exactly once. The earliest time is `00:00` and the latest time is `23:59`.

Examples:

- `1, 8, 3, 2` -> `6`
- `2, 3, 3, 2` -> `3`
- `6, 2, 4, 7` -> `0`

Approach:

Try every permutation of the four digits and interpret each arrangement as `HH:MM`.

- The first two digits form the hour.
- The last two digits form the minutes.
- A time is valid when `hour < 24` and `minute < 60`.
- Use a `HashSet<string>` so repeated digits do not cause duplicate times to be counted twice.

```csharp
public int CountValidTimes(int a, int b, int c, int d)
{
    int[] digits = { a, b, c, d };
    HashSet<string> validTimes = new HashSet<string>();

    for (int i = 0; i < 4; i++)
    {
        for (int j = 0; j < 4; j++)
        {
            if (j == i)
                continue;

            for (int k = 0; k < 4; k++)
            {
                if (k == i || k == j)
                    continue;

                for (int l = 0; l < 4; l++)
                {
                    if (l == i || l == j || l == k)
                        continue;

                    int hour = digits[i] * 10 + digits[j];
                    int minute = digits[k] * 10 + digits[l];

                    if (hour < 24 && minute < 60)
                    {
                        validTimes.Add(hour.ToString("D2") + ":" + minute.ToString("D2"));
                    }
                }
            }
        }
    }

    return validTimes.Count;
}
```

Step-by-step example for `1, 8, 3, 2`:

Try each ordering of the digits as `H1 H2 : M1 M2`.

- `1 2 3 8` -> `12:38`, valid
- `1 3 2 8` -> `13:28`, valid
- `1 8 2 3` -> `18:23`, valid
- `1 8 3 2` -> `18:32`, valid
- `2 1 3 8` -> `21:38`, valid
- `2 3 1 8` -> `23:18`, valid
- `3 1 2 8` -> `31:28`, invalid because the hour is greater than `23`
- `2 8 1 3` -> `28:13`, invalid because the hour is greater than `23`

After checking all `4! = 24` permutations, the valid times are:

- `12:38`
- `13:28`
- `18:23`
- `18:32`
- `21:38`
- `23:18`

So the total count is `6`.

### Find Number Combinations That Add Up to a Target

```csharp
public static IEnumerable<string> GetCombinations(int[] set, int sum, string values)
{
    for (int i = 0; i < set.Length; i++)
    {
        int left = sum - set[i];
        string vals = set[i] + "," + values;

        if (left == 0)
        {
            yield return vals;
        }
        else
        {
            int[] possible = set.Take(i).Where(n => n <= sum).ToArray();

            if (possible.Length > 0)
            {
                foreach (string s in GetCombinations(possible, left, vals))
                    yield return s;
            }
        }
    }
}
```

Usage:

```csharp
int[] set = { 1, 2, 3, 4, 5, 6, 7, 8, 9, 10 };

foreach (string s in GetCombinations(set, 18, ""))
{
    Console.WriteLine(s);
}
```

## Object and Collection Problems

### Neighboring Salaries in an Employee List

Problem statement: for a given employee list, find the employee with salary immediately less than the target salary and the employee with salary immediately greater than it.

Example: if the target salary is `3000`, then the expected neighboring salaries are `2000` and `4000`.

```csharp
List<Employee> employeeList = new List<Employee>()
{
    new Employee() { EmployeeName = "Satish", Salary = 1000 },
    new Employee() { EmployeeName = "Chintu", Salary = 3000 },
    new Employee() { EmployeeName = "Sudeep", Salary = 2000 },
    new Employee() { EmployeeName = "Shripad", Salary = 5000 },
    new Employee() { EmployeeName = "Dayanand", Salary = 4000 }
};

Employee minEmp, maxEmp;
int findSalary = 3000;

foreach (Employee emp in employeeList)
{
    if (emp.Salary < findSalary)
        minEmp = emp;
    else if (emp.Salary > findSalary)
        maxEmp = emp;
}
```

### Sort a Large Collection Efficiently

Problem statement: efficiently sort or search a large in-memory collection by a property without using LINQ for the core lookup path.

#### Solution 1: dictionary of lists

```csharp
List<Order> orderCollection = new List<Order>();

for (int i = 0; i < 800000; i++)
{
    orderCollection.Add(new Order()
    {
        OrderID = i < 100000 ? 1 :
                  i < 200000 ? 2 :
                  i < 300000 ? 3 :
                  i < 400000 ? 4 :
                  i < 500000 ? 5 :
                  i < 600000 ? 6 :
                  i < 700000 ? 7 : 8,
        Price = i + 100,
        OrderTitle = "Chromecast" + i
    });
}

var orderIndexDictionary = new Dictionary<int, List<Order>>();

foreach (Order item in orderCollection)
{
    List<Order> allOrders;

    if (!orderIndexDictionary.TryGetValue(item.OrderID, out allOrders))
    {
        allOrders = new List<Order>();
        orderIndexDictionary.Add(item.OrderID, allOrders);
    }

    allOrders.Add(item);
}

var filteredItems = from items in orderIndexDictionary[8]
                    where items.Price < 705000
                    select items;

var result = filteredItems.ToList();
```

#### Solution 2: multiple indexes over an in-memory collection

Reference:

- http://codereview.stackexchange.com/questions/40811/multiple-indexes-over-an-in-memory-collection-for-faster-search

```csharp
public class FastCollection<T> : IEnumerable<T>
{
    private IList<T> _items;
    private IList<Expression<Func<T, object>>> _lookups;
    private Dictionary<string, ILookup<object, T>> _indexes;

    public FastCollection(IList<T> data)
    {
        _items = data;
        _lookups = new List<Expression<Func<T, object>>>();
        _indexes = new Dictionary<string, ILookup<object, T>>();
    }

    public void AddIndex(Expression<Func<T, object>> property)
    {
        _lookups.Add(property);
        _indexes.Add(property.ToString(), _items.ToLookup(property.Compile()));
    }

    public void Add(T item)
    {
        _items.Add(item);
        RebuildIndexes();
    }

    public void Remove(T item)
    {
        _items.Remove(item);
        RebuildIndexes();
    }

    public void RebuildIndexes()
    {
        if (_lookups.Count > 0)
        {
            _indexes = new Dictionary<string, ILookup<object, T>>();

            foreach (var lookup in _lookups)
                _indexes.Add(lookup.ToString(), _items.ToLookup(lookup.Compile()));
        }
    }

    public IEnumerable<T> FindValue<TProperty>(Expression<Func<T, TProperty>> property, TProperty value)
    {
        var key = property.ToString();

        if (_indexes.ContainsKey(key))
            return _indexes[key][value];

        var compiled = property.Compile();
        return _items.Where(x => compiled(x).Equals(value));
    }

    public IEnumerator<T> GetEnumerator()
    {
        return _items.GetEnumerator();
    }

    System.Collections.IEnumerator System.Collections.IEnumerable.GetEnumerator()
    {
        return GetEnumerator();
    }
}
```

### Sort a Collection Without LINQ

```csharp
orderList.Sort(
    delegate(Order p1, Order p2)
    {
        int compareDate = p1.Date.CompareTo(p2.Date);

        if (compareDate == 0)
            return p2.OrderID.CompareTo(p1.OrderID);

        return compareDate;
    });
```

## Formatting and Validation Problems

### Remove and Reinstate Dashes in an ID

Problem statement:

Consider the string `ABC-150225-8493`.

- The string begins with 1 to 3 letters.
- Then a dash.
- Then exactly 6 digits.
- Then another dash.
- Then exactly 4 digits.

Write one function to remove the dashes and another to reinstate them.

```csharp
public string ReinstateDashes(string inputID)
{
    if (string.IsNullOrEmpty(inputID))
        throw new NullReferenceException("Input is either null or empty");

    bool isPatternMatch = Regex.IsMatch(inputID, "");

    // char[] idArray = inputID.ToCharArray();
    // char[] newArray = new char[idArray.Length + 2];
    // int index = 0;
    // int count = 10;
    // foreach (char c in idArray)
    // {
    //     if (char.IsLetter(c))
    //     {
    //         newArray[index++] = c;
    //         continue;
    //     }
    //
    //     if (count == 10 || count == 6)
    //         newArray[index] = '-';
    //
    //     if (char.IsNumber(c))
    //     {
    //         newArray[index] = c;
    //         count--;
    //     }
    //
    //     index++;
    // }
}
```

## Design and Language Questions

### Design a Simple Dependency Injection Container

Prompt: if you were to design a dependency injection container of your own, how would you go about it? Give a simple code example.

#### Delegate-based registration

```csharp
using System;
using System.Collections.Generic;

public class DIContainer
{
    private readonly Dictionary<Type, Func<object>> _dependencies;

    public DIContainer()
    {
        _dependencies = new Dictionary<Type, Func<object>>();
    }

    public void Register<T>(Func<T> resolver)
    {
        _dependencies[typeof(T)] = () => resolver();
    }

    public T Resolve<T>()
    {
        if (_dependencies.TryGetValue(typeof(T), out var resolver))
            return (T)resolver();

        throw new ArgumentException($"No registration found for type {typeof(T)}");
    }
}

public interface ILogger
{
    void Log(string message);
}

public class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine($"[LOG] {message}");
    }
}

public class ExampleService
{
    private readonly ILogger _logger;

    public ExampleService(ILogger logger)
    {
        _logger = logger;
    }

    public void DoSomething()
    {
        _logger.Log("Doing something...");
    }
}

class Program
{
    static void Main(string[] args)
    {
        var container = new DIContainer();
        container.Register(() => new ConsoleLogger());
        container.Register(() => new ExampleService(container.Resolve<ILogger>()));

        var exampleService = container.Resolve<ExampleService>();
        exampleService.DoSomething();
    }
}
```

#### Reflection-based registration

```csharp
using System;
using System.Collections.Generic;

public class DIContainer
{
    private readonly Dictionary<Type, Type> _dependencies;

    public DIContainer()
    {
        _dependencies = new Dictionary<Type, Type>();
    }

    public void Register<TInterface, TImplementation>() where TImplementation : TInterface
    {
        _dependencies[typeof(TInterface)] = typeof(TImplementation);
    }

    public TInterface Resolve<TInterface>()
    {
        if (_dependencies.TryGetValue(typeof(TInterface), out var implementationType))
            return (TInterface)Activator.CreateInstance(implementationType);

        throw new ArgumentException($"Type {typeof(TInterface).Name} is not registered.");
    }
}
```

Usage:

```csharp
public interface IService
{
    void DoSomething();
}

public class MyService : IService
{
    public void DoSomething()
    {
        Console.WriteLine("MyService is doing something.");
    }
}

class Program
{
    static void Main(string[] args)
    {
        var container = new DIContainer();
        container.Register<IService, MyService>();

        var service = container.Resolve<IService>();
        service.DoSomething();
    }
}
```

### Generics and a Stack Example

Prompt: do you understand generics? What problem do they solve? What are their benefits? Write a simple program to create a number stack, then convert it to a generic stack.

#### Number stack

```csharp
using System;

public class NumberStack
{
    private int[] items;
    private int top;
    private const int DefaultCapacity = 10;

    public NumberStack()
    {
        items = new int[DefaultCapacity];
        top = -1;
    }

    public void Push(int number)
    {
        if (top == items.Length - 1)
            Array.Resize(ref items, items.Length * 2);

        top++;
        items[top] = number;
    }

    public int Pop()
    {
        if (IsEmpty())
            throw new InvalidOperationException("The stack is empty.");

        int result = items[top];
        top--;
        return result;
    }

    public int Peek()
    {
        if (IsEmpty())
            throw new InvalidOperationException("The stack is empty.");

        return items[top];
    }

    public bool IsEmpty()
    {
        return top == -1;
    }

    public void Clear()
    {
        top = -1;
    }
}
```

#### Generic stack implementation

```csharp
using System;

public class GenericStack<T>
{
    private T[] items;
    private int top;
    private const int DefaultCapacity = 10;

    public GenericStack()
    {
        items = new T[DefaultCapacity];
        top = -1;
    }

    public void Push(T item)
    {
        if (top == items.Length - 1)
            Array.Resize(ref items, items.Length * 2);

        top++;
        items[top] = item;
    }

    public T Pop()
    {
        if (IsEmpty())
            throw new InvalidOperationException("The stack is empty.");

        T result = items[top];
        top--;
        return result;
    }

    public T Peek()
    {
        if (IsEmpty())
            throw new InvalidOperationException("The stack is empty.");

        return items[top];
    }

    public bool IsEmpty()
    {
        return top == -1;
    }

    public void Clear()
    {
        top = -1;
    }
}
```

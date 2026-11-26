# ✔ Recursion

## Table of Contents
  - [Recursion](#recursion)
### Recursion

Recursion is an important programming technique that causes a function to call itself. One example is the calculation of factorials. The factorial of n, an integer greater than 0, is the product of all the integers in the range from 1 to n.
(Exercise: Give a try to implement a recursive function for finding factorial of a number)

Every recursive method sequence must be somehow terminated. Often the first part of the recursive method will have a branch that tests for a condition being met. In this way, the recursive method continues until the result is attained.

The call stack has six method stack frames with the Recursive method signature in them.

Uses
Recursive methods and approaches are widely used in compiler technologies, and also for algorithms that solve puzzles. Recursion allows you to implement a brute force search of all possibilities and all possibilities after that.

The following paragraph is a function, defined in words, that calculates a factorial.
“If the number is less than zero, reject it. If it is not an integer, reject it. If the number is zero, its factorial is one. If the number is larger than zero, multiply it by the factorial of the next lesser number.”

To calculate the factorial of any number that is larger than zero, you must calculate the factorial of at least one other number. The function must call itself for the next smaller number before it can execute on the current number. This is an example of recursion.

Recursion and iteration (looping) are strongly related - a function can return the same results either with recursion or iteration. Usually a particular computation will lend itself to one technique or the other, and you simply choose the most natural or preferable approach.

Despite the usefulness of recursion, you can easily create a recursive function that never returns a result and cannot reach an endpoint. Such a recursion causes the computer to execute an infinite loop.

Another problem that can occur with recursion is that a recursive function can use all available resources (such as system memory and stack space). Each time a recursive function calls itself (or calls another function that calls the original function), it uses some resources. These resources are freed when the recursive function exits, but a function that has too many levels of recursion may use all the available resources. When this happens, an exception is thrown.

Thus, it is important to design recursive functions with care. If you suspect any change of an excessive (or infinite) recursion, design the function to count the number of times it calls itself and set a limit on the number of calls. If the function calls itself more times than the threshold, the function can quit automatically. The optimum maximum number of iterations depends on the recursive function.

[↑ Back to Top](#table-of-contents)

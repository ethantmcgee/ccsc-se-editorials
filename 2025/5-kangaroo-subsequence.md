# Kangaroo Subsequence

This Java solution determines if one word (string1) is a kangaroo word for another word (string2) by checking if string2 is a subsequence of string1.

The program uses a greedy search approach implemented within the kangaroo method.
Input Handling: Reads the total number of test cases (count). It then enters a loop to process each pair of strings.

Case Conversion: For each pair, it reads the two strings and immediately converts them to uppercase using .toUpperCase(), fulfilling a key requirement of the problem.
Output Formatting: Calls the kangaroo method to perform the check. It then prints the complete, formatted result, including the strings and the returned "is" or "is not" phrase.

kangaroo(String s1, String s2) Method

1. Initialization
  1. test: A boolean initialized to true, assuming success until proven otherwise.
  2. index: Will store the current location of the character.
  3. last: Stores the index immediately before where the search for the next character must begin. 
2. Subsequence Search Loop
  1. Find Character: s1.indexOf(s2.charAt(i), ++last) is the crucial line.
  2. Check for Order and Existence
  3. Failure Condition
3. Return Value: Returns the string "is" if the test is true, and "is not" otherwise.

# Solution

```java
import java.util.*;
public class Kangaroo
{
        public static void main(String[] args)
        {
          Scanner stdin = new Scanner(System.in);
          int count = stdin.nextInt();
          stdin.nextLine();
          String s1, s2;
          for (int i = 1;  i <= count;   i++)
          {
              s1 = stdin.nextLine().toUpperCase();
              s2 = stdin.nextLine().toUpperCase();
              System.out.print(s1 + " ");
              System.out.print(kangaroo(s1, s2));
              System.out.println(" a kangaroo word for " + s2);
        }
        }

        public static String kangaroo(String s1, String s2)
        {
                boolean test = true;

                int index = -1;
                int last = -1;

                for (int i = 0; i < s2.length();  i++)
                {
                        index = s1.indexOf(s2.charAt(i), ++last);
                        if (last <= index)
                                last = index;
                        else
                        {
                                test = false;
                                break;
                        }
                }

                return test ? "is" : "is not";
        }
}
```
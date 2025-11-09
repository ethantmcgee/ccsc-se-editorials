# Diabolical

This problem asks you to determine if a given square matrix is a diabolical square, a magic square, or not magic. This involves a series of checks based on the sums of rows, columns, diagonals, and quadrants.  The core idea is to calculate various sums and compare them.

1. Read Input and Initialization
2. Check for the Magic Sum - A square is magic if the sum of every row, every column, and both main diagonals is the same constant.
	1. Calculate the Target Sum
	2. Check All Rows
	3. Check All Columns
	4. Check Both Main Diagonals
	
If the square passes all these subchecks (1-4), it is a Magic Square. Proceed to step 3.

3. Check for Diabolical Square (Additional Property)

A square is diabolical if it is a magic square AND it has a dimension 4 or 8.   When the dimension is 8, the only way that each of the quadrants can sum to the same integer is if they are all 0.

1. Check Dimension Constraint
2. Check Quadrant Sums

# Solution

```java
import java.util.HashSet;
import java.util.Scanner;
import java.util.Set;


public class Solution {
    public static boolean diabolical = false;
    public static void main(String[] args) {
        Scanner scanner = new Scanner(System.in);
        int c = scanner.nextInt();
        for (int testCase = 1; testCase <= c; testCase++) {

            int n = scanner.nextInt();
            int[][] matrix = new int[n][n];

            for (int i = 0; i < n; i++) {
                for (int j = 0; j < n; j++) {
                    matrix[i][j] = scanner.nextInt();
                }
            }

            boolean isMagic = isMagicMatrix(matrix, n);
            if (diabolical) {
                System.out.println("Matrix #" + testCase + " is diabolical.");
            } else if (isMagic){
                System.out.println("Matrix #" + testCase + " is magic.");
            }
            else
                System.out.println("Matrix #" + testCase + " is not magic.");

            System.out.println();
            diabolical = false;
        }

        scanner.close();
    }

    private static boolean isMagicMatrix(int[][] matrix, int n) {

        long expectedSum = 0;
        for (int j = 0; j < n; j++) {
            expectedSum += matrix[0][j];
        }

        long currentRowSum=0, currentColSum=0;
        for (int i = 0; i < n; i++) {
            currentRowSum = 0;
            for (int j = 0; j < n; j++) {
                int num = matrix[i][j];
                currentRowSum += num;
            }

            if (currentRowSum != expectedSum) {
                return false;
            }
        }

        for (int j = 0; j < n; j++) {
            currentColSum = 0;
            for (int i = 0; i < n; i++) {
                currentColSum += matrix[i][j];
            }

            if (currentColSum != expectedSum) {
                return false;
            }
        }

        long mainDiagonalSum = 0;
        for (int i = 0; i < n; i++) {
            mainDiagonalSum += matrix[i][i];
        }

        if (mainDiagonalSum != expectedSum) {
            return false;
        }

        long antiDiagonalSum = 0;
        for (int i = 0; i < n; i++) {
            antiDiagonalSum += matrix[i][n - 1 - i];
        }

        if (antiDiagonalSum != expectedSum) {
            return false;
        }

        if ((n == 8) && (mainDiagonalSum == 0) && (antiDiagonalSum == 0) &&
           (currentRowSum == 0) && (currentColSum == 0) )
        {
            diabolical = true;  return true;
        }

        if (n == 4)
        {
                int quad1=0, quad2=0, quad3=0, quad4 =0;
                for (int i=0; i < n/2;  i++)
                        for (int j=0; j < n/2;  j++)
                                quad1 += matrix[i][j];
                for (int i=0; i < n/2;  i++)
                        for (int j=2; j < n;  j++)
                                quad2 += matrix[i][j];
                for (int i=2; i < n;  i++)
                        for (int j=0; j < n/2;  j++)
                                quad3 += matrix[i][j];
                for (int i=2; i < n;  i++)
                        for (int j=2; j < n;  j++)
                                quad4 += matrix[i][j];
                int[] sums = {quad1, quad2, quad3, quad4};
                int key = quad1;
                for (int i=1; i < 4;  i++)
                        if (sums[i] != key)
                                diabolical = false;
                diabolical = true;
        }

        return true;
    }
}
```
# Eight Bit Interpreter

This is the easiest problem in the set, hidden in slot 9.   This problem is solved by processing the binary input in eight-bit segments, converting each segment from binary to its decimal (base 10) equivalent, and then treating that decimal value as an ASCII code to find the corresponding character.

```c++
#include <fstream>
#include <iostream>
#include <cmath>
using namespace std;
int binary_to_decimal (int binary);

int main()
{
        int ch;

        while (cin >> ch)
                cout << (char) binary_to_decimal(ch);
        return 0;
}

int binary_to_decimal (int binary)
{
        int sum, temp, place, peel;

        temp = binary;  sum = 0;  place = 0;
        while (temp != 0)
        {
                peel = temp % 10;
                temp = temp / 10;
                sum = sum + peel * int(pow(2.0,double(place)));
                place = place + 1;
        }

        return sum;

}
```
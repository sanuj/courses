## Lecture 2

- How are data-types like bool, int, float etc. represented in the memory.

#### C / C++

- `bool`
- `char`      1 byte
- `short`     2 bytes
- `int`       2-4 bytes (but we assume it is always 4 bytes)
- `long`      4 bytes (another minor data-type called `long long` which has 8 bytes)
- `float`     4 bytes
- `double`    8 bytes

- binary digit ➡️ bit (store a 0/1)
- **char**
- 8 bits ➡️ 2^8 = 256
- 'A' ➡️ 65 = 64 + 1 = 2^6 + 2^0 ➡️ | 0 1 0 0 0 0 0 1 |
- **short**
- 2 bytes ➡️ 2^16 values
- | 0 0 0 0 0 0 1 0 | 0 0 0 0 0 1 1 1 | ➡️ 2^0 + 2^1 + 2^2 + 2^9 = 519
- | 0 1 1 1 1 1 1 1 | 1 1 1 1 1 1 1 1 | ➡️ 2^15 - 1
- Left most bit is signed bit i.e 0 is ➕ and 1 is ➖.
- But, -7 is not | 1 0 0 0 0 0 0 0 | 0 0 0 0 0 1 1 1 |, because
```
    0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1     --> +7
  + 1 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1     --> -7
  ------------------------------------
    1 0 0 0 0 0 0 0 0 0 0 0 1 1 1 0     --> -14
```
- So addition is not working properly and we want to keep this as simple as possible because electrons on the hardware will be doing this. The above representation is called 1's complement and is not used to represent negative numbers in computers.
- Therefore, we use 2's complement
```
    0 0 0 0 0 0 0 0 0 0 0 0 0 1 1 1     --> +7
  + 1 1 1 1 1 1 1 1 1 1 1 1 1 0 0 0     --> ? (inverted bits in +7)
  ------------------------------------
    1 1 1 1 1 1 1 1 1 1 1 1 1 1 1 1     --> ?
                                + 1     --> add 1
  ------------------------------------
    0 0 0 0 0 0 0 0 0 0 0 0 0 0 0 0     --> 0!!
```
- 2's complement of a number can be found by:
    - inverting all bits
    - adding one after inversion
    - Eg -7 is | 1 1 1 1 1 1 1 1 | 1 1 1 1 1 0 0 1 |
- And (+7) + (-7) results in 0 in 2's complement system.
- `short` range from (-2^15) to (2^15 - 1).
- Eg -
```cpp
char ch = 'A';
short s = ch;
cout << s << endl;  // outputs '65'
```
```
Bits of char 'ch' are copied in short 's'.

                      | 0 1 0 0 0 0 0 1 |     --> char ch (1 byte)
    | 0 0 0 0 0 0 0 0 | 0 1 0 0 0 0 0 1 |     --> short s (2 bytes)

   zeros are padded   |   Rest of 'ch' is
   here since the     |   copied here.
   left most bit in   |
   'ch' is 0.         |
```
```cpp
// If we do the opposite.
short s = 67;
char ch = s;
cout << ch << endl; // outputs 'C'
```
```
    | 0 0 0 0 0 0 0 0 | 0 1 0 0 0 0 0 1 |     --> short s (2 bytes)
                      | 0 1 0 0 0 0 0 1 |     --> char ch (1 byte)
    everything here is
    dumped due to lack
    of memory.
```
- Similar things happen for `int` and `long`.
```
int i = 2^23 + 2^21 + 2^14 + 7;
short s = i;

    | 0 0 0 0 0 0 0 0 | 1 0 1 0 0 0 0 0 | 0 1 0 0 0 0 0 0 | 0 0 0 0 0 1 1 1 | --> int i
                                        | 0 0 0 0 0 0 0 0 | 0 1 0 0 0 0 0 1 | --> short s
    discards all the bits here
```
```cpp
short s = -1;
int i = s;

                                        | 1 1 1 1 1 1 1 1 | 1 1 1 1 1 1 1 1 | --> short s
    | 1 1 1 1 1 1 1 1 | 1 1 1 1 1 1 1 1 | 1 1 1 1 1 1 1 1 | 1 1 1 1 1 1 1 1 | --> int i

```
- `int` range from (-2^31) to (2^31 - 1).
- Let's assume `float` (4 bytes) is stored like this:
```
    | _ _ _ _ _ _ _ _ | _ _ _ _ _ _ _ _ | _ _ _ _ _ _ _ _ | _ _ _ _ _ _ _ _ |

    2^23 . . . . . . . . . . . . . . . . .  2^2, 2^1, 2^0 |  2^-1, 2^-2, 2^-3 . . . 2^-8
```
- There is nothing wrong with the above representation, basic arithmetic would work fine. The last 8 bits would be used to store the decimal. But this is not how this is done.
- The actual standard to represent floating point numbers are:
```
      _ _   _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _
    | +/- | <---------- 8 ----------> | <--------------- 23 ---------------> |
                                        2^-1, 2^-2, 2^-3 . . . . . . . . . 2^-23

    first bit is signifies sign (s) -> + or -.
    next 8 bits represent exponent (exp).
    last 23 bits represent number after decimal (xxxxxx).

    where the number is: (-1)^s 1.xxxxxx * 2^(exp-127)

    Suppose we want to represent 7.0 with this new standard:
    7.0 * 2^0
    3.5 * 2^1
    1.75 * 2^2
```
- Now, what happens when we assign a `float` to an `int` or vice versa?
```cpp
    int i = 5;
    float f = i;
    cout << f << endl;  // prints 5.0
    // program internally converts the bit pattern for int 5 to float 5.0

    // But if we do
    int i = 37;
    float f = *(float*)&i;
    cout << f << endl;  // prints a different number than 37.0
    // program doesn't internally convert the bit pattern but
    // simply copies the same bit pattern which results in different
    // interpretation of the integer 37 (which turns out to be a very small float).
```
- If we convert `float` to `short`:
```cpp
    float f = 7.0;
    short s = *(short*)&f;
    // in this case only the first 16 bits will be copied
    // without any internal conversion.
    pointer --->| _ _ _ _ _ _ _ _ | _ _ _ _ _ _ _ _ | _ _ _ _ _ _ _ _ | _ _ _ _ _ _ _ _ |
                |   first 8 bits are copied         |
```

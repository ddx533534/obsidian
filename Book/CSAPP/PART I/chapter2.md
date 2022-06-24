# Representing and Manipulating Information

## 1.OverView

In isolation, a single bit is not very useful. When we group bits together and apply some **interpretation** that gives meaning to the different possible bit patterns, however, we can represent the elements of any fifinite set

We consider the three most important representations of numbers. 

1. **Unsigned** encodings are based on traditional binary notation, representing numbers greater than or equal to 0. 
2. **Two’s-complement** encodings are the most common way to represent **signed** integers, that is, numbers that may be either positive or negative.
3. **Floating-point** encodings are a base-2 version of scientifific notation for representing real numbers.  Floating-point arithmetic is not associative due to the finite precision of the representation.

Computers implement arithmetic operations, such as addition and multiplication, with these different representations, similar to the correspond ing operations on integers and real numbers. **The computer might not generate the expected result, but at least it is consistent!**

And a number of computer security vulnerabilities have arisen due to some of the subtleties of computer arithmetic.



## 2.Information Storage

1. Rather than accessing individual bits in memory, **most computers use blocks of 8 bits, or bytes, as the smallest addressable unit of memory.** 
2. A machine-level program views memory as a very large array of bytes, referred to as **virtual memory**. 
3. Every byte of memory is identified by a unique number, known as its address, and the set of all possible addresses is known as the virtual address space

> C Pointer?
>
> Pointers are a central feature of C. They provide the mechanism for referencing elements of data structures, including arrays. Just like a variable, a pointer has two aspects: its *value* and its type. The value indicates the location of some object, while its type indicates what kind of object (e.g., integer or flfloating-point number) is stored at that location.

### 2.1 **Hexadecimal Notation**

Binary notation is too verbose, while with decimal notation it is tedious to convert to and from bit patterns. Instead, we write bit patterns as base-16, or **hexadecimal** numbers. Hexadecimal (or simply “hex”) uses digits ‘0’ through ‘9’ along with characters ‘A’ through ‘F’ to represent 16 possible values.

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220528113548145.png" alt="image-20220528113548145" style="zoom:50%;" />

### 2.2 Data Size

Every computer has a **word size**, **indicating the nominal size of pointer data**. Since a virtual address is encoded by such a word, the most important system parameter determined by the word size is the maximum size of the virtual address space. **That is, for a machine with a w-bit word size, the virtual addresses can range from 0 to ^  2w − 1, giving the program access to at most 2^w bytes.**

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220528155731544.png" alt="image-20220528155731544" style="zoom: 33%;" />

1. Most 64-bit machines can also run programs compiled for use on 32-bit machines, a form of backward compatibility.

2. Computers and compilers support multiple data formats using different ways to encode data, such as integers and flfloating point, as well as different lengths.

   <img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220528145038360.png" alt="image-20220528145038360" style="zoom:50%;" />

### 2.3 Addressing and Byte Ordering

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220528160023976.png" alt="image-20220528160023976" style="zoom:50%;" />

1. A common problem is for data produced by a little-endian machine to be sent to a big-endian machine, or vice versa, leading to the bytes within the words being in reverse order for the receiving program
2. A second case where byte ordering becomes important is when looking at the byte sequences representing integer data.



> Naming data types with typedef ？
>
> The typedef declaration in C provides a way of giving a name to a data type. This can be a great help in improving code readability, since deeply nested type declarations can be difficult to decipher. The syntax for typedef is exactly like that of declaring a variable, except that it uses a type name rather than a variable name.
> For example, the declaration 
>
> ​     typedef int *int_pointer; 
> ​     int_pointer ip; 
>
> defines type int_pointer to be a pointer to an int, and declares a variable ip of this type. Alternatively, we could declare this variable directly as int *ip;



### 2.4 Boolean Algebra

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220529105411312.png" alt="image-20220529105411312" style="zoom:50%;" />

### 2.5 Bit-Level Operations in C

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220529172233672.png" alt="image-20220529172233672" style="zoom:50%;" />

Note that unlike the usual technique for swapping two values, we do not need a third location to temporarily store one value while we are moving the other. **There is no performance advantage to this way of swapping; it is merely an intellectual amusement.😓**

### 2.6 **Logical Operations in C**

C also provides a set of *logical* operators ||, &&, and !, which correspond to the or, and, and not operations of logic. 

1. These can easily be confused with the bit level operations, but their behavior is quite different. **The logical operations treat any nonzero argument as representing true and argument 0 as representing false.**
2. A second important distinction between the logical operators ‘&&’ and ‘||’ versus their bit-level counterparts ‘&’ and ‘|’ is that the **logical operators do not evaluate their second argument if the result of the expression can be determined by evaluating the first argument(short circuit🤓)**



### 2.7 Shift Operations in C

1. Left Shift: The C expression x << k yields a value with bit representation [*x**w*−*k*−1*, x**w*−*k*−2*,...,x*0*,* 0*,...,* 0]. That is, x is shifted *k* bits to the left, dropping off the *k* most signifificant bits and filling the right end with *k* zeros.
2. Right Shift: Two forms：
   - **Logical**. A logical right shift fills the left end with *k* zeros, giving a result [0*,...,* 0*, x**w*−1*, x**w*−2*,...x**k*].
   - **Arithmetic** An arithmetic right shift fills the left end with *k* repetitions of the most signifificant bit, giving a result [*x**w*−1*,...,x**w*−1*, x**w*−1*, x**w*−2*,...x**k*]. This convention might seem peculiar, but as we will see, it is useful for operating on signed integer data.

​     **The C standards do not precisely defifine which type of right shift should be used with signed numbers—either arithmetic or logical shifts may be used**. This unfortunately means that any code assuming one form or the other will potentially encounter portability problems. **In practice, however, almost all compiler/machine combinations use arithmetic right shifts for signed data, and many programmers assume this to be the case. For unsigned data, on the other hand, right shifts must be logical**. **In contrast to C, Java has a precise defifinition of how right shifts should be performed. The expression x >> k shifts x arithmetically by k positions, while x >>> k shifts it logically.**

> Operator precedence issues with shift operations
>
> It might be tempting to write the expression 1<<2 + 3<<4, intending it to mean (1<<2) + (3<<4). However, in C the former expression is equivalent to 1 << (2+3) << 4, since addition (and subtraction) have higher precedence than shifts. T
>
> Getting the precedence wrong in C expressions is a common source of program errors, and often these are diffificult to spot by inspection. **When in doubt, put in parentheses**



## 3.Integer Representations

### 3.1 Two’s-Complement Encodings

For many applications, we wish to represent negative values as well. The most common computer representation of signed numbers is known as *two’s-complement* form. **This is defifined by interpreting the most signifificant bit of the word to have negative weight.** We express this interpretation as a function *B2Tw* (for “binary to two’s complement” length *w*):

 PRINCIPLE: Defifinition of two’s-complement encoding For vector *x* = [x^w−1, x^w−2,...,x^0]:

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220619201944787.png" alt="image-20220619201944787" style="zoom:50%;" />

1. The  greatestvalue :  **TMin w= -2^(w-1)**
2. The Least value: TMax w = <img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220619202342587.png" alt="image-20220619202342587" style="zoom:50%;" /> =2^(w-1) - 1
3. The two’s-complement range is **asymmetric**: |*TMin*| = |*TMax*| + 1; that is, there is no positive counterpart to *TMin*
4. <img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220619203130210.png" alt="image-20220619203130210" style="zoom:50%;" />

### 3.2 Conversions between Signed and Unsigned

What we see here is that the effect of casting is to keep the bit values identical **but change how these bits are interpreted**. **Casting from short to unsigned short changed the numeric value, but not the bit representation.**

C allows conversion between unsigned and signed. Although the C standard does not specify precisely how this conversion should be made, most systems follow the rule that the underlying bit representation does not change.

1. Conversions can happen due to explicit casting, such as in the following code:

   ```c
   int tx, ty;
   unsigned ux, uy;
   
   tx = (int) ux;
   uy = (unsigned) ty;
   ```

   

2. Alternatively, they can happen implicitly when an expression of one type is as signed to a variable of another, as in the following code:

   ```c
   int tx, ty;
   unsigned ux, uy;
   
   tx = ux; /* Cast to signed */
   uy = ty; /* Cast to unsigned */
   ```

3. When printing numeric values with printf, the directives %d, %u, and %x are used to print a number as a signed decimal, an unsigned decimal, and in hexadecimal format, respectively. Note that printf does not make use of any type information, and **so it is possible to print a value of type int with directive %u and a value of type unsigned with directive %d**.

4.  When an operation is performed where one operand is signed and the other is unsigned, C implicitly casts the signed argument to unsigned and performs the operations assuming the numbers are nonnegative.

   <img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220621102104744.png" alt="image-20220621102104744" style="zoom:50%;" />

### 3.3 Expanding the Bit Representation of a Number

One common operation is to convert between integers having different word sizes while retaining the same numeric value. Of course, this may not be possible when the destination data type is too small to represent the desired value. Converting from a smaller to a larger data type, however, should always be possible.

1. Expansion of an unsigned number by zero extension

   <img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220622095808062.png" alt="image-20220622095808062" style="zoom:50%;" />

2. Expansion of a two’s-complement number by sign extension

   <img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220622095636161.png" alt="image-20220622095636161" style="zoom:50%;" />

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220622095752963.png" alt="image-20220622095752963" style="zoom:50%;" />

### 3.4 Truncating Numbers

Suppose that, rather than extending a value with extra bits, we reduce the number of bits representing a number. This occurs, for example, in the following code:

```c
int x = 53191;
short sx = (short) x;	 /* -12345 */ 
int y = sx;  					/* -12345 */
```

We have seen multiple ways in which the subtle features of unsigned arithmetic, and especially the implicit conversion of signed to unsigned, can lead to  errors or vulnerabilities. One way to avoid such bugs is to never use unsigned numbers. In fact, few languages other than C support unsigned integers. Apparently, these other language designers viewed them as more trouble than they are worth. For example, Java supports only signed integers, and it requires that they be implemented with two’s-complement arithmetic. The normal right shift operator >> is guaranteed to perform an arithmetic shift. The special operator >>> is defifined to perform a logical right shift. Unsigned values are very useful when we want to think of words as just collections of bits with no numeric interpretation. This occurs, for example, when packing a word with *flflags* describing various Boolean conditions. Addresses are naturally unsigned, so systems programmers fifind unsigned types to be helpful. Unsigned values are also useful when implementing mathematical packages for modular arithmetic and for multiprecision arithmetic, in which numbers are represented by arrays of words

## 4.Integer Arthmetic

Many beginning programmers are surprised to fifind that adding two positive numbers can yield a negative result, and that the comparison x<y can yield a different result than the comparison x-y < 0. These properties are artifacts of the fifinite nature of computer arithmetic

### 4.1 unsigned addtion

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220622103746734.png" alt="image-20220622103746734" style="zoom:33%;" />

Let us defifine the operation +u*w* for arguments *x* and *y*, where 0 ≤ *x, y <* 2^w, as **the result of truncating the integer sum *x* + *y* to be *w* bits long and then viewing the result as an unsigned number.**

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220622103337131.png" alt="image-20220622103337131" style="zoom:50%;" />

<img src="/Users/dudongxu/Library/Application Support/typora-user-images/image-20220622104154695.png" alt="image-20220622104154695" style="zoom:50%;" />

#Binary/bitwise

### Basics:


- A bit is set if it is 1 and unset if it is 0. Example: 10 = 1010 in binary. The lowest bit is unset, the next is set, the next unset, etc. 
- 65535 occurs frequently in the field of computing because it is (one less than 2 to the 16th power), which is the highest number that can be represented by an unsigned 16-bit binary number. It can be represented in hexadecimal as 0xFFFF. 

### Bit Masking

Bit masking allows you to use operations that work on a bit-level, i.e edit particular bits in a byte or check if particular bit values are present or not. 

You apply a mask. The mask itself is a binary number which specifies which bits we're interested in. You apply the mask with the usual binary operators AND, OR, XOR, etc. 

AND will get a subset of the bits in x
OR will set a subset of the bits in x
XOR will toggle a subset of the bits in x

### & binary AND. a & b = 1 only if a AND b are 1
so 1100 & 1101 = 1100
	
	def count_bits(x):
	    num_bits = 0
	    while x:
	        num_bits += x & 1
	        x >>= 1
	    return num_bits

`count_bits` works by inspecting each binary digit and seeing if it's 1. `x & 1` will only be 1 if x is 1.
if it's 1, we increment our counter, otherwise we add 0 (effectively not incrementing)

so our function here works by just constantly shifting the bits to the right and reading
the right-most one to see if it's 1 and then adding it to the count
10 >> 1 = 1010 >> 1 = 101. Just drop the last bit.

	 bin(13)
	'0b1101'
	 
	 bin(13 >> 1)
	'0b110'


### | binary OR. a & b = 1 if either a OR b are 1

	a = 60 = 0011 1100
	b = 13 = 0000 1101
	a | b =  0011 1101

	bin(60 | 13)
	'0b111101' = 0011 1101

### ^ Binary XOR. a ^ b = It copies the bit if it is set in one operand but not both.	

	bin(60 ^ 13)
	'0b110001'
	
### ~ Binary Ones Complement	

This is a _unary_ operator, only works on one number.

Inverts all the bits. Each 1 becomes 0 and each 0 becomes 1. So 1111 -> 0000, 0000 -> 1111, 1010 -> 0101

	>>> bin(7)
	'0b111'
	>>> bin(~7)
	'-0b1000'

Remember that negative numbers are stored as the two's complement of the positive counterpart. As an example, here's the representation of -2 in two's complement: (8 bits)

`1111 1110`

Why is this `-2`? Because you take its complement (invert all bits) and add 1. So 2 in binary is 0000 0010. The complement 1111 1101. Then we add one getting us 1111 1110. The first bit is is the sign bit, implying a negative. 

**The complement operator (~) JUST FLIPS BITS. It is up to the machine to interpret these bits.**

### parity

The parity of a binary word is 1 if the number of 1s in the word is odd; otherwise, it is 0. For example, the parity of 1011 is 1, and the parity of 10001000 is 0. 

This is the brute-force approach: 

	def parity(x):
	    result = 0
	    while x:
	        result ^= x & 1
	        x >>= 1
	    return result

How does this work? `parity` returns 1 if number of 1s is odd, otherwise 0.
`x & 1` checks if x is 1 
then the XOR sets the result to 1 only it's already 0. If it's 1 then it gets set to 0. 

Effectively, every time `result` is already 1, we wipe it back to zero using `^` if we encounter another 1. 

The time complexity is _O(n)_, where n is the word size.

Can improve by using trick...

### unsetting/erasing the lowest set bit

`x &(x - 1) 

	>>> bin(10)
	'0b1010'
	>>> bin(10 & 10 -1)
	'0b1000'

How does this work? Well, first we subtract 1. So 10 = 0b1010. 10 - 1 = 0b1001. Now we use & 

	1010
	&
	1001
	=
	1000
	
	>>> bin(10 & 10 -1)
	'0b1000'
	>>> bin(10 & 9)
	'0b1000'
	
We can use this to reduce time complexity. If _k_ is number of bits set to 1, then time complexity of this algo is _O(k)_:

	def parity(x):
    result = 0
    while x:
        result ^= 1 
        x &= x - 1  # drop the lowest set bit of x
    return result
    
All we're doing here is just counting 1s because we just flip the result to = 1 if it's currently 0 and to 0 if currently 1. So that's just the odd/even tracker. And we keep dropping the lowest set digit, eventually getting to the number 0. 

Observe:

	>>> x = 13
	>>> format(x, 'b')
	'1101'
	>>> x &= x -1
	>>> x
	12
	>>> format(x, 'b')
	'1100'
	>>> x &= x -1
	>>> x
	8
	>>> format(x, 'b')
	'1000'
	>>> x &= x -1
	>>> x
	0
	>>> format(x, 'b')
	'0'


### dealing with a very large number of words 

Two keys to performance 

1. process multiple bits at a time
2. cache results in array-based lookup table 

Obviously can't just cache the parity of every 64-bit integer (an array with just the answer for every number). Way too much storage required! 

But doesn't matter how we group bits when computing parity. The computation is associative. 

So we can 

1. take 64-bit integer and group into 4 16-bit subwords, 
2. compute the parity of each subword
3. compute the parity of the four subresults


In other words if we have 1000 0101 0000 0001, we have 

	parity of 1000 = 1
	parity of 0101 = 0
	parity of 0000 = 0
	parity of 0001 = 1
	
	then take parity of results
	
	parity of 1001 = 0
	
So we'll take all the 16-bit subwords and we _can_ feasibly cache those in an array. Plus 16 evenly divides 64 so code is simpler. 

Let's illustrate the concept using 2-bit words (not like the 16-bit words we'll actually use). 

We'll have arrays with the parities. For example [0, 1, 1, 0] are the parities of 0, 1, 2, 3 since 0 = 0, 1 = 01, 2 = 10, 3 = 11. 

To get parity of 11001010 all we need is the parity of 11, 00, 10, 10. The cache shows us that the parity of these is 0, 0, 1, 1 which is 0!

But how do we first grab the 11 from 11101010 (diff # than before)  to lookup the parity of it? We right shift by 6. 11101010 >> 6 = 00000011. This is the number 3. Use 3 as the index into the array to grab [0, 1, 1, (0)]. Hence the parity is 0. 

Next we need the 10 from 11(10)1010. We right shift by 4 to get 0000111. But this is 7 which would give us an out-of-bounds access on the array bc the indices are only 0 through 3. 

To solve this we bitwise AND our original number with the number we got after the right shift, namely the 00000011. So we have `11101010 & 00000011`. The `00000011` is the "mask" to get the last two bits. the result is `00000010` which is 2 which is [0, 1, (1), 0] in our array so the parity is 1. 

**A mask defines which bits you want to keep, and which bits you want to clear. You apply the mask by:**

- Bitwise ANDing in order to extract a subset of the bits in the value
- Bitwise ORing in order to set a subset of the bits in the value
- Bitwise XORing in order to toggle a subset of the bits in the value


Here's the full solution from the book:

	def parity(x):
    MASK_SIZE = 16
    BIT_MASK = 0xFFFF
    return (PRECOMPUTED_PARITY[x >> (3 * MASK_SIZE)] ^
            PRECOMPUTED_PARITY[(x >> (2 * MASK_SIZE)) & BIT_MASK] ^
            PRECOMPUTED_PARITY[(x >> MASK_SIZE) & BIT_MASK] ^
            PRECOMPUTED_PARITY[x & BIT_MASK])

`MASK_SIZE` is the number of bits we're working with or want to shift (in the simpler example the book gave earlier, mask size would be 2)

Then we have this `BIT_MASK` where each bit is set to 1. `0xFFFF` is 65535, the largest number that can be stored in a 16-bit binary number. 

First we shift `x >> 3 * MAX_SIZE` to get the last MAX_SIZE bits.

For example, let's say we had the following 64-bit number:

`10001111 01110001 11001100 00010111 01010010 10100010 00110010 10010010`

Incidentally, this is `10336267020334674578` in base 10. 

Ok so we want to get the _last_ 16 bits from this. 

	>>> x =
	'1000111101110001110011000001011101010010101000100011001010010010'
	>>> b = int(x, 2)
	>>> b
	10336267020334674578 
	>>> last_bits = b >> (3 * MASK_SIZE)
	>>> bin(last_bits)
	'0b1000111101110001'
	
In other words we went from 

`10001111 01110001 11001100 00010111 01010010 10100010 00110010 10010010`

and got 

`10001111 01110001`

i.e. the last (left most) 16 bits. This result itself is used as an index into our array and get the parity for that number, either a 0 or a 1. Then we just xor this with the next subword. 

How do we get the next subword. Well, instead of right shifting with `3 * MASK_SIZE`, we do it with `2 * MASK_SIZE`, except now we have to use a bitmask to get at the those digits. 

	>>> BIT_MASK = 0xFFFF
	>>> next_bits = b >> (2 * MASK_SIZE) & BIT_MASK
	>>> bin(next_bits)
	'0b1100110000010111'
	
In other words we got the digits

`11001100 00010111`

`10001111 01110001 [11001100 00010111] 01010010 10100010 00110010 10010010`

And then the same to get the next 

	>>> next_bits = (b >> 2) & BIT_MASK
	>>> bin(next_bits)
	'0b1000110010100100' 
	
which is `10001100 10100100`

which is 

`10001111 01110001 11001100 00010111 [01010010 10100010] 00110010 10010010`

By xoring all of these we set the result to 1 only if we have 1 and 0, other we set to 0, effectively flipping between odd and even by using xor. 

In the end, the **time complexity is O(n/L) where n is the word size and L is the width of the words for which the results are cached.**

We can do one better using the fact that xor of two bits is defined to be 0 if both bits are 0 or if both are 1; otherwise it's 1. Observe:

	>>> 0 ^ 0
	0
	>>> 1 ^ 1
	0
	>>> 1 ^ 0
	1
	>>> 0 ^ 1
	1 

xor is associative and commutative - doesn't matter how we group the bits and the order doesn't matter.

	>>> 0 ^ 0 ^ 1 == 1 ^ 0 ^ 0 == (1 ^ 0) ^ 0 == 1 ^ (0 ^ 0)
	True

And the xor of a group of bits is its parity, e.g. in the above example we have an odd number of 1s so the parity is 1 (True = 1). Can exploit this and use CPU's word-level xor to process multiple bits at the same time...

Example: the parity of <b63, b62...b3, b2, b1, b0> is equal to the parity of the xor of <b63, b62, ...b32> and <b31, b30...b0> and these 32-bit values can can be computed with a single shift and single 32-bit xor instruction. Repeat same op on on 32-bit, 16-bit, 8-bit, 4-bit, 2-bit and 1-bit operands to get the final result. (Leading bits are not meaningful and we have to explicitly extract result from the least significant bit)

Example: the parity of `11010111` is the same as the parity of `1101` xored with `0111`, i.e. of `1010`

	>>> a = 13
	>>> b = 7
	>>> bin(a)
	'0b1101'
	>>> bin(b)
	'0b111'
	>>> bin(a ^ b)
	'0b1010'

This in turn has the parity of 10 xored with 10, i.e. 0

	>>> a = 2
	>>> b = 2
	>>> bin(a)
	'0b10'
	>>> bin(b)
	'0b10'
	>>> bin(a ^ b)
	'0b0'
	
Example:	
	
	a = 1000111101110001 1100110000010111 010100101010001 00011001010010010
	
We can get the first 32 bits by right shifting `a >> 32` which gives us 

	    1000111101110001 1100110000010111
	
	
So you can just keep xoring half the word size with itself to get the parity. As a final step we need to grab the last bit by using a mask of 1. 

The time complexity will be _O(logn)_

	def parity(x):
	    x ^= x >> 32
	    x ^= x >> 16
	    x ^= x >> 8
	    x ^= x >> 4
	    x ^= x >> 2
	    x ^= x >> 1
	    return x & 0x01
	
Easier to think of this on an 8-bit number 

	#take x = 11110101
	def parity(x):
	    x ^= x >> 4 # 11110101 >> 4 = 1111 and 11110101 ^ 00001111 = 11111010
	    x ^= x >> 2 # 11111010 >> 2 = 111110 and 11111010 ^ 00111110 = 11000100
	    x ^= x >> 1 # 11000100 >> 1 = 1100010 and 11000100 ^ 1100010 = 0100110
	    return x & 0x01 # take the last bit which is 0 which is the parity of 11110101
	

###Right propagate the rightmost 1-bit.

`y = x | (x-1)`

`x - 1` only ever modifies bits to the right of the right-most set bit. The right-most bit turns into a 0 and the all the lower bits become 1. 	

 Observe:

	101010(1)(0) – 1 
	=
	
	10101001
	
	00101000 – 1 
	= 
	00100111
	
	101111 – 1 
	= 
	101110
	
	
So all that's left to do is OR with that and this propagates the right-most bit. 

### swap bits

We have some tricks e.g x & (x - 1) clears lowest set bit. x & ~(x - 1) extracts lowest set bit. 

We can also extract a bit at position `i` with (x >> i) & 1

	>>> x = 9
	>>> bin(x)
	'0b1001'
	>>> bin((x >> 1) & 1)
	'0b0'
	>>> bin((x >> 2) & 1)
	'0b0'
	>>> bin((x >> 3) & 1)
	'0b1'

 
So, to swap two bits in a number we first check if they're the same. If they are, we don't need to do anything. If they're different, flipping each bit is the same thing as swapping them. 

Note: the left shift operator `x << n` has the effect of filling in `n` zeros to the right of `x`, e.g.:

`1001 << 2 = 100100`

So all we need to do is select the bits and XOR them since x ^ 1 = 1 if x is 0 otherwise x ^ 1 = 0 - so it flips/swaps them. 

	x = 10111
let's say we want to flip the 3 and 1 index
so i = 3 and j = 1
`1(0)1(1)1` so we want to get `1(1)1(0)1` or go from `10111` to `11101`.

first we check that they're different. They are so we set up a bit mask

	bit_mask = (1 << i) | (1 << j)

	1 << 3 = 1000
	1 << 1 = 0010
	
	
We're simply creating two binary numbers, one with a bit set in the `i` position and one with a bit set in the `j` position. By ORing these, we have a number with bits set in the `i` *and* `j` positions. We can apply this mask to our input `x` because `x ^ 1` = 0` when `x = 1` and 1 when `x = 0` to swap the bits. 

and now we OR them (outputs 1 if either is 1)
	
	1000
	|
	0010
	=
	1010 # <---- this is our bit_mask

now we xor our original x with this bitmask

	10111
	^
	01010
	=
	11101
	
Full solution is 

	def swap_bits(x, i, j):
	    if (x >> i) & 1 != (x >> j) & 1:
	        bit_mask = (1 << i) | (x << j)
	        x ^= bit_mask
	    return x
	    
The time complexity is `O(1)`, independent of the word size. 

### closest integer with the same weight 

The weight of an integer is the number of bits set to 1 in it, e.g. 101 weight = 2, 11110 weight = 4, etc. 

Write a program that takes x and outputs y which has the same weight as x (but not the same number duh) and where the difference |x-y| is as small as possible. If input is 6, for example, output should be 5. Since 6 = 110 and 5 = 101. 

The hint we're given is to start with the LSB. Ok... so we know that `x & 1` gives the LSB of `x. 

	>>> 0b011 & 1
	1
	>>> 0b010 & 1
	0
	
Here, if we flip the bit at index k1 and the bit at k2, k1 > k2

	>>> a = 0b111
	>>> b = 0b100 #flipped k1 and k2

Now let's check the difference. `a - b = 011` - the absolute value of the difference is 2^k1 - 2^k2. In our example here, that means a - b = 2^0 - 2^1 which is 3 or 011b. Given this, we want to make k1 as small as possible and k2 as close to k1 to have the difference be as small as possible. 

Ok so we want to take the smallest index k1 possible and have the other index k2 be as close to that as possible. But we have the constraint that the bit at index k1 has to be different from the bit in k2. If we don't do that, the flip results in an integer with a different weight. In other words, if the bits were 0, 0 and we flipped em, we'd get 1, 1 and we'd now have a different weight cuz we just added two 1s. Alternatively, if those bits were 1, 1 and we flipped the to 0, 0...once again we got rid of two 1s. Whereas if the bits were 1, 0 or 0, 1 then we flip to 0,1 or 1,0 and we have the same weight. Ok....so...

So the smallest k1 we can use is the rightmost bit that's different from the LSB and k2 must be the very next bit. You can see this here:

	>>> 0b110
	6
	>>> 0b101
	5

We flipped the two rightmost bits that differ! **In summary, we want to swap the two rightmost consecutive bits that are different.**

Solution is:

	def closest_int_same_bit_count(x):
	    NUM_UNSIGNED_BITS = 64
	    for i in range(NUM_UNSIGNED_BITS - 1): # loop will start at LSB
	        if (x >> i) & 1 != (x >> (i + 1)) & 1: # if the bits at k1 and k2 are different
	            x ^= (1 << i) | (1 << (i + 1)) # then swap em. We already know how to do this using XOR
	            return x
	
	    raise ValueError('All bits are 0 or 1') # special case - whatever




time complexity is _O(n)_ where _n_ is the integer width. 


### reverse digits

42 -> 24, 89 -> 98, -314 -> -413

Hint: how would you solve if the input was given as a string?





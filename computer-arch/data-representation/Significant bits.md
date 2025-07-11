# Most significant bits
*pwncollege*

The terms "most significant bit" (MSB) and "least significant bit" (LSB) describe the relative importance of each bit in a binary number based on how much it affects the overall value. The most significant bit is the leftmost bit and represents the highest place value (such as 2⁷ = 128 in an 8-bit number), while the least significant bit is the rightmost bit and represents the smallest place value (2^0 = 1). 

This naming reflects their impact: changing the MSB causes a large shift in value, whereas changing the LSB only alters the value slightly.  

For example, flipping the MSB in the binary number `10000000` to `00000000` drops the decimal value from 128 to 0—a significant change. In contrast, flipping the LSB in `10000000` to `10000001` increases the value from 128 to 129—only a minor change. Thus, the MSB is the most impactful in determining a number's magnitude, and the LSB is the least.


Binary: base 2, has a prefix of `0b`, 0 and 1 used
Decimal: base 10, has prefix of `0d`, 0~9 used
Hexadecimal: base 16, `0x`, 0~9 and A~F used
- Each hexadecimal digits contain 4 bits of information.
- E.g., max value of 4 binary digits (`1111`) gives the maximum hexadecimal digit (`F`).
![[Pasted image 20240901135550.png]]
## Conversion from binary to hexadecimal
![[Pasted image 20240901140041.png]]
- Split the number into 4-bit groups
- Convert each group to decimal.
- Convert the decimal number to hexadecimal.
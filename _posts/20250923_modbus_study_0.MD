# 15 (0x0F) Write multiple coils
Here is an example of a request to write a series of 10 coils starting at coil 20:
The request data contents are two bytes: CD 01 hex (1100 1101 0000 0001 binary). The binary bits correspond to the outputs in the following way.

## Target Value

| Seq | 27 | 26 | 25 | 24 | 23 | 22 | 21 | 20 | 35 | 34 | 33 | 32 | 31 | 30 | 29 | 28 |
|:-:  |:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| Bit | 1 | 1 | 0 | 0 | 1 | 1 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |
| Hex |   |   |   | C |   |   |   | D |   |   |   | 0 |   |   |   | 1 |


## Request Message

| Field Name | Value(HEX) |
| :-- | :-: |
| Function | 0F |
| Starting Address Hi | 00 |
| Starting Address Lo | 13 |
| Quantity Outputs Hi | 00 |
| Quantity Outputs Lo | 0A |
| Byte Count | 0A |
| Outputs Value Hi | CD |
| Outputs Value Lo | 01 |

## Response

| Field Name | Value(HEX) |
| :-- | :-: |
| Function | 0F |
| Starting Address Hi | 00 |
| Starting Address Lo | 13 |
| Quantity Outputs Hi | 00 |
| Quantity Outputs Lo | 0A |


# 23(0x17) Read/write multiple registers
Here is an example of a request to read six registers starting at register 4, and to write three registers starting at register 15.

## Request Message

| Field Name | Value(HEX) |
| :-- | :-: |
| Function | 17 |
| Read Starting Address Hi | 00 |
| Read Starting Address Lo | 03 |
| Quantity to Read Hi | 00 |
| Quantity to Read Lo | 06 |
| Write Starting Address Hi | 00 |
| Write Starting Address Lo | 0E |
| Quantity to Write Hi | 00 |
| Quantity to Write Lo | 03 |
| Write Byte Count | 06 |
| Write Registers Value Hi | 00 |
| Write Registers Value Lo | FF |
| Write Registers Value Hi | 00 |
| Write Registers Value Lo | FF |
| Write Registers Value Hi | 00 |
| Write Registers Value Lo | FF |

## Response

| Field Name | Value(HEX) |
| :-- | :-: |
| Function | 17 |
| Byte Count | 0C |
| Read Registers value Hi | 00 |
| Read Registers value Lo | FE |
| Read Registers value Hi | 0A |
| Read Registers value Lo | CD |
| Read Registers value Hi | 00 |
| Read Registers value Lo | 01 |
| Read Registers value Hi | 00 |
| Read Registers value Lo | 03 |
| Read Registers value Hi | 00 |
| Read Registers value Lo | 0D |
| Read Registers value Hi | 00 |
| Read Registers value Lo | FF |


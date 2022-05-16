Value Conversions
=================


Convert Between Integer and Floating-Point
------------------------------------------

| Function     |        From type        |    To type     |
| ------------ | :---------------------: | :------------: |
| `to_int`     | floating-point, decimal |    integer     |
| `to_float`   |    integer, decimal     | floating-point |
| `to_decimal` | integer, floating-point |    decimal     |

That's it; for other conversions, register custom conversion functions.

```js
let x = 42;                     // 'x' is an integer

let y = x * 100.0;              // integer and floating-point can inter-operate

let y = x.to_float() * 100.0;   // convert integer to floating-point with 'to_float'

let z = y.to_int() + x;         // convert floating-point to integer with 'to_int'

let d = y.to_decimal();         // convert floating-point to Decimal with 'to_decimal'

let w = z.to_decimal() + x;     // Decimal and integer can inter-operate

let c = 'X';                    // character

print(`c is '${c}' and its code is ${c.to_int()}`); // prints "c is 'X' and its code is 88"
```


Parse String into Number
------------------------

| Function                    |         From type          |          To type          |
| --------------------------- | :------------------------: | :-----------------------: |
| `parse_int`                 | [string](strings-chars.md) |          integer          |
| `parse_int` with radix 2-36 | [string](strings-chars.md) | integer (specified radix) |
| `parse_float`               | [string](strings-chars.md) |      floating-point       |
| `parse_decimal`             | [string](strings-chars.md) |          decimal          |

```rust
let x = parse_float("123.4");   // parse as floating-point
x == 123.4;
type_of(x) == "f64";

let x = parse_decimal("123.4"); // parse as Decimal value
type_of(x) == "decimal";

let x = 1234.to_decimal() / 10; // alternate method to create a Decimal value
type_of(x) == "decimal";

let dec = parse_int("42");      // parse as integer
dec == 42;
type_of(dec) == "i64";

let dec = parse_int("42", 10);  // radix = 10 is the default
dec == 42;
type_of(dec) == "i64";

let bin = parse_int("110", 2);  // parse as binary (radix = 2)
bin == 0b110;
type_of(bin) == "i64";

let hex = parse_int("ab", 16);  // parse as hex (radix = 16)
hex == 0xab;
type_of(hex) == "i64";
```


Format Numbers
--------------

| Function    | From type |          To type           |             Format             |
| ----------- | :-------: | :------------------------: | :----------------------------: |
| `to_binary` |  integer  | [string](strings-chars.md) | binary (i.e. only `1` and `0`) |
| `to_octal`  |  integer  | [string](strings-chars.md) |    octal (i.e. `0` ... `7`)    |
| `to_hex`    |  integer  | [string](strings-chars.md) |     hex (i.e. `0` ... `f`)     |

```rust
let x = 0x1234abcd;

x == 305441741;

x.to_string() == "305441741";

x.to_binary() == "10010001101001010101111001101";

x.to_octal() == "2215125715";

x.to_hex() == "1234abcd";
```

# novus-std — Standard Library

The Novus standard library. Provides core types, string utilities, arrays, I/O,
memory access, and platform abstractions for macOS, Linux and Windows.

> **Status note:** Novus is still incomplete. The compiler, std, and the wider
> library ecosystem still have missing features, rough edges, and known bugs.
> Treat this document as the current practical API reference, not a stability
> guarantee.

Import in any project that has `std` in `libraries.conf` (added by default by
`nox init`):

```novus
module myapp;
import lib/std std;

fn main() -> i32 {
    std.print("hello, world");
    return 0;
}
```

### Repository layout

```
core/                 cross-platform helpers (strings, math, conversions, aliases)
platforms/darwin/     macOS Apple Silicon implementation
platforms/linux/      Linux (amd64 + arm64) implementation
platforms/windows/    Windows (amd64 + x86) implementation
tests/                per-platform regression tests + unified_api.nov
main.nov              dispatcher — picks the right platform automatically
```

> All names below live in the imported `std` namespace. Functions are
> **overloaded** — call `len(x)` or `to_str(x)` without worrying about the
> underlying type.

---

## Unified API (preferred)

### Conversions

| Signature                       | Description                       |
| ------------------------------- | --------------------------------- |
| `to_str(n: i32) -> str`         | i32 → decimal string              |
| `to_str(n: i64) -> str`         | i64 → decimal string              |
| `to_str(n: u32) -> str`         | u32 → decimal string              |
| `to_str(n: u64) -> str`         | u64 → decimal string              |
| `to_str(n: f32) -> str`         | f32 → trimmed decimal string      |
| `to_str(n: f64) -> str`         | f64 → trimmed decimal string      |
| `to_str(b: bool) -> str`        | bool → `"true"` / `"false"`       |
| `to_str(s: str) -> str`         | identity                          |
| `to_i32(s: str) -> i32`         | parse decimal string              |
| `to_i32(n: i64) -> i32`         | truncating cast                   |
| `to_i32(n: u64) -> i32`         | reinterpret                       |
| `to_i32(n: u32) -> i32`         | parse via decimal string          |
| `to_i32(n: f32) -> i32`         | truncate toward zero              |
| `to_i32(n: f64) -> i32`         | truncate toward zero              |
| `to_i32(n: i32) -> i32`         | identity                          |
| `to_i64(s: str) -> i64`         | parse decimal string              |
| `to_i64(n: i32) -> i64`         | sign-extend                       |
| `to_i64(n: u64) -> i64`         | reinterpret                       |
| `to_i64(n: u32) -> i64`         | parse via decimal string          |
| `to_i64(n: f32) -> i64`         | truncate toward zero              |
| `to_i64(n: f64) -> i64`         | truncate toward zero              |
| `to_i64(n: i64) -> i64`         | identity                          |
| `to_u32(s: str) -> u32`         | parse unsigned decimal string     |
| `to_u32(n: i32/i64/u64) -> u32` | convert via decimal string        |
| `to_u32(n: f32/f64) -> u32`     | truncate decimal part via string  |
| `to_u32(n: u32) -> u32`         | identity                          |
| `to_u64(s: str) -> u64`         | parse unsigned decimal string     |
| `to_u64(n: i32) -> u64`         | zero-extend                       |
| `to_u64(n: i64) -> u64`         | reinterpret                       |
| `to_u64(n: u32) -> u64`         | convert via decimal string        |
| `to_u64(n: f32/f64) -> u64`     | truncate decimal part via string  |
| `to_u64(n: u64) -> u64`         | identity                          |
| `to_f32(s: str) -> f32`         | parse decimal string              |
| `to_f32(n: i32/i64/u32/u64) -> f32` | convert through numeric parser |
| `to_f32(n: f64) -> f32`         | convert through string formatting |
| `to_f32(n: f32) -> f32`         | identity                          |
| `to_f64(s: str) -> f64`         | parse decimal string              |
| `to_f64(n: i32) -> f64`         | integer → float conversion        |
| `to_f64(n: i64) -> f64`         | integer → float conversion        |
| `to_f64(n: u32/u64) -> f64`     | convert through numeric parser    |
| `to_f64(n: f32) -> f64`         | convert through string formatting |
| `to_f64(n: f64) -> f64`         | identity                          |
| `to_bool(s: str) -> bool`       | `"true"` → true                   |
| `to_bool(b: bool) -> bool`      | identity                          |
| `to_hex(n: i32) -> str`         | hexadecimal representation        |
| `to_hex(n: i64/u32/u64) -> str` | hexadecimal representation        |
| `to_bin(n: i32) -> str`         | binary representation             |
| `to_oct(n: i32) -> str`         | octal representation              |

### Math

All math functions are **overloaded** — call `abs(x)`, `min(a, b)`, `max(a, b)`
on any supported scalar type.

| Signature                                   | Description                          |
| ------------------------------------------- | ------------------------------------ |
| `abs(n: i32/i64/f32/f64)`                   | absolute value                       |
| `min(a, b)` / `max(a, b)`                   | i32/i64/u32/u64/f32/f64 min & max    |
| `clamp(v, lo, hi)`                          | constrain to `[lo, hi]` (i32/i64/f)  |
| `sign(n) -> i32`                            | `-1`, `0` or `1` (i32/i64/f32/f64)   |
| `is_even(n) / is_odd(n)`                    | parity test (i32/i64)                |
| `gcd(a: i32, b: i32) -> i32`                | greatest common divisor              |
| `lcm(a: i32, b: i32) -> i32`                | least common multiple                |
| `pow(base, exp: i32)`                       | integer/float exponentiation         |
| `factorial(n: i32) -> i64`                  | `n!`                                 |
| `floor(x: f64) / ceil(x: f64)`             | round toward -∞ / +∞                  |
| `round(x: f64) -> f64`                      | round to nearest                     |
| `sqrt(x: f64) -> f64`                       | square root (Newton's method)        |

### Containers

| Signature                                    | Description                       |
| -------------------------------------------- | --------------------------------- |
| `len(s: str) -> i32`                         | string length (compiler builtin)  |
| `len(arr: []i32) -> i32`                     | array length                      |
| `len(arr: []i64) -> i32`                     | array length                      |
| `len(arr: []u32) -> i32`                     | array length                      |
| `len(arr: []u64) -> i32`                     | array length                      |
| `len(arr: []bool) -> i32`                    | array length                      |
| `len(arr: []f32) -> i32`                     | array length                      |
| `len(arr: []f64) -> i32`                     | array length                      |
| `len(arr: []str) -> i32`                     | array length                      |
| `contains(s: str, needle: str) -> bool`      | substring search                  |
| `contains(arr: []i32, v: i32) -> bool`       | array membership                  |
| `contains(arr: []i64, v: i64) -> bool`       | array membership                  |
| `contains(arr: []u32, v: u32) -> bool`       | array membership                  |
| `contains(arr: []u64, v: u64) -> bool`       | array membership                  |
| `contains(arr: []bool, v: bool) -> bool`     | array membership                  |
| `contains(arr: []f32, v: f32) -> bool`       | array membership                  |
| `contains(arr: []f64, v: f64) -> bool`       | array membership                  |
| `contains(arr: []str, v: str) -> bool`       | array membership                  |

### Strings (unprefixed)

| Signature                                                  | Description                |
| ---------------------------------------------------------- | -------------------------- |
| `equals(a, b: str) -> bool`                                | exact match                |
| `find(s, needle: str) -> i32`                              | first index or -1          |
| `last_find(s, needle: str) -> i32`                         | last index or -1           |
| `count(s, needle: str) -> i32`                             | non-overlapping count      |
| `is_empty(s: str) -> bool`                                 | length == 0                |
| `is_blank(s: str) -> bool`                                 | empty or whitespace        |
| `lower(s: str) -> str`                                     | to lowercase (ASCII)       |
| `upper(s: str) -> str`                                     | to uppercase (ASCII)       |
| `reverse(s: str) -> str`                                   | reverse characters         |
| `repeat(s: str, n: i32) -> str`                            | repeat n times             |
| `trim(s: str) -> str`                                      | strip whitespace both ends |
| `trim_left(s) / trim_right(s)`                             | one-sided trim             |
| `pad_left(s, total, pad) / pad_right(...)`                 | left/right padding         |
| `replace(s, old, new_val) / replace_first(...)`            | substring replacement      |
| `split_first(s, delim) / split_rest(s, delim)`             | split on first delim       |
| `char_at(s, i: i32) -> str`                                | single character (or "")   |
| `left(s, n: i32) / right(s, n: i32)`                       | first/last `n` characters  |
| `capitalize(s) -> str`                                     | upper-case first character |
| `title(s) -> str`                                          | upper-case each word       |
| `swapcase(s) -> str`                                       | invert each letter's case  |
| `remove(s, sub) -> str`                                    | delete all occurrences     |
| `compare(a, b) -> i32`                                     | lexicographic `-1/0/1`     |
| `equals_ignore_case(a, b) -> bool`                         | ASCII case-insensitive eq  |
| `contains_ignore_case(s, needle) -> bool`                  | ASCII case-insensitive find|
| `is_digits(s) / is_alpha(s) / is_alnum(s) -> bool`         | whole-string class test    |
| `count_char(s, ch) -> i32`                                 | count of a single char     |
| `index_of_char(s, ch) -> i32`                              | first index of char or -1  |
| `join(arr: []str, sep: str) -> str`                        | concatenate with separator |

### I/O

| Signature                                | Description                |
| ---------------------------------------- | -------------------------- |
| `print(msg: str) -> void`                | write to stdout + newline  |
| `print_raw(msg: str) -> void`            | write to stdout, no newline|
| `exit(code: i32) -> void`                | terminate process          |
| `get_time_ns() -> i64`                   | monotonic nanoseconds      |
| `get_username() -> str`                  | current user name          |
| `get_home_dir() -> str`                  | home directory path        |

### Memory

| Signature                       | Description                       |
| ------------------------------- | --------------------------------- |
| `load8(addr) / load32 / load64` | raw memory reads (compiler intr.) |
| `store_byte(addr, v)`           | raw byte write (compiler intr.)   |

---

## Typed underlying functions (backward compatibility)

Earlier versions exposed type-suffixed names. All of them still exist for
backward compatibility but new code should prefer the unified names above:

```
i32_to_str, i64_to_str, u32_to_str, u64_to_str, f32_to_str, f64_to_str,
bool_to_str, str_to_i32, str_to_i64, str_to_u32, str_to_u64, str_to_f32,
str_to_f64, str_to_bool, i32_to_u64, i64_to_u64, u64_to_i32, u64_to_i64,
int_to_hex,
array_len, array_len_i64, array_len_u32, array_len_u64, array_len_bool,
array_len_f32, array_len_f64, array_len_str, array_contains_i32,
array_contains_i64, array_contains_u32, array_contains_u64,
array_contains_bool, array_contains_f32, array_contains_f64, array_contains_str,
str_equals, str_find, str_last_find, str_count, str_is_empty, str_is_blank,
str_lower, str_upper, str_reverse, str_repeat, str_trim, str_trim_left, str_trim_right,
str_pad_left, str_pad_right, str_replace, str_replace_first,
str_split_first, str_split_rest, str_contains,
str_left, str_right, str_capitalize, str_title, str_swapcase, str_remove,
str_compare, str_equals_ignore_case, str_contains_ignore_case,
str_is_digits, str_is_alpha, str_is_alnum, str_count_char, str_index_of_char,
str_join, int_to_bin, int_to_oct, int_to_hex
```

---

## Files

| File                              | Purpose                              |
| --------------------------------- | ------------------------------------ |
| `main.nov`                        | Public entry — imports everything    |
| `core.nov`                        | core types, `len(str)`, conversions  |
| `math.nov`                        | numeric helpers (abs, min/max, …)    |
| `strings.nov`                     | string utilities                     |
| `aliases.nov`                     | unified-name overloads (this file)   |
| `darwin_arm64.nov`                | macOS Apple Silicon syscalls         |
| `linux.nov` / `linux_*.nov`       | Linux x86-64 / arm64 syscalls        |
| `windows_*.nov`                   | Windows API calls                    |
| `memory_*.nov`                    | per-platform mmap allocator          |
| `tests/`                          | regression tests per platform        |

---

## Testing

```bash
novus tests/unified_api.nov && ./build/<platform>/std_tests_unified_api
novus tests/darwin_arm64.nov && ./build/darwin_arm64/std_tests_darwin_arm64
```

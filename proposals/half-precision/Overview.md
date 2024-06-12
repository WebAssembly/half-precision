# FP16 proposal

## Summary

This proposal introduces a primitive data type and SIMD line data type
for half-precision floating point numbers and a set of instructions for
manipulating them.

## Motivation

The introduction of half-precision floating-point to WebAssembly holds
the potential to enhance the performance and memory efficiency of web-based
computer graphics and AI applications. Its benefits are particularly relevant
in situations where memory bandwidth, storage capacity,
or computational speed are bottlenecks.

### Hardware support
#### Intel
[F16C](https://en.wikipedia.org/wiki/F16C)

[AVX512-FP16](https://networkbuilders.intel.com/solutionslibrary/intel-avx-512-fp16-instruction-set-for-intel-xeon-processor-based-products-technology-guide)

#### Arm
[ARMv8-A NEON fp16](https://developer.arm.com/documentation/den0024/a/AArch64-Floating-point-and-NEON/NEON-and-Floating-Point-architecture/Floating-point)

ARM & ARM64 devices with ARMv8.2 FP16 arithmetics extension, and includes Android phones starting with Pixel 3, Galaxy S9 (Snapdragon SoC), Galaxy S10 (Exynos SoC), iOS devices with A11 or newer SoCs, all Apple Silicon Macs, and Windows ARM64 laptops based with Snapdragon 850 SoC or newer.

#### Lowering
For suggested translation to specific ISA see the [lowering](proposals/half-precision/Lowering.md)

# Types

WebAssembly is extended with a new `f16` SIMD lane type.
It represents IEEE754-2008 binary16 floating-point number.

## SIMD value type extension

### Floating-point interpretations

A new lane shape is added. Each lane is interpreted as an IEEE binary16 floating-point number.

* `f16x8 : v16x8`: Each lane is an `f16`.

The floating-point operations in this specification aim to be compatible with scalar
floating-point operations introduced in this proposal. In particular, the rules about
NaN propagation and default NaN values are the same, and all operations use the
default *roundTiesToZero* rounding mode.

# JavaScript API and F16 Values

F16, like I8 and I16, is not first class value type, so there is no need
to define conversions between JS and Wasm values.

## Instructions

There are basically three type of instructions: memory and SIMD.
All instructions mimic operations available for f32 and f64 types and
replicate their semantic there it is possible.

### Memory

* `f32.load_f16 memarg : [i32] -> [f32]`
* `f32.store_f16 memarg : [i32, f32] -> []`

### SIMD

#### Lane operations

* `f16x8.splat : [f32] -> [v128]`
* `f16x8.extract_lane laneindex : [v128] -> [f32]`
* `f16x8.replace_lane laneindex : [v128, f32] -> [v128]`

#### Vector relation operations

* `f16x8.eq`
* `f16x8.ne`
* `f16x8.lt`
* `f16x8.gt`
* `f16x8.le`
* `f16x8.ge`

#### Vector unary operations

* `f16x8.abs`
* `f16x8.neg`
* `f16x8.sqrt`
* `f16x8.ceil`
* `f16x8.floor`
* `f16x8.trunc`
* `f16x8.nearest`

#### Vector binary operations

* `f16x8.add`
* `f16x8.sub`
* `f16x8.mul`
* `f16x8.div`
* `f16x8.min`
* `f16x8.max`
* `f16x8.pmin`
* `f16x8.pmax`

#### Vector conversions

* `i16x8.trunc_sat_f16x8_s`
* `i16x8.trunc_sat_f16x8_u`
* `f16x8.convert_i16x8_s`
* `f16x8.convert_i16x8_u`
* `f16x8.demote_f32x4_zero`
* `f16x8.demote_f64x2_zero`
* `f32x4.promote_f16x8_low`
* `i16x8.trunc_f16x8_s`
* `i16x8.trunc_f16x8_u`

#### FMA operations
* `f16x8.madd`
* `f16x8.nmadd`

## Binary format
All opcodes have the `0xFC` prefix, which are omitted in the table below.
| instruction                           | opcode         |
| ------------------------------------- | -------------- |
| `f32.load_f16`                        | 0x30           |
| `f32.store_f16`                       | 0x31           |

All opcodes have the `0xFD` prefix, which are omitted in the table below.

| instruction                           | opcode         |
| ------------------------------------- | -------------- |
| `f16x8.splat`                         | 0x120          |
| `f16x8.extract_lane laneindex`        | 0x121          |
| `f16x8.replace_lane laneindex`        | 0x122          |
| `f16x8.abs`                           | 0x130          |
| `f16x8.neg`                           | 0x131          |
| `f16x8.sqrt`                          | 0x132          |
| `f16x8.ceil`                          | 0x133          |
| `f16x8.floor`                         | 0x134          |
| `f16x8.trunc`                         | 0x135          |
| `f16x8.nearest`                       | 0x136          |
| `f16x8.eq`                            | 0x137          |
| `f16x8.ne`                            | 0x138          |
| `f16x8.lt`                            | 0x139          |
| `f16x8.gt`                            | 0x13a          |
| `f16x8.le`                            | 0x13b          |
| `f16x8.ge`                            | 0x13c          |
| `f16x8.add`                           | 0x13d          |
| `f16x8.sub`                           | 0x13e          |
| `f16x8.mul`                           | 0x13f          |
| `f16x8.div`                           | 0x140          |
| `f16x8.min`                           | 0x141          |
| `f16x8.max`                           | 0x142          |
| `f16x8.pmin`                          | 0x143          |
| `f16x8.pmax`                          | 0x144          |
| `i16x8.trunc_sat_f16x8_s`             | 0x145          |
| `i16x8.trunc_sat_f16x8_u`             | 0x146          |
| `f16x8.convert_i16x8_s`               | 0x147          |
| `f16x8.convert_i16x8_u`               | 0x148          |
| `f16x8.demote_f32x4_zero`             | 0x149          |
| `f16x8.demote_f64x2_zero`             | 0x14a          |
| `f32x4.promote_f16x8_low`             | 0x14b          |
| `i16x8.trunc_f16x8_s`                 | 0x14c          |
| `i16x8.trunc_f16x8_u`                 | 0x14d          |
| `f16x8.madd`                          | 0x14e          |
| `f16x8.nmadd`                         | 0x14f          |

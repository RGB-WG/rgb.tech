+++
weight = 30
title = "Scripting"
[extra]
anchor = "scripts"
+++

The following fragment provides an example of how a script for RGB contract
validation may look like, written in RGB assembly (a special version of
AluVM assembly) right insite a rust program, which can compile it into a
binary form for the use in a schema.

```rust
    let code = rgbasm! {
        // SUBROUTINE 2: Transfer validation
        // Put 0 to a16[0]
        put     a16[0],0;
        // Read previous state into s16[0]
        ldp     OS_ASSET,a16[0],s16[0];
        // jump into SUBROUTINE 3 to reuse the code
        jmp     FN_SHARED_OFFSET;

        // SUBROUTINE 1: Genesis validation
        // Set offset to read state from strings
        put     a16[0],0x00;
        // Set which state index to read
        put     a8[1],0x00;
        // Read global state into s16[0]
        ldg     GS_TOKENS,a8[1],s16[0];

        // SUBROUTINE 3: Shared code
        // Set errno
        put     a8[0],ERRNO_NON_EQUAL_IN_OUT;
        // Extract 128 bits from the beginning of s16[0] into a32[0]
        extr    s16[0],a32[0],a16[0];
        // Set which state index to read
        put     a16[1],0x00;
        // Read owned state into s16[1]
        lds     OS_ASSET,a16[1],s16[1];
        // Extract 128 bits from the beginning of s16[1] into a32[1]
        extr    s16[1],a32[1],a16[0];
        // Check that token indexes match
        eq.n    a32[0],a32[1];
        // Fail if they don't
        test;

        // Set errno
        put     a8[0],ERRNO_NON_FRACTIONAL;
        // Put offset for the data into a16[2]
        put     a16[2],4;
        // Extract 128 bits starting from the fifth byte of s16[1] into a64[0]
        extr    s16[1],a64[0],a16[2];
        // Check that owned fraction == 1
        put     a64[1],1;
        eq.n    a64[0],a64[1];
        // Fail if not
        test;
    };
    Lib::assemble::<Instr<RgbIsa<MemContract>>>(&code).expect("wrong unique digital asset script")
```

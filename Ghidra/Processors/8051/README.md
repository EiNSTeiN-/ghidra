# 8051

## Sigma Designs ZW050x

Select `8051:BE:24:zw050x` and the Keil C51 compiler spec for flash
images that use the ZW050x banked memory layout. The processor's 16-bit
program counter is represented in Ghidra's CODE space as a bank number
followed by the 16-bit CPU address. Map the image's supplied ranges as
follows.

| Flash region | CPU addresses | Ghidra CODE addresses |
| --- | --- | --- |
| COMMON | `0000–7fff` | `000000–007fff` |
| Bank 1 | `8000–ffff` | `018000–01ffff` |
| Bank 2 | `8000–ffff` | `028000–02ffff` |
| Bank 3 | `8000–ffff` | `038000–03ffff` |

Direct branches from banked code select the current code bank for upper
addresses and COMMON for lower addresses. Upper addresses encoded in
direct branches from COMMON require separate flow analysis. Computed code
accesses from COMMON and return addresses use the bank bits in SFRPAGE.
Calls and returns push and pop two bytes of internal RAM. Paged MOVX uses
P2 as the high XDATA address byte.

The default compiler model places one-, two-, and four-byte scalar results
in R7, R6:R7, and R4:R5:R6:R7, respectively. Use custom function storage
for a carry result (`bool` in `Carry`) or a three-byte generic pointer
(tag in R3, address high byte in R2, address low byte in R1).
The `keil_generic_ptr` typedef of the unsigned three-byte integer lets the
C printer render a constant tagged XDATA address as `&symbol` when a data
symbol exists at that address. The underlying register value retains the tag.
Use an EXTMEM pointer typedef for a two-byte XDATA-only address.
Keil linker bank entries can be represented as Ghidra thunks after
checking each entry's DPTR immediate
and bank gate against the image. Direct IDATA operands 00–07 alias R0–R7
under the compiler model's register-bank-0 assumption. PSW-driven changes
to the active register bank, interrupt state, and peripheral behavior
require separate analysis.

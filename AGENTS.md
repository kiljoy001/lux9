# AGENTS.md - Coding Guidelines for AI Agents

## Build Commands
```bash
make              # Build kernel (lux9.elf)
make iso          # Create bootable ISO with initrd
make run          # Test in QEMU
make clean        # Clean build artifacts
make test-build   # Test compilation of first kernel file
```

## Test Commands
```bash
cd userspace && ./test_ext4fs.sh     # Test ext4fs server
cd userspace && ./test_fscheck.sh    # Test fscheck utility
```

To run a single test from test_ext4fs.sh:
```bash
cd userspace && timeout 2 ./servers/ext4fs/ext4fs "$TESTFS" < /dev/null
```

To run a single test from test_fscheck.sh:
```bash
cd userspace && ./bin/fscheck -f "$TESTFS"
```

## Code Style Guidelines

### Kernel Code (Plan 9 C)
- Use Plan 9 C style with `u.h` and kernel headers
- Function return types on separate line
- Use Plan 9 types: `uchar`, `ushort`, `uint`, `ulong`, `uvlong`, `vlong`
- Use `nil` instead of `NULL`
- Use `print()`/`fprint()` instead of `printf()`
- Use `malloc()`/`free()` with error checking
- Handle errors with `sysfatal("msg: %r")` where `%r` prints errno string
- Header includes order:
  1. `#include "u.h"`
  2. `#include "../port/lib.h"`
  3. `#include "mem.h"`
  4. `#include "dat.h"`
  5. `#include "fns.h"`

### Userspace Code (Standard C)
- Standard C with POSIX functions
- Use standard headers like `stdio.h`, `stdlib.h`, etc.
- Error handling with `perror()` or return error codes

## Naming Conventions
- Function names: `lowercase` or `snake_case`
- Variables: `lowercase` or `snake_case`
- Constants: `UPPERCASE` or `CamelCase`
- Struct/union tags: `CamelCase`
- Global variables: prefixed with `extern` in headers

## Error Handling
- Kernel: Use `sysfatal()` for fatal errors
- Userspace: Use `perror()` or return error codes
- Check return values of all system calls

## Import Order
1. System headers (`u.h`, `libc.h`)
2. Kernel headers (`mem.h`, `dat.h`, `fns.h`)
3. Local headers

## Formatting
- Indent with tabs (8 spaces)
- Opening brace on same line as function/struct
- Line length: 80 characters preferred
- No space after function names
- Space after keywords (`if`, `for`, `while`)

## Special Considerations
- Kernel code uses Plan 9 types and functions
- Userspace code can use standard C library
- All kernel code must be freestanding (-ffreestanding)
- No SSE/MMX instructions in kernel (-mno-sse -mno-mmx)
- Kernel uses custom linker script (kernel/linker.ld)

## Debug Symbols Table
The kernel outputs debug characters to COM1 serial port (0x3F8) during boot to help diagnose boot issues. Each character represents a specific point in the boot process:

### Main Function Boot Sequence
- 'M' - main function start
- 'm' - mach0init done
- 'b' - ioinit done
- 't' - trapinit0 done
- 'i' - i8250console done
- 'q' - quotefmtinstall done
- 's' - screeninit done
- 'C' - about to cpuidentify
- 'c' - cpuidentify done
- 'M' - about to meminit0
- 'm' - meminit0 done
- 'I' - about to initrd_init
- 'i' - initrd_init done
- 'A' - about to archinit
- '!' - archinit done
- '@' - returned from archinit
- '#' - about to check clockinit
- '^' - past clockinit (arch->clockinit is nil)
- '&' - about to meminit
- '*' - meminit done
- '(' - about to ramdiskinit
- ')' - ramdiskinit done
- '_' - about to confinit
- '+' - confinit done
- 'X' - about to xinit
- 'x' - xinit done
- 'T' - about to trapinit
- 't' - trapinit done
- 'M' - about to mathinit
- 'm' - mathinit done
- 'P' - about to pcicfginit
- 'p' - pcicfginit done
- 'B' - about to bootscreeninit
- 'b' - bootscreeninit done
- 'R' - about to printinit
- 'r' - printinit done
- 'U' - about to cpuidprint
- 'u' - cpuidprint done
- 'N' - about to mmuinit
- 'n' - mmuinit done
- 'I' - about to arch->intrinit
- 'i' - intrinit done
- 'T' - about to timersinit
- 't' - timersinit done
- 'C' - about to arch->clockenable
- 'c' - clockenable done
- 'P' - about to procinit0
- 'p' - procinit0 done
- 'S' - about to initseg
- 's' - initseg done
- 'L' - about to links
- 'l' - links done
- 'D' - about to chandevreset
- 'd' - chandevreset done
- 'A' - about to preallocpages
- 'a' - preallocpages done
- 'G' - about to pageinit
- 'g' - pageinit done
- 'r' - runq initialized
- 'U' - about to userinit
- 'u' - userinit done
- 'Z' - about to schedinit
- 'z' - schedinit done (shouldn't reach here)

### Screeninit Function
- 'S' - screeninit start
- '0' - about to read cursor position
- '1' - cursor position read
- '2' - cursor position calculated
- '3' - checking cursor position bounds
- '4' - checking once flag
- '5' - about to set screenputs
- '6' - screenputs set

### Trapinit0 Function
- 'T' - trapinit0 start
- '0' - about to get vectortable address
- '1' - got vectortable address
- '2' - IDT entries filled
- '3' - IDT limit set
- '4' - IDT base set
- '5' - lidt called

### MMUinit Function
- '1' - mmuinit start
- '2' - double map zapped
- '3' - checking machno
- '4' - about to allocate TSS
- '5' - TSS allocated
- '6' - TSS iomap set
- '7' - TSS IST entries set
- '=' - taskswitch start
- '+' - got TSS pointer
- '-' - setting TSS stack pointers
- '_' - TSS stack pointers set
- 'N' - TSS is nil
- 'F' - before mmuflushtlb
- '~' - after mmuflushtlb

### Mach0init Function
- '0' - mach0init start
- '1' - conf.nmach set
- 'A' - about to set MACHP(0)
- 'B' - MACHP(0) set
- '2' - about to zero Mach structure
- '3' - Mach structure zeroed
- '4' - machno set
- '5' - pml4 set
- '6' - gdt set
- '7' - ticks and ilockdepth set

### Bootargsinit Function
- 'B' - bootargsinit start

These debug symbols can be viewed in QEMU by running `make run` and observing the serial output, or by connecting to the serial port directly.
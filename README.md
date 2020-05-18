# Sample NES Repository
This repo serves as an example of a base repo for starting an NES project.

## File Heirarchy
Files are stored in the following heirarchy. At the project root, there is main.asm.

### Root
`main.asm`

This is the primary assembly file to assemble and link to create your .NES binary.

`ines.cfg`

The linker configuration file for .NES files.

`test.scm`

A test-runner script in guile scheme. Usage is `test.scm name`, where name is a filename prefix in `test/`.
If no name is given, all tests will be run.
It loads the test description, generates `out/test/test.run` output,
feeds it into `soft6502`, parses the output as test results, and shows passes and failures.

`test.cfg`

Used to link test.asm files, specifies the correct memory layout for soft6502

### Test
`test/harness.asm`

The test harness. Individual test files `include` the harness and use its provided SHOW macros to export test data.

`test/core.test`

A test description file for the `core` library. Defines a list of tests.
Can be run with `./test.scm core`
Each test is defined by having 3 paragraphs. The first starts with the line NAME, the second DESCRIPTION, the third ROWS.
Name and Description are pretty strings for the test runner to display, whereas ROWS defines the # of 16-byte rows
of memory to compare actual test results in memory to expected test results in memory inside the simulator.

`test/core_test.asm`

A test file. Includes the test harness, and defines the procedure `RunTests`.
Test results are exported when calling the SHOW macro. SHOW will compare the data
in TEST_EXPECTED with the data in TEST_ACTUAL, for as many rows as specified in the test description file.

### Out
`out/main.dbg`

Additional debug file generated during assembly and linking, for emulators like Mesen to give more detailed debugging.

`out/main.o`

The object file created from assembling `main.asm`

`out/main.nes`

The .NES binary file created from linking `out/main.o`

### Out/Test

`out/test/<name>.o`

The object file created from assembling a test file located at `test/<name>_test.asm`

`out/test/<name>.bin`

The soft6502 binary file created from linking `out/test/<name>.o`

`out/test/test.run`

Generated from test.scm. Can be used with soft6502 to manually explore the 6502 simulator and debug.

### Lib
The directory for all library asm files. Libraries can expect to have access to the symbols in `core.asm` and `stack.asm`.

`lib/core.asm`

Core 6502 functions, with no other library dependencies or hardware-specific dependencies.

`lib/stack.asm`

A software stack with a zeropage stack pointer SP.

`lib/nes.asm`

General-purpose NES library

`lib/mmc1.asm`

Library for using the memory mapper MMC1

### Defs
The directory for all assembler-level definitions. No code or data, so can be freely included without changing the resulting binary

```
  defs/core.def
  defs/mmc1.def
  defs/nes.def
```

Definitions for the corresponding libraries in `lib/`

### Data
The directory for data. Used to store name tables, attr tables, palettes, sprites, strings, music, etc. Also includes the .NES header bytes and the memory layout of the zeropage and bss areas.

`data/header.asm`
Holds the byte layout of the .iNES header. Should be the first bytes of any NES rom binary.

```
  data/attr_table.asm
  data/name_table.asm
  data/integers.asm
  data/palette.asm
  data/sprites.asm
  data/strings.asm
```
Hold some reasonable default data

### Mem
The directory for reserved areas of memory. Libraries that depend on certain addresses in zeropage or general RAM reserve them here.

`mem/core.zp.asm`

Holds the essential zero page variables. These are relied upon by `core`. Note this includes a 1-byte stack pointer, as core
declares all zeropage variables it protects from push/pop.

`mem/nes.zp.asm`

NES-specific zero page variables.

`mem/stack.bss.asm`

Reserves a 0-aligned page of memory for the software stack

`mem/nes.bss.asm`

Reserves nes-specific memory addresses

### Assets
For all third-party assets

`assets/mario.chr `
A CHR bank from Super Mario.

## Compiling
Main can be compiled with `make` or `make main`
Tests can be compiled with `make test`

Or for main:
```
To assemble:
  ca65 -t nes "main.asm" -g -o "out/main.o"
To link:
  ld65 -C "ines.cfg" -o "out/main.nes" --dbgfile "out/main.dbg" "out/main.o"
```

And test:
```
To assemble manually:
  ca65 -t nes "test/name_test.asm" -g -o "out/test/name.o"
To link manually:
  ld65 -C "test.cfg" -o "out/test/name.bin" "out/test/name.o"
```

## Testing
With tests compiled, run
```
  ./test.scm <name>
```

## Running
The bare project should compile as is and load in an emulator showing a simple test pattern screen

## Dependencies
Depends on a bunch of software: ca65, ld65, soft65c02, guile, make, even rg because I'm lazy

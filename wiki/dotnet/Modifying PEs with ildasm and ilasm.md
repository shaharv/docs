## Modifying PE's with ildasm and ilasm

### Preface

For troubleshooting non-trivial issues with instrumentation, or modifying IL code
in a specific PE (`.exe` or `.dll`), the following technique may come handy:

1. Disassembling the PE using `ildasm`, into textual IL code.
2. Modifying the IL code as needed, for example - adding IL code similar to that is added during instrumentation.
3. Assembling the modified IL with `ilasm` into a new PE.
4. Testing the generated PE for the expected functionality.

This technique is very effective for debugging instrumentation issues, and IL in general, since:
1. We can run `peverify` on the modified PE, which will verify the PE for correctness and will alert on erroneous IL code.
2. It does not require modifying the agent code.

### Steps

#### 1. Disassembling the PE into textual IL

- Run `ildasm` on the PE to disassemble: `ildasm myprog.exe`.
- In `ildasm` GUI, click `File -> Dump`.
- Check the following options:
  - `Encoding: ANSI`
  - `Dump IL Code -> Token Values`
  - `Dump IL Code -> Actual Bytes`
- Click OK, and choose destination for the `.il` text file.

#### 2. Assembling the IL back into PE

- Run `ilasm` on the disassembled IL: `ilasm myprog.il`.
- For creating a DLL, add the flag `/dll`.

#### 3. Verifying the PE

- Run `peverify` on the new PE: `peverify /IL myprog.exe`.
- Watch for warnings and errors in the modified code.

### Links

- ildasm.exe (Microsoft Docs):  
  https://docs.microsoft.com/en-us/dotnet/framework/tools/ildasm-exe-il-disassembler

- ilasm.exe (Microsoft Docs):  
  https://docs.microsoft.com/en-us/dotnet/framework/tools/ilasm-exe-il-assembler

- peverify.exe (Microsoft Docs):  
  https://docs.microsoft.com/en-us/dotnet/framework/tools/peverify-exe-peverify-tool

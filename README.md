# String Extraction & Display COFF Information

`strings.exe` extracts strings from binary data. You can turn on the `Filter Strings` option, which removes parts like:
```
Φü¬;µò┤τÄï
ΦüèkΦü₧k
φÿÇ?φÿò?
φÿÇ+φÿö+
Ω│╜T∩╝│∩╜ü
Ω╖£∩╛Äδºî\
Ω╡┤aΩ╖╣a
Ω┤Ç:Ω╕╕:δéî
```
You're also able to enter the minimum string length (number of characters), which can improve the overview, by removing parts like:
```
Fxt
F:H
-fy
fyl
fy-
fZ~
FzG
```
You should not set it too high, as this will otherwise remove lines that you may want. Default is set to `4`.

`Recurse` - `ON` means that it'll process all subdirectories, e.g. `C:\Windows\System32\DriverStore`. If it's set to `OFF`, subdirectories won’t be processed and only files located in `C:\Windows\System32\` will be processed.

`dumpbin.exe` displays information about Common Object File Format (COFF) binary files. The `Preconfigured Flags` list has the following meaning:

__If set to `OFF`, it uses these flags:__
`/HEADERS` - File headers (COFF + PE headers)
`/ARCHIVEMEMBERS` - For .lib files, list contents
`/EXPORTS` -Exported functions/symbols
`/IMPORTS` - Imported functions and DLLs
`/SYMBOLS` - Symbol table (COFF debug info)
`/LINENUMBERS` - Line numbers (if available)
`/RAWDATA` - Raw section data
`/RELOCATIONS` - Base relocations
`/TLS` - TLS directory (Thread-Local Storage)
`/SUMMARY` - Section sizes and usage
`/CLRHEADER` - .NET managed code header (if present)
`/LOADCONFIG` - Load configuration directory
`/DIRECTIVES` - Linker directives
`/PDATA` - Exception data (used for SEH)
`/DEBUGDIRECTORIES` - Debug-related directories
`/FPO` - Frame pointer omission data
`/PDBPATH` - Embedded PDB path (debug info)
-# Info was taken from [DUMPBIN options](https://learn.microsoft.com/en-us/cpp/build/reference/dumpbin-options?view=msvc-170)

__If set to `ON`, it uses these flags, which reduce the size by a lot:__
`/ARCHIVEMEMBERS`
`/CLRHEADER`
`/DEPENDENTS`
`/EXPORTS`
`/IMPORTS`
`/SUMMARY`
`/SYMBOLS`
`/DIRECTIVES`

`One File` option writes all strings into one file, instead of creating a new file for each. It uses the `-f` flag, which shows the filename for the string, e.g.:
```cmd
C:\Windows\System32/ntoskrnl.exe,ReservedCpuSets
```
`P` lets you set a process ID (PID) - this will only work while `One File` is turned off.

__Additional information:__
- Turning off `Preconfigured Dumpbin Flags` increases the execution time by a lot - should only be changed if single files are extracted
- It is recommended to use `One File`, as it speeds up the process and it'll be easier to search for strings
- String length size should stay at `3-5`

It is recommended to use the default preset and let it run once, which will extract your whole `System32` folder. Afterwards you can use this file to check for the existence of any string (doesn't mean that it's a DWORD). Use WPR or IDA for it (e.g. <#1371224441568231516>). This tool should be used to search the binary file for a specific string or to check whether a string exists anywhere.

__References:__
> https://learn.microsoft.com/en-us/cpp/build/reference/dumpbin-options?view=msvc-170
> https://learn.microsoft.com/en-us/sysinternals/downloads/strings
> https://visualstudio.microsoft.com/ (`mspdbcore.dll`, `tbbmalloc.dll`, `link.exe`, `dumpbin.exe`)
> https://github.com/5Noxi/strings2 (`strings.exe`)
> https://github.com/glmcdona/binary2strings (python module version)


# Strings2 CL Information

Strings2 is a Windows command-line tool for extracting strings from binary data. On top of the classic Sysinternals strings approach, this tool includes:
- Multi-lingual string extraction, such as Russian, Chinese, etc.
- Machine learning model filters out junk erroneous string extractions to reduce noise.
- String extractions from process memory.
- Recursive and wildcard filename matching.
- Json output option for automation integration. (Also see python module version [binary2strings](https://github.com/glmcdona/binary2strings))

I also recommend looking at [FLOSS](https://github.com/mandiant/flare-floss) from Mandiant a cross-platform string extraction solver with a different set of features.

## Installation
Download the [latest release binary](https://github.com/5Noxi/strings2/releases).

## Example Usage

Dump all strings from `malware.exe` to stdout:

* ```strings2 malware.exe```

Dump all strings from all `.exe` files in the `files` folder to the file `strings.txt`:
* ```strings2 ./files/*.exe > strings.txt```

Dump strings from a specific process id, including logging the module name and memory addresses of each match:
* ```strings2 -f -s -pid 0x1a3 > process_strings.txt```

Extract strings from `malware.exe` to a json file:
* ```strings2 malware.exe -json > strings.json```

## Documentation

```strings.exe (options) file_pattern```

* `file_pattern` can be a folder or file. Wildcards (`*`) are supported in the filename parts - eg `.\files\*.exe`.

|Option|Description|
|--|--|
|-r|Recursively process subdirectories.|
|-f|Prints the filename/processname for each string.|
|-F|Prints the full path and filename for each string.|
|-s|Prints the file offset or memory address span of each string.|
|-t|Prints the string type for each string. UTF8, or WIDE_STRING.|
|-wide|Prints only WIDE_STRING strings that are encoded as two bytes per character.|
|-utf|Prints only UTF8 encoded strings.|
|-a|Prints both interesting and not interesting strings. Default only prints interesting non-junk strings.|
|-ni|Prints only not interesting strings. Default only prints interesting non-junk strings.|
|-e|Escape new line characters.|
|-l [num_chars]|Minimum number of characters that is a valid string. Default is 4.|
|-b [start]\(:[end]\)|Scan only the specified byte range for strings. Optionally specify an end offset as well.|
|-pid [pid]|The strings from the process address space for the specified PID will be dumped. Use a '0x' prefix to specify a hex PID.|
|-system|Dumps strings from all accessible processes on the system. This takes awhile.|
|-json|Writes output as json. Many flags are ignored in this mode.|

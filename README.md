# Tilibs

## Windows Driver Kit 10 (Windows 10/11 v.22621 x64)

After two days of experimenting with idaclang I found a way to compile the WDK without obscure errors.

```
idaclang.exe --idaclang-tilname D:\wdk10.til --idaclang-log-target --idaclang-tildesc "Windows Driver Kit 10 (Windows 10/11 v.22621 x64)" -target x86_64-pc-win32 -x c++ -D_WIN32_ -D_WIN32 -D_AMD64_ -D_NTIFS_ -D_NDIS_ -D_M_AMD64 -D_inline=inline -D__inline=inline -D__forceinline=inline -isysroot "C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0"  -I"C:\Program Files (x86)\Windows Kits\10\include\10.0.22621.0\shared"  -I"C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\km" -I"C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\ucrt" -I"C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\um" -I"C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\winrt" -I"C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\cppwinrt\winrt" -I"C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\km\crt" -I"E:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.34.31933\include" "C:\Program Files (x86)\Windows Kits\10\Include\10.0.22621.0\km\test.h"
```

The initial experiments were applied to the big header file after parsing the WDK directory with the following python-script.

```python
import os

inc_open = '#include "' 
inc_close = '"'

def get_paths():
    root = "C:\\Program Files (x86)\\Windows Kits\\10\Include\\10.0.22621.0\\km"
    paths = []
    for root, subdirs, files in os.walk(root):
        for name in files:
            if name.endswith(".h") or name.endswith(".hpp"):
                paths.append(os.path.join(root, name))   
    return paths


def main():
    paths = get_paths()
    for filename in paths:
        filename = filename.removeprefix("C:\\Program Files (x86)\\Windows Kits\\10\Include\\10.0.22621.0\\km\\")
        print(inc_open + filename + inc_close)
        
main() 
```

Unfortunately it's not worked. The header file contained approx 500 includes with excessive nesting, so I decided to reduce a header file manualy. The final version contain only a few headers, which types are very common in windows drivers. 

```
#include "wdm.h"
#include "ntddk.h"
#include "ndis.h"
#include "ntifs.h"
#include <windef.h>
#include <winternl.h>
```

The final version contain now:

**Total 3012 symbols, 3302 types, 9347 macros**

One more interesting observation was connected with the tilib itself. Tilib also capable to create *.til* files, but compared to idaclang tilib worse, but not to much.

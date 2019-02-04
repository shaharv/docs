### Access C++ class from C#

A C++ class could be made accessible from C#, or any other .NET languague, using the following simple procedure:
1. Creating a C++/CLI project and implementing a simple wrapper to the C++ class.  
2. Making the C++/CLI wrapper class available to the C# project, by adding a reference to its DLL.

### Step 1: Create a C++/CLI wrapper class and project
- Run Visual Studio 2015 as Administrator.
- Create a new project: `Visual C++ -> CLR -> Class Library`. Let's call it `CLRClassWrapper`.
- Add the source and header files of the C++ class: in Solution Explorer, right click `Source Files -> Add -> Existing Item`.
- Add `#include "stdafx.h"` to the added source files, if missing.
- Edit the `CLRClassWrapper.h` header file and implement the wrapper class.  
  For example:

```c++
#include "TestClass.h" // the header file of our C++ class

namespace CLRClassWrapper 
{
    public ref class NativeClassWrapper 
    {
    	TestClass* m_nativeClass;

    public:
    	NativeClassWrapper() { m_nativeClass = new TestClass(); }
    	~NativeClassWrapper() { this->!NativeClassWrapper(); }
    	!NativeClassWrapper() { delete m_nativeClass; }
    	void test()
    	{
    	   m_nativeClass->DoCSharpStuffWithCOM();
    	}
    };
}
````

- Build the project for `x64`.

### Step 2: Connect to the C# project

- In the C# project, in Solution Explorer, right click `References` and `Add Reference`.
- Click the `Browse` button (window bottom), and select the `CLRClassWrapper.dll` that was just built.
- Modify your C# code to use the wrapper class.  
  For example:
```csharp
using CLRClassWrapper;

// ...
static void Main(string[] args)
{
    NativeClassWrapper p = new NativeClassWrapper();
    p.test();
```
- Build the project for `x64`, and run.

**Note:** The CLR wrapper and C# projects must target the same .NET version.

### Useful Links

- C++/CLI (Wikipedia)  
  https://en.wikipedia.org/wiki/C%2B%2B/CLI

- C++/CLI migration primer  
  https://msdn.microsoft.com/en-us/library/ms235289.aspx
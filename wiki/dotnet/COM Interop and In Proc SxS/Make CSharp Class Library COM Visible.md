### Make C# Class Library COM Visible

For a C# class to be made visible to COM, the following basic requirements apply:
- The class must be declared public.
- It must implement an interface.
- It must implement a public default constructor.

The interface is used to expose the class functionality to COM. Only methods declared within the interface can be made COM visible.
Optionally, the class may implement an additional interface for events.

For example, the following class is suitable for COM:

```csharp
namespace CSharpClassLib
{
    public interface IMyCOMTestClassInterface
    {
        void testPrint();
    }

    public class MyCOMTestClass : IMyCOMTestClassInterface
    {
        public MyCOMTestClass()
        {
        }

        public void testPrint()
        {
            Console.WriteLine("Hello from C#!");
            Console.WriteLine("CLR Version = " + Environment.Version.ToString());
        }
    }
}
```

The class could be made COM visible with the following modifications:

```csharp
using System.Runtime.InteropServices;

namespace CSharpClassLib
{
    [Guid("6b08a84b-a24c-489f-ad24-b6ba430663ec")]
    [ComVisible(true)]
    public interface IMyCOMTestClassInterface
    {
        void testPrint();
    }

    [Guid("a420b9f4-4dd7-4e84-9df4-76996fe5be54"),
        ClassInterface(ClassInterfaceType.None),
        ComSourceInterfaces(typeof(IMyCOMTestClassInterface))]
    [ComVisible(true)]
    [ProgId("MyProgram.MyCOMTestClass")]
    public class MyCOMTestClass : IMyCOMTestClassInterface
    {
        // ....
    }
}
```

Where:
- COM Interop attributes are provided by the `System.Runtime.InteropServices` package.
- Classes and interfaces should be assigned with a GUID: `[Guid(guid-string)]`.  
A GUID (or UUID) is a random 128-bit number used as a "sufficiently-unique" identifier. For generating GUID's, you may use Visual Studio 2015 generator via `Tools -> Create GUID`, or an [online GUID generator](https://www.guidgenerator.com/online-guid-generator.aspx).
- The `[ComVisible(true)]` is required for exposing classes and interfaces to COM.  
Alternatively, the entire assembly could be made COM visible, by checking `Project Properties -> Application -> Assembly Information -> Make Assembly COM-Visible` (in which case the individual `ComVisible` attributes are not needed).
- The class should be decorated with the `ClassInterface`, `ComSourceInterfaces` attributes for declaring the COM interfaces.
- The class may be tagged with a [ProgId](https://msdn.microsoft.com/en-us/library/windows/desktop/dd542719(v=vs.85).aspx) key, which is useful for class identification.

**Important:**

- The project must be built for `x64` or `x86`, but not for `AnyCPU`.  
  DLL's targeting `AnyCPU` will not be recognized by COM.

### Register COM DLL

For the DLL to be available to COM, it has to be registered.

#### From Visual Studio:
- Run Visual Studio 2015 as Administrator.
- In the class library project, check `Project Properties -> Build -> Register for COM interop`.
- Build.

#### From command line:
- Open command prompt as Administrator.
- `cd` into the appropriate .NET installation folder, depending on the project target .NET version.  
For example: `cd\Windows\Microsoft.NET\Framework64\v2.0.50727`
- Run: `regasm /tlb <full path to DLL>`

### Useful Links

- Example COM Class (C# Programming Guide)  
  https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interop/example-com-class

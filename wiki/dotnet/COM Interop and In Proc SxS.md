## COM Interop and In Proc SxS

### In Proc SxS

"In-Process Runtime Side-by-Side" (or **_In Proc SxS_**) is the capability of loading
and running several .NET runtimes in the same process.

This feature was introduced in .NET version 4.0, for the purpose of maximizing compatibility of applications with independent managed components that are loaded by, and communicate with, a native layer (namely COM).

In .NET versions prior to 4.0, each process was limited to only one runtime version, and was bound to that 
runtime (CLR) for remainder of that process lifetime.

Note this feature is only relevant for mixed assemblies: a pre-4.0 .NET assembly, that is loaded from program running on .NET 4.0 or later, will run with the already loaded V4 CLR.

### COM Interop

COM Interop is the CLR ability that allows .NET and COM objects to interact with each other.  
For more details, refer to "Useful Links" below.

### In Proc SxS and COM - Sample Program

For creating a sample program in which CLRs of different versions are loaded into the same process, we'd need to create a mixed mode assembly targeting an older .NET version (pre-4.0), and load it from a .NET 4.0 assembly.
The procedure would consist of the following steps:
1) Create a C# class library, targeting .NET 2.0 (or 3.5).
2) Expose the C# class to COM:  
   https://github.com/takipi-dev/takipi/wiki/Make-CSharp-Class-Library-COM-Visible
3) Consume the C# COM class from a C++ class:  
   https://github.com/takipi-dev/takipi/wiki/Use-C%23-Classes-from-C%CB%96%CB%96-Using-COM
4) Create a C++/CLI wrapper for the C++ class:  
   https://github.com/takipi-dev/takipi/wiki/Use-C%CB%96%CB%96-class-from-C%23
5) Create a C# test project targeting .NET 4.0 or higher, and use the wrapper class, as described in 4.

### Useful Links - In Proc SxS

- CLR Inside Out - In-Process Side-by-Side  
  https://msdn.microsoft.com/en-us/magazine/ee819091.aspx

- In-Proc SxS and Migration Quick Start  
  https://blogs.msdn.microsoft.com/dotnet/2010/06/23/in-proc-sxs-and-migration-quick-start

- In-Process Side-by-Side Profiling (VS2010)  
  https://msdn.microsoft.com/en-us/library/ee461607(v=vs.100).aspx

- Loading multiple CLR Runtimes (InProc SxS) – Sample Code  
  https://blogs.msdn.microsoft.com/carlos/2013/08/23/loading-multiple-clr-runtimes-inproc-sxs-sample-code

- New stuff in Profiling API for upcoming CLR 4.0 (under "In-process side-by-side CLR instances")  
  https://blogs.msdn.microsoft.com/davbr/2008/11/10/new-stuff-in-profiling-api-for-upcoming-clr-4-0

### Useful Links - COM

- Introduction to COM Interop (Visual Basic)  
  https://docs.microsoft.com/en-us/dotnet/visual-basic/programming-guide/com-interop/introduction-to-com-interop

- Introduction to COM - What It Is and How to Use It  
  https://www.codeproject.com/Articles/633/Introduction-to-COM-What-It-Is-and-How-to-Use-It

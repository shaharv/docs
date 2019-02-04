### Calling a C# class from C++ using COM

This example builds on the COM C# class `MyCOMTestClass`, which is defined [here](https://github.com/takipi-dev/takipi/wiki/Make-CSharp-Class-Library-COM-Visible).  
It demonstrates calling a COM class, for which we are only given:
- The class `ProgID` identifier.
- The signature of a class method to be called, in our case `public void testPrint()`.

That is, we don't have the class GUID, or the interface IID to query (such as `ICorProfilerInfo`).

_TestClass.h_:
````c++
#pragma once

class TestClass
{
public:
	void DoCSharpStuffWithCOM();
};
````
_TestClass.cpp_:
````c++
#include "atlbase.h"
#include "TestClass.h"

void TestClass::DoCSharpStuffWithCOM()
{
	CLSID clsId;

	// Retrieve CLSID from the specified ProgID
	HRESULT hRes = CLSIDFromProgID(OLESTR("MyProgram.MyCOMTestClass"), &clsId);

	if (FAILED(hRes))
	{
		return;
	}

	// Create an instance of the class described by clsId
	IDispatch * pDispatch = NULL;
	hRes = CoCreateInstance(clsId, NULL, CLSCTX_INPROC_SERVER,
		IID_IDispatch, (void **)&pDispatch);

	if (FAILED(hRes))
	{
		return;
	}

	OLECHAR *sMethodName = L"testPrint";
	DISPID idTestPrintMethod;

	hRes = pDispatch->GetIDsOfNames(IID_NULL, &sMethodName, 1, LOCALE_SYSTEM_DEFAULT, &idTestPrintMethod);

	if (FAILED(hRes))
	{
		return;
	}

	DISPPARAMS pParams;
	memset(&pParams, 0, sizeof(DISPPARAMS));
	pParams.cArgs = 0;

	hRes = pDispatch->Invoke(
		idTestPrintMethod,
		IID_NULL,
		LOCALE_SYSTEM_DEFAULT,
		DISPATCH_METHOD,
		&pParams,
		NULL, /*&VarResult*/
		NULL, /*&pExcepInfo*/
		NULL  /*&puArgErr*/
	);

	// Release the object
	pDispatch->Release();
}
````

_main.cpp_:
````c++
#include "TestClass.h"
#include "atlbase.h"
#include <iostream>
#include <conio.h>

int main()
{
	// Initialize COM
	HRESULT hRes = CoInitializeEx(NULL, COINIT_APARTMENTTHREADED);

	if (FAILED(hRes))
	{
		return 0;
	}	
	
	TestClass o;
	o.DoCSharpStuffWithCOM();

	CoUninitialize();

	std::cout << "Press any key...";
	_getch();
	
	return 0;
}
````

Test output would vary, depending on the .NET framework the `MyCOMTestClass` assembly is targeting.  
For example, if the assembly is built against .NET 2.0:

````
Hello from C#!
CLR Version = 2.0.50727.8784
Press any key.
````

### Useful Links

- COM Interop Part 1: C# Client Tutorial  
  https://msdn.microsoft.com/en-us/library/aa645736(v=vs.71).aspx

- COM Interop Part 1: C# Server Tutorial  
  https://msdn.microsoft.com/en-us/library/aa645738(v=vs.71).aspx

- Accessing Members Through IDispatch  
  https://msdn.microsoft.com/en-us/library/windows/desktop/ms221694(v=vs.85).aspx
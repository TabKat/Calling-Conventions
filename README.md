based on [article](https://www.codeproject.com/Articles/1388/Calling-Conventions-Demystified)

## Example

```cpp
int sumExample (int a, int b)
{
    return a + b;
}
int c = sum (2, 3);

```

## C calling convention (__cdecl)

1. **Arguments** are passed from **right to left**, and placed on **the stack**
2. Stack **cleanup** is **performed by the caller**
3. Function name is decorated by prefixing it with an underscore character **'_'**

### assembly

```c++
// 1) push arguments to the stack, from right to left
push        3    
push        2    

// 2) call the function (look at the prefix '_')
call        _sumExample 

// 3) cleanup the stack by adding the size of the arguments to ESP register
add         esp,8 

// 4) copy the return value from EAX to a local variable (int c)
mov         dword ptr [c],eax
```

## Standard calling convention (__stdcall)

1. Arguments are passed from **right to left**, and placed on **the stack**.
2. Stack **cleanup** is performed by **the called function**.
3. Function name is decorated by prepending an underscore character and appending a **@** character and **the number of bytes of stack space required** (example: _sumExample **@8**)

### caller function

```cpp
// push arguments to the stack, from right to left
push        3    
push        2    

// call the function
call        _sumExample@8

// copy the return value from EAX to a local variable (int c)  
mov         dword ptr [c],eax
```

### called function

```cpp
// function prolog goes here (the same code as in the __cdecl example)

// return a + b;
mov         eax,dword ptr [a] 
add         eax,dword ptr [b] 

// function epilog goes here (the same code as in the __cdecl example)

// cleanup the stack and return
ret         8
```

## Fast calling convention (__fastcall)

Fast calling convention indicates that the **arguments should be placed in registers, rather than on the stack**, whenever possible. This reduces the cost of a function call, because operations with registers are faster than with the stack.

We can also use the compiler option **/Gr** to specify **__fastcall** for all functions not explicitly declared with some other calling convention.

1. **The first two arguments** that require **32 bits** or less are placed into registers **ECX** and **EDX**. The rest of them are pushed on the stack from right to left
2. Arguments are popped from the stack by the called function
3. Function name is decorated by by prepending a **@** character and appending a **@ and the number of bytes (decimal) of space required by the arguments**

```cpp
// put the arguments in the registers EDX and ECX
mov         edx,3 
mov         ecx,2 

// call the function
call        @fastcallSum@8

// copy the return value from EAX to a local variable (int c)  
mov         dword ptr [c],eax
```

Set the compiler option **/Gr**, and compare the execution time. I didn't find __fastcall to be any faster than other calling conventons, but you may come to different conclusions.

## Thiscall

Thiscall is the default calling convention for calling member functions of **C++ classes** (except for those with a variable number of arguments)

1. Arguments are passed from **right to left**, and placed on the stack. **this** is placed in **ECX**.
2. Stack **cleanup** is performed **by the called function**

### Example

```cpp
struct CSum
{
    int sum ( int a, int b) {return a+b;}
};
```

### Assembly

```cpp
push        3
push        2
lea         ecx,[sumObj]
call        ?sum@CSum@@QAEHHH@Z            ; CSum::sum
mov         dword ptr [s4],eax
```

### Struct CSum

```cpp
push        ebp
mov         ebp,esp
sub         esp,0CCh
push        ebx
push        esi
push        edi
push        ecx
lea         edi,[ebp-0CCh]
mov         ecx,33h
mov         eax,0CCCCCCCCh
rep stos    dword ptr [edi]
pop         ecx
mov         dword ptr [ebp-8],ecx
mov         eax,dword ptr [a]
add         eax,dword ptr [b]
pop         edi
pop         esi
pop         ebx
mov         esp,ebp
pop         ebp
ret         8
```

## Conclusion
- **__cdecl** is the default calling convention for **C and C++** programs. **The advantage of this calling convetion is that it allows functions with a variable number of arguments to be used**. **The disadvantage is that it creates larger executables**.
- **__stdcall** is used to call **Win32 API** functions. It does not allow functions to have a variable number of arguments.
- **__fastcall** attempts **to put arguments in registers**, rather than on the stack, thus making function **calls faster**.
- **Thiscall** calling convention is the **default calling convention used by C++** member functions that **do not use variable arguments**.

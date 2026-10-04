# miniLua - Lua for Pascal

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)

miniLua is a Pascal binding for [Lua 5.5](https://www.lua.org/), providing both low-level API access via `LuaAPI.pas` and high-level object-oriented wrappers via `LuaClasses.pas`. This package is compatible with both **FreePascal (FPC)** and **Delphi**.

## Features

- Support for Lua 5.5
- Works with FreePascal (Lazarus) and Delphi
- Low-level API wrapper (`LuaAPI.pas`) - Direct bindings to Lua C API
- High-level OOP wrapper (`LuaClasses.pas`) - Easy to use classes and helpers for embedding Lua
- Cross-platform support (Windows, Linux, macOS depending on Lua libraries)
- Simple and clean interface

## Installation

### For Lazarus (FPC)

1. Open `minilua.lpk` in Lazarus
2. Compile and install the package
3. Add the package to your project

### For Delphi

1. Open `minilua.dpk` in Delphi
2. Compile and install the package
3. Add the package to your project

### Manual Compilation

Add `LuaAPI.pas` and `LuaClasses.pas` to your project directly.

## Usage

### Basic Example - Running Lua Scripts

Here's a simple example of how to initialize Lua and execute a script:

```pascal
uses
  SysUtils, LuaAPI, LuaClasses;

var
  Lua: TLua;
  Output: string;
begin
  Lua.Init; // Initialize Lua (SafeMode enabled by default)
  try
    // Execute a simple Lua script
    if Lua.State.RunString('return "Hello from Lua!"', Output) then
      WriteLn(Output)
    else
      WriteLn('Error running script');
  finally
    Lua.Close;
  end;
end.
```

### Registering Pascal Functions with Lua

You can expose Pascal functions to Lua scripts:

```pascal
uses
  SysUtils, LuaAPI, LuaClasses;

function AddNumbers(L: Plua_State): Integer; cdecl;
begin
  // Get arguments from Lua stack (arg1, arg2)
  Result := 1; // Return 1 value
  L.PushInteger(L.ToInteger(1) + L.ToInteger(2));
end;

var
  Lua: TLua;
  Output: string;
begin
  Lua.Init;
  try
    // Register the function as 'add' in Lua
    Lua.State.RegisterGlobal('add', @AddNumbers);
    
    // Call it from Lua
    if Lua.State.RunString('local result = add(5, 3); return result', Output) then
      WriteLn('5 + 3 = ' + Output)
    else
      WriteLn('Error');
  finally
    Lua.Close;
  end;
end.
```

### Exposing Pascal Objects to Lua

Using `TLuaObject`, you can easily expose object methods to Lua:

```pascal
uses
  SysUtils, Classes, LuaAPI, LuaClasses;

type
  TCalculator = class(TLuaObject)
  public
    function Multiply(L: Plua_State): Integer; cdecl;
  end;

function TCalculator.Multiply(L: Plua_State): Integer; cdecl;
begin
  Result := 1;
  L.PushNumber(L.ToNumber(1) * L.ToNumber(2));
end;

var
  Lua: TLua;
  Calc: TCalculator;
  Output: string;
begin
  Lua.Init;
  try
    Calc := TCalculator.Create;
    try
      // Register object methods in a Lua table
      Lua.State.Register('calc', Calc);
      // Register specific method
      Lua.State.Register('calc', 'multiply', Calc, @TCalculator.Multiply);
      
      // Use from Lua
      if Lua.State.RunString('return calc.multiply(6, 7)', Output) then
        WriteLn('6 * 7 = ' + Output)
      else
        WriteLn('Error');
    finally
      Calc.Free;
    end;
  finally
    Lua.Close;
  end;
end.
```

### Working with Lua Values

The `TLuaHelper` record helper provides convenient methods to work with the Lua stack:

```pascal
var
  L: Plua_State;
begin
  // Push values
  L.PushString('Hello');
  L.PushInteger(42);
  L.PushNumber(3.14);
  L.PushBoolean(True);
  L.PushNil;
  
  // Check types
  if L.IsString(1) then
    WriteLn(L.ToString(1));
  if L.IsInteger(2) then
    WriteLn(IntToStr(L.ToInteger(2)));
  if L.IsNumber(3) then
    WriteLn(FloatToStr(L.ToNumber(3)));
  if L.IsBoolean(4) then
    WriteLn(BoolToStr(L.ToBoolean(4)));
  
  // Pop values
  L.Pop(5);
end;
```

## Project Structure

- `LuaAPI.pas` - Low-level Lua 5.5 C API bindings (lua.h, lauxlib.h, lualib.h)
- `LuaClasses.pas` - High-level object-oriented interface for embedding Lua in Pascal applications
- `minilua.pas` - Package unit (for package compilation)
- `minilua.lpk` - Lazarus package file
- `minilua.dpk` / `minilua.dproj` - Delphi package files
- `lib/` - Lua library files
- `demos/` - Demo applications for Delphi and Lazarus

## Requirements

- FreePascal 3.0+ or Delphi (2009+ recommended for Unicode support)
- Lua shared library (`lua55.dll`, `liblua55.so`, `liblua55.dylib`, `pluto.dll`, `pluto.so`) or static library

## Documentation

For more information about Lua scripting, visit [Lua 5.5 Manual](https://www.lua.org/manual/5.5/).

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

## Credits

- Original C headers and Lua libraries: TeCGraf / [Lua.org](https://www.lua.org)
- Original Pascal translation: Lavergne Thomas
- Updates: Bram Kuijvenhoven, Egor Skriptunoff, Vladimir Klimov, Malcome@Japan, Zaher Dirkey
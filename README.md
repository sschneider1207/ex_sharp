# ExSharp

Call C# code from Elixir!

Note: requires C# Interactive (csi.exe) to be installed and included in your path

## Installation

If [available in Hex](https://hex.pm/docs/publish), the package can be installed as:

  1. Add ex_sharp to your list of dependencies in `mix.exs`:

        def deps do
          [{:ex_sharp, "~> 0.0.1"}]
        end

  2. Ensure ex_sharp is started before your application:

        def application do
          [applications: [:ex_sharp]]
        end
        
## Usage
  
Using the following C# code in `Foo.csx`:
 
    using System;
    using ExSharp;
    using static ExSharp.Runner;
    
    [ExSharpModule("Foo")]
    public static class Foo 
    {
      [ExSharpFunction("pi", 0)]
      public static ElixirTerm Pi(ElixirTerm[] argv, int argc) 
      {
        return ElixirTerm.MakeDouble(3.14159);
      }
      
      [ExSharpFunction("echo", 1)]
      public static ElixirTerm Baz(ElixirTerm[] argv, int argc) 
      {
        var str = ElixirTerm.GetUTF8String(argv[0]);
        return ElixirTerm.MakeUTF8String($"from csharp: {str}");
      }
    }
    
    Init(typeof(Foo), typeof(Bar), etc...);
        
Create an `ExSharp.Roslyn` process with it's first argument being the path to said `.csx` script.
It is recommended that this process be supervised.

      Path.expand("../priv/Foo.csx", __DIR__)
      |> ExSharp.Roslyn.start_link
  
The following functions will become available at runtime:
  
  1. Foo.pi/0
  2. Foo.echo/1

## Supported Types

So far the following Elixir types are able to be retrieved/returned from C# code:

  * Integer
  * Float
  * Atom
  * Byte String
  * PID
  * Reference
  * Empty List
  * Tuple
  * Byte List ("String")
  * List
  * Export
  
## C# API

### Declare Module and Functions

    [ExSharpModule(module_name)]
    public static class ModuleName
    {
      [ExSharFunction(function_name, arity)]
      public static ElixirTerm Foo(ElixirTerm[] argv, int argc) 
      {
        ...
      }
      
      [ExSharFunction(function_name, arity)]
      public static void Foo(ElixirTerm[] argv, int argc) 
      {
        (functions that return void will return `:ok`)
        ...
      }
    }

### Type Conversions

  * Byte:
  
      ```
      byte? b = ElixirTerm.GetByte(argv[0]);
      ElixirTerm t = ElixirTerm.MakeByte(b);
      ```
      
  * Int:
      
      ```
      int? i = ElixirTerm.GetInt(argv[0]);
      ElixirTerm t = ElixirTerm.MakeInt(i);
      ```
      
  * Double:  
      
      ```
      double? d = ElixirTerm.GetDouble(argv[0]);
      ElixirTerm t = ElixirTerm.MakeDouble(d);
      ```
      
  * Atom:
      
      ```
      string a = ElixirTerm.GetAtom(argv[0]);
      ElixirTerm t = ElixirTerm.MakeAtom(a);
      ```
      
  * Byte String: 
      
      ```
      string s = ElixirTerm.GetUTF8String(argv[0]);
      ElixirTerm t = ElixirTerm.MakeUTF8String(s);
      ```
      
  * PID:
      
      ```        
      PID p = ElixirTerm.GetPID(argv[0]);
      ElixirTerm t = ElixirTerm.MakePID(p);
      ```
      
  * Reference:
      
      ```  
      Reference r = ElixirTerm.GetReference(argv[0]);
      ElixirTerm t = ElixirTerm.MakeReference(r);
      ```
      
  * Empty List:  
      
      ```  
      EmptyList e = ElixirTerm.GetEmptyList(argv[0]);
      ElixirTerm t = ElixirTerm.MakeEmptyList();
      ```
      
  * Tuple:
      
      ```
      Tuple t = ElixirTerm.GetTuple(argv[0]);
      ElixirTerm t = ElixirTerm.MakeTuple(new ElixirTerm[]{
        ElixirTerm.MakeAtom("error"), 
        ElixirTerm.MakeAtom("timeout")
      });
      ```
  
  * Byte List:
  
      ```
      byte[] b ElixirTerm.GetByteString(argv[0]);
      ElixirTerm t = ElixirTerm.MakeByteList(b);
      ```
  
  * List:
      
      ```
      ElixirTerm[] l = ElixirTerm.GetTuple(argv[0]);
      ElixirTerm t = ElixirTerm.MakeList(new ElixirTerm[]{
        ElixirTerm.MakeAtom("option1"), 
        ElixirTerm.MakeAtom("option2"),
        ElixirTerm.MakeEmptyList()
      });
      ```
      
  * Export:
  
      ```
      Export e1 = ElixirTerm.GetExport(argv[0]);
      Export e2 = new Export("String", "downcase", 1);
      ElixirTerm t = ElixirTerm.MakeExport(e2);
      ```

### Custom C# Types

  Read only properties:
  
  * PID
    * Node (string)
    * ID (int)
    * Serial (int)
    * Creation (byte)
  * Reference
    * Node (string)
    * Creation (byte)
    * ID (int[])
  * Tuple
    * Arity (int)
  * Export
    * Module (string)
    * Function (string)
    * Arity (byte)
  
  
  #### Tuple Usage
  
      Tuple t = ElixirTerm.GetTuple(argv[0]);
      string a = ElixirTerm.GetAtom(t[0]);
      string s = ElixirTerm.GetUTF8String(t[1]);

### Elixir Callbacks From C#

  You can call elixir functions or set up events to trigger them by using `ExSharp.Runner.ElixirCallback(fun, args)`.
  
  `fun` is an ElixirTerm with tag `TagType.NEW_FUN` or `TagType.EXPORT`.
  
  #### Examples  
  
      [ExSharpFunction("call", 0)]
      public static void Call(ElixirTerm[] argv, int argc)
      {
        ExSharp.Runner.ElixirCallback(argv[0], new ElixirTerm[0]);
      }      
      
      iex> fun = fn -> IO.puts "Hi!" end
      iex> Foo.call(fun)
      iex> :ok
      iex> HI    
      
      
      

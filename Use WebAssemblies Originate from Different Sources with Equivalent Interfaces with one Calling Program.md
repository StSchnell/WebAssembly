# Use WebAssemblies Originate from Different Sources with Equivalent Interfaces with one Calling Program

This article focuses on the development of WebAssemblies (Wasm) with equivalent interfaces, but with different compilers. Their uniform use is also considered, with one calling program. For this purpose as examples a Wasm is coded in C and compiled via [emscripten](https://emscripten.org/) and another Wasm in [Rust](https://www.rust-lang.org/). Both Wasm files have the same interface. On this way the Wasm file can be exchanged. This means that both, the C Wasm and the Rust Wasm file, can be used with the same calling program, in this case on Java base via [Chicory](https://chicory.dev/). This implementation shows that with a clever definition of the interface to the Wasm functions, the programming language or tool chain is irrelevant, which has built the Wasm. One calling program can be used to call Wasm functions that originate from different sources.

![image](https://github.com/user-attachments/assets/32078ccd-28a1-4a5c-92d4-a6db6539e6c6)

The examples that have been chosen are simple. It is not the function that is important here. What is important are the interfaces used.

## C Code

Here a tiny C program with two functions, named add and hello. The add function sums two numbers. The hello function delivers a Hello World message, depending on whether a name is passed as a parameter. If a name is passed it is used, otherwise a standard text.

```c
#include <emscripten.h>
#include <stdlib.h>
#include <string.h>

EMSCRIPTEN_KEEPALIVE 
int add(int value1, int value2) {
  int result = value1 + value2;
  return result;
}

EMSCRIPTEN_KEEPALIVE 
void hello(char* name, int len, char* result) {
  if (len == 0) {
    char* ret = "Hello World from C Language\n";
    strcpy(result, ret);
  } else {
    char* ret = "Hello ";
    strcat(ret, name);
    strcat(ret, " from C Language\n");
    strcpy(result, ret);
  }
}
```

```
@emcc wasmTest.c -o wasmTest.c.wasm --no-entry -s EXPORTED_FUNCTIONS=_malloc,_free
```

Emscripten eliminate functions that are not called from the compiled code. The standard C functions malloc and free are required to pass the string parameter. malloc is defined in stdlib.h and allocates memory. free is defined in stdlib.h and deallocates the space previously allocated by malloc. To make sure that the C functions are available, it must be added to the EXPORTED_FUNCTIONS.

## Rust Code

Here a tiny Rust program with four functions. Beside to the add and hello functions, as explained above, there are also malloc and free. These are available as equivalents for the corresponding standard C functions, which are exported during compilation with emscripten. The code was taken from the example, only the functions were renamed from alloc to malloc and from dealloc to free.

```rust
use std::*;

#[no_mangle]
pub extern "C" fn malloc(len: u32) -> *mut u8 {
  let mut buf = Vec::with_capacity(len as usize);
  let ptr = buf.as_mut_ptr();
  mem::forget(buf);
  ptr
}

#[no_mangle]
pub unsafe extern "C" fn free(ptr: &mut u8, len: u32) {
  let _ = Vec::from_raw_parts(ptr, 0, len as usize);
}

#[no_mangle]
pub extern fn add(value1: i32, value2: i32) -> i32 {
  let result: i32 = value1 + value2;
  result
}

#[no_mangle]
pub extern fn hello(name: u8, len: u32, result: *mut u8) {
  let bytes = unsafe {
    slice::from_raw_parts(name as *const u8, len as usize)
  };
  let str_name = str::from_utf8(bytes).unwrap().trim();
  let mut out_text = "".to_string();
  if len == 0 {
    out_text = "Hello World from Rust Language".to_string();
  } else {
    out_text = "Hello ".to_string() + str_name + " from Rust Language";
  }
  let out = out_text.as_bytes();
  unsafe {
    std::ptr::copy(out.as_ptr().cast(), result, out.len());
  }
}
```

```
rustc --target wasm32-unknown-unknown -O --crate-type=cdylib wasmTest.rs -o wasmTest.rs.wasm
```

## Comparison of C and Rust WebAssembly

Both Wasm files have the same interface, with the exception of the free function. The C function does not require the length of the allocated memory, while the Rust function expects this. But the calling of the free function in the C Wasm with an unnecessary additional parameter, in this case the memory size, does not lead to an error.

## Call with Rhino JavaScript

[Rhino](https://github.com/mozilla/rhino) is an open-source implementation of JavaScript written entirely in Java. It is great for rapid prototyping and is a [component of many products](https://github.com/mozilla/rhino/discussions/1425).

The following code contains in the executeWasm function several steps. The Wasm is instantiated and the functions are determined. At the add functions the parameters can be passed directly. At the hello function the memory, for the parameter and the return value, must be allocated first and then the parameter is set. The functions are executed and the return value is read. Finally the allocated memory is released.
The call is made via a loop in the main function, which passes the C Wasm file and the Rust Wasm file.

```javascript
function executeWasm(fileName) {

  try {

    const file = new java.io.File(fileName);
    const module = com.dylibso.chicory.wasm.Parser.parse(file);
    const instance = com.dylibso.chicory.runtime.Instance.builder(module).build();

    const malloc = instance.export("malloc");
    const free = instance.export("free");
    const memory = instance.memory();

    const add = instance.export("add");
    const addResult = add.apply(5, 2)[0];
    // Should print 7.0
    java.lang.System.out.println(addResult);

    const subtract = instance.export("subtract");
    const subtractResult = subtract.apply(5, 2)[0];
    // Should print 3.0
    java.lang.System.out.println(subtractResult);

    const hello = instance.export("hello");
    const name = "Stefan";
    const ptrName = malloc.apply(name.length)[0];
    memory.writeString(ptrName, name);
    const ptrResult = malloc.apply(128)[0];
    hello.apply(ptrName, name.length, ptrResult);
    // Should print Hello Stefan from Rust Language
    java.lang.System.out.println(
      memory.readString(ptrResult, 128).trim()
    );

    free.apply(ptrResult, 128);
    free.apply(ptrName, name.length);

  } catch(exception) {
    java.lang.System.out.println(exception);
  }

}

function main() {
  const wasmNames = [
    "wasmTest2.rs.wasm",
    "wasmTest2.c.wasm"
  ]
  wasmNames.forEach( function(wasmName) {
    executeWasm(wasmName);
  });
}

main();
```

The Chicory Java archives are named in the class path with the Rhino engine.

```
java -cp ".:rhino-1.7.15.jar:runtime-1.1.0.jar:wasm-1.1.0.jar:log-1.1.0.jar:wasi-1.1.0.jar" org.mozilla.javascript.tools.shell.Main wasmTest.js

```

## Call with JShell

The [Java Shell tool (JShell)](https://docs.oracle.com/en/java/javase/23/jshell/introduction-jshell.html) is an interactive tool for the Java programming language and prototyping Java code.

The explanations above also apply to this code, which is really very similar. The Chicory Java archives are named here in the class path with the environment command.

```java
/env --class-path .:log-1.1.0.jar:runtime-1.1.0.jar:wasi-1.1.0.jar:wasm-1.1.0.jar

import com.dylibso.chicory.runtime.ExportFunction;
import com.dylibso.chicory.runtime.Instance;
import com.dylibso.chicory.runtime.Memory;
import com.dylibso.chicory.wasm.Parser;
import com.dylibso.chicory.wasm.WasmModule;
import java.io.File;

void executeWasm(String fileName) {

  try {

    File file = new File(fileName);
    WasmModule module = Parser.parse(file);
    Instance instance = Instance.builder(module).build();

    ExportFunction malloc = instance.export("malloc");
    ExportFunction free = instance.export("free");
    Memory memory = instance.memory();

    ExportFunction add = instance.export("add");
    int addResult = (int) add.apply(5, 2)[0];
    java.lang.System.out.println(addResult);

    ExportFunction subtract = instance.export("subtract");
    int subtractResult = (int) subtract.apply(5, 2)[0];
    java.lang.System.out.println(subtractResult);

    ExportFunction hello = instance.export("hello");
    String name = "Stefan";
    int ptrName = (int) malloc.apply(name.length())[0];
    memory.writeString(ptrName, name);
    int ptrResult = (int) malloc.apply(128)[0];
    hello.apply(ptrName, name.length(), ptrResult);
    java.lang.System.out.println(
      memory.readString(ptrResult, 128).trim()
    );

    free.apply(ptrResult, 128);
    free.apply(ptrName, name.length());

  } catch(Exception exception) {
    java.lang.System.out.println(exception.toString());
  }

}

void main() {
  String[] wasmNames = new String[] {
    "wasmTest2.rs.wasm",
    "wasmTest2.c.wasm"
  };
  for (String wasmName : wasmNames) {
    executeWasm(wasmName);
  }
}

main();
/exit
```

```
jshell wasmTest.jsh
```

## Conclusion

A smart definition of functions and their interfaces of a Wasm can ensure simple interchangeability, regardless of the origin of the Wasm. With emscripten it is very easy to convert C / C++ functions into Wasm. With Rust it is possible to build new developments and compile it into Wasm too. With Chicory it is very easy to test or use these Wasm files. This makes it easy to combine existing code and new functionalities.


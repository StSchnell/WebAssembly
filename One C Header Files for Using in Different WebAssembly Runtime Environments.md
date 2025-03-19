
# One C Header Files for Using in Different WebAssembly Runtime Environments

This article describes the development of WebAssembly (Wasm) with the same code, but with different frameworks. For this purpose the code to be shared is stored in an external file, which is included during compilation. In this example the programming language C and the frameworks [Emscripten](https://emscripten.org/) and [Extism](https://extism.org/) are used. The code to be used is stored in a header file and with the frameworks a wrapper is built around this functions.

## Shared Header File

In this example the header file contains only one function, named _hello. It delivers a Hello World message, depending on whether a name is passed as a parameter. If a name is passed it is used, otherwise a standard text.

```c
/*
 * fileName: helloWorld.h
 */

void _hello(char* name, char* result) {
  if (strlen(name) == 0) {
    char* ret = "Hello World from C Language";
    strcpy(result, ret);
  } else {
    char* ret = "Hello ";
    strcat(ret, name);
    strcat(ret, " from C Language");
    strcpy(result, ret);
  }
}
```

## Emscripten Approach

The wrapper with Emscripten is very easy, the function call is simply passed through.

```c
/*
 * fileName: helloWorld.wasm
 */

#include <emscripten.h>
#include <stdlib.h>
#include <string.h>

#include "helloWorld.h"

EMSCRIPTEN_KEEPALIVE 
void hello(char* name, char* result) {
  _hello(name, result);
}
```

Below is a simple test program with [Rhino JavaScript](https://github.com/mozilla/rhino) and [Chicory](https://chicory.dev/).

```javascript
function main() {

  try {

    var file = new java.io.File("./helloWorld.wasm");
    var module = com.dylibso.chicory.wasm.Parser.parse(file);
    var instance = com.dylibso.chicory.runtime.Instance.builder(module).build();

    var malloc = instance.export("malloc");
    var free = instance.export("free");
    var memory = instance.memory();

    var hello = instance.export("hello");
    var strHelloName = "Stefan";
    var helloName = malloc.apply(strHelloName.length)[0];
    memory.writeString(helloName, strHelloName);
    var helloResult = malloc.apply(128)[0];
    hello.apply(helloName, helloResult);
    // Should print Hello Stefan from C Language
    java.lang.System.out.println(memory.readString(helloResult, 128).trim());

    free.apply(helloName);
    free.apply(helloResult);

  } catch(exception) {
    print(exception);
  }

}

// Main
main();
```

## Extism Approach

The wrapper with [Extism C-PDK](https://github.com/extism/c-pdk) is more complex, because the input and output variables are prepared via the framework. However, we can use this Wasm with the 15 programming languages for which an Extism SDK is available.

The code looks a bit redundant, this can certainly be realized even better. Our focus here is on code from a single source, so we do not care about that.

```c
/*
 * fileName: helloWorld.wasm
 */

#define EXTISM_ENABLE_LOW_LEVEL_API
#define EXTISM_IMPLEMENTATION
#include "extism-pdk.h"

#include <stdio.h>
#include <string.h>

#include "helloWorld.h"

int32_t EXTISM_EXPORTED_FUNCTION(hello) {

  char helloMsg[128];
  char helloAnswer[128];

  uint64_t inputLen = extism_input_length();
  if (inputLen == 0) {
    _hello(NULL, helloMsg);
  } else {
    uint8_t inputData[128];
    extism_load_input(0, inputData, inputLen);
    inputData[inputLen] = '\0';
    _hello((char*)inputData, helloMsg);
  }

  int helloMsgLen = sizeof(helloMsg);
  int numberOfChars = snprintf(helloAnswer, helloMsgLen, "%s", helloMsg);

  ExtismHandle buffer = extism_alloc(numberOfChars);
  extism_store(buffer, helloAnswer, numberOfChars);
  extism_output_set(buffer, numberOfChars);

  return 0;

}
```

Below a simple test program with Python and [Extism Python-SDK](https://github.com/extism/python-sdk).

```python
import extism

def main():

    try:

        wasmFile = open("./helloWorld.wasm", mode="rb")
        wasmData = wasmFile.read()
        wasmFile.close()

        if wasmData:

            manifest = {"wasm": [{"data": wasmData}]}
            plugin = extism.Plugin(manifest)

            print(
                plugin.call("hello", "")
            )

            print(
                plugin.call("hello", "Stefan")
            )

    except Exception as ex:
        print(ex)

if __name__ == "__main__":
    main()
```

## Conclusion

As we can see, it is possible to build web assemblies using code from a single source with different frameworks. This approach generally gives us further independence when building Wasm modules.

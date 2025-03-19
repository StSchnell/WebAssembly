# WebAssembly

This section presents development approaches for [WebAssembly (Wasm)](https://webassembly.org/) that relates to its use in server environments. This is not Wasm's original focus. But Wasm offers the basics of being able to run on different systems without the need for changes. This means that it is not necessary to write different code for different platforms or architectures. This simplifies development and deployment and saves time and resources. This perspective is elaborated here.

## Optimization Scenarios for Development

The current variety of different possibilities for building WebAssembly modules also leads to a variety of individual programming approaches. The use of an individual framework or tool chain can also lead to a dependency, if building and using it is only possible with it. This simplifies development, but the WebAssembly module loses its independence and is bound to the framework or tool chain used. Therefore approaches will be presented here that can be used to mitigate these restrictions.

### Interfaces

By standardizing the interfaces of WebAssembly modules, that were created with different frameworks or tool chains, uniform calls can be made.

* [Example how to build C and Rust code to achieve a standardized interface](https://github.com/StSchnell/WebAssembly/blob/main/Use%20WebAssemblies%20Originate%20from%20Different%20Sources%20with%20Equivalent%20Interfaces%20with%20one%20Calling%20Program.md)

### Code

If the code is saved in independent files, e.g. C header files which are included at building, it can be used from different frameworks or tool chains. This allows us to build a standardized code base, which is bound to the programming language.

### wasm 内存
---
1. wasm 目前没有 `HEAPU64`, 如果需要 `HEAPU64`, 可以自己定义
```javascript
javascript
   Module['HEAPU64'] = HEAPU64 = new BigUint64Array(memory.buffer);
```

2. `wasmMemory.buffer` 和 `Module['memory'].buffer` 都是指向 wasm 内存缓冲区的引用。
	在 Emscripten 的胶水代码中，`Module['memory']` 是一个指向分配给 wasm 模块的内存的引用。它在 Emscripten 的初始化过程中被自动创建并分配内存空间。你可以通过 `Module['memory'].buffer` 来访问该内存的底层 ArrayBuffer 对象。


3. 这段代码的作用是更新 wasm 内存的视图，以便在 JavaScript 环境中使用。它将创建不同类型的 TypedArray，将它们连接到 wasm 内存的底层 ArrayBuffer 上，并将它们赋值给 `Module` 对象的属性。
```markdown 
- `HEAP8`：Int8Array 类型的视图，用于操作 8 位有符号整数数据。
- `HEAP16`：Int16Array 类型的视图，用于操作 16 位有符号整数数据。
- `HEAP32`：Int32Array 类型的视图，用于操作 32 位有符号整数数据。
- `HEAPU8`：Uint8Array 类型的视图，用于操作 8 位无符号整数数据。
- `HEAPU16`：Uint16Array 类型的视图，用于操作 16 位无符号整数数据。
- `HEAPU32`：Uint32Array 类型的视图，用于操作 32 位无符号整数数据。
- `HEAPF32`：Float32Array 类型的视图，用于操作 32 位浮点数数据。
- `HEAPF64`：Float64Array 类型的视图，用于操作 64 位浮点数数据。
```

4. wasm 内存的底层 ArrayBuffer 可以看作是 C 层分配的内存堆。当你在 C 层使用内存分配函数（如 `malloc`、`calloc` 或 `realloc`）分配内存时，实际上是在 wasm 内存的底层 ArrayBuffer 上分配了一块连续的内存空间。

5. 目前wasm的内存使用一般不会超过4G， 所以32bit的指针放的下地址。
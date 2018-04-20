# Swift指针使用
原文: <https://www.raywenderlich.com/148569/unsafe-swift>

[TOC]

## 内存布局 MemoryLayout

### 结构的内存布局
```swift
struct EmptyStruct {}

MemoryLayout<EmptyStruct>.size      // returns 0
MemoryLayout<EmptyStruct>.alignment // returns 1
MemoryLayout<EmptyStruct>.stride    // returns 1

struct SampleStruct {
  let number: UInt32
  let flag: Bool
}

MemoryLayout<SampleStruct>.size       // returns 5
MemoryLayout<SampleStruct>.alignment  // returns 4
MemoryLayout<SampleStruct>.stride     // returns 8
```

### 类的内存布局 

```swift
class EmptyClass {}

MemoryLayout<EmptyClass>.size      // returns 8 (on 64-bit)
MemoryLayout<EmptyClass>.stride    // returns 8 (on 64-bit)
MemoryLayout<EmptyClass>.alignment // returns 8 (on 64-bit)

class SampleClass {
  let number: Int64 = 0
  let flag: Bool = false
}

MemoryLayout<SampleClass>.size      // returns 8 (on 64-bit)
MemoryLayout<SampleClass>.stride    // returns 8 (on 64-bit)
MemoryLayout<SampleClass>.alignment // returns 8 (on 64-bit)
```

## 指针  Pointers

名字构成规则
`Unsafe[Mutable][Raw][Buffer]Pointer[<T>]`

- **Unsafe Pointer**: 固定部分，表示是针对地址的指针操作，都是不安全的操作
- **Mutable**: 是否可以写内存内容
- **Raw**: 是否是针对无类型的内存块(void*)
- **Buffer**: 是否是针对数组的
- **Generic<T>**: 针对不是Raw的指针，需要绑定一个底层的类型≈ß


| Pointer Name | Unsafe? | Write Access? | Collection? | Strideable? | Typed? |
| --- | :-: | :-: | :-: | :-: | :-: |
| UnsafeMutablePointer<T> | Yes | Yes | No | Yes | Yes |
| UnsafePointer<T> | Yes | No | No | Yes | Yes |
| UnsafeMutableBufferPointer<T> | Yes | Yes | Yes | No | Yes |
| UnsafeBufferPointer<T> | Yes | No | Yes | No | Yes |
| UnsafeMutableRawPointer | Yes | Yes | No | Yes | No |
| UnsafeRawPointer | Yes | No | No | Yes | No |
| UnsafeMutableRawBufferPointer | Yes | Yes | Yes | No | No |
| UnsafeRawBufferPointer | Yes | No | Yes | No | No |

### 使用无类型指针

```swift
// 1
let count = 2
let stride = MemoryLayout<Int>.stride
let alignment = MemoryLayout<Int>.alignment
let byteCount = stride * count

// 2
do {
  print("Raw pointers")
  
  // 3
  let pointer = UnsafeMutableRawPointer.allocate(byteCount: byteCount, alignment: alignment)
  // 4
  defer {
    pointer.deallocate()
  }
  
  // 5
  pointer.storeBytes(of: 42, as: Int.self)
  pointer.advanced(by: stride).storeBytes(of: 6, as: Int.self)
  pointer.load(as: Int.self)
  pointer.advanced(by: stride).load(as: Int.self)
  
  // 6
  let bufferPointer = UnsafeRawBufferPointer(start: pointer, count: byteCount)
  for (index, byte) in bufferPointer.enumerated() {
    print("byte \(index): \(byte)")
  }
}
```

1. 定义后续会被使用到的常量:
    - **count** 存储的Int的数量
    - **stride** Int类型的stride
    - **alignment** Int类型的alignment
    - **byteCount** 需要分配的总内存数量

2. 增加一个do代码块，确保变量能够在后续的例子中使用

3. UnsafeMutableRawPointer.allocate用于分配内存，返回UnsafeMutableRawPointer类型。 

4. 通过defer块确保内存能够正确的被释放. ARC在这种情况下不能自动管理内存，需要使用者仔细处理。

5. `storeBytes`和`load`方法用户存储和读取bytes。读取内容的地址可以通过advance方法移动。
   由于指针式有Stride的，我们也可以用+来进行一定，类似`(pointer+stride).storeBytes(of: 6, as: Int.self)`。

6. UnsafeRawBufferPointer允许您像访问字节容器一样访问内存。 这意味着您可以迭代器或者下标来访问它们，甚至可以使用诸如`filter`，`map`和`reduce`等方法进行操作。`UnsafeRawBufferPointer`是通过`UnsafeRawPointer`进行初始化的。

### 使用带类型指针

```swift
do {
  print("Typed pointers")
  
  let pointer = UnsafeMutablePointer<Int>.allocate(capacity: count)
  pointer.initialize(to: 0, count: count)
  defer {
    pointer.deinitialize(count: count)
    pointer.deallocate(capacity: count)
  }
  
  pointer.pointee = 42
  pointer.advanced(by: 1).pointee = 6
  pointer.pointee
  pointer.advanced(by: 1).pointee
  
  let bufferPointer = UnsafeBufferPointer(start: pointer, count: count)
  for (index, value) in bufferPointer.enumerated() {
    print("value \(index): \(value)")
  }
}
```

注意无类型指针和带类型指针之间的差异

1. 内存是通过`UnsafeMutablePointer.allocate`进行分配的。类型参数让Swift知道指针会被用于存取Int类型的数据。

2. 使用前必须通过`initialize`初始化，使用后必须通过`deinitialize`清理。Update: as noted by user atrick in the comments below, deinitialization is only required for non-trivial types. That said, including deinitialization is a good way to future proof your code in case you change to something non-trivial. Also, it usually doesn’t cost anything since the compiler will optimize it out.

3. pointer拥有一个pointee属性，通过这个属性，提供一种类型安全的存取数据的方式。

### 将无类型指针转换为带类型指针

```Swift
do {
  print("Converting raw pointers to typed pointers")
  
  let rawPointer = UnsafeMutableRawPointer.allocate(bytes: byteCount, alignedTo: alignment)
  defer {
    rawPointer.deallocate(bytes: byteCount, alignedTo: alignment)
  }
  
  let typedPointer = rawPointer.bindMemory(to: Int.self, capacity: count)
  typedPointer.initialize(to: 0, count: count)
  defer {
    typedPointer.deinitialize(count: count)
  }

  typedPointer.pointee = 42
  typedPointer.advanced(by: 1).pointee = 6
  typedPointer.pointee
  typedPointer.advanced(by: 1).pointee
  
  let bufferPointer = UnsafeBufferPointer(start: typedPointer, count: count)
  for (index, value) in bufferPointer.enumerated() {
    print("value \(index): \(value)")
  }
}
```

### 获得实例的Bytes

```Swift
do {
  print("Getting the bytes of an instance")
  
  var sampleStruct = SampleStruct(number: 25, flag: true)

  withUnsafeBytes(of: &sampleStruct) { bytes in
    for byte in bytes {
      print(byte)
    }
  }
}
```

## 使用Unsafe系列操作的3条准则

1. **不要在withUnsafeBytes中返回指针**

   ```Swift
   // Rule #1
    do {
      print("1. Don't return the pointer from withUnsafeBytes!")
    
      var sampleStruct = SampleStruct(number: 25, flag: true)
    
      let bytes = withUnsafeBytes(of: &sampleStruct) { bytes in
        return bytes // strange bugs here we come ☠☠☠
      }
    
      print("Horse is out of the barn!", bytes)  /// undefined !!!
    }
    ```

2. **一次只绑定一个类型**

   ```Swift
   // Rule #2
    do {
      print("2. Only bind to one type at a time!")
    
      let count = 3
      let stride = MemoryLayout<Int16>.stride
      let alignment = MemoryLayout<Int16>.alignment
      let byteCount =  count * stride
      
      let pointer = UnsafeMutableRawPointer.allocate(bytes: byteCount, alignedTo: alignment)
    
      let typedPointer1 = pointer.bindMemory(to: UInt16.self, capacity: count)
    
      // Breakin' the Law... Breakin' the Law  (Undefined behavior)
      let typedPointer2 = pointer.bindMemory(to: Bool.self, capacity: count * 2)
    
      // If you must, do it this way:
      typedPointer1.withMemoryRebound(to: Bool.self, capacity: count * 2) {
        (boolPointer: UnsafeMutablePointer<Bool>) in
        print(boolPointer.pointee)  // See Rule #1, don't return the pointer
      }
    }
   ```

3. **不要越界**

   ```Swift
   // Rule #3... wait
    do {
      print("3. Don't walk off the end... whoops!")
    
      let count = 3
      let stride = MemoryLayout<Int16>.stride
      let alignment = MemoryLayout<Int16>.alignment
      let byteCount =  count * stride
    
      let pointer = UnsafeMutableRawPointer.allocate(bytes: byteCount, alignedTo: alignment)
      let bufferPointer = UnsafeRawBufferPointer(start: pointer, count: byteCount + 1) // OMG +1????
    
      for byte in bufferPointer {
        print(byte)  // pawing through memory like an animal
      }
    } 
   ```


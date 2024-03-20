# Rust跨平台-Android和鸿蒙OS



## **安装** **`rustup`**

rustup 是 Rust 的安装和版本管理工具

```Shell
$ curl --proto '=https' --tlsv1.2 https://sh.rustup.rs -sSf | sh
```

该命令会安装 rusup 和最新的稳定版本的 Rust；包括：

- `rustc` Rust 编译器，用于将 Rust 代码编译成可执行文件或库。
- `cargo` Rust 的包管理器和构建工具，用于管理项目依赖、编译项目、运行测试等。
- `rustfmt` 代码格式化工具，用于自动格式化 Rust 代码以符合官方风格指南。
- `clippy` 静态分析工具，用于捕捉常见错误和改进代码质量。
- 其他工具，如`rustdoc`用于生成文档等。

成功后控制台会输出：`Rust is installed now. Great!`

macOS系统上需要安装：`xcode-select --install`

**cargo** 在开发中较为常用，算是打交道最多的工具之一



## 标准库 Rust Standard Library

标准库是 Rust 编程语言的官方库，提供了一系列预先编写好的类型和函数，用来处理常见的任务，如：

1. 基本数据类型（比如`i32`, `u64`, `f32`等）。
2. 集合类型（如`Vec<T>`, `HashMap<K, V>`等）。
3. 输入/输出（I/O）操作，包括文件操作和网络编程。
4. 线程和并发编程工具。
5. 其他有用的工具，如字符串处理、日期和时间操作等。

### 渠道

通常情况下安装 rustup 的时候，标准库就已经安装到本地；但是 rust 有几种发布渠道，用于提供不同稳定程度的 Rust 版本，Rust 的三个主要发布渠道是：

1. **Stable（稳定版）**：这是大多数用户推荐使用的版本。它每六周发布一次，提供最新的功能和改进，但只包括那些经过充分测试和认为稳定的特性。
2. **Beta（测试版）**：这个版本比 Stable 新，但可能包含一些即将纳入下一个 Stable 版本的特性和改进。它主要用于测试即将发布的功能，以确保它们在正式成为稳定版之前没有问题。
3. **Nightly（每夜构建版）**：这是最前沿的版本，包括了所有最新开发的特性。这些特性可能未完全稳定或待评估，因此这个版本主要用于实验和评估最新的语言改进。Nightly 版本，顾名思义，每夜更新一次，包括最新的代码提交。

### 安装

- **列出已安装的版本**

```Shell
rustup toolchain list
```

- 安装新的版本

```Shell
rustup toolchain install beta
```

或者

```Shell
rustup toolchain install nightly
```

### 切换版本

切换全局（即默认）Rust版本，使用`rustup default`命令：

```Shell
rustup default stable
rustup default beta
rustup default nightly
```

这些命令会将你的系统默认Rust版本切换为相应的版本。

### 为**特定****项目****切换版本**

如果你只想为特定的项目切换Rust版本，而不影响全局设置，可以在项目目录内使用以下命令设置目录级别的默认版本：

```Shell
rustup override set stable
rustup override set beta
rustup override set nightly
```

### 补装标准库源码

```Shell
rustup component add rust-src
```

每一个 toolchain 都有自己的源码

建议安装 stable 和 nightly 的源码，因为只有 nightly 版本支持编译鸿蒙系统

如果不安装后续鸿蒙OS下编译会报错，根据提示安装也行

### 为特定目标平台编译代码

在 stable 下，rust 支持 android 平台的编译，通过 `rustup target list |grep android` 可以查看支持的所有平台架构

```Shell
% rustup target list |grep android                                                                                                                                                    24-03-19 - 15:46:34
aarch64-linux-android (installed)
arm-linux-androideabi (installed)
armv7-linux-androideabi (installed)
i686-linux-android (installed)
thumbv7neon-linux-androideabi (installed)
x86_64-linux-android (installed)
```

如果已安装，后面会有 (installed) 标识；建议一次性都安装上：

```Shell
rustup target add aarch64-linux-android arm-linux-androideabi armv7-linux-androideabi i686-linux-android thumbv7neon-linux-androideabi x86_64-linux-android
```

鸿蒙OS下需要切换到 nightly，通过 `rustup target list |grep ohos` 可以查看支持的所有平台架构：

```Shell
% rustup target list |grep ohos                                                         
aarch64-unknown-linux-ohos (installed)
armv7-unknown-linux-ohos (installed)
x86_64-unknown-linux-ohos (installed)
```

同样，建议一次性都安装上



## 创建 Rust Library 工程

使用命令行创建：

```Shell
cargo new demo --lib
```

或者使用 IDE，推荐使用 Jetbrains 的 RustRover

<img src="https://raw.githubusercontent.com/liuqingdada/PicGo/main/rust-rustrover-new-library.png" style="float: left;"/>

此时目录结构如下：

```Shell
demo
├── Cargo.toml
└── src
    └── lib.rs
```

### Cargo.toml 的配置

#### [lib]

```TOML
[lib]
crate-type = ["cdylib"]
```

`crate-type`属性用于指定编译目标类型。这些类型决定了编译器会如何编译你的代码。以下是一些常见的`crate-type`值及其区别：

**1.** **`bin`**

- **描述**：一个可执行的二进制文件。
- **使用****场景**：当你想要创建一个可以直接运行的程序时，使用此类型。大多数应用程序都是以`bin`类型编译的。

**2.** **`lib`**

- **描述**：一个库文件，可以被其他Rust包作为依赖使用。
- **使用****场景**：如果你正在开发一个提供函数、类型或特性给其他包使用的库，应选择此类型。

**3.** **`rlib`**

- **描述**：Rust编译的库文件，包含元数据和符号，供后续的Rust编译阶段使用。
- **使用****场景**：当你想要编译一个Rust库供其他Rust项目使用，并期望进行链接和代码生成优化时。

**4.** **`dylib`**

- **描述**：一个动态链接库（DLL），可以在运行时被Rust或其他语言的应用程序动态链接。
- **使用****场景**：当你想要创建一个可以被多个程序共享的库，或者当你需要和其他使用动态链接的语言互操作时。

**5.** **`cdylib`**

- **描述**：一个为C语言接口定制的动态链接库。它移除了Rust特有的元数据，只保留了可以从C或其他语言调用的符号。
- **使用****场景**：当你开发一个Rust库，希望能够被C或其他语言作为动态链接库使用时。这是创建跨语言共享库的常见方式。

**6.** **`staticlib`**

- **描述**：静态库（`.a`文件），可以被C语言或其他语言的应用程序在编译时静态链接。
- **使用****场景**：如果你想要创建一个可以被其他语言静态链接的库，或希望你的Rust代码被编译进一个单独的二进制文件，而不依赖于Rust的运行时或其他动态库。

**7.** **`proc-macro`**

- **描述**：一个过程宏库，用于创建自定义`#[derive]`宏或其他类型的宏。
- **使用****场景**：当你想要创建新的宏来扩展Rust语法，比如自定义派生属性或宏指令时。

#### [dependencies] 和 [features]

由于需要区分 android 和 ohos 两个平台的特定库，所以有一些依赖库需要配置为可选的，然后使用 `cargo` 构建的时候添加 `--features` 参数来分别进行交叉编译

对于 android 平台，需要引入 jni 库，来和 java/kotlin 互相调用

`rust` 和 `node` 互相调用可以使用 `node-bindgen`，但遗憾的是，`node-bindgen`并不兼容鸿蒙系统；不过已经有人基于`node-bindgen`兼容了ohos：https://crates.io/crates/ohos-node-bindgen

对于 `ohos` 平台，需要引入 `ohos-node-bindgen` 库，来和 `node` 通信；由于 `ohos-node-bindgen` 依赖 `socket2`，然而 `socket2` 在 `ohos` 下有 `bug`，所以这里需要使用`https://github.com/stuartZhang/socket2.git`` `来替换 `ohos-node-bindgen` 内部依赖的`socket2`版本

最终配置如下

```TOML
[features]
default = ["android"]
android = ["dep:jni", "dep:android_logger"]
ohos = ["dep:ohos-node-bindgen", "dep:socket2"]

[dependencies]
jni = { version = "0.19.0", optional = true }
android_logger = { version = "0.13.3", optional = true }
ohos-node-bindgen = { version = "6.0.3", optional = true }
socket2 = { version = "0.4.10", optional = true }
dashmap = "5.5.3"
threadpool = "1.8.1"
log = "0.4.21"

[patch.crates-io]
socket2 = { version = "0.4.10", git = "https://github.com/stuartZhang/socket2.git", branch = "v0.4.x" }
```

也就是说：

- `dashmap`、`threadpool` 和 `log` 是所有平台下都参与编译的库
- `android` 单独编译：`jni` 和 `android_logger`
- `ohos` 单独编译：`ohos-node-bindgen` 和 `socket2`
- 另外，`features` 的默认值为 `android`

### 编写代码 - lib.rs

由于存在不同的 features，所以对于 android：

```Rust
#[cfg(feature = "android")]
#[no_mangle]
pub extern "system" fn Java_com_haier_uhome_uplus_hook_monitor_app_NativeLib_hello(
    env: JNIEnv,
    _class: JClass,
) -> jstring {
    // 将 Rust 字符串转换为 JNI 字符串
    let result = env.new_string("Hello from Rust!").expect("Couldn't create Java string!");
    // 返回结果
    result.into_inner()
}
```

`#[cfg(feature = "android")]`：与上述 `features` 对应

`#[no_mangle]` 则是禁用驼峰警告

对于 ohos：

```Rust
#[cfg(feature = "ohos")]
#[ohos_node_bindgen]
pub extern "C" fn add(l: i32, r: i32) -> i32 {
    l + r
}
```

`#[cfg(feature = "ohos")]`：与上述 `features` 对应

`#[ohos_node_bindgen]` 则是标识 add 函数可以被 node 端调用

node-bindgen 的大致原理如下：

1. **FFI（外部函数接口）**

Node.js 的原生模块基于 C++ 和 Node.js 的 N-API（原生API），N-API 提供了一套与 V8 引擎解耦的接口，使原生模块在 Node.js 版本升级时保持兼容。`node-bindgen` 底层利用 Rust 的外部函数接口（FFI）能力，通过这些接口与 Node.js 通信。

Rust 的 FFI 功能允许其调用 C 语言API。因此，`node-bindgen` 实际上是通过 Rust 的 FFI 调用 Node.js 的 N-API 来创建和管理 JavaScript 值，以及执行与 JavaScript 环境的交互。

1. **宏和属性**

`node-bindgen` 提供了一系列宏（例如 `#[node_bindgen]`），这些宏在编译时自动生成将 Rust 函数暴露为 Node.js 可调用函数的胶水代码。这个过程包括自动生成用于参数转换和返回值处理的代码，使 Rust 函数能够直接接收来自 JavaScript 的参数并返回可以直接在 JavaScript 中使用的结果。

1. **内存管理**

Rust 和 JavaScript 之间的内存管理是 `node-bindgen` 的关键部分。Rust 有自己的内存管理规则，主要基于所有权和生命周期，而 JavaScript 的内存则由垃圾收集器自动管理。`node-bindgen` 必须确保在这两种内存管理模型之间正确地桥接，包括处理 Rust 中的数据所有权转移和确保 JavaScript 对象在需要时保持存活。

1. **异步操作**

Node.js 广泛使用异步操作，而 Rust 也有强大的异步支持。`node-bindgen` 支持将 Rust 的异步操作暴露给 Node.js。这通过将 Rust 的 `Future` 转换为 Node.js 的 `Promise` 来实现。`node-bindgen` 会自动处理这种转换，允许开发者以 Promise 的形式在 JavaScript 中接收 Rust 异步操作的结果。

1. **类型转换**

`node-bindgen` 自动处理 Rust 类型和 JavaScript 类型之间的转换。对于简单类型（如数字和字符串），这通常是直接的。但对于复杂类型（如结构体或枚举），`node-bindgen` 生成的代码会负责序列化和反序列化操作，确保两种语言之间可以无缝交换复杂数据结构。

### **总结**

`node-bindgen` 利用 Rust 的 FFI 能力、宏系统、强类型系统和异步特性，提供了一种高效、类型安全的方式来将 Rust 代码与 Node.js 集成。它自动处理大部分繁琐的胶水代码编写工作，使得 Rust 和 Node.js 之间的交互变得更加简单直接。这样的设计允许开发者专注于实现应用逻辑，而无需深入底层的语言绑定细节。

### 编译 - android 平台的产物

官方提供了 `cargo-ndk` 工具，它简化了为 Android 使用 Rust 编写原生代码库（.so 文件）的过程。

- 下载安卓 NDK，并配置到环境变量

```Bash
export ANDROID_HOME=$HOME/ssd/Android/sdk
PATH="$ANDROID_HOME/ndk-bundle:$PATH"
export PATH
```

- 安装 cargo-ndk

```Shell
cargo install cargo-ndk
```

- **使用** **`cargo-ndk`** **构建你的****项目**

```Shell
cargo ndk -t armv7-linux-androideabi -t aarch64-linux-android -o ../../MonitorTestClient/app/src/main/jniLibs build --release
```

- 参数解释
  - `-t` 或 `--target`：指定目标架构
  - `-o` 或 `--output`：指定输出目录，这里的目录会用于存放编译生成的 `.so` 文件
  - `build`：是 `cargo` 的子命令，用于编译项目，会传递它以及任何附加参数给 `cargo build`

### 编译 - ohos 平台的产物

官方没有为鸿蒙系统提供类似`cargo-ndk`的工具，需要手动配置编译参数

1. 首先切换到 nightly 渠道

```Shell
rustup override set nightly
```

1. 配置环境变量：

```Bash
# huawei
export OHOS_HOME=$HOME/ssd/huawei/sdk
export OHOS_API_V=9
export OHOS_CORE_V=3.1.0
export OH_NDK_ROOT=$OHOS_HOME/openharmony/$OHOS_API_V/native
PATH="$OHOS_HOME/hmscore/$OHOS_CORE_V/toolchains:$PATH"
export PATH
```

单独配置 `OHOS_API_V` 的好处是如果华为更新了 `Native SDK`，可以更方便的动态切换

1. 创建 ohos 对应 target 的可执行脚本，名称和上述“为特定目标平台编译代码”小节中对应
   1. 创建位置：`~/.cargo`
   2. aarch64-unknown-linux-ohos-clang.sh
   3. ```Bash
      #!/bin/sh
      . $HOME/.bash_profile
      exec $OH_NDK_ROOT/llvm/bin/clang \
        -target aarch64-linux-ohos \
        --sysroot=$OH_NDK_ROOT/sysroot \
        -D__MUSL__ \
        "$@"
      ```

   4. armv7-unknown-linux-ohos-clang.sh
   5. ```Bash
      #!/bin/sh
      . $HOME/.bash_profile
      exec $OH_NDK_ROOT/llvm/bin/clang \
        -target arm-linux-ohos \
        --sysroot=$OH_NDK_ROOT/sysroot \
        -D__MUSL__ \
        -march=armv7-a \
        -mfloat-abi=softfp \
        -mtune=generic-armv7-a \
        -mthumb \
        "$@"
      ```

   6. x86_64-unknown-linux-ohos-clang.sh
   7. ```Bash
      #!/bin/sh
      . $HOME/.bash_profile
      exec $OH_NDK_ROOT/llvm/bin/clang \
        -target x86_64-linux-ohos \
        --sysroot=$OH_NDK_ROOT/sysroot \
        -D__MUSL__ \
        "$@"
      ```

   8. `. $HOME/.bash_profile` 根据实际情况进行修改，只要能拿到 `$OH_NDK_ROOT` 即可
2. 通用配置：`config.toml`
   1. 创建位置：`~/.cargo`
   2. ```TOML
      # 鸿蒙编译工具链-目前只能手动配置:
      [target.aarch64-unknown-linux-ohos]
      linker = ".cargo/aarch64-unknown-linux-ohos-clang.sh"
      
      [target.armv7-unknown-linux-ohos]
      linker = ".cargo/armv7-unknown-linux-ohos-clang.sh"
      
      [target.x86_64-unknown-linux-ohos]
      linker = ".cargo/x86_64-unknown-linux-ohos-clang.sh"
      
      # 会概率性地失败于exit code: 0xc0000005, STATUS_ACCESS_VIOLATION错误 - https://rustcc.cn/article?id=568d35d6-b782-49e9-b9b1-5d870d28f927
      [profile.dev.package.compiler_builtins]
      opt-level = 2
      
      [alias]
      ohos-build = ["build", "-Zbuild-std", "--target=aarch64-unknown-linux-ohos", "--target=armv7-unknown-linux-ohos", "--target=x86_64-unknown-linux-ohos"]
      ```

   3. `[alias]` 作用是使得：
      -   `cargo ohos-build --release` 等价于 `cargo build -Zbuild-std --target=aarch64-unknown-linux-ohos --target=armv7-unknown-linux-ohos --target=x86_64-unknown-linux-ohos --release`
   4. 对于我们演示的 demo 工程，最终编译命令行如下
   5. ```Shell
      cargo ohos-build --release --features ohos
      ```



## Android 工程测试 rust 产物

把动态库拷贝到 app 模块中

```Shell
src
├── androidTest
├── main
│   ├── jniLibs
│   │   ├── arm64-v8a
│   │   │   └── libdemo.so
│   │   └── armeabi-v7a
│   │       └── libdemo.so
```

创建对应包名的单例

```Kotlin
object NativeLib {
  init {
    System.loadLibrary("demo")
  }

  external fun hello(): String

  external fun mapTest()
}
```

在 MainActivity 中调用

```Kotlin
val nstr = NativeLib.hello()
Log.d(TAG, "onCreate: $nstr")
```

```tex
2024-03-19 19:32:32.207 11941-11941 MainActivity             D  onCreate: Hello from Rust!
```





## ohos 工程测试 rust 产物

把动态库拷贝到 entry 模块中

```Shell
entry
├── build-profile.json5
├── hvigorfile.ts
├── libs
│   ├── arm64-v8a
│   │   └── libdemo.so
│   └── armeabi-v7a
│       └── libdemo.so
```

在 Index.ets 中

```TypeScript
import hello from "libdemo.so"

@Entry
@Component
struct Index {
  @State message: string = 'Hello World';

  build() {
    Row() {
      Column() {
        Text(this.message)
          .fontSize(50)
          .fontWeight(FontWeight.Bold)
        Button("计 算")
          .fontSize(50)
          .fontWeight(FontWeight.Bold)
          .onClick(() => {
            let sum = hello.add(3, 5);
            this.message = "3 + 5 = " + sum.toString();
          })
      }
      .width('100%')
    }
    .height('100%')
  }
}
```

运行结果：

<img src="https://raw.githubusercontent.com/liuqingdada/PicGo/main/rust-demo-ohos-hello.png" style="width: 30%; float: left; margin-right: 2%;"/><img src="https://raw.githubusercontent.com/liuqingdada/PicGo/main/rust-demo-ohos-add.png" style="width: 30%; float: left; margin-right: 2%;"/>












































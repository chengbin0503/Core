# 不适合 NativeAOT 编译的程序和功能

在考虑是否将 **.NET 项目**编译为**本地代码（NativeAOT）**时，有一些重要的限制需要注意。**NativeAOT** 编译器的设计目标是将应用程序生成为本地机器码，消除对 **.NET 运行时** 的依赖，从而获得**更快的启动速度**和**更低的内存占用**。但是，某些类型的应用程序和功能由于过度依赖 **.NET 运行时** 的动态特性、反射、JIT（即时编译）等机制，无法或不适合被编译为本地代码。

## 绝对不能生成本地代码（NativeAOT）的程序和功能

### 1. 强依赖反射的程序

**反射（Reflection）** 是 .NET 运行时的重要特性，它允许程序在运行时检查和修改自己的结构（类型、方法、属性等）。例如，使用 `Type.GetType()`、`Assembly.Load()`、`Activator.CreateInstance()` 等方法来动态加载类型或调用方法。

**NativeAOT** 编译会去除 **.NET 运行时**，因此没有反射的支持。你可以通过配置来保留反射元数据，但这会削弱 NativeAOT 的优势。

#### 示例代码：

    Type type = Type.GetType("MyNamespace.MyClass");
    var instance = Activator.CreateInstance(type);

**解决方案**：如果反射是不可或缺的，则该项目不适合使用 NativeAOT。如果只在少数地方使用反射，你可以考虑静态化这些部分，或者通过 `rd.xml` 保留必要的元数据。

### 2. 依赖动态代码生成的程序

**动态代码生成**（如 **C# 动态类型**、**Expression Trees（表达式树）**的编译、**Emit** 指令、**Roslyn 编译器服务**等）是通过运行时生成和执行代码。

**NativeAOT** 编译时，所有的代码都必须在编译阶段完成，并编译为本地机器码。因此，任何依赖于动态代码生成或即时编译（JIT）的代码都无法工作。

#### 示例代码：

    dynamic obj = GetDynamicObject();
    obj.SomeMethod(); // 动态调用

**解决方案**：如果代码中大量使用 `dynamic` 或表达式树的运行时编译，这部分代码需要重构为静态代码，或者避免使用 NativeAOT。

### 3. 依赖 `System.Reflection.Emit` 或 IL 生成的程序

**`System.Reflection.Emit`** 允许程序在运行时生成和执行新的 IL 代码，主要用于动态方法或代理的生成。

**NativeAOT** 不支持此功能，因为 NativeAOT 编译时已经将所有代码编译为本地机器码，无法在运行时生成新的 IL 指令。

#### 示例代码：

    var dynamicMethod = new DynamicMethod("MyMethod", typeof(void), null);
    ILGenerator il = dynamicMethod.GetILGenerator();
    il.Emit(OpCodes.Ldstr, "Hello, World");

**解决方案**：移除或替代对 `Emit` 的依赖，例如使用静态代理生成工具（如 **Source Generators**）。

### 4. 依赖 `AppDomain` 隔离和动态加载程序集的程序

**AppDomain** 用于隔离不同的程序执行环境，并允许在运行时加载和卸载程序集（DLL）。

**NativeAOT** 不支持 `AppDomain` 和动态程序集加载功能，所有依赖动态加载程序集的代码（如插件架构）都会失效。

#### 示例代码：

    AppDomain newDomain = AppDomain.CreateDomain("NewDomain");
    var asm = newDomain.Load("MyAssembly");

**解决方案**：使用静态引用的程序集，而不是在运行时加载新程序集。

### 5. 依赖 `System.Runtime.Serialization` 的复杂序列化场景

**NativeAOT** 对序列化的支持有限。像 `DataContractSerializer`、`BinaryFormatter` 这样的序列化器会使用反射和动态代码生成来序列化和反序列化对象。

#### 示例代码：

    var formatter = new BinaryFormatter();
    using (var stream = new MemoryStream())
    {
        formatter.Serialize(stream, myObject);
    }

**解决方案**：可以使用 **JSON.NET**、**System.Text.Json** 等序列化库，这些库可以通过配置在 NativeAOT 中支持。

### 6. 依赖于 `Assembly.Load` 的程序集动态加载

在 **NativeAOT** 中，所有代码在编译时必须被静态引用，因此不能在运行时通过 `Assembly.Load()` 加载额外的程序集。

#### 示例代码：

    Assembly asm = Assembly.Load("MyDynamicAssembly");

**解决方案**：通过编译时明确引用所有程序集，或使用静态方式引用而不是动态加载。

### 7. 依赖于 COM Interop 的程序

**NativeAOT** 目前不支持 **COM Interop**，因此任何与 **COM** 进行互操作的代码都会失效。

#### 示例代码：

    [ComImport, Guid("00000000-0000-0000-C000-000000000046")]
    public interface IMyComInterface { ... }

**解决方案**：如果必须与 COM 互操作，NativeAOT 不是一个合适的选择。

### 8. 依赖于 WebAssembly、Blazor 或动态 Web 资源加载的程序

如果项目是基于 **Blazor** 或 **WebAssembly**，这些类型的应用程序通常需要在浏览器环境中运行，并依赖于动态加载资源或代码，NativeAOT 目前并不支持此类场景。

### 9. 复杂的依赖注入（DI）场景

**NativeAOT** 对 **依赖注入（Dependency Injection, DI）** 的支持有限。特别是如果使用了反射或动态方法来解析依赖，NativeAOT 可能无法正确处理。

#### 示例代码：

    services.AddScoped<IMyService, MyService>();

**解决方案**：简化 DI 逻辑，并确保所有类型在编译时可以被静态解析。

## 程序能否生成本地代码的判别方法

### 1. 使用反射吗？

如果使用大量反射、动态类型、或依赖于 `Type`、`MethodInfo` 等进行运行时类型操作，项目可能不适合 NativeAOT。

### 2. 动态生成代码吗？

如果使用 `dynamic`、`Expression Trees` 的运行时编译，或者使用 `Emit` 生成 IL 代码，项目不适合 NativeAOT。

### 3. 依赖即时加载程序集或动态模块加载吗？

如果项目需要在运行时通过 `Assembly.Load` 或类似的机制动态加载模块或程序集，也不适合 NativeAOT。

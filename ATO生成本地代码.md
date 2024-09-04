# ATO 编译成二进制 .dll 文件


## 安装 Visual Studio 的 C++ 桌面开发工具
NativeAOT 编译（生成本地二进制文件）需要依赖 C++ 工具链，因此你需要确保安装了 Visual Studio 的 C++ 桌面开发工具。

步骤：
1. **打开 Visual Studio Installer：**
    - 在 Windows 开始菜单中搜索 Visual Studio Installer，然后打开它。
2. **修改 Visual Studio 安装：**
    - 在 Visual Studio Installer 中，找到你当前安装的 Visual Studio 版本，点击 修改（Modify）。
3. **选择 C++ 桌面开发工作负载：**
    - 在工作负载（Workloads）列表中，找到 使用 C++ 的桌面开发（Desktop development with C++），然后选中它。

## 验证 C++ 工具链的安装
```bash
where link
```

## 运行 NativeAOT 编译
编辑项目 `.csproj` 文件，增加以下内容：
```xml
  <PropertyGroup>
    <!-- 表示生成类库 (DLL) -->
    <OutputType>Library</OutputType>

    <!-- 或者根据目标平台选择，Windows：win-x64，Linux: linux-x64, macOS: osx-x64 -->
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>

    <!-- 启用 AOT 编译 -->
    <PublishAot>true</PublishAot>

    <!-- 关闭生成 NuGet 包的选项 -->
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
  </PropertyGroup>

  <ItemGroup>
    <!-- 添加 Microsoft.DotNet.ILCompiler 包，用于 NativeAOT 编译 -->
    <PackageReference Include="Microsoft.DotNet.ILCompiler" Version="8.0.0" />
  </ItemGroup>
```
在终端当前项目目录下，编译为本地代码：
```bash
dotnet publish -c Release -r win-x64 --self-contained
```
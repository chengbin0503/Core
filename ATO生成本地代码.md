# ATO ����ɶ����� .dll �ļ�


## ��װ Visual Studio �� C++ ���濪������
NativeAOT ���루���ɱ��ض������ļ�����Ҫ���� C++ ���������������Ҫȷ����װ�� Visual Studio �� C++ ���濪�����ߡ�

���裺
1. **�� Visual Studio Installer��**
    - �� Windows ��ʼ�˵������� Visual Studio Installer��Ȼ�������
2. **�޸� Visual Studio ��װ��**
    - �� Visual Studio Installer �У��ҵ��㵱ǰ��װ�� Visual Studio �汾����� �޸ģ�Modify����
3. **ѡ�� C++ ���濪���������أ�**
    - �ڹ������أ�Workloads���б��У��ҵ� ʹ�� C++ �����濪����Desktop development with C++����Ȼ��ѡ������

## ��֤ C++ �������İ�װ
```bash
where link
```

## ���� NativeAOT ����
�༭��Ŀ `.csproj` �ļ��������������ݣ�
```xml
  <PropertyGroup>
    <!-- ��ʾ������� (DLL) -->
    <OutputType>Library</OutputType>

    <!-- ���߸���Ŀ��ƽ̨ѡ��Windows��win-x64��Linux: linux-x64, macOS: osx-x64 -->
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>

    <!-- ���� AOT ���� -->
    <PublishAot>true</PublishAot>

    <!-- �ر����� NuGet ����ѡ�� -->
    <GeneratePackageOnBuild>false</GeneratePackageOnBuild>
  </PropertyGroup>

  <ItemGroup>
    <!-- ��� Microsoft.DotNet.ILCompiler �������� NativeAOT ���� -->
    <PackageReference Include="Microsoft.DotNet.ILCompiler" Version="8.0.0" />
  </ItemGroup>
```
���ն˵�ǰ��ĿĿ¼�£�����Ϊ���ش��룺
```bash
dotnet publish -c Release -r win-x64 --self-contained
```
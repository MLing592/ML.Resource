# MSBuild 全局配置

这个文件包含 MSBuild 的本地路径配置，用于避免每次都查找 MSBuild 路径。

## MSBuild 路径配置

当需要编译 Visual Studio 项目时，使用以下 MSBuild 路径：

`
D:\Visual Studio\2022\MSBuild\Current\Bin\MSBuild.exe
`

## 使用说明

- 使用完整路径调用 MSBuild，避免路径查找
- 支持所有标准的 MSBuild 参数
- 适用于 Visual Studio 2022 项目
- 可以通过参数指定 C++ 标准版本

## 常用编译命令示例

`ash
# 编译 Debug 配置，使用 C++14
"D:\Visual Studio\2022\MSBuild\Current\Bin\MSBuild.exe" YourProject.sln -p:Configuration=Debug -p:Platform=x64 -p:LanguageStandard=stdcpp14

# 编译 Release 配置，使用 C++17
"D:\Visual Studio\2022\MSBuild\Current\Bin\MSBuild.exe" YourProject.sln -p:Configuration=Release -p:Platform=x64 -p:LanguageStandard=stdcpp17

# 编译指定项目，使用 C++20
"D:\Visual Studio\2022\MSBuild\Current\Bin\MSBuild.exe" YourProject\YourProject.vcxproj -p:Configuration=Debug -p:Platform=x64 -p:LanguageStandard=stdcpp20

# 清理项目
"D:\Visual Studio\2022\MSBuild\Current\Bin\MSBuild.exe" YourProject.sln -t:Clean -p:Configuration=Debug -p:Platform=x64

# 重新编译，使用 C++17
"D:\Visual Studio\2022\MSBuild\Current\Bin\MSBuild.exe" YourProject.sln -t:Rebuild -p:Configuration=Debug -p:Platform=x64 -p:LanguageStandard=stdcpp17

# 编译多个配置
"D:\Visual Studio\2022\MSBuild\Current\Bin\MSBuild.exe" YourProject.sln -p:Configuration=Debug;Release -p:Platform=x64 -p:LanguageStandard=stdcpp17
`

## C++ 标准参数

可用的 LanguageStandard 值：
- stdcpp14 - C++14 标准
- stdcpp17 - C++17 标准  
- stdcpp20 - C++20 标准
- stdcppLatest - 最新 C++ 标准

## 注意事项

- 路径中包含空格，需要用双引号包围
- 确保 Visual Studio 2022 已正确安装
- 如果 MSBuild 路径发生变化，请更新此配置文件
- LanguageStandard 参数会覆盖项目文件中的设置
- 建议根据项目需求选择合适的 C++ 标准版本

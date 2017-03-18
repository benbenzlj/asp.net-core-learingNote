# 1.运行环境配置

# 2.程序调试

## 2.1 Visual Studio Code 会自动安装Core项目的调试环境

* Mono Runtime
* Mono Framework Assemblies
* OmniSharp \(Mono 4.6\)
* .NET Core Debugger

## 2.2修改launch.json文件

```js
"program": "${workspaceRoot}/bin/Debug/netcoreapp1.1/demo01.dll"
```

* netcoreapp1.1为框架版本，在项目文件demo01.csproject中TargetFramework
* demo01为项目名称




# NC Studio

当前版本：1.0.5

1.0.5 修复了从界面选择文件后后台任务被提前回收、界面持续等待的问题，分析和转换任务都会保留到线程结束。自检现在覆盖真实 Qt 事件循环中的文件加载、元数据展示和按钮触发的 GeoTIFF 转换。等待日志会显示实际阶段。

NC Studio 是一个 Windows 桌面工具，用来查看 NetCDF 元数据，并把可空间化的变量批量转换为 GeoTIFF。

界面使用 Python + PySide6 6.8.3，实现了极简浅色风格、`.nc/.nc4/.cdf` 拖拽、悬停和已选择状态反馈。数据处理使用 xarray、netCDF4、rasterio 和 pyproj，优先按 CF Convention 识别坐标、时间、坐标系和规则网格。

## 功能

- 拖拽或点击选择 NetCDF 文件。
- 自动展示绝对路径、变量清单、单位、时间、坐标系、维度、经纬度范围、分辨率和全局属性。
- 自动筛选可导出的规则二维空间变量。
- 对带时间维度的变量逐时间层导出 GeoTIFF。
- 对高度、深度、集合成员等额外维度逐层展开导出。
- 文件名格式为 `变量名-时间-空间范围.tif`，重名时自动编号。
- 显示进度、日志、跳过原因和错误信息。
- 取消任务时保留已完成的 GeoTIFF，并清理未完成的临时文件。

## 运行源码

```powershell
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements-dev.txt
.\.venv\Scripts\python.exe -m ncgeotiff
```

也可以直接打开一个文件：

```powershell
.\.venv\Scripts\python.exe -m ncgeotiff C:\data\sample.nc
```

## 命令行

查看元数据：

```powershell
.\.venv\Scripts\python.exe -m ncgeotiff inspect C:\data\sample.nc --report inspect.json
```

转换全部可空间化变量：

```powershell
.\.venv\Scripts\python.exe -m ncgeotiff convert C:\data\sample.nc C:\data\out --report convert.json
```

只转换指定变量：

```powershell
.\.venv\Scripts\python.exe -m ncgeotiff convert C:\data\sample.nc C:\data\out -v temperature -v precipitation
```

运行自检：

```powershell
.\.venv\Scripts\python.exe -m ncgeotiff --self-test --report selftest.json
```

验证实际界面的文件加载流程（指定自己的文件时只读取元数据，不批量转换）：

```powershell
.\dist\NCStudio\NCStudio.exe --gui-self-test "C:\data\sample.nc" --report gui-test.json
```

不指定文件时，`--gui-self-test` 自动创建小样本，经过界面选择文件、显示元数据和点击运行转换，并检查生成的 GeoTIFF。默认 `--self-test` 也包含这一检查，避免只测读取函数而遗漏界面后台任务问题。

## 打包

默认生成 `dist\NCStudio\NCStudio.exe`

```powershell
.\build.ps1
```

生成单文件 exe：

```powershell
.\build.ps1 -OneFile
```

构建脚本会先运行测试和自检，再用 PyInstaller 打包，并对打包后的 exe 再跑一次自检。

## 兼容范围

当前版本支持常见 CF 规则网格：

- 经纬度规则网格，包含纬度正序或倒序、经度正序或倒序。
- 投影坐标规则网格，优先读取 CF `grid_mapping`、WKT、`spatial_ref` 或 EPSG。
- 标准和非标准 CF 时间，文件名中会尽量使用可读时间标签。
- 浮点缺失值、整数变量、布尔变量、比例缩放和偏移量。

为了避免静默写出位置错误，当前版本会跳过曲线网格、非等间距网格、旋转经纬度网格、轨迹数据和缺少可靠空间坐标的数据，并在界面和日志中说明原因。

参考文档：xarray `open_dataset`、CF 解码、pyproj CF 坐标系、rasterio windowed writing、Qt QThread 和 PyInstaller hooks。链接见 `docs` 目录。



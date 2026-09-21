# WindVault · 测风数据管理工作台

<p align="center">
  <img src="icon.png" width="96" alt="WindVault logo">
</p>

WindVault 是风电测风数据的日常管理工具：测风塔 / 雷达数据的**导入、标准化、
质量控（QC）、台账管理**，并内置**站点总览地图**。数据存储采用开源格式
（SQLite / CSV），便于其它软件直接读取。

---

## 功能特性

### 台账管理
- 9 列台账：序号 / 站点号 / 设备类型 / 项目 / 位置 / 坐标 / 海拔 / 时间范围 / 完整率
- **项目**组织：同一测风项目下的多座塔共用一个项目名
- 点击任意表头排序（序号 / 海拔 / 完整率按数值比较），排序后行联动不串行
- 序号自 100001 起 6 位连续编号，删除站点留下的空号新站自动复用
- 站点新增 / 编辑：省 / 市 / 县三级级联下拉 + 详细地名手输

### 数据导入
- 异构格式自动识别：**NRG SymphoniePRO 文本导出、Molas 雷达、WRA 标准化
  格式**走精确解析；其它厂商数据走通用解析并标准化为统一通道
  （`Speed 10m Avg` 规范名）
- 逐通道标准化确认：类型 / 高度 / 方位 / 统计 / 后缀可改，取消勾选的通道不入库
- **原始数据 / 有效数据**：选「有效数据」时按标准 QC 规则剔除异常点，
  同时入库**原始 + 有效两套数据集**（原始留档、有效日常使用）
- 完整率按数据步长自动计算（1s / 5min / 10min / 60min / 1天 均支持），
  10min 数据一天 144 点

### 站点总览地图
- 天地图矢量 / 影像（需 Key）、OSM、谷歌、ArcGIS 影像、自定义瓦片 URL；
  离线中国省界与分级注记始终可显示（断网也不白屏）
- 组合搜索：综合 / 站点号 / 位置 / 坐标 / 序号 / 项目 / 海拔，另支持
  坐标半径筛选与年均风速分档着色
- 标记**单击**弹出站点综合信息（含「在台账中打开」），**双击**放大定位
- 图层切换 / 缩放 / 复位视图内嵌于地图右上工具组；地图固定内嵌主窗口

### 数据版本与导出
- 一个站点可同时存「原始数据」与「有效数据」两套；所有查看入口默认取
  有效数据，没有则自动回退原始数据
- 导出 CSV（utf-8-sig 带 BOM，Excel 直接打开不乱码）：先选数据版本
  （原始 / 有效，可同时导出），再勾选字段；文件名自动按
  `序号_站点号_日期` 生成

### 其它
- 自动更新：启动后台静默检查更新源，新版下载校验后重启自动应用
  （详见 [RELEASE.md](RELEASE.md)）
- 中英双语界面（设置内切换）、自启动、窗口尺寸记忆

## 快速开始

### 环境要求
- Windows 10/11
- Python 3.10+（3.13 实测）

### 安装运行
```bat
:: 1. 安装依赖
pip install -r requirements.txt

:: 2. 启动软件（唯一入口）
python windvault.py
```

首次启动自动在 `data/` 下创建台账库；菜单「工具 → 站点总览」打开地图，
「设置 → 站点地图 → 地图源与天地图 Key…」可配置底图与 Key。

### 打包发布
```bat
pip install pyinstaller
python build.py          :: 生成 dist/WindVault/
```
发布与自动更新的完整流程见 [RELEASE.md](RELEASE.md)。

## 目录结构

```
WindVault/
├── windvault.py            # 唯一入口 + 主窗口（菜单/台账/地图/更新）
├── core/                   # 数据与业务核心（无 UI）
│   ├── library.py          #   SQLite 台账库 CRUD（data/windkit.db）
│   ├── io_import.py        #   异构格式导入引擎 + 完整率计算
│   ├── qc_rules.py         #   标准 QC 规则引擎（导入/异常处理/导出共用）
│   ├── settings.py         #   JSON 设置持久化 + 自启动
│   └── i18n.py             #   中英翻译词典
├── ui/
│   ├── dialogs/            # 导入/导出/站点/通道/预览/标准化/QC/设置/关于
│   ├── modules/            # 台账库表 + 列宽分配规则
│   ├── map/                # 站点总览地图（QtWebEngine + 本地 Leaflet）
│   ├── style.py            # Fusion + 浅色主题
│   └── update_tools.py     # 自动更新混入（含软件版本号）
├── updater/                # 自动更新：多源拉取/下载校验/重启应用
├── assets/                 # 品牌资产（logo 矢量原稿与图标生成脚本）
├── data/                   # windkit.db 台账库 / settings.json / pcas.json
├── build.py                # PyInstaller 打包脚本
└── icon.ico / icon.png     # 应用图标（assets/gen_icons.py 生成）
```

## 数据存储

- 台账与时序数据存于 `data/windkit.db`（SQLite 单文件），通道注册、
  QC 结果、项目、坐标等全部结构化存储
- 库内（internal）模式时序写入独立宽表；外链（external）模式只登记
  原始文件路径，文件由用户自行保管
- 所有导出为带 BOM 的 UTF-8 CSV

## 文档

| 文档 | 内容 |
|---|---|
| [ARCHITECTURE.md](ARCHITECTURE.md) | 软件架构、数据模型、地图实现要点 |
| [RELEASE.md](RELEASE.md) | 发版与自动更新操作手册 |
| [REFACTOR_PLAN.md](REFACTOR_PLAN.md) | 重构记录与后续待办 |

## 反馈

- 作者：风
- 邮件：kyarazhan@qq.com
- 数据格式识别异常或新增设备类型，请附样例文件反馈

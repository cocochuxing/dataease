# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

本项目是 **DataEase v2**，一个开源的 BI（商业智能）平台。仓库是一个多模块 Maven 项目，包含 Spring Boot 后端和 Vue 3 + Vite 前端。

- **后端**：Java 21、Spring Boot 3.5、MyBatis-Plus、Flyway、Apache Calcite（自定义 `de` classifier）、EhCache
- **前端**：Vue 3、TypeScript、Vite、Element Plus、Pinia、AntV / ECharts
- **主应用类**：`core/core-backend` 中的 `io.dataease.CoreApplication`

## 仓库结构

```
.
├── pom.xml                 # 根 BOM；统一管理依赖版本
├── sdk/                    # 共享库（构建 core 前需先 mvn install）
│   ├── common/             # 工具类、DTO、通用异常、mybatis/knife4j 依赖
│   ├── api/                # 按领域划分的 API 接口 / DTO
│   │   ├── api-base
│   │   ├── api-permissions
│   │   └── api-sync
│   ├── distributed/        # 企业版 / 分布式（Nacos）支持
│   └── extensions/         # 数据源、视图、数据填报的插件 SDK
│       ├── extensions-datasource
│       ├── extensions-view
│       └── extensions-datafilling
├── core/
│   ├── core-backend/       # Spring Boot 应用
│   └── core-frontend/      # Vue 3 SPA
├── de-xpack/               # 闭源企业扩展的 Git 子模块（可能为空）
├── docker/                 # Docker Compose 运行文件
├── drivers/                # 构建时复制到镜像中的 JDBC 驱动
├── staticResource/         # 构建时复制到镜像中的静态资源
└── installer/              # 离线安装包打包工程
```

`de-xpack` 在 `.gitmodules` 中引用。如果该目录为空，表示闭源企业功能未引入；不要向其中添加代码。

## 常用命令

### 构建

```bash
# 1. 先将 SDK 模块安装到本地 Maven 仓库（构建 core 之前必须执行）
cd /mnt/d/dev/luomor/dataease
mvn clean install

# 2. 构建后端（默认 standalone profile；会将前端 dist 复制到 static 资源目录）
cd core
mvn clean package

# 3. 仅构建 SDK，跳过测试
mvn clean install -DskipTests
```

### 前端开发

```bash
cd core/core-frontend

# 安装依赖
npm install

# 启动开发服务器（Vite 在 8080 端口，将 /api 代理到 localhost:8100/de2api）
npm run dev

# Windows 环境
npm run dev:win

# TypeScript 类型检查
npm run ts:check

# 代码检查与自动修复
npm run lint
npm run lint:stylelint

# 生产构建
npm run build:base           # 单机版构建
npm run build:distributed    # 分布式/企业版构建
npm run build:lib            # 库构建
```

开发代理配置在 `core/core-frontend/config/dev.ts` 中：
- `/api/f` → `http://localhost:8100`（重写为 `/`）
- `/api` → `http://localhost:8100/de2api`

### 后端开发

```bash
cd core/core-backend

# 仅编译
mvn compile

# 打包（需先确保 ../core-frontend/dist 存在，构建过程会复制到 src/main/resources/static）
mvn clean package

# 使用指定 profile 打包
mvn clean package -P desktop
mvn clean package -P distributed
```

后端 profile 通过 `application.yml` 中的 `spring.profiles.active=@profiles.active@` 选择，默认为 `standalone`。

### 测试

`core/core-backend` 目前**没有可执行的测试套件**：`core/core-backend/pom.xml` 中的 `maven-surefire-plugin` 配置了 `<skip>true</skip>`，且 `src/test` 目录被 `.gitignore` 忽略。

如果 SDK 模块中存在测试，可运行标准 Maven 命令：

```bash
mvn test -pl sdk/common
```

### Docker

```bash
# 打包出 core/core-backend/target/CoreApplication.jar 后构建镜像
docker build -t dataease .

# 使用提供的 compose 运行（MySQL + DataEase）
cd docker
docker compose up -d
docker compose logs -f
```

Docker 镜像运行时依赖的路径：
- 应用配置：`/opt/apps/config/`
- 日志：`/opt/dataease2.0/logs/`
- 静态资源：`/opt/dataease2.0/data/static-resource/`
- 驱动：`/opt/dataease2.0/drivers/`

## 构建 Profile

| Profile | 用途 | 主要区别 |
|---------|------|---------|
| `standalone`（默认） | 单机安装 | 包含 H2 运行支持、Angus Mail，将前端复制到 static，使用 `db/migration` 的 Flyway 迁移 |
| `desktop` | 桌面版 / 社区版 | 包含 H2，使用替补权限实现，将前端复制到 static |
| `distributed` | 企业版 / 分布式 | 引入 `sdk/distributed`，排除 `io.dataease.substitute.**`，依赖 Nacos 和外部 MySQL（`de_manage`） |

前端构建也有对应的模式：`base`、`distributed`。

## 高层架构

### 后端（`core/core-backend`）

后端按领域组织在 `src/main/java/io/dataease` 下：

- **`datasource/`** — 连接管理、JDBC 提供者（`MysqlEngineProvider`、`H2EngineProvider`、`CalciteProvider` 等）、数据源类型定义（`Mysql`、`Pg`、`CK`、`Es` 等）、JDBC URL 安全策略。
- **`dataset/`** — 数据集分组、数据表、字段、SQL 数据集、权限、SQL Bot 集成。
- **`engine/`** — SQL 生成引擎。通过 `SQLProvider` 及 `Table2SQLObj`、`Dimension2SQLObj`、`Quota2SQLObj`、`WhereTree2Str` 等类将数据集/字段对象转换为 SQL。
- **`chart/`** — 图表定义、图表数据加载（`ChartDataManage`）、图表处理器（`ChartHandlerManager`）、视图服务门面。
- **`visualization/`** — 仪表板、可视化、模板、分享。
- **`exportCenter/`** — 异步导出任务。
- **`datasource/provider/CalciteProvider.java`** 与 **`extensions/extensions-datasource`** — 使用 Apache Calcite 实现跨数据源 SQL 联邦。
- **`substitute/`** — 当真实的企业版/权限实现不存在时使用的替补实现（在 `distributed` profile 中被排除）。
- **`license/`** — 许可证处理。
- **`websocket/`** — 长任务 WebSocket 端点。

### SDK（`sdk`）

SDK 模块仅作为依赖模块，定义共享接口、DTO 和扩展点：

- **`sdk/common`** — 基础异常、工具类、常量、通用依赖（knife4j、mybatis-plus、easyexcel、ehcache）。
- **`sdk/api/*`** — 按功能领域拆分的 API DTO 和接口。
- **`sdk/distributed`** — Nacos 服务发现及分布式运行组件。
- **`sdk/extensions/*`** — 用于扩展数据源类型、视图类型和数据填报功能的扩展点。

`core-backend` 依赖 `api-base`、`api-permissions`、`api-sync` 和 `common`。

### 前端（`core/core-frontend`）

- **`src/views/`** — 页面组件（`dashboard`、`chart`、`data-visualization`、`panel`、`share`、`mobile` 等）。
- **`src/custom-component/`** — 画布组件（图表、指标卡、媒体、形状、Tab、查询组件等）。
- **`src/components/`** — 可复用 UI 组件。
- **`src/store/`** — Pinia 状态管理；核心状态位于 `store/modules/data-visualization/`。
- **`src/api/`** — 按后端领域分组的 API 客户端模块。
- **`src/router/`** — Vue Router 配置。
- **`config/`** — 按模式拆分的 Vite 配置（`common.ts`、`dev.ts`、`base.ts`、`distributed.ts`、`lib.ts`）。

## 数据库迁移

- `standalone` 使用 Flyway，迁移脚本位于 `core/core-backend/src/main/resources/db/migration`。
- `desktop` 使用 `core/core-backend/src/main/resources/db/desktop` 中的脚本。
- `distributed` 禁用 Flyway（`spring.flyway.enabled=false`），依赖已初始化的外部 MySQL 库（`de_manage`）。

## 重要说明

- 根 `pom.xml` 只声明了 `<module>sdk</module>`。`core` 需在安装 `sdk` 后单独构建。
- `core/core-backend/pom.xml` 在 `generate-resources` 阶段将 `../core-frontend/dist` 复制到 `src/main/resources/static`。打包完整应用时请先构建前端。
- `de-xpack` 是闭源子模块。不要向其中添加开源代码，也不要假设其内容存在。
- `core/core-backend/src/test` 被 `.gitignore` 忽略；除非项目规范变更，否则不要在其中创建测试。
- 自定义 Calcite 构件（`org.apache.calcite:calcite-core:de`）从 `https://repository.fit2cloud.com/repository/fit2cloud-public/` 解析。

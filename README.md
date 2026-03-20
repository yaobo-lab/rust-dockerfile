# rust-dockerfile

用于 Rust 项目的 Docker 镜像构建脚本集合，主要面向 `aarch64/arm64` 交叉编译场景，同时保留了一些早期的 Debian/Ubuntu 版本镜像定义与配套工具。

## 仓库定位

这个仓库不是一个单一应用，而是一组可复用的构建环境定义，目标是解决以下问题：

- 为 Rust 项目准备统一的 Docker 构建环境
- 在 `x86_64` 主机上交叉编译 `aarch64-unknown-linux-gnu`
- 为依赖 `cmake`、`protoc`、`bindgen`、`pkg-config` 的项目补齐基础工具链
- 为部分 ARM64 目标提供 `qemu` 运行支持

## 目录结构

```text
.
|-- README.md
|-- Dockerfile_debian
|-- Dockerfile_rk3308
|-- arm64/
|   |-- Dockerfile
|   |-- Dockerfile_aarch64
|   |-- Dockerfile_aarch64-2
|   |-- *.sh
|   |-- linux-runner
|   `-- toolchain.cmake
`-- tool/
    |-- protoc-3.20.1-linux-x86_64/
    `-- protoc-32.0-linux-aarch_64.zip
```

## 主要文件说明

### 根目录

- `Dockerfile_debian`
  - 基于 Debian 11。
  - 安装 Rust、`protoc` 和 `aarch64` 交叉编译工具链。
  - 适合想快速构建一个通用 Debian 交叉编译环境的场景。

- `Dockerfile_rk3308`
  - 基于 Ubuntu 20.04。
  - 安装 Rust 与 `aarch64` 交叉编译依赖。
  - 从命名看更偏向 RK3308/ARM64 设备的构建环境。

### `arm64/`

这个目录下的内容更完整，也更接近一个可复用的 `cross-rs` 风格交叉编译镜像。

- `arm64/Dockerfile`
  - 入口 Dockerfile。
  - 通过 `common.sh`、`cmake.sh`、`xargo.sh` 等脚本安装基础依赖。
  - 构建阶段加入 `qemu`、`dropbear`、Linux runner、CMake toolchain 等配置。
  - 已配置好 `aarch64-unknown-linux-gnu` 相关环境变量。

- `arm64/Dockerfile_aarch64`
  - 较早期的 Ubuntu 20.04 版本方案。
  - 逻辑较简单，适合作为历史参考或最小化自定义基础。

- `arm64/Dockerfile_aarch64-2`
  - 基于已有镜像再次补充 `protoc`。
  - 更像是二次封装，而不是完整独立入口。

- `arm64/common.sh`
  - 安装基础构建依赖。
  - 处理 Ubuntu 多架构源配置。

- `arm64/cmake.sh`
  - 下载并安装固定版本的 CMake。

- `arm64/xargo.sh`
  - 安装 `xargo`。

- `arm64/toolchain.cmake`
  - 给 CMake 项目做交叉编译时使用的工具链配置。

## 已包含的工具

从当前仓库内容来看，镜像构建过程会涉及以下能力：

- Rust / Cargo
- `aarch64-linux-gnu-gcc` 交叉编译工具链
- `pkg-config`
- `libssl-dev`
- `libclang-dev`
- `cmake`
- `xargo`
- `protoc`
- `qemu-user-static` 或对应 ARM64 runner 支持

其中 `tool/` 目录已经包含了 `protoc` 相关文件，避免在构建时额外查找二进制。

## 快速开始

### 1. 构建 Debian 交叉编译镜像

```bash
docker build -f Dockerfile_debian -t rust-debian-aarch64:1.0.0 .
```

### 2. 构建 RK3308/Ubuntu 版本镜像

```bash
docker build -f Dockerfile_rk3308 -t rust-rk3308-aarch64:1.0.0 .
```

### 3. 构建 `arm64` 主镜像

如果要使用 `arm64/` 下的完整方案，通常需要把构建上下文切到该目录：

```bash
cd arm64
docker build -f Dockerfile -t rust-cross-arm64:1.0.0 .
```

## 典型使用方式

构建完镜像后，可以把业务项目目录挂载进容器，在容器内执行交叉编译：

```bash
docker run --rm -it \
  -v "$(pwd):/work" \
  -w /work \
  rust-cross-arm64:1.0.0 \
  cargo build --target aarch64-unknown-linux-gnu --release
```

如果项目依赖 CMake 或 `bindgen`，`arm64/Dockerfile` 方案通常更合适，因为它已经补齐了：

- 交叉编译 linker / ar / cc / cxx 环境变量
- `CMAKE_TOOLCHAIN_FILE`
- `BINDGEN_EXTRA_CLANG_ARGS`
- `PKG_CONFIG_ALLOW_CROSS`
- `QEMU_LD_PREFIX`

## 使用建议

- 优先使用 `arm64/Dockerfile` 作为主入口，它的工具链配置最完整。
- `Dockerfile_debian` 适合做基础环境或快速验证。
- `Dockerfile_rk3308` 更适合特定板卡/项目的历史环境复用。
- `Dockerfile_aarch64` 与 `Dockerfile_aarch64-2` 可以保留作兼容或迁移参考，但不建议作为首选入口。

## 注意事项

- 部分基础镜像使用的是私有镜像仓库地址，直接构建前需要确认本机可访问。
- 仓库内存在多个历史版本 Dockerfile，能力并不完全一致。
- 某些 Dockerfile 中使用了固定版本工具或手工复制的 `protoc`，升级时建议同步检查兼容性。
- 当前仓库更像“内部构建环境沉淀”，如果要公开复用，建议后续继续补充：
  - 支持的宿主机平台说明
  - 目标架构列表
  - 示例 Rust 项目
  - 与 `cross`/CI 的集成示例

## 推荐后续优化

如果你准备长期维护这个仓库，建议下一步做这几件事：

- 统一镜像命名与标签规范
- 明确每个 Dockerfile 的状态：`active`、`legacy`、`experimental`
- 补充一个最小示例项目验证 `cargo build --target aarch64-unknown-linux-gnu`
- 增加 CI，至少校验 Dockerfile 能成功构建
- 清理失效注释、无效路径和历史遗留命令

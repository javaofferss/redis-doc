# Redis 源码编译与启动指南

## 环境要求

- Linux 系统（本文基于 Linux Mint 21.3 / Ubuntu 22.04）
- gcc（建议 >= 9.0）
- make
- 可选：cmake、libssl-dev（如需 TLS 支持）

安装基础依赖（Ubuntu/Mint）：

```bash
sudo apt-get install -y gcc g++ make libssl-dev
```

## 一、编译 Redis

### 1. 进入项目根目录

```bash
cd /home/cmj/CLionProjects/redis-doc
```

### 2. 基础编译（不含扩展模块）

```bash
make -j "$(nproc)"
```

- `-j "$(nproc)"` 表示使用所有 CPU 核心并行编译，加快速度
- 编译产物在 `src/` 目录下，主要是：
  - `src/redis-server` — Redis 服务端
  - `src/redis-cli` — Redis 命令行客户端
  - `src/redis-benchmark` — 性能基准测试工具

### 3. 带 TLS 和模块的完整编译（可选）

```bash
export BUILD_TLS=yes BUILD_WITH_MODULES=yes INSTALL_RUST_TOOLCHAIN=yes
make -j "$(nproc)" all
```

注意：完整编译需要额外依赖（cmake、python3、rust 工具链等），参考 README.md。

### 4. 清理编译产物

```bash
make clean        # 清理 src/ 下的编译结果
make distclean    # 清理所有，包括 deps/ 下的依赖库
```

## 二、启动 Redis

### 1. 最简启动（前台运行）

```bash
./src/redis-server
```

- 默认监听端口 6379
- 按 Ctrl+C 停止

### 2. 后台守护进程启动

```bash
./src/redis-server --daemonize yes
```

### 3. 指定端口和常用选项

```bash
./src/redis-server --daemonize yes --port 6399 --protected-mode no --save ""
```

参数说明：
- `--daemonize yes` — 后台运行
- `--port 6399` — 监听端口（避免与系统已有 Redis 冲突）
- `--protected-mode no` — 关闭保护模式（仅限本地开发）
- `--save ""` — 禁用 RDB 持久化（开发调试用）

### 4. 使用配置文件启动

```bash
./src/redis-server redis.conf
```

项目自带 `redis.conf`（基础配置）和 `redis-full.conf`（含模块的完整配置）。

## 三、连接和测试

### 1. 连接 Redis

```bash
./src/redis-cli -p 6399
```

### 2. 验证是否正常

```bash
./src/redis-cli -p 6399 ping
# 应返回: PONG
```

### 3. 查看服务器信息

```bash
./src/redis-cli -p 6399 INFO server
```

## 四、停止 Redis

```bash
./src/redis-cli -p 6399 shutdown
```

或者带 `nosave` 参数跳过持久化：

```bash
./src/redis-cli -p 6399 shutdown nosave
```

## 五、运行测试（可选）

```bash
make test
```

这会运行 Redis 的单元测试和集成测试，耗时较长。

## 六、项目源码结构概览

```
.
├── src/              # Redis 核心源码（C 语言）
│   ├── server.c     # 服务端主入口
│   ├── redis-cli.c  # CLI 客户端
│   ├── networking.c # 网络层
│   ├── db.c         # 数据库核心操作
│   ├── aof.c        # AOF 持久化
│   ├── rdb.c        # RDB 持久化
│   └── ...
├── deps/             # 第三方依赖（hiredis, lua, jemalloc 等）
├── tests/            # 测试用例（Tcl 脚本 + C 模块测试）
├── modules/          # 内置扩展模块（RediSearch, RedisJSON 等）
├── redis.conf        # 默认配置文件
├── redis-full.conf   # 完整配置（含模块）
└── Makefile          # 顶层构建入口
```

## 七、CLion 中开发和调试 Redis

Redis 使用 Makefile 构建（没有 CMakeLists.txt），CLion 支持直接打开 Makefile 项目。

### 1. 打开项目

- 打开 CLion → **File → Open**
- 选择项目根目录 `/home/cmj/CLionProjects/redis-doc`
- CLion 会提示项目类型，选择 **Open as Makefile Project**（作为 Makefile 项目打开）

> 如果 CLion 没有自动识别，也可以：File → New CMake Project from Sources，但推荐 Makefile 方式。

### 2. 配置 Makefile 项目参数

打开后 CLion 会弹出 Makefile 设置，或者手动配置：

**Settings → Build, Execution, Deployment → Makefile**

- **Build directory**: 保持默认（项目根目录）
- **Build options**: `-j $(nproc)`（并行编译）
- **Pre-configuration commands**（可选）: 留空即可

点击 **Reload Makefile Project**，CLion 会运行 `make --just-print` 解析编译命令，从而获得代码索引（头文件路径、宏定义等）。

### 3. 配置 Run/Debug Configuration

点击顶部菜单 **Run → Edit Configurations → + → Makefile Application**：

| 字段 | 值 |
|------|------|
| Name | redis-server |
| Executable | `/home/cmj/CLionProjects/redis-doc/src/redis-server` |
| Program arguments | `--port 6399 --protected-mode no --save ""` |
| Working directory | `/home/cmj/CLionProjects/redis-doc` |
| Before launch | 保留默认的 Build（即 make） |

> **注意**：Executable 必须使用绝对路径，CLion 不会自动拼接 Working directory。

同理可以再添加一个 redis-cli 的配置：

| 字段 | 值 |
|------|------|
| Name | redis-cli |
| Executable | `/home/cmj/CLionProjects/redis-doc/src/redis-cli` |
| Program arguments | `-p 6399` |
| Working directory | `/home/cmj/CLionProjects/redis-doc` |

### 4. 编译

- 点击顶部 **Build → Build Project**（或快捷键 Ctrl+F9）
- CLion 会调用 `make -j $(nproc)` 进行编译
- 编译输出在底部 Messages 窗口查看

### 5. 运行和调试

- 选择顶部的 **redis-server** 配置
- 点击绿色三角 ▶ 运行，或点击虫子图标 🪲 启动调试
- 调试时可以：
  - 在 `src/server.c` 的 `main()` 函数设置断点
  - 单步跟踪启动流程
  - 在 `src/networking.c` 中跟踪客户端连接处理
  - 在各命令实现文件（如 `src/t_string.c`）中断点调试具体命令

### 6. 调试技巧

**推荐的断点位置（适合学习 Redis 源码）：**

| 文件 | 函数 | 说明 |
|------|------|------|
| `src/server.c` | `main()` | 服务启动入口 |
| `src/server.c` | `initServer()` | 初始化流程 |
| `src/networking.c` | `readQueryFromClient()` | 客户端请求入口 |
| `src/server.c` | `call()` | 命令执行核心 |
| `src/t_string.c` | `setCommand()` | SET 命令实现 |
| `src/t_string.c` | `getCommand()` | GET 命令实现 |
| `src/db.c` | `lookupKeyRead()` | 键查找逻辑 |

**调试流程示例：**

1. 在 `readQueryFromClient()` 设置断点
2. 启动调试模式运行 redis-server
3. 打开终端执行 `./src/redis-cli -p 6399`，然后输入 `SET foo bar`
4. CLion 会在断点处暂停，你可以单步跟踪一条命令从接收到执行的完整流程

### 7. 核心概念：CLion 调试的本质

CLion 调试 C 项目时，**不是直接运行 .c 源文件**，而是运行编译好的二进制可执行文件。

```
你在 CLion 做的事：            等价于终端操作：
─────────────────────────     ────────────────────────
Build Project (Ctrl+F9)   →   make -j $(nproc)
Run ▶                     →   ./src/redis-server --port 6399 ...
Debug (虫子图标)           →   gdb ./src/redis-server（带断点）
```

**为什么不能直接运行 server.c？**

Redis 由几十个 .c 文件组成，它们编译后链接成一个可执行文件：

```
server.c ─┐
networking.c ─┤
db.c ─┤
aof.c ─┤──→ [编译+链接] ──→ src/redis-server（二进制文件）
rdb.c ─┤
t_string.c ─┤
...（约 100 个 .c 文件）─┘
```

单独编译 `server.c` 会报 "undefined reference" 或找不到头文件（如 `lua.h`），
因为它依赖其他文件的函数和 `deps/` 目录下的库。

### 8. CLion 常见报错与解决

#### 报错：`File 'redis-server' not found in directory`

**原因**：Executable 路径不对。

**解决**：使用绝对路径 `/home/cmj/CLionProjects/redis-doc/src/redis-server`。

#### 报错：`fatal error: lua.h: No such file or directory`

**原因**：CLion 尝试直接编译单个 .c 文件，缺少 include 路径。

**解决**：
1. 先在终端完成一次编译：`make -j "$(nproc)"`
2. 回到 CLion，菜单 → File → **Reload Makefile Project**
3. CLion 会解析 Makefile 中的 `-I` 路径，红色报错消失

#### 代码跳转失效 / 大量红色波浪线

**原因**：CLion 没有正确解析 Makefile。

**解决**：
1. 确保终端 `make` 能编译通过
2. File → Reload Makefile Project
3. 等待右下角索引进度条走完

### 9. 完整操作流程总结

```
第一步（只需做一次）
  终端: cd /home/cmj/CLionProjects/redis-doc && make -j "$(nproc)"
  → 产出 src/redis-server

第二步（CLion 配置）
  1. File → Open → 选择项目根目录 → Open as Makefile Project
  2. File → Reload Makefile Project（等索引完成）
  3. Run → Edit Configurations → + → Makefile Application
     Executable: /home/cmj/CLionProjects/redis-doc/src/redis-server
     Arguments:  --port 6399 --protected-mode no --save ""

第三步（日常开发循环）
  1. 修改源码
  2. 点 Build 编译（或 Debug 时自动编译）
  3. 点 Debug 启动调试 → 断点命中 → 单步跟踪
  4. 另开终端用 redis-cli 发命令触发断点
```

### 10. 注意事项

- **不要用 `--daemonize yes`**：CLion 调试需要前台运行，守护进程模式会导致调试器脱离
- **索引加载需要时间**：首次打开项目后 CLion 需要几分钟解析 Makefile 和建立索引
- **如果代码跳转不准**：尝试 File → Reload Makefile Project 重新加载
- **修改代码后**：不需要手动 make，点 Debug/Run 时 CLion 会自动执行 Before launch 的 Build 步骤

## 常见问题

### Q: 端口被占用怎么办？

换一个端口启动，例如 `--port 6399`。

### Q: WARNING Memory overcommit 警告

这是内核参数提示，开发环境可忽略。如需消除：

```bash
sudo sysctl vm.overcommit_memory=1
```

### Q: 编译报错缺少依赖

```bash
sudo apt-get install -y gcc g++ make libssl-dev pkg-config
```

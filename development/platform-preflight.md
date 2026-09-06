# Engine 接入预检

[预检脚本](https://github.com/metasequoiaime/.github/blob/main/scripts/check-platform-adoption.py)保留在组织自动化仓库；本页维护操作说明。接入状态与迁移验收见[平台接入矩阵](../architecture/platform-adoption.md)。示例中的 Engine 提交是历史基线，运行时应明确选择待核对的提交。

在 macOS 或 Linux 上安装 Python 3.12+、CMake 和 Engine 的开发依赖，初始化 Engine 第三方子模块，然后从工作区父目录运行：

```bash
python3 .github/scripts/check-platform-adoption.py --workspace . --engine-ref cc21e42916cf93ce43051fec474e87eeeba305bf
```

macOS Homebrew 依赖可加 `--prefix-path /opt/homebrew`（按本机安装位置设置）。用待接入 Engine 的提交替换 `--engine-ref` 再跑一次，与平台当前 gitlink 的基线比较。脚本读取各平台的 `origin/main`；运行前自行获取最新引用，也可通过 `--platform-ref` 指定两仓共同存在的引用。

脚本只读取 Git 对象，忽略工作区未提交内容和脏子模块指针；把 Engine 实际固定的第三方提交和平台测试源码导出到临时目录，保存提交清单与配置、构建、CTest 日志。失败返回非零，目录保留供排障。它不改产品锁、不获取网络数据、不安装输入法。

此预检运行 iOS bridge 和 Linux 控制器的既有可移植测试，适合提前发现 Engine 升级导致的输入行为回归。它不覆盖 Windows、AppKit/UIKit、IBus/D-Bus、语音、词库打包或真实设备；通过后仍要执行平台原生和产品组合检查。

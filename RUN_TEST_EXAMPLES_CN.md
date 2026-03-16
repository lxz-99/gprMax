# gprMax（devel）测试与示例运行指南（中文）

> 目标：帮助你在本地**跑通单元测试**和**官方示例**，并快速定位常见问题。

## 1. 我先读了哪些关键代码入口（建议你也从这里看）

- 主入口包：`gprMax/`（核心求解器、场更新、网格、场景构建）。
- 命令行运行方式：`python -m gprMax <inputfile>`（README 和文档中的标准方式）。
- 单元测试目录：`tests/`（偏 Python 逻辑层测试）。
- 模型/回归验证目录：`testing/` 和 `reframe_tests/`（前者含模型与对比脚本，后者偏 HPC 回归）。
- 示例输入：`examples/`（你要跑通的主要对象）。

## 2. 最小可用环境（本地）

在仓库根目录执行：

```bash
python -m pip install -U pip
python -m pip install -e . --no-build-isolation
```

说明：
- `--no-build-isolation` 可避免某些受限网络环境在安装 build dependency 时失败。
- gprMax 首次安装会编译多个 Cython 扩展，耗时可能较长。

## 3. 先验证“能跑”

```bash
python -c "import gprMax; print('gprMax import OK')"
```

如果你看到 `ModuleNotFoundError: gprMax.cython.xxx`，基本就是扩展还没编译完成或安装中断，回到第 2 步重装。

## 4. 跑单元测试（建议先小后大）

### 4.1 快速冒烟
```bash
pytest -q tests/grid/test_fdtd_grid.py
```

### 4.2 全量 tests
```bash
pytest -q tests
```

## 5. 跑官方示例（你最关心）

### 5.1 A-scan（2D）
```bash
python -m gprMax examples/cylinder_Ascan_2D.in
python -m toolboxes.Plotting.plot_Ascan examples/cylinder_Ascan_2D.h5
```

### 5.2 B-scan（2D，多道）
```bash
python -m gprMax examples/cylinder_Bscan_2D.in -n 60
python -m toolboxes.Utilities.outputfiles_merge examples/cylinder_Bscan_2D
python -m toolboxes.Plotting.plot_Bscan examples/cylinder_Bscan_2D_merged.h5 Ez
```

## 6. 并行/加速运行（可选）

- MPI：
```bash
mpirun -n 4 python -m gprMax examples/cylinder_Ascan_2D.in --mpi 2 2 1
```

- GPU/CUDA（有 NVIDIA 环境时）：
```bash
python -m gprMax examples/cylinder_Ascan_2D.in -gpu
```

## 7. 常见报错与处理

1. **安装时无法联网拉依赖**
   - 现象：`Could not find a version that satisfies ...`
   - 处理：优先用已准备好的 conda 环境；或使用 `--no-build-isolation`。

2. **导入时报 cython 模块缺失**
   - 现象：`No module named gprMax.cython...`
   - 处理：重新执行 `python -m pip install -e . --no-build-isolation`，并等待编译完成。

3. **MPI/GPU 命令失败**
   - 现象：找不到 `mpirun`、CUDA 设备不可见等。
   - 处理：先跑 CPU 示例确认主流程，再逐步打开并行/加速。

## 8. 推荐你的执行顺序（最稳）

1) 安装（第 2 步）  
2) import 验证（第 3 步）  
3) `pytest -q tests/grid/test_fdtd_grid.py`  
4) `python -m gprMax examples/cylinder_Ascan_2D.in`  
5) `python -m gprMax examples/cylinder_Bscan_2D.in -n 60`

---

如果你愿意，我下一步可以直接按你机器环境（CPU/GPU、Linux/macOS、是否有 MPI）给你写一套“一键脚本”（含超时控制、失败重试、输出日志）。

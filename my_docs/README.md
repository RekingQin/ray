# Ray 代码仓库分析文档索引

这些文档基于 `/data/ruiqinchen/ray` 当前的代码和配置整理，分析过程中没有修改任何源码。重要结论都标注了文件路径和行号。行号对应文档编写时的代码状态，代码更新后可能会有偏移。

| 文档 | 内容 |
|---|---|
| [01-项目架构概览.md](01-项目架构概览.md) | 顶层目录、`src/ray` 与 `python/ray` 各模块职责、运行时进程架构图、技术栈 |
| [02-核心模块说明.md](02-核心模块说明.md) | 模块分层与依赖、GCS、Raylet、CoreWorker、对象存储、RPC 与配置、关键数据结构、Proto 接口 |
| [03-核心调用链路.md](03-核心调用链路.md) | 入口一览、`ray.init` 启动链路、Normal Task、Actor、put/get/wait、Worker 生命周期、线程模型 |
| [04-开发与测试指南.md](04-开发与测试指南.md) | Bazel 与 setup.py 构建、依赖管理、Lint、pytest/gtest/Java 测试、CI、Benchmark、文档构建 |
| [05-特殊约束与踩坑点.md](05-特殊约束与踩坑点.md) | 构建环境、运行时模型、测试和提交流程中需要注意的地方 |
| [06-关键文件索引.md](06-关键文件索引.md) | 按主题列出值得深入阅读的文件 |
| [07-可观测性专题.md](07-可观测性专题.md) | 可观测性全景：指标、日志、事件、State API、Dashboard、Tracing、Profiling、调试、OOM，以及自定义开发指南、配置速查与坑点 |
| [08-容错与稳定性专题.md](08-容错与稳定性专题.md) | 容错全景：故障检测、Task 重试、对象 lineage 重建与 spill、Actor 重启、GCS FT、节点 drain/抢占、PG、RPC 重试与混沌测试、OOM/cgroup 资源保护、Autoscaler、Job，以及 Serve/Train/Tune/Data/RLlib 容错、错误速查、配置速查、22 条代码与文档冲突、最佳实践 |

## 一图速览

```
 用户代码  ──>  ray.init / @ray.remote / ray.get           (python/ray/_private/worker.py)
                   │
                   ▼
            _raylet.pyx (Cython)  ──>  C++ CoreWorker       (src/ray/core_worker)
                   │                       │
       ┌───────────┼───────────────────────┼─────────────────────┐
       ▼           ▼                       ▼                     ▼
   GCS(控制面)   Raylet(节点调度)       Worker 进程(执行)       Plasma(共享内存对象)
  src/ray/gcs   src/ray/raylet        default_worker.py      src/ray/object_manager/plasma
                                      + CoreWorker           (以线程形式运行在 raylet 内)
```

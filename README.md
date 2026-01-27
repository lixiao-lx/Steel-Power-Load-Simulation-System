# Steel Power Load Simulation System
# 钢铁全流程电力负荷混合仿真系统

[![Python](https://img.shields.io/badge/Python-3.9%2B-blue)](https://www.python.org/)
[![AnyLogic](https://img.shields.io/badge/Simulation-AnyLogic-green)](https://www.anylogic.com/)
[![MATLAB](https://img.shields.io/badge/Model-MATLAB%2FSimulink%20FMU-orange)](https://www.mathworks.com/)
[![License](https://img.shields.io/badge/license-MIT-yellow)](LICENSE)

**Steel Power Load Simulation System** 是一个用于模拟和分析钢铁生产过程中电力负荷特性的多层混合仿真平台。

本项目创新性地采用了 **LLM（大语言模型）+ AnyLogic（离散事件仿真）+ MATLAB FMU（物理机理模型）** 的三层架构，实现了从模糊的工况描述到精确的秒级电力负荷波形的端到端生成。

**Steel Power Load Simulation System** is a multi-layer hybrid simulation platform for analyzing power load characteristics in steel production. It innovatively integrates **LLM (Configuration)**, **AnyLogic (Process Simulation)**, and **MATLAB FMU (Physical Modeling)** to achieve end-to-end generation from vague operational descriptions to precise second-level power load waveforms.


## 📖 项目简介

钢铁行业是电力消耗大户，其负荷具有冲击性强、波动大、非线性的特点。本系统旨在帮助：
*   🏭 **工厂端**：优化生产排程，降低需量电费，进行虚拟调试。
*   ⚡ **电网端**：分析电能质量，评估微电网稳定性。
*   🔬 **科研端**：为负荷预测算法提供高质量的合成数据。

### 核心优势
1.  **智能化配置**：利用内置冶金专家知识库的 LLM，自动将自然语言描述转化为专业的仿真参数。
2.  **机理与流程融合**：AnyLogic 负责宏观的物流与工序约束，MATLAB FMU 负责微观的电气物理响应。
3.  **高保真度**：基于 FMU (Functional Mock-up Unit) 标准封装物理模型，确保能耗计算的物理真实性。

## 🏗️ 系统架构

本系统采用 Python 作为数据流转层，连接 LLM、AnyLogic 和 MATLAB FMU 模块，形成闭环数据流。

### 三层架构详解
1.  **LLM 参数配置层**:
    *   内置冶金专家知识库 Prompt。
    *   解析用户需求（如“由100t剪切废钢生产一炉高强钢...”），生成两份标准 JSON 配置文件：`control_params.json` (Control Profile) 和 `config_params.json` (System Configuration Profile)。
2.  **AnyLogic 工序流程层**:
    *   作为仿真主引擎，读取 JSON 文件设定约束。
    *   模拟行车调度、钢包周转、工序等待等离散事件。
    *   负责触发各个工序的开始与结束。
3.  **MATLAB/Simulink 物理机理层**:
    *   单个能耗设备（EAF、LF、CC）封装为 FMU 模块。
    *   被 AnyLogic 调用，输入状态变量，输出实时功率、电压、电流。

```mermaid
graph TD        
    subgraph Python Layer [Python管控层 & LLM]
        User[用户输入: 生产场景/钢种] --> LLM[LLM (冶金专家知识库)]
        LLM --> JSON1[production_schedule.json<br>流程约束]
        LLM --> JSON2[device_params.json<br>物理参数]
    end

    subgraph AnyLogic Layer [AnyLogic 流程仿真层]
        JSON1 --> AL_Process[工序逻辑 & 物流调度]
        JSON2 --> AL_Param[参数初始化]
        AL_Process -- 状态触发 --> FMU_Call[FMU 调用接口]
    end

    subgraph FMU Layer [MATLAB 物理机理层]
        FMU_Call -- 输入: 设定值/时间 --> EAF_FMU[电弧炉 FMU]
        FMU_Call -- 输入: 温度/速度 --> LF_FMU[精炼炉 FMU]
        EAF_FMU -- 输出: P/Q/I/V --> Data[实时负荷数据]
    end

    Data --> Python_Vis[Python 可视化 & 分析]
```

## ✨ 主要功能 (Key Features)

*   **LLM 驱动的参数生成**：
    *   自动生成合理的冶炼时长、变压器档位、喷碳喷氧量等参数。
    *   支持“不同炉型”、“不同废钢种类”、“满负荷模式”等场景切换。
*   **FMU 模块化联合仿真**：
    *   支持 FMI 2.0 标准，集成 MATLAB/Simulink 编译的电气模型。
*   **全流程覆盖**：
    *   涵盖电弧炉 (EAF) -> 精炼炉 (LF) -> 连铸 (CC) -> 轧机 (Rolling Mill) 全流程。
*   **数据标准化**：
    *   Python 脚本自动清洗输出的原始数据，依照生产排程生成秒级到小时级的标准 CSV 数据文件。

## 📂 目录结构 (Directory Structure)

```text
steel-load-simulation/
├── LLM_config/            # [层级1] LLM配置生成模块
│   ├── prompt_templates/  # 冶金专家Prompt模板
│   ├── llm_client.py      # LLM API调用接口
│   └── output_json/       # 生成的JSON配置文件存放区
├── AnyLogic_simulation/   # [层级2] AnyLogic仿真工程
│   ├── models/            # .alp 仿真模型文件
│   └── database/          # 仿真所需的数据库文件
├── matlab_fmu_library/    # [层级3] FMU物理模型库
│   ├── eaf_model.fmu      # 电弧炉FMU
│   ├── lf_model.fmu       # 精炼炉FMU
│   └── rolling_mill.fmu   # 轧机FMU
├── python/                # Python脚本与工具
│   ├── run_eaf_fmu.py     # 运行电弧炉FMU脚本
│   ├── run_lf_fmu.py      # 运行精炼炉FMU脚本
│   ├── run_cc_fmu.py      # 运行连铸机FMU脚本
│   └── data_processing.py # AnyLogic输出数据处理脚本
├── data/
│   ├── PV/Wind/ESS/       # 光伏风电储能等
│   └── office_schedule/   # 办公照明负荷数据
├── results/               # 仿真结果输出目录
│   ├── visualization.py
│   ├── data_loader.py
│   └── .csv.plot          # 结果可视化图表
├── main.py                # 系统启动入口
├── requirements.txt       # 环境
└── README.md              # 说明文档
```

## 🚀 快速开始 (Quick Start)

### 依赖环境
*   Python 3.8+
*   AnyLogic Professional / University 
*   MATLAB/Simulink R2020b 及以上 

### 步骤 1: 生成配置 (Generate Config)
运行 Python 脚本，利用 LLM 生成仿真所需的 JSON 参数文件。

```bash
python main.py --mode config --steel_grade "SWRH82B" --scenario "high_throughput"
```
*输出：`LLM_config/output_json/` 目录下生成 `control_params.json` 和 `config_params.json`。*

### 步骤 2: 运行仿真 (Run Simulation)
启动 AnyLogic (可通过 Python 桥接)，加载上一步生成的 JSON 文件进行推理。

```bash
python main.py --mode simulation --config_dir ./LLM_config/output_json/
```

### 步骤 3: 结果分析 (Analyze Results)

```bash
python main.py --mode analyze --input ./results/simulation_output.csv
```

## 📊 示例结果 (Examples)

### 1. 复杂工况下的电弧炉电流波形
由 FMU 机理模型计算得出，包含高次谐波与非线性闪变特性。
*(在此处插入一张静态图片，展示电弧炉不同阶段（加料、熔化、精炼）的电流/功率变化)*

### 2. 钢铁冶炼流程工序图
*(在此处插入一张静态图片，展示从废钢加料到成品出炉的全流程工序图)*

### 3. 全流程功率叠加图
![alt text](modified_steel_plant_plot1.png)

## 🗓️ 路线图 (Roadmap)

- [√] **v0.1**: 完成 EAF 和 LF 的 MATLAB Simulink 模型并导出为 FMU。
- [√] **v0.2**: 搭建 AnyLogic 基础物流框架，实现 FMU 的静态调用。
- [ ] **v0.3**: 集成 LLM 模块，实现 text-to-config 的参数生成功能。
- [ ] **v0.4**: 引入强化学习 (RL) 代理到 AnyLogic 中，优化调度策略。

## 🤝 贡献指南 (Contributing)

非常欢迎社区贡献！尤其是具有 **冶金背景** 或 **AnyLogic/Simulink 仿真经验** 的开发者。

1.  Fork 本仓库
2.  新建 Feat_xxx 分支
3.  提交代码
4.  新建 Pull Request

## 📜 开源协议 (License)

本项目基于 **MIT 协议** 开源，详情请参阅 [LICENSE](LICENSE) 文件。

## 🔗 引用 (Citation)

如果您在研究中使用了本系统的架构或代码，请按如下格式引用：

```bibtex
@software{steel_hybrid_sim_2026,
  author = {Your Name Team},
  title = {Steel Power Load Simulation System},
  year = {2026},
  url = {https://github.com/yourusername/steel-load-simulation},
  note = {Integrates Continuous Modeling with Discrete Event Simulation}
}
```

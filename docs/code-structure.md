# 总体代码结构（建议版）

## 1. 设计原则

1. **五维解耦**：人 / 车 / 桩 / 场 / 况分层建模与存储，跨维度交互只能通过编排层发生。
2. **演进式 Demo**：先跑通 Demo1，再通过可插拔能力扩展 Demo2/3。
3. **前后端边界清晰**：前端仅做展示与交互，BFF 负责聚合与协议适配。
4. **可模拟、可替换**：场和况以规则模拟起步，后续可替换真实服务。
5. **可观测性优先**：每一层均输出结构化日志与事件，支撑 demo 展示与调试。

---

## 2. 仓库目录结构

```text
charging-reco/
├─ apps/
│  ├─ web/                               # React 前端
│  │  ├─ src/
│  │  │  ├─ app/                         # 路由、全局布局、主题
│  │  │  ├─ pages/
│  │  │  │  ├─ demo1-basic-match/
│  │  │  │  ├─ demo2-proactive-predict/
│  │  │  │  └─ demo3-voice-ecosystem/
│  │  │  ├─ modules/
│  │  │  │  ├─ user-profile/             # 人维度前端模型与组件
│  │  │  │  ├─ vehicle-profile/          # 车维度前端模型与组件
│  │  │  │  ├─ station-profile/          # 桩维度前端模型与组件
│  │  │  │  ├─ scene-profile/            # 场维度前端模型与组件
│  │  │  │  ├─ context-profile/          # 况维度前端模型与组件
│  │  │  │  ├─ recommendation/           # 推荐结果、解释、排序展示
│  │  │  │  ├─ map/                      # 高德地图封装
│  │  │  │  └─ voice/                    # Demo3 语音交互
│  │  │  ├─ stores/                      # Zustand 状态管理
│  │  │  ├─ services/                    # Axios API 封装
│  │  │  ├─ charts/                      # Echarts 图表
│  │  │  ├─ components/                  # AntD/Tailwind 通用组件
│  │  │  ├─ hooks/
│  │  │  ├─ utils/
│  │  │  └─ types/
│  │  ├─ public/
│  │  └─ package.json
│  │
│  ├─ bff-api/                           # FastAPI BFF 层
│  │  ├─ app/
│  │  │  ├─ main.py
│  │  │  ├─ api/
│  │  │  │  ├─ routes_demo1.py
│  │  │  │  ├─ routes_demo2.py
│  │  │  │  ├─ routes_demo3.py
│  │  │  │  ├─ routes_profiles.py
│  │  │  │  └─ routes_health.py
│  │  │  ├─ schemas/                     # 请求/响应 DTO
│  │  │  ├─ dependencies/
│  │  │  ├─ clients/                     # 调用 orchestrator/domain 的 client
│  │  │  ├─ middleware/
│  │  │  └─ config/
│  │  ├─ tests/
│  │  └─ pyproject.toml
│  │
│  └─ orchestrator/                      # 业务编排层
│     ├─ app/
│     │  ├─ main.py
│     │  ├─ workflows/
│     │  │  ├─ wf_demo1_match.py
│     │  │  ├─ wf_demo2_predict_and_push.py
│     │  │  └─ wf_demo3_voice_and_filter.py
│     │  ├─ policies/                    # 编排规则：降级、兜底、超时、A/B
│     │  ├─ ports/                       # 对下游 Domain Services 的抽象接口
│     │  ├─ adapters/                    # 接口实现（HTTP/RPC/本地）
│     │  ├─ events/                      # 编排事件模型
│     │  └─ config/
│     ├─ tests/
│     └─ pyproject.toml
│
├─ services/                             # 核心能力层（按五维拆分）
│  ├─ profile-human-service/
│  │  ├─ app/
│  │  │  ├─ main.py
│  │  │  ├─ domain/
│  │  │  │  ├─ entities.py
│  │  │  │  ├─ value_objects.py
│  │  │  │  └─ rules.py
│  │  │  ├─ repositories/                # SQLite
│  │  │  ├─ application/
│  │  │  ├─ api/
│  │  │  └─ tests/
│  │  └─ pyproject.toml
│  │
│  ├─ profile-vehicle-service/
│  │  ├─ app/
│  │  │  ├─ domain/
│  │  │  ├─ repositories/                # SQLite / 本地文件
│  │  │  ├─ application/
│  │  │  └─ api/
│  │  └─ pyproject.toml
│  │
│  ├─ profile-station-service/
│  │  ├─ app/
│  │  │  ├─ domain/
│  │  │  ├─ repositories/                # 桩静态 + 内存实时态
│  │  │  ├─ simulators/                  # 桩实时数据模拟
│  │  │  ├─ application/
│  │  │  └─ api/
│  │  └─ pyproject.toml
│  │
│  ├─ profile-scene-service/
│  │  ├─ app/
│  │  │  ├─ domain/
│  │  │  ├─ gaode_adapter/               # 高德地图 API 接入
│  │  │  ├─ simulators/                  # 场规则模拟
│  │  │  ├─ application/
│  │  │  └─ api/
│  │  └─ pyproject.toml
│  │
│  ├─ profile-context-service/
│  │  ├─ app/
│  │  │  ├─ domain/
│  │  │  ├─ gaode_adapter/               # 路网/天气/时间外部信息
│  │  │  ├─ simulators/                  # 况规则模拟
│  │  │  ├─ application/
│  │  │  └─ api/
│  │  └─ pyproject.toml
│  │
│  ├─ recommendation-service/
│  │  ├─ app/
│  │  │  ├─ strategies/                  # 规则策略与融合策略
│  │  │  ├─ explainability/              # 可解释推荐
│  │  │  ├─ ranking/
│  │  │  ├─ constraints/
│  │  │  ├─ api/
│  │  │  └─ tests/
│  │  └─ pyproject.toml
│  │
│  └─ prediction-service/
│     ├─ app/
│     │  ├─ features/
│     │  ├─ models/                      # 预测模型注册（Rule/ML）
│     │  ├─ inference/
│     │  ├─ api/
│     │  └─ tests/
│     └─ pyproject.toml
│
├─ libs/                                 # 共享库
│  ├─ py-common/
│  │  ├─ logging/
│  │  ├─ tracing/
│  │  ├─ errors/
│  │  ├─ config/
│  │  └─ contracts/                      # 跨服务 DTO/事件协议
│  └─ ts-common/
│     ├─ types/
│     ├─ constants/
│     └─ api-contracts/
│
├─ data/
│  ├─ static/
│  │  ├─ vehicles/                       # 10 台车全量数据
│  │  ├─ stations/                       # 桩全量静态数据
│  │  └─ mock_scenes_context/
│  ├─ sqlite/
│  │  ├─ user_vehicle.db
│  │  └─ migrations/
│  ├─ realtime/
│  │  └─ station_state_seed.json
│  └─ logs/
│
├─ tools/
│  ├─ simulators/
│  │  ├─ station_realtime_simulator.py
│  │  ├─ scene_simulator.py
│  │  └─ context_simulator.py
│  ├─ data_gen/
│  └─ scripts/
│
├─ deploy/
│  ├─ docker/
│  ├─ compose/
│  └─ env/
│
├─ docs/
│  ├─ architecture/
│  │  ├─ 01-bounded-context.md
│  │  ├─ 02-data-contract.md
│  │  ├─ 03-demo-roadmap.md
│  │  └─ 04-observability.md
│  ├─ api/
│  └─ code-structure.md
│
├─ Makefile
├─ pnpm-workspace.yaml
└─ README.md
```

---

## 3. 五维画像的代码边界（强约束）

### 3.1 人维度（Human）
- 输入：用户客观属性、历史充电行为偏好（仅用户相关）。
- 禁止：引入车辆参数、场站硬件指标、天气路况数据。
- 产出：`human_profile_score`、偏好标签、价格敏感度等。

### 3.2 车维度（Vehicle）
- 输入：车型原厂参数、电池状态、SOC、可接受功率、续航估计。
- 禁止：引入用户主观偏好、站点环境、外部天气路况。
- 产出：`vehicle_constraints`（兼容性、补能窗口、功率上限）。

### 3.3 桩维度（Station/Pile）
- 输入：站点位置、枪口、功率、占用、故障、排队。
- 禁止：直接读用户画像与车辆画像。
- 产出：`station_availability`、`service_quality`。

### 3.4 场维度（Scene）
- 输入：出行目的、路径阶段、剩余行程、停留时长（场景抽象量）。
- 禁止：用户身份属性、车辆硬件明细、桩硬件实时状态。
- 产出：`scene_urgency`、`detour_tolerance`。

### 3.5 况维度（Context）
- 输入：路网拥堵、时间段、天气、节假日等外部客观因素。
- 禁止：用户/车辆/场站内部状态。
- 产出：`context_adjustment_factor`。

---

## 4. 三个 Demo 的分层落地

## Demo1：基础匹配推荐
- Orchestrator 工作流：
  1. 拉取五维画像服务输出。
  2. 进入 recommendation-service 规则引擎。
  3. 返回 TopN 站点 + 解释。
- 模型层：以规则 + 打分卡为主。

## Demo2：主动预测 + 主动推荐
- 在 Demo1 基础上增加 prediction-service：
  - 预测下一次充电时间窗、地点倾向、SOC 风险。
  - 触发主动推荐策略（消息提醒/卡片）。
- 模型层：规则 + 轻量 ML（可先逻辑回归/GBDT）。

## Demo3：语音交互 + 生态筛选
- 在 Demo2 基础上新增：
  - 语音意图识别（地理范围、价格、品牌、周边业态）。
  - recommendation-service 增加生态过滤器（餐饮/商超/休息区）。
- 前端：语音输入、会话态与可解释结果展示。

---

## 5. 关键接口契约（建议）

### 5.1 BFF 对前端
- `POST /api/demo1/recommend`
- `POST /api/demo2/proactive/recommend`
- `POST /api/demo3/voice/recommend`
- `GET /api/profiles/{user_id}`

### 5.2 Orchestrator 对 Domain Services
- `get_human_profile(user_id)`
- `get_vehicle_constraints(vehicle_id)`
- `get_station_state(area_id)`
- `get_scene_features(trip_id)`
- `get_context_features(area_id, ts)`
- `rank_recommendations(input_bundle)`
- `predict_charging_need(user_id, vehicle_id, trip_context)`

---

## 6. 数据与存储映射

- **SQLite**（人 + 车）
  - `t_user_profile`
  - `t_user_preference_history`
  - `t_vehicle_spec`
  - `t_vehicle_battery_state`
- **内存字典**（桩实时态）
  - `station_realtime_state[station_id]`
- **规则模拟输入**（场 + 况）
  - 高德路网耗时
  - 天气/时间段映射规则

---

## 7. 状态管理与前端页面建议

- Zustand 切分：
  - `useHumanStore`
  - `useVehicleStore`
  - `useStationStore`
  - `useSceneStore`
  - `useContextStore`
  - `useRecommendationStore`
  - `useVoiceSessionStore`（Demo3）
- 页面组织：
  - `/demo1`：地图 + 站点列表 + 五维解释条
  - `/demo2`：预测面板 + 主动推荐时间线
  - `/demo3`：语音会话 + 筛选器 + 生态卡片

---

## 8. 开发迭代顺序（最小可用）

1. **M1（Demo1）**
   - 建五维 profile service 空壳 + recommendation-service 规则引擎。
   - 打通 BFF → Orchestrator → Domain Services 全链路。
2. **M2（Demo2）**
   - 增 prediction-service 与主动触发策略。
   - 增加通知/推荐理由可解释字段。
3. **M3（Demo3）**
   - 接语音输入 + 生态过滤器。
   - 做语音到结构化过滤条件的映射。

---

## 9. 命名规范与工程约束

- Python 包统一使用 `snake_case`，服务目录统一 `kebab-case`。
- 前端模块统一以 `feature` 划分，不以页面重复建逻辑。
- 所有跨层调用必须有契约（Pydantic/TypeScript 类型）。
- 日志字段统一：`trace_id`, `user_id`, `vehicle_id`, `demo_stage`, `latency_ms`。

---

## 10. 你可以直接开工的初始化任务

1. 创建 monorepo 基础目录与 `pnpm-workspace.yaml`。
2. 初始化 `apps/web` 与 `apps/bff-api`。
3. 先实现 Demo1 的一条最短路径：
   - 输入：用户 + 车辆 + 当前定位
   - 输出：Top3 推荐站点 + 三条解释
4. 再逐层接入 Demo2 与 Demo3。


<h1 align="center">Motion Parkour</h1>

<p align="center">
  <b>站到摄像头前，用身体去跑酷。</b><br>
  体态识别驱动的三车道像素跑酷 —— 横移换道，真的跳，真的蹲，张臂开盾，举手冲刺。
</p>

<p align="center">
  <img alt="Python 3.12" src="https://img.shields.io/badge/Python-3.12-3776AB?logo=python&logoColor=white">
  <img alt="MediaPipe" src="https://img.shields.io/badge/MediaPipe-0.10.35-0097A7?logo=google&logoColor=white">
  <img alt="OpenCV" src="https://img.shields.io/badge/OpenCV-4.14-5C3EE8?logo=opencv&logoColor=white">
  <img alt="pygame-ce" src="https://img.shields.io/badge/pygame--ce-2.5-4B8BBE">
  <img alt="tests" src="https://img.shields.io/badge/tests-114%20passed-brightgreen">
  <img alt="license" src="https://img.shields.io/badge/license-MIT-green">
</p>

---

## 这是什么

一个**用身体当手柄**的像素跑酷游戏。摄像头读出你的真实姿态，你在房间里横移、起跳、下蹲、张臂、举手，屏幕里的像素小人就跟着做同样的动作。

项目拆成两块，各自独立运行：

- **体态识别模块**（`camera_behavior_detect/`）—— 摄像头采集、MediaPipe 姿态推理、站立校准与动作判定，对外只暴露一个**六字段接口** `ControlState`。
- **游戏本体**（`game_ui_acheive/`）—— 三车道跑酷、波次关卡、像素渲染、HUD 与结算，只消费 `ControlState`，完全不碰关键点数据。

这样拆开最大的好处是游戏逻辑不依赖任何视觉库：**同一个游戏既能用摄像头玩，也能用键盘玩**，因为两条输入路径实现的是同一个接口。

## 演示

<p align="center">
  <img width="45%" alt="游戏画面" src="https://github.com/user-attachments/assets/c587e9b0-a804-4750-9734-b4438761ffd6">
  <img width="45%" alt="游戏画面" src="https://github.com/user-attachments/assets/5bd621da-03e6-40d1-a187-94749fe4e437">
</p>
<p align="center">
  <img width="30%" alt="游戏画面" src="https://github.com/user-attachments/assets/e818bedb-2fdc-43b4-abd2-f23e98bea000">
  <img width="30%" alt="游戏画面" src="https://github.com/user-attachments/assets/dfdbe14a-35cb-485d-83f5-3a9674716f58">
</p>

▶ [**体态模式实机录像**](https://github.com/user-attachments/assets/a7dca967-dab1-41c0-8210-641cf34ce593)

## 玩法

规则一句话：**砖墙和缺口要跳，隧道和低标牌要蹲**，不想做动作就直接换道躲开。

| 想做什么 | 键盘模式 | 体态模式 |
| --- | --- | --- |
| 换道 | `←` `→` 或 `A` `D` | 身体向左右横移并站稳 |
| 跳跃 | `空格` / `W` / `↑` | 真的跳一下 |
| 下蹲 | 按住 `↓` / `S` | 屈膝下蹲 |
| 张臂（护盾） | `O` | 双臂向两侧水平张开 |
| 举手（冲刺） | `U` | 双手举过肩膀 |

**两个技能**：张臂开护盾，消耗 20 点能量持续 3 秒，期间免疫伤害；举手进入冲刺，消耗 30 点能量持续 2 秒，速度提升到 1.65 倍。能量每 0.8 秒自动回复 1 点，上限 100。技能需要先结束当前动作、再重新做一次才能触发，持续保持不会反复扣能量。

界面操作：`P` 开关摄像头预览、`G` 说明页、`M` 静音、`Esc` 暂停、`F11` 全屏。失焦会自动暂停并松开下蹲。

## 难度与关卡

| 难度 | 速度倍率 | 波次间隔 | 关卡特点 |
| --- | --- | --- | --- |
| 简单 | 0.80 | 2.4–3.4 秒 | 以单点障碍和金币引导为主，并提示前方需要的动作 |
| 普通 | 1.00 | 1.7–2.5 秒 | 加入斜向连续、同车道连续与跳蹲连招 |
| 困难 | 1.25 | 1.15–1.8 秒 | 三道全封、双动作混合，几乎没有喘息 |

障碍按「波次」生成，共 8 种模式：

| 模式 | 形状 | 设计意图 |
| --- | --- | --- |
| `single` | 单条道一个障碍 | 基础节奏，六成概率在别的道铺金币 |
| `pair` | 两条道同动作封死 | 留一条安全道，可以硬吃动作也可以换道 |
| `split` | 两条道分别要跳和蹲 | 必须看清自己所在道，或换到空道 |
| `corridor` | 一条道障碍 + 金币铺路 | 金币提前铺在安全道上，引导换道 |
| `slalom` | 三条道依次各来一个 | 走 S 形路线，金币沿路线摆放 |
| `gauntlet` | 同一条道连续三个 | 固定节奏的跳跃或下蹲 |
| `combo` | 同一条道先跳后蹲 | 间隔 1.25 秒，考动作切换 |
| `wave3` | 三道同时封死 | 无法靠移动躲避，只能跳或只能蹲 |

金币和障碍是配合设计的：铺在安全道上引导走位、压在跳跃轨迹上方鼓励起跳、沿 S 形路线串联。波次之间会参考上一波的动作留出恢复时间——跳跃有 0.8 秒硬直，所以「跳跃波 → 下蹲波」的间隔会被强制拉长。

游戏速度从 300 在 120 秒内逐渐降到 190，越跑越慢，容错越来越高。

## 快速开始

### 键盘模式（不需要摄像头）

只需要 `pygame-ce` 和 `numpy`，不装 OpenCV、MediaPipe，也不用摄像头。

```powershell
cd preject_python_class/game_ui_acheive
py -3.12 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe game.py --fake
```

### 体态模式（需要摄像头）

先把视觉模块的环境配好，`setup_camera.bat` 会创建虚拟环境、安装依赖并下载姿态模型：

```powershell
cd preject_python_class/camera_behavior_detect
.\setup_camera.bat
```

然后在同一个 Python 环境里装上游戏依赖：

```powershell
cd ../game_ui_acheive
python -m pip install -r requirements.txt
python game.py
```

启动后先选控制方式：按 `K` 选键盘，按 `C` 选体态。体态模式会先做一次**站立校准**——退到全身入框、双手自然下垂、站定约 2 秒，等校准进度条走完就能开跑。换人、挪动摄像头或明显改变站立距离后，按 `C` 重新校准。

### 常用命令

```powershell
python game.py --difficulty hard        # 直接指定难度：easy / normal / hard
python game.py --density 0.58           # 障碍密度，数值越大波次越密集
python camera_demo.py                   # 单独打开体态调试窗口
python camera_demo.py --camera 1        # 切换到编号 1 的摄像头
python camera_demo.py --fake            # 用键盘模拟六字段接口，不用摄像头
python camera_demo.py --headless --duration 10   # 无窗口跑 10 秒，终端输出统计
```

体态识别模块的完整安装步骤、**45 秒动作自测清单**、逐字段接口说明和游戏接入示例，见 [README_CAMERA.md](preject_python_class/camera_behavior_detect/README_CAMERA.md)。

## 项目结构

```text
preject_python_class/
├── camera_behavior_detect/          体态识别模块（可独立使用）
│   ├── camera_control/
│   │   ├── control_state.py         ★ 六字段接口定义
│   │   ├── camera_controller.py     采集线程 + 异步推理 + 非阻塞 get_state()
│   │   ├── pose_estimator.py        MediaPipe LIVE_STREAM 适配
│   │   ├── calibration.py           站立校准
│   │   ├── detectors/               换道 / 跳跃 / 下蹲 / 张臂 / 举手 判定
│   │   ├── debug_view.py            调试窗口：骨架、六字段读数、FPS、延迟
│   │   └── config.py                所有识别阈值集中在这里
│   ├── models/                      Pose Landmarker Lite 模型
│   ├── tests/                       39 项视觉模块测试
│   ├── camera_demo.py               独立调试入口
│   └── README_CAMERA.md             安装、45 秒自测与接口详解
└── game_ui_acheive/               游戏本体
    ├── parkour_game.py            主循环、状态机、碰撞与计分
    ├── hud.py                     菜单 / 设置 / 说明 / 暂停 / 结算 / 游戏内 HUD
    ├── level.py                   八种波次模式与生成节奏
    ├── rendering/                 像素渲染器与粒子效果
    ├── input_adapter.py           键盘与体态的统一入口
    ├── camera_preview.py          摄像头帧转 pygame Surface
    ├── audio.py                   代码合成的音效与循环背景乐
    └── test_game.py               75 项游戏测试
```

## 游戏接口：六个字段

游戏和视觉模块之间只有这一层约定，字段是固定的：

```python
from camera_control.control_state import ControlState

state = controller.get_state()   # 非阻塞，每次读到的都是最新结果

state.lane                  # -1 左道 / 0 中道 / 1 右道
state.jump_triggered        # 一次性事件：刚起跳
state.crouch                # 当前是否保持下蹲
state.arms_open             # 当前是否双臂水平张开
state.hands_up              # 当前是否双手举过肩
state.tracking_confidence   # 0.0–1.0，综合关键关节可见度
```

接进自己的游戏循环：

```python
controller = CameraController(camera_index=0)
controller.start()
try:
    if not controller.calibrate(timeout=15):
        raise RuntimeError("校准尚未完成，请全身入镜、自然站稳后重试")

    while game.running:
        state = controller.get_state()
        if state.tracking_confidence >= 0.5:
            player.set_lane(state.lane)
            if state.jump_triggered:
                player.jump()
            player.set_crouch(state.crouch)
            game.set_arms_open(state.arms_open)
            game.set_hands_up(state.hands_up)
        else:
            game.show_tracking_warning()
        game.update_and_render()
finally:
    controller.stop()
```

几个约定：

- `jump_triggered` 是**上升沿事件**，每个更新周期读一次、当次处理完；不要缓存成 `True` 再在后续渲染帧里反复触发。
- 可信度 ≥ 0.70 正常更新；0.50–0.70 冻结换道并禁止新动作；低于 0.50 持续约 300 ms 就清除持续动作、保留车道。
- 想先在没接上摄像头的情况下跑通游戏，把 `CameraController` 换成 `FakeCameraController` 即可，两者接口完全一致。
- 退出时记得调用 `controller.stop()` 释放摄像头。

## 技术要点

- **不阻塞游戏循环**——采集线程只保留最新一帧，姿态推理走 MediaPipe LIVE_STREAM 异步回调。游戏永远读「最近一次结果」，不为推理等待。
- **以身体为基准**——校准先建立个人基准（身体中心、肩宽、身高），之后所有位移都用肩宽归一化，因此站位远近不影响判定。
- **防抖与滞回**——换道有进入/退出双阈值和确认时长；跳跃结合髋部上升高度、上升速度、落地复位与冷却；下蹲同时看髋部下沉和膝角。
- **失追踪降级**——追踪丢失会冻结游戏并给出文字提示，回到镜头里自动恢复；短暂丢帧不会误触发动作。
- **零外部素材**——像素角色、场景、粒子与背景音乐全部由代码生成，仓库里没有图片和音频文件。

## 测试

```powershell
cd preject_python_class/game_ui_acheive
python -B test_game.py                    # 75 项

cd ../camera_behavior_detect
python -m unittest discover -s tests      # 39 项
```

游戏测试运行在 SDL 虚拟视频/音频设备上，不打开窗口、不占用摄像头，覆盖玩法规则、波次结构、难度曲线、预览转换、界面按钮、状态机与资源清理；视觉测试覆盖换道滞回、跳跃上升沿、失追踪降级与校准流程。合计 **114 项，全部通过**。

## 已知限制

- 单目摄像头对「向上跳」的判断天然受限，跳跃阈值需要在真实环境下按 45 秒动作序列确认；换人或换位置后建议重新校准。
- 体态识别的真实准确率和音效听感仍需要在目标机器上实测，自动化测试不覆盖这两项。
- 目前只在 Windows + Python 3.12 上验证过。

## 后续计划

- [ ] 继续增加关卡模式与可用动作
- [ ] 结算页加入历史成绩记录

## 参考资料

Pygame 官方示例 [aliens.py](https://github.com/pygame/pygame/blob/main/examples/aliens.py)——参考其事件循环、按状态触发音效、以及混音器不可用时降级的组织方式。本项目没有复制其代码、图片或声音。

---

## 许可证

本项目基于 [MIT 许可证](LICENSE) 开源。

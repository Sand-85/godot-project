---
name: godot-fps-movement
description: 在 Godot 4.x 项目中创建可复用的 3D 第一人称移动控制器(CharacterBody3D + Capsule)。涵盖缓入缓出(slowstart 时间参数)、冲刺、下蹲(C 键)、跳跃(含高度限制开关)、鼠标视角、HUD 移速显示、空中惯性(air_control)、Minecraft 式自动上台阶(形状查询探测+平滑抬升+多点采样)。输入此类需求时使用本 skill 的逻辑要点与参数说明,不重复设计。
license: MIT
compatibility: Godot 4.5+
metadata:
  language: GDScript
  genre: first-person
---

# Godot 3D 第一人称移动控制器(可复用)

为 Godot 4.x 项目生成 3D 第一人称移动系统。占位用:角色 = 胶囊体(CapsuleMesh/CapsuleShape3D),地面 = 平面(PlaneMesh)。所有可调参数均用 `@export` 暴露,可在编辑器检查器直接调整。

## 节点结构

```
Player (CharacterBody3D)          # 挂 player.gd
├── MeshInstance3D                # CapsuleMesh,radius=0.5 height=2.0 (占位视觉)
├── CollisionShape3D              # CapsuleShape3D,radius=0.5 height=2.0 (物理,高度会被下蹲逻辑动态改)
├── Head (Node3D)                 # 位于站立眼高(默认 y=1.5),相机父级,下蹲时移动它带动相机
│   └── Camera3D
HUD (CanvasLayer)                 # 左上角调试文本
└── SpeedLabel (Label)            # 显示移速,格式 "Speed: %.2f m/s"
```

`@onready` 引用:`_head`、`_camera`、`_shape`(碰撞体 shape)、`_speed_label`(经 `get_node_or_null`)。

## 输入映射

| Action | 按键 | 说明 |
|--------|------|------|
| move_forward / move_back | W/S 或 ↑/↓ | 前后 |
| move_left / move_right | A/D 或 ←/→ | 左右 |
| sprint | Shift | 按住冲刺 |
| jump | Space | 跳跃 / 下蹲时站起 |
| crouch | C | 切换下蹲 |
| ui_cancel | Esc | 释放/捕获鼠标 |

`Input.get_vector("move_left","move_right","move_forward","move_back")` 读取方向;`direction = (transform.basis * Vector3(x,0,y)).normalized()` 转为世界方向。

## 移动逻辑(缓入缓出)

核心思想:用**时间参数 slowstart**(秒)替代固定加速度。速率 = `target_speed / slowstart`,数字越小加速/减速越快、响应越灵敏。用 `move_toward` 实现平滑缓入缓出。

```gdscript
@export_range(0.01, 1.0, 0.01) var slowstart: float = 0.15

# 仅在地面时调整水平速度:
var target := direction * target_speed
var rate := target_speed / maxf(slowstart, 0.001)
velocity.x = move_toward(velocity.x, target.x, rate * delta)
velocity.z = move_toward(velocity.z, target.z, rate * delta)
```

关键规则:
- **空中保留惯性为主**:`is_on_floor()` 为 false 时水平速度仅按 `air_control` 倍率缓动(默认 0.15,0=纯惯性),保留跳跃瞬间的大部分动量,允许轻微空中转向。
- **上台阶期间保持水平动量**(Minecraft 式):抬升进行时(`_step_target_y > 0`)水平速度按地面逻辑缓动,否则撞墙清速后会悬停-下落循环。
- 无输入时目标速度归零,自动形成减速缓出。
- 目标速度优先级:冲刺 `sprint_speed` > 行走 `walk_speed`;下蹲时再乘 `crouch_speed_multiplier`。

## 导出参数表(全部可在调试器调整)

| 参数 | 类型/默认 | 说明 |
|------|-----------|------|
| walk_speed | float 6.0 | 行走速度 m/s |
| sprint_speed | float 7.5 | 冲刺速度(健康成年男性奔跑约 7.5) |
| slowstart | range 0.01~1.0, 0.15 | 缓入缓出时间(秒),越小越灵敏 |
| air_control | range 0.0~1.0, 0.15 | 空中控制力倍率(相对地面缓动),0=纯惯性 |
| gravity | float 9.8 | 重力加速度 |
| jump_height | float 1.0 | 跳跃高度(米) |
| clamp_jump_to_capsule | bool true | 限制跳跃高度 ≤ 胶囊体半高 |
| crouch_height | range 0.3~2.0, 1.4 | 下蹲碰撞体高度(默认原高 2.0 的 0.7) |
| crouch_eye_height | range 0.0~2.0, 0.0 | 下蹲相机高度;0 = 按站立比例自动缩放 |
| crouch_speed | float 12.0 | 下蹲过渡速度 |
| crouch_speed_multiplier | range 0.1~1.0, 0.5 | 下蹲移速倍率 |
| jump_to_stand_when_crouched | bool true | 下蹲时按跳跃改为站起 |
| step_height | range 0.1~1.0, 0.5 | 自动上台阶最大高度(米) |
| step_lift_speed | range 0.5~12.0, 8.0 | 上台阶抬升速度(米/秒),Minecraft 手感约 8 |
| mouse_sensitivity | float 0.002 | 鼠标灵敏度 |
| invert_y | bool false | 反转垂直视角 |

## 跳跃

- `velocity.y = sqrt(2.0 * gravity * height)`,height 为 `jump_height`。
- `clamp_jump_to_capsule=true` 时 `height = minf(height, capsule.height * 0.5)`,保证跳不过半身。
- 下蹲 + 按跳跃:`jump_to_stand_when_crouched=true` 时改为 `_crouching=false`(站起),否则正常起跳。

## 下蹲(相机必须随 Head 平滑降低)

用指数平滑 `t = 1.0 - exp(-crouch_speed * delta)` 同时 lerp 两处:
1. `_shape.height` 由站立高度 `_stand_height` 过渡到 `crouch_height`(碰撞体)
2. `_head.position.y` 由站立眼高过渡到下蹲眼高(相机,经 Head 父级带动)

**相机高度陷阱**:下蹲眼高若写成高于场景中 Head 的实际站立高度,相机会上移。稳妥做法是 `crouch_eye_height == 0` 时按比例: `target_eye = _stand_eye_height * (target_height / _stand_height)`,保证永远低于站立视线。

**脚底贴地修正**:胶囊体以原点为中心,改变 height 后底部会抬起,需补偿:
```gdscript
var height_delta := old_height - _shape.height
if is_on_floor():
    position.y -= height_delta * 0.5
```
站立高度在 `_ready()` 从 `_shape.height` 与 `_head.position.y` 记录,不写死。

## 自动上台阶(Minecraft 式)

玩家接近 ≤`step_height` 的障碍时**提前平滑抬升**(不撞墙),抬升快、保持水平动量,像"滑"上去。状态:`_step_target_y > 0` 表示抬升中。

**探测(移动前、`is_on_floor()` 时,每帧)**——用扁盒**形状查询**而非单射线(单射线斜向接近台阶角落会擦过无法命中):

```gdscript
# 扁盒:覆盖玩家正面全宽(直径×0.06 厚),缓存为成员变量避免每帧分配
_step_probe_shape = BoxShape3D.new()
_step_probe_shape.size = Vector3(radius * 2.0, 0.06, radius * 2.0)

# 1. 低查询:脚底+0.05、前方偏移 0.15 处有障碍(intersect_shape,exclude=[get_rid()])
# 2. 高查询:脚底+step_height+EPSILON+0.04 处无障碍(高于阈值的墙被拦截;盒底需抬高 +0.04
#    避免与可上台阶的顶面重叠误拦;EPSILON≈0.03 吸收碰撞体实际尺寸偏差,过大则 0.6m 墙误放行)
# 3. 台阶顶采样:4 点向下射线(中心/前方 radius/左右侧向 ±radius 垂直于 dir),取最高命中
#    —— 斜向时扁盒仅与障碍角落重叠,侧向采样点才能落到障碍顶上
```

**抬升(探测成功后,move_and_slide 前)**——用 velocity.y 而非直接改 position(position 会被物理引擎复位):

```gdscript
_step_target_y = global_position.y + lift  # 探测成功时记录
if _step_target_y > 0.0:
    var remaining: float = _step_target_y - global_position.y
    if remaining <= 0.0:
        _step_target_y = 0.0
        velocity.y = 0.0
    else:
        # 接近目标时收敛速度,精确到顶即停(不过冲)
        velocity.y = maxf(velocity.y, minf(step_lift_speed, remaining / delta))
```

**关键陷阱**(实测踩过):
- 直接改 `global_position.y` 抬升会被 `move_and_slide` 重力复位——必须走 `velocity.y`;
- 抬升期间玩家 `is_on_floor()` 为 false,若水平速度按空中逻辑(air_control 低倍率)缓动,撞墙清速后无法恢复 → 悬停-下落死循环;抬升期间必须按地面逻辑缓动水平速度;
- 探测在 move_and_slide **前**(提前抬升不撞墙),探测距离用形状查询偏移 0.15(盒半宽即覆盖半径)。

## 鼠标视角

`_unhandled_input` 处理 `InputEventMouseMotion`:
- `rotate_y(-event.relative.x * mouse_sensitivity)`(水平,身体)
- `_head.rotation.x` 俯仰,clamp 到 ±89°
- 捕获鼠标:`Input.mouse_mode = MOUSE_MODE_CAPTURED`;Esc 切换捕获/可见。

## HUD 移速显示

每物理帧:`Vector2(velocity.x, velocity.z).length()` 得水平速度,`"Speed: %.2f m/s" % v` 写入 `SpeedLabel`。节点引用用 `get_node_or_null("/root/Main/HUD/SpeedLabel")` 容错。

## 自检清单

- [ ] 每个参数均 `@export` 且注释中文说明
- [ ] `_physics_process` 无每帧分配(探测形状缓存为成员变量,Vector2/3 字面量除外)
- [ ] 空中按 air_control 倍率缓动水平速度;上台阶抬升期间按地面逻辑
- [ ] 下蹲时相机高度 < 站立眼高,且平滑过渡
- [ ] 上台阶:正向/斜向 45° 均能登上;高于 step_height 的墙被拦截;无瞬移、极少撞墙帧
- [ ] `godot --headless --path .` 加载无报错;`--check-only --script <player.gd>` 语法通过
- [ ] 地面占位网格用 ShaderMaterial 网格着色器(1m×1m 可选,`fwidth` 抗锯齿)
- [ ] 验证方式:headless 测试脚本模拟 `Input.action_press` 移动,检测玩家 y 抬升与位移穿越

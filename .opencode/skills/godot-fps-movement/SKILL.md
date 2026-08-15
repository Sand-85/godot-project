---
name: godot-fps-movement
description: 在 Godot 4.x 项目中创建可复用的 3D 第一人称移动控制器(CharacterBody3D + Capsule)。涵盖缓入缓出(slowstart 时间参数)、冲刺、下蹲(C 键)、跳跃(含高度限制开关)、鼠标视角、HUD 移速显示、空中惯性。输入此类需求时使用本 skill 的逻辑要点与参数说明,不重复设计。
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
- **空中保留惯性**:`is_on_floor()` 为 false 时不修改水平速度,只受重力(`velocity.y -= gravity * delta`),保留跳跃瞬间的动量。
- 无输入时目标速度归零,自动形成减速缓出。
- 目标速度优先级:冲刺 `sprint_speed` > 行走 `walk_speed`;下蹲时再乘 `crouch_speed_multiplier`。

## 导出参数表(全部可在调试器调整)

| 参数 | 类型/默认 | 说明 |
|------|-----------|------|
| walk_speed | float 6.0 | 行走速度 m/s |
| sprint_speed | float 7.5 | 冲刺速度(健康成年男性奔跑约 7.5) |
| slowstart | range 0.01~1.0, 0.15 | 缓入缓出时间(秒),越小越灵敏 |
| gravity | float 9.8 | 重力加速度 |
| jump_height | float 1.0 | 跳跃高度(米) |
| clamp_jump_to_capsule | bool true | 限制跳跃高度 ≤ 胶囊体半高 |
| crouch_height | range 0.3~2.0, 1.4 | 下蹲碰撞体高度(默认原高 2.0 的 0.7) |
| crouch_eye_height | range 0.0~2.0, 0.0 | 下蹲相机高度;0 = 按站立比例自动缩放 |
| crouch_speed | float 12.0 | 下蹲过渡速度 |
| crouch_speed_multiplier | range 0.1~1.0, 0.5 | 下蹲移速倍率 |
| jump_to_stand_when_crouched | bool true | 下蹲时按跳跃改为站起 |
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

## 鼠标视角

`_unhandled_input` 处理 `InputEventMouseMotion`:
- `rotate_y(-event.relative.x * mouse_sensitivity)`(水平,身体)
- `_head.rotation.x` 俯仰,clamp 到 ±89°
- 捕获鼠标:`Input.mouse_mode = MOUSE_MODE_CAPTURED`;Esc 切换捕获/可见。

## HUD 移速显示

每物理帧:`Vector2(velocity.x, velocity.z).length()` 得水平速度,`"Speed: %.2f m/s" % v` 写入 `SpeedLabel`。节点引用用 `get_node_or_null("/root/Main/HUD/SpeedLabel")` 容错。

## 自检清单

- [ ] 每个参数均 `@export` 且注释中文说明
- [ ] `_physics_process` 无每帧分配(Vector2/3 字面量除外)
- [ ] 空中不改变水平速度
- [ ] 下蹲时相机高度 < 站立眼高,且平滑过渡
- [ ] `godot --headless --path .` 加载无报错;`--check-only --script <player.gd>` 语法通过
- [ ] 地面占位网格用 ShaderMaterial 网格着色器(1m×1m 可选,`fwidth` 抗锯齿)

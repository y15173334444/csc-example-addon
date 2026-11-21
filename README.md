# CSC Example Addon（公开 API 示例模组）

Create: Schematic Compute 公开 API 的第三方示例模组 —— **完整接入链的参考载体**：
第三方方块 ↔ 自定义节点 ↔ 图计算 ↔ 输出，闭环可跑。

A third-party example addon for Create: Schematic Compute's public API — the reference
carrier for the full integration chain: third-party block ↔ custom node ↔ graph ↔ output, a
runnable loop.

本仓是独立仓库（原住在主仓 `tools/example-addon/`），与主仓、电推仓并列：

This is a standalone repo (it used to live inside the main repo at `tools/example-addon/`),
sitting beside the main repo and the electric-push addon:

| 姊妹仓 / Sibling repo | 路径 / Path | 关系 / Relation |
|---|---|---|
| 主仓 Create: Schematic Compute | `../create-schematic-compute` · [GitHub](https://github.com/y15173334444/create-schematic-compute) | 编译期契约面（公开 API）+ 开发运行期本体 / the compile-time contract surface (public API) and the dev runtime body |
| CSC: Electric Push（电推原型） | `../csc-electric-push` · [GitHub](https://github.com/y15173334444/csc-electric-push) | 另一个附属、真实接入形态参照（自持图宿主 + Sable 执行面）/ the other addon and the live-integration reference (self-hosted graph host + Sable execution plane) |

## 演示内容 / What it demonstrates

| 节点 | 演示点 |
|---|---|
| `cscexample:signal` | **提供方直连**：求值回调里经 `ctx.level()` 读 `SignalProviderBlockEntity` 的信号值（TEXT 参数 `pos` = `"x y z"` 绝对坐标）；「参数即寻址」约定的最小实现 |
| `cscexample:heavy` | **可恢复重节点**：伪工作循环在边界检查 `ctx.sliceExpired()`，进度打包进检查点跨 tick 续算；把 `iterations` 调大到几百万即可用 `/cscperf` 观察耗时与挂起轮转 |

两者都走完整接入链：`CustomNodes.register(schema, behavior)` →
`CustomNodes.admitToHost(nodeId, CustomNodes.HOST_BLUEPRINT)`（**注册 ≠ 准入**：注册登记节点类型，还需显式登记进目标宿主的准入名单）。

## 构建 / Build

本仓对主仓产出的 CSC jar 编译，兄弟目录布局：

```bash
# ① 先出主仓 jar
../create-schematic-compute/gradlew jar

# ② 构建示例模组
./gradlew build
# 产物：build/libs/csc-example-addon-1.0.0.jar
```

第三方移植本示例时：把 `build.gradle` 里 `cscJar` 指向自己的 CSC jar 所在处
（`compileOnly` 即可——运行环境里由已安装的 CSC 提供 api）。

## 实机闭环 / In-game loop

1. 放置蓝图机与 **Example Signal Provider**（外观复用原版铁块占位，右键信号 0→5→10→15 循环）。
2. 蓝图机图内添加 **Example: Signal Read** 节点，TEXT 参数 `pos` 填提供方块坐标
   （如 `12 -58 -204`）；节点输出连 `REDSTONE_OUT` 或任意数值面。
3. 添加 **Example: Heavy (Resumable)**，`iterations` 调大，图运行后用
   `/cscperf`（看向蓝图机）观察其逐 tick 轮转推进与耗时。
4. 移除 CSC 后重进：节点降级为占位壳、图不损，提供方回归后自动恢复形状。

## 路线 / Roadmap

**主仓附属 → 动力类（舵机 / 周期变距）**：示例内容之外，本仓后续承接主仓第一方附属的
动力类形态——图给目标量、设备按动力执行（舵机角度伺服；周期变距的桨距随旋转相位变化）。
设备形态尚未拍板，实现不动；定形后按既有接入链模式（自定义节点 + 准入）扩展。

**Main-repo addon → power class (servo / cyclic pitch)**: beyond the example content this
repo will carry the first-party addon's power-class shape — the graph commands a target and
the device executes through kinetic power (a servo's angle; cyclic pitch varying once per
revolution). The device shape is not yet decided, so no implementation lands until it is;
the extension then follows the existing integration chain (custom nodes + admission).

兼容性按 **API 自己的版本号** 判别（`api.ApiVersion.CURRENT`，与模组版本解耦）：本示例只用
API 0.1.0 面，`init` 里以 `ApiVersion.isAtLeast(0, 1, 0)` 守门；`mods.toml` 的模组区间
`[1.3.0,)` 只表达最低发行线，不拿它推断 API 兼容性。API 版本 0.x = experimental（签名与
语义可变），首次破坏性变更升 1.0.0——那一刻才是首个稳定面的冻结点。

Compatibility is indexed by the **API's own version** (`api.ApiVersion.CURRENT`, decoupled
from the mod version): this example uses the API 0.1.0 surface and gates on
`ApiVersion.isAtLeast(0, 1, 0)` in `init`; the mod range `[1.3.0,)` only states the minimum
release line and never implies API compatibility. API versions in 0.x are experimental
(signatures and semantics may change); the first breaking change bumps to 1.0.0 — the freeze
point of the first stable face.

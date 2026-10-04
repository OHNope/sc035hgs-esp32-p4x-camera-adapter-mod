# SC035HGS Camera Adapter Mod

[中文](#中文) | [English](#english)

---

## 中文

本仓库在 [ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD](https://github.com/OHNope/ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD)（Espressif 官方相机转接板的 KiCad 转换版）基础上，修改为 SC035HGS 相机模组用的转接板。基础工程（导入修复、检查结果）的说明见它的 [README](https://github.com/OHNope/ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD#readme)。

### 原理图改动

依据：SC035HGS 数据手册 V0.8、模组图纸（24P FPC，0.5 mm 间距，0.3 mm 厚，触点在模组背面）、[ESP32-P4-Function-EV-Board v1.5.2 原理图](sources/esp32-p4-function-ev-board-schematics_v1.52.pdf)。模组图纸上标的传感器是 SC035GS；SC035HGS 是它的升级版，完全兼容，FPC 引脚定义不变。

**J2（模组 FPC）**：沿用原连接器 FPC-05F-24PH20（24P，0.5 mm，翻盖下接）和封装 `Camera_Sub:FFC_24P_0P5`。FPC 触点朝下插入时，模组第 n 脚落在焊盘 n 上，所以封装几何不变，只重做引脚定义。新符号 `SC035HGS_Mod:SC035HGS_FPC_24P` 的引脚名与模组图纸一致：

| J2 | 模组 | 网络 | J2 | 模组 | 网络 |
|----|------|------|----|------|------|
| 1 | LED-strobe | `/LED_STROBE` | 13–16 | NC | — |
| 2 | TRIG | `/TRIG` | 17, 20 | GND | GND |
| 3 | PWDN (xshutdn) | `/XSHUTDN` | 18 / 19 | MIPI_RCN / RCP | `/CSI_CLKN` / `/CSI_CLKP` |
| 4 | NC | — | 21 | MCLK | `/MCLK` |
| 5 / 6 | SDA / SCL | `/SENSOR_SDA` / `/SENSOR_SCL` | 22 | DOVDD 1.8 V | DOVDD_1V8 |
| 7, 8 | GND | GND | 23 | DVDD 1.5 V | DVDD_1V5 |
| 9 / 10 | MIPI_RDN0 / RDP0 | `/CSI_DATA0N` / `P` | 24 | AVDD 2.8 V | AVDD_2V8 |
| 11 / 12 | MIPI_RDN1 / RDP1 | `/CSI_DATA1N` / `P` | 25, 26 | 固定焊盘 | GND |

**电源**：新增 U3 ME6211C15M5G-N（LCSC C53100）提供 DVDD 1.5 V。上电顺序按手册 DOVDD → DVDD → AVDD：U1 EN 接 3V3，U3 EN 经 R12/C18 接 DOVDD_1V8，U2 EN 经 R14/C15 接 DVDD_1V5，各有约 1 ms 的 RC 延时，前一路稳定后下一路才开启。

**控制信号**：传感器侧都是 1.8 V（输入上限 DOVDD+0.3 V），主板和 J3 都是 3.3 V。三路都用 SN74LVC1G07 开漏缓冲（VCC=1.8 V，输入/输出可耐 5.5 V）做电平转换：

- XSHUTDN（U4）：输入 `/XSHUTDN_3V3` 来自 J1.5 和 J3.3，R24 上拉，默认使能。输出经 R10/C3 延时，XSHUTDN 在 DOVDD 上电或 J3.3 释放后约 12–14 ms 才达到高电平（手册只要求 AVDD 稳定后再拉高，T3 ≥ 0）。SW1 和 J3.3 拉低 XSHUTDN 是让传感器进入休眠，寄存器内容保留，并不是复位（手册 p.14）。原来 J1.5 直接接到 1.8 V 复位网络，主板 3.3 V 上拉和板上 1.8 V 上拉分压后约 2.55 V，超过 SC035HGS 的绝对最大值，所以改为缓冲。
- TRIG（U5）：J3.1 → TRIG，R26 2.2 kΩ 上拉，R25 让 J3 悬空时保持低电平。外触发模式下上升沿开始曝光。
- LED_STROBE（U6）：J2.1 → J3.2，R28 上拉到 3V3，R27 让传感器关断时输出保持低电平。

**J3**：JAE IL-G-4P-S3L2-SA（2.5 mm 间距，卧式，摩擦锁），线端配 IL-G-4S-S3C2-SA。1 TRIG_3V3、2 LED_STROBE_3V3、3 XSHUTDN_3V3、4 GND，接到 P4 开发板的 GPIO 排针。封装 `SC035HGS_Mod:JAE_IL-G-4P-S3L2-SA_1x04_P2.50mm_Horizontal` 按 JAE 图纸 SJ019575 画：1 脚编号与 JAE 一致（元件面看，开口朝 +Y 时 1 脚在左），孔径 1.1 mm，焊盘沿用社团的椭圆焊盘；板边应距针排 7–8.5 mm。注意：社团原有 IL-G 封装的 1 脚编号与 JAE 相反，用社团线束时要按 JAE 编号核对。EV 板 CSI 座的 CAM_IO0/CAM_IO1（对应 J1.5/J1.4）没有接到任何 GPIO，CAM_IO0 只有上拉（EV-Board 原理图 p.3：J9.11 CAM_IO0 只接 R123 10 kΩ 到 3.3 V，J9.12 CAM_IO1 的 R111 不贴），所以必须通过 J3 控制。J1.4 保持不接。

**C12/C13/C14**：值标为 NC 的三颗电容在原理图和 PCB 上都设为 DNP（不贴）。

**网络改名**：`RESET` → `XSHUTDN`（1.8 V）/ `XSHUTDN_3V3`（3.3 V），`XVCLK` → `MCLK`。新增器件放在项目库 `libraries/SC035HGS_Mod.*`（符号和封装），连接器出线方向等元数据在 `libraries/manifest.json`；`orcad_import` / `Camera_Sub` 与上游保持一致。

**固件注意**：
- 首次 I2C 访问：XSHUTDN 变高且 MCLK 已在运行后至少等 4 ms（手册 p.13 图 1-4 的 T4）。板上 XSHUTDN 由 R10/C3 延时，所以上电或释放 J3.3 后至少等约 20 ms 再访问传感器。
- 需要恢复默认寄存器时写 `0x0103=0x01`（软复位），拉低 XSHUTDN 不会清除寄存器。
- MCLK 为 24 MHz。写 `0x0100=0x01` 后至少等 7 ms，再写 `0x4418`/`0x4419`（SmartSens 配置说明）。
- 外触发模式：`0x3222[1]=1`。STROBE 使能：`0x3361[7:6]=00`。
- I2C 地址由模组内部的 SID0/SID1 决定（FPC 未引出），首次上电请扫描 0x30–0x33 确认。

### PCB

PCB 在 KiCad 里按新原理图更新后全部重做：旧铜皮全部删除，重新摆放和走线。

- **板框与叠层**：40 × 24 mm，R3 圆角。4 层，嘉立创 JLC04161H-7628 叠层，1.6 mm，沉金（ENIG）；加工参数在 [`pcb-fabrication-profile.json`](pcb-fabrication-profile.json)，Board Setup 的工艺下限也按它设置。
- **布局**：J1（接 P4 开发板的 FPC）在下边，J2（模组 FPC）在上边，J3 在右边、开口朝板外，SW1 在右上角。
- **层分配**：In1 整层 GND，In2 整层 DOVDD_1V8；顶层和底层走信号并铺 GND。3V3、DVDD_1V5、AVDD_2V8 走线，线宽按 SC035HGS 最大 120 mW 的电流算，对应网络类 `PWR_*`。
- **MIPI CSI-2**：三对差分全部走顶层、不打孔，100 Ω（线宽 0.2258 mm，间距 0.2032 mm），以 In1 为参考面；每次转弯只用一个 45° 折角，对内长度差为 0，三对之间的长度差 0.21 mm。
- **过孔**：全部 0.6/0.3 mm。
- **丝印**：字高 1.0 mm、线宽 0.15 mm（嘉立创下限）；放不下的位号移到 Fab 层。底面丝印为板名和 "Same-Side FFC" 提示。
- **清理**：删除了 Allegro 导入时留下的尺寸标注（MECH1/MECH2）、钻孔表、板外的空规则区和右边缘的旧 Route Keepin 规则区；底面丝印上 Espressif 的 logo 和 WEEE/ESD/无铅/无卤标志也已删除。
- **位置**：板子放在 A4 图框正中（板框 x 128.5–168.5 mm，y 93–117 mm），出图时板子在图框内。
- **检查**：ERC 0；DRC 0 错误、0 未连接，与原理图一致。剩余 87 条警告都是丝印：57 条是相邻 `Camera_Sub` 库封装的丝印互相碰到（封装丝印超出 courtyard），30 条是丝印压在焊盘上（主要是 J1/SW1，生产时由阻焊裁掉），不影响加工。

`sources/espressif-original/` 里是 Espressif 原板的 BOM、贴片坐标和 PCB 加工说明，对应的是旧版 32 mm 板，**不能用来给本板下单**。本板的 Gerber、BOM 和贴片文件由 `-fab` 发布生成，下单用最新的 `v0.9.1-fab`。

### 上电调试

首批板子上电时逐项确认，结果补记在这里：

- [ ] **MCLK 占空比**：Y1 没有可查的数据手册。在 J2.21 测 24 MHz 波形，占空比应在 45–55 %，高、低电平各至少 5 ns（SC035HGS 手册 p.37）。
- [ ] **各路电流**：手册 V0.8 p.36 的工作电流是空白的，分别测 DOVDD_1V8、DVDD_1V5、AVDD_2V8 的电流。按手册 p.2 的最大功耗 120 mW 推算，单路上限约为 DOVDD 67 mA、DVDD 80 mA、AVDD 43 mA，都在 ME6211 的额定范围内；测得值明显更大时先断电排查。
- [ ] **I2C 地址**：在 0x30–0x33 范围扫描，记下模组的实际地址。
- [ ] **J3 → P4 GPIO**：J3 接 EV 板的 GPIO 排针（J1，2×20）时，避开启动 strapping 引脚 GPIO34–GPIO38；排针上是 GPIO37（8 脚）、GPIO38（10 脚）和 GPIO36（24 脚）（EV-Board 原理图 p.3）。P4 复位期间本板会给 J3 加上固定电平：J3.3 经 R24 上拉到 3V3，J3.1 经 R25 下拉，DOVDD 有电且 STROBE 空闲时 J3.2 被 U6 拉低，接在 strapping 引脚上可能改变启动模式。J3.1（TRIG）和 J3.3（XSHUTDN）的 GPIO 设为推挽输出（U4/U5 的输入要求快边沿），J3.2（STROBE）设为输入，J3.4 接排针上的 GND。选定的 GPIO 记在这里。
- [ ] **J3 实物配合（可选）**：封装已按 JAE 图纸 SJ019575 核对；有 IL-G-4P-S3L2-SA 样品时，可再实际插装确认孔位和板边距离。

**已接受的风险**（2026-10-03 决定本版不改）：

- **突然断电时的 XSHUTDN**：手册图 1-5 要求断电前先拉低 XSHUTDN。3V3 突然断开时，C3（R10 × C3 = 10 ms）会让 XSHUTDN 在 DOVDD 已经下降后仍短暂保持高电平，可能超过 DOVDD + 0.3 V 的绝对最大值。能量很小，Espressif 原板的 RESET RC 也是如此。正常关机时先把 J3.3 拉低再断电即可避免。若要修改：在 R10 两端并联一个二极管（阳极接 /XSHUTDN，阴极接 DOVDD_1V8）。
- **U4 直接给 C3 放电**：U4 拉低 XSHUTDN 时，C3（1 µF）经 U4 输出直接放电，中间没有电阻。峰值灌电流只受 U4 输出电阻限制，超过推荐的 4 mA IOL，在几十微秒内可能接近 50 mA 的绝对最大值。若要修改：在 U4.4 与 R10/C3/J2.3 节点之间串 100–220 Ω。

**下一版待改**：U4 的 Description 字段写的是 "R10 + C3 set the >=4 ms power-up delay"，这个说法不对。手册的 4 ms（T4，p.13）是 XSHUTDN 变高到第一次 I2C 访问之间的时间，由固件等待（见上文“固件注意”）；R10/C3 的作用是让 XSHUTDN 在 AVDD 之后变高（T3 ≥ 0）。下一版改为 "R10 + C3 delay XSHUTDN high ~12 ms after DOVDD (T3 >= 0); firmware waits >= 4 ms (T4) before I2C"。这只是字段文字，不影响网表和 PCB，本版不改。

### 仓库关系

| Remote     | 仓库                                         | 用途                        |
|------------|----------------------------------------------|-----------------------------|
| `origin`   | `OHNope/sc035hgs-esp32-p4x-camera-adapter-mod` | 本仓库，改版开发            |
| `upstream` | `OHNope/ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD`  | 基础工程，只拉取，不推送    |

基础工程有更新时合并进来：

```bash
git fetch upstream
git merge upstream/main
```

工程文件名 `SCH_ESP32-P4_FUNCTION_EV_BOARD_MIPI_Camera_Sub_V1.1_20240529.*` 与基础工程保持一致，不要改名，否则上游修改无法直接合并。本仓库的 README.md 是改版说明，合并时 README.md 有冲突就保留本仓库的版本（`git checkout --ours README.md`）。

### 发布

[`.github/workflows/pcb-release.yml`](.github/workflows/pcb-release.yml) 调用共享流水线 [OHNope/kicad-release](https://github.com/OHNope/kicad-release)，标签规则、各次发布的内容和 Google Drive 配置都以那边的 README 为准。

| 标签 | 内容 |
|------|------|
| `v0.9.0` | 只有 SC035HGS 原理图，PCB 还是旧板 |
| `v0.9.1` / `v0.9.1-fab` | 第一版 PCB，用于打样。贴片坐标和 iBOM 不含不贴的 C12–C14 |

```bash
git tag -a v0.9.1 -m "v0.9.1: ..." && git push origin v0.9.1
```

```bash
git tag -a v0.9.1-fab -m "v0.9.1-fab: ..." "v0.9.1^{}" && git push origin v0.9.1-fab
```

`v0.9.1` 发布原理图/PCB 文档，并把已提交的整个工程目录传到 Drive；`v0.9.1-fab` 才发布 Gerber，且必须与 `v0.9.1` 指向同一个 commit，所以先推设计标签、再推 `-fab`。C12–C14 在 F.Paste 上仍有钢网开口，下单 SMT 时以 BOM 和贴片坐标为准。Google Drive 需要在 Settings → Secrets and variables → Actions 中设置 secret `GDRIVE_TOKEN` 和 variable `GDRIVE_FOLDER_ID`。

### kicad-auto

`.kicad-auto/` 是 kicad-auto 工具的本地状态（评审、语义分析、生成工作区），已加入 `.gitignore`，不提交。

### 版权与许可

设计版权归 Espressif Systems (Shanghai) Co., Ltd. 所有。本仓库**不授予任何许可**，详见 [LICENSE](LICENSE)。本仓库为非营利性的学习与参考分享，与 Espressif 无关联、未获其背书；如 Espressif 提出异议，将立即删除。

---

## English

This repository modifies [ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD](https://github.com/OHNope/ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD), the KiCad conversion of Espressif's camera adapter board, into an adapter for the SC035HGS camera module. See its [README](https://github.com/OHNope/ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD#readme) for the base project (import fixes, check results).

### Schematic changes

Sources: SC035HGS datasheet V0.8, the module drawing (24-pin FPC, 0.5 mm pitch, 0.3 mm thick, contacts on the module's bottom side), [ESP32-P4-Function-EV-Board v1.5.2 schematic](sources/esp32-p4-function-ev-board-schematics_v1.52.pdf). The module drawing names the sensor SC035GS; SC035HGS is its fully compatible upgrade, so the FPC pinout is unchanged.

- **J2 (module FPC)**: same connector (FPC-05F-24PH20, 24-pin 0.5 mm flip-lock, bottom contact) and footprint `Camera_Sub:FFC_24P_0P5`. With the FPC inserted contacts-down, module pin n lands on pad n, so only the pin assignment changes. The new symbol `SC035HGS_Mod:SC035HGS_FPC_24P` uses the module's pin names; see the table in the Chinese section.
- **Power**: U3 ME6211C15M5G-N (LCSC C53100) adds DVDD 1.5 V. Power-up order follows the datasheet (DOVDD → DVDD → AVDD): U1 EN from 3V3, U3 EN from DOVDD_1V8 through R12/C18, U2 EN from DVDD_1V5 through R14/C15. Each RC adds about 1 ms, so a rail starts only after the previous one has settled.
- **Control signals**: the sensor side is 1.8 V (inputs limited to DOVDD + 0.3 V), and the main board and J3 are 3.3 V. Three SN74LVC1G07 open-drain buffers at VCC = 1.8 V (5.5 V-tolerant input and output) translate them:
  - XSHUTDN (U4): input `/XSHUTDN_3V3` from J1.5 and J3.3, pulled up by R24 (enabled by default). R10/C3 on the output delay it: XSHUTDN reaches a valid high about 12–14 ms after DOVDD comes up or J3.3 is released (the datasheet only requires it after AVDD is stable, T3 ≥ 0). Pulling XSHUTDN low with SW1 or J3.3 puts the sensor to sleep with its registers kept; it is not a reset (datasheet p.14). J1.5 used to sit directly on the 1.8 V reset net, where the main board's 3.3 V pull-up and the local 1.8 V pull-up divided to about 2.55 V, above the sensor's absolute maximum.
  - TRIG (U5): J3.1 → TRIG, 2.2 kΩ pull-up (R26). R25 holds it low when J3 is open. In external-trigger mode a rising edge starts exposure.
  - LED_STROBE (U6): J2.1 → J3.2, pulled up to 3V3 by R28. R27 holds it low while the sensor is off.
- **J3**: JAE IL-G-4P-S3L2-SA (2.5 mm pitch, right-angle, friction lock); mating socket IL-G-4S-S3C2-SA. 1 TRIG_3V3, 2 LED_STROBE_3V3, 3 XSHUTDN_3V3, 4 GND, wired to the P4 board's GPIO header. The footprint `SC035HGS_Mod:JAE_IL-G-4P-S3L2-SA_1x04_P2.50mm_Horizontal` follows JAE drawing SJ019575: JAE terminal numbering (pin 1 on the left seen from the component side with the opening toward +Y), 1.1 mm holes, the club's oval pads; keep the board edge 7–8.5 mm from the pin row. Note that the club's original IL-G footprints number the pins in the opposite direction, so check club harnesses against the JAE numbering. CAM_IO0/CAM_IO1 on the EV board's CSI connector (J1.5/J1.4 here) reach no GPIO, and CAM_IO0 has only a pull-up (EV-Board schematic p.3: J9.11 CAM_IO0 has only R123 10 kΩ to 3.3 V; R111 on J9.12 CAM_IO1 is not fitted), so XSHUTDN has to be controlled through J3. J1.4 stays unconnected.
- **C12/C13/C14**: the three capacitors valued NC are DNP (not fitted) in both schematic and PCB.
- **Net renames**: `RESET` → `XSHUTDN` (1.8 V) / `XSHUTDN_3V3` (3.3 V), `XVCLK` → `MCLK`. New parts live in the project library `libraries/SC035HGS_Mod.*` (symbols and footprints), with connector metadata such as cable direction in `libraries/manifest.json`. `orcad_import` and `Camera_Sub` stay identical to upstream.
- **Firmware notes**:
  - First I2C access: wait at least 4 ms after XSHUTDN goes high with MCLK running (datasheet p.13, Fig. 1-4, T4). XSHUTDN is delayed by R10/C3, so wait about 20 ms after power-up or after releasing J3.3.
  - To restore default registers, write `0x0103=0x01` (soft reset); pulling XSHUTDN low does not clear them.
  - MCLK is 24 MHz. Wait at least 7 ms after writing `0x0100=0x01` before writing `0x4418`/`0x4419` (SmartSens configuration note).
  - External trigger mode: `0x3222[1]=1`. Strobe enable: `0x3361[7:6]=00`.
  - The I2C address depends on SID0/SID1 inside the module (not on the FPC). Scan 0x30–0x33 on first power-up.

### PCB

The PCB was updated from the new schematic in KiCad and then redone: all old copper was removed, the parts re-placed and the board re-routed.

- **Outline and stackup**: 40 × 24 mm with R3 corners. 4 layers on JLCPCB's JLC04161H-7628 stackup, 1.6 mm, ENIG. The fabrication parameters are in [`pcb-fabrication-profile.json`](pcb-fabrication-profile.json), which also sets the Board Setup manufacturing minimums.
- **Placement**: J1 (FPC to the P4 board) on the bottom edge, J2 (module FPC) on the top edge, J3 on the right edge opening outward, SW1 in the top-right corner.
- **Layers**: In1 is a solid GND plane and In2 a solid DOVDD_1V8 plane; top and bottom carry signals and GND pours. 3V3, DVDD_1V5 and AVDD_2V8 are tracks sized for the SC035HGS's 120 mW maximum, in the `PWR_*` netclasses.
- **MIPI CSI-2**: all three pairs on the top layer with no vias, 100 Ω (0.2258 mm width, 0.2032 mm gap) over the In1 reference; a single 45° bend at each turn, zero skew within each pair and 0.21 mm length difference between pairs.
- **Vias**: all 0.6/0.3 mm.
- **Silkscreen**: 1.0 mm text height and 0.15 mm stroke (JLCPCB's minimum); references that do not fit are on the Fab layer. The bottom silkscreen carries the board name and a "Same-Side FFC" note.
- **Cleanup**: the Allegro import's dimension annotations (MECH1/MECH2), drill chart, off-board empty rule areas and the stale Route Keepin area on the right edge are removed, as are Espressif's logo and the WEEE/ESD/Pb-free/halogen-free marks on the bottom silkscreen.
- **Position**: the board sits in the middle of the A4 drawing sheet (outline x 128.5–168.5 mm, y 93–117 mm), so plots show it inside the frame.
- **Checks**: ERC 0; DRC 0 errors and 0 unconnected, schematic and PCB in sync. The 87 remaining warnings are all silkscreen: 57 where neighbouring `Camera_Sub` library footprints' silkscreen touches (it extends past their courtyards) and 30 where silkscreen crosses pads (mostly J1/SW1, clipped by the solder mask at fabrication). None affects fabrication.

`sources/espressif-original/` holds Espressif's BOM, placement file and PCB fabrication notes for the original 32 mm board. **Do not order this board from them.** This board's Gerbers, BOM and placement file come from the `-fab` releases; order from the latest, `v0.9.1-fab`.

### Bring-up

Check these on the first powered boards and record the results here:

- [ ] **MCLK duty cycle**: Y1 has no datasheet on file. Measure the 24 MHz waveform at J2.21: duty cycle 45–55 %, high and low time at least 5 ns each (SC035HGS datasheet p.37).
- [ ] **Rail currents**: datasheet V0.8 leaves the operating currents on p.36 blank. Measure DOVDD_1V8, DVDD_1V5 and AVDD_2V8 separately. The 120 mW maximum on p.2 bounds a single rail to about 67 mA DOVDD, 80 mA DVDD or 43 mA AVDD, all within the ME6211 ratings; power down and investigate if a reading is clearly higher.
- [ ] **I2C address**: scan 0x30–0x33 and note the module's actual address.
- [ ] **J3 → P4 GPIOs**: when wiring J3 to the EV board's GPIO header (J1, 2×20), avoid the boot-strapping pins GPIO34–GPIO38; on the header these are GPIO37 (pin 8), GPIO38 (pin 10) and GPIO36 (pin 24) (EV-Board schematic p.3). This board holds J3 at fixed levels while the P4 is in reset (J3.3 pulled up to 3V3 by R24, J3.1 pulled down by R25, J3.2 sunk by U6 whenever DOVDD is up and STROBE idles), which could change the boot mode on a strapping pin. Drive the J3.1 (TRIG) and J3.3 (XSHUTDN) GPIOs push-pull, since the U4/U5 inputs need fast edges; configure J3.2 (STROBE) as an input and tie J3.4 to a header GND. Record the chosen GPIOs here.
- [ ] **J3 fit (optional)**: the footprint is checked against JAE drawing SJ019575. If an IL-G-4P-S3L2-SA sample is available, also confirm the hole positions and edge distance with the real part.

**Accepted risks** (decided 2026-10-03; not changed in this revision):

- **XSHUTDN on sudden power loss**: the datasheet (Fig. 1-5) wants XSHUTDN low before the supplies go off. If 3V3 drops out suddenly, C3 (R10 × C3 = 10 ms) keeps XSHUTDN high briefly after DOVDD has fallen, possibly above the DOVDD + 0.3 V absolute maximum. The energy is small and Espressif's RESET RC behaves the same way. Pulling J3.3 low before removing power avoids it in a normal shutdown. To change it: a diode across R10 (anode on /XSHUTDN, cathode on DOVDD_1V8).
- **U4 discharges C3 directly**: when U4 pulls XSHUTDN low it discharges C3 (1 µF) with no series resistor. The peak sink current is limited only by U4's output resistance; it exceeds the recommended 4 mA IOL and may approach the 50 mA absolute maximum for tens of microseconds. To change it: 100–220 Ω between U4.4 and the R10/C3/J2.3 node.

**Next revision**: U4's Description field says "R10 + C3 set the >=4 ms power-up delay", which is wrong. The datasheet's 4 ms (T4, p.13) runs from XSHUTDN high to the first I2C access and is a firmware wait (see the firmware notes above); R10/C3 make XSHUTDN rise after AVDD (T3 ≥ 0). Change it to "R10 + C3 delay XSHUTDN high ~12 ms after DOVDD (T3 >= 0); firmware waits >= 4 ms (T4) before I2C". The field is text only and affects neither the netlist nor the PCB, so this revision keeps it.

### Remotes

| Remote     | Repository                                     | Role                          |
|------------|------------------------------------------------|-------------------------------|
| `origin`   | `OHNope/sc035hgs-esp32-p4x-camera-adapter-mod` | This repository, mod work     |
| `upstream` | `OHNope/ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD`  | Base project, fetch only      |

Merge base-project updates with:

```bash
git fetch upstream
git merge upstream/main
```

Keep the project file names `SCH_ESP32-P4_FUNCTION_EV_BOARD_MIPI_Camera_Sub_V1.1_20240529.*` identical to the base project. Renaming them stops upstream changes from merging cleanly. This repository's README.md describes the mod; if a merge conflicts on it, keep this repository's version (`git checkout --ours README.md`).

### Release

[`.github/workflows/pcb-release.yml`](.github/workflows/pcb-release.yml) calls the shared pipeline in [OHNope/kicad-release](https://github.com/OHNope/kicad-release), which documents the tags, what each release contains and the Google Drive setup.

| Tag | Contents |
|-----|----------|
| `v0.9.0` | SC035HGS schematic only; the PCB is still the old board |
| `v0.9.1` / `v0.9.1-fab` | First PCB, for the prototype order. The placement file and iBOM leave out the DNP parts C12–C14 |

```bash
git tag -a v0.9.1 -m "v0.9.1: ..." && git push origin v0.9.1
```

```bash
git tag -a v0.9.1-fab -m "v0.9.1-fab: ..." "v0.9.1^{}" && git push origin v0.9.1-fab
```

`v0.9.1` publishes the schematic/PCB documents and uploads the committed project tree to Drive. `v0.9.1-fab` publishes the Gerbers and must point at the same commit as `v0.9.1`, so push the design tag before the `-fab` tag. C12–C14 still have F.Paste apertures; for SMT assembly, the BOM and placement file decide what gets fitted. For Google Drive, set the secret `GDRIVE_TOKEN` and the variable `GDRIVE_FOLDER_ID` in Settings → Secrets and variables → Actions.

### kicad-auto

`.kicad-auto/` holds kicad-auto's machine-local state (reviews, semantic analysis, generation workspaces). It is in `.gitignore` and is never committed.

### Copyright and license

The design is copyright Espressif Systems (Shanghai) Co., Ltd. This repository **grants no license**; see [LICENSE](LICENSE). It is a non-commercial share for study and reference, is not affiliated with or endorsed by Espressif, and will be taken down if Espressif objects.

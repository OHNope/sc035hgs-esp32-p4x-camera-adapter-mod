# SC035HGS Camera Adapter Mod

[中文](#中文) | [English](#english)

---

## 中文

本仓库在 [ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD](https://github.com/OHNope/ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD)（Espressif 官方相机转接板的 KiCad 转换版）基础上，修改为 SC035HGS 相机模组用的转接板。基础工程的说明见 [README.md](README.md)。

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

工程文件名 `SCH_ESP32-P4_FUNCTION_EV_BOARD_MIPI_Camera_Sub_V1.1_20240529.*` 与基础工程保持一致，不要改名，否则上游修改无法直接合并。

### 发布

推送 `v*` 标签会运行 [`.github/workflows/pcb-release.yml`](.github/workflows/pcb-release.yml)：

```bash
git tag v0.9.0 && git push origin v0.9.0
```

KiBot（[`.kibot.yaml`](.kibot.yaml)）先跑 ERC/DRC，有错误则发布失败；然后生成 `output/`：Gerber、钻孔、给板厂的 zip、BOM、贴片坐标、交互式 BOM、原理图/PCB PDF、STEP 模型，以及与上一个标签的原理图/PCB 差异图。打包后附在 GitHub Release 和 workflow run 上；配置了 Google Drive 时同时上传。

不打标签测试：**Actions → PCB release → Run workflow**，这种运行不创建 GitHub Release。

Google Drive 上传（可选）在 Settings → Secrets and variables → Actions 中配置：secret `GDRIVE_SA_JSON`（服务账号 JSON），variable `GDRIVE_TEAM_DRIVE_ID`（共享盘 ID，未设置则跳过上传），variable `GDRIVE_FOLDER_ID`（目标文件夹，空为共享盘根目录）。

### kicad-auto

`.kicad-auto/` 是 kicad-auto 工具的本地状态（评审、语义分析、生成工作区），已加入 `.gitignore`，不提交。

---

## English

This repository modifies [ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD](https://github.com/OHNope/ESP32-P4X_MIPI_Camera_Sub_V1.1_KiCAD), the KiCad conversion of Espressif's camera adapter board, into an adapter for the SC035HGS camera module. See [README.md](README.md) for the base project.

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

Keep the project file names `SCH_ESP32-P4_FUNCTION_EV_BOARD_MIPI_Camera_Sub_V1.1_20240529.*` identical to the base project. Renaming them stops upstream changes from merging cleanly.

### Release

Push a `v*` tag to run [`.github/workflows/pcb-release.yml`](.github/workflows/pcb-release.yml):

```bash
git tag v0.9.0 && git push origin v0.9.0
```

KiBot ([`.kibot.yaml`](.kibot.yaml)) runs ERC/DRC and fails the release on errors. It then writes `output/`, which holds the Gerbers and drill files plus a fab zip, BOM, pick-and-place, interactive BOM, schematic/PCB PDFs, a STEP model and schematic/PCB diffs against the previous tag. The zipped `output/` is attached to the GitHub Release and the workflow run, and is uploaded to Google Drive when configured.

To test without tagging, use **Actions → PCB release → Run workflow**. That run skips the GitHub Release.

Optional Google Drive upload (Settings → Secrets and variables → Actions): secret `GDRIVE_SA_JSON` (service-account JSON key), variable `GDRIVE_TEAM_DRIVE_ID` (shared drive ID; upload is skipped if unset), variable `GDRIVE_FOLDER_ID` (target folder; empty = shared drive root).

### kicad-auto

`.kicad-auto/` holds kicad-auto's machine-local state (reviews, semantic analysis, generation workspaces). It is in `.gitignore` and is never committed.

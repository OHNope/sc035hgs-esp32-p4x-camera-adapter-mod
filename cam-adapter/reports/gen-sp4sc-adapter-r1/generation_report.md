# 原理图生成技术报告

- Generation: `gen-sp4sc-adapter-r1`
- 状态: **candidate_ready**
- 生成时间: `2026-11-20T11:58:59.156008+00:00`
- Schematic snapshot: `sha256:7be6eb8b361a28b6595ef6afe2323f081c47a0b089804e169327ccb53e07b8ef`
- Topology fingerprint: `sha256:0f09444372df5ed2c6b9c5be0dbd669d10a51c6690e96180074e7746c56436f5`

## 1. 结果摘要

本报告由已持久化的 generation artifact 和来源证据自动编译。

- 设计决策: 15
- 来源: 18
- 证据条目: 0
- 待人工确认: 7

## 2. 设计决策与实现依据

### `relation.decouple.c1`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/0`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c2`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/1`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c7`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/2`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c3`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/3`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c4`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/4`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c8`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/5`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c5`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/6`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c6`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/7`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c9`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/8`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c17`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/9`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c18`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/10`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c10`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/11`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c11`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/12`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c12`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/13`
- 状态/置信度: `recorded` / `high`

### `relation.decouple.c13`

decoupling_service decision

- 来源: 无直接来源引用
- 证据引用: `connectivity_contract#/decoupling_capacitors/14`
- 状态/置信度: `recorded` / `high`

## 3. 来源与缓存溯源

| 来源 | 类型 | URL | PDF 页码 | 缓存状态 |
|---|---|---|---:|---|
| `source.private.04d9cdf8b6287cbd525a6b1d4f2f02e5373a9660c83cd843ca0757ee30753384` User decision: MCLK level translation | user_confirmation | - | - | `project_private / hit` |
| `source.private.1103e2dbb90207868c7d44a633f2034dd227bc5ad87df53ae41b2812788d978e` User decision: adapter retention | user_confirmation | - | - | `project_private / hit` |
| `source.private.285645a8a14ad212f41cd24ebc3153c6651787c996d5873f1197b706341cf7b7` User decision: Stamp-P4 USB-C edge | user_confirmation | - | - | `project_private / hit` |
| `source.private.86c6d3aa577225334dc313505aaa85e64671d31bf59ec8e9943e7bdc818a9096` User decision: TRIG and STROBE in flight | user_confirmation | - | - | `project_private / hit` |
| `source.private.a6dfd52ce3d206c5a4c1717eadca9e39d5096eabd855785964fee43fb8d814bc` User decision: MIPI routing rules | user_confirmation | - | - | `project_private / hit` |
| `source.private.b266777c32f7a3fd6977e3a24fb91fb10765e182801a69fe07ee1351af089ae6` User decision: adapter scope | user_confirmation | - | - | `project_private / hit` |
| `source.private.c935b94e47bf4635f94190980b148ead1e81158d906567ad7cd936639f2cbb9c` User decision: camera 3.3 V source | user_confirmation | - | - | `project_private / hit` |
| `source.datasheet.4b14a5427cc7` source.datasheet.4b14a5427cc7 | datasheet | - | - | `project_materialized_from_global / hit` |
| `source.datasheet.52fc8b26988c` source.datasheet.52fc8b26988c | datasheet | - | - | `project_materialized_from_global / hit` |
| `source.datasheet.5c68b82a5110` source.datasheet.5c68b82a5110 | datasheet | - | - | `project_materialized_from_global / hit` |
| `source.datasheet.6a9542752ceb` source.datasheet.6a9542752ceb | datasheet | - | - | `project_materialized_from_global / hit` |
| `source.datasheet.c03db6567d8d` source.datasheet.c03db6567d8d | datasheet | - | - | `project_materialized_from_global / hit` |
| `source.datasheet.d7c9bad425b6` source.datasheet.d7c9bad425b6 | datasheet | - | - | `project_materialized_from_global / hit` |
| `source.datasheet.dcb9040dd589` source.datasheet.dcb9040dd589 | datasheet | - | - | `project_materialized_from_global / hit` |
| `source.mechanical_drawing.662380c754fc` source.mechanical_drawing.662380c754fc | mechanical_drawing | - | - | `project_materialized_from_global / hit` |
| `source.mechanical_drawing.bbef069ca396` source.mechanical_drawing.bbef069ca396 | mechanical_drawing | - | - | `project_materialized_from_global / hit` |
| `source.reference_design.26331789e137` source.reference_design.26331789e137 | reference_design | - | - | `project_materialized_from_global / hit` |
| `source.schematic.896d1560b734` source.schematic.896d1560b734 | schematic | - | - | `project_materialized_from_global / hit` |

## 4. Datasheet 页面截图

没有可用的页码截图；请检查 source evidence 是否包含 `pdf_page`/`page` 定位，以及全局 PDF cache 是否可用。
## 5. 验证与流程收据

```json
{
  "counts": {
    "erc_violations": 0,
    "presentation_blocking": 0,
    "static_blocking": 0
  },
  "generation_checks": {
    "erc_ok": true,
    "evidence_closure_ok": true,
    "footprint_contracts_ok": true,
    "grid_integrity_ok": true,
    "gui_state_not_conflicting": true,
    "library_resolution_ok": true,
    "native_format_ok": true,
    "netlist_ok": true,
    "parse_ok": true,
    "presentation_ok": true,
    "reopen_ok": true,
    "required_pin_closure_ok": true,
    "save_ok": true,
    "semantic_handoff_ready": true,
    "static_integrity_ok": true
  },
  "receipt": {
    "path": "verification_receipt.json",
    "sha256": "sha256:abc8d29f1ac90659f1c8c198db7122b21d59720f5a3f6a54c9701a22428a172a"
  },
  "status": "passed"
}
```

## 6. 待处理与证据缺口

- source.datasheet.4b14a5427cc7: datasheet evidence has no PDF page locator
- source.datasheet.52fc8b26988c: datasheet evidence has no PDF page locator
- source.datasheet.5c68b82a5110: datasheet evidence has no PDF page locator
- source.datasheet.6a9542752ceb: datasheet evidence has no PDF page locator
- source.datasheet.c03db6567d8d: datasheet evidence has no PDF page locator
- source.datasheet.d7c9bad425b6: datasheet evidence has no PDF page locator
- source.datasheet.dcb9040dd589: datasheet evidence has no PDF page locator

## 7. Artifact 索引

| 类型 | 路径 | SHA-256 |
|---|---|---|
| `project_spec` | `project_spec.json` | `sha256:fb1733112da88eaab81c47cad17f0cb4b63d5ad22ac23eafeda0d383caa39875` |
| `topology_spec` | `topology_spec.json` | `sha256:048150f45bd3546acf885b493613eb87cc5aa8b296cbaf19ce7b8b20476e46d9` |
| `semantic_manifest` | `semantic_manifest.json` | `sha256:61652907c46efb74511c4855a717c9a234ba93a7ad90012b9133e64a7073660f` |
| `constraint_set` | `constraint_set.json` | `sha256:72e98546ee640547d6f51de6f8ccb3bef9eb04d66ace26d6c850d4984270f430` |
| `source_refs` | `source_refs.json` | `sha256:d12de39f80da29775cc81c29dfe634090540d85c801297fcdaae7794d7c1697e` |
| `footprint_assignment` | `footprint_assignment.json` | `sha256:f9f0a6820dfc4cb981e790805812212386930d3965f197d1696f1384608a29a1` |
| `parameter_results` | `parameter_results.json` | `sha256:dca2da80abc71b35a3e1d6201c78502884ffd692915c00fb84f4cfc00505628c` |
| `confidence_annotations` | `confidence_annotations.json` | `sha256:19325f17de04a7f97e85ac51b8d142988d2b6d0441bcc422717b1421ef29752c` |
| `generation_errors` | `generation_errors.json` | `sha256:5c39aa42bae98f03c7e70c4fd64d0938937ba46d8afd1ae139722e15485bda81` |
| `object_refs` | `object_refs.json` | `sha256:cc10edc088c5760e3df5c1937321220e2cfd1e63ab913640f85a086f4977e8d1` |
| `placement_analysis_evidence` | `placement_analysis_evidence.json` | `sha256:5e7a62e17f428b0485cb6a1a4b860b60fb997b608d323b6b6d94c7ee7856a83f` |
| `verification_receipt` | `verification_receipt.json` | `sha256:abc8d29f1ac90659f1c8c198db7122b21d59720f5a3f6a54c9701a22428a172a` |
| `final_object_binding_receipt` | `final_object_binding_receipt.json` | `sha256:76767f725fedba08acd5abf65b9f93d306963c3cbc8593aacf62903ee156b1e0` |

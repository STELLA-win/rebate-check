# Q4 代理商返点跨区域包名核对

基于跨区域不能返点政策，核对指定代理商（CA ID）在2025年Q4投放的所有有收入的效果广告包名，与区域归属表比对，找出不属于CIS地区的包名。

## 输入参数

$ARGUMENTS = CA ID（核代账户ID），例如：4812

## 执行步骤

### 第一步：切换到新加坡集群

使用 `switch_region` 切换到 singapore 工作空间（广告数据存储在新加坡集群 paimon_alsgprc_hadoop）。

### 第二步：查询效果广告投放数据

使用以下 SQL 模板查询指定 CA 的 Q4 效果广告数据（通过 Spark 引擎执行）：

```sql
SELECT core_id, package_name, SUM(dsp_fee) / 100000 AS dsp_fee
FROM paimon_alsgprc_hadoop.miuiads.dws_ad_event_after_link_core_data_v2_hi
WHERE core_id IN ($ARGUMENTS)
AND dsp_level1 IN ('效果')
AND date >= 20251001
AND date < 20260101
GROUP BY core_id, package_name
ORDER BY dsp_fee DESC
LIMIT 5000
```

注意：
- 使用 `engine: spark` 执行
- `LIMIT` 语法不支持 `LIMIT 0, 5000`，只能写 `LIMIT 5000`
- 如果同步查询超时，使用 `submit_query` 异步提交

### 第三步：筛选有收入的包名

从查询结果中筛选 `dsp_fee > 0` 的包名（排除空包名），这些是实际有收入的效果广告包名。

### 第四步：与归属表比对

读取归属对照表 Excel 文件：
- 路径：`C:\Users\dailanxin\Desktop\Rebate for Q4\【12.1-12.22】avow（368）、flymobi（8342）新增包名区域归属判定.xlsx`
- Sheet：`【For 计提】包名对应区域`
- 字段：`客户下包名` -> `包名所属区域`
- 8个区域：APAC, CIS, LATAM, MENA, NA, SA, SEA, WEU/CEE

将有收入的包名逐一与归属表比对，标记出：
1. 属于 CIS 的包名（正常）
2. 不属于 CIS 的包名（异常，需剔除）
3. 未找到映射的包名（需进一步模糊查询）

### 第五步：对未找到映射的包名进行模糊查询

对归属表中未找到精确匹配的包名，执行以下模糊匹配策略：

1. **同开发商匹配**：提取包名前缀（如 `com.platfomni`），在归属表中查找同开发商的其他包名及其区域
2. **关键词匹配**：将包名按 `.` 拆分，用每个长度 > 3 的关键词在归属表中搜索
3. **词根匹配**：对包名中有意义的词根片段进行部分匹配

根据匹配结果推测未映射包名的区域归属，给出判断依据。

### 第六步：输出最终结论

以表格形式输出：

1. **完整比对结果**：所有有收入包名及其归属区域
2. **不属于CIS的包名清单**：包名、归属区域、收入金额
3. **模糊匹配推测结果**：未映射包名的推测区域及依据

## 关键配置

- 数据工场集群：新加坡 (alsgprc)，工作空间 16436
- 底层数据表：`paimon_alsgprc_hadoop.miuiads.dws_ad_event_after_link_core_data_v2_hi`
- 归属对照表：Excel 文件（路径见第四步）
- 查询引擎：Spark

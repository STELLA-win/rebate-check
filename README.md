# 自定义 Skills 说明

## /rebate-check

**用途：** Q4 代理商返点跨区域包名核对

**背景：** 基于2025年Q4跨区域不能返点政策，核对指定代理商投放的效果广告包名是否属于CIS地区。

**调用方式：**
```
/rebate-check <CA_ID>
```

**示例：**
```
/rebate-check 4812
```

**执行流程：**
1. 切换到新加坡集群（paimon_alsgprc_hadoop）
2. 查询该CA在Q4（2025.10-12）所有有收入的效果广告包名
3. 与归属对照表精确比对，标记非CIS包名
4. 对未找到映射的包名进行模糊查询（同开发商/关键词/词根匹配），推测区域归属
5. 输出最终结论表格

**依赖：**
- 数据工场 MCP 连接（需配通新加坡 token）
- 归属对照表：`Desktop\Rebate for Q4\【12.1-12.22】avow（368）、flymobi（8342）新增包名区域归属判定.xlsx`

**已核对记录：**

| CA ID | 非CIS包数 | 详情 |
|-------|----------|------|
| 4812 | 2 | org.telegram.messenger(MENA), com.paysend.app(WEU/CEE) |

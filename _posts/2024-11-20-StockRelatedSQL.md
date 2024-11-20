---
layout: post
title: "Stock related SQL exercises"
date: 2024-11-20
---



好的，我来设计几道进出库相关的SQL练习题，涉及库存管理系统的常见场景：

### 表结构设计
````sql
-- 商品表
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100),
    category VARCHAR(50),
    unit_price DECIMAL(10,2),
    stock_quantity INT
);

-- 入库记录表
CREATE TABLE stock_in (
    in_id INT PRIMARY KEY,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    supplier_id INT,
    in_time DATETIME,
    operator VARCHAR(50)
);

-- 出库记录表
CREATE TABLE stock_out (
    out_id INT PRIMARY KEY,
    product_id INT,
    quantity INT,
    unit_price DECIMAL(10,2),
    customer_id INT,
    out_time DATETIME,
    operator VARCHAR(50)
);

-- 供应商表
CREATE TABLE suppliers (
    supplier_id INT PRIMARY KEY,
    supplier_name VARCHAR(100),
    contact VARCHAR(50),
    phone VARCHAR(20)
);
````

### 练习题

**1. 基础查询题：统计某个时间段内的入库总量和总金额**
````sql
-- 问题：统计2024年1月的所有入库记录的总数量和总金额
SELECT 
    COUNT(*) as total_records,
    SUM(quantity) as total_quantity,
    SUM(quantity * unit_price) as total_amount
FROM stock_in
WHERE in_time BETWEEN '2024-01-01' AND '2024-01-31';
````

**2. 多表关联题：查询最近一周入库量前五的商品详情**
````sql
-- 问题：查询最近7天入库数量最多的5种商品及其详细信息
SELECT 
    p.product_id,
    p.product_name,
    p.category,
    SUM(si.quantity) as total_in_quantity
FROM products p
JOIN stock_in si ON p.product_id = si.product_id
WHERE si.in_time >= DATE_SUB(CURDATE(), INTERVAL 7 DAY)
GROUP BY p.product_id, p.product_name, p.category
ORDER BY total_in_quantity DESC
LIMIT 5;
````

**3. 库存变动题：计算某商品的库存变动情况**
````sql
-- 问题：计算指定商品（product_id = 1）在2024年每个月的库存变动情况
SELECT 
    DATE_FORMAT(t.date, '%Y-%m') as month,
    SUM(CASE WHEN t.type = 'in' THEN t.quantity ELSE -t.quantity END) as stock_change
FROM (
    SELECT product_id, quantity, in_time as date, 'in' as type
    FROM stock_in
    WHERE product_id = 1
    UNION ALL
    SELECT product_id, quantity, out_time as date, 'out' as type
    FROM stock_out
    WHERE product_id = 1
) t
WHERE YEAR(t.date) = 2024
GROUP BY DATE_FORMAT(t.date, '%Y-%m')
ORDER BY month;
````

**4. 复杂统计题：分析供应商供货情况**
````sql
-- 问题：统计每个供应商的供货品类数量、总金额和平均单价
SELECT 
    s.supplier_id,
    s.supplier_name,
    COUNT(DISTINCT p.category) as category_count,
    SUM(si.quantity * si.unit_price) as total_amount,
    AVG(si.unit_price) as avg_unit_price
FROM suppliers s
JOIN stock_in si ON s.supplier_id = si.supplier_id
JOIN products p ON si.product_id = p.product_id
GROUP BY s.supplier_id, s.supplier_name
ORDER BY total_amount DESC;
````

**5. 库存预警题：查询低库存商品**
````sql
-- 问题：查询库存量低于安全库存（10件）的商品，并计算最近一个月的出库频率
SELECT 
    p.product_id,
    p.product_name,
    p.stock_quantity,
    COUNT(so.out_id) as monthly_out_frequency
FROM products p
LEFT JOIN stock_out so ON p.product_id = so.product_id 
    AND so.out_time >= DATE_SUB(CURDATE(), INTERVAL 1 MONTH)
WHERE p.stock_quantity < 10
GROUP BY p.product_id, p.product_name, p.stock_quantity
ORDER BY monthly_out_frequency DESC;
````

**6. 综合分析题：库存周转率分析**
````sql
-- 问题：计算每种商品在2024年第一季度的库存周转率
WITH quarterly_stats AS (
    SELECT 
        p.product_id,
        p.product_name,
        SUM(so.quantity) as total_out_quantity,
        AVG(p.stock_quantity) as avg_stock
    FROM products p
    LEFT JOIN stock_out so ON p.product_id = so.product_id
        AND so.out_time BETWEEN '2024-01-01' AND '2024-03-31'
    GROUP BY p.product_id, p.product_name
)
SELECT 
    product_id,
    product_name,
    total_out_quantity,
    avg_stock,
    CASE 
        WHEN avg_stock > 0 THEN (total_out_quantity / avg_stock) * 4
        ELSE 0 
    END as turnover_rate
FROM quarterly_stats
ORDER BY turnover_rate DESC;
````

### 进阶练习题

**7. 批次管理题：先进先出分析**
````sql
-- 问题：对指定商品按照批次进行库存分析
WITH batch_inventory AS (
    SELECT 
        product_id,
        in_time as batch_time,
        quantity as original_quantity,
        quantity - COALESCE(used_quantity, 0) as remaining_quantity
    FROM stock_in si
    LEFT JOIN (
        SELECT product_id, SUM(quantity) as used_quantity
        FROM stock_out
        GROUP BY product_id
    ) so ON si.product_id = so.product_id
    WHERE si.product_id = 1
)
SELECT * FROM batch_inventory
WHERE remaining_quantity > 0
ORDER BY batch_time;
````

**8. 库存成本分析题**
````sql
-- 问题：计算加权平均库存成本
SELECT 
    p.product_id,
    p.product_name,
    SUM(si.quantity * si.unit_price) / SUM(si.quantity) as avg_cost
FROM products p
JOIN stock_in si ON p.product_id = si.product_id
GROUP BY p.product_id, p.product_name;
````

### 注意事项：
1. 在实际应用中需要注意添加适当的索引
2. 考虑大数据量下的查询性能优化
3. 注意日期时间类型的处理
4. 考虑并发情况下的数据一致性

这些练习题涵盖了进出库管理中的常见场景，包括：
- 基础的进出库统计
- 库存变动分析
- 供应商分析
- 库存预警
- 周转率计算
- 批次管理
- 成本核算

你可以根据实际需求调整这些SQL语句，或者组合它们来解决更复杂的业务问题。
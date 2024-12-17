---
layout: post
title:  "Invitation App"
date:   2024-12-17 12:30:01 +0800
categories: Apps
---


# 大连市吉林大学校友会2025年迎新年会邀请函需求文档

## 1. 项目概述
本项目旨在为大连市吉林大学校友会2025年迎新年会创建一个在线报名系统。该系统将允许校友通过网络报名参加活动，并管理报名名单及报名费用的提取。

## 2. 目标
- 提供一个用户友好的在线报名平台。
- 允许校友在线支付报名费（100元），并支持退款。
- 校友会能够管理报名名单，查看报名信息和提取报名费。

## 3. 功能需求

### 3.1 用户功能
- **在线报名**
  - 用户可以通过填写报名表单进行报名。
  - 表单字段包括：姓名、性别、毕业年份、联系方式、是否需要住宿等。
  
- **支付功能**
  - 用户在提交报名表单后，能够通过支付宝或微信支付报名费（100元）。
  - 支持报名费的退款申请。

- **报名状态查询**
  - 用户可以查询自己的报名状态，包括是否已支付、是否成功报名等。

### 3.2 管理员功能
- **报名名单管理**
  - 管理员可以查看所有报名用户的信息，包括姓名、联系方式、支付状态等。
  - 支持导出报名名单为Excel或CSV格式。

- **费用管理**
  - 管理员可以查看报名费的支付情况，包括已支付和未支付的用户。
  - 支持手动标记退款申请，并记录退款状态。

- **通知功能**
  - 管理员可以向所有报名用户发送通知邮件或短信，告知活动相关信息。

## 4. 技术需求
- **前端技术**
  - 使用HTML、CSS和JavaScript构建用户界面。
  - 响应式设计，确保在手机和电脑上均可良好显示。

- **后端技术**
  - 使用Node.js或Python Flask作为后端框架。
  - 数据库使用MySQL或MongoDB存储用户报名信息和支付记录。

- **支付接口**
  - 集成支付宝和微信支付的API，处理支付和退款请求。

## 5. 安全性需求
- 确保用户数据的安全性，使用HTTPS协议加密传输。
- 对用户输入进行验证，防止SQL注入和XSS攻击。

## 6. 时间计划
- **需求分析与设计**：2024年1月 - 2024年2月
- **开发阶段**：2024年3月 - 2024年5月
- **测试阶段**：2024年6月
- **上线与推广**：2024年7月

## 7. 预算
- 开发费用：预计XX万元
- 服务器费用：预计XX万元/年
- 其他费用（如域名、SSL证书等）：预计XX万元

## 8. 结论
本需求文档为大连市吉林大学校友会2025年迎新年会的在线报名系统提供了详细的功能和技术需求。通过该系统，校友会将能够高效地管理活动报名和费用，提高用户体验。



要将整个项目代码整合到一个 Markdown 文档中，包含了项目的各个部分代码。

```markdown
# 大连市吉林大学校友会2025年迎新年会在线报名系统

## 项目概述
本项目旨在为大连市吉林大学校友会2025年迎新年会创建一个在线报名系统。

## 目录
- [前端代码](#前端代码)
- [后端代码](#后端代码)
- [数据库设计](#数据库设计)
- [单元测试](#单元测试)
- [部署说明](#部署说明)

## 前端代码

### `app/index.html`
```html
<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>校友会迎新年会报名</title>
    <link rel="stylesheet" href="styles.css">
</head>
<body>
    <h1>大连市吉林大学校友会2025年迎新年会报名</h1>
    <form id="registrationForm">
        <label for="name">姓名:</label>
        <input type="text" id="name" name="name" required>
        
        <label for="gender">性别:</label>
        <select id="gender" name="gender" required>
            <option value="male">男</option>
            <option value="female">女</option>
        </select>
        
        <label for="graduationYear">毕业年份:</label>
        <input type="number" id="graduationYear" name="graduationYear" required>
        
        <label for="contact">联系方式:</label>
        <input type="tel" id="contact" name="contact" required>
        
        <label for="accommodation">是否需要住宿:</label>
        <select id="accommodation" name="accommodation">
            <option value="yes">是</option>
            <option value="no">否</option>
        </select>
        
        <button type="submit">提交报名</button>
    </form>
    <script src="script.js"></script>
</body>
</html>
```

### `app/script.js`
```javascript
document.getElementById('registrationForm').addEventListener('submit', async (event) => {
    event.preventDefault();
    
    const formData = new FormData(event.target);
    const data = Object.fromEntries(formData.entries());

    // 调用后端支付接口
    const response = await fetch('/api/pay', {
        method: 'POST',
        headers: {
            'Content-Type': 'application/json',
        },
        body: JSON.stringify(data),
    });

    const result = await response.json();
    if (result.success) {
        // 跳转到支付页面或显示二维码
        window.location.href = result.paymentUrl; // 假设返回支付链接
    } else {
        alert('支付失败: ' + result.message);
    }
});
```

## 后端代码

### `app/server.js`
```javascript
const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios'); // 用于发送HTTP请求
const app = express();
const port = 3000;

// Middleware
app.use(bodyParser.json());
app.use(express.static('public'));

// 用户报名接口
app.post('/api/register', (req, res) => {
    const { name, gender, graduationYear, contact, accommodation } = req.body;
    // TODO: 保存报名信息到数据库
    res.status(201).send({ message: '报名成功' });
});

// 支付接口
app.post('/api/pay', async (req, res) => {
    const { name, gender, graduationYear, contact, accommodation } = req.body;

    // TODO: 保存报名信息到数据库

    // 调用支付接口（示例为支付宝）
    try {
        const paymentResponse = await axios.post('https://api.alipay.com/gateway/pay', {
            out_trade_no: 'unique_order_id', // 生成唯一订单号
            total_amount: 100, // 支付金额
            subject: '校友会报名费',
        });

        res.send({ success: true, paymentUrl: paymentResponse.data.paymentUrl });
    } catch (error) {
        res.send({ success: false, message: '支付接口调用失败' });
    }
});

// 提款接口
app.post('/api/withdraw', async (req, res) => {
    const { amount } = req.body; // 提款金额

    // TODO: 验证管理员身份

    // 调用支付接口进行提款（示例为支付宝）
    try {
        const withdrawResponse = await axios.post('https://api.alipay.com/gateway/withdraw', {
            out_biz_no: 'unique_withdraw_id', // 生成唯一提款订单号
            amount: amount, // 提款金额
        });

        res.send({ success: true, message: '提款成功' });
    } catch (error) {
        res.send({ success: false, message: '提款接口调用失败' });
    }
});

// 启动服务器
app.listen(port, () => {
    console.log(`服务器正在运行在 http://localhost:${port}`);
});
```

## 数据库设计

### `app/database.sql`
```sql
CREATE TABLE payments (
    id INT AUTO_INCREMENT PRIMARY KEY,
    user_id INT,
    amount DECIMAL(10, 2),
    status ENUM('pending', 'completed', 'failed'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE withdrawals (
    id INT AUTO_INCREMENT PRIMARY KEY,
    amount DECIMAL(10, 2),
    status ENUM('pending', 'completed', 'failed'),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

## 单元测试

### `__tests__/server.test.js`
```javascript
const request = require('supertest');
const express = require('express');
const bodyParser = require('body-parser');
const axios = require('axios');
const app = express();
const port = 3000;

// Middleware
app.use(bodyParser.json());

// Mock the payment and withdraw functions
jest.mock('axios');

// 用户报名接口
app.post('/api/register', (req, res) => {
    const { name, gender, graduationYear, contact, accommodation } = req.body;
    res.status(201).send({ message: '报名成功' });
});

// 支付接口
app.post('/api/pay', async (req, res) => {
    const { name, gender, graduationYear, contact, accommodation } = req.body;

    axios.post.mockResolvedValue({
        data: { paymentUrl: 'http://payment.url' }
    });

    res.send({ success: true, paymentUrl: 'http://payment.url' });
});

// 提款接口
app.post('/api/withdraw', async (req, res) => {
    const { amount } = req.body;

    axios.post.mockResolvedValue({
        data: { success: true }
    });

    res.send({ success: true, message: '提款成功' });
});

// 启动服务器
const server = app.listen(port);

// 测试用例
describe('API Tests', () => {
    afterAll(() => {
        server.close();
    });

    test('用户报名接口', async () => {
        const response = await request(server)
            .post('/api/register')
            .send({
                name: '张三',
                gender: 'male',
                graduationYear: 2020,
                contact: '123456789',
                accommodation: 'yes'
            });

        expect(response.status).toBe(201);
        expect(response.body.message).toBe('报名成功');
    });

    test('支付接口', async () => {
        const response = await request(server)
            .post('/api/pay')
            .send({
                name: '张三',
                gender: 'male',
                graduationYear: 2020,
                contact: '123456789',
                accommodation: 'yes'
            });

        expect(response.body.success).toBe(true);
        expect(response.body.paymentUrl).toBe('http://payment.url');
    });

    test('提款接口', async () => {
        const response = await request(server)
            .post('/api/withdraw')
            .send({
                amount: 100
            });

        expect(response.body.success).toBe(true);
        expect(response.body.message).toBe('提款成功');
    });
});
```

## 部署说明
1. 创建阿里云ECS实例。
2. 安装Node.js和npm。
3. 上传项目代码到ECS实例。
4. 安装依赖并启动应用。
5. 配置Nginx（可选）以支持域名访问。

---

以上是整个项目的代码整合到一个 Markdown 文档中的示例。您可以根据需要进行调整和扩展。

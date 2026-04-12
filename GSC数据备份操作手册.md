# GSC数据备份操作手册

## 第一阶段：创建载体

我们需要一个 Google Sheet 来承载数据，以及一个脚本容器。

1. 打开浏览器，访问 [sheets.new](https://sheets.new) 创建一个新的 Google 表格。

2. 给表格起个名字，例如：` GSC 数据备份`。

3. 点击顶部菜单栏的 **扩展程序 (Extensions)** > **Apps Script**。

   ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E5%88%9B%E5%BB%BA%E8%A1%A8%E6%A0%BC%E5%B9%B6%E6%89%93%E5%BC%80Apps%20Script.png)

4. 这会打开一个新的代码编辑器窗口，这就是我们接下来操作的“控制台”。

   ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/Apps%20Script.png)

## 第二阶段：获取 Google Cloud 通行证

由于这是调用 Google 的底层数据，我们需要借用 Google Cloud Platform (GCP) 的通道。

**1. 创建/查找 GCP 项目**

- 在浏览器新标签页打开：[Google Cloud Console](https://console.cloud.google.com/)。

- 如果你没有项目，点击左上角选择“新建项目” (Create Project)，名字随意（例如 `GSC-Backup-Tool`）。

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/GCP-%E6%96%B0%E5%BB%BA%E9%A1%B9%E7%9B%AE.png)

- 创建完成后，停留在仪表盘首页。

**2. 复制项目编号 (Project Number)**

- 在仪表盘向下滚动找到“项目信息 (Project Info)”卡片，找到 **项目编号 (Project Number)**。

- ⚠️ **注意**：我们要全是数字的那个“编号”，**不是**带有字母的“项目 ID”。

- **复制这串数字。**

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E9%A1%B9%E7%9B%AEID.png)

**3. 开启 GSC API 服务**

- 在 GCP 控制台左侧菜单，点击 **API 和服务 (APIs & Services)** > **库 (Library)**。

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E6%89%BE%E5%88%B0GSC-API.png)

- 在搜索框输入：`Google Search Console API`。

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E6%90%9C%E7%B4%A2Google%20Search%20Console%20AP.png)

- 点击进入，并点击蓝色的 **启用 (Enable)** 按钮。

- *(如果不做这一步，脚本会报错提示 API 未开启)*

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/Google%20Search%20Console%20API%E5%90%AF%E7%94%A8.png)

## 第三阶段：连接脚本与云端

回到刚才打开的 **Apps Script 代码编辑器** 窗口。

**1. 绑定项目**

- 点击左侧侧边栏的 **⚙️ 项目设置 (Project Settings)** 图标。

- 找到 **Google Cloud Platform (GCP) 项目** 一栏。

- 点击 **更改项目 (Change project)**。

- 在弹出的框中，粘贴刚才复制的 **项目编号 (纯数字)**，点击设置。

- *(成功标志：页面会显示绑定成功的项目 ID)*

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E7%BB%91%E5%AE%9A%E9%A1%B9%E7%9B%AE%E6%8F%90%E7%A4%BA.png)

  PS.点击**设置项目**会出现图中红字提示，点击连接进行OAuth授权。

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/OAuth%E6%8E%88%E6%9D%83.png)

  1. 点击**开始**。

  2. 填写应用信息

     ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/1.png)

  3. 受众群体选择**外部**

     ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/2.png)

  4. 填写联系邮箱

     ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/3.png)

  5. 勾选同意，选择创建

  6. 点击**目标对象**，下滑找到**Add Users**，点击后填入自己的Google邮箱，点击保存

     ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E6%B7%BB%E5%8A%A0%E7%94%A8%E6%88%B72.png)

  7. 回到刚才的**设置项目**重新进行一次项目设置，可以发现成功设置项目

     ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E9%A1%B9%E7%9B%AE%E8%AE%BE%E7%BD%AE%E6%88%90%E5%8A%9F.png)

**2. 修改权限清单 (Manifest)**

回到Apps Script

- 仍在“项目设置”页面，勾选下方的 **“在编辑器中显示 'appsscript.json' 清单文件”**。

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E6%98%BE%E7%A4%BA%E6%B8%85%E5%8D%95%E6%96%87%E4%BB%B6.png)

- 点击左侧 **< > 编辑器 (Editor)** 图标回到代码界面。

- 你会发现左侧文件列表多了一个 `appsscript.json`。点击它。

- **完全清空**里面的内容，复制粘贴以下代码，并点击保存：

  ```JSON
  {
    "timeZone": "Asia/Shanghai",
    "dependencies": {
    },
    "exceptionLogging": "STACKDRIVER",
    "oauthScopes": [
      "https://www.googleapis.com/auth/script.external_request",
      "https://www.googleapis.com/auth/spreadsheets", 
      "https://www.googleapis.com/auth/webmasters.readonly"
    ],
    "runtimeVersion": "V8"
  }
  ```

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E4%BF%AE%E6%94%B9%E6%B8%85%E5%8D%95%E6%96%87%E4%BB%B6.png)

- (这一步是告诉 Google：我需要“联网”、“读写表格”和“读取GSC”这三个权限)

## 第四阶段：部署核心代码

1. 在左侧文件列表点击 `Code.gs`（如果没有就新建一个脚本文件）。

2. **清空**里面原本的 `function myFunction() {...}`。

3. 复制粘贴以下完整代码：

   ```javascript
   /**
    * GSC Data Saver - v0.1 (Internal MVP)
    * 能够绕过 Apps Script 内置服务，直接通过 REST API 获取数据
    */
   
   function getGSCDataViaRest() {
     // ================= 配置区域 (修改这里) =================
     
     // 你的 GSC 资源地址。
     // 如果是网域资源 (Domain Property)，保留前缀，如 'sc-domain:example.com'
     // 如果是网址前缀 (URL Prefix)，写完整链接，如 'https://example.com/'
     var siteUrl = 'sc-domain:你的域名.com'; 
     
     // =======================================================
   
     var sheetName = 'GSC_Backup_Rest';
     var date = new Date();
     date.setDate(date.getDate() - 3); // 获取3天前的数据（避开数据延迟）
     var dateString = Utilities.formatDate(date, Session.getScriptTimeZone(), 'yyyy-MM-dd');
     
     Logger.log('正在获取日期: ' + dateString);
   
     var apiUrl = 'https://www.googleapis.com/webmasters/v3/sites/' + encodeURIComponent(siteUrl) + '/searchAnalytics/query';
     
     // 构建请求
     var payload = {
       "startDate": dateString,
       "endDate": dateString,
       "dimensions": ["date", "query", "page", "device"], 
       "rowLimit": 25000
     };
   
     try {
       var token = ScriptApp.getOAuthToken(); // 获取 GCP 授权
       var options = {
         "method": "post",
         "contentType": "application/json",
         "headers": {"Authorization": "Bearer " + token},
         "payload": JSON.stringify(payload),
         "muteHttpExceptions": true
       };
   
       var response = UrlFetchApp.fetch(apiUrl, options);
       var json = JSON.parse(response.getContentText());
   
       if (response.getResponseCode() !== 200) {
         Logger.log('API 错误: ' + JSON.stringify(json));
         return;
       }
   
       if (json.rows && json.rows.length > 0) {
         saveToSheet(json.rows, sheetName);
         Logger.log('成功保存 ' + json.rows.length + ' 条数据');
       } else {
         Logger.log('该日期暂无数据');
       }
     } catch (e) {
       Logger.log('执行错误: ' + e.toString());
     }
   }
   
   function saveToSheet(rows, sheetName) {
     var ss = SpreadsheetApp.getActiveSpreadsheet();
     var sheet = ss.getSheetByName(sheetName);
     if (!sheet) {
       sheet = ss.insertSheet(sheetName);
       sheet.appendRow(['Date', 'Query', 'Page', 'Device', 'Clicks', 'Impressions', 'CTR', 'Position']);
       sheet.setFrozenRows(1);
     }
     var output = rows.map(function(row) {
       return [row.keys[0], row.keys[1], row.keys[2], row.keys[3], row.clicks, row.impressions, row.ctr, row.position];
     });
     sheet.getRange(sheet.getLastRow() + 1, 1, output.length, output[0].length).setValues(output);
   }
   ```

   4. **重要**：修改代码第 12 行的 `siteUrl`，填入你自己的 GSC 资源地址。

   5. 点击上方磁盘图标 💾 **保存**。

## 第五阶段：首次运行与自动化

**1. 首次授权运行**

- 在工具栏确选定函数为 `getGSCDataViaRest`。

- 点击 **“运行 (Run)”** 按钮。

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E7%82%B9%E5%87%BB%E8%BF%90%E8%A1%8C.png)

- 屏幕会弹出一个“Authorization required”窗口。

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E6%8E%88%E6%9D%83.png)

  - 点击“Review Permissions”。

  - 选择你的 Google 账号。

  - **关键点**：你可能会看到“Google hasn't verified this app”（应用未验证）的红色警告。这是正常的，因为这是你自己写的脚本。

    ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E4%B8%8D%E5%AE%89%E5%85%A8%E6%8E%88%E6%9D%83.png)

  - 点击左下角的 **Advanced (高级)**。

  - 点击最下方的 **Go to ... (unsafe) (转至... 不安全)**。

    ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E5%85%A8%E9%80%89.png)

  - **全选**，在最后一步点击 **Allow (允许)**。

**2. 验证结果**

- 回到你的 Google Sheet 页面，查看底部是否多了一个 `GSC_Backup_Rest` 的工作表，里面是否有数据。

  ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E6%95%B0%E6%8D%AE.png)

**3. 设置自动化**

- 回到 Apps Script 编辑器。

- 点击左侧菜单的 **⏰ 触发器 (Triggers)**（闹钟图标）。

- 点击右下角蓝色的 **+ 添加触发器 (Add Trigger)**。

- 配置如下：

  - 要运行的函数：`getGSCDataViaRest`

  - 事件源：**时间驱动 (Time-driven)**

  - 触发器类型：**天定时器 (Day timer)**

  - 时间：建议选择 **凌晨 1点 到 2点 (1am to 2am)**

    ![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E5%AE%9A%E6%97%B6%E5%99%A8.png)

- 点击保存。

**🎉 恭喜！** 现在，你的 Google Sheet 每天凌晨都会自动去 Google Search Console 备份3天前的数据并存入你的个人数据表。哪怕 2 年后，你依然能查到今天的每一个搜索词。

---

现在的脚本版虽然免费，但配置起来确实需要一点耐心。

我正在计划开发一个 **GSC Data Saver 浏览器插件版**：

- ✅ **零门槛**：无需配置 API，一键安装即用。
- ✅ **多号管理**：自动备份你名下所有站点的数据。
- ✅ **可视化看板**：内置 Looker Studio 模板，直接生成直观可视化的报表。

**🎁 内测福利** 如果你对这个“懒人版”感兴趣，欢迎加入等待名单。产品上线时，我会第一时间为你发送 **5 折早鸟优惠码**。

[👉 点击这里加入 Waitlist (早鸟优惠)](https://tally.so/r/PdOB7V)



也可以扫码预约

![](https://hexo-mice.oss-cn-hongkong.aliyuncs.com/img/%E4%BA%8C%E7%BB%B4%E7%A0%81.png)

---

> 👨‍💻 关于作者
>
> 本手册由Mice制作，我是一名致力于通过代码解决实际问题的独立开发者。
>
> 如果你在使用本脚本过程中遇到报错，或者有关于 SEO 数据分析的新想法，欢迎随时通过以下方式联系我：
>
> - **Twitter/X**: @Mice242697 (分享独立开发日常)
> - **Email**: maxinhao1997@gmail.com
> - **WeChat**:mice1024aa (请备注：GSC)

---

*本文档专供 **哥飞群友** 内部交流使用，请勿外传。*
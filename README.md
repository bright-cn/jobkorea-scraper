# JOBKOREA 抓取工具使用指南

本项目实现了三种抓取 [JOBKOREA](https://www.jobkorea.co.kr/) 的方法，展示了从手动抓取到由 AI 生成抓取代码的演进过程。

## 前提条件

开始之前，请确保你已具备：

- 已安装 **Python 3.9+**
- **Node.js 和 npx**（用于方法 2）
- 拥有带 API 令牌的 **[Bright Data 账户](https://www.bright.cn/?hs_signup=1&utm_source=github)**（新用户可获得免费额度）
- 基本熟悉 Python 和 JSON

## 项目设置

克隆存储库并安装依赖项：

```bash
python -m venv venv
source venv/bin/activate      # macOS / Linux
venv\Scripts\activate         # Windows

pip install -r requirements.txt
```

### 所需的 Python 库

- **[requests](https://requests.readthedocs.io/)** - 用于获取网页的 HTTP 库
- **[BeautifulSoup4](https://www.crummy.com/software/BeautifulSoup/)** - HTML/XML 解析库
- **[Pydantic](https://docs.pydantic.dev/)** - 使用 Python 类型注解进行数据验证
- **[mcp](https://github.com/modelcontextprotocol/python-sdk)** - 模型上下文协议 Python SDK

---

## 1. 手动爬虫（Requests + BeautifulSoup）

**文件：** `manual_scraper.py`

尝试使用 **[requests](https://requests.readthedocs.io/)** 库发送标准 HTTP 请求来抓取网站，并使用 **[BeautifulSoup](https://docs.brightdata.com/integrations/beautifulsoup)** 进行解析。

**结果：** **成功**（通过高级解析）

### 工作原理

- JOBKOREA 使用 Next.js 客户端渲染（CSR），这意味着职位数据隐藏在 JSON 脚本中，而不是静态 HTML 中。
- **问题：** 标准 `BeautifulSoup` 无法在渲染后的 HTML 中找到列表项。
- **解决方案：** 该脚本使用**正则表达式**直接从 Next.js 注水脚本（`self.__next_f.push`）中提取职位数据。
- **结果：** 无需完整浏览器即可成功抓取职位，不过这种方法比 MCP 方法更脆弱。

### 使用方法

```bash
python manual_scraper.py "https://www.jobkorea.co.kr/Search/?stext=python"
```

### 何时使用此方法

手动抓取适用于：
- 学习网站如何组织其数据
- 快速构建原型和进行实验
- 结构稳定且可预测的网站
- 低数据量抓取需求

**局限性：** 与 JOBKOREA 当前的页面结构紧密耦合，网站发生变化时可能失效。

---

## 2. Bright Data MCP 抓取工具

**文件：** `mcp_scraper.py`

使用 **[@brightdata/mcp](https://docs.brightdata.com/ai/mcp-server/overview)** 服务器绕过封锁并自动处理渲染。

### 工作原理

1. 通过 `npx @brightdata/mcp` 连接到本地 MCP 服务器
2. 调用 `scrape_as_markdown` 获取并渲染页面内容
3. 将原始 Markdown 保存到 `scraped_data.md`
4. 解析 Markdown，并将提取的结构化数据写入 `jobs_mcp.json`

### 设置

1. 从 [Bright Data 用户设置](https://www.bright.cn/cp/setting/users)获取你的 API 令牌
2. 在项目根目录中创建 `.env` 文件：
   ```bash
   BRIGHT_DATA_API_TOKEN=your_token_here
   ```

### 使用方法

```bash
python3 mcp_scraper.py "https://www.jobkorea.co.kr/Search/?stext=python"
```

### 何时使用此方法

基于 MCP 的爬虫适用于：
- 可用于生产环境的可靠抓取
- 自动化任务和定时任务
- 结构频繁变化的网站
- 大规模数据采集
- 集成到数据管道中

**优势：** 
- 处理渲染、网络连接和反爬虫措施
- 受布局变化的影响更小
- 基于 [Bright Data 网络解锁器](https://www.bright.cn/products/web-unlocker)技术构建
- 易于自动化和集成

---

## 3. AI 生成的抓取工具（Bright Data 网页爬虫 IDE）

**无需本地文件** - 此方法使用 Bright Data 基于云的 IDE 和 AI 代码生成功能。

### 概述

你无需从头编写抓取代码，而是可以使用 **[Bright Data 网页爬虫 IDE](https://www.bright.cn/products/web-scraper/functions)** 自动生成可用于生产环境的 JavaScript 抓取代码。AI 会分析目标网站，并为你创建功能完整的爬虫工具。

### 工作原理

1. 在 Bright Data 控制面板中打开网页爬虫 IDE
2. 输入目标 URL（JOBKOREA 搜索页面）
3. 点击“生成代码”：AI 会分析网站并创建爬虫
4. 根据需要检查并自定义生成的代码
5. 直接从 IDE 或通过 API 运行抓取工具
6. 以 JSON、CSV 或其他格式下载结果

### 分步指南

#### 第 1 步：访问网页爬虫 IDE

1. 登录 [Bright Data 控制面板](https://www.bright.cn/cp/zones)
2. 从左侧边栏依次转到**数据** → **我的爬虫工具**
3. 点击右上角的**新建**
4. 选择**开发你自己的网页爬虫**

![Bright Data 控制面板](https://hackmd.io/_uploads/H1Wb3sImZx.png)

#### 第 2 步：生成爬虫代码

1. 在 IDE 中输入目标 URL：
   ```
   https://www.jobkorea.co.kr/Search/?stext=python
   ```

2. 点击**生成代码**

3. AI 将分析页面结构并生成 JavaScript 代码，以执行以下操作：
   - 前往搜索结果
   - 提取职位列表（职位名称、公司、地点、日期、URL）
   - 处理分页
   - 构建数据结构

![爬虫 IDE](https://hackmd.io/_uploads/HyOuAsUQ-e.png)

4. 代码准备就绪后，你将收到电子邮件通知

#### 第 3 步：检查和自定义

生成的代码通常包括：

- **交互代码**：用于浏览网站、点击按钮、滚动页面等操作的 JavaScript
- **解析器代码**：提取职位数据并构建其结构的逻辑
- **错误处理**：内置重试逻辑和异常处理

你可以自定义：
- 要提取哪些字段
- 要抓取多少页
- 筛选条件和搜索参数
- 输出格式

#### 第 4 步：测试爬虫

1. 点击**预览**（播放）按钮测试你的爬虫工具
2. 在 IDE 中检查提取的数据
3. 根据需要进行调整
4. 保存你的抓取工具

#### 第 5 步：大规模运行

现在你可以：

**选项 A：从 IDE 运行**
- 点击“运行”执行爬虫
- 实时监控进度
- 完成后下载结果

**选项 B：使用 API**
```python
import requests

response = requests.post(
    'https://api.brightdata.com/datasets/v3/trigger?dataset_id=YOUR_SCRAPER_ID',
    headers={
        'Authorization': 'Bearer YOUR_API_TOKEN',
        'Content-Type': 'application/json'
    },
    json=[{
        'url': 'https://www.jobkorea.co.kr/Search/?stext=python',
        'max_pages': 5
    }]
)
```

**选项 C：安排定期运行**
- 设置自动抓取计划
- 通过 webhook、电子邮件或云存储接收数据
- 通过控制面板进行监控

### 何时使用此方法

AI 生成的抓取方式适用于：
- 快速设置：只需几分钟，而非数小时
- 无需手动开发即可获得可用于生产环境的代码
- 维护爬虫工具：网站发生变化时，AI 可以重新生成代码
- 需要可靠抓取工具的非开发人员
- 希望专注于数据分析而非抓取逻辑的团队
- 快速扩展到多个网站

### 优势

✅ **AI 驱动的代码生成** - 分析网站并编写抓取逻辑  
✅ **预构建模板** - 基于热门网站的成熟模式快速开始  
✅ **完全托管** - 无需管理基础设施  
✅ **内置解锁功能** - 自动破解 CAPTCHA 并轮换代理  
✅ **易于维护** - 网站发生变化时重新生成代码  
✅ **API 访问** - 集成到现有工作流中  
✅ **实时监控** - 跟踪爬虫性能和成本

### 主要功能

- **JavaScript 环境**：完整支持现代 JS/Node.js
- **浏览器自动化**：内置无头浏览器功能
- **实用函数**：为常见抓取任务提供预构建的辅助函数
- **调试工具**：控制台日志、网络检查器和错误跟踪
- **交付选项**：JSON、CSV、Amazon S3、webhook、电子邮件
- **计划任务**：使用类似 Cron 的计划机制自动运行

### 其他资源

- **[网页爬虫 IDE 文档](https://docs.brightdata.com/scraping-automation/web-scraper-ide/overview)** - 完整指南
- **[爬虫开发指南](https://docs.brightdata.com/scraping-automation/web-scraper-ide/develop-a-scraper)** - 分步教程
- **[视频教程](https://docs.brightdata.com/scraping-automation/web-scraper-ide/videos)** - 观看 IDE 使用方法
- **[API 集成](https://docs.brightdata.com/scraping-automation/web-scraper-ide/scraper-api)** - 通过程序触发爬虫

---

## 三种方法对比

| 方法                       | 设置工作量 | 可靠性 | 自动化 | 是否需要代码 | 维护 | 最佳使用场景                         |
|---------------------------------|--------------|-------------|------------|---------------|-------------|---------------------------------------|
| **手动 Python 抓取**      | 低          | 中低  | 有限    | 是（Python）  | 高        | 学习、快速实验           |
| **Bright Data MCP（Python）**    | 中       | 高        | 高       | 是（Python）  | 中      | 生产环境抓取、定时任务   |
| **AI 生成的爬虫（IDE）**  | 极低     | 极高   | 极高  | 否（由 AI 生成） | 极低 | 快速设置、托管式生产级爬虫 |

---

## 项目结构

```
jobkorea_scraper/
│
├── manual_scraper.py        # Technique 1: Manual scraping
├── mcp_scraper.py           # Technique 2: MCP-based scraping
├── parsers/
│   └── jobkorea.py          # Shared parsing logic
├── schemas.py               # Pydantic models for validation
├── requirements.txt         # Python dependencies
├── .env                     # API token (create this)
├── jobs.json                # Output from manual scraper
├── jobs_mcp.json            # Output from MCP scraper
├── scraped_data.md          # Raw markdown from MCP
└── README.md                # This file
```

**注意：** 方法 3 不需要本地文件，它完全在 Bright Data 的云 IDE 中运行。

---

## 输出文件

运行爬虫工具后，你将获得：

- **`jobs.json`** - 手动爬虫生成的结构化职位数据
- **`jobs_mcp.json`** - MCP 抓取工具生成的结构化职位数据
- **`scraped_data.md`** - MCP 返回的原始 Markdown（用于调试）
- **`debug.html`** - 手动爬虫返回的原始 HTML（用于调试）

你可以从网页爬虫 IDE 下载多种格式的结果：
- JSON
- CSV
- NDJSON
- 直接发送到 Amazon S3、Google Cloud Storage 或 Azure Blob Storage

---

## 其他资源

### Bright Data 产品

- **[网页爬虫 IDE](https://www.bright.cn/products/web-scraper)** - 支持 AI 代码生成的 JavaScript 爬虫工具
- **[网络解锁器](https://www.bright.cn/products/web-unlocker)** - 绕过 CAPTCHA 和反爬虫系统
- **[搜索引擎 API](https://www.bright.cn/products/serp-api)** - 大规模采集搜索引擎结果
- **[代理网络](https://www.bright.cn/proxy-types)** - 住宅代理、ISP 代理和数据中心代理
- **[抓取浏览器](https://www.bright.cn/products/scraping-browser)** - 具备解锁功能的无头浏览器

### 文档

- **[网页爬虫 IDE 概述](https://docs.brightdata.com/scraping-automation/web-scraper-ide/overview)** - 完整 IDE 指南
- **[MCP 服务器文档](https://docs.brightdata.com/ai/mcp-server/overview)** - 模型上下文协议
- **[BeautifulSoup 集成](https://docs.brightdata.com/integrations/beautifulsoup)** - 将 Bright Data 与 BeautifulSoup 结合使用
- **[Python SDK](https://docs.brightdata.com/api-reference/sdk)** - Bright Data 官方 Python SDK
- **[API 参考](https://docs.brightdata.com/api-reference/authentication)** - 完整 API 文档

### 学习资源

- **[Bright Data 控制面板](https://www.bright.cn/cp/zones)** - 管理区域并监控使用情况
- **[获取 API 令牌](https://www.bright.cn/cp/setting/users)** - 用户设置和 API 密钥
- **[视频教程](https://www.youtube.com/@BrightData)** - 分步指南
- **[IDE 视频教程](https://docs.brightdata.com/scraping-automation/web-scraper-ide/videos)** - 学习使用网页爬虫 IDE

---

## 故障排除

### 手动爬虫问题

- **未找到职位：** JOBKOREA 的结构可能已发生变化。检查 `debug.html` 以查看实际的 HTML 结构。
- **编码错误：** 确保为韩文文本正确设置 UTF-8 编码。
- **连接错误：** 在请求之间添加延迟，或使用 Bright Data 代理。

### MCP 抓取工具问题

- **MCP 服务器无法启动：** 确保已安装并更新 Node.js 和 npx。
- **API 令牌错误：** 验证 `.env` 中的令牌是否与你的 Bright Data 账户匹配。
- **结果为空：** 检查 `scraped_data.md`，查看 MCP 返回的内容。

### 网页爬虫 IDE 问题

- **代码生成失败：** 尝试其他 URL，或简化目标页面。
- **生成的代码无法运行：** 使用 IDE 调试器检查页面结构。
- **找不到数据：** 自定义解析器代码，使其与确切的选择器匹配。
- **速率限制：** IDE 使用内置动态代理自动处理此问题。

---

## 总结

本项目展示了三种循序渐进的 JOBKOREA 抓取方法：

1. **手动抓取**可帮助你了解网站的工作原理，并学习网页爬虫的基本概念
2. **基于 MCP 的抓取**通过 Python 集成为生产环境提供可靠性和自动化能力
3. **AI 生成的抓取**无需编写代码，即可通过最快的方式获得可用于生产环境的托管式爬虫工具

### 应该选择哪种方法？

- 如果你正在学习网页爬虫或需要快速构建原型，请**选择方法 1**
- 如果你需要 Python 集成、定时任务或自定义数据管道，请**选择方法 2**
- 如果你希望以最快速度完成设置，并获得托管式基础设施和自动维护功能，请**选择方法 3**

对于抓取 JOBKOREA 这类采用客户端渲染的现代网站的生产应用，<strong>[Bright Data 网页爬虫 IDE](https://www.bright.cn/products/web-scraper)</strong>提供了从构想到部署爬虫的最高效途径。

### 后续步骤

- ✅ 使用不同的搜索查询尝试全部三种方法
- ✅ 通过 [AI 爬虫工作室](https://www.bright.cn/products/web-scraper/studio)探索[网页爬虫 IDE](https://www.bright.cn/products/web-scraper) 中由 AI 生成的抓取工具 
- ✅ 使用 [Bright Data API](https://docs.brightdata.com/api-reference/overview)构建自动化数据管道
- ✅ 使用 [Bright Data 代理网络](https://www.bright.cn/proxy-types)扩展抓取规模
- ✅ 通过[视频教程](https://docs.brightdata.com/scraping-automation/web-scraper-ide/videos)了解更多信息

**需要帮助？** 请查看 [Bright Data 文档](https://docs.brightdata.com/)或[联系支持团队](https://www.bright.cn/contact)。

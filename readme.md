# Lark Todo记忆管理应用 - Claude Code + MCP实现计划

## 🏗 技术架构

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Lark MCP      │    │   Claude Code    │    │   本地应用      │
│                 │◄──►│                  │◄──►│                 │
│ - 消息同步      │    │ - MCP集成        │    │ - UI界面        │
│ - 群组管理      │    │ - 数据处理       │    │ - 本地存储      │
│ - 用户信息      │    │ - Claude API调用 │    │ - 任务管理      │
└─────────────────┘    └──────────────────┘    └─────────────────┘
                              │
                              ▼
                    ┌──────────────────┐
                    │   Claude API     │
                    │                  │
                    │ - 智能提取       │
                    │ - 内容总结       │
                    │ - 记忆生成       │
                    └──────────────────┘
```

---

## 🎯 第一阶段：Claude Code + MCP集成 (1-2周)

### 核心任务
- [x] **设置Claude Code环境**
  - 安装和配置Claude Code
  - 创建项目结构
  - 熟悉MCP协议

- [ ] **Lark MCP集成**
  ```bash
  # 使用Claude Code创建MCP客户端
  claude-code create lark-mcp-client
  ```

- [ ] **基础数据获取**
  - 连接Lark MCP服务器
  - 获取用户消息列表
  - 获取群组信息
  - 测试数据同步

### 技术实现
```javascript
// Claude Code项目结构
├── src/
│   ├── mcp/
│   │   ├── lark-client.js     // Lark MCP客户端
│   │   ├── message-handler.js // 消息处理
│   │   └── sync-manager.js    // 同步管理
│   ├── storage/
│   │   ├── database.js        // 本地数据库
│   │   └── models.js          // 数据模型
│   └── claude/
│       └── api-client.js      // Claude API调用
├── data/                      // 本地数据存储
└── config/
    └── config.json           // 配置文件
```

### 验收标准
- ✅ 成功连接Lark MCP
- ✅ 能获取到消息数据
- ✅ 本地存储消息历史

---

## 🔄 第二阶段：数据处理管道 (2-3周)

### 核心功能
- [ ] **消息预处理**
  ```javascript
  // Claude Code处理逻辑
  class MessageProcessor {
    async processNewMessages(messages) {
      const filtered = this.filterRelevantMessages(messages);
      const structured = this.structureData(filtered);
      return await this.saveToLocal(structured);
    }
  }
  ```

- [ ] **Claude API集成**
  ```javascript
  // 调用Claude API进行智能分析
  async function extractTodos(messageText) {
    const response = await window.claude.complete(`
      分析以下Lark消息，提取其中的任务信息：
      
      消息内容: "${messageText}"
      
      请以JSON格式返回：
      {
        "hasTodo": boolean,
        "tasks": [
          {
            "title": "任务标题",
            "description": "详细描述", 
            "assignee": "负责人",
            "dueDate": "截止时间",
            "priority": "high/medium/low"
          }
        ]
      }
    `);
    return JSON.parse(response);
  }
  ```

- [ ] **本地数据库设计**
  ```sql
  -- SQLite数据库结构
  CREATE TABLE messages (
    id TEXT PRIMARY KEY,
    content TEXT,
    sender TEXT,
    timestamp DATETIME,
    chat_type TEXT,
    processed BOOLEAN DEFAULT FALSE
  );

  CREATE TABLE todos (
    id TEXT PRIMARY KEY,
    title TEXT,
    description TEXT,
    assignee TEXT,
    status TEXT DEFAULT 'pending',
    priority TEXT DEFAULT 'medium',
    due_date DATE,
    source_message_id TEXT,
    created_at DATETIME,
    updated_at DATETIME
  );

  CREATE TABLE memories (
    id TEXT PRIMARY KEY,
    title TEXT,
    content TEXT,
    type TEXT,
    participants TEXT,
    related_todos TEXT,
    created_at DATETIME
  );
  ```

### 验收标准
- ✅ 消息能自动处理并存储
- ✅ Claude API成功提取任务信息
- ✅ 数据库正常运行

---

## 🤖 第三阶段：智能分析优化 (2-3周)

### 核心功能
- [ ] **上下文感知提取**
  ```javascript
  // 增强的Claude提示词
  async function extractWithContext(messages, userProfile, projectContext) {
    const prompt = `
    你是一个专业的项目管理助手，负责从团队对话中提取任务信息。

    上下文信息：
    - 用户：${userProfile.name} (${userProfile.role})
    - 当前项目：${projectContext.projects.join(', ')}
    - 团队成员：${projectContext.members.join(', ')}

    对话历史：
    ${messages.map(m => `${m.sender}: ${m.content}`).join('\n')}

    请分析并提取：
    1. 明确的任务分配
    2. 隐含的工作项
    3. 决策和重要信息
    4. 需要跟进的事项

    返回JSON格式...
    `;
    
    return await window.claude.complete(prompt);
  }
  ```

- [ ] **批量处理优化**
  ```javascript
  // Claude Code后台任务
  class BatchProcessor {
    async processPendingMessages() {
      const pending = await this.db.getPendingMessages();
      const batches = this.createBatches(pending, 10);
      
      for (const batch of batches) {
        await this.processBatch(batch);
        await this.sleep(1000); // 避免API限流
      }
    }
  }
  ```

### 验收标准
- ✅ 提取准确率提升到85%+
- ✅ 能处理复杂的多轮对话
- ✅ 支持批量处理历史消息

---

## 🧠 第四阶段：记忆系统构建 (3-4周)

### 核心功能
- [ ] **智能总结生成**
  ```javascript
  async function generateMemory(completedTodos, relatedMessages) {
    const prompt = `
    基于以下已完成的任务和相关讨论，生成一份工作记忆总结：

    完成的任务：
    ${completedTodos.map(t => `- ${t.title}: ${t.description}`).join('\n')}

    相关讨论：
    ${relatedMessages.map(m => `${m.sender}: ${m.content}`).join('\n')}

    请生成：
    1. 工作总结
    2. 关键决策
    3. 经验教训
    4. 后续建议

    以JSON格式返回记忆内容...
    `;
    
    return await window.claude.complete(prompt);
  }
  ```

- [ ] **模式识别和推荐**
  ```javascript
  async function analyzeWorkPatterns(userHistory) {
    const prompt = `
    分析用户的工作模式，提供个性化建议：

    用户工作历史：${JSON.stringify(userHistory)}

    请分析：
    1. 工作习惯和偏好
    2. 高效时间段
    3. 常见的延期原因
    4. 改进建议

    返回分析结果...
    `;
    
    return await window.claude.complete(prompt);
  }
  ```

### 验收标准
- ✅ 能生成有价值的工作总结
- ✅ 识别个人和团队工作模式
- ✅ 提供有用的改进建议

---

## 🚀 第五阶段：前端界面完善 (2-3周)

### 核心功能
- [ ] **实时数据同步**
  ```javascript
  // 前端与Claude Code通信
  class DataSync {
    constructor() {
      this.ws = new WebSocket('ws://localhost:8080');
      this.setupEventHandlers();
    }

    async getTodos() {
      return await fetch('http://localhost:3001/api/todos').then(r => r.json());
    }

    async syncWithLark() {
      return await fetch('http://localhost:3001/api/sync', {
        method: 'POST'
      });
    }
  }
  ```

- [ ] **数据可视化**
  ```javascript
  // 使用之前创建的React应用，增加数据绑定
  const Dashboard = () => {
    const [todos, setTodos] = useState([]);
    const [memories, setMemories] = useState([]);

    useEffect(() => {
      // 连接本地API获取数据
      dataSync.getTodos().then(setTodos);
      dataSync.getMemories().then(setMemories);
    }, []);

    return (
      // 渲染界面...
    );
  };
  ```

### 验收标准
- ✅ 前端界面与后端数据完全同步
- ✅ 用户操作响应流畅
- ✅ 支持离线查看

---

## 📦 部署和运行

### Claude Code项目启动
```bash
# 启动MCP客户端
claude-code run lark-sync

# 启动API服务器
claude-code serve --port 3001

# 启动前端开发服务器
npm run dev
```

### 配置文件示例
```json
{
  "lark": {
    "mcp_server": "lark://mcp.larksuite.com",
    "credentials": "your-credentials"
  },
  "claude": {
    "api_key": "your-claude-api-key",
    "model": "claude-sonnet-4-20250514"
  },
  "database": {
    "path": "./data/todos.db"
  },
  "sync": {
    "interval": 300000,
    "batch_size": 10
  }
}
```

---

## 🎯 关键优势

### 1. **本地优先**
- 数据完全本地化，隐私安全
- 离线可用，不依赖网络
- 快速响应，无延迟

### 2. **智能增强**
- Claude API提供高质量的分析
- 上下文感知的任务提取
- 个性化的工作建议

### 3. **灵活扩展**
- MCP协议标准化集成
- Claude Code提供强大的自动化能力
- 模块化架构便于维护

### 4. **成本控制**
- 只在需要时调用Claude API
- 本地缓存减少重复请求
- 批量处理提高效率

---

## 📅 实施时间线

| 阶段 | 时间 | 重点 | Claude Code任务 |
|------|------|------|----------------|
| 第一阶段 | 1-2周 | MCP集成 | 建立Lark连接 |
| 第二阶段 | 2-3周 | 数据管道 | 消息处理自动化 |
| 第三阶段 | 2-3周 | 智能分析 | Claude API集成 |
| 第四阶段 | 3-4周 | 记忆系统 | 高级分析算法 |
| 第五阶段 | 2-3周 | 界面完善 | API服务化 |

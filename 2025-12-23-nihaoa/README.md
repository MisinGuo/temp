# 文章发布到存储功能说明

## 功能概述

文章管理页面已经升级，支持将 Markdown 格式的文章直接发布到对象存储（GitHub、MinIO、R2等），并自动生成访问URL。

## 主要特性

### 1. Markdown 编辑器
- 使用 `md-editor-v3` 实现所见即所得的 Markdown 编辑
- 支持实时预览
- 支持图片上传（当前支持 base64，可扩展到存储服务）
- 完整的 Markdown 语法支持（标题、列表、代码块、表格、公式等）

### 2. 存储配置选择
- 支持选择已配置的存储服务（GitHub、MinIO、R2、OSS、COS等）
- 自动获取启用状态的存储配置列表
- 支持多种存储类型

### 3. 路径规则配置
提供4种路径规则：

| 规则 | 说明 | 示例 |
|------|------|------|
| 日期+标题 | `YYYY-MM-DD-{slug}` | `2025-12-23-game-guide/README.md` |
| 年月+标题 | `YYYY-MM-{slug}` | `2025-12-game-guide/README.md` |
| 时间戳+标题 | `{timestamp}-{slug}` | `1703312345678-game-guide/README.md` |
| 仅标题 | `{slug}` | `game-guide/README.md` |

所有路径规则都会在末尾追加 `/README.md`

### 4. 自动URL构建
发布后自动构建访问URL，优先级：
1. 自定义域名（`custom_domain`）
2. CDN URL（`cdn_url`）
3. 默认存储URL：
   - GitHub: `https://raw.githubusercontent.com/{owner}/{repo}/{branch}/{path}`
   - R2: 使用配置的 `r2_public_url`

## 使用流程

### 步骤1: 新建文章
1. 点击"新增"按钮
2. 填写基本信息：
   - **文章标题**（必填）
   - **Slug**（必填）：URL友好的标识符，如 `game-review-2025`
   - **分类**：选择文章分类
   - **作者**：文章作者
   - **语言**：选择语言（zh-CN、en-US、ja-JP）

### 步骤2: 选择存储配置
1. 在"存储配置"下拉框中选择目标存储
2. 选择"路径规则"
3. 查看"预览路径"确认生成的文件路径

### 步骤3: 编写内容
1. 在 Markdown 编辑器中编写文章内容
2. 支持直接粘贴 Markdown 格式文本
3. 使用工具栏插入各种元素（图片、表格、代码块等）
4. 实时预览效果

### 步骤4: 补充信息
- **关键词**：多个关键词用逗号分隔
- **文章摘要**：简短描述
- **封面图**：封面图片URL
- **置顶**：是否置顶显示
- **推荐**：是否推荐
- **排序**：显示顺序

### 步骤5: 发布
1. 点击"发布到存储"按钮
2. 系统将自动：
   - 验证必填字段
   - 将 Markdown 内容上传到选定的存储
   - 生成访问URL
   - 保存文章记录到数据库
   - 设置发布状态为"已发布"

## 数据库字段说明

### 文章表 (gb_articles) 关键字段

| 字段名 | 类型 | 说明 |
|--------|------|------|
| `slug` | varchar(255) | 文章路径标识，用于生成URL |
| `locale` | varchar(10) | 文章语言代码 |
| `storage_config_id` | bigint | 使用的存储配置ID |
| `storage_key` | varchar(500) | 文件在存储中的完整路径 |
| `storage_url` | varchar(1000) | 文章的完整访问URL |
| `is_published` | char(1) | 是否已发布：0-否，1-是 |
| `published_at` | datetime | 发布时间 |

## API接口

### POST /gamebox/article/publish

发布文章到存储

**请求参数：**
```json
{
  "title": "文章标题",
  "slug": "article-slug",
  "locale": "zh-CN",
  "content": "# Markdown内容\n\n这是文章内容...",
  "storageConfigId": 1,
  "storageKey": "2025-12-23-article-slug/README.md",
  "categoryId": 10,
  "author": "作者名",
  "keywords": "关键词1,关键词2",
  "description": "文章摘要",
  "coverUrl": "https://example.com/cover.jpg",
  "isTop": "0",
  "isRecommend": "1",
  "sortOrder": 0
}
```

**响应：**
```json
{
  "code": 200,
  "msg": "发布成功",
  "data": {
    "id": 123,
    "title": "文章标题",
    "storageUrl": "https://example.com/2025-12-23-article-slug/README.md",
    "isPublished": "1",
    "publishedAt": "2025-12-23 10:30:00"
  }
}
```

## 前端组件

### 依赖包
```json
{
  "md-editor-v3": "^6.2.1"
}
```

### 页面路径
```
RuoYi-Vue3/src/views/gamebox/content/article/index.vue
```

## 后端实现

### Controller
```
ruoyi-admin/src/main/java/com/ruoyi/web/controller/gamebox/GbArticleController.java
```

**关键方法：**
- `publishToStorage()`: 处理文章发布请求
- `buildStorageUrl()`: 构建存储访问URL

### Service
- `IGbArticleService`: 文章服务接口
- `IStorageFileService`: 存储文件服务接口
- `IGbStorageConfigService`: 存储配置服务接口

## 扩展功能建议

### 1. 图片上传到存储
当前图片处理使用 base64，可以扩展为：
- 上传图片到同一存储配置
- 自动替换 Markdown 中的图片引用
- 支持拖拽上传

### 2. 文章版本管理
- 每次发布创建新版本
- 支持版本回滚
- 版本对比

### 3. 批量发布
- 选择多篇文章批量发布
- 支持定时发布
- 发布队列管理

### 4. 多语言翻译
- 集成AI翻译
- 一键生成多语言版本
- 自动同步到不同语言目录

### 5. 预览功能增强
- 发布前预览
- 移动端预览
- 暗色模式预览

## 权限配置

需要配置以下权限：

```
gamebox:article:add      # 新增文章
gamebox:article:edit     # 修改文章
gamebox:article:remove   # 删除文章
gamebox:article:publish  # 发布文章到存储
```

## 注意事项

1. **Slug 命名规范**
   - 只使用小写字母、数字和连字符
   - 避免使用特殊字符
   - 保持简洁且有意义

2. **存储配置**
   - 确保存储配置已正确设置并测试通过
   - 注意存储容量限制
   - 定期备份重要内容

3. **路径规则**
   - 选择适合项目的路径规则并保持一致
   - 避免路径冲突

4. **内容格式**
   - 使用标准 Markdown 语法
   - 图片建议使用外链或上传到存储
   - 避免使用过大的 base64 图片

## 故障排查

### 发布失败
1. 检查存储配置是否正确
2. 验证网络连接
3. 查看后端日志

### URL 无法访问
1. 确认存储配置的访问权限
2. 检查 CDN 或自定义域名配置
3. 验证文件是否成功上传

### 编辑器显示异常
1. 清除浏览器缓存
2. 检查依赖包是否正确安装
3. 查看浏览器控制台错误信息

## 更新日志

### v1.0.0 (2025-12-23)
- ✅ 实现 Markdown 编辑器集成
- ✅ 支持多种存储配置
- ✅ 自动路径规则生成
- ✅ URL 自动构建
- ✅ 发布状态管理
- ✅ 列表页面展示优化

---

如有问题或建议，请联系开发团队。

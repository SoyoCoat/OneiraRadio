# 轻量级智能文档配图工具 - 设计文档

## 项目定位

### 产品愿景
一个超简单的工具，让创业者和技术博主能够一键把纯文字内容变成图文并茂的精美文档。

**核心体验：**
> 粘贴文字 → 点击按钮 → 30秒后得到配好图的内容

### 目标用户

**主要用户群体：**

1. **初创公司CEO/创始人**
   - 需要产出内容（公司博客、产品介绍、Newsletter）
   - 有小运营但希望提效
   - 没预算请设计师
   - 对工具的要求：快速、好用、不用学习

2. **兼职技术博主**
   - 在小红书、掘金、知乎等平台发内容
   - 有技术内容但不会配图
   - 希望内容更有吸引力
   - 对工具的要求：结果好看、省时间、适合社交媒体

**用户规模预期：**
- 第一批：10-20个朋友
- 自然增长：口碑传播
- 不主动推广，靠产品本身

### 核心价值主张

**对创业者：**
- "让你的小运营效率翻倍"
- "10分钟产出专业内容"
- "不需要设计师也能做出好看的内容"

**对技术博主：**
- "写完文章，自动配图"
- "小红书笔记秒变图文并茂"
- "技术内容也能很好看"

### 非目标

**不是：**
- ❌ 复杂的企业工具
- ❌ 需要培训才能用的系统
- ❌ 全能的内容创作平台

**只是：**
- ✅ 一个小工具
- ✅ 解决一个具体问题：给文档配图
- ✅ 用完就走

---

## 产品设计

### 核心功能

#### 1. 智能配图（核心）

**输入：**
- Markdown文本
- 或纯文本（自动识别段落）

**处理：**
1. AI分析内容结构
2. 决定在哪里插图（3-5张）
3. 决定每张图的内容
4. 自动生成图片
5. 插入到合适位置

**输出：**
- 带图片的Markdown
- 可直接复制或下载
- 图片已上传到CDN（永久链接）

**特点：**
- 全自动，不需要用户做任何决策
- 快速（30-60秒完成）
- 质量稳定

#### 2. 简单反馈（轻量级"学习"）

**用户可以：**
- 给结果打分（1-5星）
- 简单评论："太好了" / "图片位置不对" / "风格不喜欢"
- 标记特别好的案例："这个很棒！"

**系统会：**
- 记录所有反馈
- 定期（每周/每月）人工回顾
- 手动调整生成策略
- 逐渐优化效果

**不会：**
- ❌ 实时自动学习（太复杂）
- ❌ 个性化每个用户（早期不需要）
- ❌ A/B测试系统（overkill）

#### 3. 平台适配（针对博主）

**小红书模式：**
- 自动生成封面图（1:1或3:4比例）
- 图片风格更活泼、更吸引眼球
- 自动加水印（可选）
- 生成适合手机阅读的排版

**技术博客模式：**
- 生成标准博客配图（16:9）
- 风格简约、专业
- 适合掘金、知乎、个人博客

**通用模式：**
- 标准配图
- 适合任何场景

---

## 系统架构（极简版）

### 整体架构

```
用户界面（Web）
    ↓
后端服务（Flask/FastAPI）
    ↓
┌─────────────┬─────────────┐
│   Claude    │   DALL-E 3  │
│（布局分析）  │  （图片生成）│
└─────────────┴─────────────┘
    ↓
图片CDN（Cloudflare/阿里云）
    ↓
数据库（SQLite/PostgreSQL）
（只存基本记录和反馈）
```

**关键原则：**
- 简单优先
- 能用第三方就不自己造
- 早期不考虑性能问题

### 技术选型

**后端：**
- **语言**：Python 3.11+
- **框架**：FastAPI（快速、现代、异步）
- **部署**：Vercel / Railway / Render（免费或低成本）

**前端：**
- **纯HTML + Vanilla JS**（不用React，保持简单）
- **CSS**：Tailwind CDN（不需要构建）
- **图标**：Lucide Icons（CDN）

**数据库：**
- **早期**：SQLite（文件数据库，够用）
- **扩展**：PostgreSQL（如果真的需要）

**AI服务：**
- **Claude**：Anthropic API（布局分析）
- **DALL-E 3**：OpenAI API（图片生成）

**存储：**
- **图片**：Cloudflare R2（便宜）或阿里云OSS
- **CDN**：自带CDN

**监控：**
- **错误追踪**：Sentry（免费版）
- **分析**：简单的日志就够了

---

## 核心模块设计

### 1. 文档处理器（Document Processor）

**职责：**
- 接收用户输入
- 解析文档结构
- 协调AI服务
- 返回处理结果

**关键代码结构：**

```python
class DocumentProcessor:
    """
    文档处理核心类
    """
    
    async def process(
        self,
        content: str,
        mode: str = "general"  # general/xiaohongshu/blog
    ) -> ProcessedDocument:
        """
        处理文档的主流程
        
        Args:
            content: 原始文本内容
            mode: 平台模式
            
        Returns:
            ProcessedDocument: 处理后的文档对象
        """
        
        # 1. 清理和解析内容
        parsed = self.parse_content(content)
        
        # 2. 调用Claude分析布局
        layout_plan = await self.analyze_layout(
            parsed,
            mode=mode
        )
        
        # 3. 并发生成所有图片
        images = await self.generate_images(
            layout_plan.image_specs
        )
        
        # 4. 组装最终文档
        result = self.assemble_document(
            original_content=content,
            layout_plan=layout_plan,
            images=images,
            mode=mode
        )
        
        # 5. 保存记录（用于后续学习）
        await self.save_record(result)
        
        return result
    
    def parse_content(self, content: str) -> ParsedContent:
        """
        解析内容结构
        
        识别：
        - 标题（# ## ###）
        - 段落
        - 列表
        - 代码块
        - 总字数
        """
        # 使用markdown parser或简单的正则表达式
        pass
    
    async def analyze_layout(
        self,
        parsed: ParsedContent,
        mode: str
    ) -> LayoutPlan:
        """
        调用Claude分析布局
        """
        # 构建prompt（根据mode调整）
        prompt = self.build_analysis_prompt(parsed, mode)
        
        # 调用Claude
        response = await self.claude_client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            system=self.get_system_prompt(mode),
            messages=[{"role": "user", "content": prompt}]
        )
        
        # 解析返回的JSON
        layout = self.parse_layout_response(response.content[0].text)
        
        return layout
    
    async def generate_images(
        self,
        image_specs: List[ImageSpec]
    ) -> List[GeneratedImage]:
        """
        并发生成所有图片
        """
        tasks = [
            self.generate_single_image(spec)
            for spec in image_specs
        ]
        
        images = await asyncio.gather(*tasks)
        
        return images
    
    async def generate_single_image(
        self,
        spec: ImageSpec
    ) -> GeneratedImage:
        """
        生成单张图片
        """
        # 调用DALL-E 3
        response = await self.openai_client.images.generate(
            model="dall-e-3",
            prompt=spec.prompt,
            size=spec.size,
            quality="standard"
        )
        
        image_url = response.data[0].url
        
        # 下载并上传到CDN
        cdn_url = await self.upload_to_cdn(image_url)
        
        return GeneratedImage(
            spec=spec,
            url=cdn_url,
            original_prompt=spec.prompt
        )
    
    def assemble_document(
        self,
        original_content: str,
        layout_plan: LayoutPlan,
        images: List[GeneratedImage],
        mode: str
    ) -> ProcessedDocument:
        """
        组装最终文档
        
        根据mode不同，输出格式可能不同：
        - general: 标准markdown
        - xiaohongshu: 添加emoji，调整排版
        - blog: 添加HTML标签
        """
        # 按段落分割原文
        paragraphs = self.split_into_paragraphs(original_content)
        
        # 插入图片
        for i, image in enumerate(images):
            insert_pos = layout_plan.image_specs[i].position
            
            # 生成图片markdown
            img_markdown = self.format_image_markdown(
                image,
                mode=mode
            )
            
            # 插入到指定位置
            paragraphs.insert(insert_pos, img_markdown)
        
        # 根据mode进行后处理
        final_content = self.post_process(
            paragraphs,
            mode=mode
        )
        
        return ProcessedDocument(
            content=final_content,
            images=images,
            mode=mode
        )
```

---

### 2. Prompt管理器（简化版）

**职责：**
- 管理不同模式的prompt
- 支持手动更新优化
- 记录prompt版本历史

**设计：**

```python
class PromptManager:
    """
    Prompt版本管理（简化版）
    
    核心理念：
    - 不做自动优化（太复杂）
    - 手动迭代优化（基于反馈）
    - 版本控制（记录历史）
    """
    
    def __init__(self):
        self.prompts = self.load_prompts()
        self.current_version = self.get_latest_version()
    
    def get_system_prompt(self, mode: str) -> str:
        """
        获取system prompt
        
        根据mode返回不同的prompt：
        - general: 通用模式
        - xiaohongshu: 小红书模式
        - blog: 技术博客模式
        """
        return self.prompts[mode]['system']
    
    def get_analysis_template(self, mode: str) -> str:
        """
        获取分析prompt模板
        """
        return self.prompts[mode]['analysis']
    
    def update_prompt(
        self,
        mode: str,
        prompt_type: str,  # 'system' or 'analysis'
        new_content: str,
        reason: str  # 更新原因
    ):
        """
        手动更新prompt
        
        由人工根据反馈调整
        """
        # 保存旧版本
        old_version = self.prompts[mode][prompt_type]
        self.save_version_history(
            mode=mode,
            prompt_type=prompt_type,
            old_content=old_version,
            new_content=new_content,
            reason=reason
        )
        
        # 更新当前版本
        self.prompts[mode][prompt_type] = new_content
        
        # 写入文件
        self.save_prompts()
    
    def load_prompts(self) -> Dict:
        """
        从YAML文件加载prompts
        """
        # prompts.yaml:
        # general:
        #   system: "..."
        #   analysis: "..."
        # xiaohongshu:
        #   system: "..."
        #   analysis: "..."
        pass
```

**Prompt文件结构：**

```yaml
# prompts.yaml

version: "1.2.0"
last_updated: "2026-01-15"

general:
  system: |
    你是一个专业的内容编辑和视觉设计专家。
    你的任务是分析文档内容，决定在哪里插入插图。
    
    核心原则：
    1. 插图要有助于理解内容，不是装饰
    2. 位置要合理，不打断阅读流畅度
    3. 数量适中：短文1-2张，中等3-4张，长文4-5张
    4. 风格统一，简约现代
    
  analysis: |
    请分析以下文档，决定插图方案。
    
    文档信息：
    - 总字数: {word_count}
    - 段落数: {paragraph_count}
    
    文档内容：
    {content}
    
    请返回JSON格式...

xiaohongshu:
  system: |
    你是小红书内容运营专家。
    你的任务是为小红书笔记配图。
    
    小红书配图特点：
    1. 封面图最重要，要吸引眼球
    2. 风格活泼、色彩鲜艳
    3. 可以使用emoji和网络流行元素
    4. 图片比例适合手机（1:1或3:4）
    
  analysis: |
    这是一篇准备发小红书的笔记。
    
    内容：
    {content}
    
    请设计配图方案，特别注意封面图...

blog:
  system: |
    你是技术博客编辑。
    你的任务是为技术文章配图。
    
    技术博客配图特点：
    1. 专业、简约
    2. 以图表、示意图为主
    3. 避免过于花哨
    4. 标准16:9比例
```

---

### 3. 反馈收集器（轻量级）

**职责：**
- 收集用户反馈
- 存储反馈数据
- 提供简单的分析界面

**设计：**

```python
class FeedbackCollector:
    """
    反馈收集器（简化版）
    
    只做两件事：
    1. 记录反馈
    2. 方便查看
    
    不做：
    - 自动分析
    - 自动优化
    - 复杂统计
    """
    
    async def collect_rating(
        self,
        document_id: str,
        rating: int,  # 1-5星
        comment: Optional[str] = None,
        user_id: Optional[str] = None
    ):
        """
        收集评分反馈
        """
        feedback = Feedback(
            document_id=document_id,
            rating=rating,
            comment=comment,
            user_id=user_id,
            timestamp=datetime.now()
        )
        
        # 保存到数据库
        await self.db.save_feedback(feedback)
        
        # 如果评分很低（<=2），发送通知
        if rating <= 2:
            await self.notify_low_rating(feedback)
    
    async def mark_as_good_example(
        self,
        document_id: str,
        user_id: str,
        note: str
    ):
        """
        用户标记为"好案例"
        """
        await self.db.mark_example(
            document_id=document_id,
            user_id=user_id,
            note=note
        )
    
    def get_feedback_summary(
        self,
        days: int = 7
    ) -> FeedbackSummary:
        """
        获取反馈摘要
        
        用于人工回顾
        """
        feedbacks = self.db.get_recent_feedbacks(days)
        
        return FeedbackSummary(
            total_count=len(feedbacks),
            avg_rating=sum(f.rating for f in feedbacks) / len(feedbacks),
            low_ratings=[f for f in feedbacks if f.rating <= 2],
            high_ratings=[f for f in feedbacks if f.rating >= 4],
            comments=[f for f in feedbacks if f.comment],
            good_examples=self.db.get_marked_examples(days)
        )
    
    def export_for_review(self) -> str:
        """
        导出反馈数据供人工review
        
        生成markdown格式的报告
        """
        summary = self.get_feedback_summary(days=7)
        
        report = f"""
# 本周反馈报告

## 基本数据
- 总处理文档：{summary.total_count}
- 平均评分：{summary.avg_rating:.2f}/5

## 低分案例（需要关注）
{self._format_low_ratings(summary.low_ratings)}

## 高分案例（值得学习）
{self._format_high_ratings(summary.high_ratings)}

## 用户标记的好案例
{self._format_good_examples(summary.good_examples)}

## 建议
基于本周反馈，考虑：
1. ...
2. ...
"""
        return report
```

---

### 4. 平台适配器

**职责：**
- 根据不同平台调整输出格式
- 处理平台特定的需求

**设计：**

```python
class PlatformAdapter:
    """
    平台适配器
    
    为不同平台做特殊处理
    """
    
    def adapt_for_xiaohongshu(
        self,
        content: str,
        images: List[GeneratedImage]
    ) -> str:
        """
        适配小红书
        
        特殊处理：
        1. 添加emoji
        2. 调整段落间距
        3. 生成话题标签建议
        """
        # 1. 添加emoji（在关键位置）
        content = self.add_emojis(content)
        
        # 2. 生成话题标签
        tags = self.generate_hashtags(content)
        
        # 3. 格式化
        formatted = f"""
{content}

---

{tags}

📸 图片均由AI生成
"""
        return formatted
    
    def adapt_for_blog(
        self,
        content: str,
        images: List[GeneratedImage]
    ) -> str:
        """
        适配技术博客
        
        特殊处理：
        1. 添加目录
        2. 代码高亮标记
        3. SEO优化
        """
        # 1. 生成目录
        toc = self.generate_toc(content)
        
        # 2. 添加元数据
        metadata = f"""
---
title: {self.extract_title(content)}
date: {datetime.now().strftime('%Y-%m-%d')}
tags: {self.extract_tags(content)}
---

{toc}

{content}
"""
        return metadata
    
    def add_emojis(self, content: str) -> str:
        """
        智能添加emoji
        
        规则：
        - 标题前加相关emoji
        - 关键词后加强调emoji
        """
        # 简单规则匹配
        emoji_map = {
            "介绍": "📝",
            "特点": "✨",
            "优势": "💡",
            "总结": "🎯",
            "步骤": "📋"
        }
        
        for keyword, emoji in emoji_map.items():
            if keyword in content:
                content = content.replace(
                    f"## {keyword}",
                    f"{emoji} {keyword}"
                )
        
        return content
    
    def generate_hashtags(self, content: str) -> str:
        """
        生成话题标签
        
        用Claude快速生成
        """
        # 调用Claude生成3-5个相关话题标签
        pass
```

---

## 数据模型（极简）

### 数据库Schema

```sql
-- 文档记录表
CREATE TABLE documents (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(100),  -- email或匿名ID
    mode VARCHAR(20),  -- general/xiaohongshu/blog
    original_content TEXT,
    processed_content TEXT,
    image_count INT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 图片记录表
CREATE TABLE images (
    id VARCHAR(36) PRIMARY KEY,
    document_id VARCHAR(36) REFERENCES documents(id),
    url VARCHAR(500),
    prompt TEXT,
    position INT,  -- 在文档中的位置
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 反馈表
CREATE TABLE feedback (
    id VARCHAR(36) PRIMARY KEY,
    document_id VARCHAR(36) REFERENCES documents(id),
    user_id VARCHAR(100),
    rating INT,  -- 1-5
    comment TEXT,
    is_good_example BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Prompt版本历史表
CREATE TABLE prompt_versions (
    id VARCHAR(36) PRIMARY KEY,
    mode VARCHAR(20),
    prompt_type VARCHAR(20),  -- system/analysis
    content TEXT,
    reason TEXT,  -- 更新原因
    created_by VARCHAR(100),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 核心数据结构

```python
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime

@dataclass
class ParsedContent:
    """解析后的内容"""
    raw_content: str
    paragraphs: List[str]
    headings: List[str]
    word_count: int
    has_code: bool
    has_lists: bool

@dataclass
class ImageSpec:
    """图片规格"""
    position: int  # 插入位置（段落索引）
    prompt: str  # DALL-E prompt
    caption: str  # 图片说明
    size: str = "1024x1024"  # 图片尺寸
    style_hint: str = "modern minimalist"  # 风格提示

@dataclass
class LayoutPlan:
    """布局计划"""
    image_specs: List[ImageSpec]
    total_images: int
    strategy: str  # 策略说明

@dataclass
class GeneratedImage:
    """生成的图片"""
    spec: ImageSpec
    url: str  # CDN URL
    original_url: str  # DALL-E返回的URL
    generated_at: datetime

@dataclass
class ProcessedDocument:
    """处理后的文档"""
    id: str
    original_content: str
    processed_content: str
    images: List[GeneratedImage]
    mode: str
    created_at: datetime

@dataclass
class Feedback:
    """用户反馈"""
    document_id: str
    user_id: Optional[str]
    rating: int
    comment: Optional[str]
    is_good_example: bool
    timestamp: datetime
```

---

## API设计（简化版）

### RESTful Endpoints

```python
from fastapi import FastAPI, UploadFile, File
from pydantic import BaseModel

app = FastAPI()

# ===== 核心功能 =====

class ProcessRequest(BaseModel):
    content: str
    mode: str = "general"  # general/xiaohongshu/blog

class ProcessResponse(BaseModel):
    document_id: str
    processed_content: str
    images: List[dict]
    preview_url: str  # 预览页面URL

@app.post("/api/process")
async def process_document(request: ProcessRequest) -> ProcessResponse:
    """
    处理文档 - 核心API
    
    输入：文本内容 + 模式
    输出：处理后的文档
    """
    processor = DocumentProcessor()
    result = await processor.process(
        content=request.content,
        mode=request.mode
    )
    
    return ProcessResponse(
        document_id=result.id,
        processed_content=result.processed_content,
        images=[img.to_dict() for img in result.images],
        preview_url=f"/preview/{result.id}"
    )

# ===== 反馈相关 =====

class FeedbackRequest(BaseModel):
    document_id: str
    rating: int  # 1-5
    comment: Optional[str] = None

@app.post("/api/feedback")
async def submit_feedback(request: FeedbackRequest):
    """
    提交反馈
    """
    collector = FeedbackCollector()
    await collector.collect_rating(
        document_id=request.document_id,
        rating=request.rating,
        comment=request.comment
    )
    
    return {"status": "ok"}

@app.post("/api/mark-example/{document_id}")
async def mark_as_example(document_id: str, note: str):
    """
    标记为好案例
    """
    collector = FeedbackCollector()
    await collector.mark_as_good_example(
        document_id=document_id,
        note=note
    )
    
    return {"status": "ok"}

# ===== 管理功能（需要认证）=====

@app.get("/api/admin/feedback-summary")
async def get_feedback_summary(days: int = 7):
    """
    获取反馈摘要（管理员）
    """
    collector = FeedbackCollector()
    summary = collector.get_feedback_summary(days)
    
    return summary

@app.get("/api/admin/export-feedback")
async def export_feedback():
    """
    导出反馈报告（管理员）
    """
    collector = FeedbackCollector()
    report = collector.export_for_review()
    
    return {"report": report}

@app.post("/api/admin/update-prompt")
async def update_prompt(
    mode: str,
    prompt_type: str,
    new_content: str,
    reason: str
):
    """
    更新prompt（管理员）
    """
    manager = PromptManager()
    manager.update_prompt(
        mode=mode,
        prompt_type=prompt_type,
        new_content=new_content,
        reason=reason
    )
    
    return {"status": "ok"}
```

---

## 前端设计

### 主界面（单页面）

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>智能配图工具</title>
    <script src="https://cdn.tailwindcss.com"></script>
</head>
<body class="bg-gray-50">
    <div class="max-w-4xl mx-auto p-6">
        <!-- 头部 -->
        <div class="mb-8 text-center">
            <h1 class="text-3xl font-bold mb-2">✨ 智能配图工具</h1>
            <p class="text-gray-600">粘贴文字，一键生成图文内容</p>
        </div>

        <!-- 模式选择 -->
        <div class="mb-4 flex gap-4 justify-center">
            <button class="mode-btn active" data-mode="general">
                📝 通用
            </button>
            <button class="mode-btn" data-mode="xiaohongshu">
                🌟 小红书
            </button>
            <button class="mode-btn" data-mode="blog">
                💻 技术博客
            </button>
        </div>

        <!-- 输入区域 -->
        <div class="mb-6">
            <textarea 
                id="content" 
                class="w-full h-64 p-4 border rounded-lg font-mono text-sm"
                placeholder="在这里粘贴你的内容...

支持Markdown格式，也支持纯文本"
            ></textarea>
        </div>

        <!-- 操作按钮 -->
        <div class="text-center mb-6">
            <button 
                id="process-btn"
                class="bg-blue-600 text-white px-8 py-3 rounded-lg font-medium hover:bg-blue-700 disabled:bg-gray-400"
            >
                🚀 开始处理
            </button>
        </div>

        <!-- 状态显示 -->
        <div id="status" class="hidden mb-6">
            <!-- 状态内容由JS动态更新 -->
        </div>

        <!-- 结果显示 -->
        <div id="result" class="hidden">
            <!-- 结果内容由JS动态更新 -->
        </div>

        <!-- 反馈区域 -->
        <div id="feedback" class="hidden mt-6 p-4 bg-white rounded-lg shadow">
            <h3 class="font-medium mb-3">这个结果怎么样？</h3>
            <div class="flex gap-2 mb-3">
                <button class="rating-btn" data-rating="5">⭐⭐⭐⭐⭐</button>
                <button class="rating-btn" data-rating="4">⭐⭐⭐⭐</button>
                <button class="rating-btn" data-rating="3">⭐⭐⭐</button>
                <button class="rating-btn" data-rating="2">⭐⭐</button>
                <button class="rating-btn" data-rating="1">⭐</button>
            </div>
            <textarea 
                id="feedback-comment"
                class="w-full p-2 border rounded"
                placeholder="有什么建议吗？（可选）"
            ></textarea>
            <button id="submit-feedback" class="mt-2 bg-gray-200 px-4 py-2 rounded">
                提交反馈
            </button>
        </div>
    </div>

    <script>
        // 状态管理
        let currentMode = 'general';
        let currentDocId = null;

        // 模式切换
        document.querySelectorAll('.mode-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelectorAll('.mode-btn').forEach(b => 
                    b.classList.remove('active')
                );
                btn.classList.add('active');
                currentMode = btn.dataset.mode;
            });
        });

        // 处理按钮
        document.getElementById('process-btn').addEventListener('click', async () => {
            const content = document.getElementById('content').value;
            
            if (!content.trim()) {
                alert('请先输入内容');
                return;
            }

            // 显示进度
            showStatus('processing');

            try {
                const response = await fetch('/api/process', {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ content, mode: currentMode })
                });

                const data = await response.json();
                
                // 显示结果
                showResult(data);
                
                // 保存文档ID
                currentDocId = data.document_id;
                
                // 显示反馈表单
                document.getElementById('feedback').classList.remove('hidden');

            } catch (error) {
                showStatus('error');
                console.error(error);
            }
        });

        // 反馈提交
        document.getElementById('submit-feedback').addEventListener('click', async () => {
            if (!currentDocId) return;

            const rating = document.querySelector('.rating-btn.selected')?.dataset.rating;
            const comment = document.getElementById('feedback-comment').value;

            if (!rating) {
                alert('请先打分');
                return;
            }

            await fetch('/api/feedback', {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    document_id: currentDocId,
                    rating: parseInt(rating),
                    comment: comment || null
                })
            });

            alert('感谢反馈！🙏');
            document.getElementById('feedback').classList.add('hidden');
        });

        // 评分按钮
        document.querySelectorAll('.rating-btn').forEach(btn => {
            btn.addEventListener('click', () => {
                document.querySelectorAll('.rating-btn').forEach(b =>
                    b.classList.remove('selected')
                );
                btn.classList.add('selected');
            });
        });

        function showStatus(type) {
            const status = document.getElementById('status');
            status.classList.remove('hidden');

            if (type === 'processing') {
                status.innerHTML = `
                    <div class="text-center p-6 bg-blue-50 rounded-lg">
                        <div class="animate-spin h-8 w-8 border-4 border-blue-600 border-t-transparent rounded-full mx-auto mb-4"></div>
                        <p class="text-blue-600 font-medium">AI正在处理中...</p>
                        <p class="text-sm text-gray-600 mt-2">预计需要30-60秒</p>
                    </div>
                `;
            } else if (type === 'error') {
                status.innerHTML = `
                    <div class="p-6 bg-red-50 rounded-lg text-center">
                        <p class="text-red-600">❌ 处理失败，请重试</p>
                    </div>
                `;
            }
        }

        function showResult(data) {
            document.getElementById('status').classList.add('hidden');
            
            const result = document.getElementById('result');
            result.classList.remove('hidden');
            result.innerHTML = `
                <div class="bg-white rounded-lg shadow-lg p-6">
                    <div class="flex justify-between items-center mb-4">
                        <h2 class="text-xl font-bold">✅ 处理完成</h2>
                        <div class="flex gap-2">
                            <button onclick="downloadMarkdown()" class="px-4 py-2 bg-gray-200 rounded">
                                📥 下载
                            </button>
                            <button onclick="copyToClipboard()" class="px-4 py-2 bg-blue-600 text-white rounded">
                                📋 复制
                            </button>
                        </div>
                    </div>
                    
                    <div class="prose max-w-none">
                        <div class="bg-gray-50 p-4 rounded border overflow-auto">
                            <pre class="whitespace-pre-wrap font-mono text-sm">${escapeHtml(data.processed_content)}</pre>
                        </div>
                    </div>

                    <div class="mt-4">
                        <p class="text-sm text-gray-600">
                            生成了 ${data.images.length} 张图片
                        </p>
                    </div>
                </div>
            `;

            // 保存结果供下载/复制
            window.processedContent = data.processed_content;
        }

        function downloadMarkdown() {
            const blob = new Blob([window.processedContent], { type: 'text/markdown' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = 'processed.md';
            a.click();
        }

        function copyToClipboard() {
            navigator.clipboard.writeText(window.processedContent);
            alert('✅ 已复制到剪贴板');
        }

        function escapeHtml(text) {
            return text
                .replace(/&/g, '&amp;')
                .replace(/</g, '&lt;')
                .replace(/>/g, '&gt;');
        }
    </script>

    <style>
        .mode-btn {
            @apply px-4 py-2 rounded-lg border border-gray-300 hover:border-blue-500 transition;
        }
        .mode-btn.active {
            @apply bg-blue-600 text-white border-blue-600;
        }
        .rating-btn {
            @apply px-3 py-1 border rounded hover:bg-gray-100;
        }
        .rating-btn.selected {
            @apply bg-yellow-100 border-yellow-500;
        }
    </style>
</body>
</html>
```

---

## 核心Prompt设计

### 通用模式Prompt

```yaml
system_prompt: |
  你是一个专业的内容编辑和视觉设计专家。
  你的任务是分析文档内容，决定在哪里插入插图来增强内容的可读性和吸引力。

  核心原则：
  1. 插图要服务于内容理解，不是纯装饰
  2. 位置要合理，不打断阅读流畅度
  3. 数量适中：
     - 短文（<500字）：1-2张
     - 中等（500-1500字）：3-4张
     - 长文（>1500字）：4-5张
  4. 风格统一：简约、现代、专业

  插图位置选择策略：
  - 在解释完一个重要概念后
  - 在长段落之后（>200字）
  - 在话题转换的地方
  - 在数据或案例之后

  插图内容设计策略：
  - 抽象概念 → 用比喻/象征物
  - 具体场景 → 用写实风格
  - 技术内容 → 用简单的图表/示意图
  - 情感内容 → 用温暖的场景

analysis_template: |
  请分析以下文档，设计插图方案。

  文档信息：
  - 总字数: {word_count}
  - 段落数: {paragraph_count}
  - 有代码块: {has_code}
  - 有列表: {has_lists}

  文档内容：
  ```
  {content}
  ```

  请返回JSON格式（只返回JSON，不要其他内容）：
  ```json
  {
    "images": [
      {
        "position": 2,  // 在第几段之后插入（从0开始）
        "prompt": "A minimalist illustration of...",  // 详细的英文描述，DALL-E 3会用
        "caption": "图片说明文字",  // 中文说明
        "reasoning": "为什么在这里插入这张图"  // 简单说明理由
      }
    ],
    "overall_strategy": "整体策略说明"
  }
  ```

  注意：
  - prompt必须详细，包含风格、色彩、构图
  - prompt必须是英文
  - 每个图片都要有明确的purpose
```

### 小红书模式Prompt

```yaml
system_prompt: |
  你是小红书内容运营专家。
  你的任务是为小红书笔记设计配图方案。

  小红书配图特点：
  1. 封面图最重要 - 必须吸引眼球，决定点击率
  2. 风格活泼、色彩鲜艳、有网感
  3. 可以使用流行元素和emoji
  4. 图片比例：1:1（方图）或3:4（竖图）
  5. 配图数量：3-6张为佳

  封面图设计原则：
  - 视觉冲击力强
  - 主题明确
  - 色彩鲜艳但不俗气
  - 文字少或无文字（图片会自动加标题）

  内容图设计原则：
  - 呼应文字内容
  - 风格统一
  - 适合手机阅读

analysis_template: |
  这是一篇准备发小红书的笔记。

  内容：
  ```
  {content}
  ```

  请设计配图方案，特别注意：
  1. 第一张一定是封面图，必须吸引人
  2. 其他图片要呼应内容
  3. 整体风格要适合小红书氛围

  返回JSON：
  ```json
  {
    "cover": {
      "prompt": "Vibrant and eye-catching illustration for social media cover...",
      "caption": "封面图说明",
      "style_keywords": ["colorful", "modern", "social media style"]
    },
    "images": [
      {
        "position": 1,
        "prompt": "...",
        "caption": "...",
        "purpose": "配合第一个重点"
      }
    ],
    "hashtag_suggestions": ["#技术分享", "#干货"],
    "overall_strategy": "策略说明"
  }
  ```
```

### 技术博客模式Prompt

```yaml
system_prompt: |
  你是技术博客编辑。
  你的任务是为技术文章设计配图。

  技术博客配图特点：
  1. 专业、简约、清晰
  2. 以示意图、流程图、架构图为主
  3. 避免过于花哨
  4. 标准16:9比例，适合博客平台
  5. 颜色：中性色调，突出重点

  插图类型：
  - 概念解释 → 简单的图标或示意图
  - 架构说明 → 方框图、流程图风格
  - 对比说明 → 左右对比图
  - 步骤说明 → 带数字的步骤图

analysis_template: |
  这是一篇技术博客文章。

  内容：
  ```
  {content}
  ```

  请设计配图方案，要求：
  1. 专业、简约
  2. 有助于技术理解
  3. 适合博客平台展示

  返回JSON：
  ```json
  {
    "images": [
      {
        "position": 1,
        "prompt": "Clean and professional technical diagram showing...",
        "caption": "图1: 架构示意图",
        "type": "diagram",  // diagram/flowchart/comparison/icon
        "style_keywords": ["minimalist", "technical", "professional"]
      }
    ],
    "overall_strategy": "为这篇技术文章设计了X张示意图，帮助读者理解..."
  }
  ```
```

---

## 部署方案

### 推荐方案：Vercel + Cloudflare

**为什么选这个组合：**
- ✅ 部署简单（几分钟搞定）
- ✅ 免费额度够用（早期）
- ✅ 自动HTTPS
- ✅ 全球CDN
- ✅ 零运维

### 部署步骤

```bash
# 1. 项目结构
smart-illustration-tool/
├── api/
│   ├── __init__.py
│   ├── main.py          # FastAPI入口
│   ├── processor.py     # 文档处理器
│   ├── prompts.py       # Prompt管理
│   └── feedback.py      # 反馈收集
├── static/
│   └── index.html       # 前端页面
├── prompts.yaml         # Prompt配置
├── requirements.txt     # Python依赖
├── vercel.json         # Vercel配置
└── README.md

# 2. vercel.json配置
{
  "builds": [
    {
      "src": "api/main.py",
      "use": "@vercel/python"
    },
    {
      "src": "static/**",
      "use": "@vercel/static"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "api/main.py"
    },
    {
      "src": "/(.*)",
      "dest": "static/$1"
    }
  ],
  "env": {
    "ANTHROPIC_API_KEY": "@anthropic_api_key",
    "OPENAI_API_KEY": "@openai_api_key"
  }
}

# 3. 一键部署
vercel deploy --prod
```

### 图片CDN：Cloudflare R2

```python
# 图片上传到R2
import boto3

s3 = boto3.client(
    's3',
    endpoint_url='https://YOUR_ACCOUNT_ID.r2.cloudflarestorage.com',
    aws_access_key_id=os.getenv('R2_ACCESS_KEY'),
    aws_secret_access_key=os.getenv('R2_SECRET_KEY')
)

def upload_to_cdn(image_data: bytes, filename: str) -> str:
    """上传图片到R2并返回CDN URL"""
    s3.put_object(
        Bucket='illustrations',
        Key=filename,
        Body=image_data,
        ContentType='image/png'
    )
    
    # 返回CDN URL
    return f"https://cdn.yourdomain.com/{filename}"
```

---

## 成本估算

### API成本（单次处理）

**Claude分析：**
- 输入：~1K tokens（文档内容）
- 输出：~1K tokens（布局分析JSON）
- 成本：$0.003 (input) + $0.015 (output) = **$0.018**

**DALL-E 3生成：**
- 3张图片 × $0.04 = **$0.12**

**单次总成本：~$0.14**

### 月度成本预估

**场景1：轻度使用（10个朋友，每人每周5次）**
- 使用次数：10人 × 5次/周 × 4周 = 200次/月
- AI成本：200 × $0.14 = **$28/月**
- 基础设施：**$0**（Vercel免费版够用）
- **总计：~$30/月**

**场景2：中度使用（50个用户，每人每周3次）**
- 使用次数：50人 × 3次/周 × 4周 = 600次/月
- AI成本：600 × $0.14 = **$84/月**
- 基础设施：Vercel Pro **$20/月**（需要更多带宽）
- **总计：~$100/月**

**场景3：重度使用（200个用户）**
- 使用次数：200人 × 3次/周 × 4周 = 2400次/月
- AI成本：2400 × $0.14 = **$336/月**
- 基础设施：**$50/月**（需要独立服务器）
- **总计：~$400/月**

### 收费策略（如果需要）

**免费版：**
- 每月5次免费
- 适合尝鲜用户

**付费版：**
- $9.9/月：50次/月
- $29/月：200次/月
- $99/月：无限使用

**成本回收：**
- 10个付费用户（$29） = $290收入
- 覆盖50人的使用成本
- 早期不收费，积累用户和反馈

---

## 开发路线图

### Phase 1: MVP（1周）

**目标：**
核心功能可用，自己和朋友能用

**任务：**
- [x] 基础文档处理流程
- [x] Claude布局分析
- [x] DALL-E图片生成
- [x] 简单的Web界面
- [x] 通用模式prompt
- [x] 部署到Vercel

**不包含：**
- 用户系统
- 反馈收集
- 平台适配
- 数据库

**交付标准：**
能把markdown变成带图的markdown

### Phase 2: 反馈收集（3天）

**目标：**
收集用户反馈，用于后续优化

**任务：**
- [x] 添加评分功能（1-5星）
- [x] 添加评论功能
- [x] 简单的反馈数据库（SQLite）
- [x] 管理后台（查看反馈）

**交付标准：**
能看到用户觉得好不好

### Phase 3: 平台适配（3天）

**目标：**
针对小红书和技术博客优化

**任务：**
- [x] 小红书模式prompt
- [x] 技术博客模式prompt
- [x] 平台适配器（emoji、标签等）
- [x] 模式切换UI

**交付标准：**
不同平台生成结果有明显差异

### Phase 4: 体验优化（1周）

**任务：**
- [x] 处理速度优化（并发生成）
- [x] 错误处理优化
- [x] UI美化
- [x] 添加示例/教程
- [x] 图片预览功能

**交付标准：**
用户体验smooth，没有明显bug

### Phase 5: 基础调教（持续）

**目标：**
根据反馈逐步优化prompt

**任务：**
- 每周review反馈数据
- 找出低分案例的共同问题
- 手动调整prompt
- 记录优化历史

**节奏：**
- 每周五review
- 每两周更新一次prompt
- 持续3个月

### Phase 6: 扩展功能（如果需要）

**可能的方向：**
- 用户系统（邮箱登录）
- 历史记录
- 批量处理
- API接口
- WordPress插件
- 收费功能

**原则：**
只有在用户明确需要时才做

---

## 风险与应对

### 技术风险

**1. AI服务不稳定**
- **风险**：Claude或DALL-E可能downtime
- **应对**：
  - 添加重试机制
  - 缓存成功的生成结果
  - 准备降级方案（手动处理）

**2. 成本失控**
- **风险**：用户很多时，API成本可能很高
- **应对**：
  - 设置每用户使用上限
  - 早期自己cover，够用前不收费
  - 考虑缓存相似prompt的结果

**3. 生成质量不稳定**
- **风险**：AI生成的图片有时候很糟糕
- **应对**：
  - 允许用户regenerate单张图
  - 收集bad cases，优化prompt
  - 实在不行就人工介入

### 产品风险

**1. 用户不愿意用**
- **风险**：朋友试了觉得不好用
- **应对**：
  - 早期密集收集反馈
  - 快速迭代
  - 如果确实需求不存在，及时stop

**2. 结果不如人意**
- **风险**：AI配图效果不如人工
- **应对**：
  - 定位为"辅助工具"，不是完全替代
  - 强调"省时间"而不是"完美"
  - 允许用户手动调整

**3. 竞品出现**
- **风险**：大公司可能推出类似功能
- **应对**：
  - 快速积累用户
  - 建立口碑
  - 垂直场景（博主）深耕

### 运营风险

**1. 维护负担**
- **风险**：出bug时没时间修
- **应对**：
  - 代码简单易维护
  - 添加完善的错误监控
  - 早期用户少，问题少

**2. 用户期望过高**
- **风险**：朋友期望太高，失望
- **应对**：
  - 明确定位：小工具，不是完整产品
  - 管理预期：省时间，不是完美
  - 快速响应反馈

---

## 成功指标

### 产品指标

**核心指标：**
- **用户满意度**：平均评分 >4.0/5
  - 衡量：好不好用
- **使用频率**：周活跃用户 / 总用户 >30%
  - 衡量：有没有用
- **完成率**：处理成功 / 尝试处理 >90%
  - 衡量：稳不稳定

**次要指标：**
- 平均处理时间 <60秒
- 用户留存率（1周后还用） >50%
- 推荐率（用户推荐给朋友） >20%

### 学习系统指标

**优化效果：**
- Prompt版本迭代频率：每2周1次
- 每次迭代评分提升 >0.1分
- 低分案例占比 <20%

### 增长指标（如果做大）

**用户增长：**
- 月新增用户 >20%
- 自然增长（朋友推荐） >50%
- 留存率（3个月） >30%

**商业指标（如果收费）：**
- 付费转化率 >5%
- 客单价 >$20
- CAC < $10（通过口碑，获客成本低）

---

## 后续迭代方向

### 短期（1-3个月）

**基于反馈优化：**
1. Prompt持续优化（最重要）
2. 增加图片风格选项
3. 支持用户自定义规则
4. 添加更多平台模式

### 中期（3-6个月）

**扩展功能：**
1. 用户系统（保存历史）
2. 批量处理
3. 团队协作功能
4. WordPress/Notion插件

### 长期（6-12个月）

**如果真的做大：**
1. 自动学习系统（而不是手动）
2. 个性化风格记忆
3. API开放
4. 企业版功能

### 但是...

**现在不想这么多。**

**先做MVP，给朋友用，看反馈。**

好用就继续，不好用就算了。

反正就写着玩。

---

## 附录：快速启动指南

### 最小可行版本（200行代码）

**文件1：api/main.py**

```python
from fastapi import FastAPI, Request
from fastapi.staticfiles import StaticFiles
from anthropic import Anthropic
from openai import OpenAI
import json
import os
import asyncio

app = FastAPI()
app.mount("/static", StaticFiles(directory="static"), name="static")

claude = Anthropic(api_key=os.getenv("ANTHROPIC_API_KEY"))
openai_client = OpenAI(api_key=os.getenv("OPENAI_API_KEY"))

PROMPT = """
分析这个文档，决定在哪里插图（3-5张）。

文档：
{content}

返回JSON：
{{
  "images": [
    {{
      "position": 段落索引,
      "prompt": "英文描述",
      "caption": "中文说明"
    }}
  ]
}}
"""

@app.post("/api/process")
async def process(request: Request):
    data = await request.json()
    content = data['content']
    
    # 1. Claude分析
    response = claude.messages.create(
        model="claude-sonnet-4-20250514",
        max_tokens=2000,
        messages=[{
            "role": "user",
            "content": PROMPT.format(content=content)
        }]
    )
    
    layout = json.loads(response.content[0].text)
    
    # 2. 生成图片（并发）
    async def gen(spec):
        r = await openai_client.images.generate(
            model="dall-e-3",
            prompt=spec['prompt'],
            size="1024x1024"
        )
        return r.data[0].url
    
    tasks = [gen(spec) for spec in layout['images']]
    images = await asyncio.gather(*tasks)
    
    # 3. 插入markdown
    paras = content.split('\n\n')
    for i, spec in enumerate(layout['images']):
        img_md = f"\n\n![{spec['caption']}]({images[i]})\n*{spec['caption']}*\n\n"
        paras.insert(spec['position'], img_md)
    
    result = '\n\n'.join(paras)
    
    return {"processed_content": result, "images": images}

if __name__ == "__main__":
    import uvicorn
    uvicorn.run(app, host="0.0.0.0", port=8000)
```

**文件2：static/index.html**

（前面已经提供，这里省略）

**部署：**

```bash
# 1. 安装依赖
pip install fastapi anthropic openai uvicorn

# 2. 本地运行
python api/main.py

# 3. 访问
http://localhost:8000
```

**完成！**

---

## 总结

这是一个**轻量级**的智能配图工具设计。

**核心特点：**
- ✅ 简单易用（一键处理）
- ✅ 快速开发（1周MVP）
- ✅ 低成本运行（早期<$50/月）
- ✅ 渐进优化（基于反馈手动调教）
- ✅ 适合小规模（10-100个朋友）

**不是：**
- ❌ 复杂的企业系统
- ❌ 自动学习的AI系统
- ❌ 需要融资的startup

**就是：**
- ✅ 一个好用的小工具
- ✅ 给朋友省时间
- ✅ 写着玩，顺便可能有用

**下一步：**
1. 花1周时间实现MVP
2. 给CEO朋友试用
3. 给几个博主朋友试用
4. 收集反馈
5. 决定要不要继续

**如果好用：**
自然会有更多人要用，到时候再迭代。

**如果不好用：**
也就几天时间，学到了什么是真实需求。

---

*设计文档 v1.0*
*创建日期: 2026-01-15*
*作者: Zhilin*
*定位: 写着玩的小工具*

# 智能文档插图系统设计文档

## 项目概述

### 产品定位
一个AI驱动的文档美化工具，能够：
1. 分析Markdown文档内容
2. 智能决定插图位置和内容
3. 自动生成并插入插图
4. 通过人类反馈持续学习优化
5. 最终目标：自动化初创公司运营人员的内容美化工作

### 核心价值
- **自动化内容美化**：从纯文字到图文并茂
- **持续学习优化**：系统会越用越聪明
- **降低内容生产成本**：减少对专业设计师的依赖

---

## 系统架构

### 整体架构图

```
┌─────────────────────────────────────────────────────────────┐
│                         用户层                               │
│  ┌──────────┐  ┌──────────┐  ┌──────────────────┐         │
│  │ Web UI   │  │ API接口  │  │  反馈训练界面    │         │
│  └──────────┘  └──────────┘  └──────────────────┘         │
└─────────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────────┐
│                      业务逻辑层                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │           文档处理引擎 (Orchestrator)               │    │
│  │  • 解析Markdown                                     │    │
│  │  • 协调各个AI服务                                   │    │
│  │  • 管理生成流程                                     │    │
│  └────────────────────────────────────────────────────┘    │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │ 布局分析器   │  │ 插图生成器   │  │  质量评估器    │  │
│  │ (Claude)     │  │ (DALL-E 3)   │  │  (Claude)      │  │
│  └──────────────┘  └──────────────┘  └────────────────┘  │
│                                                              │
│  ┌────────────────────────────────────────────────────┐    │
│  │           学习系统 (Learning System)                │    │
│  │  • 收集人类反馈                                     │    │
│  │  • 更新System Prompt                                │    │
│  │  • A/B测试不同策略                                  │    │
│  └────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────┘
                           │
┌─────────────────────────────────────────────────────────────┐
│                       数据层                                 │
│  ┌────────────┐  ┌────────────┐  ┌──────────────────┐     │
│  │ 文档数据库 │  │ 反馈数据库 │  │  Prompt版本库    │     │
│  │ (文档历史) │  │ (训练数据) │  │  (系统提示词)    │     │
│  └────────────┘  └────────────┘  └──────────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心模块设计

### 1. 文档处理引擎 (Document Processor)

**职责：**
- 接收Markdown输入
- 解析文档结构
- 协调AI服务
- 生成最终输出

**核心流程：**

```python
class DocumentProcessor:
    """
    文档处理主引擎
    """
    
    def __init__(self):
        self.layout_analyzer = LayoutAnalyzer()      # 布局分析器
        self.image_generator = ImageGenerator()       # 图片生成器
        self.quality_evaluator = QualityEvaluator()  # 质量评估器
        self.learning_system = LearningSystem()       # 学习系统
    
    async def process_document(
        self, 
        markdown_content: str,
        user_requirements: Optional[str] = None,
        document_type: Optional[str] = None
    ) -> ProcessedDocument:
        """
        处理文档的主流程
        
        Args:
            markdown_content: 原始markdown内容
            user_requirements: 用户特殊要求（可选）
            document_type: 文档类型（博客/报告/社交媒体等）
            
        Returns:
            ProcessedDocument: 处理后的文档对象
        """
        
        # 1. 解析文档结构
        doc_structure = self.parse_markdown(markdown_content)
        
        # 2. 布局分析 - 决定在哪插图
        layout_plan = await self.layout_analyzer.analyze(
            doc_structure=doc_structure,
            user_requirements=user_requirements,
            document_type=document_type
        )
        
        # 3. 生成图片
        images = await self.image_generator.generate_batch(
            layout_plan.illustration_specs
        )
        
        # 4. 组装文档
        processed_doc = self.assemble_document(
            original_content=markdown_content,
            layout_plan=layout_plan,
            images=images
        )
        
        # 5. 质量评估（可选）
        if self.should_evaluate():
            quality_score = await self.quality_evaluator.evaluate(
                processed_doc
            )
            processed_doc.quality_score = quality_score
        
        # 6. 保存到数据库（用于后续学习）
        await self.save_for_learning(processed_doc)
        
        return processed_doc
    
    def parse_markdown(self, content: str) -> DocumentStructure:
        """解析markdown结构"""
        # 使用markdown parser提取：
        # - 标题层级
        # - 段落
        # - 列表
        # - 代码块
        # - 现有图片
        pass
    
    def assemble_document(
        self, 
        original_content: str,
        layout_plan: LayoutPlan,
        images: List[GeneratedImage]
    ) -> ProcessedDocument:
        """组装最终文档"""
        # 将生成的图片插入到指定位置
        # 生成图片的markdown语法
        # 处理图片caption
        pass
```

---

### 2. 布局分析器 (Layout Analyzer)

**职责：**
- 分析文档内容和结构
- 决定插图位置
- 生成插图prompt
- 应用学习到的规则

**核心设计：**

```python
class LayoutAnalyzer:
    """
    布局分析器 - 决定在哪里插什么图
    """
    
    def __init__(self):
        self.claude_client = anthropic.Anthropic()
        self.prompt_manager = PromptManager()  # 管理system prompt版本
    
    async def analyze(
        self,
        doc_structure: DocumentStructure,
        user_requirements: Optional[str],
        document_type: Optional[str]
    ) -> LayoutPlan:
        """
        分析文档并生成布局计划
        """
        
        # 1. 获取当前最优的system prompt
        system_prompt = self.prompt_manager.get_current_prompt(
            document_type=document_type
        )
        
        # 2. 构建分析请求
        analysis_request = self._build_analysis_request(
            doc_structure=doc_structure,
            user_requirements=user_requirements,
            document_type=document_type
        )
        
        # 3. 调用Claude分析
        response = await self.claude_client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4000,
            system=system_prompt,
            messages=[{
                "role": "user",
                "content": analysis_request
            }]
        )
        
        # 4. 解析返回的布局计划
        layout_plan = self._parse_layout_plan(response.content[0].text)
        
        # 5. 后处理和验证
        layout_plan = self._validate_and_adjust(layout_plan, doc_structure)
        
        return layout_plan
    
    def _build_analysis_request(
        self,
        doc_structure: DocumentStructure,
        user_requirements: Optional[str],
        document_type: Optional[str]
    ) -> str:
        """
        构建给Claude的分析请求
        
        核心信息：
        - 文档结构（标题、段落、长度）
        - 文档类型（博客、报告、社交媒体）
        - 用户特殊要求
        - 要求返回结构化的布局计划
        """
        
        request = f"""
# 文档插图分析任务

## 文档信息
- 文档类型: {document_type or '通用'}
- 总字数: {doc_structure.total_words}
- 章节数: {len(doc_structure.sections)}

## 文档结构
{self._format_structure(doc_structure)}

## 用户要求
{user_requirements or '无特殊要求'}

## 任务
请分析这个文档，决定：
1. 是否需要封面图（cover）
2. 需要在哪些位置插入插图
3. 每个插图的内容应该是什么

## 输出格式
请返回JSON格式：
```json
{{
  "cover": {{
    "needed": true/false,
    "prompt": "封面图的详细描述（如果需要）",
    "style": "插图风格（minimalist/photorealistic/illustration等）",
    "reasoning": "为什么需要/不需要封面图"
  }},
  "illustrations": [
    {{
      "position": {{"after_line": 行号, "section": "章节标题"}},
      "prompt": "插图的详细英文描述",
      "purpose": "插图的作用（说明概念/增强情感/展示案例等）",
      "style": "插图风格",
      "caption": "图片说明文字（中文）",
      "reasoning": "为什么在这里插入这个插图"
    }}
  ],
  "overall_strategy": "整体布局策略的说明"
}}
```

## 布局原则
{self._get_layout_principles()}
"""
        return request
    
    def _get_layout_principles(self) -> str:
        """
        获取布局原则
        这些原则会随着系统学习而更新
        """
        # 从PromptManager获取当前最优的原则
        return self.prompt_manager.get_layout_principles()
```

---

### 3. Prompt管理系统 (Prompt Manager)

**职责：**
- 管理system prompt的版本
- 根据反馈更新prompt
- A/B测试不同版本
- 追踪哪个版本效果最好

**核心设计：**

```python
class PromptManager:
    """
    Prompt版本管理和优化系统
    
    核心功能：
    1. 版本控制 - 管理不同版本的system prompt
    2. 动态更新 - 根据反馈自动优化prompt
    3. A/B测试 - 测试不同prompt的效果
    4. 回滚机制 - 如果新版本效果差可以回滚
    """
    
    def __init__(self):
        self.db = Database()
        self.current_version = self._load_current_version()
        self.ab_test_manager = ABTestManager()
    
    def get_current_prompt(
        self, 
        document_type: Optional[str] = None
    ) -> str:
        """
        获取当前最优的system prompt
        
        可能基于：
        - A/B测试中的随机分配
        - 文档类型的特定prompt
        - 最新的优化版本
        """
        
        # 如果在A/B测试中，随机选择版本
        if self.ab_test_manager.is_testing():
            version = self.ab_test_manager.select_version()
        else:
            version = self.current_version
        
        # 根据文档类型可能有不同的prompt变体
        base_prompt = self._load_prompt_version(version)
        
        if document_type:
            type_specific = self._load_type_specific_rules(document_type)
            return self._merge_prompts(base_prompt, type_specific)
        
        return base_prompt
    
    def get_layout_principles(self) -> str:
        """
        获取当前的布局原则
        这些原则会被插入到分析请求中
        """
        principles = self.db.get_latest_principles()
        return self._format_principles(principles)
    
    async def update_from_feedback(
        self, 
        feedback: UserFeedback
    ):
        """
        根据用户反馈更新prompt
        
        这是系统"学习"的核心机制
        """
        
        # 1. 分析反馈内容
        analysis = await self._analyze_feedback(feedback)
        
        # 2. 提取可学习的规则
        new_rules = self._extract_learnable_rules(analysis)
        
        # 3. 生成新的prompt版本
        new_version = await self._generate_new_prompt_version(
            current_prompt=self.current_version,
            new_rules=new_rules,
            feedback=feedback
        )
        
        # 4. 保存新版本
        version_id = self.db.save_prompt_version(new_version)
        
        # 5. 启动A/B测试（对比新旧版本）
        self.ab_test_manager.start_test(
            version_a=self.current_version,
            version_b=version_id,
            test_duration_days=7
        )
    
    async def _generate_new_prompt_version(
        self,
        current_prompt: str,
        new_rules: List[str],
        feedback: UserFeedback
    ) -> str:
        """
        用Claude生成新的prompt版本
        
        这是"元学习"：用AI来优化AI的prompt
        """
        
        meta_prompt = f"""
你是一个prompt工程专家。你的任务是优化一个文档布局分析系统的system prompt。

## 当前System Prompt
```
{current_prompt}
```

## 新的学习内容
基于用户反馈，我们发现了以下规律：
{self._format_rules(new_rules)}

## 用户反馈详情
{feedback.to_json()}

## 任务
请更新system prompt，使其：
1. 保留所有有效的现有规则
2. 整合新学到的规律
3. 解决用户反馈中指出的问题
4. 保持prompt的清晰度和可执行性

请返回完整的新版本system prompt。
"""
        
        # 调用Claude生成新prompt
        response = await self.claude_client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=8000,
            messages=[{"role": "user", "content": meta_prompt}]
        )
        
        return response.content[0].text
    
    async def _analyze_feedback(
        self, 
        feedback: UserFeedback
    ) -> FeedbackAnalysis:
        """
        用Claude分析用户反馈
        
        提取：
        - 用户喜欢什么
        - 用户不喜欢什么
        - 可以改进的地方
        - 可以提炼为规则的模式
        """
        
        analysis_prompt = f"""
分析这个用户反馈，提取可学习的规律。

## 生成的文档
{feedback.generated_document}

## 用户评价
评分: {feedback.rating}/5
评论: {feedback.comment}

## 用户修改
{feedback.user_modifications if feedback.user_modifications else '无修改'}

## 分析任务
1. 用户最满意的是什么？
2. 用户最不满意的是什么？
3. 如果用户做了修改，修改了什么？这说明了什么？
4. 从这个案例中可以提炼出什么普遍规律？

请返回结构化的分析结果。
"""
        
        # 调用Claude分析
        response = await self.claude_client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            messages=[{"role": "user", "content": analysis_prompt}]
        )
        
        return FeedbackAnalysis.from_json(response.content[0].text)
```

---

### 4. 学习系统 (Learning System)

**职责：**
- 收集用户反馈
- 构建训练数据集
- 驱动prompt优化
- 追踪系统性能指标

**核心设计：**

```python
class LearningSystem:
    """
    系统学习和优化模块
    
    学习来源：
    1. 显式反馈 - 用户评分和评论
    2. 隐式反馈 - 用户的修改行为
    3. 示例学习 - 用户上传的"好案例"
    """
    
    def __init__(self):
        self.db = Database()
        self.prompt_manager = PromptManager()
    
    async def collect_explicit_feedback(
        self,
        document_id: str,
        rating: int,  # 1-5星
        comment: Optional[str],
        user_id: str
    ):
        """
        收集显式反馈
        
        触发时机：
        - 用户对生成结果打分
        - 用户写评论
        """
        
        feedback = UserFeedback(
            document_id=document_id,
            rating=rating,
            comment=comment,
            user_id=user_id,
            timestamp=datetime.now()
        )
        
        # 保存反馈
        await self.db.save_feedback(feedback)
        
        # 如果是极端反馈（很好或很差），立即分析
        if rating >= 4 or rating <= 2:
            await self.prompt_manager.update_from_feedback(feedback)
    
    async def collect_implicit_feedback(
        self,
        document_id: str,
        user_modifications: DocumentModifications,
        user_id: str
    ):
        """
        收集隐式反馈
        
        触发时机：
        - 用户修改了生成的文档
        - 用户删除了某些插图
        - 用户调整了插图位置
        - 用户修改了图片prompt
        """
        
        # 分析用户做了什么修改
        modification_analysis = self._analyze_modifications(
            user_modifications
        )
        
        # 如果修改很significant，这可能说明生成策略有问题
        if modification_analysis.is_significant:
            feedback = UserFeedback(
                document_id=document_id,
                rating=3,  # 中性评分
                implicit_feedback=modification_analysis,
                user_id=user_id,
                timestamp=datetime.now()
            )
            
            await self.prompt_manager.update_from_feedback(feedback)
    
    async def learn_from_example(
        self,
        example_document: str,
        user_annotation: str,
        document_type: str
    ):
        """
        从用户提供的"好案例"中学习
        
        用户可以上传：
        - 他们认为排版很好的文档
        - 并说明为什么好
        
        系统会：
        1. 分析这个案例的特点
        2. 提取布局规律
        3. 更新prompt
        """
        
        # 1. 用Claude分析这个好案例
        analysis_prompt = f"""
分析这个用户认为"排版很好"的文档案例。

## 文档内容
{example_document}

## 用户评价
{user_annotation}

## 分析任务
1. 这个文档的布局有什么特点？
2. 插图的位置选择有什么规律？
3. 插图的内容和风格有什么特点？
4. 可以提炼出什么普遍适用的规则？

请详细分析，提取可以应用到其他文档的布局原则。
"""
        
        response = await self.claude_client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=3000,
            messages=[{"role": "user", "content": analysis_prompt}]
        )
        
        analysis = response.content[0].text
        
        # 2. 将分析结果转化为规则
        new_rules = self._extract_rules_from_analysis(analysis)
        
        # 3. 更新prompt
        await self.prompt_manager.update_from_feedback(
            UserFeedback(
                example_document=example_document,
                example_analysis=analysis,
                new_rules=new_rules,
                document_type=document_type
            )
        )
        
        # 4. 保存案例到知识库
        await self.db.save_example_case(
            document=example_document,
            annotation=user_annotation,
            analysis=analysis,
            rules=new_rules,
            document_type=document_type
        )
    
    def get_performance_metrics(
        self,
        time_range: TimeRange
    ) -> PerformanceMetrics:
        """
        获取系统性能指标
        
        追踪：
        - 平均用户评分
        - 用户修改率（多少用户修改了生成结果）
        - 不同prompt版本的效果对比
        - 不同文档类型的表现
        """
        
        return self.db.get_metrics(time_range)
```

---

### 5. 图片生成器 (Image Generator)

**职责：**
- 调用DALL-E 3生成图片
- 批量生成管理
- 图片缓存和CDN
- 失败重试

**核心设计：**

```python
class ImageGenerator:
    """
    图片生成模块
    """
    
    def __init__(self):
        self.openai_client = OpenAI()
        self.image_cache = ImageCache()
        self.cdn_manager = CDNManager()
    
    async def generate_batch(
        self,
        illustration_specs: List[IllustrationSpec]
    ) -> List[GeneratedImage]:
        """
        批量生成图片
        
        优化：
        - 并发生成
        - 检查缓存（相同prompt可能已经生成过）
        - 失败重试
        """
        
        tasks = []
        for spec in illustration_specs:
            # 检查缓存
            cached = await self.image_cache.get(spec.prompt)
            if cached:
                tasks.append(asyncio.create_task(
                    self._return_cached(cached)
                ))
            else:
                tasks.append(asyncio.create_task(
                    self._generate_single(spec)
                ))
        
        results = await asyncio.gather(*tasks)
        return results
    
    async def _generate_single(
        self,
        spec: IllustrationSpec
    ) -> GeneratedImage:
        """
        生成单张图片
        """
        
        # 1. 优化prompt（确保DALL-E 3能理解）
        optimized_prompt = self._optimize_prompt_for_dalle(spec.prompt)
        
        # 2. 调用DALL-E 3
        try:
            response = await self.openai_client.images.generate(
                model="dall-e-3",
                prompt=optimized_prompt,
                size=spec.size or "1024x1024",
                quality=spec.quality or "standard",
                style=spec.style or "natural",
                n=1
            )
            
            image_url = response.data[0].url
            revised_prompt = response.data[0].revised_prompt
            
        except Exception as e:
            # 失败重试
            return await self._retry_generation(spec, e)
        
        # 3. 下载图片并上传到CDN
        cdn_url = await self.cdn_manager.upload_from_url(image_url)
        
        # 4. 缓存
        generated = GeneratedImage(
            spec=spec,
            url=cdn_url,
            original_url=image_url,
            revised_prompt=revised_prompt
        )
        
        await self.image_cache.set(spec.prompt, generated)
        
        return generated
    
    def _optimize_prompt_for_dalle(
        self,
        original_prompt: str
    ) -> str:
        """
        优化prompt使其更适合DALL-E 3
        
        DALL-E 3的特点：
        - 偏好详细描述
        - 理解风格词（minimalist, photorealistic等）
        - 可以理解一些艺术家风格
        """
        
        # 可以用Claude来优化prompt
        # 或者有一套规则
        
        # 简单示例：确保prompt足够详细
        if len(original_prompt.split()) < 10:
            # prompt太短，可能需要扩展
            return self._expand_prompt(original_prompt)
        
        return original_prompt
```

---

### 6. 质量评估器 (Quality Evaluator)

**职责：**
- 评估生成文档的质量
- 提供改进建议
- 用于A/B测试的自动评估

**核心设计：**

```python
class QualityEvaluator:
    """
    质量评估模块
    
    评估维度：
    1. 插图位置是否合理
    2. 插图内容是否相关
    3. 整体布局是否和谐
    4. 是否符合文档类型的特点
    """
    
    def __init__(self):
        self.claude_client = anthropic.Anthropic()
    
    async def evaluate(
        self,
        document: ProcessedDocument
    ) -> QualityScore:
        """
        评估文档质量
        """
        
        evaluation_prompt = f"""
作为专业的内容编辑，请评估这个生成的文档。

## 原始文档
{document.original_content}

## 布局计划
{document.layout_plan.to_json()}

## 生成的插图
{self._format_illustrations(document.images)}

## 评估维度
1. 插图位置（1-5分）
   - 是否在合适的段落
   - 是否打断了阅读流畅度
   - 频率是否合适（太多/太少）

2. 插图内容（1-5分）
   - 是否和文字内容相关
   - 是否有助于理解
   - 风格是否统一

3. 整体布局（1-5分）
   - 视觉平衡
   - 图文比例
   - 是否符合该类型文档的标准

4. 创意性（1-5分）
   - 插图是否有趣
   - 是否超出预期
   - 是否memorable

请返回JSON格式的评估结果：
```json
{{
  "scores": {{
    "position": X,
    "relevance": X,
    "layout": X,
    "creativity": X
  }},
  "overall": X.X,
  "strengths": ["优点1", "优点2"],
  "weaknesses": ["不足1", "不足2"],
  "suggestions": ["建议1", "建议2"]
}}
```
"""
        
        response = await self.claude_client.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            messages=[{"role": "user", "content": evaluation_prompt}]
        )
        
        return QualityScore.from_json(response.content[0].text)
```

---

## 数据模型

### 核心数据结构

```python
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime
from enum import Enum

class DocumentType(Enum):
    """文档类型"""
    BLOG = "blog"                    # 博客文章
    REPORT = "report"                # 报告
    SOCIAL_MEDIA = "social_media"    # 社交媒体内容
    TUTORIAL = "tutorial"            # 教程
    NEWS = "news"                    # 新闻
    PRODUCT = "product"              # 产品介绍
    GENERAL = "general"              # 通用

class IllustrationStyle(Enum):
    """插图风格"""
    MINIMALIST = "minimalist"
    PHOTOREALISTIC = "photorealistic"
    ILLUSTRATION = "illustration"
    ABSTRACT = "abstract"
    ISOMETRIC = "isometric"

@dataclass
class DocumentStructure:
    """文档结构"""
    total_words: int
    sections: List['Section']
    has_code_blocks: bool
    has_lists: bool
    has_tables: bool
    sentiment: str  # positive/neutral/negative/mixed
    
@dataclass
class Section:
    """文档章节"""
    level: int              # 标题级别 (h1, h2, h3...)
    title: str
    content: str
    word_count: int
    line_start: int
    line_end: int

@dataclass
class IllustrationSpec:
    """插图规格"""
    position: 'InsertPosition'
    prompt: str
    purpose: str            # 插图目的
    style: IllustrationStyle
    caption: str            # 图片说明
    size: str = "1024x1024"
    quality: str = "standard"
    reasoning: str = ""     # 为什么在这里插入
    
@dataclass
class InsertPosition:
    """插入位置"""
    after_line: int
    section_title: Optional[str]
    before_element: Optional[str]  # 在某个元素之前（如"第一个列表"）

@dataclass
class LayoutPlan:
    """布局计划"""
    cover: Optional['CoverSpec']
    illustrations: List[IllustrationSpec]
    overall_strategy: str
    estimated_image_count: int
    
@dataclass
class CoverSpec:
    """封面规格"""
    needed: bool
    prompt: str
    style: IllustrationStyle
    reasoning: str

@dataclass
class GeneratedImage:
    """生成的图片"""
    spec: IllustrationSpec
    url: str                    # CDN URL
    original_url: str           # DALL-E返回的原始URL
    revised_prompt: str         # DALL-E修正后的prompt
    generated_at: datetime
    
@dataclass
class ProcessedDocument:
    """处理后的文档"""
    document_id: str
    original_content: str
    layout_plan: LayoutPlan
    images: List[GeneratedImage]
    final_markdown: str
    quality_score: Optional['QualityScore']
    prompt_version: str         # 使用的prompt版本
    created_at: datetime
    user_id: str

@dataclass
class UserFeedback:
    """用户反馈"""
    document_id: str
    user_id: str
    rating: Optional[int]       # 1-5星
    comment: Optional[str]
    user_modifications: Optional['DocumentModifications']
    implicit_feedback: Optional['ImplicitFeedback']
    example_document: Optional[str]  # 用户提供的好案例
    example_analysis: Optional[str]
    new_rules: Optional[List[str]]
    timestamp: datetime
    
@dataclass
class DocumentModifications:
    """用户对文档的修改"""
    deleted_images: List[int]        # 删除了哪些图片（索引）
    moved_images: List['ImageMove']  # 移动了哪些图片
    modified_prompts: List['PromptModification']  # 修改了哪些prompt
    added_images: List['ManualImage']  # 手动添加的图片
    
@dataclass
class QualityScore:
    """质量评分"""
    position_score: float       # 位置评分
    relevance_score: float      # 相关性评分
    layout_score: float         # 布局评分
    creativity_score: float     # 创意性评分
    overall_score: float        # 总分
    strengths: List[str]
    weaknesses: List[str]
    suggestions: List[str]
```

---

## 数据库设计

### 表结构

```sql
-- 文档表
CREATE TABLE documents (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    original_content TEXT NOT NULL,
    document_type VARCHAR(50),
    final_markdown TEXT,
    prompt_version VARCHAR(36),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at)
);

-- 布局计划表
CREATE TABLE layout_plans (
    id VARCHAR(36) PRIMARY KEY,
    document_id VARCHAR(36) NOT NULL,
    plan_json JSON NOT NULL,  -- 存储完整的LayoutPlan
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (document_id) REFERENCES documents(id) ON DELETE CASCADE
);

-- 生成的图片表
CREATE TABLE generated_images (
    id VARCHAR(36) PRIMARY KEY,
    document_id VARCHAR(36) NOT NULL,
    spec_json JSON NOT NULL,
    url VARCHAR(500) NOT NULL,
    original_url VARCHAR(500),
    revised_prompt TEXT,
    generated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (document_id) REFERENCES documents(id) ON DELETE CASCADE,
    INDEX idx_document_id (document_id)
);

-- 用户反馈表
CREATE TABLE user_feedback (
    id VARCHAR(36) PRIMARY KEY,
    document_id VARCHAR(36) NOT NULL,
    user_id VARCHAR(36) NOT NULL,
    rating INT,  -- 1-5
    comment TEXT,
    modifications_json JSON,  -- DocumentModifications
    implicit_feedback_json JSON,
    feedback_type VARCHAR(50),  -- explicit/implicit/example
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (document_id) REFERENCES documents(id) ON DELETE CASCADE,
    INDEX idx_document_id (document_id),
    INDEX idx_user_id (user_id),
    INDEX idx_created_at (created_at)
);

-- Prompt版本表
CREATE TABLE prompt_versions (
    id VARCHAR(36) PRIMARY KEY,
    version_number INT AUTO_INCREMENT UNIQUE,
    system_prompt TEXT NOT NULL,
    layout_principles TEXT,
    document_type VARCHAR(50),
    parent_version VARCHAR(36),  -- 基于哪个版本改进的
    created_by VARCHAR(50),  -- human/auto
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    is_active BOOLEAN DEFAULT FALSE,
    performance_score FLOAT,  -- 基于A/B测试的效果评分
    INDEX idx_version_number (version_number),
    INDEX idx_created_at (created_at)
);

-- 好案例库
CREATE TABLE example_cases (
    id VARCHAR(36) PRIMARY KEY,
    document_type VARCHAR(50),
    example_document TEXT NOT NULL,
    user_annotation TEXT,
    ai_analysis TEXT,
    extracted_rules JSON,
    upvotes INT DEFAULT 0,  -- 其他用户觉得这是好案例
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_document_type (document_type),
    INDEX idx_upvotes (upvotes)
);

-- A/B测试表
CREATE TABLE ab_tests (
    id VARCHAR(36) PRIMARY KEY,
    test_name VARCHAR(100),
    version_a VARCHAR(36),
    version_b VARCHAR(36),
    start_date DATE,
    end_date DATE,
    is_active BOOLEAN DEFAULT TRUE,
    result_json JSON,  -- 测试结果统计
    winner VARCHAR(36),  -- 哪个版本赢了
    FOREIGN KEY (version_a) REFERENCES prompt_versions(id),
    FOREIGN KEY (version_b) REFERENCES prompt_versions(id)
);

-- 性能指标表
CREATE TABLE performance_metrics (
    id VARCHAR(36) PRIMARY KEY,
    date DATE NOT NULL,
    prompt_version VARCHAR(36),
    document_type VARCHAR(50),
    total_documents INT,
    avg_rating FLOAT,
    modification_rate FLOAT,  -- 用户修改率
    avg_quality_score FLOAT,
    metrics_json JSON,  -- 详细指标
    INDEX idx_date (date),
    INDEX idx_prompt_version (prompt_version)
);
```

---

## API设计

### RESTful API

```python
from fastapi import FastAPI, UploadFile, File, BackgroundTasks
from pydantic import BaseModel

app = FastAPI()

# ============ 核心功能 API ============

@app.post("/api/v1/documents/process")
async def process_document(
    markdown_content: str,
    document_type: Optional[DocumentType] = None,
    user_requirements: Optional[str] = None,
    user_id: str = None
) -> ProcessedDocumentResponse:
    """
    处理文档 - 核心API
    
    输入：
    - markdown_content: 原始markdown文本
    - document_type: 文档类型（可选）
    - user_requirements: 用户特殊要求（可选）
    
    返回：
    - 处理后的文档
    - 生成的图片URL
    - 最终的markdown
    """
    pass

@app.get("/api/v1/documents/{document_id}")
async def get_document(document_id: str) -> ProcessedDocumentResponse:
    """获取已处理的文档"""
    pass

@app.post("/api/v1/documents/{document_id}/regenerate")
async def regenerate_illustrations(
    document_id: str,
    regenerate_specs: List[int]  # 要重新生成哪些图片（索引）
) -> ProcessedDocumentResponse:
    """重新生成部分插图"""
    pass

# ============ 反馈 API ============

@app.post("/api/v1/feedback/rating")
async def submit_rating(
    document_id: str,
    rating: int,  # 1-5
    comment: Optional[str] = None,
    user_id: str = None
):
    """
    提交评分反馈
    
    触发系统学习
    """
    pass

@app.post("/api/v1/feedback/modifications")
async def submit_modifications(
    document_id: str,
    modifications: DocumentModifications,
    user_id: str = None
):
    """
    提交修改反馈
    
    当用户修改了生成的文档时调用
    """
    pass

@app.post("/api/v1/feedback/example")
async def submit_example_case(
    example_document: str,
    annotation: str,
    document_type: DocumentType,
    user_id: str = None
):
    """
    提交好案例
    
    用户上传一个他们认为排版很好的文档
    系统会分析并学习
    """
    pass

# ============ 学习系统 API ============

@app.get("/api/v1/learning/metrics")
async def get_learning_metrics(
    start_date: str,
    end_date: str
) -> PerformanceMetrics:
    """
    获取系统性能指标
    
    用于监控系统是否在进步
    """
    pass

@app.get("/api/v1/learning/prompt-versions")
async def get_prompt_versions() -> List[PromptVersionInfo]:
    """
    获取prompt版本历史
    
    查看系统是如何演进的
    """
    pass

@app.post("/api/v1/learning/manual-update")
async def manual_prompt_update(
    new_principles: List[str],
    admin_token: str
):
    """
    手动更新布局原则
    
    管理员可以手动添加规则
    """
    pass

# ============ A/B测试 API ============

@app.post("/api/v1/ab-test/start")
async def start_ab_test(
    version_a: str,
    version_b: str,
    duration_days: int,
    admin_token: str
):
    """启动A/B测试"""
    pass

@app.get("/api/v1/ab-test/{test_id}/results")
async def get_test_results(test_id: str) -> ABTestResults:
    """获取A/B测试结果"""
    pass

# ============ 示例案例库 API ============

@app.get("/api/v1/examples")
async def get_example_cases(
    document_type: Optional[DocumentType] = None,
    limit: int = 10
) -> List[ExampleCase]:
    """
    获取好案例
    
    用户可以浏览系统学到的好案例
    """
    pass

@app.post("/api/v1/examples/{example_id}/upvote")
async def upvote_example(example_id: str, user_id: str):
    """给好案例点赞"""
    pass
```

---

## 前端界面设计

### 主要页面

#### 1. 文档处理页面

```
┌────────────────────────────────────────────────────┐
│  智能文档插图系统                          用户：张三  │
├────────────────────────────────────────────────────┤
│                                                     │
│  [上传文档] [粘贴文本]                              │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │                                               │  │
│  │  # 这是文档标题                               │  │
│  │                                               │  │
│  │  这里是文档内容...                            │  │
│  │                                               │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  文档类型: [博客文章 ▼]                            │
│                                                     │
│  特殊要求（可选）:                                  │
│  ┌──────────────────────────────────────────────┐  │
│  │ 例如: 插图要简约风格，多用图表...            │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  [生成插图] 预计需要 15 秒                         │
│                                                     │
└────────────────────────────────────────────────────┘
```

#### 2. 结果展示页面

```
┌────────────────────────────────────────────────────┐
│  生成结果                              [下载] [分享]  │
├────────────────────────────────────────────────────┤
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │  [封面图]                                     │  │
│  │  ┌────────────────────────────┐              │  │
│  │  │                             │              │  │
│  │  │    AI生成的封面             │   [重新生成] │  │
│  │  │                             │              │  │
│  │  └────────────────────────────┘              │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  # 文档标题                                        │
│                                                     │
│  第一段内容...                                      │
│                                                     │
│  ┌────────────────────────────┐                    │
│  │                             │                    │
│  │   插图1                     │  [重新生成]       │
│  │                             │  [删除]           │
│  │   图片说明文字              │  [调整位置]       │
│  └────────────────────────────┘                    │
│                                                     │
│  第二段内容...                                      │
│                                                     │
│  ──────────────────────────────────────────────    │
│                                                     │
│  满意这个结果吗？                                   │
│  ★ ★ ★ ★ ☆  (4/5)                              │
│                                                     │
│  [有话要说？添加评论]                              │
│                                                     │
└────────────────────────────────────────────────────┘
```

#### 3. 反馈训练页面

```
┌────────────────────────────────────────────────────┐
│  帮助系统学习                                       │
├────────────────────────────────────────────────────┤
│                                                     │
│  上传你认为排版很好的文档案例                       │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │  [拖拽文件到这里] 或 [点击上传]               │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  为什么你觉得这个案例好？                          │
│  ┌──────────────────────────────────────────────┐  │
│  │  • 插图位置选择很合理                         │  │
│  │  • 图文比例恰当                               │  │
│  │  • 风格统一                                   │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  [提交案例]                                         │
│                                                     │
│  ──────────────────────────────────────────────    │
│                                                     │
│  其他用户提交的好案例:                             │
│                                                     │
│  [案例1] "技术博客的极简风格"  ↑125               │
│  [案例2] "产品介绍的活泼风格"  ↑98                │
│  [案例3] "深度报告的专业风格"  ↑87                │
│                                                     │
└────────────────────────────────────────────────────┘
```

#### 4. 系统学习仪表盘（管理员）

```
┌────────────────────────────────────────────────────┐
│  系统学习仪表盘                          管理员界面  │
├────────────────────────────────────────────────────┤
│                                                     │
│  性能指标（最近30天）                              │
│  ┌──────────────────────────────────────────────┐  │
│  │  平均用户评分: 4.2/5  ↑ 0.3                  │  │
│  │  用户修改率: 23%  ↓ 5%                        │  │
│  │  文档处理量: 1,247  ↑ 15%                     │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  Prompt版本历史                                    │
│  ┌──────────────────────────────────────────────┐  │
│  │  v1.0  2026-01-01  基础版本  评分: 3.8       │  │
│  │  v1.1  2026-01-08  学习案例#1-5  评分: 4.0   │  │
│  │  v1.2  2026-01-15  当前版本  评分: 4.2  ✓   │  │
│  │  v1.3  2026-01-22  测试中 (A/B)  评分: ?     │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  当前A/B测试                                       │
│  ┌──────────────────────────────────────────────┐  │
│  │  v1.2 vs v1.3                                 │  │
│  │  剩余时间: 3天                                │  │
│  │  样本量: 156 vs 162                           │  │
│  │  v1.2: 4.2分  v1.3: 4.3分                    │  │
│  │  [查看详情]                                   │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│  [手动添加规则] [查看学习日志]                     │
│                                                     │
└────────────────────────────────────────────────────┘
```

---

## 技术栈建议

### 后端
- **框架**: FastAPI (Python)
  - 异步支持
  - 自动API文档
  - 类型检查
  
- **AI服务**:
  - Claude (Anthropic) - 布局分析、质量评估、学习
  - DALL-E 3 (OpenAI) - 图片生成
  
- **数据库**:
  - MySQL/PostgreSQL - 结构化数据
  - Redis - 缓存
  
- **消息队列**:
  - Celery + Redis - 异步任务（图片生成）
  
- **存储**:
  - AWS S3 / 阿里云OSS - 图片存储
  - CloudFront / 阿里云CDN - 图片CDN

### 前端
- **框架**: React + TypeScript
- **UI库**: Ant Design / Material-UI
- **Markdown编辑器**: Monaco Editor / CodeMirror
- **状态管理**: Zustand / Jotai

### DevOps
- **容器化**: Docker
- **编排**: Kubernetes (可选)
- **CI/CD**: GitHub Actions
- **监控**: Prometheus + Grafana
- **日志**: ELK Stack

---

## 开发路线图

### Phase 1: MVP（2周）
**目标**: 基础功能可用

**功能**:
- ✅ Markdown解析
- ✅ Claude布局分析
- ✅ DALL-E 3图片生成
- ✅ 文档组装
- ✅ 基础Web UI

**不包含**:
- 学习系统（用固定prompt）
- 质量评估
- A/B测试

### Phase 2: 反馈系统（1-2周）
**功能**:
- ✅ 用户评分和评论
- ✅ 修改行为追踪
- ✅ 反馈数据库
- ✅ 基础的反馈界面

### Phase 3: 学习系统（2-3周）
**功能**:
- ✅ Prompt版本管理
- ✅ 基于反馈的prompt优化
- ✅ 好案例学习
- ✅ 性能指标追踪

### Phase 4: 高级功能（2-3周）
**功能**:
- ✅ A/B测试系统
- ✅ 质量自动评估
- ✅ 多种文档类型支持
- ✅ 批量处理

### Phase 5: 商业化（持续）
**功能**:
- ✅ 用户系统和认证
- ✅ 付费计划
- ✅ API服务
- ✅ 企业功能（团队协作、品牌定制等）

---

## 商业模式

### 目标客户
1. **初创公司**
   - 需要产出大量内容（博客、社交媒体）
   - 预算有限，没有专职设计师
   - 需要快速迭代

2. **内容创作者**
   - 自媒体博主
   - Newsletter作者
   - 技术文档作者

3. **企业**
   - 市场部门
   - 产品部门（产品文档）
   - HR部门（员工手册等）

### 定价策略

**免费版**:
- 10个文档/月
- 基础插图生成
- 标准质量图片

**Pro版** ($29/月):
- 100个文档/月
- 高质量图片
- 自定义风格
- 优先生成
- 反馈训练功能

**Team版** ($99/月):
- 500个文档/月
- 团队协作
- 品牌定制
- API访问
- 专属训练模型

**Enterprise版** (定制):
- 无限文档
- 私有部署
- 专属支持
- SLA保证

### 收入预测（保守估计）

假设：
- 第一年获得 1000个付费用户
- 80% Pro用户，20% Team用户
- 月留存率 85%

**第一年收入**:
- Pro: 800 × $29 × 12 × 0.85 = $236,160
- Team: 200 × $99 × 12 × 0.85 = $201,960
- **总计**: ~$438,000

### 成本结构

**固定成本**:
- 服务器 + CDN: $500/月
- 数据库: $200/月
- 监控和日志: $100/月

**变动成本** (最大头):
- Claude API: ~$0.015/1K tokens输出
  - 每个文档约 2K tokens = $0.03
- DALL-E 3: $0.04/张图
  - 每个文档平均 3张图 = $0.12
- **每个文档总成本**: ~$0.15

**毛利率**:
- Pro用户: $29/100文档 = $0.29/文档
- 成本: $0.15/文档
- **毛利率**: ~48%

---

## 风险与挑战

### 技术风险
1. **AI不稳定性**
   - Claude/DALL-E可能downtime
   - 生成质量波动
   - **缓解**: 多个fallback方案，图片缓存

2. **成本控制**
   - API调用成本可能失控
   - **缓解**: 严格的rate limiting，缓存策略

3. **学习系统效果**
   - Prompt优化可能没效果
   - **缓解**: 人工介入，A/B测试验证

### 商业风险
1. **市场需求验证**
   - 目标客户是否真的需要这个？
   - **缓解**: 尽快MVP，快速验证

2. **竞争**
   - 大公司可能快速跟进
   - **缓解**: 快速积累学习数据，建立护城河

3. **用户获取成本**
   - To B获客成本可能很高
   - **缓解**: 内容营销，PLG策略

### 法律风险
1. **版权问题**
   - AI生成图片的版权归属
   - **缓解**: 明确ToS，使用允许商用的模型

2. **数据隐私**
   - 用户文档可能包含敏感信息
   - **缓解**: 数据加密，明确隐私政策，GDPR合规

---

## 成功指标 (KPIs)

### 产品指标
- **用户留存率**: 月活/注册用户
  - 目标: >40% (第一个月)
- **文档完成率**: 生成后用户实际使用的比例
  - 目标: >70%
- **用户满意度**: 平均评分
  - 目标: >4.0/5

### 学习系统指标
- **系统改进速度**: Prompt版本迭代频率
  - 目标: 每2周一个新版本
- **学习效果**: 新版本vs旧版本评分提升
  - 目标: 每次迭代提升 >0.1分
- **反馈收集率**: 多少用户提供反馈
  - 目标: >30%

### 商业指标
- **MRR**: 月度经常性收入
  - 目标: 第6个月达到 $10K
- **CAC**: 客户获取成本
  - 目标: <$50
- **LTV/CAC**: 客户生命周期价值 / 获客成本
  - 目标: >3

---

## 附录：示例System Prompt

### 布局分析器的初始Prompt

```
你是一个专业的内容编辑和布局设计专家。你的任务是分析文档内容，决定在哪里插入插图，以及插图应该是什么内容。

## 核心原则

### 插图位置选择
1. **引入复杂概念后**：在解释了一个复杂概念后，用插图帮助理解
2. **段落间过渡**：在话题转换的地方，用插图作为视觉分隔
3. **长文本分割**：如果连续文字超过500字，考虑插入插图
4. **关键信息强调**：在重要的结论或数据后，用视觉化强化记忆

### 插图内容设计
1. **概念性插图**：用于解释抽象概念
   - 使用比喻或象征
   - 简约风格为主
   
2. **场景性插图**：用于展示具体场景
   - 写实风格
   - 包含人物和环境
   
3. **数据可视化**：用于展示数据和趋势
   - 图表风格
   - 清晰的视觉层次

### 插图数量控制
- **短文** (<1000字): 1-2张
- **中等** (1000-3000字): 2-4张
- **长文** (>3000字): 4-6张
- **过多会分散注意力，过少显得单调**

### 不应该插图的情况
- 代码块之间（会打断阅读）
- 列表项中间
- 引用文字中间
- 段落开头（应该在段落结尾或章节开始）

## 文档类型特殊规则

### 博客文章
- 必须有吸引人的封面图
- 插图要活泼，带有情感
- 可以使用meme风格或幽默插图

### 技术报告
- 封面要专业、简约
- 插图以图表和示意图为主
- 避免过于花哨的风格

### 社交媒体内容
- 封面图最重要，要有冲击力
- 插图要简单、易懂
- 可以使用大胆的颜色

### 教程文档
- 每个步骤配插图
- 插图要清晰展示操作
- 使用等距视角或UI截图风格

## 输出要求

返回JSON格式的布局计划，包含：
- cover: 封面图规格（如果需要）
- illustrations: 插图列表
- overall_strategy: 整体策略说明

每个插图必须包含：
- position: 精确的插入位置
- prompt: 详细的英文描述（DALL-E 3会理解）
- purpose: 插图的作用
- style: 推荐的风格
- caption: 中文图片说明
- reasoning: 为什么在这里插入

## 注意事项

1. **Prompt质量决定图片质量**：
   - 要具体、详细
   - 描述风格、色彩、构图
   - 避免抽象词汇，用具体意象

2. **风格一致性**：
   - 同一文档中所有插图应该风格统一
   - 除非特殊需要，不要混用写实和插画风格

3. **文化敏感性**：
   - 考虑目标读者的文化背景
   - 避免可能冒犯的意象

4. **可行性**：
   - DALL-E 3擅长：风景、物体、场景、人物
   - DALL-E 3不擅长：精确的文字、复杂图表、特定人物肖像

记住：好的插图应该**增强理解，而不是分散注意力**。
```

---

## 总结

这个系统的核心创新点：

1. **AI分析 + AI生成**：用Claude分析布局，用DALL-E生成图片
2. **持续学习**：系统会从用户反馈中学习，越用越聪明
3. **可训练性**：用户可以教系统什么是"好的布局"
4. **商业化路径清晰**：针对初创公司的实际需求

**下一步行动**：
1. 实现MVP，验证核心价值
2. 找到第一批种子用户
3. 快速迭代，积累训练数据
4. 建立学习飞轮，形成护城河

---

*设计文档 v1.0*
*创建日期: 2026年1月14日*
*为 Wonderland 项目设计*

# 智能内容生产系统 - 完整设计文档

## 产品愿景

一个为初创公司CEO和自媒体博主打造的**智能内容生产中台**。

**核心理念：**
> CEO专注思考和决策，系统负责内容生产。

**用户场景：**
```
周一早上，CEO喝着咖啡：
"上周我们拿到了新一轮融资，帮我写篇博文宣布一下，风格参考YC那篇"

系统30秒后：
✅ 从CEO日记里提取融资相关的进展
✅ 分析YC那篇文章的风格特点
✅ 从素材库里找到合适的团队照片
✅ 生成草稿：标题+正文+配图+排版
✅ CEO看一眼，调整两句话，发布

总共耗时：3分钟
```

---

## 系统架构总览

### 四大核心模块

```
┌────────────────────────────────────────────────────────────┐
│                     智能内容生产系统                        │
├────────────────────────────────────────────────────────────┤
│                                                             │
│  ┌──────────────┐  ┌──────────────┐  ┌─────────────────┐ │
│  │  Module 1    │  │  Module 2    │  │   Module 3      │ │
│  │  图片素材库  │  │ 公众号学习器 │  │   CEO日记本     │ │
│  │              │  │              │  │                 │ │
│  │ • AI标签     │  │ • 自动抓取   │  │ • 进展记录      │ │
│  │ • 智能搜索   │  │ • 风格分析   │  │ • 想法速记      │ │
│  │ • 优先使用   │  │ • 套路提取   │  │ • 自动整理      │ │
│  └──────────────┘  └──────────────┘  └─────────────────┘ │
│         │                  │                   │           │
│         └──────────────────┴───────────────────┘           │
│                            ↓                                │
│              ┌──────────────────────────┐                  │
│              │    Module 4 (核心)       │                  │
│              │   智能博文生成器          │                  │
│              │                          │                  │
│              │ • 上下文理解             │                  │
│              │ • 市场趋势分析           │                  │
│              │ • 自动配图               │                  │
│              │ • 风格适配               │                  │
│              └──────────────────────────┘                  │
│                            ↓                                │
│                   发布就绪的博文                            │
└────────────────────────────────────────────────────────────┘
```

---

## Module 1: 图片素材库 + AI标签器

### 核心价值

**问题：**
- 公司积累了几百张图片（产品截图、活动照片、办公室、团队）
- 每次配图都要：
  1. 翻半天找合适的图
  2. 或者重新生成（花钱+没公司特色）
  3. 用过的图记不住，重复使用

**解决方案：**
- AI自动给每张图打标签
- 智能搜索："找张适合产品介绍的图"
- 配图时优先用素材库（省钱+有公司特色）

### 功能设计

#### 1.1 图片上传和管理

```python
class ImageLibrary:
    """
    图片素材库管理
    """
    
    async def upload_image(
        self,
        image_file: UploadFile,
        user_id: str,
        manual_tags: List[str] = []
    ) -> ImageAsset:
        """
        上传图片到素材库
        
        流程：
        1. 上传到CDN
        2. AI自动分析并打标签
        3. 提取图片特征（颜色、构图等）
        4. 存入数据库
        """
        
        # 1. 上传到CDN
        cdn_url = await self.cdn.upload(image_file)
        
        # 2. AI视觉分析
        analysis = await self.analyze_image(cdn_url)
        
        # 3. 生成标签
        auto_tags = self.generate_tags(analysis)
        all_tags = list(set(auto_tags + manual_tags))
        
        # 4. 保存
        asset = ImageAsset(
            id=generate_id(),
            user_id=user_id,
            url=cdn_url,
            tags=all_tags,
            metadata=analysis,
            uploaded_at=datetime.now()
        )
        
        await self.db.save_asset(asset)
        
        return asset
    
    async def analyze_image(self, image_url: str) -> ImageAnalysis:
        """
        用Claude Vision分析图片
        
        提取：
        - 主要内容（人物、产品、场景）
        - 情绪色调（专业、活泼、温馨）
        - 构图特点（中心、留白、对称）
        - 颜色主调
        - 适用场景
        """
        
        # 调用Claude Vision
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1000,
            messages=[{
                "role": "user",
                "content": [
                    {
                        "type": "image",
                        "source": {
                            "type": "url",
                            "url": image_url
                        }
                    },
                    {
                        "type": "text",
                        "text": IMAGE_ANALYSIS_PROMPT
                    }
                ]
            }]
        )
        
        # 解析分析结果
        analysis = self.parse_analysis(response.content[0].text)
        
        return analysis
    
    def generate_tags(self, analysis: ImageAnalysis) -> List[str]:
        """
        基于分析结果生成标签
        
        标签类型：
        - 内容标签：人物、产品、办公室、活动
        - 场景标签：封面图、插图、配图、社交媒体
        - 风格标签：专业、活泼、温馨、极简
        - 情感标签：正式、轻松、激动、平静
        """
        tags = []
        
        # 内容标签
        if analysis.has_people:
            tags.append("人物")
            if analysis.people_count > 5:
                tags.append("团队照")
        
        if analysis.has_product:
            tags.append("产品")
            tags.append("产品截图")
        
        # 场景标签
        if analysis.composition == "centered":
            tags.append("适合封面")
        
        if analysis.has_whitespace:
            tags.append("适合配文字")
        
        # 风格标签
        if analysis.color_palette == "corporate":
            tags.append("专业")
        elif analysis.color_palette == "vibrant":
            tags.append("活泼")
        
        return tags
```

#### 1.2 智能搜索

```python
class ImageSearch:
    """
    智能图片搜索
    """
    
    async def search(
        self,
        query: str,
        user_id: str,
        limit: int = 10
    ) -> List[ImageAsset]:
        """
        自然语言搜索图片
        
        例子：
        - "找张适合博客封面的产品图"
        - "办公室的照片，要有人，要活泼"
        - "上周活动的团队照"
        """
        
        # 1. 用Claude理解搜索意图
        intent = await self.understand_query(query)
        
        # 2. 转换为搜索条件
        filters = self.build_filters(intent)
        
        # 3. 搜索数据库
        results = await self.db.search_images(
            user_id=user_id,
            filters=filters,
            limit=limit
        )
        
        # 4. 相关性排序
        ranked = self.rank_by_relevance(results, intent)
        
        return ranked
    
    async def understand_query(self, query: str) -> SearchIntent:
        """
        理解搜索意图
        """
        
        prompt = f"""
        用户在搜索图片，理解他的需求。
        
        搜索词："{query}"
        
        返回JSON：
        {{
          "content_type": ["product", "people", "office", "event"],
          "scene": ["cover", "illustration", "social"],
          "style": ["professional", "casual", "vibrant"],
          "requirements": ["has_whitespace", "centered"],
          "time_range": "last_week" or null,
          "reasoning": "为什么这么理解"
        }}
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=500,
            messages=[{"role": "user", "content": prompt}]
        )
        
        intent = SearchIntent.from_json(response.content[0].text)
        
        return intent
    
    async def suggest_for_context(
        self,
        document_content: str,
        position: str,  # "cover" or "illustration"
        user_id: str
    ) -> List[ImageAsset]:
        """
        根据文档内容推荐图片
        
        用于博文生成时自动配图
        """
        
        # 1. 分析文档内容
        doc_analysis = await self.analyze_document(document_content)
        
        # 2. 生成搜索query
        query = self.generate_search_query(doc_analysis, position)
        
        # 3. 搜索
        results = await self.search(query, user_id)
        
        return results
```

#### 1.3 与博文生成器集成

```python
class ImageSelector:
    """
    配图选择器
    
    策略：
    1. 优先用素材库的图（省钱+有公司特色）
    2. 素材库没有合适的，才用AI生成
    3. 记录哪些图效果好，下次优先推荐
    """
    
    async def select_image(
        self,
        context: str,
        position: str,
        user_id: str
    ) -> ImageSource:
        """
        为某个位置选择图片
        
        返回：
        - 如果素材库有合适的：返回素材库图片
        - 如果没有：返回AI生成的prompt
        """
        
        # 1. 先在素材库找
        library_images = await self.image_search.suggest_for_context(
            document_content=context,
            position=position,
            user_id=user_id
        )
        
        # 2. 评估相关性
        if library_images:
            best_match = library_images[0]
            confidence = self.calculate_confidence(best_match, context)
            
            if confidence > 0.7:  # 足够相关
                return ImageSource(
                    type="library",
                    image=best_match,
                    confidence=confidence
                )
        
        # 3. 素材库没有，生成prompt
        generation_prompt = await self.generate_image_prompt(context, position)
        
        return ImageSource(
            type="generated",
            prompt=generation_prompt
        )
```

### 数据模型

```python
@dataclass
class ImageAsset:
    """图片素材"""
    id: str
    user_id: str
    url: str
    tags: List[str]  # AI生成的标签
    metadata: ImageAnalysis  # AI分析的详细信息
    upload_source: str  # 上传来源
    usage_count: int  # 使用次数
    last_used: Optional[datetime]
    uploaded_at: datetime

@dataclass
class ImageAnalysis:
    """图片分析结果"""
    has_people: bool
    people_count: int
    has_product: bool
    has_text: bool
    scene_type: str  # indoor/outdoor/product
    composition: str  # centered/rule_of_thirds/etc
    color_palette: str  # corporate/vibrant/pastel/monochrome
    dominant_colors: List[str]
    has_whitespace: bool
    suitable_for: List[str]  # cover/illustration/social
    mood: str  # professional/casual/energetic/calm
    description: str  # 详细描述

@dataclass
class SearchIntent:
    """搜索意图"""
    content_type: List[str]
    scene: List[str]
    style: List[str]
    requirements: List[str]
    time_range: Optional[str]
    reasoning: str
```

---

## Module 2: 公众号学习器

### 核心价值

**问题：**
- 想学习同行/竞品的内容风格
- 想知道行业热点话题
- 想了解什么样的文章受欢迎

**但是：**
- 没时间每天看
- 看了也记不住规律
- 不知道具体学什么

**解决方案：**
- 每天自动抓取指定公众号
- AI分析内容、风格、配图策略
- 提取可学习的套路
- CEO说"照着36氪的风格写"，系统就懂

### 功能设计

#### 2.1 公众号监控

```python
class WeChatMonitor:
    """
    公众号监控器
    """
    
    async def add_subscription(
        self,
        account_name: str,
        user_id: str,
        reason: str  # 为什么要关注这个号
    ) -> Subscription:
        """
        添加要监控的公众号
        
        例子：
        - 36氪：学习科技新闻写作风格
        - YC：学习创业故事叙事
        - 即刻：学习话题选择
        """
        
        subscription = Subscription(
            id=generate_id(),
            account_name=account_name,
            user_id=user_id,
            reason=reason,
            status="active",
            created_at=datetime.now()
        )
        
        await self.db.save_subscription(subscription)
        
        return subscription
    
    async def crawl_articles(self):
        """
        定时任务：抓取新文章
        
        每天运行一次
        """
        
        subscriptions = await self.db.get_active_subscriptions()
        
        for sub in subscriptions:
            try:
                # 抓取最新文章
                articles = await self.crawler.fetch_recent_articles(
                    account_name=sub.account_name,
                    days=1
                )
                
                # 保存
                for article in articles:
                    await self.save_article(article, sub.id)
                
                # 触发分析
                for article in articles:
                    await self.analyze_article(article.id)
                    
            except Exception as e:
                logger.error(f"Failed to crawl {sub.account_name}: {e}")
```

#### 2.2 内容分析

```python
class ArticleAnalyzer:
    """
    文章分析器
    
    分析维度：
    1. 内容特点（选题、结构、论证）
    2. 写作风格（语气、用词、节奏）
    3. 配图策略（数量、类型、位置）
    4. 互动数据（阅读、点赞、留言）
    """
    
    async def analyze_article(
        self,
        article_id: str
    ) -> ArticleAnalysis:
        """
        深度分析一篇文章
        """
        
        article = await self.db.get_article(article_id)
        
        # 1. 内容分析
        content_analysis = await self.analyze_content(article)
        
        # 2. 风格分析
        style_analysis = await self.analyze_style(article)
        
        # 3. 配图分析
        image_analysis = await self.analyze_images(article)
        
        # 4. 互动数据（如果有）
        engagement = article.engagement_stats
        
        # 5. 综合评估
        overall = await self.synthesize_analysis(
            content_analysis,
            style_analysis,
            image_analysis,
            engagement
        )
        
        # 6. 保存
        analysis = ArticleAnalysis(
            article_id=article_id,
            content=content_analysis,
            style=style_analysis,
            images=image_analysis,
            engagement=engagement,
            overall=overall,
            analyzed_at=datetime.now()
        )
        
        await self.db.save_analysis(analysis)
        
        return analysis
    
    async def analyze_content(
        self,
        article: Article
    ) -> ContentAnalysis:
        """
        分析内容特点
        """
        
        prompt = f"""
        分析这篇文章的内容特点。
        
        标题：{article.title}
        正文：{article.content}
        
        分析以下方面：
        1. 选题特点：为什么选这个话题？切入角度是什么？
        2. 结构：怎么组织的？（总分总、递进、对比）
        3. 论证方式：用了什么手法？（案例、数据、引用）
        4. 核心观点：想表达什么？
        5. 目标读者：写给谁看的？
        
        返回JSON格式的分析结果。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        analysis = ContentAnalysis.from_json(response.content[0].text)
        
        return analysis
    
    async def analyze_style(
        self,
        article: Article
    ) -> StyleAnalysis:
        """
        分析写作风格
        """
        
        prompt = f"""
        分析这篇文章的写作风格。
        
        正文：{article.content}
        
        分析：
        1. 语气：正式/轻松/幽默/严肃
        2. 用词特点：专业术语多/口语化/网络用语
        3. 句式：长句/短句/混合
        4. 节奏：快/慢/张弛有度
        5. 修辞手法：比喻、排比、反问等
        6. 可学习的特点：这个风格最值得学习的是什么
        
        返回JSON。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        analysis = StyleAnalysis.from_json(response.content[0].text)
        
        return analysis
    
    async def analyze_images(
        self,
        article: Article
    ) -> ImageStrategyAnalysis:
        """
        分析配图策略
        """
        
        if not article.images:
            return ImageStrategyAnalysis(has_images=False)
        
        # 1. 统计基本信息
        image_count = len(article.images)
        positions = [img.position for img in article.images]
        
        # 2. 用Vision分析每张图
        image_analyses = []
        for img in article.images:
            analysis = await self.analyze_single_image(img.url)
            image_analyses.append(analysis)
        
        # 3. 总结配图策略
        strategy = await self.summarize_image_strategy(
            image_count,
            positions,
            image_analyses,
            article.content
        )
        
        return strategy
```

#### 2.3 套路提取

```python
class PatternExtractor:
    """
    套路提取器
    
    从多篇文章中提取共同的规律
    """
    
    async def extract_patterns(
        self,
        account_name: str,
        min_articles: int = 10
    ) -> AccountPatterns:
        """
        提取某个账号的写作套路
        """
        
        # 1. 获取最近的文章分析
        analyses = await self.db.get_recent_analyses(
            account_name=account_name,
            limit=min_articles
        )
        
        if len(analyses) < min_articles:
            raise NotEnoughDataError()
        
        # 2. 用Claude总结共同规律
        prompt = f"""
        分析这个公众号的写作套路。
        
        账号：{account_name}
        已分析文章数：{len(analyses)}
        
        各篇文章的分析结果：
        {self.format_analyses(analyses)}
        
        请总结：
        1. 选题规律：喜欢写什么话题？什么角度？
        2. 结构模板：常用的文章结构是什么？
        3. 风格特点：一贯的写作风格？
        4. 配图习惯：配图的数量、类型、位置？
        5. 可复用的套路：如果要模仿，应该学习什么？
        
        返回JSON格式的总结。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=3000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        patterns = AccountPatterns.from_json(response.content[0].text)
        
        # 3. 保存
        await self.db.save_patterns(account_name, patterns)
        
        return patterns
```

#### 2.4 风格应用

```python
class StyleApplicator:
    """
    风格应用器
    
    将学到的风格应用到新内容
    """
    
    async def apply_style(
        self,
        content: str,
        reference_account: str
    ) -> str:
        """
        用某个账号的风格改写内容
        
        用户说：
        "照着36氪的风格写"
        """
        
        # 1. 获取该账号的套路
        patterns = await self.db.get_patterns(reference_account)
        
        if not patterns:
            raise StyleNotLearnedError(
                f"还没有学习{reference_account}的风格，请先添加监控"
            )
        
        # 2. 应用风格改写
        prompt = f"""
        将以下内容改写成{reference_account}的风格。
        
        原始内容：
        {content}
        
        {reference_account}的风格特点：
        {patterns.to_prompt()}
        
        要求：
        - 保留原始内容的核心信息
        - 采用{reference_account}的语气和节奏
        - 使用类似的结构
        - 如果有配图建议，也按照他们的习惯
        
        返回改写后的内容。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        rewritten = response.content[0].text
        
        return rewritten
```

### 数据模型

```python
@dataclass
class Subscription:
    """订阅的公众号"""
    id: str
    account_name: str
    user_id: str
    reason: str  # 为什么关注
    status: str  # active/paused
    created_at: datetime

@dataclass
class Article:
    """抓取的文章"""
    id: str
    subscription_id: str
    title: str
    author: str
    content: str
    images: List[ArticleImage]
    published_at: datetime
    crawled_at: datetime
    engagement_stats: Optional[EngagementStats]

@dataclass
class ContentAnalysis:
    """内容分析"""
    topic_type: str
    angle: str
    structure: str
    argumentation: List[str]
    core_message: str
    target_audience: str

@dataclass
class StyleAnalysis:
    """风格分析"""
    tone: str
    vocabulary: str
    sentence_structure: str
    pace: str
    rhetorical_devices: List[str]
    learnable_features: List[str]

@dataclass
class AccountPatterns:
    """账号套路总结"""
    account_name: str
    topic_patterns: str
    structure_template: str
    style_characteristics: str
    image_strategy: str
    reusable_tactics: List[str]
    updated_at: datetime
```

---

## Module 3: CEO日记 + 智能博文建议

### 核心价值

**问题：**
- CEO有很多想法和进展，但没时间整理成文章
- 不知道什么时候该写什么内容
- 不知道怎么把内部进展转化为对外博文

**解决方案：**
- CEO随手记录（像日记，不用完整）
- AI定期分析：
  - 哪些进展值得对外讲
  - 现在市场关心什么话题
  - 怎么把内部语言转化为对外内容
- CEO一句话："写篇关于XX的"
- AI自动从日记里找素材+组织成文章

### 功能设计

#### 3.1 CEO日记本

```python
class CEODiary:
    """
    CEO日记系统
    
    特点：
    - 随手记，不用完整
    - 支持语音转文字
    - 自动分类整理
    - 智能提取可对外的内容
    """
    
    async def add_entry(
        self,
        user_id: str,
        content: str,
        entry_type: str = "progress",  # progress/idea/milestone
        tags: List[str] = []
    ) -> DiaryEntry:
        """
        添加一条日记
        
        CEO随手记录：
        - "今天见了投资人，对我们的AI agent方向很感兴趣"
        - "新版本上线，用户留存提升了15%"
        - "团队讨论了Q2的产品规划"
        """
        
        entry = DiaryEntry(
            id=generate_id(),
            user_id=user_id,
            content=content,
            entry_type=entry_type,
            tags=tags,
            created_at=datetime.now()
        )
        
        # 自动分析和增强
        entry = await self.enrich_entry(entry)
        
        # 保存
        await self.db.save_entry(entry)
        
        # 触发智能分析
        await self.analyze_recent_entries(user_id)
        
        return entry
    
    async def enrich_entry(
        self,
        entry: DiaryEntry
    ) -> DiaryEntry:
        """
        自动增强日记条目
        
        提取：
        - 关键信息（数据、milestone、合作）
        - 情感色调
        - 是否适合对外
        - 相关话题
        """
        
        prompt = f"""
        分析这条CEO日记。
        
        内容：{entry.content}
        
        提取：
        1. 关键信息：这条记录的核心是什么？
        2. 类型：进展/想法/milestone/日常
        3. 是否适合对外：能否写成博文？为什么？
        4. 相关话题：这条记录涉及什么话题？
        5. 潜在价值：如果要写文章，能怎么用？
        
        返回JSON。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        analysis = EntryAnalysis.from_json(response.content[0].text)
        
        entry.analysis = analysis
        entry.auto_tags = analysis.related_topics
        
        return entry
    
    async def voice_to_entry(
        self,
        user_id: str,
        audio_file: UploadFile
    ) -> DiaryEntry:
        """
        语音转日记
        
        CEO可以直接说，不用打字
        """
        
        # 1. 语音转文字
        transcript = await self.transcribe_audio(audio_file)
        
        # 2. 添加日记
        entry = await self.add_entry(
            user_id=user_id,
            content=transcript,
            entry_type="voice"
        )
        
        return entry
```

#### 3.2 智能内容建议

```python
class ContentSuggestionEngine:
    """
    内容建议引擎
    
    定期分析：
    - CEO最近的进展
    - 市场热点话题
    - 建议写什么内容
    """
    
    async def generate_suggestions(
        self,
        user_id: str
    ) -> List[ContentSuggestion]:
        """
        生成内容建议
        
        每周一运行，给CEO发建议
        """
        
        # 1. 获取最近的日记（2周内）
        recent_entries = await self.db.get_recent_entries(
            user_id=user_id,
            days=14
        )
        
        # 2. 分析市场热点
        market_trends = await self.analyze_market_trends()
        
        # 3. 生成建议
        suggestions = await self.generate_content_ideas(
            entries=recent_entries,
            trends=market_trends
        )
        
        return suggestions
    
    async def analyze_market_trends(self) -> MarketTrends:
        """
        分析当前市场热点
        
        来源：
        - 监控的公众号
        - 行业新闻
        - 社交媒体话题
        """
        
        # 1. 获取最近的文章
        recent_articles = await self.db.get_recent_articles(days=7)
        
        # 2. 用Claude总结热点话题
        prompt = f"""
        分析最近的科技/创业圈热点话题。
        
        数据来源：
        - 36氪、虎嗅等媒体最近的文章
        - 标题和摘要：
        {self.format_articles(recent_articles)}
        
        请总结：
        1. 当前的3-5个热点话题
        2. 每个话题的关注度
        3. 哪些话题适合初创公司CEO写文章
        4. 建议的切入角度
        
        返回JSON。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        trends = MarketTrends.from_json(response.content[0].text)
        
        return trends
    
    async def generate_content_ideas(
        self,
        entries: List[DiaryEntry],
        trends: MarketTrends
    ) -> List[ContentSuggestion]:
        """
        生成具体的内容建议
        """
        
        prompt = f"""
        基于CEO的日记和市场热点，建议内容方向。
        
        CEO最近的进展：
        {self.format_entries(entries)}
        
        市场热点：
        {trends.to_text()}
        
        请生成3-5个内容建议：
        1. 话题：写什么
        2. 角度：怎么写
        3. 理由：为什么现在写这个
        4. 素材来源：从哪些日记条目中提取
        5. 预期效果：这篇文章的目标
        
        返回JSON数组。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=3000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        suggestions = [
            ContentSuggestion.from_dict(s)
            for s in json.loads(response.content[0].text)
        ]
        
        return suggestions
```

#### 3.3 一句话生成博文

```python
class DiaryToBlogGenerator:
    """
    从日记生成博文
    
    CEO说："写篇关于我们融资的"
    系统自动：
    1. 从日记里找相关的条目
    2. 结合市场趋势
    3. 参考学到的风格
    4. 生成完整博文草稿
    """
    
    async def generate_from_command(
        self,
        user_id: str,
        command: str,
        style_reference: Optional[str] = None
    ) -> BlogDraft:
        """
        一句话生成博文
        
        例子：
        - "写篇关于融资的，风格参考YC"
        - "写个产品更新，活泼点"
        - "总结下Q1的进展"
        """
        
        # 1. 理解指令
        intent = await self.understand_command(command)
        
        # 2. 从日记里找相关素材
        relevant_entries = await self.find_relevant_entries(
            user_id=user_id,
            topic=intent.topic,
            time_range=intent.time_range
        )
        
        # 3. 生成文章大纲
        outline = await self.generate_outline(
            topic=intent.topic,
            entries=relevant_entries,
            style_reference=style_reference
        )
        
        # 4. 生成正文
        content = await self.generate_content(
            outline=outline,
            entries=relevant_entries,
            style_reference=style_reference
        )
        
        # 5. 配图
        images = await self.suggest_images(
            content=content,
            user_id=user_id
        )
        
        # 6. 组装
        draft = BlogDraft(
            title=outline.title,
            content=content,
            images=images,
            source_entries=[e.id for e in relevant_entries],
            generated_at=datetime.now()
        )
        
        return draft
    
    async def understand_command(
        self,
        command: str
    ) -> GenerationIntent:
        """
        理解CEO的指令
        """
        
        prompt = f"""
        理解CEO的内容生成指令。
        
        指令："{command}"
        
        提取：
        1. 话题：想写什么？
        2. 风格要求：正式/轻松/活泼/专业？
        3. 风格参考：提到了哪个账号的风格？
        4. 时间范围：最近/Q1/本月？
        5. 特殊要求：其他要求？
        
        返回JSON。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=500,
            messages=[{"role": "user", "content": prompt}]
        )
        
        intent = GenerationIntent.from_json(response.content[0].text)
        
        return intent
    
    async def find_relevant_entries(
        self,
        user_id: str,
        topic: str,
        time_range: Optional[str]
    ) -> List[DiaryEntry]:
        """
        找相关的日记条目
        """
        
        # 1. 获取候选条目
        if time_range:
            candidates = await self.db.get_entries_in_range(
                user_id=user_id,
                time_range=time_range
            )
        else:
            candidates = await self.db.get_recent_entries(
                user_id=user_id,
                days=90
            )
        
        # 2. 用Claude评估相关性
        relevant = []
        for entry in candidates:
            relevance = await self.calculate_relevance(entry, topic)
            if relevance > 0.6:
                relevant.append(entry)
        
        # 3. 按相关性排序
        relevant.sort(key=lambda e: e.relevance_score, reverse=True)
        
        return relevant[:10]  # 最多用10条
    
    async def generate_content(
        self,
        outline: Outline,
        entries: List[DiaryEntry],
        style_reference: Optional[str]
    ) -> str:
        """
        生成正文
        """
        
        # 准备上下文
        context = {
            "outline": outline,
            "materials": [e.content for e in entries],
            "style": None
        }
        
        # 如果有风格参考，加载风格
        if style_reference:
            patterns = await self.db.get_patterns(style_reference)
            context["style"] = patterns
        
        # 生成
        prompt = self.build_generation_prompt(context)
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=4000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        content = response.content[0].text
        
        return content
```

### 数据模型

```python
@dataclass
class DiaryEntry:
    """日记条目"""
    id: str
    user_id: str
    content: str
    entry_type: str  # progress/idea/milestone/voice
    tags: List[str]
    auto_tags: List[str]  # AI自动打的标签
    analysis: Optional[EntryAnalysis]
    created_at: datetime

@dataclass
class EntryAnalysis:
    """日记分析"""
    key_info: str
    entry_category: str
    suitable_for_public: bool
    reasoning: str
    related_topics: List[str]
    potential_value: str

@dataclass
class ContentSuggestion:
    """内容建议"""
    topic: str
    angle: str
    reasoning: str
    source_entries: List[str]
    expected_outcome: str
    priority: int  # 1-5
    suggested_at: datetime

@dataclass
class BlogDraft:
    """博文草稿"""
    title: str
    content: str
    images: List[ImageSource]
    source_entries: List[str]  # 来自哪些日记
    style_reference: Optional[str]
    generated_at: datetime
```

---

## Module 4: 智能博文生成器（整合所有模块）

### 核心流程

```python
class SmartBlogGenerator:
    """
    智能博文生成器
    
    整合所有模块：
    - 图片素材库
    - 公众号学习
    - CEO日记
    - 原有的配图功能
    """
    
    def __init__(self):
        self.image_library = ImageLibrary()
        self.wechat_monitor = WeChatMonitor()
        self.ceo_diary = CEODiary()
        self.diary_to_blog = DiaryToBlogGenerator()
        self.document_processor = DocumentProcessor()  # 原有的
    
    async def generate_blog(
        self,
        request: BlogGenerationRequest
    ) -> GeneratedBlog:
        """
        生成博文的统一入口
        
        支持多种模式：
        1. 从日记生成：CEO说一句话
        2. 从草稿生成：已有文字，需要配图
        3. 从建议生成：选择系统建议的话题
        """
        
        if request.mode == "from_diary":
            return await self.generate_from_diary(request)
        elif request.mode == "from_draft":
            return await self.generate_from_draft(request)
        elif request.mode == "from_suggestion":
            return await self.generate_from_suggestion(request)
    
    async def generate_from_diary(
        self,
        request: BlogGenerationRequest
    ) -> GeneratedBlog:
        """
        从日记生成
        
        完整流程：
        1. 理解CEO指令
        2. 找相关日记
        3. 生成文章草稿
        4. 智能配图（优先素材库）
        5. 应用风格（如果指定）
        6. 返回完整博文
        """
        
        # 1. 生成草稿
        draft = await self.diary_to_blog.generate_from_command(
            user_id=request.user_id,
            command=request.command,
            style_reference=request.style_reference
        )
        
        # 2. 配图
        final_images = []
        for img_suggestion in draft.images:
            # 优先用素材库
            if img_suggestion.type == "library":
                final_images.append(img_suggestion.image)
            else:
                # 生成新图片
                generated = await self.generate_image(
                    img_suggestion.prompt
                )
                final_images.append(generated)
        
        # 3. 组装最终博文
        blog = await self.assemble_blog(
            title=draft.title,
            content=draft.content,
            images=final_images
        )
        
        # 4. 保存元数据
        blog.metadata = {
            "source": "diary",
            "source_entries": draft.source_entries,
            "style_reference": request.style_reference,
            "command": request.command
        }
        
        return blog
    
    async def generate_from_draft(
        self,
        request: BlogGenerationRequest
    ) -> GeneratedBlog:
        """
        从草稿生成
        
        用户已经写了文字，只需要配图
        这是原有的功能
        """
        
        # 使用原有的文档处理器
        processed = await self.document_processor.process(
            content=request.content,
            mode=request.platform_mode
        )
        
        return processed
```

---

## 系统集成架构

### 数据流

```
CEO的一天：

早上9点：
CEO: "昨天见了投资人，他们对AI agent方向很感兴趣"
→ 存入日记
→ AI自动分析：这是个好素材，适合写融资相关的文章

上午11点：
系统提醒: "本周热点：AI agent创业，建议写篇关于你们方向的文章"

下午3点：
CEO: "写篇关于我们产品方向的，风格学YC那种，用上周的团队照"
→ 系统开始工作：
   1. 从日记里找：投资人反馈、产品进展、市场判断
   2. 应用YC的风格（之前学到的）
   3. 从素材库找团队照
   4. 生成草稿

→ 30秒后返回完整博文

CEO: 看了一眼，改了两句话，发布
总耗时：2分钟
```

### 模块交互

```
┌──────────────────────────────────────────────────────────┐
│                     用户请求入口                          │
│              "写篇关于XX的，风格学YC"                     │
└──────────────────────────────────────────────────────────┘
                          ↓
┌──────────────────────────────────────────────────────────┐
│                   请求解析器                              │
│   • 理解意图                                              │
│   • 提取参数（话题、风格、素材要求）                      │
└──────────────────────────────────────────────────────────┘
                          ↓
                ┌─────────┴─────────┐
                ↓                   ↓
┌─────────────────────┐   ┌─────────────────────┐
│   CEO日记模块       │   │  公众号学习模块      │
│                     │   │                     │
│ • 找相关进展        │   │ • 加载YC风格        │
│ • 提取素材          │   │ • 应用到生成        │
└─────────────────────┘   └─────────────────────┘
          ↓                         ↓
          └──────────┬──────────────┘
                     ↓
          ┌─────────────────────┐
          │   内容生成引擎       │
          │                     │
          │ • 生成大纲          │
          │ • 生成正文          │
          │ • 决定配图位置      │
          └─────────────────────┘
                     ↓
          ┌─────────────────────┐
          │   图片选择器         │
          │                     │
          │ 1. 查素材库         │
          │ 2. 如果没有→生成    │
          └─────────────────────┘
                     ↓
          ┌─────────────────────┐
          │   最终组装          │
          │                     │
          │ • 插入图片          │
          │ • 排版优化          │
          │ • 生成预览          │
          └─────────────────────┘
                     ↓
               完整博文
```

---

## 开发路线图（完整版）

### Phase 1: 基础配图工具（1周）✅
**已完成设计**

### Phase 2: 图片素材库（1周）

**任务：**
- [ ] 图片上传和CDN存储
- [ ] Claude Vision分析图片
- [ ] 自动打标签
- [ ] 简单的标签搜索
- [ ] 与配图工具集成（优先用素材库）

**验收：**
- 能上传图片
- 能搜索图片
- 配图时会优先用素材库的图

### Phase 3: 公众号监控（1周）

**任务：**
- [ ] 公众号文章抓取（先手动）
- [ ] 内容分析（用Claude）
- [ ] 风格分析
- [ ] 套路总结
- [ ] 风格应用到生成

**验收：**
- 能监控2-3个公众号
- 能说"照着36氪的风格写"
- 生成的内容确实有那个风格

### Phase 4: CEO日记（1周）

**任务：**
- [ ] 日记本（文字输入）
- [ ] 日记分析（自动打标签）
- [ ] 智能建议（分析该写什么）
- [ ] 从日记生成博文

**验收：**
- CEO能记日记
- 系统能建议写什么
- 能说一句话生成博文

### Phase 5: 整合优化（1周）

**任务：**
- [ ] 打通所有模块
- [ ] 端到端测试
- [ ] 性能优化
- [ ] UI/UX优化

**验收：**
- 完整的workflow能跑通
- 从日记到博文一气呵成
- CEO满意

### Phase 6: 持续优化（持续）

**基于反馈：**
- 优化生成质量
- 增加更多平台模式
- 改进风格学习
- ...

---

## 技术栈（完整版）

### 后端
- **框架**: FastAPI
- **AI服务**: 
  - Claude Sonnet 4（文本分析、生成）
  - Claude Vision（图片分析）
  - DALL-E 3（图片生成）
  - Whisper（语音转文字，如果要做）
- **数据库**: PostgreSQL（数据量大了）
- **缓存**: Redis
- **队列**: Celery（异步任务）
- **爬虫**: BeautifulSoup + Selenium（公众号抓取）

### 前端
- **框架**: 还是简单点，Vue.js或React
- **UI**: Ant Design或Tailwind
- **图表**: ECharts（可能需要可视化分析）

### 基础设施
- **部署**: 
  - 后端：Railway/Render（比Vercel更适合长期运行任务）
  - 前端：Vercel
- **存储**: Cloudflare R2（图片）
- **监控**: Sentry

---

## 成本估算（完整版）

### 月度API成本（假设100个活跃用户）

**每用户每月：**
- CEO日记：10条 × $0.02（分析）= $0.2
- 公众号监控：抓取20篇 × $0.05（分析）= $1
- 博文生成：4次 × $0.3（生成+配图）= $1.2
- 图片分析：5张 × $0.02 = $0.1

**单用户成本：~$2.5/月**
**100用户成本：$250/月**

### 基础设施
- 服务器：$50/月
- 数据库：$20/月
- CDN：$10/月

**总成本：~$330/月（100用户）**

### 收费策略

**个人版：$29/月**
- 适合个人博主
- 每月生成20篇
- 素材库500张图

**团队版：$99/月**
- 适合小团队（<5人）
- 无限生成
- 素材库无限
- 团队协作

**企业版：定制**
- 私有部署
- 专属训练
- API访问

---

## 关键创新点

这个完整系统的核心创新：

1. **从"工具"到"助手"**
   - 不是被动的工具
   - 而是主动的助手（建议、提醒、学习）

2. **闭环学习**
   - 学习同行（公众号）
   - 学习自己（CEO日记）
   - 学习用户（反馈）

3. **上下文理解**
   - 知道CEO的进展
   - 知道市场的热点
   - 知道什么时候该写什么

4. **素材复用**
   - 公司的图片库
   - CEO的历史记录
   - 学到的风格模板

---

## 风险

### 1. 复杂度风险
**问题**：功能太多，开发周期长

**应对**：
- 分阶段，每个模块独立可用
- 先做MVP验证需求
- 不急于求成

### 2. 爬虫风险
**问题**：公众号可能封IP

**应对**：
- 早期手动+半自动
- 使用代理
- 控制频率
- 考虑付费数据源

### 3. 质量风险
**问题**：AI生成质量不稳定

**应对**：
- 强调是"草稿"不是"成品"
- CEO最后把关
- 持续优化prompt

---

## 总结

这是一个**野心勃勃但可行**的系统。

**核心价值：**
> 让CEO从"内容生产者"变成"内容指挥者"

**开发策略：**
1. 先做基础配图工具（已设计）
2. 逐步加addon（每个1周）
3. 每个模块独立可用
4. 最后整合成完整系统

**时间表：**
- Month 1: 基础工具 + 图片库
- Month 2: 公众号学习 + CEO日记
- Month 3: 整合优化

**3个月后，你会有一个完整的内容生产中台。**

---

*完整设计文档 v1.0*
*创建日期: 2026-01-15*
*产品定位：智能内容生产中台*

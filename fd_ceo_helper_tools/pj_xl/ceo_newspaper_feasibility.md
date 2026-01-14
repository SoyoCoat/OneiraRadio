# CEO Helper: Newspaper - 可行性分析与技术设计

## 项目概述

### 产品定位

**一句话描述：**
> 每天早上，CEO打开手机，看到5篇最相关的行业动态 + 1份AI总结，了解同行在干什么。

**核心价值：**
- 替代COO每天花1-2小时看行业新闻
- 自动筛选最相关的内容
- AI总结关键信息
- CEO只需要5分钟就能了解行业动态

### 目标用户

**主要用户：**
- 初创公司CEO（50人以下）
- 没有专职的行研/战略岗位
- 需要关注竞品和行业动态
- 时间很宝贵

**典型场景：**
```
早上8点，CEO起床，打开邮件：

📰 今日行业动态 (2026-01-15)

【今日总结】
本周AI agent赛道融资活跃，3家公司获得新一轮融资。
其中Adept获得$350M，估值突破$1B。
主要玩家都在押注"AI工作流自动化"方向。
你们的竞争对手AutoAgent发布了新的API...

【今日必读 TOP 5】
1. Adept完成$350M融资，估值$1B+
   来源：36氪 | 相关度：95% | 阅读时长：3分钟
   
2. AutoAgent发布新API，支持企业级部署
   来源：公司公众号 | 相关度：92% | 阅读时长：5分钟
   
3. Y Combinator 2025冬季班：AI创业公司占比40%
   来源：YC官方 | 相关度：85% | 阅读时长：4分钟

...

CEO花5分钟看完，知道了行业发生了什么。
```

---

## 可行性分析

### 1. 需求可行性

#### 真实痛点

**CEO的痛点：**
- ✅ 需要了解同行动态（融资、产品、战略）
- ✅ 需要知道行业趋势（热点话题、技术方向）
- ✅ 没时间自己看（每天看新闻要1-2小时）
- ✅ 不知道哪些重要（信息过载）

**现有解决方案的问题：**
- 36氪/虎嗅：内容太泛，90%不相关
- 微信公众号：需要自己筛选，太多
- COO人工筛选：成本高，不稳定
- RSS订阅：还是要自己看，没有总结

**我们的方案优势：**
- ✅ 自动抓取（不需要人工）
- ✅ AI筛选（只推送最相关的）
- ✅ AI总结（快速了解关键信息）
- ✅ 每天5分钟（节省时间）

#### 市场验证

**类似产品：**
- **Feedly + AI总结**: 存在，但不够垂直
- **Brief（新闻聚合）**: 存在，但是通用的，不针对CEO
- **企业级竞品分析工具**: 太贵，太复杂

**我们的差异化：**
- 专门针对初创公司CEO
- 聚焦"同行在干什么"
- 简单、便宜、好用

#### 用户验证方法

**Week 1-2: 手动验证**
1. 找3-5个CEO朋友
2. 每天手动给他们发5篇文章 + 总结
3. 问他们：有用吗？愿意付费吗？

**如果反馈好 → 值得做**
**如果反馈一般 → 调整或放弃**

---

### 2. 技术可行性

#### 核心技术挑战

**挑战1: 大批量抓取公众号文章**

**难点：**
- 公众号没有官方API
- 反爬虫机制
- 需要大量抓取（每天几百篇）

**解决方案：**

**方案A: 第三方服务（推荐）**
- 使用现成的公众号数据服务
- 例如：新榜、清博、微小宝
- 优点：稳定、合规、快速
- 缺点：要付费（但不贵）
- **成本：~$100-300/月**

**方案B: 自己爬（不推荐，除非要省钱）**
- 技术路线：
  - 搜狗微信搜索 + Selenium
  - 或者通过第三方爬虫工具
- 优点：省钱
- 缺点：不稳定，可能被封，需要维护
- **风险：高**

**推荐：用方案A（第三方服务）**
- 早期稳定性更重要
- 省下的开发时间更值钱
- 合规性更好

**挑战2: AI相关性判断**

**难点：**
- 如何判断一篇文章是否与CEO相关？
- 如何排序？

**解决方案：**

```python
class RelevanceScorer:
    """
    相关性评分器
    
    判断一篇文章对某个CEO的相关度
    """
    
    async def score_article(
        self,
        article: Article,
        ceo_profile: CEOProfile
    ) -> RelevanceScore:
        """
        综合评分：0-100
        
        考虑因素：
        1. 主题相关性（40%）
        2. 竞品相关性（30%）
        3. 时效性（20%）
        4. 重要性（10%）
        """
        
        # 1. 用Claude判断相关性
        prompt = f"""
        判断这篇文章对CEO的相关度。
        
        CEO背景：
        - 公司：{ceo_profile.company_name}
        - 行业：{ceo_profile.industry}
        - 产品：{ceo_profile.product_description}
        - 竞品：{ceo_profile.competitors}
        - 关注点：{ceo_profile.interests}
        
        文章：
        - 标题：{article.title}
        - 摘要：{article.summary}
        - 来源：{article.source}
        
        评分（0-100）并说明理由：
        - 主题相关性：这篇文章的主题和CEO的业务相关吗？
        - 竞品相关性：提到竞品了吗？
        - 实用性：对CEO决策有帮助吗？
        
        返回JSON：
        {{
          "topic_score": 85,
          "competitor_score": 90,
          "actionable_score": 70,
          "overall_score": 82,
          "reasoning": "文章报道了竞品AutoAgent的新产品发布..."
        }}
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=1000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        score = RelevanceScore.from_json(response.content[0].text)
        
        return score
```

**技术可行性：✅ 高**
- Claude很擅长这种判断任务
- 成本可控（每篇$0.02）

**挑战3: 生成每日总结**

**难点：**
- 如何从5篇文章中提炼关键信息？
- 如何写出有用的总结？

**解决方案：**

```python
class DailySummaryGenerator:
    """
    每日总结生成器
    """
    
    async def generate_summary(
        self,
        top_articles: List[Article],
        ceo_profile: CEOProfile
    ) -> DailySummary:
        """
        生成每日总结
        
        总结内容：
        1. 行业大事（融资、并购、新品）
        2. 竞品动态（产品、战略）
        3. 趋势观察（热点话题）
        4. 建议（是否需要行动）
        """
        
        prompt = f"""
        基于今天的文章，为CEO生成每日总结。
        
        CEO背景：{ceo_profile.to_text()}
        
        今日TOP 5文章：
        {self.format_articles(top_articles)}
        
        请生成总结（200-300字）：
        1. 今天最重要的事情是什么？
        2. 对CEO的业务有什么影响？
        3. 竞品有什么新动作？
        4. 需要采取什么行动吗？（可选）
        
        语气：简洁、直接、像COO跟CEO汇报
        不要空话套话，直接说重点。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        summary = DailySummary(
            content=response.content[0].text,
            key_events=self.extract_key_events(response),
            recommendations=self.extract_recommendations(response),
            generated_at=datetime.now()
        )
        
        return summary
```

**技术可行性：✅ 高**

---

### 3. 商业可行性

#### 成本结构

**固定成本（月）：**
- 服务器：$20
- 数据库：$10
- 公众号数据源：$100-300
- 监控工具：$10
- **小计：~$150/月**

**变动成本（每用户/月）：**
- Claude API（相关性判断）：
  - 每天100篇文章 × $0.02 = $2/天 = $60/月
  - 每天1次总结 × $0.05 = $1.5/月
- **小计：~$60/用户/月**

**总成本（10个用户）：**
- 固定：$150
- 变动：10 × $60 = $600
- **总计：$750/月**

**总成本（100个用户）：**
- 固定：$150
- 变动：100 × $60 = $6000
- **总计：$6150/月**

#### 定价策略

**个人版：$99/月**
- 1个用户
- 每天5篇文章 + 总结
- 支持3个竞品监控

**团队版：$299/月**
- 5个用户
- 每天10篇文章 + 总结
- 支持10个竞品监控
- 团队共享

**毛利率：**
- 个人版：($99 - $60) / $99 = 39%
- 团队版：($299 - $300) / $299 = 0%（需要优化）

**问题：成本偏高**

**优化方案：**

1. **批量处理降低成本**
   - 不是每个用户单独跑
   - 而是：
     - 抓取一次所有文章
     - 所有用户共享文章池
     - 只对每个用户做个性化筛选
   - **节省90%的抓取和初步分析成本**

2. **优化后的成本（100用户）：**
   - 抓取：$300/月（固定，所有用户共享）
   - 个性化筛选：100用户 × $20/月 = $2000/月
   - **总计：$2300/月**
   - **每用户成本：$23/月**
   - **毛利率：($99 - $23) / $99 = 77%** ✅

#### 市场规模

**目标市场：**
- 中国初创公司CEO：~50,000人
- 其中愿意为信息付费的：~5%（2,500人）
- 目标获取：1%（250人）

**收入预测（保守）：**
- Year 1: 50个付费用户 × $99 × 12 = $59,400
- Year 2: 150个付费用户 × $99 × 12 = $178,200

**CAC（获客成本）：**
- 早期靠口碑：~$20/用户
- LTV/CAC = ($99 × 12) / $20 = 59x ✅

---

### 4. 竞争分析

#### 现有竞品

**1. Feedly + AI摘要**
- 优点：成熟、稳定
- 缺点：通用型，不针对CEO，不够智能
- 差异化：我们更垂直、更智能

**2. 企业级竞品分析工具（如SimilarWeb）**
- 优点：数据全面
- 缺点：贵（$200-500/月），复杂，学习成本高
- 差异化：我们简单、便宜、聚焦

**3. COO人工筛选**
- 优点：灵活、理解深
- 缺点：贵（人力成本高），不稳定，效率低
- 差异化：我们自动化、24/7、成本低

#### 护城河

**短期（6个月）：**
- 产品体验（简单、好用）
- 早期用户积累
- 口碑传播

**中期（1-2年）：**
- 数据积累（哪些文章真的有用）
- 个性化算法优化
- CEO Profile越来越准

**长期（2年+）：**
- 网络效应（用户越多，数据越好，推荐越准）
- 品牌认知（"CEO看行业动态就用XX"）

---

## 技术架构设计

### 系统架构图

```
┌─────────────────────────────────────────────────────────────┐
│                        数据采集层                            │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │  公众号API   │  │  新闻网站    │  │   RSS Feed   │     │
│  │  (第三方)    │  │  (36氪等)    │  │              │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
│         │                  │                  │             │
│         └──────────────────┴──────────────────┘             │
│                          ↓                                   │
│              ┌─────────────────────┐                        │
│              │  统一数据收集器      │                        │
│              │  (每小时运行)        │                        │
│              └─────────────────────┘                        │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                        数据处理层                            │
│                                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              文章预处理器                            │   │
│  │  • 去重                                              │   │
│  │  • 提取摘要                                          │   │
│  │  • 分类（融资/产品/人事）                           │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              文章池（共享）                          │   │
│  │  今日文章：500篇                                     │   │
│  │  已分类、已去重                                      │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                      个性化推荐层                            │
│                                                              │
│  For each CEO:                                              │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         相关性评分器 (Claude)                        │   │
│  │  • 基于CEO Profile                                   │   │
│  │  • 评估每篇文章的相关度                              │   │
│  │  • 排序：Top 5                                       │   │
│  └─────────────────────────────────────────────────────┘   │
│                          ↓                                   │
│  ┌─────────────────────────────────────────────────────┐   │
│  │         每日总结生成器 (Claude)                      │   │
│  │  • 总结Top 5文章                                     │   │
│  │  • 提取关键信息                                      │   │
│  │  • 生成行动建议                                      │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                          ↓
┌─────────────────────────────────────────────────────────────┐
│                        推送层                                │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐     │
│  │   Email      │  │   微信通知   │  │   Web App    │     │
│  │   (主要)     │  │   (可选)     │  │   (查看)     │     │
│  └──────────────┘  └──────────────┘  └──────────────┘     │
└─────────────────────────────────────────────────────────────┘
```

---

## 核心模块设计

### Module 1: 数据采集器

```python
class ArticleCrawler:
    """
    文章采集器
    
    职责：
    - 从多个数据源采集文章
    - 去重、清洗
    - 存入文章池
    """
    
    def __init__(self):
        self.wechat_api = WeChatAPIClient()  # 第三方API
        self.news_scrapers = [
            Source36Kr(),
            SourceHuxiu(),
            SourceITjuzi()
        ]
        self.db = Database()
    
    async def crawl_daily(self):
        """
        每日抓取任务
        
        运行时间：每小时
        """
        
        # 1. 从各个源抓取
        all_articles = []
        
        # 1.1 公众号文章（主要来源）
        wechat_articles = await self.crawl_wechat_accounts()
        all_articles.extend(wechat_articles)
        
        # 1.2 新闻网站
        news_articles = await self.crawl_news_sites()
        all_articles.extend(news_articles)
        
        # 2. 去重
        unique_articles = self.deduplicate(all_articles)
        
        # 3. 预处理
        processed = await self.preprocess(unique_articles)
        
        # 4. 存入文章池
        await self.save_to_pool(processed)
        
        logger.info(f"Crawled {len(processed)} articles")
    
    async def crawl_wechat_accounts(self) -> List[Article]:
        """
        抓取公众号文章
        
        使用第三方API（推荐）
        """
        
        # 预定义的公众号列表
        accounts = [
            "36氪",
            "虎嗅网",
            "IT桔子",
            "创业邦",
            "极客公园",
            # ... 根据行业添加
        ]
        
        articles = []
        
        for account in accounts:
            try:
                # 调用第三方API
                # 例如：新榜API、清博API
                result = await self.wechat_api.get_recent_articles(
                    account_name=account,
                    days=1  # 最近1天的文章
                )
                
                articles.extend(result)
                
            except Exception as e:
                logger.error(f"Failed to crawl {account}: {e}")
        
        return articles
    
    async def preprocess(
        self,
        articles: List[RawArticle]
    ) -> List[Article]:
        """
        预处理文章
        
        1. 提取摘要（如果没有）
        2. 初步分类（融资/产品/人事/其他）
        3. 提取关键词
        """
        
        processed = []
        
        for raw in articles:
            # 1. 提取摘要
            if not raw.summary:
                summary = await self.extract_summary(raw.content)
            else:
                summary = raw.summary
            
            # 2. 初步分类
            category = await self.classify_article(raw.title, summary)
            
            # 3. 提取关键词
            keywords = await self.extract_keywords(raw.title, summary)
            
            # 4. 构建Article对象
            article = Article(
                id=generate_id(),
                title=raw.title,
                summary=summary,
                content=raw.content,
                source=raw.source,
                url=raw.url,
                category=category,
                keywords=keywords,
                published_at=raw.published_at,
                crawled_at=datetime.now()
            )
            
            processed.append(article)
        
        return processed
    
    def deduplicate(
        self,
        articles: List[RawArticle]
    ) -> List[RawArticle]:
        """
        去重
        
        基于标题相似度
        """
        
        seen_titles = set()
        unique = []
        
        for article in articles:
            # 简单去重：标题完全相同
            title_hash = self.normalize_title(article.title)
            
            if title_hash not in seen_titles:
                seen_titles.add(title_hash)
                unique.append(article)
        
        return unique
    
    async def extract_summary(self, content: str) -> str:
        """
        用Claude提取摘要
        """
        
        # 如果内容太长，只取前1000字
        content_preview = content[:1000]
        
        prompt = f"""
        提取这篇文章的摘要（100字以内）。
        
        文章：
        {content_preview}
        
        摘要应该回答：
        1. 主要讲了什么？
        2. 核心信息是什么？
        
        只返回摘要文本，不要其他内容。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=300,
            messages=[{"role": "user", "content": prompt}]
        )
        
        summary = response.content[0].text.strip()
        
        return summary
    
    async def classify_article(
        self,
        title: str,
        summary: str
    ) -> str:
        """
        分类文章
        
        类别：
        - funding: 融资、投资
        - product: 产品发布、更新
        - personnel: 人事变动
        - strategy: 战略、合作
        - other: 其他
        """
        
        prompt = f"""
        判断这篇文章的类型。
        
        标题：{title}
        摘要：{summary}
        
        类型（只返回一个词）：
        - funding: 如果是融资、投资相关
        - product: 如果是产品发布、更新
        - personnel: 如果是人事变动、高管任命
        - strategy: 如果是战略调整、合作
        - other: 其他
        
        只返回类型词，不要解释。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=50,
            messages=[{"role": "user", "content": prompt}]
        )
        
        category = response.content[0].text.strip().lower()
        
        return category
```

---

### Module 2: CEO Profile管理

```python
class CEOProfileManager:
    """
    CEO Profile管理器
    
    维护每个CEO的背景信息，用于个性化推荐
    """
    
    async def create_profile(
        self,
        user_id: str,
        company_info: CompanyInfo
    ) -> CEOProfile:
        """
        创建CEO Profile
        
        初次注册时填写
        """
        
        profile = CEOProfile(
            user_id=user_id,
            company_name=company_info.name,
            industry=company_info.industry,
            product_description=company_info.product,
            competitors=company_info.competitors,
            interests=company_info.interests,
            created_at=datetime.now()
        )
        
        # 用Claude生成更详细的profile
        enriched = await self.enrich_profile(profile)
        
        await self.db.save_profile(enriched)
        
        return enriched
    
    async def enrich_profile(
        self,
        profile: CEOProfile
    ) -> CEOProfile:
        """
        用Claude丰富profile
        
        生成：
        - 关键词列表
        - 关注的话题
        - 相关的公司列表
        """
        
        prompt = f"""
        基于这个CEO的信息，生成详细的关注点。
        
        公司：{profile.company_name}
        行业：{profile.industry}
        产品：{profile.product_description}
        竞品：{profile.competitors}
        
        生成：
        1. 关键词列表（20个，用于匹配文章）
        2. 应该关注的话题（10个）
        3. 应该关注的公司（除了已知竞品外）
        4. 应该关注的公众号
        
        返回JSON。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        enrichment = json.loads(response.content[0].text)
        
        profile.keywords = enrichment['keywords']
        profile.topics = enrichment['topics']
        profile.related_companies = enrichment['companies']
        profile.recommended_sources = enrichment['sources']
        
        return profile
    
    async def update_from_feedback(
        self,
        user_id: str,
        feedback: UserFeedback
    ):
        """
        根据用户反馈更新profile
        
        如果用户标记某些文章"有用"或"无用"，
        调整profile
        """
        
        profile = await self.db.get_profile(user_id)
        
        # 简单策略：
        # - 用户喜欢的文章 → 提取关键词，加入profile
        # - 用户不喜欢的文章 → 降低相关关键词的权重
        
        if feedback.rating >= 4:
            # 提取这篇文章的关键词
            article_keywords = feedback.article.keywords
            
            # 加入profile（如果不存在）
            for keyword in article_keywords:
                if keyword not in profile.keywords:
                    profile.keywords.append(keyword)
        
        await self.db.update_profile(profile)
```

---

### Module 3: 个性化推荐引擎

```python
class RecommendationEngine:
    """
    个性化推荐引擎
    
    核心算法：
    1. 从文章池中筛选候选文章
    2. 用Claude评估相关性
    3. 排序，取Top 5
    """
    
    async def generate_daily_recommendation(
        self,
        user_id: str
    ) -> DailyRecommendation:
        """
        生成每日推荐
        
        每天早上6点运行
        """
        
        # 1. 获取CEO profile
        profile = await self.db.get_profile(user_id)
        
        # 2. 从文章池获取候选文章（昨天的）
        candidate_articles = await self.db.get_articles(
            date=datetime.now() - timedelta(days=1)
        )
        
        # 3. 初步筛选（基于关键词）
        filtered = self.keyword_filter(candidate_articles, profile)
        
        # 4. AI评分（相关性）
        scored = await self.score_articles(filtered, profile)
        
        # 5. 排序，取Top 5
        top_5 = sorted(scored, key=lambda x: x.score, reverse=True)[:5]
        
        # 6. 生成总结
        summary = await self.generate_summary(top_5, profile)
        
        # 7. 构建推荐结果
        recommendation = DailyRecommendation(
            user_id=user_id,
            date=datetime.now().date(),
            summary=summary,
            articles=top_5,
            generated_at=datetime.now()
        )
        
        # 8. 保存
        await self.db.save_recommendation(recommendation)
        
        return recommendation
    
    def keyword_filter(
        self,
        articles: List[Article],
        profile: CEOProfile
    ) -> List[Article]:
        """
        初步筛选
        
        基于关键词匹配，减少需要AI评分的文章数量
        """
        
        filtered = []
        
        for article in articles:
            # 计算关键词重合度
            overlap = self.calculate_keyword_overlap(
                article.keywords,
                profile.keywords
            )
            
            # 或者标题/摘要包含竞品名称
            mentions_competitor = any(
                comp.lower() in article.title.lower() or
                comp.lower() in article.summary.lower()
                for comp in profile.competitors
            )
            
            # 筛选条件：
            # - 关键词重合度 > 20%
            # - 或者提到竞品
            if overlap > 0.2 or mentions_competitor:
                filtered.append(article)
        
        return filtered
    
    async def score_articles(
        self,
        articles: List[Article],
        profile: CEOProfile
    ) -> List[ScoredArticle]:
        """
        用Claude评估每篇文章的相关性
        """
        
        # 批量评分（提高效率）
        batch_size = 10
        scored = []
        
        for i in range(0, len(articles), batch_size):
            batch = articles[i:i+batch_size]
            
            # 构建批量评分prompt
            prompt = self.build_batch_scoring_prompt(batch, profile)
            
            response = await self.claude.messages.create(
                model="claude-sonnet-4-20250514",
                max_tokens=2000,
                messages=[{"role": "user", "content": prompt}]
            )
            
            # 解析评分结果
            scores = self.parse_batch_scores(response.content[0].text)
            
            # 构建ScoredArticle对象
            for article, score_data in zip(batch, scores):
                scored_article = ScoredArticle(
                    article=article,
                    score=score_data['overall_score'],
                    topic_score=score_data['topic_score'],
                    competitor_score=score_data['competitor_score'],
                    actionable_score=score_data['actionable_score'],
                    reasoning=score_data['reasoning']
                )
                scored.append(scored_article)
        
        return scored
    
    def build_batch_scoring_prompt(
        self,
        articles: List[Article],
        profile: CEOProfile
    ) -> str:
        """
        构建批量评分prompt
        """
        
        articles_text = "\n\n".join([
            f"【文章{i+1}】\n标题：{a.title}\n摘要：{a.summary}\n来源：{a.source}"
            for i, a in enumerate(articles)
        ])
        
        prompt = f"""
        为CEO评估这些文章的相关度。
        
        CEO背景：
        - 公司：{profile.company_name}
        - 行业：{profile.industry}
        - 产品：{profile.product_description}
        - 竞品：{', '.join(profile.competitors)}
        - 关注点：{', '.join(profile.topics)}
        
        文章列表：
        {articles_text}
        
        对每篇文章评分（0-100）：
        - topic_score: 主题相关性
        - competitor_score: 竞品相关性
        - actionable_score: 是否对决策有帮助
        - overall_score: 综合评分
        - reasoning: 简短说明理由
        
        返回JSON数组：
        [
          {{
            "article_id": 1,
            "topic_score": 85,
            "competitor_score": 90,
            "actionable_score": 70,
            "overall_score": 82,
            "reasoning": "报道了竞品的融资，CEO需要了解"
          }},
          ...
        ]
        """
        
        return prompt
    
    async def generate_summary(
        self,
        top_articles: List[ScoredArticle],
        profile: CEOProfile
    ) -> str:
        """
        生成每日总结
        """
        
        articles_text = "\n\n".join([
            f"【{i+1}. {a.article.title}】\n{a.article.summary}"
            for i, a in enumerate(top_articles)
        ])
        
        prompt = f"""
        为CEO生成今日行业动态总结。
        
        CEO背景：{profile.company_name}，{profile.industry}
        
        今日TOP 5文章：
        {articles_text}
        
        生成总结（200-300字）：
        1. 今天行业发生了什么重要的事？
        2. 竞品有什么新动作？
        3. 对CEO的业务可能有什么影响？
        4. 建议关注什么？
        
        语气：
        - 简洁、直接
        - 像COO向CEO汇报
        - 不要空话，直接说重点
        - 如果需要行动，明确指出
        
        只返回总结文本。
        """
        
        response = await self.claude.messages.create(
            model="claude-sonnet-4-20250514",
            max_tokens=2000,
            messages=[{"role": "user", "content": prompt}]
        )
        
        summary = response.content[0].text.strip()
        
        return summary
```

---

### Module 4: 推送系统

```python
class DeliverySystem:
    """
    推送系统
    
    负责把推荐结果发送给CEO
    """
    
    def __init__(self):
        self.email_client = EmailClient()
        self.wechat_client = WeChatNotificationClient()
    
    async def send_daily_digest(
        self,
        user_id: str,
        recommendation: DailyRecommendation
    ):
        """
        发送每日推送
        
        时间：每天早上7点
        """
        
        # 1. 获取用户偏好
        user = await self.db.get_user(user_id)
        
        # 2. 根据偏好选择推送方式
        if user.preferences.email_enabled:
            await self.send_email(user, recommendation)
        
        if user.preferences.wechat_enabled:
            await self.send_wechat(user, recommendation)
    
    async def send_email(
        self,
        user: User,
        recommendation: DailyRecommendation
    ):
        """
        发送邮件
        """
        
        # 生成邮件HTML
        html = self.generate_email_html(recommendation)
        
        # 发送
        await self.email_client.send(
            to=user.email,
            subject=f"📰 今日行业动态 ({recommendation.date})",
            html=html
        )
    
    def generate_email_html(
        self,
        recommendation: DailyRecommendation
    ) -> str:
        """
        生成邮件HTML
        
        设计原则：
        - 简洁、清晰
        - 手机可读
        - 快速扫描
        """
        
        html = f"""
        <!DOCTYPE html>
        <html>
        <head>
            <meta charset="UTF-8">
            <meta name="viewport" content="width=device-width, initial-scale=1.0">
            <style>
                body {{ font-family: -apple-system, sans-serif; padding: 20px; }}
                .summary {{ background: #f5f5f5; padding: 15px; border-radius: 5px; margin-bottom: 20px; }}
                .article {{ margin-bottom: 20px; padding-bottom: 20px; border-bottom: 1px solid #eee; }}
                .title {{ font-size: 16px; font-weight: bold; margin-bottom: 5px; }}
                .meta {{ color: #666; font-size: 14px; margin-bottom: 10px; }}
                .summary-text {{ line-height: 1.6; }}
                .tag {{ display: inline-block; background: #e3f2fd; padding: 2px 8px; border-radius: 3px; font-size: 12px; margin-right: 5px; }}
            </style>
        </head>
        <body>
            <h1>📰 今日行业动态</h1>
            <p style="color: #666;">{recommendation.date.strftime('%Y年%m月%d日')}</p>
            
            <div class="summary">
                <h2 style="margin-top: 0;">今日总结</h2>
                <div class="summary-text">{recommendation.summary}</div>
            </div>
            
            <h2>今日必读 TOP 5</h2>
            
            {self.format_articles_html(recommendation.articles)}
            
            <hr>
            <p style="color: #999; font-size: 12px;">
                不想收到推送？<a href="#">取消订阅</a>
            </p>
        </body>
        </html>
        """
        
        return html
    
    def format_articles_html(
        self,
        articles: List[ScoredArticle]
    ) -> str:
        """
        格式化文章列表为HTML
        """
        
        html_parts = []
        
        for i, scored in enumerate(articles, 1):
            article = scored.article
            
            # 计算阅读时长（粗略估计）
            reading_time = len(article.content) // 400  # 假设每分钟400字
            
            html = f"""
            <div class="article">
                <div class="title">
                    {i}. <a href="{article.url}">{article.title}</a>
                </div>
                <div class="meta">
                    来源：{article.source} | 
                    相关度：{int(scored.score)}% | 
                    阅读时长：约{reading_time}分钟
                </div>
                <div class="summary-text">
                    {article.summary}
                </div>
                <div style="margin-top: 10px;">
                    <span class="tag">{article.category}</span>
                    <span class="tag">💡 {scored.reasoning[:30]}...</span>
                </div>
            </div>
            """
            
            html_parts.append(html)
        
        return "\n".join(html_parts)
```

---

## 数据模型

```python
from dataclasses import dataclass
from typing import List, Optional
from datetime import datetime

@dataclass
class Article:
    """文章"""
    id: str
    title: str
    summary: str
    content: str
    source: str  # 36氪、虎嗅等
    url: str
    category: str  # funding/product/personnel/strategy/other
    keywords: List[str]
    published_at: datetime
    crawled_at: datetime

@dataclass
class CEOProfile:
    """CEO档案"""
    user_id: str
    company_name: str
    industry: str
    product_description: str
    competitors: List[str]
    interests: List[str]
    keywords: List[str]  # AI生成的关键词
    topics: List[str]  # 关注的话题
    related_companies: List[str]  # 相关公司
    recommended_sources: List[str]  # 推荐关注的公众号
    created_at: datetime
    updated_at: datetime

@dataclass
class ScoredArticle:
    """评分后的文章"""
    article: Article
    score: float  # 综合评分 0-100
    topic_score: float  # 主题相关性
    competitor_score: float  # 竞品相关性
    actionable_score: float  # 可操作性
    reasoning: str  # 评分理由

@dataclass
class DailyRecommendation:
    """每日推荐"""
    user_id: str
    date: date
    summary: str  # 每日总结
    articles: List[ScoredArticle]  # Top 5
    generated_at: datetime

@dataclass
class UserFeedback:
    """用户反馈"""
    user_id: str
    article_id: str
    rating: int  # 1-5
    helpful: bool
    timestamp: datetime
```

---

## 数据库设计

```sql
-- 文章表
CREATE TABLE articles (
    id VARCHAR(36) PRIMARY KEY,
    title TEXT NOT NULL,
    summary TEXT,
    content TEXT,
    source VARCHAR(100),
    url VARCHAR(500),
    category VARCHAR(50),
    keywords JSON,  -- 存储关键词数组
    published_at TIMESTAMP,
    crawled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    INDEX idx_published_at (published_at),
    INDEX idx_source (source),
    INDEX idx_category (category)
);

-- CEO档案表
CREATE TABLE ceo_profiles (
    user_id VARCHAR(36) PRIMARY KEY,
    company_name VARCHAR(200),
    industry VARCHAR(100),
    product_description TEXT,
    competitors JSON,  -- 竞品列表
    interests JSON,  -- 兴趣列表
    keywords JSON,  -- 关键词
    topics JSON,  -- 关注话题
    related_companies JSON,
    recommended_sources JSON,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
);

-- 每日推荐表
CREATE TABLE daily_recommendations (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    date DATE NOT NULL,
    summary TEXT,
    articles JSON,  -- 存储Top 5文章信息
    generated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    UNIQUE KEY unique_user_date (user_id, date),
    INDEX idx_user_date (user_id, date)
);

-- 用户反馈表
CREATE TABLE user_feedback (
    id VARCHAR(36) PRIMARY KEY,
    user_id VARCHAR(36) NOT NULL,
    article_id VARCHAR(36) NOT NULL,
    rating INT,
    helpful BOOLEAN,
    comment TEXT,
    timestamp TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id),
    FOREIGN KEY (article_id) REFERENCES articles(id),
    INDEX idx_user_id (user_id),
    INDEX idx_article_id (article_id)
);

-- 用户表
CREATE TABLE users (
    id VARCHAR(36) PRIMARY KEY,
    email VARCHAR(200) UNIQUE,
    name VARCHAR(100),
    preferences JSON,  -- 推送偏好
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    subscription_status VARCHAR(50),  -- active/inactive/trial
    subscription_expires_at TIMESTAMP
);
```

---

## 部署架构

### 推荐方案

**后端：**
- Railway / Render
  - 支持定时任务
  - 数据库内置
  - 便宜（$5-20/月）

**数据库：**
- PostgreSQL（Railway自带）

**定时任务：**
- Celery + Redis
- 或者简单点：APScheduler

**前端：**
- 简单的Web界面（查看历史推荐）
- Vercel部署

### 定时任务设计

```python
from apscheduler.schedulers.asyncio import AsyncIOScheduler

scheduler = AsyncIOScheduler()

# 1. 每小时抓取文章
@scheduler.scheduled_job('cron', hour='*')
async def crawl_articles():
    crawler = ArticleCrawler()
    await crawler.crawl_daily()

# 2. 每天早上6点生成推荐
@scheduler.scheduled_job('cron', hour=6)
async def generate_recommendations():
    engine = RecommendationEngine()
    
    # 为所有活跃用户生成推荐
    users = await db.get_active_users()
    
    for user in users:
        recommendation = await engine.generate_daily_recommendation(user.id)

# 3. 每天早上7点推送
@scheduler.scheduled_job('cron', hour=7)
async def send_digests():
    delivery = DeliverySystem()
    
    # 获取今天的推荐
    recommendations = await db.get_todays_recommendations()
    
    for rec in recommendations:
        await delivery.send_daily_digest(rec.user_id, rec)

scheduler.start()
```

---

## 开发路线图

### Week 1: MVP验证（手动）

**目标：验证需求**

**任务：**
1. 找3-5个CEO朋友
2. 每天手动给他们发推荐：
   - 自己筛选5篇文章
   - 自己写总结
   - 发邮件
3. 收集反馈：
   - 有用吗？
   - 愿意付费吗？
   - 需要什么改进？

**如果反馈好 → 继续开发**
**如果反馈一般 → 调整或放弃**

### Week 2-3: 自动化核心流程

**任务：**
- [ ] 接入公众号数据API（第三方）
- [ ] 实现文章采集器
- [ ] 实现CEO Profile管理
- [ ] 实现相关性评分（Claude）
- [ ] 实现每日总结生成（Claude）
- [ ] 实现邮件推送

**验收：**
- 能自动抓取文章
- 能自动生成推荐
- 能自动发送邮件

### Week 4: Web界面

**任务：**
- [ ] 用户注册/登录
- [ ] 创建/编辑Profile
- [ ] 查看历史推荐
- [ ] 反馈功能（有用/无用）

### Week 5: 优化和测试

**任务：**
- [ ] 性能优化
- [ ] 推荐质量优化
- [ ] 错误处理
- [ ] 监控和日志

### Week 6+: 运营和迭代

**任务：**
- 找更多测试用户
- 根据反馈迭代
- 考虑收费

---

## 风险与挑战

### 1. 技术风险

**风险：公众号数据源不稳定**
- **应对：**使用付费的第三方API（更稳定）
- **备选方案：**如果API出问题，暂时手动补充

**风险：AI评分质量不稳定**
- **应对：**
  - 收集用户反馈，持续优化prompt
  - 加入规则过滤（先筛选，再AI评分）
  - 多轮迭代

### 2. 产品风险

**风险：推荐不准确，CEO不满意**
- **应对：**
  - 早期密集收集反馈
  - 每周优化一次
  - 提供反馈机制，快速改进

**风险：文章太多/太少**
- **应对：**
  - 可配置推送数量（3-10篇）
  - 根据用户反馈调整

### 3. 商业风险

**风险：用户不愿意付费**
- **应对：**
  - 早期免费，积累用户
  - 证明价值后再收费
  - 定价合理（$99/月，省了COO的工作）

**风险：获客困难**
- **应对：**
  - 早期靠口碑（CEO圈子小）
  - 提供免费试用（1个月）
  - 内容营销

### 4. 数据风险

**风险：第三方API数据质量差**
- **应对：**
  - 同时使用多个数据源
  - 人工抽查质量
  - 用户反馈机制

---

## 成本优化策略

### 降低Claude API成本

**问题：**每用户每月$60的API成本太高

**优化方案：**

**1. 批量处理（最重要）**
```python
# 不是每个用户单独跑
# 而是：
# 1. 所有用户共享文章池
# 2. 批量评分（一次评分10篇）
# 3. 只对每个用户做最后的个性化筛选

# 优化前：
for user in users:
    for article in all_articles:  # 100篇
        score = await score_article(article, user)  # 100次API调用

# 优化后：
all_articles = get_articles()  # 100篇
batch_scored = await batch_score(all_articles)  # 10次API调用（批量）

for user in users:
    personalized = filter_for_user(batch_scored, user)  # 1次API调用

# 成本降低90%
```

**2. 缓存相似查询**
```python
# 如果两个CEO的profile很相似（同行业、同竞品）
# 可以共享评分结果
```

**3. 使用更便宜的模型做初筛**
```python
# 不用Claude做所有筛选
# 用关键词匹配做初筛（免费）
# 只用Claude做最后的Top 20 → Top 5
```

**优化后成本：**
- 每用户每月：$20（从$60降到$20）
- 100用户：$2000/月（可接受）

---

## 关键成功因素

### 1. 推荐质量

**最重要！**
- CEO是否真的觉得推荐有用？
- 是否节省了时间？
- 是否有actionable insights？

**衡量指标：**
- 打开率 >60%
- 点击率 >40%
- 用户评价 >4/5

### 2. 简单易用

- 注册流程简单（5分钟完成Profile）
- 推送格式清晰（手机可读）
- 没有复杂配置

### 3. 稳定可靠

- 每天按时推送（早上7点准时）
- 数据质量稳定
- 不出错

### 4. 快速迭代

- 每周根据反馈优化
- 快速响应用户需求
- 持续改进推荐质量

---

## 竞争优势总结

### 我们的优势

1. **垂直专注**
   - 只服务CEO这个群体
   - 只做"同行动态"这件事
   - 不做大而全

2. **简单好用**
   - 不需要学习
   - 不需要配置
   - 每天5分钟

3. **个性化**
   - 基于每个CEO的背景
   - 持续学习优化
   - 不是通用推荐

4. **成本优势**
   - 比雇COO便宜
   - 比企业级工具便宜
   - 性价比高

---

## 总结

### 这个项目值得做吗？

**✅ 需求真实**
- CEO确实需要了解同行
- 现有方案都不够好
- 有明确的痛点

**✅ 技术可行**
- 公众号数据可以获取（第三方API）
- Claude可以做好相关性判断
- 成本可控（优化后）

**✅ 商业可行**
- 定价合理（$99/月）
- 毛利率健康（优化后70%+）
- 市场规模足够

### 建议

**Week 1: 手动验证（最重要）**
- 找3-5个CEO朋友
- 手动给他们发推荐
- 验证需求

**如果反馈好：**
→ Week 2-3: 开发MVP
→ Week 4-5: 测试和优化
→ Week 6+: 扩大用户

**如果反馈一般：**
→ 调整产品
→ 或者放弃

### 与前面的"博文生成器"的关系

**可以整合：**
1. Newspaper提供行业动态
2. 博文生成器用这些动态写文章

**但建议：**
- 先分开做（focus）
- 每个都做到能用
- 再考虑整合

### 我的判断

**这个项目比博文生成器更好做。**

**理由：**
- 需求更清晰（CEO要看同行动态）
- 技术更简单（不需要图片、不需要复杂生成）
- 价值更直接（节省时间、避免信息差）
- 更容易验证（手动测试就知道有没有需求）

**建议优先级：**
1. **Newspaper（推荐先做）** - 2周MVP
2. 博文配图工具 - 2周MVP
3. 整合（如果都成功）

---

*可行性分析与技术设计 v1.0*
*创建日期: 2026-01-15*
*建议：先手动验证需求*

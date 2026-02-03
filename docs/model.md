# 数据模型

## 文章

### 数据总览

文章全部数据模型字段说明：（待补充）

| 字段       | 说明                                      |
| ---------- | ----------------------------------------- |
| id         | 文章唯一标识符                            |
| slug       | 文章 URL 别名                             |
| title      | 文章标题                                  |
| content    | 文章内容                                  |
| renderType | 渲染方式（tech/life）                     |
| category   | 文章分类                                  |
| series     | 文章系列/专栏（与分类不同的文章收录方式） |
| tags       | 文章标签列表                              |
| createdAt  | 创建时间                                  |
| updatedAt  | 更新时间                                  |
| viewCount  | 文章浏览次数                              |
| coverImage | 文章封面图片URL                           |
| summary    | 文章摘要                                  |
| content    | 文章正文内容（markdown格式）              |

### 数据库表结构

#### A. 文章主表 (posts) - 存放不随语言改变的“硬信息”

| 字段        | 类型         | 约束/默认值               | 说明                      |
| ----------- | ------------ | ------------------------- | ------------------------- |
| id          | SERIAL       | PRIMARY KEY               | 唯一标识                  |
| slug        | VARCHAR(255) | UNIQUE, NOT NULL          | URL 别名                  |
| render_type | VARCHAR(20)  | DEFAULT 'tech'            | tech(简约) / life(二次元) |
| category    | VARCHAR(50)  | -                         | 分类                      |
| cover_image | TEXT         | -                         | 封面图                    |
| view_count  | INT          | DEFAULT 0                 | 阅读量                    |
| created_at  | TIMESTAMPTZ  | DEFAULT CURRENT_TIMESTAMP | 创建时间                  |
| updated_at  | TIMESTAMPTZ  | DEFAULT CURRENT_TIMESTAMP | 更新时间                  |

#### B. 文章内容表 (post_contents) - 存放随语言改变的文本

| 字段     | 类型         | 约束/默认值                            | 说明           |
| -------- | ------------ | -------------------------------------- | -------------- |
| id       | SERIAL       | PRIMARY KEY                            | 唯一标识       |
| post_id  | INT          | REFERENCES posts(id) ON DELETE CASCADE | 关联 posts.id  |
| language | VARCHAR(10)  | NOT NULL                               | zh / en / jp   |
| title    | VARCHAR(255) | NOT NULL                               | 标题           |
| summary  | TEXT         | -                                      | 列表展示的摘要 |
| content  | TEXT         | NOT NULL                               | Markdown 正文  |

| 约束                      | 说明           |
| ------------------------- | -------------- |
| UNIQUE(post_id, language) | 同篇同语种唯一 |

### API 数据模型

| 模型       | 说明     | 字段                                                                                                  |
| ---------- | -------- | ----------------------------------------------------------------------------------------------------- |
| PostItem   | 用于列表 | id, slug, renderType, category, coverImage, viewCount, createdAt, updatedAt, title, summary, language |
| PostDetail | 用于详情 | 继承 PostItem 全部字段，新增 content (Markdown 字符串)                                                |

### 编码数据模型

```go
// 数据库原始模型
type Post struct {
  ID         uint      `gorm:"primaryKey"`
  AuthorID   uint
  Slug       string    `gorm:"uniqueIndex"`
  RenderType string    // tech, life
  CategoryID uint
  SeriesID   *uint
  CoverImage string
  ViewCount  int
  CreatedAt  time.Time
  UpdatedAt  time.Time
}

type PostContent struct {
  PostID   uint   `gorm:"primaryKey"`
  Language string // zh / en / jp
  Title    string
  Summary  string
  Content  string
}

// 给前端返回的响应对象 (DTO)
type PostResponse struct {
  ID         uint      `json:"id"`
  Slug       string    `json:"slug"`
  Author     string    `json:"author"`
  Render     string    `json:"render"`
  Category   string    `json:"category"`
  Series     *string   `json:"series,omitempty"`
  Tags       []string  `json:"tags"`
  CoverImage string    `json:"coverImage"`
  Count      int       `json:"count"`
  CreatedAt  time.Time `json:"createdAt"`
  UpdatedAt  time.Time `json:"updatedAt"`
  Language   string    `json:"language"`
  Title      string    `json:"title"`
  Summary    string    `json:"summary"`
  Content    string    `json:"content,omitempty"` // omitempty 表示列表页不传此字段
}
```

```typescript
// 定义一个基础类型
interface BasePost {
  id: number;
  slug: string;
  author: string;
  render: "tech" | "life";
  category: string;
  series?: string;
  tags: string[];
  coverImage: string;
  count: number;
  createdAt: string;
  updatedAt: string;
}

// 列表项类型
interface PostListItem extends BasePost {
  title: string;
  summary: string;
  language: "zh" | "en" | "jp";
}

// 详情类型
interface PostDetail extends PostListItem {
  content: string;
}
```

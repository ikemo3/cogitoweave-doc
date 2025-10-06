<!-- markdownlint-disable MD024 -->

# Database

## データベース Container 契約

### Concepts テーブルスキーマ

<!-- REFERENCE_BEGIN: concept-table-schema -->

```sql
-- ============================================
-- Concept (概念) テーブル
-- ============================================
CREATE TABLE concept (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name VARCHAR(100) NOT NULL UNIQUE CHECK(name != ''),
    description VARCHAR(1000) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
);

CREATE INDEX idx_concept_name ON concept(name);
CREATE INDEX idx_concept_description ON concept(description);
```

<!-- REFERENCE_END: concept-table-schema -->

### Literatures テーブルスキーマ

<!-- REFERENCE_BEGIN: literature-table-schema -->

```sql
-- ============================================
-- Literature (文献) テーブル - 基底
-- ============================================
CREATE TABLE literature (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    kind VARCHAR(10) NOT NULL CHECK (kind IN ('web', 'book'))
);
```

<!-- REFERENCE_END: literature-table-schema -->

### Web Articles テーブルスキーマ

<!-- REFERENCE_BEGIN: web-article-table-schema -->

```sql
-- ============================================
-- WebArticle (Web記事) テーブル
-- ============================================
CREATE TABLE web_article (
    literature_id INTEGER PRIMARY KEY,
    url VARCHAR(2048) NOT NULL CHECK(url != ''),
    title VARCHAR(500) NOT NULL CHECK(title != ''),
    FOREIGN KEY (literature_id) REFERENCES literature(id) ON DELETE RESTRICT
);

CREATE INDEX idx_web_article_url ON web_article(url);
```

<!-- REFERENCE_END: web-article-table-schema -->

### Books テーブルスキーマ

<!-- REFERENCE_BEGIN: book-table-schema -->

```sql
-- ============================================
-- Book (書籍) テーブル
-- ============================================
CREATE TABLE book (
    literature_id INTEGER PRIMARY KEY,
    isbn VARCHAR(20) NOT NULL UNIQUE CHECK(isbn != ''),
    author VARCHAR(200) NOT NULL CHECK(author != ''),
    publisher VARCHAR(200) NOT NULL CHECK(publisher != ''),
    publication_year INTEGER NOT NULL,
    FOREIGN KEY (literature_id) REFERENCES literature(id) ON DELETE RESTRICT
);

CREATE INDEX idx_book_isbn ON book(isbn);
```

<!-- REFERENCE_END: book-table-schema -->

### Literature Memos テーブルスキーマ

<!-- REFERENCE_BEGIN: literature-memo-table-schema -->

```sql
-- ============================================
-- LiteratureMemo (文献メモ) テーブル
-- ============================================
CREATE TABLE literature_memo (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    concept_id INTEGER NOT NULL,
    literature_id INTEGER NOT NULL,
    title VARCHAR(500) NOT NULL CHECK(title != ''),
    content TEXT NOT NULL CHECK(content != ''),
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (concept_id) REFERENCES concept(id) ON DELETE RESTRICT,
    FOREIGN KEY (literature_id) REFERENCES literature(id) ON DELETE RESTRICT
);

CREATE INDEX idx_literature_memo_concept_id ON literature_memo(concept_id);
CREATE INDEX idx_literature_memo_literature_id ON literature_memo(literature_id);
```

<!-- REFERENCE_END: literature-memo-table-schema -->

### Relationships テーブルスキーマ

<!-- REFERENCE_BEGIN: relationship-table-schema -->

```sql
-- ============================================
-- Relationship (関係性) テーブル
-- ============================================
CREATE TABLE relationship (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    concept_a_id INTEGER NOT NULL,
    concept_b_id INTEGER NOT NULL,
    relationship_name VARCHAR(100) NOT NULL CHECK(relationship_name != ''),
    description VARCHAR(500) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    FOREIGN KEY (concept_a_id) REFERENCES concept(id) ON DELETE RESTRICT,
    FOREIGN KEY (concept_b_id) REFERENCES concept(id) ON DELETE RESTRICT,
    CHECK (concept_a_id < concept_b_id),
    UNIQUE (concept_a_id, concept_b_id, relationship_name)
);

CREATE INDEX idx_relationship_concept_a_id ON relationship(concept_a_id);
CREATE INDEX idx_relationship_concept_b_id ON relationship(concept_b_id);
```

<!-- REFERENCE_END: relationship-table-schema -->

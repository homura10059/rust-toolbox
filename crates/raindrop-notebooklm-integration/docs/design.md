# Design Document: Raindrop-NotebookLM Integration

## Overview

このドキュメントは、raindrop.ioのブックマークからNotebookLMノートを自動生成するツールの設計書です。本ツールは、指定されたタグのブックマークを取得し、各URLのコンテンツを収集してNotebookLMに統合されたノートを作成します。

## System Architecture

### High-Level Architecture

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│   Raindrop.io   │    │  Content Fetch  │    │   NotebookLM    │
│      API        │───→│    Service      │───→│     API         │
└─────────────────┘    └─────────────────┘    └─────────────────┘
         │                       │                       │
         ▼                       ▼                       ▼
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Bookmark List  │    │  Page Content   │    │  Generated Note │
└─────────────────┘    └─────────────────┘    └─────────────────┘
```

### Component Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    Main Application                     │
├─────────────────────────────────────────────────────────┤
│  Configuration Manager  │  Progress Tracker  │  Logger  │
├─────────────────────────────────────────────────────────┤
│ Raindrop Client │ Content Fetcher │ NotebookLM Client   │
├─────────────────────────────────────────────────────────┤
│        HTTP Client        │      File Storage          │
└─────────────────────────────────────────────────────────┘
```

## Module Design

### 1. Configuration Module (`config.rs`)

**Purpose:** アプリケーションの設定管理

**Components:**
- `Config` struct: アプリケーション設定を保持
- `ConfigBuilder`: 設定のビルダーパターン実装
- 環境変数とファイルからの設定読み込み

```rust
pub struct Config {
    pub raindrop_api_key: String,
    pub notebooklm_api_key: String,
    pub max_urls: Option<usize>,
    pub output_dir: PathBuf,
    pub content_filters: Vec<String>,
}
```

### 2. Raindrop Client Module (`raindrop.rs`)

**Purpose:** Raindrop.io APIとの通信

**Components:**
- `RaindropClient`: API通信のメインクライアント
- `Bookmark` struct: ブックマークデータ構造
- `Tag` enum: タグ情報の管理

**Key Methods:**
- `fetch_bookmarks_by_tag(tag: &str) -> Result<Vec<Bookmark>>`
- `authenticate() -> Result<()>`

### 3. Content Fetcher Module (`content.rs`)

**Purpose:** Webコンテンツの取得と処理

**Components:**
- `ContentFetcher`: コンテンツ取得のメインサービス
- `PageContent` struct: 取得したページコンテンツ
- `ContentExtractor`: HTMLからテキスト抽出

**Key Methods:**
- `fetch_content(url: &str) -> Result<PageContent>`
- `extract_text(html: &str) -> String`
- `generate_summary(content: &str) -> String`

### 4. NotebookLM Client Module (`notebooklm.rs`)

**Purpose:** NotebookLM APIとの通信

**Components:**
- `NotebookLMClient`: API通信クライアント
- `Note` struct: ノート構造
- `NoteBuilder`: ノート作成のビルダー

**Key Methods:**
- `create_note(content: Vec<PageContent>) -> Result<Note>`
- `format_content(contents: Vec<PageContent>) -> String`

### 5. Storage Module (`storage.rs`)

**Purpose:** データの永続化とファイル管理

**Components:**
- `FileManager`: ファイル操作の管理
- `DataSerializer`: データのシリアライゼーション
- `LogWriter`: ログファイルの管理

**Key Methods:**
- `save_json(data: &T, filename: &str) -> Result<()>`
- `save_markdown_summary(summary: &ExecutionSummary) -> Result<()>`
- `write_log(message: &str, level: LogLevel) -> Result<()>`

### 6. Progress Module (`progress.rs`)

**Purpose:** 処理進捗の表示と管理

**Components:**
- `ProgressTracker`: 進捗追跡
- `ProgressBar`: プログレスバー表示
- `StatusReporter`: 状態レポート

## Data Flow

### 1. Initialization Phase
```
User Input → Configuration Loading → API Authentication → Validation
```

### 2. Data Collection Phase
```
Tag Specification → Raindrop API Call → Bookmark List → Content Fetching → Page Content Collection
```

### 3. Processing Phase
```
Content Processing → Text Extraction → Summary Generation → Data Formatting
```

### 4. Integration Phase
```
NotebookLM API Call → Note Creation → Result Validation → Success Response
```

### 5. Storage Phase
```
Data Serialization → File Writing → Log Recording → Summary Generation
```

## Error Handling Strategy

### Error Types
```rust
#[derive(Debug, thiserror::Error)]
pub enum AppError {
    #[error("Raindrop API error: {0}")]
    RaindropError(String),

    #[error("Content fetch error: {0}")]
    ContentError(String),

    #[error("NotebookLM API error: {0}")]
    NotebookLMError(String),

    #[error("Configuration error: {0}")]
    ConfigError(String),

    #[error("Storage error: {0}")]
    StorageError(String),
}
```

### Error Recovery
- **Network Errors:** 指数バックオフによる自動リトライ
- **API Rate Limits:** レート制限対応の遅延処理
- **Content Fetch Failures:** 失敗したURLをスキップして継続
- **File System Errors:** 代替パスでの保存試行

## Security Considerations

### API Key Management
- 環境変数での安全な保存
- 設定ファイルでの暗号化オプション
- メモリ内での適切なクリアリング

### Data Privacy
- 一時ファイルの安全な削除
- ログファイルでの機密情報のマスキング
- HTTPS通信の強制

## Performance Considerations

### Concurrency
- 非同期処理によるHTTPリクエストの並列実行
- コンテンツ取得の並列処理（レート制限考慮）
- メモリ効率的なストリーミング処理

### Resource Management
- HTTPクライアントの接続プーリング
- メモリ使用量の監視と制御
- 一時ファイルのクリーンアップ

## Testing Strategy

### Unit Tests
- 各モジュールの独立したテスト
- モック使用による外部依存の分離
- エラーケースの網羅的テスト

### Integration Tests
- API通信のエンドツーエンドテスト
- ファイル操作のテスト
- 設定読み込みのテスト

### Performance Tests
- 大量URL処理のベンチマーク
- メモリ使用量のプロファイリング
- 並行処理性能の測定

## Deployment and Configuration

### Build Configuration
```toml
[dependencies]
tokio = { version = "1.0", features = ["full"] }
reqwest = { version = "0.11", features = ["json"] }
serde = { version = "1.0", features = ["derive"] }
clap = { version = "4.0", features = ["derive"] }
anyhow = "1.0"
thiserror = "1.0"
scraper = "0.18"
```

### Runtime Requirements
- Rust 1.70+
- インターネット接続
- Raindrop.io APIアクセス
- NotebookLM APIアクセス

## Future Enhancements

### Phase 2 Features
- バッチ処理のスケジューリング
- 複数タグの同時処理
- コンテンツフィルタリングの高度化
- ダッシュボードUI

### Phase 3 Features
- Webインターフェース
- 他のブックマークサービス対応
- AI要約機能の向上
- チーム機能とシェアリング

## Implementation Notes

### Directory Structure
```
crates/raindrop-notebooklm-integration/
├── src/
│   ├── main.rs
│   ├── config.rs
│   ├── raindrop.rs
│   ├── content.rs
│   ├── notebooklm.rs
│   ├── storage.rs
│   ├── progress.rs
│   └── error.rs
├── tests/
├── examples/
└── docs/
```

### CLI Interface
```bash
cargo run -p raindrop-notebooklm-integration -- \
  --tag "research" \
  --max-urls 50 \
  --output-dir ./output
```

この設計に基づいて、段階的な実装を進めることで、要件を満たすツールを構築できます。
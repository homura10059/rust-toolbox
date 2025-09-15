# Requirements Document

## Introduction

この機能は、raindrop.ioの特定のタグが付けられたブックマークからURL一覧を取得し、それらのコンテンツを使ってNotebookLMのノートを自動作成するツールです。ユーザーがraindrop.ioで整理したブックマークを効率的にNotebookLMで活用できるようになります。

## Requirements

### Requirement 1

**User Story:** ユーザーとして、raindrop.ioの特定のタグを指定して、そのタグが付けられたブックマークのURL一覧を取得したい。そうすることで、整理されたブックマークを効率的に活用できる。

#### Acceptance Criteria

1. WHEN ユーザーがraindrop.ioのタグ名を指定 THEN システム SHALL そのタグが付けられた全ブックマークのURLを取得する
2. WHEN raindrop.io APIにアクセス THEN システム SHALL 適切な認証情報を使用してAPIリクエストを実行する
3. IF 指定されたタグが存在しない THEN システム SHALL エラーメッセージを表示する
4. WHEN URL取得が完了 THEN システム SHALL 取得したURL数とタイトル一覧を表示する

### Requirement 2

**User Story:** ユーザーとして、取得したURLのコンテンツを自動的に収集したい。そうすることで、各ページの内容をNotebookLMで分析できる。

#### Acceptance Criteria

1. WHEN URL一覧が取得された THEN システム SHALL 各URLのコンテンツを順次取得する
2. WHEN Webページにアクセス THEN システム SHALL ページのテキストコンテンツを抽出する
3. IF URLにアクセスできない THEN システム SHALL そのURLをスキップして次に進む
4. WHEN コンテンツ取得中 THEN システム SHALL 進捗状況を表示する
5. WHEN コンテンツ抽出が完了 THEN システム SHALL 各ページのタイトルと要約を生成する

### Requirement 3

**User Story:** ユーザーとして、収集したコンテンツを使ってNotebookLMのノートを作成したい。そうすることで、複数のソースを統合した知識ベースを構築できる。

#### Acceptance Criteria

1. WHEN コンテンツ収集が完了 THEN システム SHALL NotebookLM APIを使用してノートを作成する
2. WHEN ノート作成時 THEN システム SHALL 各URLのコンテンツを適切にフォーマットして含める
3. WHEN ノート作成時 THEN システム SHALL ソースURLと取得日時を記録する
4. IF NotebookLM APIでエラーが発生 THEN システム SHALL エラー詳細を表示して処理を停止する
5. WHEN ノート作成が成功 THEN システム SHALL 作成されたノートのIDとURLを表示する

### Requirement 4

**User Story:** ユーザーとして、処理の設定をカスタマイズしたい。そうすることで、自分のニーズに合わせてツールを使用できる。

#### Acceptance Criteria

1. WHEN ツールを実行 THEN システム SHALL raindrop.io APIキーの設定を要求する
2. WHEN ツールを実行 THEN システム SHALL NotebookLM APIキーの設定を要求する
3. WHEN 設定ファイルが存在 THEN システム SHALL 保存された設定を読み込む
4. WHEN ユーザーが最大URL数を指定 THEN システム SHALL その数まででURL取得を制限する
5. WHEN ユーザーがコンテンツフィルターを指定 THEN システム SHALL 指定された条件でコンテンツをフィルタリングする

### Requirement 5

**User Story:** ユーザーとして、処理結果をローカルにも保存したい。そうすることで、後で参照したり、バックアップとして活用できる。

#### Acceptance Criteria

1. WHEN コンテンツ取得が完了 THEN システム SHALL 取得したデータをJSONファイルとして保存する
2. WHEN ノート作成が完了 THEN システム SHALL 作成されたノートの情報をログファイルに記録する
3. WHEN エラーが発生 THEN システム SHALL エラー詳細をログファイルに記録する
4. WHEN 処理が完了 THEN システム SHALL 実行サマリーをMarkdownファイルとして出力する
5. IF 同名のファイルが存在 THEN システム SHALL タイムスタンプを付けて重複を避ける
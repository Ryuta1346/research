# Subagent 使用例とベストプラクティス

## 実用的な使用例

### 1. コードレビュー専門エージェント
```
名前: code-reviewer
説明: Expert code review specialist. Proactively reviews code for quality, security, and maintainability.
使用場面: PRレビュー、コード品質チェック、セキュリティ監査
```

### 2. デバッグ専門エージェント
```
名前: debugger
説明: Debugging specialist for errors, test failures, and unexpected behavior.
使用場面: エラー解析、テスト失敗の調査、パフォーマンス問題の診断
```

### 3. データサイエンスエージェント
```
名前: data-scientist
説明: Data analysis expert for SQL queries, BigQuery operations, and data insights.
使用場面: データ分析、SQLクエリ最適化、データインサイトの抽出
```

### 4. API設計エージェント
```
名前: api-designer
説明: API design specialist focused on RESTful patterns and best practices.
使用場面: API設計、エンドポイント定義、スキーマ設計
```

### 5. パフォーマンス最適化エージェント
```
名前: performance-optimizer
説明: Performance tuning expert for identifying and fixing bottlenecks.
使用場面: パフォーマンス分析、最適化提案、ボトルネック特定
```

## ベストプラクティス

### 1. 単一責任の原則
- 各エージェントは1つの明確な目的に特化させる
- 複数の責任を持たせない

### 2. 詳細なシステムプロンプト
- 具体的な指示と制約を含める
- 期待される出力形式を明示
- エラーハンドリングの方針を記載

### 3. ツールアクセスの制限
- 必要最小限のツールのみアクセス許可
- セキュリティリスクを最小化

### 4. プロジェクト固有エージェントの共有
- `.claude/agents/`ディレクトリに保存
- チーム全体で共有可能
- バージョン管理対象に含める

### 5. 段階的な機能拡張
- 最初は限定的な機能から開始
- 動作確認後に徐々に拡張
- フィードバックを反映して改善
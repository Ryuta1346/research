# Claude Code Subagent 網羅的調査報告書

作成日: 2025年7月30日  
調査実施者: Claude Code (using gemini)

## エグゼクティブサマリー

Claude Code の Subagent（サブエージェント）機能は、特定タスクに特化した独立したAIアシスタントを作成・管理できる強力な機能である。本報告書では、Subagent の仕組み、使用方法、ベストプラクティス、制限事項について網羅的に調査した結果をまとめる。

## 1. Subagent とは

### 1.1 定義
Subagent は Claude Code 内で動作する特化型AIアシスタントで、以下の特徴を持つ：

- **独立したコンテキストウィンドウ**で動作
- **特定の目的とタスク**に特化
- **カスタムシステムプロンプト**を設定可能
- **ツールアクセスを制限**可能

### 1.2 主な利点
- メイン会話のコンテキストを保持
- 専門的で焦点を絞った専門知識の提供
- タスク固有のパフォーマンス向上
- 再利用可能で共有可能なワークフロー

## 2. 動作原理

### 2.1 Task Tool による起動
Subagent は主に Task Tool を通じて起動される。このツールは：

- 自律的な操作が可能（コードベース検索、ファイル読み取り、コマンド実行）
- 並列実行により効率的なタスク処理
- ステートレスな実行環境（タスク完了後は結果のみ返す）

### 2.2 コンテキスト分離
各 Subagent は独立したコンテキストで動作し、メインの会話に影響を与えない。これにより：

- 大規模なコードベースの探索が可能
- 複数の独立したタスクを並列処理
- メモリ効率の最適化

## 3. 使用方法

### 3.1 Subagent の作成

#### コマンドによる作成
```bash
/agents
```

#### 設定項目
- **名前**: 一意の識別子
- **説明**: 目的と機能の詳細
- **ツール制限**: アクセス可能なツールの指定
- **カスタムシステムプロンプト**: 専門的な指示

### 3.2 Subagent の保存場所

#### プロジェクトレベル
```
.claude/agents/[agent-name].md
```
- プロジェクト固有のエージェント
- Git管理対象として共有可能

#### ユーザーレベル
```
~/.claude/agents/[agent-name].md
```
- 全プロジェクトで利用可能
- 個人的な汎用エージェント

### 3.3 Subagent の起動方法

#### 自動デリゲーション
Claude Code がタスクの内容を判断し、適切な Subagent を自動選択

#### 明示的な起動
```
"use the code-reviewer subagent to check the auth module"
"have the debugger subagent investigate why users can't log in"
```

## 4. 実用的な使用例

### 4.1 専門エージェントの例

| エージェント名 | 用途 | 主な機能 |
|--------------|------|---------|
| code-reviewer | コードレビュー | 品質、セキュリティ、保守性のチェック |
| debugger | デバッグ | エラー解析、テスト失敗調査 |
| data-scientist | データ分析 | SQLクエリ、BigQuery操作 |
| api-designer | API設計 | RESTfulパターン、スキーマ設計 |
| performance-optimizer | パフォーマンス最適化 | ボトルネック特定、最適化提案 |

### 4.2 使用シナリオ

#### シナリオ1: 大規模コードベースの探索
```
タスク: "findメソッドの全実装を調査"
→ Subagent が並列で複数ファイルを検索・分析
```

#### シナリオ2: セキュリティ監査
```
タスク: "認証モジュールのセキュリティ脆弱性をチェック"
→ security-auditor Subagent が専門的な分析を実施
```

## 5. ベストプラクティス

### 5.1 設計原則

#### 単一責任の原則
- 各エージェントは1つの明確な目的に特化
- 複数の責任を持たせない

#### 詳細なシステムプロンプト
```markdown
---
name: code-reviewer
description: Expert code review specialist
---

You are a code review specialist focused on:
1. Code quality and maintainability
2. Security best practices
3. Performance implications
4. Design patterns adherence

Always provide:
- Specific line numbers for issues
- Severity levels (critical/high/medium/low)
- Suggested fixes with code examples
```

### 5.2 ツールアクセス管理

#### 最小権限の原則
- 必要最小限のツールのみ許可
- セキュリティリスクの最小化

#### 推奨設定例
```yaml
tools:
  - Read
  - Grep
  - Glob
  # Write や Bash は許可しない
```

### 5.3 開発フロー

1. **プロトタイプ作成**: Claude に初期エージェントを生成させる
2. **段階的拡張**: 限定的な機能から開始
3. **テストと改善**: 実際の使用でフィードバック収集
4. **共有と標準化**: チーム内で有用なエージェントを共有

## 6. 制限事項と注意点

### 6.1 技術的制限

| 制限項目 | 詳細 | 対策 |
|---------|------|------|
| コンテキストウィンドウ | 独立したウィンドウのため制限あり | タスクを適切に分割 |
| ステートレス実行 | 継続的な対話は不可 | 明確な一度きりのタスク定義 |
| レイテンシ | コンテキスト収集で遅延発生 | 並列実行で最適化 |

### 6.2 使用上の注意

#### セキュリティ考慮事項
- ツールアクセスは慎重に設定
- 機密情報を扱う場合は特に注意
- 監査ログの確認

#### エラーハンドリング
- Subagent の出力は基本的に信頼可能
- 重要な処理では検証ステップを追加

### 6.3 設定ファイルの制約

- **形式**: Markdown with YAML frontmatter のみ
- **エンコーディング**: UTF-8
- **ファイル名**: 英数字とハイフンのみ推奨

## 7. 高度な活用法

### 7.1 チェーンエージェント
複数の Subagent を連携させた複雑なワークフロー

```
1. analyzer エージェント → 問題分析
2. solution-designer エージェント → 解決策設計
3. implementer エージェント → 実装
4. validator エージェント → 検証
```

### 7.2 プロジェクト固有の標準化
`.claude/agents/` ディレクトリに標準エージェントセットを配置

```
.claude/
├── agents/
│   ├── pr-reviewer.md      # PR レビュー標準
│   ├── test-writer.md      # テスト作成標準
│   └── doc-generator.md    # ドキュメント生成標準
└── CLAUDE.md              # プロジェクト共通設定
```

## 8. 今後の展望と推奨事項

### 8.1 推奨される次のステップ
1. 基本的な Subagent から開始（code-reviewer, debugger）
2. プロジェクト固有のニーズに応じてカスタマイズ
3. チーム内でベストプラクティスを共有
4. 継続的な改善サイクルの確立

### 8.2 期待される発展
- より高度な並列処理機能
- エージェント間の直接通信
- 動的なツール権限管理
- 学習機能の統合

## 9. 結論

Claude Code の Subagent 機能は、開発作業を効率化し、専門的なタスクを高品質に実行するための強力なツールである。適切に設計・管理されたSubagentは、開発チームの生産性を大幅に向上させ、コード品質の維持に貢献する。

本報告書で示したベストプラクティスと注意点を踏まえ、プロジェクトの特性に応じた Subagent エコシステムを構築することを推奨する。

---

## 付録A: サンプルSubagent設定

### A.1 Code Reviewer エージェント
```markdown
---
name: code-reviewer
description: Expert code review specialist
tools:
  - Read
  - Grep
  - Glob
---

# Code Review Specialist

You are an expert code reviewer focused on:

## Primary Responsibilities
1. **Code Quality**: Identify code smells, anti-patterns, and maintainability issues
2. **Security**: Detect potential vulnerabilities and security risks
3. **Performance**: Flag performance bottlenecks and inefficient algorithms
4. **Best Practices**: Ensure adherence to language-specific best practices

## Review Process
1. Analyze code structure and architecture
2. Check for proper error handling
3. Verify test coverage
4. Assess documentation quality

## Output Format
Provide findings in the following structure:
- Issue: [Description]
- Severity: [Critical/High/Medium/Low]
- File: [Path:LineNumber]
- Suggestion: [Specific fix with code example]
```

### A.2 Performance Optimizer エージェント
```markdown
---
name: performance-optimizer
description: Performance analysis and optimization expert
tools:
  - Read
  - Grep
  - Bash
---

# Performance Optimization Specialist

You are a performance optimization expert specializing in:

## Core Competencies
1. **Profiling**: Identify performance bottlenecks using appropriate tools
2. **Algorithm Analysis**: Evaluate time and space complexity
3. **Resource Usage**: Monitor CPU, memory, and I/O patterns
4. **Optimization**: Suggest concrete improvements

## Analysis Approach
1. Measure baseline performance
2. Identify hotspots and bottlenecks
3. Analyze algorithmic complexity
4. Propose optimizations
5. Estimate performance improvements

## Deliverables
- Performance metrics before/after
- Bottleneck analysis report
- Optimization recommendations with code
- Trade-off analysis (performance vs. readability/maintainability)
```

## 付録B: トラブルシューティングガイド

### B.1 Subagent が起動しない
- `/agents` コマンドでエージェントリストを確認
- エージェント名のスペルミスをチェック
- ファイル権限を確認

### B.2 期待通りの動作をしない
- システムプロンプトの明確性を確認
- ツールアクセス権限を見直し
- タスク指示の具体性を向上

### B.3 パフォーマンス問題
- タスクを小さく分割
- 並列実行を活用
- 不要なツールアクセスを削除

---

*本報告書は Claude Code による自動調査に基づいて作成されました。*
## プロダクトマネージャーによるキックオフ

### 背景
簡単に使える計算機プログラムを提供し、開発者が日常的に行う基本的な計算作業を素早く正確にサポートすることを目的とします。複雑な環境設定や GUI が不要な CLI ツールとして提供し、どの OS でも動作するシンプルさを重視します。

### ユーザー価値
1. スクリプトやシェル上で即座に四則演算を実行できる
2. 追加ライブラリなしで動作し、学習コストが低い
3. 開発や学習環境に組み込みやすく、生産性を向上

### 基本要求
- 四則演算（加算、減算、乗算、除算）をサポートする
- コマンドライン引数または対話式入力で数値と演算子を受け取る
- 数値は整数および小数に対応する
- 不正入力時はエラーメッセージを表示し終了コード≠0で返す
- 標準出力に結果を表示する
- オープンソースライセンス（MIT）で公開する

## システムアーキテクトの意見

### 論点
- 実装言語の選定: Python 3 で標準ライブラリのみを使用すればクロスプラットフォーム性と保守性を確保できる
- CLI UX: 対話式とワンライナーの両立方法（サブコマンド vs 対話モード切替）
- 精度要件: 浮動小数点誤差を許容範囲に収めるため decimal モジュール利用を検討

### リスク
- 除算時のゼロ除算やオーバーフロー処理を怠ると異常終了につながる
- ユーザー入力を eval 等で解釈すると任意コード実行脆弱性となる
- 異なるロケールでの小数点記号差異によりパース失敗が発生する可能性

### アイデア
- argparse で安全にパースし、内部計算に decimal.Decimal を採用
- GitHub Actions でリンターと単体テストを自動実行し品質を担保
- 後方互換を保ちつつ将来的に指数演算や括弧対応を拡張可能な設計

## リード開発者の意見

### 論点
- CLI と対話モードを単一バイナリで共存させるエントリーポイント設計
- decimal.Decimal 採用時のパフォーマンス低下をどの程度許容するか

### リスク
- 小数点入力を locale 依存にしないため '.' 固定とすると誤入力が増える可能性
- テスト不足により浮動小数点→Decimal 変換まわりで精度リグレッションが検知されない恐れ

### アイデア
- GitHub Actions で macOS / Linux / Windows 向けバイナリを自動生成しリリース手間を削減
- ソースを 200 行以内に保ち読みやすさとレビュー容易性を最優先
- 演算コアをモジュール分割し演算子追加を O(1) コストで実装できる構造にする

## QAエンジニアの意見

### 論点
- ユニットテストとE2Eテストの責務分離をどこまで明確化するか
- 浮動小数点演算の許容誤差をテスト基準値として定義する必要性
- CLI 戻り値を仕様化し、スクリプト連携時の互換性を保証する方法

### リスク
- 入力バリデーション不足により英数字混在や空文字入力がクラッシュ要因となる
- 並列実行時に共有リソース（設定ファイルなど）をロックしないと競合が発生
- CI で異常系を網羅しないとリリース後に再現性の低い不具合が顕在化

### アイデア
- pytest を採用し、正常系・異常系・境界値のデータ駆動テストを100ケース以上準備
- 除算のゼロ除算、オーバーフロー、丸め誤差をパラメトライズして自動評価
- GitHub Actions で macOS/Linux/Windows 3環境のマトリクスビルドを行いクロスプラットフォーム品質を担保
- mutmut 等を用いたミューテーションテストでテスト網羅率では捕捉しきれない欠陥を検出

## システムアーキテクトによる設計提案

### 1. 目的と範囲
本計算機プログラムは CLI ベースのユーティリティとして四則演算を高速かつ安全に提供する。対象ユーザーは開発者・CI スクリプト作成者であり、追加依存なしで利用可能とする。

### 2. 技術選定
- **言語**: Python 3.11 以上（標準ライブラリのみ）
- **数値型**: `decimal.Decimal` に統一し、浮動小数点誤差を最小化
- **パッケージ管理**: `pyproject.toml` (PEP 621) + `pipx` 配布

### 3. アーキテクチャ
| レイヤ | 役割 |
| --- | --- |
| CLI 層 | `argparse` で引数/対話入力をパースし `Calculator` に委譲 |
| ドメイン層 | `Calculator` クラスが演算ディスパッチ (`+ - * /`) をカプセル化 |
| インフラ層 | I/O（stdin/stdout/exit code）とロギングのみ |

モジュール構成例:
```
calculator/
  __init__.py
  cli.py        # エントリーポイント
  core.py       # Calculator クラス
  errors.py     # 独自例外定義
```

### 4. 入力仕様
1. ワンライナー: `calc 3 + 4.5`
2. 対話式: 演算子と数値を段階的に入力 (`--interactive`)
3. ロケール非依存で小数点は `.` 固定

### 5. 出力仕様
- 標準出力に結果のみを表示
- エラー時は標準エラーにメッセージ、終了コード `1`

### 6. 例外・エラーハンドリング
| 事象 | 例外 | 挙動 |
| --- | --- | --- |
| 0 除算 | `ZeroDivisionError` | メッセージ出力後 exit 1 |
| 不正演算子 | `InvalidOperatorError` | 同上 |
| 数値解析失敗 | `InvalidNumberError` | 同上 |

### 7. テスト戦略
- `pytest` + `pytest-cov` で 100% ブランチ網羅を目標
- 正常系・異常系・境界値を Parametrize で定義
- mutmut によるミューテーションテストを週次 CI で実施

### 8. CI/CD
- GitHub Actions
  - `lint` (`ruff`), `typecheck` (`mypy`), `test` (3OS matrix)
  - 成功時に PyPI 発行 & Windows/Mac/Linux 自動バイナリリリース

### 9. セキュリティ
- `eval` 等の危険 API を使用しない
- 入力は全て厳格にバリデーションし、例外を捕捉

### 10. 将来拡張
- 演算子追加時は `Calculator.OPERATIONS` dict に関数を登録するだけで対応
- 式全体を解析するパーサを別モジュールで後付け可能


### リード開発者によるレビュー (1回目)
```json
{"status": "approved", "comment": "実装可能です。この設計で進めましょう。"}
```

## プロダクトマネージャーによる最終仕様

### 1. 目的
CLI で動作するシンプルな四則演算ツールを提供し、開発者が OS 依存なく即座に計算できる環境を実現する。

### 2. 機能要件
1. 四則演算: `+ - * /` をサポートし整数・小数を扱う。
2. 2 つの操作モードを提供。
   - ワンライナー: `calc 3 + 4.5`
   - 対話式: `calc --interactive` でプロンプト表示。
3. 返却値は標準出力に結果のみを表示。
4. 不正入力時は標準エラーにメッセージ出力し終了コード `1`。

### 3. 非機能要件
1. 算出結果は `decimal.Decimal` を用い IEEE754 誤差を回避。
2. 実行時間: 1 演算あたり 100 ms 以内 (平均) を目標。
3. 依存: Python 3.11 以上、追加パッケージなし。
4. セキュリティ: `eval` 等の危険 API 不使用。

### 4. 入出力仕様
| 入力例 | 出力 | 備考 |
| --- | --- | --- |
| `calc 2 * 3` | `6` | ワンライナー |
| `calc 10 / 3` | `3.3333333333333333333` | Decimal 既定精度 |

### 5. エラー仕様
| 事象 | メッセージ例 | 終了コード |
| --- | --- | --- |
| 0 除算 | `Error: division by zero` | 1 |
| 不正演算子 | `Error: invalid operator` | 1 |
| 数値変換失敗 | `Error: invalid number` | 1 |

### 6. テスト基準
- `pytest` によるユニットテスト 100 ケース以上。
- 正常系・異常系を網羅しブランチ網羅率 100%。
- mutmut によるミューテーションスコア 90% 以上。

### 7. CI/CD
- GitHub Actions で lint (ruff) / typecheck (mypy) / test を 3OS マトリクスで実行。
- main ブランチへのマージで PyPI & 各 OS 向け実行ファイルを自動リリース。

### 8. リリース判定基準
1. 全 CI ジョブ成功。
2. テスト・ミューテーション基準達成。
3. PM 最終確認にて仕様逸脱がないこと。

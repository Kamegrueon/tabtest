# tabtest テスト構成

このディレクトリには、`tabtest`ライブラリの包括的なテストスイートが含まれています。

## テスト構成

### 📁 テストファイル

#### **基本テスト**
- `test_workbook_parser.py` - WorkbookParserの基本機能テスト
- `test_basic_integration.py` - 基本的な結合テスト（推奨）

#### **単体テスト**
- `test_parsers.py` - 各パーサーモジュールの単体テスト
- `test_models.py` - 各モデルクラスの単体テスト
- `test_suite_helpers.py` - suiteヘルパー関数の単体テスト

#### **結合テスト**
- `test_integration.py` - モックファイルを使用した結合テスト
- `test_error_handling.py` - エラーハンドリングのテスト

#### **テストデータ**
- `assets/mock_workbook.twb` - テスト用のモックTableauワークブックファイル

## テストの実行

### 全テストの実行
```bash
python -m pytest tests/ -v
```

### 基本テストのみ実行（推奨）
```bash
python -m pytest tests/test_basic_integration.py -v
python -m pytest tests/test_workbook_parser.py -v
```

### 特定のテストファイル実行
```bash
python -m pytest tests/test_parsers.py -v
python -m pytest tests/test_models.py -v
python -m pytest tests/test_suite_helpers.py -v
```

### 特定のテストクラス実行
```bash
python -m pytest tests/test_basic_integration.py::TestBasicIntegration -v
```

## テストカバレッジ

### ✅ カバーされている機能

#### **パーサー**
- XMLファイル読み込み（.twb/.twbx）
- データソース解析
- ワークシート解析
- ダッシュボード解析
- パラメータ解析
- 計算フィールド解析
- フィルタ解析
- セット解析
- 参照解決

#### **モデル**
- WorkbookModel
- DatasourceModel
- WorksheetModel
- DashboardModel
- ParameterModel
- CalculatedFieldModel
- DataSourceFilterModel
- SetModel

#### **ヘルパー関数**
- データソース存在確認
- ワークシート存在確認
- ダッシュボード存在確認
- 計算フィールド存在確認
- フィルタ存在確認

#### **エラーハンドリング**
- 無効なファイル
- サポートされていない形式
- 不完全なXML
- モデルバリデーションエラー

### 🔧 テストの種類

#### **単体テスト**
- 各モジュールの個別機能をテスト
- モックデータを使用
- 高速実行

#### **結合テスト**
- 複数モジュールの連携をテスト
- モックファイルを使用
- 実際の解析処理をテスト

#### **エラーハンドリングテスト**
- 異常系の処理をテスト
- エラー発生時の動作を確認

## テストデータ

### モックファイル
`tests/assets/mock_workbook.twb`には以下の要素が含まれています：

- **データソース**: 1個
  - 通常フィールド: 2個
  - 計算フィールド: 1個
  - セット: 1個
  - データソースフィルタ: 1個

- **パラメータ**: 1個

- **ワークシート**: 1個
  - 行・列の配置
  - フィルタ設定

- **ダッシュボード**: 1個
  - サイズ設定
  - ワークシート配置

## 注意事項

### 実行順序
1. まず`test_basic_integration.py`を実行して基本機能を確認
2. 次に`test_workbook_parser.py`でパーサーの基本機能を確認
3. 必要に応じて他のテストを実行

### エラーが発生した場合
- モックファイルのXML形式を確認
- モデルのフィールド定義を確認
- パーサーの実装を確認

### テストの追加
新しい機能を追加する際は、対応するテストも追加してください：
1. 単体テスト（該当するモジュールのテストファイル）
2. 結合テスト（`test_basic_integration.py`）
3. エラーハンドリングテスト（必要に応じて）

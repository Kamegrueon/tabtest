# tabtest 開発ガイド

## 開発環境のセットアップ

### 前提条件

- Python 3.8以上
- uv（パッケージマネージャー）

### 環境構築

```bash
# リポジトリのクローン
git clone <repository-url>
cd tabtest

# 依存関係のインストール
uv sync

# 開発環境でのインストール
uv pip install -e .
```

### 開発用ツールのインストール

```bash
# 開発用依存関係の追加
uv add --dev pytest-cov mypy ruff pre-commit

# pre-commitの設定
pre-commit install

# コードフォーマッターの設定
ruff format tabtest/ tests/
```

## コーディング規約

### Python コーディング規約

- **PEP 8**に準拠
- **型ヒント**を必ず使用
- **docstring**を日本語で記述
- **変数名・関数名**は英語で記述

### ファイル命名規則

- **クラス**: `PascalCase`（例：`WorkbookParser`）
- **関数・変数**: `snake_case`（例：`parse_workbook`）
- **定数**: `UPPER_SNAKE_CASE`（例：`DEFAULT_ENCODING`）
- **ファイル名**: `snake_case`（例：`workbook_parser.py`）

### インポート順序

```python
# 標準ライブラリ
import xml.etree.ElementTree as ET
from typing import List, Optional

# サードパーティライブラリ
import pytest
from pydantic import BaseModel

# ローカルインポート
from tabtest.models.workbook import WorkbookModel
from tabtest.parser.xml_loader import XMLLoader
```

## pre-commit

### 概要

pre-commitは、コミット前に自動的にコード品質チェックを実行するツールです。以下のフックが設定されています：

- **ruff**: リンターとフォーマッター
- **mypy**: 型チェック

### 設定

```bash
# pre-commitのインストール
pre-commit install

# 手動で全ファイルをチェック
pre-commit run --all-files

# 特定のフックのみ実行
pre-commit run ruff
pre-commit run mypy
```

### 設定ファイル

`.pre-commit-config.yaml`でフックの設定を管理しています：

```yaml
repos:
  - repo: https://github.com/astral-sh/ruff-pre-commit
    rev: v0.12.0
    hooks:
      - id: ruff
        args: ["check", "--diff", "--fix"]
      - id: ruff-format
        args: ["--diff"]

  - repo: https://github.com/pre-commit/mirrors-mypy
    rev: v1.16.1
    hooks:
      - id: mypy
```

### 使用方法

```bash
# pre-commitフックのインストール
pre-commit install

# 全ファイルでpre-commitフックを実行
pre-commit run --all-files

# 詳細出力でpre-commitフックを実行
pre-commit run --all-files --verbose

# 特定のフックのみ実行
pre-commit run ruff
pre-commit run mypy
```

## テスト開発

### テストファイルの構造

```python
"""テストファイルの説明."""

import pytest
from tabtest.models.workbook import WorkbookModel


class TestWorkbookModel:
    """WorkbookModelのテストクラス."""

    def test_workbook_creation(self) -> None:
        """ワークブック作成のテスト."""
        workbook = WorkbookModel(
            name="Test Workbook",
            datasources=[],
            sheets={},
            dashboards=[],
            parameters=[]
        )
        assert workbook.name == "Test Workbook"

    def test_workbook_invalid_data(self) -> None:
        """無効なデータでのテスト."""
        with pytest.raises(ValidationError):
            WorkbookModel()  # 必須フィールドが不足
```

### テスト実行

```bash
# 全テストの実行
uv run pytest

# 特定のテストファイル
uv run pytest tests/test_workbook_parser.py

# 特定のテストクラス
uv run pytest tests/test_workbook_parser.py::TestWorkbookParser

# 特定のテストメソッド
uv run pytest tests/test_workbook_parser.py::TestWorkbookParser::test_parse_workbook

# カバレッジ付きでテスト実行
uv run pytest --cov=tabtest --cov-report=html

# 並列実行
uv run pytest -n auto

# 詳細出力でテスト実行
uv run pytest -v

# 失敗したテストのみ再実行
uv run pytest --lf
```

### フィクスチャの使用

```python
import pytest
from tabtest.suite.fixtures import workbook, workbook_parser


def test_with_workbook_fixture(workbook):
    """ワークブックフィクスチャを使用したテスト."""
    assert workbook is not None
    assert len(workbook.datasources) > 0


@pytest.mark.parametrize("workbook_parser", ["./path/to/your/workbook.twbx"], indirect=True)
def test_with_workbook_parser_fixture(workbook_parser):
    """ワークブックパーサーフィクスチャを使用したテスト."""
    assert workbook_parser.workbook is not None
    assert workbook_parser.workbook.name == "Expected Name"
```

## 新しい機能の追加

### 1. データモデルの追加

```python
# tabtest/models/new_model.py
from typing import Optional
from pydantic import BaseModel, Field


class NewModel(BaseModel):
    """新しいモデルの説明."""

    name: str = Field(description="モデル名")
    description: Optional[str] = Field(None, description="説明")

    def some_method(self) -> str:
        """メソッドの説明."""
        return f"Processed: {self.name}"
```

### 2. パーサーの追加

```python
# tabtest/parser/new_parser.py
import xml.etree.ElementTree as ET
from typing import Optional

from tabtest.models.new_model import NewModel


class NewParser:
    """新しいパーサーの説明."""

    @staticmethod
    def parse_new_element(element: ET.Element) -> Optional[NewModel]:
        """XML要素からNewModelを解析."""
        name = element.get("name")
        if not name:
            return None

        description = element.get("description")

        return NewModel(name=name, description=description)
```

### 3. テストの追加

```python
# tests/test_new_parser.py
import xml.etree.ElementTree as ET
import pytest

from tabtest.parser.new_parser import NewParser


class TestNewParser:
    """NewParserのテスト."""

    def test_parse_valid_element(self) -> None:
        """有効な要素の解析テスト."""
        xml_str = '<element name="test" description="test desc" />'
        element = ET.fromstring(xml_str)

        result = NewParser.parse_new_element(element)

        assert result is not None
        assert result.name == "test"
        assert result.description == "test desc"

    def test_parse_invalid_element(self) -> None:
        """無効な要素の解析テスト."""
        xml_str = '<element description="test desc" />'
        element = ET.fromstring(xml_str)

        result = NewParser.parse_new_element(element)

        assert result is None
```

## デバッグ

### ログの使用

```python
import logging

logger = logging.getLogger(__name__)

def some_function():
    logger.debug("デバッグ情報")
    logger.info("情報メッセージ")
    logger.warning("警告メッセージ")
    logger.error("エラーメッセージ")
```

### デバッグ実行

```bash
# デバッグモードでテスト実行
uv run pytest -s -v

# 特定のテストをデバッグ実行
uv run pytest -s -v tests/test_workbook_parser.py::TestWorkbookParser::test_parse_workbook

# 失敗したテストのみ実行
uv run pytest -x

# 特定のテストパターンで実行
uv run pytest -k "test_parse"
```

## パフォーマンス最適化

### プロファイリング

```bash
# cProfileを使用したプロファイリング
uv run python -m cProfile -o profile.stats examples/basic_usage.py

# 結果の表示
uv run python -c "import pstats; p = pstats.Stats('profile.stats'); p.sort_stats('cumulative').print_stats(10)"
```

### メモリ使用量の監視

```python
import tracemalloc

tracemalloc.start()

# 処理実行
workbook = WorkbookParser("large_workbook.twb").workbook

current, peak = tracemalloc.get_traced_memory()
print(f"現在のメモリ使用量: {current / 1024 / 1024:.1f} MB")
print(f"ピークメモリ使用量: {peak / 1024 / 1024:.1f} MB")

tracemalloc.stop()
```

## リリース準備

### バージョン管理

`bump2version`を使用してバージョン管理を自動化します。

#### 基本的な使用方法

```bash
# バージョンの更新
uv run bump2version patch  # パッチバージョン (0.1.0 → 0.1.1)
uv run bump2version minor  # マイナーバージョン (0.1.0 → 0.2.0)
uv run bump2version major  # メジャーバージョン (0.1.0 → 1.0.0)
```

#### 高度な使用方法

```bash
# ドライラン（実際には変更しない）
uv run bump2version --dry-run --verbose patch

# ワーキングディレクトリがクリーンでない場合
uv run bump2version --allow-dirty patch

# 特定のファイルのみ更新
uv run bump2version --new-version 1.0.0
```

#### 更新されるファイル

- `pyproject.toml`の`version`フィールド
- `tabtest/__init__.py`の`__version__`
- `.bumpversion.cfg`の`current_version`

#### セマンティックバージョニング

- **patch**: バグ修正（後方互換性のある修正）
- **minor**: 新機能追加（後方互換性のある新機能）
- **major**: 破壊的変更（後方互換性のない変更）

### テストスイートの実行

```bash
# 全テストの実行
uv run pytest

# カバレッジチェック
uv run pytest --cov=tabtest --cov-fail-under=80

# pre-commitフックを実行（推奨）
pre-commit run --all-files

# 個別のツールを実行
uv run mypy tabtest/
uv run ruff check tabtest/ tests/
uv run ruff format tabtest/ tests/

# テストファイルの一覧
# - tests/test_basic_integration.py: 基本的な統合テスト
# - tests/test_integration.py: 統合テスト
# - tests/test_models.py: モデルの単体テスト
# - tests/test_parsers.py: パーサーの単体テスト
# - tests/test_error_handling.py: エラーハンドリングテスト
# - tests/test_suite_helpers.py: ヘルパー関数のテスト
# - tests/test_workbook_parser.py: ワークブックパーサーのテスト
```

### ドキュメントの更新

1. `docs/README.md`の更新
2. `docs/API_REFERENCE.md`の更新
3. `docs/er_diagram.md`の更新（必要に応じて）
4. `CHANGELOG.md`の更新

## トラブルシューティング

### よくある問題

#### 1. インポートエラー

```bash
# 仮想環境の確認
uv run which python

# 依存関係の再インストール
uv sync --reinstall

# 開発環境でのインストール
uv pip install -e .
```

#### 2. テストが失敗する

```bash
# テストの詳細出力
uv run pytest -v -s

# 特定のテストのみ実行
uv run pytest -k "test_name"

# 失敗したテストのみ再実行
uv run pytest --lf
```

#### 3. 型チェックエラー

```bash
# pre-commitの詳細出力
pre-commit run --all-files --verbose

# mypyの詳細出力
uv run mypy --show-error-codes tabtest/

# ruffの詳細出力
uv run ruff check --show-source tabtest/

# 特定のファイルのみチェック
uv run mypy tabtest/models/workbook.py
uv run ruff check tabtest/models/workbook.py

# フォーマットの適用
uv run ruff format tabtest/ tests/
```

### サポート

問題が解決しない場合は、以下を確認してください：

1. **ログファイル**の確認
2. **依存関係のバージョン**の確認
3. **Pythonバージョン**の確認
4. **環境変数**の確認

## 貢献ガイドライン

### プルリクエストの作成

1. **ブランチの作成**: `feature/new-feature`または`fix/bug-fix`
2. **変更の実装**: 機能追加またはバグ修正
3. **テストの追加**: 新しい機能に対するテスト
4. **ドキュメントの更新**: 必要に応じてドキュメントを更新
5. **プルリクエストの作成**: 詳細な説明を含む

### レビュープロセス

1. **自動テスト**: CI/CDパイプラインでの自動テスト
2. **コードレビュー**: チームメンバーによるレビュー
3. **承認**: レビュー承認後のマージ

### コミットメッセージの規約

```
feat: 新しい機能の追加
fix: バグ修正
docs: ドキュメントの更新
style: コードスタイルの修正
refactor: リファクタリング
test: テストの追加・修正
chore: その他の変更
```

例：
```
feat: 新しいデータソースパーサーの追加

- XML要素からデータソース情報を解析する機能を追加
- 接続情報、フィールド、フィルターの解析をサポート
- テストケースとドキュメントを追加
```

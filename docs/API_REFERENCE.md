# tabtest API リファレンス

## パーサー

### WorkbookParser

Tableauワークブックファイルを解析するメインクラス。

```python
from tabtest.parser.workbook_parser import WorkbookParser

parser = WorkbookParser("path/to/workbook.twb")
workbook = parser.workbook
```

#### メソッド

- `__init__(file_path: str)`: ワークブックファイルを読み込み、解析を実行
- `parse()`: ワークブックの解析を実行し、WorkbookModelを返す

### DatasourceParser

データソースの解析を行うクラス。

```python
from tabtest.parser.datasource_parser import DatasourceParser

datasource = DatasourceParser.parse_datasource(element)
```

#### メソッド

- `parse_datasource(element: ET.Element) -> DatasourceModel`: XML要素からデータソースを解析
- `parse_parameter(element: ET.Element) -> ParameterModel`: パラメータを解析
- `_parse_calculated_field(element: ET.Element) -> CalculatedFieldModel`: 計算フィールドを解析

### WorksheetParser

ワークシートの解析を行うクラス。

```python
from tabtest.parser.worksheet_parser import WorksheetParser

worksheet = WorksheetParser.parse_worksheet(element, datasources)
```

#### メソッド

- `parse_worksheet(element: ET.Element, datasources: List[DatasourceModel]) -> WorksheetModel`: ワークシートを解析

### DashboardParser

ダッシュボードの解析を行うクラス。

```python
from tabtest.parser.dashboard_parser import DashboardParser

dashboard = DashboardParser.parse_dashboard(element, sheets)
```

#### メソッド

- `parse_dashboard(element: ET.Element, sheets: Dict[str, WorksheetModel]) -> DashboardModel`: ダッシュボードを解析
- `_safe_int(value: Any) -> int`: 安全な整数変換

## データモデル

### WorkbookModel

ワークブック全体を表すモデル。

```python
from tabtest.models.workbook import WorkbookModel

workbook = WorkbookModel(
    name="My Workbook",
    datasources=[...],
    sheets={...},
    dashboards=[...],
    parameters=[...]
)
```

#### プロパティ

- `name: str`: ワークブック名
- `version: str`: バージョン
- `author: str`: 作成者
- `created_date: str`: 作成日
- `modified_date: str`: 更新日
- `description: str`: 説明
- `datasources: List[DatasourceModel]`: データソース一覧
- `sheets: Dict[str, WorksheetModel]`: ワークシート一覧
- `dashboards: List[DashboardModel]`: ダッシュボード一覧
- `parameters: List[ParameterModel]`: パラメータ一覧

#### メソッド

- `get_datasource(name: str) -> Optional[DatasourceModel]`: 名前でデータソースを取得
- `get_datasource_by_caption(caption: str) -> Optional[DatasourceModel]`: キャプションでデータソースを取得
- `get_sheet(name: str) -> Optional[WorksheetModel]`: 名前でワークシートを取得
- `get_dashboard(name: str) -> Optional[DashboardModel]`: 名前でダッシュボードを取得
- `get_parameter(name: str) -> Optional[ParameterModel]`: 名前でパラメータを取得
- `get_all_parameters() -> List[ParameterModel]`: 全パラメータを取得
- `find_sheets_by_datasource(datasource_name: str) -> List[WorksheetModel]`: データソースを使用するワークシートを検索

### DatasourceModel

データソースを表すモデル。

```python
from tabtest.models.datasource import DatasourceModel

datasource = DatasourceModel(
    name="main_datasource",
    caption="Main Data Source",
    fields=[...],
    sets=[...],
    datasource_filters=[...]
)
```

#### プロパティ

- `name: str`: データソース名
- `caption: str`: キャプション
- `connection_type: str`: 接続タイプ
- `server: str`: サーバー
- `database: str`: データベース
- `schema_name: str`: スキーマ名
- `table: str`: テーブル名
- `query: str`: クエリ
- `fields: List[DatasourceFieldModel]`: フィールド一覧
- `sets: List[SetModel]`: セット一覧
- `datasource_filters: List[DataSourceFilterModel]`: データソースフィルター一覧

#### メソッド

- `get_field(name: str) -> Optional[DatasourceFieldModel]`: 名前でフィールドを取得
- `get_calculated_field(name: str) -> Optional[CalculatedFieldModel]`: 名前で計算フィールドを取得
- `get_set(name: str) -> Optional[SetModel]`: 名前でセットを取得
- `get_datasource_filters_with_expression(expression: str) -> List[DataSourceFilterModel]`: 式でフィルターを検索
- `get_datasource_filters_with_member(member: str) -> List[DataSourceFilterModel]`: メンバーでフィルターを検索

### WorksheetModel

ワークシートを表すモデル。

```python
from tabtest.models.worksheet import WorksheetModel

worksheet = WorksheetModel(
    name="main_sheet",
    datasource_name="main_datasource",
    mark_type="bar",
    title="Main Chart"
)
```

#### プロパティ

- `name: str`: ワークシート名
- `datasource_name: str`: 使用するデータソース名
- `mark_type: str`: マークタイプ
- `mark_color: str`: マーク色
- `mark_size: str`: マークサイズ
- `mark_shape: str`: マーク形状
- `mark_label: str`: マークラベル
- `mark_tooltip: str`: マークツールチップ
- `title: str`: タイトル
- `subtitle: str`: サブタイトル
- `caption: str`: キャプション
- `show_title: bool`: タイトル表示フラグ
- `show_subtitle: bool`: サブタイトル表示フラグ
- `show_caption: bool`: キャプション表示フラグ

### DashboardModel

ダッシュボードを表すモデル。

```python
from tabtest.models.dashboard import DashboardModel

dashboard = DashboardModel(
    name="main_dashboard",
    title="Main Dashboard",
    size_width=1200,
    size_height=800
)
```

#### プロパティ

- `name: str`: ダッシュボード名
- `title: str`: タイトル
- `subtitle: str`: サブタイトル
- `show_title: bool`: タイトル表示フラグ
- `show_subtitle: bool`: サブタイトル表示フラグ
- `size_width: int`: 幅
- `size_height: int`: 高さ
- `sheets: List[DashboardSheetModel]`: シート一覧
- `objects: List[DashboardObjectModel]`: オブジェクト一覧
- `actions: List[DashboardActionModel]`: アクション一覧
- `zones: List[DashboardZoneModel]`: ゾーン一覧

### ParameterModel

パラメータを表すモデル。

```python
from tabtest.models.parameters import ParameterModel

parameter = ParameterModel(
    name="my_parameter",
    caption="My Parameter",
    datatype="string",
    default_value="default"
)
```

#### プロパティ

- `name: str`: パラメータ名
- `caption: str`: キャプション
- `datatype: str`: データ型
- `role: str`: 役割
- `param_domain_type: str`: ドメインタイプ
- `default_value: str`: デフォルト値
- `current_value: str`: 現在の値
- `allow_null: bool`: NULL許可フラグ
- `formula: str`: 計算式

### CalculatedFieldModel

計算フィールドを表すモデル。

```python
from tabtest.models.calculated_fields import CalculatedFieldModel

calculated_field = CalculatedFieldModel(
    name="my_calc",
    caption="My Calculation",
    datatype="string",
    formula="[field1] + [field2]"
)
```

#### プロパティ

- `name: str`: 計算フィールド名
- `caption: str`: キャプション
- `datatype: str`: データ型
- `role: str`: 役割
- `type: str`: タイプ
- `formula: str`: 計算式

### SetModel

セットを表すモデル。

```python
from tabtest.models.sets import SetModel

set_model = SetModel(
    name="my_set",
    caption="My Set",
    field_name="category",
    set_type="manual"
)
```

#### プロパティ

- `name: str`: セット名
- `caption: str`: キャプション
- `field_name: str`: 対象フィールド名
- `set_type: str`: セットタイプ
- `formula: str`: 計算式

## テストスイート

### fixtures

テスト用のフィクスチャを提供。

```python
from tabtest.suite.fixtures import workbook_fixture

def test_workbook(workbook_fixture):
    workbook = workbook_fixture
    # テストコード
```

### helpers

テスト用のヘルパー関数を提供。

```python
from tabtest.suite.helpers import create_mock_workbook

workbook = create_mock_workbook()
```

## エラーハンドリング

### 一般的な例外

- `FileNotFoundError`: ファイルが見つからない場合
- `ValueError`: サポートされていないファイル形式の場合
- `ValidationError`: データモデルのバリデーションエラー

### エラーハンドリングの例

```python
from tabtest.parser.workbook_parser import WorkbookParser
import pytest

def test_error_handling():
    # ファイルが見つからない場合
    with pytest.raises(FileNotFoundError):
        WorkbookParser("nonexistent_file.twb")

    # サポートされていない形式
    with pytest.raises(ValueError, match="Unsupported file type"):
        WorkbookParser("test.txt")
```

- Source
    - **freshness**
        チェックコマンド：```dbt source freshness```
    - 鮮度とは ソースデータの最終ロード日時のカラムを指定し、そのカラム値が一定時間更新されていない状態を監視する機能 です
    - この機能を使えば、データローダーがエラーで止まっているなど障害を検知し、データパイプラインの品質を監視することができます

    version: 2

sources:
  - name: staging
    schema: raw
    freshness: # default freshness
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}
    loaded_at_field: loaded_at
・・・

        上記の例では、 「 loaded_at カラムが12時間更新されていなければ警告、 24時間更新されていなければエラー」 という設定になっています。
        一般的なELTツールはデータロード日時を記録するカラムを付与しますので、それを load_at_field に設定しましょう。
        freshness はプロジェクトごとのデータ品質要件に準じて設定しましょう。
        (上記の例では staging ソース全体に鮮度を設定しましたが、下記のようにテーブルごとに設定することも出来ます。)

- Snapshot

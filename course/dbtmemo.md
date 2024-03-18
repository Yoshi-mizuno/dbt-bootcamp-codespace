- Source
    - **freshness**
        チェックコマンド：`dbt source freshness`
        - 鮮度とは ソースデータの最終ロード日時のカラムを指定し、そのカラム値が一定時間更新されていない状態を監視する機能 です
    - この機能を使えば、データローダーがエラーで止まっているなど障害を検知し、データパイプラインの品質を監視することができます

```yml
    version: 2

sources:
  - name: staging
    schema: raw
    freshness: # default freshness
      warn_after: {count: 12, period: hour}
      error_after: {count: 24, period: hour}
    loaded_at_field: loaded_at
・・・
```

        上記の例では、 「 loaded_at カラムが12時間更新されていなければ警告、 24時間更新されていなければエラー」 という設定になっています。
        一般的なELTツールはデータロード日時を記録するカラムを付与しますので、それを load_at_field に設定しましょう。
        freshness はプロジェクトごとのデータ品質要件に準じて設定しましょう。
        (上記の例では staging ソース全体に鮮度を設定しましたが、下記のようにテーブルごとに設定することも出来ます。)

- **Snapshot**
  cmd：```dbt snapshot```

  - スナップショット(snapshot)は dbt の強力な機能の一つです。データ分析をしていると、「あの時点のデータに戻りたい」と思うことがしばしばあります

(ymlファイルの例)
```sql
  {% snapshot orders_snapshot %}

{{
  config(
    target_database='analytics',
    target_schema='snapshots',
    unique_key='id',
    strategy='timestamp',＃'Check'コマンドも存在する
    updated_at='updated_at',
    invalidate_hard_deletes=True #物理削除を検知するか？
  )
}}

select * from {{ source('raw', 'hoge') }}

{% endsnapshot %}
```

下記のようなデータがスナップショットとして保存される。
1行目はデータが変更され、2行目が現在の最新であることを示している。

d	status	updated_at	dbt_valid_from	dbt_valid_to
1	todo	2022-01-01	2022-01-01	2022-01-02
1	doing	2022-01-02	2022-01-02	null

- **Tests** //データ品質検査
  ```dbt test```

  -下記がUdemyコースで作成したymlファイル。注意点としてはymlファイルの名称は`schema.yml`とする点。
   `unique`,`not_null`, `accepted_values`,`relationships`のテストが標準機能だと可能。


```yml
  version : 2

models:
  - name : dim_listings_cleansed
    columns: 

      - name: listing_id
        tests:
          - unique
          - not_null

      - name : host_id
        tests:
          -  not_null
          -  relationships:
              to: ref('dim_hosts_cleansed')
              field: host_Id

      - name : room_type
        tests:
          - accepted_values:
             values: ['Entire home/apt',
                      'Private room',
                      'Shared room',
                      'Hotel room']
```

-**Macros, custom test**



  ```sql

  ```

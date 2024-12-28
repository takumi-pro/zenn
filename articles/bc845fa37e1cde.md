---
title: "データ指向アプリケーションデザイン#7章まとめ"
emoji: "🐗"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["トランザクション", "ACID", "分離レベル"]
published: true
---

[データ指向アプリケーションデザイン](https://www.amazon.co.jp/%E3%83%87%E3%83%BC%E3%82%BF%E6%8C%87%E5%90%91%E3%82%A2%E3%83%97%E3%83%AA%E3%82%B1%E3%83%BC%E3%82%B7%E3%83%A7%E3%83%B3%E3%83%87%E3%82%B6%E3%82%A4%E3%83%B3-%E2%80%95%E4%BF%A1%E9%A0%BC%E6%80%A7%E3%80%81%E6%8B%A1%E5%BC%B5%E6%80%A7%E3%80%81%E4%BF%9D%E5%AE%88%E6%80%A7%E3%81%AE%E9%AB%98%E3%81%84%E5%88%86%E6%95%A3%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E8%A8%AD%E8%A8%88%E3%81%AE%E5%8E%9F%E7%90%86-Martin-Kleppmann/dp/4873118700)の7章「トランザクション」を読んだので、そのまとめ

---

- システムはネットーワークの断絶やレース条件によるバグなどのあらゆる障害に対処する必要がある。
- これら全ての問題に対する耐障害性の仕組みを実装するのは、大量の考慮点やテストが必要になる。
  - 上記の問題を単純化するためにあるのがトランザクション
- NoSQLではデフォルトでレプリケーションやパーティショニングが提供されており、トランザクションが犠牲となった
    - トランザクションはスケーラビリティとトレードオフの関係にある

## ACIDの意味
- トランザクションは安全の保証を提供する
- それらは**ACID**で示される
  - 原子性
  - 一貫性
  - 分離性
  - 永続性
- ACIDの実装はデータベースごとに異なる
- ACIDの各項目が実際に何を保証するのか曖昧な部分が多いため、それぞれのデータベースでACIDがなんの保証を提供するのか確認が必要
- ACIDを満たさないシステムではBASEと呼ばれる
  - Basically Available - 基本的に利用可能
  - Soft state - 厳密ではない状態遷移
  - Eventual consistency - 結果整合性

### 原子性
- 意味のある処理単位が完了するか全く実行されないかを保証する
- 処理途中（コミット前）にアプリケーションがクラッシュした場合はその処理の書き込みは全て破棄・取り消される

### 一貫性
- ACIDにおける一貫性は「データについて常に真でなければならない」ということ
- 一貫性は他のACIDと異なり、データベースによる保証ができない
  - 例えば
  - アプリケーションバリデーション不足でデータベースに不正なデータ挿入されてしまった場合など
  - start_hour（int）、end_hour（int）カラムがある場合に開始時と終了時が逆転しているパターンはデータベースで違反を検知できない
- なので、一貫性はデータベースの特性ではなくアプリケーションの特性と言える

### 分離性
- 同時実行されているトランザクションがお互いに影響することがないことを保証する
- 同時に同じデータを更新しあうとレース条件が生じるため、分離性で対処
- 完全に直列にするとパフォーマンスの問題があるため、多くのデータベースでは弱い分離を提供している
- 弱い分離
  - repeatable read
  - serializable
  - read committed
  - read uncommitted

### 永続性
- コミットに成功したらそのトランザクションで書き込んだデータは失われることがないこと

## 単一オブジェクトへの書き込み
- 単一のオブジェクト（レコード）が変更される場合にも原子性・分離性は適用される
- 例えば
  - 10KBのJSON型のデータが5KBの送信に成功した直後にネットワーク接続が切断されたら5KBだけ保存された状態になってしまうのか？
  - ほとんどのデータベースは上記のようなパターンに対処するために単一オブジェクトの原子性と分離性を提供している

## エラーと中断の処理
- トランザクションはエラーが発生したら処理が中断され、リトライできる機能がある
  - 普段はActiveRecordを使ってアプリケーション開発をしている立場だが、この機能はあまり聞かない
  - ActiveRecordやDjangoなどのORMにはリトライ機能は備わっていない

## 弱い分離
- トランザクション同士が並行に実行され同じデータにアクセスするとレース条件が問題となる
- レース条件はトランザクション分離性によって改善される
- トランザクションを直列に実行できればレース条件の問題は起きないが、パフォーマンスに問題があるためある種の並行性の問題に対する対処をするのが弱い分離
- 齢分離では並行性の問題を全て解消できるわけではないのでどのような問題があるのか理解しておく必要はある

### read committed

- 以下のことを保証する分離性
  - コミットされる前のデータを読み取りしない（ダーティリードが発生しない）
  - コミットされる前のデータを上書きしない（ダーティライトが発生しない）
- read committedの実装はロックが用いられる
  - トランザクション1が書き込みをする対象のオブジェクトのロックを取得すると、トランザクション2がそのオブジェクトに書き込みをするにはトランザクション1がコミットまたは中断されるのを待つしかない
    - これでダーティライトを防げる
  - 書き込みが行われたオブジェクトに対して、前回のコミット値とトランザクション1で設定された値を持っておいて、そのオブジェクトを読み取るトランザクション2には古い値（前回のコミット値）を返す
    - これでダーティリードを防げる

### nonrepeatable read（読み取りスキュー）
- nonrepeatable readは「読んだ内容が繰り返し読むたびに変わってしまう」現象
- 同じトランザクション内で同じデータにアクセスしたが、内容が変わっているということが起きてしまう
- 例
  - トランザクション1でsalaryテーブルで従業員Aの給料を読み取る→10万円
  - トランザクション2がsalaryテーブルの従業員Aの給料を10万円から12万円に更新してコミットする
  - トランザクション1で再度従業員Aの給料を読み取る→12万円
- 例で示したようにトランザクション2はコミット済みなので、ダーティリードではない
- なので、read committedの分離レベルでは防げないパターン
- スナップショット分離がこのパターンの解決策
  - トランザクションが一貫性のある（コミット済みの値）スナップショットから読み取りを行う
  - PotgreSQLやMySQLで利用されている
- repeatable read分離レベルとスナップショット分離の違い（ChatGPTに聞いた結果）
  - repeatable read
    - SQLの標準として定義されている
    - nonrepeatable readを防ぐ
    - 明確な実装方法はない？
  - スナップショット分離
    - **MVCC（Multi-Version Concurrency Control）** を活用する
    - 実装方法が決まっている（ある時点でのスナップショットを活用する）
  - PostgreSQLやMySQLではrepeatable read分離レベルでスナップショット文理が使われているため、ほぼイコール？

### 書き込みスキュー

- read committedやスナップショット分離でも防げないのが書き込みスキュー
- 2つのトランザクションがそれぞれ同じあるいは別々のオブジェクトを更新した場合に生じる不整合が書き込みスキュー
- 書き込みスキューを生じさせるファントム
  - ファントムリードとは
    - 同じトランザクション内で同じクエリを2回以上実行した際に、前回の結果にはなかった行が出現したり、あった行が消えているという現象
  - 書き込みスキューによるファントムはスナップショット分離では防ぐことができない

### 直列化可能性

- 最も強い分離レベル
- トランザクション同士が並行に実行されることなく順番に一つずつ実行される
- 書き込みスキューやファントムなどのレース条件の問題を解決できる
- 実現方法
    - 順次実行
    - 2相ロック（two phase lock）
    - 直列化スナップショット分離（SSI、serializable snapshot isolation）
# SpringBatch

## SpringBatch の全体像

バッチ処理とは、「一定量のデータを集め、一括自動処理する」ための処理方法のこと。

### 特徴

1. 定期コミット  
   バッチ処理は大量データを処理するので、処理し終えるまでに長時間かかることが予想される。途中でエラーが発生したときに、すべての処理をやり直すことにならないように、Spring Batch ではバッチ処理を定期的にコミットし、処理をリスタートする仕組みを構築できる
2. 並列、直列処理  
   Job（1 つのバッチ処理のこと）を構成する処理のことを Step と呼ぶ。Spring Batch は Step の並列実行、直列処理を設定ベースで柔軟に実装できる。並列実行することにより、大量データに対して処理スループットが向上する
3. リトライ  
   大量データを扱うバッチ処理が異常終了したとき、エラーデータの修復後、エラーデータから処理を簡単にリトライできる
4. その他  
   SpringBatch の機能ではないが、Spring Boot の機能である Schedule アノテーションを利用することで、任意の時間でバッチを実行できます。この使われ方もよく行われる。

## 構造

## Listener

リスナーとは、ジョブやステップを実行する前後に処理を挿入するためのインターフェースである。
BatchConfigでstepやjobを生成するときにどのListenerを使用するか指定する

- JobListener  
   ジョブの実行に対して処理を挟み込むためのインターフェース。
   JobListenerのインターフェースは、`JobExecutionListener`の1つのみ。
- StepListener  
   ステップの実行に対して処理を挟み込むためのインターフェース。`StepExecutionListener`インタフェースの実装クラスを作ることで実現できる。
   また、Step

### Listener の種類

| Listener | 実装 | 処理を挟み込むタイミング |
| --- | -- | - |
|JobListener| JobExecutionListenerインタフェースの実装クラスを作る |Job の実行前後|
|StepListener | StepExecutionListenerインタフェースの実装クラスを作る |Step の実行前後|
|ChunkListener| | 1つのチャンクを処理する前後とエラー発生時|
|ItemReadListener| |ItemReaderが1件のデータを取得する前後とエラー発生|
|ItemProcessListener| | ItemProcessorが1件のデータを加工する前後 の実行前後とエラー発生時|
|ItemWriteListener  | | ItemWriterが1つのチャンクを出力する前後とエラー発生時|
|RetryListener| |Retry の実行前後とエラー発生時|
|SkipListener| | Chunk 内の処理でエラー発生時に、そのデータの処理をスキップできます。スキップする場合の Listener が SkipListener です。Reader、Processor、Writer でスキップした場合に処理を実行できます。|

## その他メモ

JobExecutionにはJobの実行結果の情報が入っている。

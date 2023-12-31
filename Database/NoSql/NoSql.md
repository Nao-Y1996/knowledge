# NoSQL

## NoSQLとは

NoSQL（Not Only SQL）は、伝統的なリレーショナルデータベース（SQLデータベース）の制約を緩和し、大量のデータを扱ったり、様々なデータ形式を扱ったりする際に特に有用なデータベース。RDBではないデータベースの総称。
以下の4つが代表的。

1. Kiey-Value Sotre
2. Column-Oriented Databases
3. Document Database
4. Grapf Databese

### Key-Value Stores

最もシンプルな形式のNoSQLデータベースで、一意のキーとそのキーに関連付けられた値の組み合わせを保存する。高速な読み書きが可能で、データ分散とスケーリングに優れている。ただし、基本的に、キーによるデータの検索しか基本的には提供しないため、複雑なクエリの実行には向いていない。大量のデータを高速に読み書きする必要がある場合や、キーによる単純な検索が主な要件である場合に適している。

例：Redis、Amazon DynamoDB

### Column-Oriented Databases

列単位でデータを保存する。これにより、特定の列に対するクエリを高速に実行することが可能になる。例えば、数百万人のユーザーの年齢の平均を計算するようなケースでは、列指向データベースが有用（RDBではwhere句などを使用しても内部では行が全て読み込まれるため、行が増えるにつれ特定の列に対するクエリは遅くなっていく）。一つの列にあるデータは通常、類似のデータ型を持つため、データ圧縮も効率的に行うことが可能。列指向データベースは、大量のデータを効率的に分析し、特定の列に対する操作を高速に行う必要がある場合に適している。

例：BigTable、Cassandra

### Document Databases

JSON、XML、BSONなどの形式でデータを保存する。各ドキュメントは一意のキーによって識別され、そのキーを使用してドキュメントを取得する。ドキュメントは柔軟なデータ構造を持っているため、異なるフィールドやフィールドの集合を持つことができる。

たとえば、一つのドキュメントが一人のユーザーを表し、そのユーザーの名前、メールアドレス、住所などの情報をフィールドとして持つことができる。さらに、フィールドはネスト（入れ子に）されており、たとえば住所は、都道府県、市町村、番地、といったサブフィールドを持つことができる。

さらに、ドキュメント間でスキーマ（データ構造）が異なってもよい。つまり、一部のユーザードキュメントには電話番号フィールドがあり、一部にはない、という状態を容易に許容することができる。このため、データ構造が頻繁に変わるアプリケーションや、データ間で構造が大きく異なる場合にドキュメントデータベースが有用。

例：MongoDB、CouchDB

### Graph Databases

ノード、エッジ、プロパティ（ノードとエッジのもつ属性情報）により構成されるグラフとしてデータを保存する。ネットワーク、ソーシャルネットワーキング、リコメンデーションエンジン、などの、関連性とパターンが重要な要件である場合に特に有用。また、複雑な階層構造や連結を持つデータを扱うのにも適している。

例：Neo4j、Amazon Neptune

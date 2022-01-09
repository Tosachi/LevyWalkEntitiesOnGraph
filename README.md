# LevywalkEntitiesOnGraph
  
ユニットディスクグラフ上を単一のエンティティが探索するシミュレータである [LevyWalkOnGraph](https://github.com/KentaToshikura/LevyWalkOnGraph) を、複数のエンティティで相互探索できるように改良したシミュレータ。
  
## バージョン  
Java version: 16.0.2  
maven version: 3.8.2  

## ライブラリ
pom.xml に記載  
追記したものは以下  
  
- [Apache Commons Math](https://mvnrepository.com/artifact/org.apache.commons/commons-math3) : 3.6.1
- [Animal Sniffer Annotations](https://mvnrepository.com/artifact/org.codehaus.mojo/animal-sniffer-annotations) : 1.20
- [J2ObjC Annotations](https://mvnrepository.com/artifact/com.google.j2objc/j2objc-annotations) : 1.3
- [Error Prone Annotations](https://mvnrepository.com/artifact/com.google.errorprone/error_prone_annotations) : 2.9.0
- [Apache Commons CLI](https://mvnrepository.com/artifact/commons-cli/commons-cli) : 1.4
- [Guava: Google Core Libraries For Java](https://mvnrepository.com/artifact/com.google.guava/guava) : 30.1.1-jre
  
【graphstream系】  
- [Mbox2](https://mvnrepository.com/artifact/org.graphstream/mbox2) : 1.0
- [pherd](https://mvnrepository.com/artifact/org.graphstream/pherd) : 1.0
- [GS Core](https://mvnrepository.com/artifact/org.graphstream/gs-core) : 1.3
- [GS Algo](https://mvnrepository.com/artifact/org.graphstream/gs-algo) : 1.3
  
## 実行方法  
実行結果はターミナルの出力及び `result/*` に指定したファイル名で出力される。  
  
1. jar ファイルから実行 (シミュレータ自体を用いたい場合)  
  
    `% java -jar levywalk.jar data.txt`  

    `data.txt` は任意のデータファイルを指定することができる
  
2. maven で実行 (コード自体を変更して実行する場合)  
  - Eclipse で実行  
  調べてください  
  
  - maven をインストール後以下のコマンドを実行
    - コンパイル  
      `% mvn compile`  
        
    - 実行  
      `% mvn exec:java -Dexec.args='data.txt'`  
        
      `data.txt` は任意のデータファイルを指定することができる  
        
    - jar 化(ライブラリが内包された jar ファイルが作成される)  
      `% mvn compile assembly:single`  
  

## オプションの説明  
`data.txt` または `src/main/resources/sample.txt` を参照  
| オプション | 説明 | 型 | 備考 |
|:---------|:-----|:-------|:--------|
| trial | 試行回数 | Integer | デフォルト: 1 |
| node | ノード数 | Integer | デフォルト: 1000 |
| threshold | しきい値 | Double | デフォルト: 0.05 |
| graphSeed | グラフ生成用ランダムシード | Long |指定がなければランダムに決定 |
| walkSeed | 探索用ランダムシード | Long | 指定がなければランダムに決定 |
| researchCoverRatio | カバー率の調査(true),到達率の調査(false) | Boolean | デフォルト: true |
| entityClass | LevyWalk もしくは RandomWalk | String | デフォルト: LevyWalk |
| step | ステップ数 | Integer | デフォルト: 1000 |
| entity | エンティティ数 | Integer | デフォルト: 1, 1 <= n < 100 |
| remake | グラフの再構成回数の上限 | Integer | デフォルト: 1000 |
| file | 出力ファイル名(.csv) | String | デフォルト: result.txt |
| permissibleError | 許容誤差 | Double | デフォルト: 20.0 |
| lambda | パラメータ | Double | デフォルト: 1.2, 複数の場合はカンマ区切り空白なし |
| interval | カバー率を調査するstep数の間隔 | Integer | デフォルト: 1000 |
  
## 複数エンティティの相互探索の仕様  
- オプション `entity` に 2以上100以下の **偶数** を指定することで複数の物体が動く。  
- オプション `lambda` に `entity` で指定した数の値を指定可能。  
  
  [例1]
  ```
  entity 4
  lambda 1.2,1.2,3.0,3.0
  ```  
  
- 前半に指定したエンティティと後半に指定したエンティティは同じグループとみなされる。  
  
  [例2]
  ```
  entity 4
  lambda [1.2,1.2],[3.0,3.0]
          ↑group A  ↑group B 
  ```  
  
- 違うグループのエンティティと出会うと、ターミナルに出会ったステップ数、エンティティ番号、lambda 値 が出力される。ファイルには出会ったステップ数が出力される。  
- 一度出会ったエンティティは除かれる。(出会っても出力されない。)  
- 同じグループ同士のエンティティは出会っても出力されない。  
- 違うグループのエンティティはある程度離れた位置からスタートする。同じグループのエンティティは同じエリアの近い位置からスタートする。  
- 出力グラフはエンティティごとの色分けをしてないので複数動かす場合はあまり参考にならない。

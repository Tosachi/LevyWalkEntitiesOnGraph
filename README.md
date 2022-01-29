# LevywalkEntitiesOnGraph
  
ユニットディスクグラフ上を単一のエンティティが Levy walk で探索することができるシミュレータである [LevyWalkOnGraph](https://github.com/KentaToshikura/LevyWalkOnGraph) を、複数のエンティティで相互探索できるように改良したシミュレータ。  
2021年度卒業研究で実装・実験に使用。  
  
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
実行結果はターミナル及び `result/*` にオプションファイルで指定したファイル名で出力される。  
  
1. jar ファイルから実行 (シミュレータ自体を用いたい場合)  
  
    `% java -jar levywalk.jar data.txt`  

    `data.txt` は任意のデータファイルを指定することができる
  
2. maven で実行 (コード自体を変更して実行する場合)  
  - Eclipse で実行  
    調べてください。  
    一応参考：https://reasonable-code.com/eclipse-maven/  
  
  - maven をインストール後以下のコマンドを実行 (`brew install maven` とかでできます)
    - コンパイル  
      `% mvn compile`  
        
    - 実行  
      `% mvn exec:java -Dexec.args='data.txt'`  
        
      `data.txt` は任意のデータファイルを指定することができる  
        
    - jar 化(ライブラリが内包された jar ファイルが src/main/target 作成される)  
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
- オプション `entity` に 2以上100以下の **偶数** を指定することで複数の物体が動く。(奇数でも動くができるだけ偶数)  
- オプション `lambda` に `entity` で指定した数の値を指定可能。  
  
  [例1]
  ```
  entity 4
  lambda 1.2,1.2,3.0,3.0
  ```  
  
- 前半に指定した lambda と後半に指定した lambda は同じグループとみなされる。  
  
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

## クラスの説明
コードはのリポジトリは [src/main/java/levywalk/](https://github.com/nischis/LevyWalkEntitiesOnGraph/tree/main/src/main/java/levywalk) である。  
  
### [Mainクラス](https://github.com/nischis/LevyWalkEntitiesOnGraph/blob/main/src/main/java/levywalk/Main.java)
  入力ファイルを処理して String 型の配列を取得する。
  取得した配列は OptionCli クラスにて Data 型に格納される。
  グラフ生成用の RandomSeed が指定されていなかった場合、このクラスでランダムに生成される。

### [OptionCli クラス](https://github.com/nischis/LevyWalkEntitiesOnGraph/blob/main/src/main/java/levywalk/OptionCli.java)
  配列化された入力値を解析して、入力ファイルで設定した値とオプション名を結びつける。結びつけられたそれぞれの値を Data 型のインスタンスに渡す。
  
### [Data クラス](https://github.com/nischis/LevyWalkEntitiesOnGraph/blob/main/src/main/java/levywalk/Data.java)
  コンストラクタとしてのクラス。メソッドは存在せず、入力ファイルで設定したオプションの値とそのデフォルト値を保持している。詳しくは [オプションの説明](https://github.com/nischis/LevyWalkEntitiesOnGraph#オプションの説明) を参照。

### [RandomWalkOnGraph クラス](https://github.com/nischis/LevyWalkEntitiesOnGraph/blob/main/src/main/java/levywalk/RandomWalkOnGraph.java)
  GraphStream を用いて連結グラフを作成し、そのグラフ上での探索を行う。試行回数が1回の時は、通過点への色付けをしたグラフの描写も行う。
  探索アルゴリズムが Random walk の場合は、GraphStream を用いてこのクラス内で探索を行う。探索用の RandomSeed が設定されていなかった場合、このクラスでランダムに生成される。探索アルゴリズムが Levy walk の場合は、GraphStream を override した init メソッドに値を渡して、 LevyWalkEntity クラスで探索を行う。GraphStream の setEntityCount メソッドにエンティティ数を渡すことで複数のエンティティを動かしている。
  また、ランダムシードやノード数、閾値などの入力値の情報とノードやエッジのカバー率または到達率をコマンドラインと外部ファイルへ出力する。
  
### [LevyWalkEntity クラス](https://github.com/nischis/LevyWalkEntitiesOnGraph/blob/main/src/main/java/levywalk/LevyWalkEntity.java)
  GarphStream の init メソッドと step メソッドを override して、Levy walk での探索を行う。  


  このクラスはそれぞれのエンティティに対してインスタンスが作成され、1ステップずつ並行して実行される。そのため、それぞれのノードが出会ったかどうかを判定するために、全てのエンティティがそのステップで通ったノードID を格納するための配列と、そのステップまでに遭遇済みかどうかを格納するための配列を静的フィールドで作成した。そして、1ステップが終わるごとに、そのステップで辿り着いたノードID と既にそのステップが終わっている別グループのエンティティのノードID をそれぞれ比較する。一致していれば遭遇済みのエンティティとして配列に記録し、探索を終了する。この時、遭遇したステップ数は外部ファイルに出力する。図1 にとあるエンティティの1ステップの処理を図で示した。
   
   また、エンティティがグループ分けされるように、エンティティ番号が半分以下のエンティティと半分以上のエンティティしか遭遇判定を行わないようにしている。図2に示すように、ユニットディスクグラフの座標を取得して開始位置を対角線上の範囲にし、別グループのエンティティとは開始位置が離れるようにした。また、全く同じユニットディスクグラフ上であれば毎回同じ位置から探索が開始されることになる。

![example_hop3.pdf](https://github.com/nischis/LevyWalkEntitiesOnGraph/files/7963309/example_hop3.pdf)  
図1: ホップ長が 3 の場合の 1ステップの処理例  
  
![separate_start.pdf](https://github.com/nischis/LevyWalkEntitiesOnGraph/files/7963310/separate_start.pdf)  
図2: 探索の開始位置が離れていることを示した実行結果(step = 3，lambda = 6)
  
  
## 探索アルゴリズム
このシミュレータでは信貴氏、西田氏が提案しユニットディスクグラフ上での Levy walk のアルゴリズムを元に実装している。ユニットディスクグラフ上での Levy walk では、1ステップで移動するノード数の分布が式(1)の冪分布に従う。  
  
  $$P(l) \propto l^{-\frac{1}{\lambda}} (0.0 < l < 1.0, 1.0 < \lambda \le 3.0)  ・・・(1)$$  
  
まず、ホップ長とホップ方向を取得する。ホップ長は、式(1) の移動距離 l にランダムな値を代入して取得し、ホップ方向はランダムに取得する。現在のノードからホップ方向と隣接ノードの方向の誤差をそれぞれ取得し、その誤差が許容誤差内であれば移動可能とする。そして、最も誤差が小さい隣接ノードへホップする。これをホップ長の回数繰り返して1ステップの移動とする。もしホップ可能なノードがなかった場合は、再度ホップする方向を取得し、それでもホップ不可能であった場合は探索不可能として終了する。
  
# ソート

## 目次

- [ソート](#ソート)
  - [目次](#目次)
  - [基本的なソートの考え方](#基本的なソートの考え方)
  - [Collections を使ったソート](#collections-を使ったソート)
    - [要素の大小関係が明らかなとき](#要素の大小関係が明らかなとき)
    - [大小比較の方法を制御したい時](#大小比較の方法を制御したい時)
    - [Collections.sort のデメリット](#collectionssort-のデメリット)
  - [Stream を使ったソート](#stream-を使ったソート)
    - [3 種類の整数を返す処理でソート](#3-種類の整数を返す処理でソート)
      - [コード](#コード)
      - [解説](#解説)
    - [キーの指定だけでソート](#キーの指定だけでソート)
    - [複数のキーでソート](#複数のキーでソート)
  - [参考書籍](#参考書籍)

## 基本的なソートの考え方

ソートは一言で言うと、「昇順 or 降順で並び替えること」
ここで、「名前」と「年齢」を持つ人オブジェクト並べる際に、「まず名前順で並べて同じなら年齢順で並べる」というルールにすると、単に昇順・降順では並べられないと感じてしまいますが、人オブジェクトの大小関係を決める際に名前が優先されて次に年齢を使うというだけであって、あくまで、人オブジェクトの大小関係に従って昇順・降順に並べていると考えるとシンプルになる

つまり、ソートしたい要素に対して、どのように大小を決定するかを決めてやれば、あとは API を使ってソートができる。

- **【重要】ソートするのに必要なことは、要素の大小関係を決めてあげること**

## Collections を使ったソート

ここではまず、大小関係が明らかなものをソートする方法と、大小関係が明らかでない人オブジェクトをソートする方法をみていく。

どちらも `Stream` が使えるようになる以前によく使われていた、`Collections.sort()`を使って解説をして、`Collections.sort()`のデメリットにも言及します。その後、`Stream`を使ったエレガントなソートの方法を解説していきます。

### 要素の大小関係が明らかなとき

数値や文字列のように大小関係が明らかなときは、大小関係を決めてやらなくてもソートすることができる。また、ソートの結果は自然順序（明らかな大小関係に基づく昇順）になる。

ソートするには Collections.sort()を呼び出すだけ。

```java
List<Integer> numbers = new ArrayList<>(Arrays.asList(5,4,8,3,1));
// ソートを実行
Collections.sort(numbers);
// 結果を出力
for (int num: numbers){
   System.out.print(num + " "); // 1 3 4 5 8
}
```

### 大小比較の方法を制御したい時

次に、age と name をプロパティにもつ Person をソートすることを考える。
Person オブジェクトの年齢差を返す ageDifference メソッドも定義している。

```java
public class Person {

   private final int age;
   private final String name;

   public Person(String name, int age) {
       this.name = name;
       this.age = age;
   }
   public int getAge() {
       return age;
   }

   public String getName() {
       return name;
   }

   @Override
    public String toString() {
        return "Person{" +
                "age=" + age +
                ", name='" + name + '\'' +
                '}';
    }

   public int ageDifference(final Person other) {
       return this.age - other.age;
   }
}
```

Java では、要素の大小比較の方法を自分で決定するには Comparator というインタフェースを利用する。

Comparator は関数型インタフェースで、抽象メソッドとして compare メソッドが定義されている。
cpmpare メソッドの実際の処理に「大小比較の方法」を記述していく。また、この処理はある決まりに従って書く必要がある。具体的には、「2 つの引数をとり、 第 1 引数が第 2 引数より小さいとする場合は負の整数、両方が等しい場合は 0、第 1 引数が第 2 引数より大きいとする場合は正の整数を返す」ような処理となる。
（参考：[Comparator インタフェースの compare メソッド](<https://docs.oracle.com/javase/jp/11/docs/api/java.base/java/util/Comparator.html#compare(T,T)>)）

まず「まず名前順で並べて同じなら年齢順で並べる」というルールで、ソートをする処理は以下のようになる。

```java
List<Person> people = Arrays.asList(
        new Person("John", 20),
        new Person("Sara", 21),
        new Person("Jane", 21),
        new Person("Greg", 35)
);

// compareメソッドをオーバーライドしてPersonオブジェクトの大小比較の方法を定義する
// 比較方法：年齢で比較して同じなら名前で比較
Comparator<Person> personComparator = new Comparator<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        int ageDiff = p1.ageDifference(p2);
        if (ageDiff != 0) {
            return ageDiff;
        } else {
            int nameDiff = p1.getName().compareTo(p2.getName());
            return nameDiff;
        }
    }
};
// 年齢でソートする
// 第1引数：ソート対象のpeople
// 第2引数：Personの大小比較方法が定義されたComparator
Collections.sort(people, personComparator);

// 並びを確認
for (
        Person person : people) {
    System.out.println(person);
}
// ソート結果
// Person{age=20, name='John'}
// Person{age=21, name='Jane'}
// Person{age=21, name='Sara'}
// Person{age=35, name='Greg'}

```

重要なのは Person オブジェクトの大小比較の方法を定義する部分。

ここでは、compare メソッドの実装の決まりに従うと、

2 つの Person オブジェクト p1, p2 を引数とし、以下のような処理にする必要があります

- `p1`が`p2`より**小さい**とする場合は**負の整数**  
- `p1` が `p2` より**大きい**とする場合は**正の整数**  
- 両方が**等しい**場合は **0** を返す  

また、並べ方は「まず名前順で並べて同じなら年齢順で並べる」というルールなので、Person の大小関係と compare メソッドで返す値の対応は以下のようになります。

1. `p1`の年齢が`p2`の年齢より小さければ`p1`は`p2`より**小さい**（**負の整数**を返す）
2. `p1`の年齢が`p2`の年齢より大きければ`p1`は`p2`より**大きい**（**正の整数**を返す）
   もし年齢が同じなら、
3. `p1`の名前が`p2`の名前より小さければ`p1`は`p2`より**小さい**（**負の整数**を返す）
4. `p1`の名前が`p2`の名前より大きければ`p1`は`p2`より**大きい**（**正の整数**を返す）
5. 名前の大小も同じなら、`p1`と`p2`の大小は**同じ**（**0** を返す）

上記のコードの実装では 1,2 は年齢差を返すことで実現している。
3,4,5 は String クラスに用意された compareTo()メソッドを使って実現している。

※補足
String クラスに用意されたcompareTo()は2つの文字列を辞書的に比較するメソッド。
Comparable インタフェースの実装であり、これも、「2 つの引数をとり、 第 1 引数が第 2 引数より小さいとする場合は負の整数、両方が等しい場合は 0、第 1 引数が第 2 引数より大きいとする場合は正の整数を返す」ように実装されている。（参考：[Comparable インタフェースの compareTo メソッド](<https://docs.oracle.com/javase/jp/11/docs/api/java.base/java/lang/Comparable.html#compareTo(T)>)）

### Collections.sort のデメリット

1. コードを見ただけでは何しているか分かりにくい  
   上の実装のようなコードを理解するには、Comparator インタフェースの compare メソッドをどのように実装するかを知ってる必要がある。さらに、正の整数を返すのか負の整数を返すのか常に覚えておくことはなかなか大変。しかも、そのような処理を書く必要があるのでコード自体も長く読みづらいものになってしまう。
2. 元のリストが破壊されてしまう  
   Collectons の sort メソッドは、ソートしたいリストの要素を直接並び替えている。プログラミングの世界では、このようにオブジェクトを直接変更するようなメソッドは破壊的メソッドと呼ばれ、それを使うときは特に注意が必要。（オブジェクトを直接変更してしまうと、後でまたそのオブジェクトを利用する際にソートされているのかされていないのかを意識しなければならず、処理を追うのが大変になりバグが混入しやすくなるため）そのため、元のオブジェクトは変更せず、処理結果の新しいオブジェクトを返すような非破壊的メソッドを利用することが好ましかったりする。

## Stream を使ったソート

次に Stream を使って見やすくかつ簡潔に記述できるソート方法をみていく。

Stream はデータの集まりに対して、さまざまな処理（ソートや絞り込み、各データに対する処理など）を簡潔に記述できるようにするインタフェース。  
Stream を使うことで、正の整数 or 負の整数 or 0 を返す処理（compare メソッドの実装）を意識しなくても柔軟なソートが可能になる。  
ここでは、Stream を使って、これまで通り 3 種類の整数を返す処理を書いてソートするところから始め、コードを改良していきながら、最後に、compare メソッドの実装を意識しなくてもソートできるような方法をみていく。

### 3 種類の整数を返す処理でソート

Comparator を使って大小関係（比較方法）を定義するのは先ほどと同じ。
繰り返しになるが、
「2 つの引数をとり、 第 1 引数が第 2 引数より小さいとする場合は負の整数、両方が等しい場合は 0、第 1 引数が第 2 引数より大きいとする場合は正の整数を返すような処理」を渡す。

#### コード

```java
// Comparatorの定義
Comparator<Person> personComparator = new Comparator<Person>() {
    @Override
    public int compare(Person p1, Person p2) {
        return p1.ageDifference(p2);
    }
};
// stream.sortでソートする
List<People> sortedPeopleByAge =
   people.stream.sorted(personComparator).collect(toList())

// Comparatorはラムダ式を使って処理だけを直接渡すことも可能
// List<People> sortedPeopleByAge =
//    people.stream.sorted((person1, person2) -> pserson1.ageDifference(person2)).collect(toList())
```

名前でソート

```java
// ラムダ式を使用
List<People> sortedPeopleByName =
   people.stream.sorted((person1, person2) -> pserson1.getName().compareTo(person2.getName()))
```

#### 解説

Stream でソートをする場合は sorted メソッドを使用する。
これは非破壊的メソッドなので、ソート結果を新しい変数に格納している。

sorted メソッドの引数には Comparator を入れ、これまで通り、3 種類のどの整数を返のか意識した処理を引数に渡す。

また、Comparator は一度変数に格納してから sorted メソッドに渡すが、ラムダ式を使うと処理だけを直接渡すことも可能（コメントアウト部分）。

`.collect(toList())`は sorted メソッドの戻り値型が `Stream` なので型を`List`に戻しているだけ。

非破壊的メソッドになったのは良いことだが、どんな整数を返すか意識する必要があるので、まだ、それほどメリットがない。次は comparing メソッドを使ってさらに改良する。

なお、大小関係が明らかな場合は、引数なしで使うことで、自然順序（明らかな大小関係に基づく昇順）でソートできる。

（参考：[Stream クラスの sorted メソッド](<https://docs.oracle.com/javase/jp/11/docs/api/java.base/java/util/stream/Stream.html#sorted(java.util.Comparator)>)）

### キーの指定だけでソート

少しずつ改良して、steam のメリットを見ていく。

comparing メソッドは、ソートに使うキーを返す関数を受け取り、そのキーで大小比較する Comparator を返す。

例えば、「Person を受け取って Person の name を返す関数」を受け取ると、Person の name でソートする Comparator を返すことができる。

つまり、「名前の大小関係を使う」ということを指定してあげれば、3 種類の整数を返す Comparator を返してくれる便利なもの。

```java
List<People> sortedPeopleByName =
   people.stream().sorted(comparing((Person person) -> person.getName())).collect(toList()));
```

3 種類の整数を返す処理ではなく、ソートするキーを返す処理を渡すだけになり、考えることが減った。

キーを返す処理自体を定数としてやるともっとコードも見やすくなる。

```java
final Function<Person, String> byName = person -> person.getName();
final Function<Person, String> byAge = person -> person.getAge();

List<Person> sortedPeopleByName =
   people.stream().sorted(comparing(byName));

List<Person> sortedPeopleByAge =
   people.stream().sorted(comparing(byAge));
```

メソッド参照を使うとさらに簡略化できる。

```java
List<People> sortedPeopleByName =
   people.stream().sorted(comparing(Person::getName));

List<People> sortedPeopleByAge =
   people.stream().sorted(comparing(Person::getAge));
```

正の整数を返すんだったかな？負の整数を返すんだったかな？と迷うこともなく、コードも記述しやすく理解しやすいものになった。

（参考：[Comparator インタフェースの comparing メソッド](<https://docs.oracle.com/javase/jp/11/docs/api/java.base/java/util/Comparator.html#comparing(java.util.function.Function)>)）

### 複数のキーでソート

最後に、名前で比較した後に年齢で比較して人オブジェクトをソートしてみる。
Stream を使うとこの処理も非常に簡単に書ける。comparing に続けて thenComparing メソッドを使うだけ。
（参考：[Comparator インタフェースの thenComparing メソッド](<https://docs.oracle.com/javase/jp/11/docs/api/java.base/java/util/Comparator.html#thenComparing(java.util.function.Function)>)）

```java
List<People> sortedPeopleByAgeAndName =
   people.stream().sorted(comparing(Person::getAge).thenComparing(Person::getName)).collect(toList()));
```

## 参考書籍

- Java による関数型プログラミング Java8 ラムダ式と Stream

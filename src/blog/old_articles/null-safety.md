---
  type = "post"
  title = "null 安全についてのポエム"
  post_date = 2016-12-09T11:32:00+09:00
  tags = ["programming language", "poem"]
  description = "ポエムです 愛や気持ちの話をするので話半分くらいに読んで下さい."
---

## ポエムです

愛や気持ちの話をするので話半分くらいに読んで下さい.

また多数の方の気分を不愉快にさせてしまう恐れがあり, その場合はすぐにタブを閉じてください.

## 「null 安全」という言葉

そもそも何が「安全」なのだろうか. 「null が安全」なのか「null から安全」なのか. 

これはおそらく「型安全 (type safety)」の語感から来ているのだと思う. 

[Type safety - Wikipedia](https://en.wikipedia.org/wiki/Type_safety)

> In computer science, type safety is the extent to which a programming language discourages or prevents type errors. 

> 計算機科学において, 型安全性とは, プログラミング言語がどの程度型エラーを (プログラマに) 起こさせないようにするのか, または実際に防止するのか, ということを指す. (私訳)

しかし, [Google で "null safety" で調べ](https://www.google.co.jp/search?q=null+safety)ても Kotlin 関連の記事しか出てこない. つまりは Kotlin で初めて使われた言葉ということになる.

また, 検索結果に [Void safety - Wikipedia](https://en.wikipedia.org/wiki/Void_safety) があった. 

> Void safety is a guarantee within an object-oriented programming language that no object references will have null or void values.

> Void 安全とは, オブジェクト指向言語において, オブジェクトの参照が null や void といった値を持ちえないことが保証されていることを指す. (私訳)

以上から「null 安全」という言葉の意味を推測してみると, ざっくりと「null エラーを防止するために、あるオブジェクトが null でないことが保証されていること」となるだろう.

## null とは何だっけ？

この場合の null というのは [Null pointer - Wikipedia](https://en.wikipedia.org/wiki/Null_pointer) のことである.

> In computing, a null pointer has a value reserved for indicating that the pointer does not refer to a valid object.

> プログラミングにおいて, ヌルポインタとはポインタが有効なオブジェクトを指していないことを示すために予約されたポインタのことである.

> Programs routinely use null pointers to represent conditions such as the end of a list of unknown length or the failure to perform some action

> プログラムは, 不定長のリストの終端を表したり, 何らかの処理が失敗したことを表したりする時に, 日常的にヌルポインタを使っている. (私訳)

最近の言語で null をリストの末尾に突っ込むような言語はないし, そんな使い方をする人は誰もいないと思うので, こんにちでは後者の使われ方が主なものだということになる.

ここで一つ重大な疑問が生じる. それは, **なぜ処理の失敗を表現するためにわざわざ null を使っておいて, それを null 安全を用いて親の敵のように排除しようとしているのか** ということだ.

## null, 要る？

少なくとも, null 安全に魅力を感じるような人はそもそも null が要らないのだ. 

最近の言語は間違いなくポイントフリーな方向に進んでいて, それは彼らがポインタ(とその操作)を憎んでいるからだ.

彼らは, 帰ってきた結果からある処理が成功したか失敗したかを判断できて, それをもとにエラーが起きないような処理が書ければよいだけなのだ.

## 成功と失敗

> 以下は代表的なオブジェクト指向プログラミング言語の一つである C# を用いて解説するが, C# を知らなくてもなるべくわかるように標準ライブラリなどの使用は極力控えている.
 
> C# において値型を Nullable にするには ```int?``` のように書き, この場合 int 型数値と null の両方を持つことができる.

そんな彼らを扉の陰から見ている, 皆様におなじみの物体がある. 

彼の名は enum だ. 

```csharp
enum ServerStatus
{
    Idle, Busy, Error
}

public ServerStatus CheckServerStatus (Server s)
{
    try
    {
        var res = s.Access();
        if (res.IsBusy)
            return ServerStatus.Busy;
        else
            return ServerStatus.Idle;
    }
    catch
    {
        return ServerStatus.Error;
    }
}
```

これは明らかに「null 安全」だ. 定義の煩雑さにさえ目をつぶれば, ```bool?``` を使って bool に null の使用を許す必要など全くない.

しかし, 大抵の場合は Busy か Idle かだけでなく, サーバの接続人数を同時に欲しいものだ.

```csharp

class DetailedServerStatus
{
    public ServerStatus Status;
    int _count;
    public int? ConnectionsCount
    {
        get
        {
            switch(Status)
            {
                case ServerStatus.Idle:
                case ServerStatus.Busy:
                    return (int?)_count;
                case ServerStatus.Error:
                    return null;
        }
    }
}
```
結局 ```int?``` が必要になった. これでは台無しだ. しかも記述が面倒くさくてとてもやっていられない.

しかし, 接続人数が得られるはそもそも処理に成功した場合だけなのだ. つまり, ```ServerStatus.Idle``` や ```ServerStatus.Busy``` 自体がクラスのように振る舞ってくれれば都合がいい.

interface と型チェックでやってもいいが, これもまた面倒だ. そこで, enum にフィールドが直接書ければよいのでは？と考える.

## enum に値を持たせる

おことわり: ここから先は正しい C# ではない.

```csharp
enum ServerStatus
{
    Idle
    {
        int ConncetionsCount; 
    },
    Busy
    {
        int ConnectionsCount;
    },
    Error
}

void PrintStatus(ServerStatus s)
{
    switch(s)
    {
        case ServerStatus.Idle as i:
            Console.WriteLine("Idle ({0} connections)", i.ConncetionsCount);
            break;
        case ServerStatus.Busy as b:
            Console.WriteLine("Busy ({0} connections)", b.ConncetionsCount);
            break;
        case ServerStatus.Error:
            Console.WriteLine("Error!");
            break;
    }
}
```

というように書ければどうだろう. これなら「null 安全」である. 

このような「フィールドを持てるenum」は, 一般に「代数的データ型」や「ヴァリアント」と呼ばれているものと本質的には同じである.

しかしまだ記述がめんどくさいようにも思える. 特に同じ意味の2つの ```int ConnectionsCount``` を一緒くたに扱えないのがよくない.

## 「null 安全」の正体

しかし, 勘のよい読者の皆様ならすでにお気づきのように, このようにすればよいのだ.

```csharp
enum Result<T>
{
    Success
    {
        T Value;
    },
    Failure
}

class ServerStatus
{
    public bool IsBusy;
    public int ConnectionsCount;
    public ServerStatus(bool _isBusy, int _count)
    {
        ...
    }
}

public Result<ServerStatus> CheckServerStatus (Server s)
{
    try
    {
        var res = s.Access();
        return Result.Success(new ServerStatus(res.IsBusy, res.Connections.Count));
    }
    catch
    {
        return Result.Failure;
    }
}
```

この ```Result<T>``` が肝だ. これは ML というプログラミング言語の系譜で古くから「Option」や「Maybe」として知られているものと全く同じ形をしている.

```ocaml
type 'a option = 
    Some of 'a 
  | None
```

```'a``` は ```<T>``` と同じで, C# におけるジェネリック型と同じものだ.

また, OCaml では一つに対して一つのフィールドしか持つことができないので、名前はつけずに単に型名 ```'a``` だけを書く(ただし, OCaml には複数のフィールドを持つ型をかんたんに作れる仕組みがあるため, これは問題にはならない).

もう一度上の ```Result<T>``` を載せておくので比べてみて欲しい.

```csharp
enum Result<T>
{
    Success
    {
        T Value;
    },
    Failure
}
```

この ```Result<T>``` が, 今話題となっている「null 安全」の正体だ.

## 機能の拡張, あるいは「null 安全」の特長がいかにして実現されているか

つまり, 「null 安全」とは, 「フィールドを持てる enum」の特別な場合にすぎないということだ.

しかし, 本当にこれだけで「null 安全」の主張する安全性・利便性をすべて備えているのだろうか？やってみよう.

### 「スマートキャスト」

これは, 「null チェックと non-null へのキャストをセットで行う仕組み」とのことだ.

実際, これは値が返せる switch 式があれば問題ない.

```csharp
enum Result<T>
{
    Success
    {
        T Value;
    },
    Failure
}

var length =
    switch(result)
    {
        case Result.Success as s:
            return s.Value.Length;
        case Result.Failure:
            return 0;
    }
```

このようなものは, ML 界隈では「パターンマッチ」として知られている.

### ```!```, ```!!```

「null である可能性を明示的に無視して nullable を non-null に変換する簡単な方法」, これは上のコードで ```case Failure``` の部分を書かない, もしくは単に例外を投げてしまうことに相当する.

```csharp
void ForceUnwrap<T>(this Result<T> res)
{
    switch(res)
    {
        case Result.Success as x:
            return x.Value;
        case Result.Failure:
            throw new FailureException("given value is Failure");
    }
}

var x = result.ForceUnwrap();
```

これは, OCaml における ```Option.get``` に相当する.

しかし, 本来はこれは要らないかもしれない.

そもそも, したい処理の内部から値を返す必要がないのなら, ```Failure``` の場合は単に処理を無視してくれればいい.

```csharp

void DoWhenSuccess<T>(this Result<T> res, Action<T> action)
{
    switch(res)
    {
        case Result.Success as x:
            action(x.Value);
            break;
        case Result.Failure:
            break;
    }
}

result.DoWhenSuccess(x => 
{
    if(x > 1000)
        Console.WriteLine("x is bigger than 1000!");
    else
        Console.WriteLine(x);
});

```

もしロジック上絶対 ```Failure``` が降ってこないことが分かっているなら, 単に以後の処理すべてを ```DoWhenSuccess``` 内でやってしまえばよいのだ.

この ```DoWhenSuccess``` は, OCaml では ```Option.may``` という名前で知られている.

> C# では, ラムダ式 (クロージャ, ローカルな関数) は
> ```引数 => 式``` というように書く.
> 式は, 波括弧を用いることで複文にすることもできる.

### ```?.```, ```?->```, ```map```

これは単に ```map``` を定義してやればいいだろう.

```csharp
Result<T2> Map<T1, T2>(this Result<T1> res, Func<T1, T2> mapper)
{
    switch(res)
    {
        case Result.Success as x:
            return Result.Success(mapper(x.Value));
        case Result.Failure:
            return Result.Failure;
    }
}

var x2 = result.Map(x => x.StatusCode).Map(x => DoSomething(x));
```

OCaml での呼び名は ```Option.map``` である.

### ```?:```, ```??```

これも同様である.

```csharp
T OrDefault<T>(this Result<T> res, T defaultValue)
{
    switch(res)
    {
        case Result.Success as x:
            return x.Value;
        case Result.Failure:
            return defaultValue;
    }
}

var x2 = result.OrDefault(StaticData.ErrorPage);
```

OCaml では、先にデフォルト値を渡す形だが ```Option.default``` が同じものである.

### ```flatMap```, ```do```

これはパターンマッチの表現能力の問題である.

今までの ```case Result.Success as x``` 方式では, 入れ子になった構造 (```Result``` の中に ```Result``` が入っているなど) を表現・分解することができない.

ここで, 

```csharp
enum Example
{
    Hoge
    {
       T1 Item1, T2 Item2, T3 Item3
    },
    Piyo
    {
       T4 Item1, T5 Item2
    }
}
```

というものがあるとき,

```csharp
switch(x)
{
    case Example.Hoge(x1, x2, x3):
        // ...
        break;
    case Example.Hoge(y1, y2):
        // ...
        break;
}
```

と書けば, ```x1``` に ```Item1```, ```x2``` に ```Item2``` ... と代入されてケース内で使用できる, というようにする.

この時, ```case x in Result.Success(x)``` と書けば, ```T``` 型の ```x``` がケース内で使えるようになる

```csharp
T OrDefault<T>(this Result<T> res, T defaultValue)
{
    switch(res)
    {
        case x in Result.Success(x):
            return x;
        case Result.Failure:
            return defaultValue;
    }
}
```

また, 2つの値をひとまとめにできる ```Tuple``` (タプル, 2つ組) を導入する.

そしてこの ```Tuple``` もパターンマッチでバラせるようにする.

```csharp
class Tuple<T1, T2>
{
    T1 Left, T2 Right
}

var sum =
    switch(new Tuple(1, 2))
    {
        case (x1, x2) in Tuple(x1, x2):
            return x1 + x2;
        default:
            return 0;
    }
```

この時, ```flatMap``` や ```do``` は下のように書き表せる.

```csharp
// flatMap
switch(result)
{
    case x in Result.Success(Result.Success(x)):
        DoSomething(x);
        break;
    default:
        break;
}

// do
switch(new Tuple(res1, res2))
{
    case (x1, x2) in Tuple(Result.Success(x1), Result.Success(x2)):
        DoSomething2(x1, x2);
        break;
    default:
        break;
}
```

ここで ```default``` は, すべての場合にマッチするものである.

2つ目の例で, ```Result``` 2つからなる ```Tuple``` を作って, それに対してパターンマッチをかけているのがミソである.

```Tuple(Result.Success, Result.Failure)``` のようなものはすべて ```default``` に流れることになり, 結果的に ```res1``` と ```res2``` が両方 ```Success``` でないと処理が実行されない, ということが実現されている.

これらは, それぞれ OCaml では次のようである.

```ocaml
(* definition *)
type 'a option = 
    Some of 'a 
  | None

(* flatMap *)
match result with
| Some(Some x) -> doSomething x
| _ -> ()

(* do *)
match (res1, res2) with
| (Some x1, Some x2) -> doSomething2 x1 x2
| _ -> ()
```

同じようなことが, OCaml ではいかに簡単にできるかがわかってもらえると思う.

先ほど,

> OCaml には複数のフィールドを持つ型をかんたんに作れる仕組みがある

と言ったのは, このように OCaml ではタプルをごく簡単に扱えることを指している.

### ```?```

これは, ```T``` を ```Result<T>``` にするだけである.

## 主張:「null 安全」であることは本質的ではない

何が言いたいのかというと, 本当に必要なことは「複数の状態を持つ値を簡単に作れて, それを簡単に扱えること」であって, 決して「null を安全に扱える」ことではないのだ.

つまり, null とかどうとか以前に, 静的型チェックに押し付けられるものはなるべく押し付けてしまったほうがいい, と言い換えることもできる.

例えば, エラー情報も一緒に扱いたい場合は,

```csharp
enum TryResult<T, E>
    where E : Exception // E は Exception の派生クラスに制限
{
    Success
    {
        T Value;
    },
    Failure
    {
        E InnerException;
    }
}
```

というようなものを書けば, 成功したら結果を, 失敗したら起こった例外を持つようなものを作ることができる.

そして, そうやって作った複数の状態を持つ値を, 構造を用いて, 簡単に分解できるような仕組み (高機能なパターンマッチ) があればよいことも, 上で示したとおりである.

こういうことを簡単に, そして当たり前にできる言語が OCaml や Haskell などの言語であって, null 安全を気に入ったあなたならば必ずそのような言語も気に入るはずだ.


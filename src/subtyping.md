<!--
# Subtyping and Variance
-->

# サブタイピングと変位指定

<!--
Although Rust doesn't have any notion of structural inheritance, it *does*
include subtyping. In Rust, subtyping derives entirely from lifetimes. Since
lifetimes are scopes, we can partially order them based on the *contains*
(outlives) relationship. We can even express this as a generic bound.
-->

Rust には構造的な継承を表す記法はありませんが、サブタイピングを含んで *います* 。
Rust では、サブタイピングは完全にライフタイムに由来します。
ライフタイムはスコープであるため、それらを *包含* 関係（寿命）で部分的に順序付けることができます。
これは一般的な境界として表すことさえ可能です。

<!--
Subtyping on lifetimes is in terms of that relationship: if `'a: 'b` ("a contains
b" or "a outlives b"), then `'a` is a subtype of `'b`. This is a large source of
confusion, because it seems intuitively backwards to many: the bigger scope is a
*subtype* of the smaller scope.
-->

ライフタイムは、それらの関係の観点でサブタイピングが行われます：
`'a: 'b`（「a は b を含む」あるいは「a は b よりも長生きである」）のとき、`'a` は `'b` のサブタイプです。
これは直感に反しているように見えるため、大きな混乱の原因となります：
大きなスコープが小さいスコープの *サブタイプ* です。

<!--
This does in fact make sense, though. The intuitive reason for this is that if
you expect an `&'a u8`, then it's totally fine for me to hand you an `&'static
u8`, in the same way that if you expect an Animal in Java, it's totally fine for
me to hand you a Cat. Cats are just Animals *and more*, just as `'static` is
just `'a` *and more*.
-->

実際には、これは意味をなしません。
この直感的な理由は、
Java において Animal を期待しているときに Cat を渡すことが全く問題がないのと同様に、
ある `&'a u8` を期待するならば `&'static u8` を渡すことはすべて上手くいくためです。
Cat は Animal であり、`'static` は `'a` です。

<!--
(Note, the subtyping relationship and typed-ness of lifetimes is a fairly
arbitrary construct that some disagree with. However it simplifies our analysis
to treat lifetimes and types uniformly.)
-->

（サブタイピングの関係とライフタイムの型指定は、いくつかの点で同意できないようなかなり恣意的な構造であることに注意してください。
しかし、ライフタイムと型を一様に扱うため分析を単純化することができます。）


<!--
Higher-ranked lifetimes are also subtypes of every concrete lifetime. This is
because taking an arbitrary lifetime is strictly more general than taking a
specific one.
-->

高階ライフタイムはすべての具体的なライフタイムのサブタイプになります。
これは、任意のライフタイムを取ることは特定のライフタイムを取ることよりも厳密に一般的だからです。

<!--
# Variance
-->

# 変位指定

<!--
Variance is where things get a bit complicated.
-->

変位指定は、物事が少しだけ複雑になります。

<!--
Variance is a property that *type constructors* have with respect to their
arguments. A type constructor in Rust is a generic type with unbound arguments.
For instance `Vec` is a type constructor that takes a `T` and returns a
`Vec<T>`. `&` and `&mut` are type constructors that take two inputs: a
lifetime, and a type to point to.
-->

変位指定は、型コンストラクタがその引数に関して持つ特性です。
Rust における型コンストラクタは、束縛されていない引数を持つジェネリック型です。
例えば、`Vec` は `T` を取り `Vec<T>` を返す型コンストラクタです。
`&` と `&mut` は、ライフタイムと指し示す型の 2 つの入力を取る型コンストラクタです。

<!--
A type constructor's *variance* is how the subtyping of its inputs affects the
subtyping of its outputs. There are two kinds of variance in Rust:
-->

型コンストラクタの **変位指定** は、入力のサブタイピングが出力のサブタイピングにどのように影響を与えるということです。
Rust では次の 2 種類のサブタイピングが存在します。

<!--
* F is *variant* over `T` if `T` being a subtype of `U` implies
  `F<T>` is a subtype of `F<U>` (subtyping "passes through")
* F is *invariant* over `T` otherwise (no subtyping relation can be derived)
-->

* `T` が `U` の部分型のとき F は `T` に関して *ヴァリアント* 、すなわち `F<T>` は `F<U>` のサブタイプとなる（サブタイピングが「通過する」）
* それ以外の場合 F は `T` に関して *インヴァリアント* である（サブタイピングの関係は導出されない）

<!--
(For those of you who are familiar with variance from other languages, what we
refer to as "just" variance is in fact *covariance*. Rust has *contravariance*
for functions. The future of contravariance is uncertain and it may be
scrapped. For now, `fn(T)` is contravariant in `T`, which is used in matching
methods in trait implementations to the trait definition. Traits don't have
inferred variance, so `Fn(T)` is invariant in `T`).
-->

（他の言語の変位指定に精通している人のために説明すると、我々が「単に」ヴァリアントと呼んでいるものは実際には共変性 (covariance) です。
Rust は関数に対し反変性 (contravariance) を持っています。
反変性は不確実なものであり、将来的に廃止される可能性があります。
現状では fn(T) は T に関して反変であり、これはトレイトの実装においてトレイトの実装をトレイトの定義と一致させるために使用されます。
トレイトは推論される変位指定を持たないため、Fn(T) は T に関して不変です。）

> [訳注]
> "variant", "invariant" の訳語として「ヴァリアント」と「インヴァリアント」を採用した。

<!--
Some important variances:
-->

いくつかの重要な変位指定があります：

<!--
* `&'a T` is variant over `'a` and `T` (as is `*const T` by metaphor)
* `&'a mut T` is variant over `'a` but invariant over `T`
* `Fn(T) -> U` is invariant over `T`, but variant over `U`
* `Box`, `Vec`, and all other collections are variant over the types of
  their contents
* `UnsafeCell<T>`, `Cell<T>`, `RefCell<T>`, `Mutex<T>` and all other
  interior mutability types are invariant over T (as is `*mut T` by metaphor)
-->

* `&'a T` は `'a` と `T` に関してヴァリアントです（これは `*const T` のメタファーです）
* `&'a mut T` は `'a` に関してはヴァリアントですが `T` に関してはインヴァリアントです
* `Fn(T) -> U` は `T` に関してインヴァリアントですが、`U` に関してはヴァリアントです
* `Box`, `Vec` などすべてのコレクションはそれらの格納する型に関しヴァリアントです
* `UnsafeCell<T>`, `Cell<T>`, `RefCell<T>`, `Mutex<T>` などの内部可変性を持つ型は `T` に関しインヴァリアントです（これは `*mut T` のメタファーです）

<!--
To understand why these variances are correct and desirable, we will consider
several examples.
-->

これらの変位指定が正当かつ望ましい理由を理解するため、いくつかの例を検討したいと思います。

<!--
We have already covered why `&'a T` should be variant over `'a` when
introducing subtyping: it's desirable to be able to pass longer-lived things
where shorter-lived things are needed.
-->

サブタイピングを導入する際、 `&'a T` が `'a` に関してインヴァリアントであるべきな理由については既に説明しました：
短命のものを受け取る場合、それよりも長い寿命の値を渡せることが望ましいです。

<!--
Similar reasoning applies to why it should be variant over T. It is reasonable
to be able to pass `&&'static str` where an `&&'a str` is expected. The
additional level of indirection does not change the desire to be able to pass
longer lived things where shorted lived things are expected.
-->

同様の推論が、`T` に関してインヴァリアントであるべきだという理由にも当てはまります。
`&&'a str` が期待されているときに `&&'static str` を渡せるというのは合理的です。
追加レベルの間接指定により、短命のものが期待される場合に長い寿命のものを通過させたいという欲求は変わりません。

<!--
However this logic doesn't apply to `&mut`. To see why `&mut` should
be invariant over T, consider the following code:
-->

しかし、この論理は `&mut` には適用されません。
`&mut` が T に関しインヴァリアントであるべき理由を説明するため、次のコードを考えます。

```rust,ignore
fn overwrite<T: Copy>(input: &mut T, new: &mut T) {
    *input = *new;
}

fn main() {
    let mut forever_str: &'static str = "hello";
    {
        let string = String::from("world");
        overwrite(&mut forever_str, &mut &*string);
    }
    // Oops, printing free'd memory
    println!("{}", forever_str);
}
```

<!--
The signature of `overwrite` is clearly valid: it takes mutable references to
two values of the same type, and overwrites one with the other. If `&mut T` was
variant over T, then `&mut &'static str` would be a subtype of `&mut &'a str`,
since `&'static str` is a subtype of `&'a str`. Therefore the lifetime of
`forever_str` would successfully be "shrunk" down to the shorter lifetime of
`string`, and `overwrite` would be called successfully. `string` would
subsequently be dropped, and `forever_str` would point to freed memory when we
print it! Therefore `&mut` should be invariant.
-->

`overwrite` のシグネチャは明らかに正当です：
同じ型をもつ 2 つの値への可変リファレンスを受け取り、一方の値でもう一方を上書きします。
`&'static str` は `&'a str` のサブタイプであるため、 `&mut T` が T に関してヴァリアントであった場合 `&mut &'static str` が `&mut &'a str` のサブタイプとなってしまいます。
その結果 `for_ever_str` のライフタイムはそれよりも短い `string` のライフタイムへと正常に「縮小」され、`overwrite` は正常に呼び出されます。
その後 `string` はドロップされ、出力するときには `forever_str` は解放済みのメモリを指しています。
したがって、`&mut` はインヴァリアントである必要があります。

<!--
This is the general theme of variance vs invariance: if variance would allow you
to store a short-lived value into a longer-lived slot, then you must be
invariant.
-->

これはヴァリアントとインヴァリアントに関する一般的なテーマです：
ヴァリアントにより短命の値が（それよりも）寿命の長いスロットに格納できるようになるのであれば、インヴァリアントでなければいけません。

<!--
However it *is* sound for `&'a mut T` to be variant over `'a`. The key difference
between `'a` and T is that `'a` is a property of the reference itself,
while T is something the reference is borrowing. If you change T's type, then
the source still remembers the original type. However if you change the
lifetime's type, no one but the reference knows this information, so it's fine.
Put another way: `&'a mut T` owns `'a`, but only *borrows* T.
-->

しかし、これは `&'a mut T` が `'a` に関してヴァリアントであるという点に関しては健全です。
`'a` と `T` の間の主要な違いは、`'a` がリファレンス自身に関する特性であるのに対し、T はリファレンスが借用しているものに関するという点です。
T の型を変更した場合、ソースは元の型を覚えていますが、
ライフタイムの型を変更した場合はリファレンス以外にこの情報を知らないので問題ありません。
別の言い方をすると、`&'a mut T` は `'a` を所有しますが T は単に *借用する* だけです。

<!--
`Box` and `Vec` are interesting cases because they're variant, but you can
definitely store values in them! This is where Rust gets really clever: it's
fine for them to be variant because you can only store values
in them *via a mutable reference*! The mutable reference makes the whole type
invariant, and therefore prevents you from smuggling a short-lived type into
them.
-->

`Box` と `Vec` はそれらがヴァリアントであるという点で興味深いですが、それらの値を問題なく格納することができます！
これは、Rust が本当に巧妙な点です：
それらは *可変リファレンスを介する* ことでしか格納できないため、それらがヴァリアントであることに問題はありません。
可変リファレンスは型全体をインヴァリアントにするため、寿命の短い型が混入することを防止します。

<!--
Being variant allows `Box` and `Vec` to be weakened when shared
immutably. So you can pass a `&Box<&'static str>` where a `&Box<&'a str>` is
expected.
-->

ヴァリアントであるため、不変に共有される場合に `Box` と `Vec` が弱体化することが許されます。
したがって、`&Box<&'a str>` が期待されているときに `&Box<&'static str>` を渡すことができます。

<!--
However what should happen when passing *by-value* is less obvious. It turns out
that, yes, you can use subtyping when passing by-value. That is, this works:
-->

しかし、 *値渡し* のときに何が起こるべきかということについてはあまり明らかではありません。
はい、値渡しの際にサブタイピングを使用することが出来ることが判明しています。
すなわち、次のコードは動作します：

```rust
fn get_box<'a>(str: &'a str) -> Box<&'a str> {
    // string literals are `&'static str`s
    Box::new("hello")
}
```

<!--
Weakening when you pass by-value is fine because there's no one else who
"remembers" the old lifetime in the Box. The reason a variant `&mut` was
trouble was because there's always someone else who remembers the original
subtype: the actual owner.
-->

Box 内の古いライフタイムを「覚えている」ものが誰もいないため、値渡しの際に弱体化することは有効です。
`&mut` においてヴァリアントが問題となった理由は、元のサブタイプ、すなわち実際の所有者を覚えている他の誰かが常に存在するためです。

<!--
The invariance of the cell types can be seen as follows: `&` is like an `&mut`
for a cell, because you can still store values in them through an `&`. Therefore
cells must be invariant to avoid lifetime smuggling.
-->

セル型におけるインヴァリアントは次のように見ることができます。
`&` を介して値を格納することが出来るため、セル型における `&` は `&mut` と似ています。
したがって、セルはライフタイムの密輸を避けるためにインヴァリアントである必要があります。

<!--
`Fn` is the most subtle case because it has mixed variance. To see why
`Fn(T) -> U` should be invariant over T, consider the following function
signature:
-->

`Fn` は混合したヴァリアントを持つため最も微妙なケースです。
`Fn(T) -> U` が `T` に対してインヴァリアントでなければいけない理由を調べるため、次の関数シグネチャを考えます：

```rust,ignore
// 'a is derived from some parent scope
fn foo(&'a str) -> usize;
```

<!--
This signature claims that it can handle any `&str` that lives at least as
long as `'a`. Now if this signature was variant over `&'a str`, that
would mean
-->

このシグネチャは、少なくとも `'a` と同じ寿命を持つ `&str` を処理できることを主張しています。
いま、このシグネチャが `&'a str` に対しヴァリアントであったとき、これは

```rust,ignore
fn foo(&'static str) -> usize;
```

<!--
could be provided in its place, as it would be a subtype. However this function
has a stronger requirement: it says that it can only handle `&'static str`s,
and nothing else. Giving `&'a str`s to it would be unsound, as it's free to
assume that what it's given lives forever. Therefore functions are not variant
over their arguments.
-->

がサブタイプであり、その場所に格納できることを意味しています。
しかし、これは `&'static str` のみを扱うことができほかは処理できないというより強い要求を持っています。
与えられた値が永遠に生存していると仮定するのは自由であり、それに `&'a str` を渡すことは不健全です。
したがって、関数は引数に対しヴァリアントではありません。

<!--
To see why `Fn(T) -> U` should be variant over U, consider the following
function signature:
-->

`Fn(T) -> U` が U に対しヴァリアントであるべき理由を見るため、次のような関数シグネチャを考えます。

```rust,ignore
// 'a is derived from some parent scope
fn foo(usize) -> &'a str;
```

<!--
This signature claims that it will return something that outlives `'a`. It is
therefore completely reasonable to provide
-->

このシグネチャは、`'a` より長く生存する何かを返すことを主張しています。
そのため、この場所に

```rust,ignore
fn foo(usize) -> &'static str;
```

<!--
in its place. Therefore functions are variant over their return type.
-->

を与えることは完全に妥当です。
したがって、関数は戻り値型に対してヴァリアントです。

<!--
`*const` has the exact same semantics as `&`, so variance follows. `*mut` on the
other hand can dereference to an `&mut` whether shared or not, so it is marked
as invariant just like cells.
-->

`*const` は `&` と全く同じセマンティクスを持っているため、変位指定は引き継がれます。
一方 `*mut` は、共有されているかどうかにかかわらず `&mut` へのデリファレンスが可能であるためセルと同じくインヴァリアントとしてマークされます。

<!--
This is all well and good for the types the standard library provides, but
how is variance determined for type that *you* define? A struct, informally
speaking, inherits the variance of its fields. If a struct `Foo`
has a generic argument `A` that is used in a field `a`, then Foo's variance
over `A` is exactly `a`'s variance. However this is complicated if `A` is used
in multiple fields.
-->

これは標準ライブラリの提供するすべての型において適用されますが、ユーザ定義型に対してはヴァリアントはどのように決定されるのでしょうか？
非形式的に言うと、構造体はそのフィールドのヴァリアントを継承します。
構造体 `Foo` があるひとつのフィールド `a` で使用されるジェネリックパラメータ `A` を持つとき、Foo は `A` に対し `a` と同じ変位指定となります。
しかし、`A` が複数のフィールドで使用される場合複雑になります。

<!--
* If all uses of A are variant, then Foo is variant over A
* Otherwise, Foo is invariant over A
-->

* A を使用するすべてがヴァリアントであるとき、Foo は A に対しヴァリアントである
* それ以外の場合、Foo は A に対しインヴァリアントである

```rust
use std::cell::Cell;

struct Foo<'a, 'b, A: 'a, B: 'b, C, D, E, F, G, H> {
    a: &'a A,     // variant over 'a and A
    b: &'b mut B, // variant over 'b and invariant over B
    c: *const C,  // variant over C
    d: *mut D,    // invariant over D
    e: Vec<E>,    // variant over E
    f: Cell<F>,   // invariant over F
    g: G,         // variant over G
    h1: H,        // would also be variant over H except...
    h2: Cell<H>,  // invariant over H, because invariance wins
}
```

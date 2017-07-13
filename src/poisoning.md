<!--
# Poisoning
-->
# 毒殺

<!--
Although all unsafe code *must* ensure it has minimal exception safety, not all
types ensure *maximal* exception safety. Even if the type does, your code may
ascribe additional meaning to it. For instance, an integer is certainly
exception-safe, but has no semantics on its own. It's possible that code that
panics could fail to correctly update the integer, producing an inconsistent
program state.
-->
すべてのアンセーフなコードは例外安全性が最小であることを保証する*必要があります*が、すべての型が*最大の*例外安全性を保証するわけではありません。
たとえ型がそれを保証していても、あなたのコードがそれに追加の意味を付与してしまうかもしれないのです。
例えば、整数は確かに例外安全ですが、それ自体に意味を持ちません。
パニックの発生により整数の値が正しく更新されず、矛盾したプログラムの状態を生み出す可能性があります。

<!--
This is *usually* fine, because anything that witnesses an exception is about
to get destroyed. For instance, if you send a Vec to another thread and that
thread panics, it doesn't matter if the Vec is in a weird state. It will be
dropped and go away forever. However some types are especially good at smuggling
values across the panic boundary.
-->
これは*通常*細かいことです。なぜなら、例外を目撃するものは今まさに破壊されようとしているためです。
例えば、ある Vec を別のスレッドに送信しそのスレッドがパニックした場合、その Vec が奇妙な状態になることは問題ではありません。
それはドロップされ永遠に消え失せてしまうからです。
しかしながら、いくつかの型はパニック境界をまたがり値を持ち込むことに特に優れています。

<!--
These types may choose to explicitly *poison* themselves if they witness a panic.
Poisoning doesn't entail anything in particular. Generally it just means
preventing normal usage from proceeding. The most notable example of this is the
standard library's Mutex type. A Mutex will poison itself if one of its
MutexGuards (the thing it returns when a lock is obtained) is dropped during a
panic. Any future attempts to lock the Mutex will return an `Err` or panic.
-->
これらの型は、パニックを目撃した場合に自分自身を明示的に*毒殺する*ことを選ぶかもしれません。
毒殺は特に何かを伴うわけではありません。
一般に、それは通常の使用が進行しないよう妨げるだけです。
最も注目すべき例は標準ライブラリの Mutex 型です。
Mutex は（ロックが取得された際に返す）MutexGuard のひとつがパニックの発生によりドロップした場合、自身を毒殺します。
その後 Mutex のロックを取得しようとすると、 `Err` を返すかパニックします。

<!--
Mutex poisons not for true safety in the sense that Rust normally cares about. It
poisons as a safety-guard against blindly using the data that comes out of a Mutex
that has witnessed a panic while locked. The data in such a Mutex was likely in the
middle of being modified, and as such may be in an inconsistent or incomplete state.
It is important to note that one cannot violate memory safety with such a type
if it is correctly written. After all, it must be minimally exception-safe!
-->
Mutex は Rust が通常気にする意味での真の安全性のために自身を毒殺するわけではありません。
ロック中にパニックしたことを目撃した Mutex は、盲目的なデータの使用を防ぐセーフティーガードとして自身を毒殺します。
そのような状況では、Mutex の持つデータは値を変更している途中で一貫性のない不完全な状態である可能性が高いためです。
正しく書かれていれば、このようなタイプのメモリ安全性の違反は生じないことに注意することが重要です。
結局、それは最小限に例外安全であるはずだからです！

<!--
However if the Mutex contained, say, a BinaryHeap that does not actually have the
heap property, it's unlikely that any code that uses it will do
what the author intended. As such, the program should not proceed normally.
Still, if you're double-plus-sure that you can do *something* with the value,
the Mutex exposes a method to get the lock anyway. It *is* safe, after all.
Just maybe nonsense.
-->
しかし、Mutex に含まれる値が実際には heap property を持たない BinaryHeap であった場合、それを使用するコードが想定通りに動作する可能性は低いでしょう。
それ自体、プログラムが正常に動作しないはずです。
それでも、あなたがその値を使って*何か*できることを二重に確信しているのであれば、
Mutex はロックを取得する方法を公開しています。
それは結局安全です。
無意味かもしれないですが。

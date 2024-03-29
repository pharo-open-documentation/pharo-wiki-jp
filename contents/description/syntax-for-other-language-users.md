# 構文比較早見表

他言語の知識がある方向けの比較表になります。

## 純粋言語機能

### 基礎構文

|                 | Pharo      | C++/C#/Java | 備考 |
| --------------- | ---------- | ------------ | ---- |
| 代入             | :=         | =           |       |
| 式の区切り        | ,          | ;           |       |
| Messageの連続送信 |          ; | ( 該当なし ) | 詳しくは "Message式" にて後述 |
| 復帰文           | ^ 0.        |  return 0;  | 後述するBlockの値を返すためには使えないため一部異なります。 |
| 局所変数宣言      | \| value \| | int value; | Pharoの変数にはintやlongといった型はありません。 |
| 注記            |       "注記" | /* 注記 */ | |

### 定数

|                    | Pharo              | C++/C#/Java | 備考 |
| ------------------ | ------------------ |  ---------- |---- |
| 整数                |                 -1 |         -1   |    |
| 少数                |               -1.2 |         -1.2 |    |
| 任意進数             |             -16rFF |        -0xFF | 16の部分は2や8といった進数を指定します。 |
| 文字                |                 $A |         'A'   |    |
| 文字列              | 'exmple'           |  "exmple"      |    |
| 識別子1( Symbol )   | #example           | <該当無し>      |     |
| 識別子2( Symbol )   | #'example'         | <該当無し>      | ''の間は英数字以外を使うことが出来ます。 |
| 真偽値              | true / false       | true / false   |    |
| 値無し              | nil                | nullptr / null |    |


|          | Pharo              | C++         　　　　　　　　　　　　　　　　| C#/Java 　　　　　　　　　　　| 備考 |
| -------- | ------------------ |  ------------------------------------ | ------------------------- | ---- |
| 定数配列　| #( 1 $a 'b' #c d ) | std::array< int, 3 >{ 1, 2, 3 } 　　　　| new int[]{ 1, 2, 3 }　　　  | 定数のみ格格納可能で「d」のような文字は識別子に変換します。変数同様型はありません。 |
| Byte配列 | #[ 16rF0 16rF1 ] | std::array< std::byte, 2 >{ 0xF0, 0xF1 } | new byte[]{ 0xF0, 0xF1 } | 1要素 1byteの配列です。 |


### 構文糖

一部のMessage式( 後述 )に対する簡易構文です。

|       | Pharo              | C++         | C# / Java | 備考 |
| ----- | ------------------ |  ---------- | ------- |---- |
| 配列 | { 1 + a. 2. } | std::array< int, 3 >{ 1 + a, 2 } | new int[]{ 1 + a, 2 }  | "Array with: ( 1 + a ) with: 2" の簡易構文です。 |

### Block

一般に無名関数と呼ばれるものになります。ただしBlockの場合、関数ではなくobjectになっており、"["と"]"の間に書いた処理をただ実行するだけでなく、Blockだけで複数の機能を持っています。

|         | Pharo             | C++         　　　　　　　　　　　　　 | C# 　       　　　　| Java 　　　        　　　　　　| 備考 |
| ------- | ----------------- |  -------------------------------- | ----------------- | ---------------------------- | ---- |
| 引数なし | ```[ 0 ]```        | ```[](){ return 0; }```            | ```() => 0``` 　| ```() -> { return 0; }```    |      |
| 引数あり | ```[ :x \| x ]```  | ```[]( auto x ){ return value }``` | ```( x ) => x``` | ```( x ) -> { return x; }```  |     |

### 特殊変数

|                  | Pharo | C++              | C# / Java   | 備考 |
| ---------------- | ----- | ---------------- | ----------- | ---- |
| 自己参照          | self  | this              | this       |      | 
| 基底( 派生元 )参照 | super | ( 基底型名 )       | super      |      | 
| 実行状態          | thisContext | ( 該当なし ) | ( 該当なし ) |     |

#### thisContext についての補足

thisContext は呼び出し履歴や局所変数、引数などMessage送信の階層が持つ様々な情報を参照できる疑似変数です。また、Message送信の階層と一対となっており、送信元で変数に保存しておいたthisContextを受信者側で使うことにより3階層上の送信元に制御を巻き戻したりすることができます。

### Message式

後述する処理方法を起動する構文で、他言語における関数呼び出しに相当します。ただし関数と異なり同じ名前の処理方法が存在しているとは限りません。

- 単項・二項・多項の3種類に分かれており、その3種類で優先度が異なります。
- 下記の "asInteger" "at:put:" "+" といった名前に相当する部分をSelectorと言います。
- Selectorは識別子として実装しているため"#at:put"と識別子形式で表現します。
- 下記の "object" に相当する部分を受信者( Receiver )と言います。
- 下記の "at: 1 put: 2" など受信者を除いた部分をMessageと言います。Messageはその部分だけを保存したりできます。

|                   | 優先度 | Pharo               | C++/C#/Java          | 備考 |
| ----------------- | ----- | ------------------- | -------------------- | ---- |
| 単項              |     1 | object asInteger    | object.AsInteger()   |      |
| 多項( 引数1個の例 ) |     3 | object at: 1        | object.At( 1 )       |      |
| 多項( 引数2個の例 ) |     3 | object at: 1 put: 2 | object.AtPut( 1, 2 ) |      |


|       | 優先度 | Pharo               | C++/C#             | 備考 |
| ----- | ----- | ------------------- | ------------------ | ---- |
| 二項   |     2 | object1 + object2    | object1 + object2 | 利用者定義演算子に相当します。3文字以内であれば ":=" 以外のあらゆる記号を名前にできます。 |

#### 二項Messageについての補足

Pharoには純粋な言語機能としては代入( ":=" )以外の演算子はありません。演算子に見えるものが多数存在しますが、それらは二項Messageとして実装したものになります。

#### 優先度変更

"(" と ")" で囲むとその間の式を優先して評価します。

#### 優先度

下記( 1 )の様に記載した場合、下記( 2 )の様に解釈します。
これによりカッコの量を減らしています。

( 1 )
```smalltalk
object1
    at:     1
    put:    2 + object2 asInterger.
```

( 2 )
```smalltalk
object1
    at:     1
    put:    ( 2 + ( object2 asInterger ) ).
```

#### Messageの連続送信

前述の ";" をMessageの後ろにつけることで、複数のMessageを受信者に送ることができます。
下記( 1 )であれば下記( 2 )と同等になります。

( 1 )
```smalltalk
collection
    add:    1;
    add:    2;
    add:    3.
```

( 2 )
```smalltalk
collection
    add:    1.

collection
    add:    2.

collection
    add:    3.
```

### Pragma

後述する処理方法に付与する識別情報兼付加情報です。Pragmaを処理方法につけると統合開発環境やその他の機能がその処理方法を特別扱いできるようになります。
例えば```< script >```が級( Class )側の処理方法についていればGUIから処理方法の名前を選ぶだけで、その処理を実行できます。
Paragmaは単項Messageまたは多項Messageを```"<"```と```">"```で囲んだ形式になっています。

|       | Pharo                  | C#                       | Java                  | 備考  |
| ----- | ---------------------- | ------------------------ | --------------------- | ---- |
| 単項   | ```< script >```       | ```[ Script ]```         | ```@Script```         |      |
| 多項   | ```< primitive: 1 >``` | ```[ Primitive( 1 ) ]``` | ```@Primitive( 1 )``` |      |

### 処理方法( Method )の記述

Pharoで定義する処理方法は必ず級に属します。
C++側の関数はclassの表記を省略しています。
Pharoにはvoidに相当する戻り値はなく、復帰文を省略した場合はselfを返します。

#### ■Pharoによる記述例

単項
```smalltalk
isString

    ^ false.
```

二項
```smalltalk
>= anInteger

    ^ anInteger <= self.
```

多項
```smalltalk
between:    aMinInteger
and:        aMaxInteger

	^ self >= aMinInteger
        and: [ self <= aMaxInteger ]
```

#### ■上記に相当するC#の記述例

単項
```CSharp
bool IsString()
{
    return false;
}
```

二項
```CSharp
bool operator >= ( int anInteger )
{
    return anInteger <= this;
}
```

多項
```CSharp
int Between( int aMinInteger, int aMaxInteger )
{
    return this >= aMinInteger && this <= aMaxInteger;
}
```

#### ■より複雑な記述例

```smalltalk
valueWithin:    aDuration
onTimeout:      timeoutBlock
    "↑( 1 )"

	< debuggerCompleteToSender >          "<- ( 2 )"
	| theProcess delay watchdog tag |     "<- ( 3 )"

	aDuration <= Duration zero
		ifTrue: [ ^ timeoutBlock value ]. "<- ( 4 )"

	theProcess := Processor activeProcess.
	delay := aDuration asDelay.
	tag := self.

    "↓( 5 )"
	watchdog :=
	[
		delay wait.
		theProcess
			ifNotNil:
			[
				theProcess signalException: (TimedOut new tag: tag)
			]
	] newProcess.

	watchdog priority: Processor timingPriority - 1.

	^
	[
		watchdog resume.
		self
            ensure:
		    [
			    theProcess := nil.
		    	delay delaySemaphore signal.
		    ]
	]
		on: TimedOut
		do:
		[ :exception |
			exception tag == tag
				ifTrue:
				[
					timeoutBlock value
				]
				ifFalse:
				[
					exception pass
				]
		]
```

- ( 1 ) Selectorに引数を挟んだものが処理方法の名前になります。この名前もSelectorと言い、先頭に書く必要があります。
- ( 2 ) PragmaはSelectorの次に書きます。Pragmaは必要なだけ複数書くことができます。
- ( 3 ) 変数宣言はBlockの先頭を除いて式の途中で書くことはできません。Pragmaよりは後で式より先に書く必要があります。
- ( 4 ) 復帰文が実行された場合、Blockの中にあってもBlockを抜けるだけで止まりません。処理方法の実行も中止して抜けます。Blockだけを中止して抜けたい場合は```thisContext```を使う必要があります。
- ( 5 ) 途中に出てくる```"["```と```"]"```は全てBlockになります。あらゆる制御構文はMessage式とBlockと復帰文からなり、純粋な言語機能としてはそれ以外の制御構文はありません。

## 準言語機能

Message送信と処理方法およびBlockの組み合わせで作られている機能です。

### 級( Class )の登録

多くの言語では級( Class )は静的に宣言するものですが、Pharoでは実行時に環境に登録する形となります。そのため反復で大量の級を登録したり削除できます。

#### ■Pharoによる記述例

```smalltalk
Object
	subclass: 				#Some
	instanceVariableNames:　'one two'
	classVariableNames: 	'three four'
	package: 				'Example'
```

下記はC#との比較用の処理手順で級を登録する上では必須ではありません。

```smalltalk
isString

    ^ false.
```

#### ■上記に相当するC#の記述例

```CSharp
class Some:
	Object
{
	// 仮にintとしていますが、Pharoの方は型はありません
	// 変数は全てprotectedになります。
	protected int 			one, two;
	protected static int 	three, four;

	public bool IsString()
	{
		return true;
	}
}
```

#### 特性( Trait )

特性はPharoにおいて多重継承を実現するための仕組みです。
特性を級に取り込むことにより複数の級で同じ実装を共有できます。特性からobjectを作ることはできません。
C#やJavaのinterfaceと類似しているため比較対象としてinterfaceをあげています。しかし実態としては真逆の関係となります。
interfaceは元々文字通りinterfaceを定義するもので実装を提供しませんが、特性は実装の提供を目的としたものになります。

#### ■Pharoによる記述例

```smalltalk
Trait
	named: 					#TSome
	instanceVariableNames:	''
	package: 				'Example'
```

下記はC#との比較用の処理手順で特性を登録する上では必須ではありません。

```smalltalk
isString

    ^ false.
```

#### ■上記に相当するC#の記述例

```CSharp
interface TSome
{
	bool IsString()
	{
		return true;
	}
}
```

#### ■特性を使う場合の級の登録

■上記のTSomeを取り込む場合

```smalltalk
Object
	subclass: 				#Some
	uses: 					TSome "「uses:」が増えます。"
	instanceVariableNames:　'one two'
	classVariableNames: 	'three four'
	package: 				'Example'
```

■複数の特性を取り込む場合

```smalltalk
Object
	subclass: 				#Some
	uses: 					TSome1 + TSome2
	instanceVariableNames:　'one two'
	classVariableNames: 	'three four'
	package: 				'Example'
```

## 制御系Message

### 分岐

#### ■Pharoによる記述例

```smalltalk
true
	ifTrue:
	[
		self
			traceCr:	'True'.
	]
	ifFalse:
	[
		self
			traceCr:	'False'.
	]
```

#### ■上記に相当するC#の記述例

```CSharp
if( true )
{
	Console.WriteLine( "True" );
}
else
{
	Console.WriteLine( "False" );
}
```

#### ※注意※

これまでも準言語機能やmessageと書いている通り、Pharoの制御は純粋な制御構文ではありません。
分岐を正確に他の言語で書くと下記のようになります。
分岐以外も同様になりますのでご注意ください。

#### ■Pharoによる記述例

```smalltalk
true
	ifTrue:
	[
		1.
	]
	ifFalse:
	[
		-1.
	]
```

#### ■上記に相当するC#の記述例

C#のtrueにはIfが無いため仮にあったとすればという例になります。

```CSharp
true.If( () => 1, () => -1 );
```

### 分岐( switch )

Pharoにも#caseOf:というものが存在しましたが、
非推奨となっているため省略します。
Pharoでは連想配列やobjectの多態性で代用してください。

### 反復

#### 列挙

##### ■Pharoによる記述例

```smalltalk
#( 1 2 3 4 )
	do:
	[ :each |
	]
```

##### ■上記に相当するC#の記述例

```CSharp
foreach( var each in new int[]{ 1, 2, 3 } )
{
}
```

#### 条件付き

##### ■Pharoによる記述例

```smalltalk
[
	true
]
	whileTrue:
	[
	]
```

##### ■上記に相当するC++/C#/Javaの記述例

```CSharp
while( true )
{
}
```

#### 無条件

##### ■Pharoによる記述例

```smalltalk
[
] repeat.
```

##### ■上記に相当するC++/C#/Javaの記述例

```CSharp
for( ;; )
{
}
```

#### 反復からの脱出

##### ■Pharoによる記述例

```smalltalk
|
	label
|

label := thisContext.

#( 1 2 3 4 )
	do:
	[ :each |
		3 < each
			ifTrue:
			[
				label resume.
			] 
	]
```

##### ■上記に相当するJavaの記述例

```Java
label: for( var each : new int[]{ 1, 2, 3, 4 } )
{
	if( 3 < each )
	{
		break label;
	}
}
```

Pharoには性能改善のため、一部のmessageを軽量な中間言語に展開してしまいます。
(例：[] repeat, ifTrue:[] ifFalse:[])
これらのmessageの中では上記は脱出先と脱出元が同じになってしまいうまく動作しません。
```smalltalk
[ [  ] repeat ] value
```
という形で脱出したいblockを展開されないblockで囲むか、後述する継続を使います。
なお展開するmessageは「Setting Browser」の「Compiler」から「Inline」と書かれているところでどれを展開するか選択できます。

### 継続

#### ■Pharoによる記述例

```smalltalk
Continuation
	currentDo:
	[ :label |
		self
			traceCr: 'one'.
		label
			value: 0. "ここでblockを抜ける。以降の処理は実行しない。0は戻り値となる"
		self
			traceCr: 'two'.
	]
```

#### ■上記に相当する他言語での記述例

＜作成中＞

### 生成式

#### ■Pharoによる記述例

```smalltalk
|
	readStream
|

readStream :=
	Generator
		on:
		[ :writeStream |
			writeStream
				nextPut: 0.
			writeStream
				nextPut: 1.
			writeStream
				nextPut: 2. "nextを2回しか送ってないのでここで止まる"
		].

readStream next. "-> 0"
readStream next. "-> 1"
```

#### ■上記に相当する他言語での記述例

＜作成中＞

### 例外

#### ■Pharoによる記述例

```smalltalk
[
	ZeroDivide
		signal: 'message'
]
	on: Error, Notification
	do:
	[ :exception |
	].
```

#### ■上記に相当するJavaの記述例

```Java
try
{
	throw new ZeroDivide( "message" );
}
catch( Error | Notification exception )
{
}
```

大まかな動きはほぼ同じですが、いくつか違いがあるので注意が必要です。
Pharoの例外は例外を投げた地点から処理を継続するか#on:do:に指定したblockで判断する想定の設計になっています。
このため#on:do:に指定したblockを実行している時点では処理手順を巻き戻しません。
例外発生地点が排他制御の中にある場合、他言語の感覚では#on:do:に指定したblockに入れば排他制御を抜けていると思いがちですが、排他制御からは抜けていません。
他言語のように処理手順を呼び出し元まで巻き戻しながら処理したい場合は、#ifCurtailed:を使います。
finallyと同様の処理をしたい場合は#ensure:を使います。

Pharoの例外は異常( Error )と通知( Notification )の2種類に別れます。
異常は他の言語と同様ですが、通知はやや特殊な動きをします。通知は#on:#do:で例外を捕まえなかった場合、例外発生地点からそのまま処理を継続します。

## 関連記事

- [記事一覧](../../README.md)
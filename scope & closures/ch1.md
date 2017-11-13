# You Don't Know JS: Scope & Closures
# Bölüm 1: Scope nedir?

Hemen hemen her programlama dilinin en temel bileşenlerinden biri, değişkenlerin içerisinde değerleri saklayabilme, daha sonra ulaşabilme ve değiştirebilme yeteneğidir. Aslında, değişkenlerin değerlerini saklamak ve değişkenlerden bu değerleri alabilmek, bir programa asıl özelliğini veren şeydir diyebiliriz.

Böyle bir kavram olmasaydı, programlar ancak çok kısıtlı ve çokta ilginç olmayan işlevler yerine getirebileceklerdi. 

Değişkenlerin programların içerisine dahil olmaları şu soruları akla getiriyor: Bu değişkenler nerede *yaşıyorlar*? Diğer bir değişle nerede saklanıyorlar, tutuluyorlar? Ve en önemlisi program bu değişkenlere ihtiyaç duyduğu zaman onları nasıl buluyor?

Bu durumda, değişkenleri bir yerlerde saklamak ve bu değişkenlere daha sonra ulaşabilmek için bir takım iyi tanımlanmış kurallar zincirinin gerekliliği akla geliyor. İşte bu kurallar dizisine *Scope* adını veriyoruz.

Peki, bu *Scope* kuralları nerede ve nasıl tanımlanıyor ve kullanılıyor?

## Derleyici (Compiler) Teorisi

Kimine göre normal, kimine göre de şaşırtıcı gelebilir ama JavaScript genellikle "dinamik" yada "yorumlanan (interpreted)" diller kategorisine girsede alında "derlenen (compiled)" bir dildir. Aslında tam olarak diğer geleneksel derlenen diller gibi derlenmiş değildir ve dağıtık sistemler arasında taşınabilir değildir.

Ama yine de, JavaScript motoru, geleneksel derleyicilerin yaptığı işlemlerin çoğunu yerine getirir (ve aslında farkında olduğumuzdan daha sofistike yöntemler kullanarak).

Geleneksel derlenen dillerde, yazılan kod, yani program, çalıştırılmadan önce genel olarak "derleme (compilation)" diyeceğimiz üç temel adımdan geçer:

1. **Tokenizing/Lexing:** Karakter dizilerini, programlama diline göre anlamlı, token denen parçalara ayırma işlemine denir. Mesela, programın `var a = 2` karakter dizisinden oluştuğunu varsayalım. Bu program şu şekilde tokenlara ayrılacaktır: `var`, `a`, `=`, `2`, ve `;`. Boşluk karakterleri dilin yapısına göre anlamlandırılıp token olarak kullanılacak veya anlamsız görülüp kullanılmayacaktır.

    **Note:** The difference between tokenizing and lexing is subtle and academic, but it centers on whether or not these tokens are identified in a *stateless* or *stateful* way. Put simply, if the tokenizer were to invoke stateful parsing rules to figure out whether `a` should be considered a distinct token or just part of another token, *that* would be **lexing**.

2. **Parsing:** Token'ların dizisini (array) alıp, programın gramer yapısını oluşturan, dallara ayrılmış ağaç yapısına dönüştürme işlemine denir. Bu ağaç yapısına "AST" (<b>A</b>bstract <b>S</b>yntax <b>T</b>ree - Soyut Söz Dizimi Ağacı) denir.

    `var a = 2;` söz dizimi için ağaç yapısı şu şekilde oluşmaktadır: En üst seviye boğum (node) `VariableDeclaration` olarak adlandırılır. Bu boğumun alt dallarından biri `Identifier` (değeri `a`'dır) olarak adlandırılır. Diğer bir alt dal `AssignmentExpression` olarak adlandırılır. Bu son alt dalın bir tane de kendi alt dalı vardır; o da `NumericLiteral` olarak adlandırılır (değeri `2`'dir).

3. **Code-Generation:** AST denilen ağaç yapısının çalıştırılabilir koda dönüştürülmesi işlemidir. Bu işlemler programlama diline, hedeflenen platforma, vb. durumlara göre farklılık gösterebilir.

    Ayrıntılarda boğulmadan bu işlemi şu şekilde özetleyebiliriz: `var a = 2;` söz dizimi için yukarıda tanımladığımız AST ağaç yapısı alınıp, makine diline çevirilir. `a` isminde bir değişken *yaratılır* (bellekte alan rezerve etme, vb. işlemler). Ve bu değişkenin içerisinde `2` değeri saklanır.

    **Not:** JavaScript motorunun sistem kaynaklarını nasıl yönettiği konumuzun sınırlarını aşıyor. Bu yüzden sadece JavaScript motorunun ihtiyaç halinde bellekte yer açıp bir değişken oluşturabileceğini ve saklayabileceğini bilmemiz yeterlidir.

JavaScript motoru da diğer programlama dilleri gibi bu üç adımdan daha karmaşık işler yürütür. Örneğin, parçalama (parsing) ve kod oluşturma (code-generation) aşamalarında, gereksiz elamanların ayıklanması, vb. gibi programın çalışma performansını artırıcı bir dizi işlemler uygulanır.

Burada genelde önemli noktaları vurgulamaya çalışıyorum. Ancak bazı durumlarda detaylara girmemizin nedenini ilerleyen aşamalarda anlayacaksınız.

JavaScript motorunun diğer dillerin derleyicileri gibi derlenecek kodu optimize etmek için bolca zaman harcama lüksü yoktur. Çünkü JavaScript derlemesi diğer dillerde olduğu gibi önceden bir build aşamasında yapılmaz.

JavaScript derleme işlemi çoğu durumda kod çalışmadan bir kaç mikro saniye (belki daha az!) öncesinde yapılır. En iyi performansı sağlayabilmek için JavaScript motoru, burada bahsettiklerimizde dahil birçok yöntemi (JIT, lazy compile, re-compile, vb. gibi) kullanır.

Basitçe yinelemek gerekirse, JavaScript'te her kod parçası çalışmadan önce (genelllikle hemen önce) derlenmesi gerekir. Yani JavaScript derleyici `var a = 2;` kodunu alır, ilk önce derleme işlemini yapar ve hemen sonrasında da kodu çalıştırır.

## Scope'u anlamak

Scope yapısını, karşılıklı konuşma şeklinde düşünüp açıklamaya çalışacağız. Peki bu karşılıklı konuşmada kimler olacak?

### Oyuncular

Gelin isterseniz biraz sonra dinleyeceğimiz karşılıklı kouşmanın karakterlerini yakından tanıyalım. Böylece dinleyeceğimiz konuşmalar bize daha anlamlı gelecektir.

1. *JS Motoru*: Programın baştan sona derleme ve çalıştırılma işlemlerinden sorumludur.

2. *Derleyici*: JS Motorunun arkadaşlarından biridir. Onun yerine, parsing ve code-generation gibi pis ileri yapar. (önceki bölüme bakınız).

3. *Scope*: JS Motorunun diğer bir arkadaşıdır. Bütün tanımlayıcıların (değişkenlerin) arama listesini toplar ve bünyesinde barındırır, bu tanımlayıcılara çalıştırılacak kod tarafından, nasıl ulaşılabileceğinin kurallarını belirler ve uygular.

JavaScript'in nasıl çalıştığını tam anlamıyla anlayabilmek için, JS Motoru (ve arkadaşları) gibi düşünmeye başlayıp, onların sorduğu soruları sorup onlar gibi cevaplamaya çalışın.

### Back & Forth

`var a = 2;` kodunu gördüğünüzde kafanızda sadece bir ifade canlanıyor olmalı. Ancak, *JS Motoru* için işler hiçde öyle değil. *JS Motoru* iki farklı ifade görür, biri derleyicinin derleme esnasında ilgilendiği, bir diğeri de *JS Motoru*'nun yürütme sırasında ilgilendiğidir.

Şimdi isterseniz *JS Motoru*'nun ve arkadaşlarının `var a = 2;` programına nasıl yaklaşacaklarını izleyelim.

*Derleyici*'nin yapacağı ilk şey, kodu tokenlara parçalamak ve bir ağaç yapısı oluşturmak olacaktır. *Derleyici* kod oluşturma aşamasına geldiğinde, programa beklenilenden farklı davranır.

Makul bir varsayımla *Derleyici*'nin şu şekilde davranacağı düşünülebilir: "Bellekte bir değişken için yer açar ve buna `a` ismini verir ve daha sonra bu değişkene `2` değerini veirir." Malesef bu varsayım çokta doğru değildir.

Bu aşamada *Derleyici* şu şekilde çalışır:

1. `var a` ifadesiyle karşılaşan *Derleyici*, *Scope*'a mevcut scope listesi içerisinde `a` isimli bir değişkenin olup olmadığını sorar. Eğer varsa *Derleyici* bu tanımlamayı yok sayar ve yoluna devam eder. Eğer yoksa *Derleyici*, *Scope*'a mevcut scope listesi içerisinde `a` isimli bir değişken tanımlamasını söyler.

2. Derleyici, daha sonra, `a = 2` atamasını işleyecek şekilde, *JS Motoru* için kod üretir. Bu aşamadan sonra *JS Motoru*'nun çalıştıracağı kod, ilk olarak *Scope*'a mevcut scope listesi içerisinde `a` isimli bir değişken erişilebilir halde bulunuyormu diye sorar. Eğer bulunuyorsa *JS Motoru* bu değişkeni kullanır, bulunmuyorsa başka bir yere (aşağıdaki *Nested Scope* bölümüne bakınız) bakar.

Eğer *JS Motoru* eninde sonunda bir `a` değişkeni bulursa ona `2` değerini atar, bulamazsa elini kaldırır ve bir hata var diye bağırır (hata fırlatır).

Özetlemek için: Değişken tanımlaması için iki farklı eylem otaya konur: Birincisi, *Derleyici* bir değişken tanımlar (eğer mevcut scope içerisinde daha önce tanımlanmamışsa), ikincisi, çalışma anında, *JS Motoru* mevcut scope'da değişkeni arar ve bulursa değerini atar.

### Derleyicinin Konuşması

Durumu daha iyi anlayabilmek için derleyicinin terminolojisini biraz daha iyi kavramamıza ihtiyaç var.

*JS Motoru*, *Derleyici*'nin adım (2) de ürettiği kodu çalıştırırken, *Scope*'a danışarak `a` değişkeninin tanımlı olup olmadığına araştırması lazım. Ancak *JS Motoru*'nun yaptığı bu araştırmanın türü araştırmanın sonucunu etkileyecektir.

Mevcut durumda, *JS Motoru*'nun, `a` değişkeni için "LHS" türünde bir arama yaptığı söylenebilir. Diğer arama türünede RHS diyeceğiz.

Burada LHS'nin "Left Hand Side - Sol Taraf" ve RHS'nin de "Right-hand Side - Sağ Taraf" anlamına geldiğini tahmin etmişsinizdir.

In other words, an LHS look-up is done when a variable appears on the left-hand side of an assignment operation, and an RHS look-up is done when a variable appears on the right-hand side of an assignment operation.

Actually, let's be a little more precise. An RHS look-up is indistinguishable, for our purposes, from simply a look-up of the value of some variable, whereas the LHS look-up is trying to find the variable container itself, so that it can assign. In this way, RHS doesn't *really* mean "right-hand side of an assignment" per se, it just, more accurately, means "not left-hand side".

Being slightly glib for a moment, you could also think "RHS" instead means "retrieve his/her source (value)", implying that RHS means "go get the value of...".

Let's dig into that deeper.

When I say:

```js
console.log( a );
```

The reference to `a` is an RHS reference, because nothing is being assigned to `a` here. Instead, we're looking-up to retrieve the value of `a`, so that the value can be passed to `console.log(..)`.

By contrast:

```js
a = 2;
```

The reference to `a` here is an LHS reference, because we don't actually care what the current value is, we simply want to find the variable as a target for the `= 2` assignment operation.

**Note:** LHS and RHS meaning "left/right-hand side of an assignment" doesn't necessarily literally mean "left/right side of the `=` assignment operator". There are several other ways that assignments happen, and so it's better to conceptually think about it as: "who's the target of the assignment (LHS)" and "who's the source of the assignment (RHS)".

Consider this program, which has both LHS and RHS references:

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

The last line that invokes `foo(..)` as a function call requires an RHS reference to `foo`, meaning, "go look-up the value of `foo`, and give it to me." Moreover, `(..)` means the value of `foo` should be executed, so it'd better actually be a function!

There's a subtle but important assignment here. **Did you spot it?**

You may have missed the implied `a = 2` in this code snippet. It happens when the value `2` is passed as an argument to the `foo(..)` function, in which case the `2` value is **assigned** to the parameter `a`. To (implicitly) assign to parameter `a`, an LHS look-up is performed.

There's also an RHS reference for the value of `a`, and that resulting value is passed to `console.log(..)`. `console.log(..)` needs a reference to execute. It's an RHS look-up for the `console` object, then a property-resolution occurs to see if it has a method called `log`.

Finally, we can conceptualize that there's an LHS/RHS exchange of passing the value `2` (by way of variable `a`'s RHS look-up) into `log(..)`. Inside of the native implementation of `log(..)`, we can assume it has parameters, the first of which (perhaps called `arg1`) has an LHS reference look-up, before assigning `2` to it.

**Note:** You might be tempted to conceptualize the function declaration `function foo(a) {...` as a normal variable declaration and assignment, such as `var foo` and `foo = function(a){...`. In so doing, it would be tempting to think of this function declaration as involving an LHS look-up.

However, the subtle but important difference is that *Compiler* handles both the declaration and the value definition during code-generation, such that when *Engine* is executing code, there's no processing necessary to "assign" a function value to `foo`. Thus, it's not really appropriate to think of a function declaration as an LHS look-up assignment in the way we're discussing them here.

### Engine/Scope Conversation

```js
function foo(a) {
	console.log( a ); // 2
}

foo( 2 );
```

Let's imagine the above exchange (which processes this code snippet) as a conversation. The conversation would go a little something like this:

> ***Engine***: Hey *Scope*, I have an RHS reference for `foo`. Ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it just a second ago. He's a function. Here you go.

> ***Engine***: Great, thanks! OK, I'm executing `foo`.

> ***Engine***: Hey, *Scope*, I've got an LHS reference for `a`, ever heard of it?

> ***Scope***: Why yes, I have. *Compiler* declared it as a formal parameter to `foo` just recently. Here you go.

> ***Engine***: Helpful as always, *Scope*. Thanks again. Now, time to assign `2` to `a`.

> ***Engine***: Hey, *Scope*, sorry to bother you again. I need an RHS look-up for `console`. Ever heard of it?

> ***Scope***: No problem, *Engine*, this is what I do all day. Yes, I've got `console`. He's built-in. Here ya go.

> ***Engine***: Perfect. Looking up `log(..)`. OK, great, it's a function.

> ***Engine***: Yo, *Scope*. Can you help me out with an RHS reference to `a`. I think I remember it, but just want to double-check.

> ***Scope***: You're right, *Engine*. Same guy, hasn't changed. Here ya go.

> ***Engine***: Cool. Passing the value of `a`, which is `2`, into `log(..)`.

> ...

### Quiz

Check your understanding so far. Make sure to play the part of *Engine* and have a "conversation" with the *Scope*:

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

2. Identify all the RHS look-ups (there are 4!).

**Note:** See the chapter review for the quiz answers!

## Nested Scope

We said that *Scope* is a set of rules for looking up variables by their identifier name. There's usually more than one *Scope* to consider, however.

Just as a block or function is nested inside another block or function, scopes are nested inside other scopes. So, if a variable cannot be found in the immediate scope, *Engine* consults the next outer containing scope, continuing until found or until the outermost (aka, global) scope has been reached.

Consider:

```js
function foo(a) {
	console.log( a + b );
}

var b = 2;

foo( 2 ); // 4
```

The RHS reference for `b` cannot be resolved inside the function `foo`, but it can be resolved in the *Scope* surrounding it (in this case, the global).

So, revisiting the conversations between *Engine* and *Scope*, we'd overhear:

> ***Engine***: "Hey, *Scope* of `foo`, ever heard of `b`? Got an RHS reference for it."

> ***Scope***: "Nope, never heard of it. Go fish."

> ***Engine***: "Hey, *Scope* outside of `foo`, oh you're the global *Scope*, ok cool. Ever heard of `b`? Got an RHS reference for it."

> ***Scope***: "Yep, sure have. Here ya go."

The simple rules for traversing nested *Scope*: *Engine* starts at the currently executing *Scope*, looks for the variable there, then if not found, keeps going up one level, and so on. If the outermost global scope is reached, the search stops, whether it finds the variable or not.

### Building on Metaphors

To visualize the process of nested *Scope* resolution, I want you to think of this tall building.

<img src="fig1.png" width="250">

The building represents our program's nested *Scope* rule set. The first floor of the building represents your currently executing *Scope*, wherever you are. The top level of the building is the global *Scope*.

You resolve LHS and RHS references by looking on your current floor, and if you don't find it, taking the elevator to the next floor, looking there, then the next, and so on. Once you get to the top floor (the global *Scope*), you either find what you're looking for, or you don't. But you have to stop regardless.

## Errors

Why does it matter whether we call it LHS or RHS?

Because these two types of look-ups behave differently in the circumstance where the variable has not yet been declared (is not found in any consulted *Scope*).

Consider:

```js
function foo(a) {
	console.log( a + b );
	b = a;
}

foo( 2 );
```

When the RHS look-up occurs for `b` the first time, it will not be found. This is said to be an "undeclared" variable, because it is not found in the scope.

If an RHS look-up fails to ever find a variable, anywhere in the nested *Scope*s, this results in a `ReferenceError` being thrown by the *Engine*. It's important to note that the error is of the type `ReferenceError`.

By contrast, if the *Engine* is performing an LHS look-up and arrives at the top floor (global *Scope*) without finding it, and if the program is not running in "Strict Mode" [^note-strictmode], then the global *Scope* will create a new variable of that name **in the global scope**, and hand it back to *Engine*.

*"No, there wasn't one before, but I was helpful and created one for you."*

"Strict Mode" [^note-strictmode], which was added in ES5, has a number of different behaviors from normal/relaxed/lazy mode. One such behavior is that it disallows the automatic/implicit global variable creation. In that case, there would be no global *Scope*'d variable to hand back from an LHS look-up, and *Engine* would throw a `ReferenceError` similarly to the RHS case.

Now, if a variable is found for an RHS look-up, but you try to do something with its value that is impossible, such as trying to execute-as-function a non-function value, or reference a property on a `null` or `undefined` value, then *Engine* throws a different kind of error, called a `TypeError`.

`ReferenceError` is *Scope* resolution-failure related, whereas `TypeError` implies that *Scope* resolution was successful, but that there was an illegal/impossible action attempted against the result.

## Review (TL;DR)

Scope is the set of rules that determines where and how a variable (identifier) can be looked-up. This look-up may be for the purposes of assigning to the variable, which is an LHS (left-hand-side) reference, or it may be for the purposes of retrieving its value, which is an RHS (right-hand-side) reference.

LHS references result from assignment operations. *Scope*-related assignments can occur either with the `=` operator or by passing arguments to (assign to) function parameters.

The JavaScript *Engine* first compiles code before it executes, and in so doing, it splits up statements like `var a = 2;` into two separate steps:

1. First, `var a` to declare it in that *Scope*. This is performed at the beginning, before code execution.

2. Later, `a = 2` to look up the variable (LHS reference) and assign to it if found.

Both LHS and RHS reference look-ups start at the currently executing *Scope*, and if need be (that is, they don't find what they're looking for there), they work their way up the nested *Scope*, one scope (floor) at a time, looking for the identifier, until they get to the global (top floor) and stop, and either find it, or don't.

Unfulfilled RHS references result in `ReferenceError`s being thrown. Unfulfilled LHS references result in an automatic, implicitly-created global of that name (if not in "Strict Mode" [^note-strictmode]), or a `ReferenceError` (if in "Strict Mode" [^note-strictmode]).

### Quiz Answers

```js
function foo(a) {
	var b = a;
	return a + b;
}

var c = foo( 2 );
```

1. Identify all the LHS look-ups (there are 3!).

	**`c = ..`, `a = 2` (implicit param assignment) and `b = ..`**

2. Identify all the RHS look-ups (there are 4!).

    **`foo(2..`, `= a;`, `a + ..` and `.. + b`**


[^note-strictmode]: MDN: [Strict Mode](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/Strict_mode)

# Swift 4 中的泛型
> 这是我基于英文原文翻译的译文，如果你对本文感兴趣而且想转发，你应该在转发文章里加上本文的[链接](https://github.com/britzlieg/translate_post/blob/master/2017-10/%E4%BD%BF%E7%94%A8%22Core%22%E6%9E%B6%E6%9E%84iOS%20Apps.md)
>
> 译者：[britzlieg](https://github.com/britzlieg)
>
> [英文原文链接](https://github.com/britzlieg/translate_post/blob/master/2017-10/Swift%204%E4%B8%AD%E7%9A%84%E6%B3%9B%E5%9E%8B.md)

![](https://user-gold-cdn.xitu.io/2017/10/13/3bb62edfdcc6fe74542689c21be79093)

作为Swift中最重要的特性之一，泛型使用起来很巧妙。很多人都不太能理解并使用泛型，特别是应用开发者。泛型最适合libraries, frameworks, and SDKs的开发。在这篇文章中，我将用不同于其他教程的角度来讲解泛型。我们将使用餐馆的例子，这个餐馆能从SwiftCity的城市理事会中获得授权。为了保持简洁，我将内容控制在以下四个主题：

- 1、泛型函数和泛型类型
- 2、关联类型协议
- 3、泛型的Where语句
- 4、泛型下标

我们接下来看看具体怎么做！

## 泛型函数和泛型类型

### 开一家Swift餐馆

让我们新开张一家餐馆。当开张的时候，我们不仅关注餐馆的结构，也关注来自城市理事会的授权。更重要的，我们将关注我们的业务，以便于它功能化和有利可图。首先，怎么让一家公司怎么看上去像一个理事会？一个公司应该要有一些基础的功能。

```Swift

protocol Company {
  func buy(product: Product, money: Money)
  func sell(product: Product.Type, money: Money) -> Product?
}

```

*```buy```*函数把商品添加到库存中，并花费公司相应的现金。```sell```函数创建／查找所需花费的该类型商品，并返回出售的商品。

### 泛型函数

在这个协议中，```Product```如果是一个确定的类型的话不太好。把每一个```product```统一成一个确定的商品类型是不可能的。每个商品都有自己的功能，属性等。在这些各种类型的函数中，使用一个确定的类型是一个坏主意。让我们回到理事会那里看看。总而言之，不管是哪个公司，它都需要购买和卖出商品。所以，理事会必须找到适合这两个功能的一种通用的解决方案，以适合于每家公司。他们可以使用泛型来解决这个问题。

```Swift
protocol Company {
  func buy<T>(product: T, with money: Money)
  func sell<T>(product: T.Type, for money: Money) -> T?
}
```

我们把我们原来的确定类型```Product```用默认类型```T```来代替。这个类型参数```<T>```把这些函数定义成泛型。在编译时，默认类型会被确定类型替代。当buy和sell函数被调用时，具体类型就会被确定下来。这使得不同产品能灵活使用同一个函数。例如，我们在Swift餐馆中卖Penne Arrabiata。我们可以像下面一样直接调用```sell```函数：

```Swift
let penneArrabiata = swiftRestaurant.sell(product: PenneArrabiata.Self, for: Money(value:7.0, currency: .dollar))
```

在编译时，编译器用类型```PenneArrabiata```替换类型```T```。当这个方法在运行时被调用的时候，它已经时有一个确定的类型```PenneArrabiata```而不是一个默认的类型。但这带来另外一个问题，我们不能只是简单的买卖各种类型的商品，还要定义哪些商品时能够被合法买卖。这里就引入where类型约束。理事会有另一个协议```LegallyTradable```。它将检查和标记我们可以合法买卖的商品。理事会强制我们对所有买卖实行这个协议，并列举每一个符合协议的从商品。所以我们需要为我们的泛型函数添加约束，以限制只能买卖符合协议的商品。


```Swift
protocol Company {
  func buy<T: LegallyTradable>(product: T, with money: Money)
  func sell<T: LegallyTradable>(product: T.Type, for money: Money) -> T?
}
```

Now, we can use these functions with inner peace. Basically, we put a constraint on our placeholder type ```T``` with saying “Only products which conform to ```LegallyTradable``` can be parameters of our ```Company``` protocol’s functions”. This constraint is called protocol as a constraint in Swift. If one product is not conforming to this protocol, it can’t be used as a parameter in this function.

现在，我们可以放心用这些函数了。通常，我们把符合```LegallyTradable```协议的默认类型```T```作为我们```Company```协议函数的参数。这个约束被叫做Swift中的协议约束。如果一个商品不遵循这个协议，它将不能作为这个函数的参数。

### 泛型类型

我们把注意力转移到我们的餐馆上。我们得到授权并准备关注餐馆的管理。我们聘请了一位出色的经理和她想建立一套能跟踪商品库存的系统。在我们的餐馆中，我们有一个面食菜单，顾客喜欢各种各样的面食。这就是我们为什么需要一个很大的地方去存储面食。我们创建一个面食套餐列表，当顾客点套餐的时候，将套餐从列表中移除。无论何时，餐馆会买面食套餐，并把它加到我们的列表中。最后，如果列表中的套餐少于三个，我们的经理将订新的套餐。这是我们的```PastaPackageList```结构：

```Swfit
struct PastaPackageList {
  var packages: [PastaPackage]
 
  mutating func add(package: PastaPackage) {
    packages.append(item)
  }
 
  mutating func remove() -> PastaPackage {
    return packages.removeLast()
  }
 
  func isCapacityLow() -> Bool {
    return packages.count < 3
  }
}
```

过了一会，我们的经理开始考虑为餐馆中的每一样商品创建一个列表，以便更好的跟踪。与其每次创建独立列表结构，不如用泛型来避免这个问题。如果我们定义我们的库存列表作为一个泛型类，我们可以很容易使用同样的结构实现创建新的库存列表。与泛型函数一样，使用参数类型```<T>```定义我们的结构。所以我们需要用```T```默认类型来替代```PastaPackage```具体类型


```Swift
struct InventoryList<T> {
  var items: [T]
  
  mutating func add(item: T) {
    items.append(item)
  }
 
  mutating func remove() -> T {
    return items.removeLast()
  }
  
  func isCapacityLow() -> Bool {
    return items.count < 3
  }
}
```

这些泛型类型让我们可以为每个商品创建不同的库存列表，而且使用一样的实现。

```Swift
var pastaInventory = InventoryList<PastaPackage>()
pastaInventory.add(item: PastaPackage())
var tomatoSauceInventory = InventoryList<TomatoSauce>()
var flourSackInventory = InventoryList<FlourSack>()
```

泛型的另外一个优势是只要我们的经理需要额外的信息，例如库存中的第一种商品，我们都可以通过使用扩展来添加功能。Swift允许我们去写结构体，类和协议的扩展。因为泛型的扩展性，当我们定义结构体时，不需要提供类型参数。在扩展中，我们仍然用默认类型。让我们看看我们如何实现我们经理的需求。

```Swift
extension InventoryList { // We define it without any type parameters
  var topItem: T? {
    return items.last
  }
}
```

*```InventoryList```*中存在类型参数```T```作为类型```topItem```的遵循类型，而不需要再定义类型参数。现在我们有所有商品的库存列表。因为每个餐馆都要从理事会中获取授权去长时间存储商品，我们依然没有一个存储的地方。所以，我们把我们的关注点放到理事会上。

## 关联类型协议

我们再次回去到城市理事会去获取存储食物的允许。理事会规定了一些我们必须遵守的规则。例如，每家有仓库的餐馆都要自己清理自己的仓库和把一些特定的食物彼此分开。同样，理事会可以随时检查每间餐馆的库存。他们提供了每个仓库都要遵循的协议。这个协议不能针对特定的餐馆，因为仓库物品可以改变成各种商品，并提供给餐馆。在Swift中，泛型协议一般用关联类型。让我们看看理事会的仓库协议是怎么样的。


```Swift
protocol Storage {
  associatedtype Item
  var items: [Item] { set get }
  mutating func add(item: Item)
  var size: Int { get }
  mutating func remove() -> Item
  func showCurrentInventory() -> [Item]
}
```

*```Storage```*协议并没有规定物品怎么存储和什么类型被允许存储。在所有商店，实现了```Storage```协议的餐馆必须制定一种他们他们存储的特定类型的商品。这要保证物品从仓库中添加和移除的正确性。同样的，它必须能够完整展示当前仓库。所以，对于我们的仓库，我们的```Storage```协议如下所示：

```Swift
struct SwiftRestaurantStorage: Storage {
  typealias Item = Food // Optional
  var items = [Food]()
  var size: Int { return 100 }
  mutating func add(item: Food) { ... }
  mutating func remove() -> Food { ... }
  func showCurrentInventory() -> [Food] { ... }
}
```

我们实现理事会的```Storage```协议。现在看来，关联类型```Item```可以用我们的```Food```类型来替换。我们的餐馆仓库都可以存储```Food```。关联类型```Item```只是一个协议的默认类型。我们用```typealias```关键字来定义类型。但是，需要指出的是，这个关键字在Swift中是可选的。即使我们不用```typealias```关键字，我们依然可以用```Food```替换协议中所有用到```Item```的地方。Swift会自动处理这个。

### 限制关联类型为特定类型

事实上，理事会总是会想出一些新的规则并强制你去遵守。一会后，理事会改变了```Storage```协议。他们宣布他们将不允许任何物品在```Storage```。所有物品必须遵循StorableItem协议，以保证他们都适合存储。换句话，它们都限制为关联类型```Item```


```Swift
protocol Storage {
  associatedtype Item: StorableItem // Constrained associated type
  var items: [Item] { set get }
  var size: Int { get }
  mutating func add(item: Item)
  mutating func remove() -> Item
  func showCurrentInventory() -> [Item]
}
```

用这个方法，理事会限制类型为当前关联类型。任何实现```Storage```协议的都必须使用实现```StorableItem```协议的类型。

## 泛型的Where语句

### 使用泛型的Where语句的泛型

让我们回到文章刚开始的时候，看看```Company```协议中的```Money```类型。当我们讨论到协议时，买卖中的money参数事实上是一个协议。

```Swift
protocol Money {
  associatedtype Currency
  var currency: Currency { get }
  var amount: Float { get }
  func sum<M: Money>(with money: M) -> M where M.Currency == Currency
}
```

然后，再过了一会，理事会打回了这个协议，因为他们有另一个规则。从现在开始，交易只能用一些特定的货币。在这个之前，我们能各种用```Money```类型的货币。不同于每种货币定义money类型的做法，他们决定用```Money```协议来改变他们的买卖函数。

```Swift
protocol Company {
  func buy<T: LegallyTradable, M: Money>(product: T.Type, with money: M) -> T? where M.Currency: TradeCurrency
  func sell<T: LegallyTradable, M: Money>(product: T, for money: M) where M.Currency: TradeCurrency
}
```

The difference between where clause and type constraint is where clauses are used to define requirements in associated type. In other words, we are not constraining the associated type inside the protocol. Instead, we’re constraining it while using the protocol.

where语句和类型约束的where语句的区别在于，where语句会被用于定义关联类型。换句话，在协议中，我们不能限制关联的类型，而会在使用协议的时候限制它。

### 泛型的where语句的扩展

泛型的where语句在扩展中有其他用法。例如，当理事会要求用漂亮的格式（例如“xxx EUR”）打印money时，他们只需要添加一个```Money```的扩展，并把```Currency```限制设置成```Euro``。


```Swift
extension Money where Currency == Euro {
  func printAmount() {
    print("\(amount) EUR")
  }
}
```

泛型的where语句允许我们添加一个新的必要条件到```Money```扩展中，因此只有当```Currency```是```Euro```时，扩展才会添加```printAmount()```方法。

### 泛型的where 语句的关联类型

在上文中，理事会给```Storage```协议做了一些改进。当他们想检查一切是否安好，他们想列出每一样物品，并控制他们。控制进程对于每个```Item```是不一样的。因为这样，理事会仅仅需要提供```Iterator```关联类型到```Storage```协议中。


```Swift
protocol Storage {
  associatedtype Item: StorableItem
  var items: [Item] { set get }
  var size: Int { get }
  mutating func add(item: Item)
  mutating func remove() -> Item
  func showCurrentInventory() -> [Item]
 
  associatedtype Iterator: IteratorProtocol where Iterator.Element == Item
  func makeIterator() -> Iterator
}
```

*```Iterator```*协议有一个叫```Element``的关联类型。在这里，我们给它加上一个必要条件，在```Storage```协议中，```Element```必须与```Item```类型相等。

## 泛型下标

来自经理和理事会的需求看起来是无穷无尽的。同样的，我们需要满足他们的要求。我们的经理跑过来跟我们说她想要用一个```Sequence```来访问存储的物品，而不需要访问所有的物品。经理想要个语法糖。


```Swift
extension Storage {
  subscript<Indices: Sequence>(indices: Indices) -> [Item] where Indices.Iterator.Element == Int {
    var result = [Item]()
    for index in indices {
      result.append(self.items[index])
    }
    return result
  }
}
```

在Swift 4中，下标也可以是泛型，我们可以用条件泛型来实现。在我们的使用中，```indices```参数必须实现```Sequence```协议。[从Apple doc](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html)中可以知道，“The generic ```where``` clause requires that the iterator for the sequence must traverse over elements of type ```Int```.”这就保证了在sequence的indices跟存储中的indices是一致的。


## 结语

我们让我们的餐馆功能完备。我们的经理和理事会看起来也很高兴。正如我们在文章中看到的，泛型是很强大的。我们可以用泛型来满足各种敏感的需求，只要我们知道概念。泛型在Swift的标准库中也应用广泛。例如，[Array](https://developer.apple.com/documentation/swift/array) 和 [Dictionary](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/CollectionTypes.html#//apple_ref/doc/uid/TP40014097-CH8-ID113)类型都是泛型集合。如果你想知道更多，你可以看看这些类是怎么实现的。 [Swift Language Doc](https://developer.apple.com/library/content/documentation/Swift/Conceptual/Swift_Programming_Language/Generics.html) 也提供了泛型的解析。最近Swift语言提供了泛型的一些说明[Generic Manifesto](https://github.com/apple/swift/blob/master/docs/GenericsManifesto.md)。我建议你去看完所有的文档，以便更好的理解当前用法和未来的规划。感谢大家的阅读！如果你对接下来的文章有疑惑，建议，评论或者是想法，清在 [Twitter](https://twitter.com/candostEN)  联系我，或者评论！你也可以在[GitHub](https://github.com/candostdagdeviren)上关注我哦！

> 本文[Github地址](https://github.com/britzlieg/translate_post/blob/master/2017-10/Swift%204%E4%B8%AD%E7%9A%84%E6%B3%9B%E5%9E%8B.md)


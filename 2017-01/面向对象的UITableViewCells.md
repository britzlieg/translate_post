# 面向协议的UITableViewCells
> 这是我基于英文原文翻译的译文，如果你对本文感兴趣而且想转发，你应该在转发文章里加上本文的[链接]()。
> [英文原文链接](https://medium.com/ios-os-x-development/protocol-oriented-uitableviewcells-6efa7ef8c45b#.vqr68egqf)

这篇文章将展示如何通过面向协议编程（POP）而不是通过类继承，来实现不同UITableViewCell对象的展示。准备好了吗？让我们一起来看看具体怎么做！

cells展示！

当我开发[Cast Player](http://castplayerapp.com/)这个app的时候，我需要添加很多设置页面来让用户调整app的一些设置，其中就包括一个反馈表格和一些关于界面。下面是这个界面的展示：

![](https://cdn-images-1.medium.com/max/800/1*pOvZMcD-I0M1uGWG54ZraQ.gif)

从上面的界面展示可以看到，我们要定义6个不同类型的cells：

![](https://cdn-images-1.medium.com/max/800/1*EK1gWLRyT7DhBVpOznAchQ.png)

总而言之，这些cells都有以下的特点（或行为）：

- Cell 高亮
- 展示一个标题
- 展示一个可读的计数

我们怎么创建能代表这6种不同类型的cells类呢？第一步应该要做要做的，就是使用表格列出所有cell的类型和特点。

![](https://cdn-images-1.medium.com/max/800/1*C3T72IQvaOldegtV-U8EMw.png)

从表格中我们可以看到：

- 没有一个特性满足所有的cells。
- 一些特性满足一些cells，但是并不满足其他的cells

这就意味着通过类继承来实现一个UITableViewCell并不是一个好的选择。事实上，这些cell 类型中没有一个适合作为基类。你认为呢？🤔

## 协议 & 扩展！Yay！

幸运的，[这篇文章](http://machinethink.net/blog/mixins-and-traits-in-swift-2.0/)给我们提供了一个解决的思路。对于这种使用场景，协议和扩展将会非常有用，但是应该怎么去使用它们呢？

这个做法是这样的，我们可以使用协议为每一个特性定义接口，并通过扩展提供接口默认的实现。然后，我们可以创建一个轻量级的UITableViewCell子类，并让它遵循所需要的接口规范。在一篇叫[《面向协议的MVVM介绍》](https://realm.io/news/doios-natasha-murashev-protocol-oriented-mvvm/)中有一个非常类似的问题。我们可以看看这个具体是怎么做的！

使用协议和扩展，我们可以写第一个特性的代码，TitlePresentable：

```
protocol TitlePresentable {
  var titleLabel: UILabel! { get set }
  func setTitle(title: String?)
}

extension TitlePresentable {
  func setTitle(title: String?) {
    titleLabel.text = title
  }
}
```

第二个特性，BytesCountPresentable使用一样的模式：

```
protocol BytesCountPresentable {
  var bytesCountLabel: UILabel! { get set }
  func setBytesCount(bytesCount: Int64?)
}

extension BytesCountPresentable {
  func setBytesCount(bytesCount: Int64?) {
    bytesCountLabel.text = bytesCountString(bytesCount)
  }
  private func bytesCountString(bytesCount: Int64?) -> String {
    if let bytesCount = bytesCount {
      return bytesCount.bytesFormattedString() // defined elsewhere
    }
    return "N/A"
  }
}
```

通过让这两个特性分成两个不同的协议，BytesCountTitleTableViewCell的实现就变得简单了。


```
class BytesCountTitleTableViewCell: UITableViewCell, TitlePresentable, BytesCountPresentable {
  @IBOutlet var titleLabel: UILabel!
  @IBOutlet var bytesCountLabel: UILabel!
}
```

通过遵循TitlePresentable和BytesCountPresentable协议，BytesCountTitleTableViewCell能够继承通过扩展添加的行为。其他类可以通过一样的接口来继承同样的行为。这真的非常强大！💪💪

注意到类需要重新声明titleLabel和bytesCountLabel两个属性，以便于能够实现对应的协议，但我们也需要指定这些IBOutlet变量，方便我们可以通过Interface Builder来链接它们。

## highlighting特性怎么办？
我们先从这个类开始：

```
class HighlightableTableViewCell: UITableViewCell {
  override func setHighlighted(highlighted: Bool, animated: Bool) {
    self.contentView.backgroundColor = highlighted ? UIColor(white: 217.0/255.0, alpha: 1.0) : nil
  }
}
```

这里我们选择这样做，通过复写UITablViewCell的setHighlighted()方法来实现cell 高亮。

根据上面总结的方法，我们可以定义一个HighlightableView的协议和扩展：

```
protocol HighlightableView {
  func setHighlighted(highlighted: Bool, animated: Bool)
}

extension HighlightableView {
  func setHighlighted(highlighted: Bool, animated: Bool) {
    print("extension highlighted") // not printed
  }
}
```

我们可以尝试创建一个实现HighlightableView 协议的UITableViewCell的子类，并看看高亮特性是否可以实现。但这次没那么幸运了。🚫

我认为这是因为当setHighlighted()方法被调用的时候，那些UITableViewCell的基类方法将会被调用，而不会调用协议扩展方法。

在一般情况下，协议扩展是为了在已有的类中加入新的行为，而不能复写已有方法。

为了Cell的高亮，再来一遍！

为了让cell的高亮特性能正常展示，如果必要的话，我们可以保留HighlightableTableViewCell的定义并从它那里派生出一个子类。例如：

```
class DisclosureTitleTableViewCell: HighlightableTableViewCell, TitlePresentable {
  @IBOutlet var titleLabel: UILabel!
}

class DisclosureBytesCountTitleTableViewCell: HighlightableTableViewCell, TitlePresentable, BytesCountPresentable {
  @IBOutlet var titleLabel: UILabel!
  @IBOutlet var bytesCountLabel: UILabel!
}
```

当我们选择要使用HighlightableTableViewCell或UITableViewCell作为基类时，取决于我们是否需要cell高亮特性，并根据需要使用TitlePresentable和BytesCountPresentable的特性。

类的层级如下图所示：

![](https://cdn-images-1.medium.com/max/800/1*zxVU-JAV1c-5J6L8wpiV_A.jpeg)

在代码中是这样的：

```
class HighlightableTableViewCell: UITableViewCell
 
class DisclosureTitleTableViewCell: HighlightableTableViewCell, TitlePresentable
 
class BytesCountTitleTableViewCell: UITableViewCell, TitlePresentable, BytesCountPresentable
 
class DisclosureBytesCountTitleTableViewCell: HighlightableTableViewCell, TitlePresentable, BytesCountPresentable
 
class ActionTitleTableViewCell: HighlightableTableViewCell, TitlePresentable
 
class ValuePickerTableViewCell: UITableViewCell
 
class ValuePickerLabelTableViewCell: ValuePickerTableViewCell, TitlePresentable
```

注意：HighlightableTableViewCell 是唯一的通过子类实现，并充当其他三个额外cell类型基类的类。一旦一个给定的类型被选择作为基类，额外的特性只能通过协议扩展添加。

## 结论

Swift的协议和扩展，是一种能添加行为（特性）的方法。这种方法在类设计上很有效，能保持类层级扁平化。🚀主要优势如下：

- 更扁平化的类层级
- API/类更容易扩展
- 相对修改代码，修改协议更容易
- 更少的代码重复

## 反馈

文章中展示的解决方法，对于我和我的特殊使用场景来说非常有效果 - 我希望这个实践例子能够帮助大家理解协议扩展是怎么使用的，最好对比下我刚开始做的那个继承例子。如果你知道更好的实现方法，可以在我的评论下留言。

注意：这篇[文章](http://bizz84.github.io/2016/06/18/Protocol-Oriented-UITableViewCells.html)最早发表于我的博客中，时间为2016年6月18日。

如果想看更多类似的文章，请订阅我的[邮件订阅](http://eepurl.com/cmi6rD)。

如果你喜欢这篇文章，请点击喜欢（💚）以便让更多人在Medium中看到。

还有，不要忘记下载我的[Cast Player](http://castplayerapp.com/)应用哦. 😇

> 如果你喜欢这篇译文，记得在[简书中给我点赞并关注我哦](http://www.jianshu.com/u/64d47b1e0fc9)
> 
> 本文[Github地址]()
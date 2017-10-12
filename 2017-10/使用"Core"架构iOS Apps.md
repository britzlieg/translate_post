# 使用"Core"架构iOS Apps
> 这是我基于英文原文翻译的译文，如果你对本文感兴趣而且想转发，你应该在转发文章里加上本文的[链接](https://github.com/britzlieg/translate_post/blob/master/2017-10/%E4%BD%BF%E7%94%A8%22Core%22%E6%9E%B6%E6%9E%84iOS%20Apps.md)
> 
> [英文原文链接](https://theswiftpost.co/architecting-ios-apps-core/)

在过去的两年，我有机会经历了不同的架构模式，例如```MVC```，```MVVM```和```VIPER```。这些都有一个通用的```V```组件，再我们的应用中代表的是视图。在一个完美的设计中，视图组件只做以下事情：

- 1、传递用户行为（触摸事件）到业务层。
- 2、监听状态变化和进行更新。

除了这些，没有其他职责。不同架构中，视图组件的功能职责是一致的。这个组件简单，独立，因此很容易被替代。但事实上，真的如此吗？看一下你所有的实现，然后告诉我...再大多数iOS应用中，它们是最难处理的组件。我们把每个智能组件（例如view model，presenter 和 interactor）注入到 视图控制器中。这就意味着如果一个视图控制器挂了，那些智能组件也一样跟着挂掉。视图就像是一个笨重的肌肉男，主宰着整个生命周期的数据流动。难道就没有其他的方案吗？

我不认为这是一个架构定义的问题，但我不能说出它们是怎么被实现的。

### 我们真的尝试去实现什么吗？

如果我们考虑到将来的扩展，一个理想的架构应该看起来是这样的：

![](https://i2.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-WfPvtPKszh_kXUsqtcwi8A.png?w=1600&ssl=1)

用户信息流

假设UI组件符合预期设计，```Core```包含所有业务逻辑，我们就真的可以移除UI，然后用任何东西去代替...例如一组测试套件。

![](https://i1.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-dG-eIlLIiDW0-yPENib4Bw.png?zoom=2&resize=912%2C366&ssl=1)

如果我们能做到这样，我们就可以用一个测试套件模拟每一个用例，而不需要任何的UI。我们看一下登录用例：

#### 登录（从用户角度）：

- 1、输入用户名和密码。
- 2、点击登录

#### 登录（测试套件）：

```core.dispatch(LoginCommand(username: "goksel", password: "123"))```

这看起来很爽吧，对不对？

### 进入 “Core” 的世界

![](https://i2.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-xZByvaEcUx-YoSAfezJWrg.jpeg?zoom=2&resize=912%2C608&ssl=1)


我们的目标是设计一个业务层，这个业务层不需要任何UI组件即可展示所有的app状态。显而易见，这不太可能，除非我们不依赖视图的生命周期。所以我们需要另外一层，这层就像我们app的生命之源。我讲这层叫”Core“。在我们的产品中，我们一般有以下功能特性：登录，注册，电影列表，电影细节等等。所有的这些功能特性应该是一个非常理想的自运行的组件，所有的状态变化都由Core来管理。最后整个流程如下图所示：

![](https://i1.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-2Wg7DttPV12OXxmtjUL8bw.png?zoom=2&resize=912%2C522&ssl=1)

从上图可以看到：

- 1、Core是一个能模拟我们app的盒子
- 2、Actions 是发生在系统中的变化。它们贯穿整个Core和触发组件状态改变。
- 3、组件包含功能特性的业务逻辑，并能运行这些actions
- 4、订阅者可以是任何东西。它可以是一个控制台的app，可以是一个iOS的App，也可以是我们的测试套件。

简单吧，是不是？

### 说是没用的，给我看看你的代码

这理论上（总是）听起来很好，但实际上是不是很容易实现的呢？让我们实现一个包括两步认证的简单登录界面。

![](https://i0.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-8-MreAYOUvLE7XWsJ52Ghw.png?zoom=2&resize=563%2C999&ssl=1)

#### 要求：

允许用户有60秒时间去输入安全码。超时则返回。

- 1、只要用户点击了登录按钮，使用网络验证安全码。
- 2、如果验证成功，跳转进home组件。
- 3、如果验证不成功，弹出一个警告框。
- 4、当网络请求还在进行时，显示一个loading指示器。

够简单吧。让我们看看具体怎么搞。

#### 1 - 定义状态

> 组件的状态由什么组成？我们需要在屏幕上渲染什么数据？

- 我们需要一个显示／隐藏loading指示器的标记。
- 我们需要跟踪计时器的状态。
- 我们需要跟踪验证的过程

```
struct LoginState: State {
  var isLoading = false
  var timerStatus: TimerStatus = .idle
  var verificationResult: Result<Void>?
}
```

#### 2 - 定义Actions

在屏幕上发生了什么了？

- 计时器计时
- 点击验证按钮

```
enum LoginAction: Action {
  case tick
  case verifyOTP(String)
}
```

#### 3 - 定义组件

现在我们定义组件。所有组件将有一个状态和一个运行函数。当一个action被core派发，所有的组件将收到通知，如果有必要会有所响应。如果把状态改变作为一个被派发的action，订阅了这个action的组件被新状态通知。

![](https://i2.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-P-W0ZcnnDSD39Uz_fet-gg-e1504957137883.gif?zoom=2&resize=472%2C354&ssl=1)

***注意***: 状态是组件类中的一个可读属性。然而，它可以通过```commit```这个函数来更新。这样设计是为了避免太频繁的状态更新。所以要改变状态，先创建一个拷贝，然后做出所有的变更，最后调用```commit```函数。这样就能设置一个新的状态，并且通知所有的订阅者。

```
class LoginComponent: Component<LoginState> {
  override func process(_ action: Action) {
    guard let action = action as? LoginAction else { return }
    switch action {
    case .tick:
      self.tick()
    case .verifyOTP(let code):
      self.verifyOTP(code)
    }
  }
  // ...
}
```


所以我们的组件就是这么简单。通过复写```process```函数，必要时我们可以对传进来的actions做出响应。正如你所想的一样，这个真正的魔术放生在```tick```和```verifyOTP```函数上。
让我们更深入一下：

```
func verifyOTP() {
  // Get a copy of the current state:
  var state = self.state
  // Start loading before network call:
  state.isLoading = true
  // Publish state:
  commit(state)
  // Start network call:
  service.verifyOTP { result in
    // Update state with response:
    state.isLoading = false
    state.timerStatus = .finished
    state.result = result
    // Publish new state (and navigation):
    switch result {
    case .success():
      // Navigate to home if success:
      let nav = BasicNavigation.push(HomeComponent(), from: self)
      self.commit(state, nav)
    case .failure(_):
      self.commit(state)
  }
}
```

虽然这个函数做了同步工作，由于提交了一个新的状态给组件产生状态传递，所以缺少一个完成的block。

#### 4 - 订阅状态

从UI视图的角度看，没有发生同步。每当状态可用时，视图只是得到了一个新的状态并进行更新。例如，LoginViewController看起来是这样的：

```
class LoginViewController: Subscriber {
  var component: LoginComponent! // Injected by previous component.
  func viewWillAppear(animated: Bool) {
    super.viewWillAppear(animated)
    component.subscribe(self)
  }
  func viewDidDisappear(animated: Bool) {
    super.viewDidDisappear(animated)
    component.unsubscribe(self)
  }
  // MARK: - Actions
  func verifyTapped(field: UITextField) {
    core.dispatch(LoginAction.verifyOTP(field.text))
  }
  // MARK: - Subscriber
  func update(with state: State) {
    guard let state = state as? LoginState else { return }
    // Update UI here.
  }
  func perform(_ navigation: Navigation) {
    // Perform navigation here.
  }
}
```

#### 5- Define “Core”

```Core```可以定义成一个全局属性。 

```let core = Core(rootComponent: LoginComponent())```

就像这样，在这个被定义后，我们可以很容易的派发各种action。🎉

### 底线


```Core```可以被看成是一个Redux，Flux和MVVM的混合体。如果我们使用```Core```来作为一个App的架构，我们将拥有类似Redux的组件交互和状态传递。然而，组件本身是非常类似于 MVVM 的 view model，非常直接和容易适应。

### ```Core```和```Redux```的主要区别

- ```Redux``` 是静态的。它需要你在编译时定义一个笨重的管理app状态的架构。如果你有重用多次的controllers，那将会特别困难。特别是，当你的应用数据流依赖于服务器响应，你将很难不通过入侵架构来在编译前定义app状态。```Core```是动态的。你仅仅需要为工作组件定义状态和actions。组件跳转将被动态处理。
- 在```Redux```中，组件间没有一个标准的导航实现。在```Core```中，你可以有原生的导航实现。
- ```Redux```专注于应用状态，而```Core```专注于独立组件状态。在这方面，我发现在一个独立组件中会比在一个笨重的应用中更容易使用。
- 在```Redux```中，因为状态是全局的，所以当一个界面从导航堆栈中被pop时，很容易忘记做状态清理。在```Core```中，因为每个组件存储它自己的状态，当你从一个组件树中移除一个组件的时候，状态会被导航机制自动处理。

![](https://i1.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-_0TFaTTM33jww7OC1fP8BQ.gif?zoom=2&w=1200&ssl=1)

### 为什么使用它？

因为对比其他架构，你能通过一样的努力获取很多益处。

- ```分离关注点```: 在```Core```中的每个变量函数都有明确的目的和清晰的定义。
- ```容易理解```: 它强制你模型化你的组件，以便其变得更容易理解. 通过为每个组件定义状态, 你也可以为它们制定文档。
- ```高度可测试```: 在```Core```中，UI只是状态存储的反映，你可以很容易地通过把UI替换成测试类来测试你的组件。
- ```可重用```: 你可以不通过UIKit来开发你的业务逻辑。因此它可以在不同平台上复用（iOS,macOS,watchOS,tvOS）。
- ```bug细节报告```: 你可以通过一个简单的中间件来记录用户行为，然后把行为堆栈记录到bug报告上。这样将会帮助你减少类似的crash，减少查bug的时间。我的意思是，减少很多查bug的时间。

然而，这并不是说```Core```比其他架构优越。在软件开发中，没有银弹，只有根据实际的需要做的妥协。

![](https://i1.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-ZYrSaLwfJbcXfS88jyPklg.gif?zoom=2&w=1200&ssl=1)

所以，请选择“正确”的架构：

- 保持自身的技术的更新，但要小心炒作的技术
- 定义好问题并根据问题选择符合实际的解决方案
- 最后...无论如何，请拒绝MVC

![](https://i0.wp.com/theswiftpost.co/wp-content/uploads/2017/09/1-5hVDI1CgvGxiC-eyFrKlkA.gif?zoom=2&w=1200&ssl=1)

### 感谢阅读
[```Core```](https://github.com/gokselkoksal/Core)的Swift实现现在已经放在Github上了。大家可以去试一下！在这里，我非常欢迎大家分享自己的见解，记得在Github上给我提issues哦。哈哈。❤️

> 译者：原文是架构类英文文章，比较抽象，建议结合源码看。[源码地址](https://github.com/gokselkoksal/Core)。

> 本文[Github地址](https://github.com/britzlieg/translate_post/blob/master/2017-10/%E4%BD%BF%E7%94%A8%22Core%22%E6%9E%B6%E6%9E%84iOS%20Apps.md)

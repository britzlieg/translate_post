# 轻松管理Swift项目中的各种环境
> 这是我基于英文原文翻译的译文，如果你对本文感兴趣而且想转发，你应该在转发文章里加上本文的[链接](https://github.com/britzlieg/translate_post/blob/master/2017-09/%E8%BD%BB%E6%9D%BE%E7%AE%A1%E7%90%86Swift%E9%A1%B9%E7%9B%AE%E4%B8%AD%E7%9A%84%E5%90%84%E7%A7%8D%E7%8E%AF%E5%A2%83.md)
> 
> [英文原文链接](https://medium.com/flawless-app-stories/manage-different-environments-in-your-swift-project-with-ease-659f7f3fb1a6)

![](https://cdn-images-1.medium.com/max/2000/1*Rk8JulyapCiTCUtLsnsEcQ.png)

想象一下当你完成开发和测试，准备提交到生产环境时的场景时，你会遇到这么一个问题：所有你的API keys，URL，icon或者其他启动配置都是开发环境的。所以在你发布你的app的时候，你不得不为了适应生产环境而把他们全部改过来。显然，这做法不太好。有时候，由于你的app太复杂，你可能会忘记一些改动，从而导致服务不能正常工作。

不像这些糟糕的做法，本文的做法将会根据我们的需要去配置不同的环境，并且改动配置非常简单。现在，我们将讨论几种最常见的环境配置管理的方法：

- 1、使用注释
- 2、使用全局变量或枚举
- 3、通过一个全局标记flag使用配置和schemes
- 4、通过 *.plist 文件使用配置和schemes

### 1、使用注释

当你有两个独立的环境，你的应用需要知道它应该使用那个环境。想象一下，你有生产，开发，演示三个环境和API接口。那么环境切换的最快最简单方法就是注释其他两个环境。

```
// MARK: - Development
let APIEndpointURL = "http://mysite.com/dev/api"
let analyticsKey = "jsldjcldjkcs"

// MARK: - Production
// let APIEndpointURL = "http://mysite.com/prod/api"
// let analyticsKey = "sdcsdcsdcdc"
// MARK: - Staging
// let APIEndpointURL = "http://mysite.com/staging/api"
// let analyticsKey = "lkjllnlnlk
```

这个方法非常暴力，混乱，让人看到都想哭。有时不需要关心代码质量和可维护性的时候，我会使用这种黑科技。换句话说，我强力建议不要使用这种方法。

### 2、使用全局变量或枚举

另一种流行的做法，是通过一个全局变量或枚举（这个会好一点）来处理不同的配置环境。你需要在某个地方（例如AppDelegate）声明您的三个不同环境的枚举和设置它们的值。

```

enum Environment {
    case development
    case staging 
    case production
}
 
let environment: Environment = .development
 
switch environment {
case .development:
    // set web service URL to development
    // set API keys to development
    print("It's for development")
case .staging:
    // set web service URL to staging
    // set API keys to development
    print("It's for staging")
case .production:
    // set web service URL to production
    // set API keys to production
    print("It's for production")
}

```

这种做法保证你可以只改变一处代码就能配置不同的环境。与之前的方法不同，这个会好很多。这种做法非常灵活，可读性更强，但它有很多局限性。首先，在不同环境下运行都只有一个相同的Bundle ID。这就意味着你讲不能同时装上不同环境的相同app。这种做法其实也是不太合适。

除此以外，这种做法也不能根据不同的环境改变app的icons。而且，如果你在提交app之前忘记更改这个全局变量，你肯定会出问题。

---

我们再看看其他两种更好的实现方法。这些方法对于新项目和旧项目都适合。跟着这个教程，你可以很容易将它应用到就项目中。

After applying these approaches your app will have the same codebase for every environment, but you will be able to have different icons and different Bundle ID’s for every configuration. The distribution process also will be very easy. And what is the most important, your managers and testers will be able to have all your environments as separated apps on their devices. So they will fully understand which version they’re trying out.

使用这些方法后，你的app只有一份代码库，但你可以根据不同环境配置有不同icons和Bundle ID。这会让你的发布变得非常简单。最重要的是，你的老大和测试可以有多个不同环境的相同app，因此他们非常清楚他们正在测试的是那个版本的应用。

### 3、通过一个全局标记flag使用配置和schemes

在这种做法中，我们将创建三个不同的配置，三个不同的schemes和与schemes相关的配置。为了演示这种做法，我将新建一个“Environments”项目，你也可以创建一个新项目或者在老项目中做。

在项目导航的面板中找到项目设置，在targets的列表中右击已存在的target，然后选择 Duplicate 来复制当前target。

![](https://cdn-images-1.medium.com/max/1600/0*kJt7iX0pJ_OCbYH7.)

现在我们有不止一个的target和叫‘Environments copy’的build scheme。我们先把它们改成对应的名字。点击您的新target和按下“Enter”键，改target的名字为“Environments Dev”。然后，去到“Manage Schemes…”，选择刚才新创建的scheme，按下“Enter”，重命名你的target，避免命名混乱。

![](https://cdn-images-1.medium.com/max/1600/0*pAV3RMB8AJBsTIgL.)


同样的，我们可以新建一个icon asset，这样测试和老大就能知道打开的是哪个app配置。去到Assets.xcassets，点击 “+” ，选择“New iOS App Icon”。修改它的名字为“AppIcon-Dev”。

![](https://cdn-images-1.medium.com/max/1600/0*Wuq-Rd6IHVMAgTm0.)

现在我们要建立新的icons asset和我们开发环境的联系。去到“Targets”，点击你的开发环境，找到“App Icon Source”选择新的icons asset。

![](https://cdn-images-1.medium.com/max/1600/0*LyxuDi3gg8Ca69p7.)

就是这样，现在你有每个环境的不同icons。请注意，当我们新建第二个配置时，第二个 *.plist 文件也会生成出来。

值得注意，我们现在有两种不同的做法去处理两种不同的配置：

- 1、添加一个预编译宏标记去区分生产和开发targets。
- 2、在 *plist 文件中添加一个变量。

我们尝试第一种方法。我们添加一个标记变量去标记选择的开发环境。在“Build Settings”中，找到“Swift Compiler — Custom Flags”这个选项。设置 -DEVELOPMENT 来标记这个target是开发的build。

![](https://cdn-images-1.medium.com/max/1600/0*Henhnxiv07NEtDkk.)

然后配置的代码就像如下所示：

```

#if DEVELOPMENT
let SERVER_URL = "http://dev.server.com/api/"
let API_TOKEN = "asfasdadasdass"
#else
let SERVER_URL = "http://prod.server.com/api/"
let API_TOKEN = "fgbfkbkgbmkgbm"
#endif

```

现在，如果你选择 Dev 的scheme运行，你就自动使用开发配置来启动你的app了。

### 4、通过 *.plist 文件使用配置和schemes

在这种做法中，我们将重复之前那个创建新scheme的步骤。在做完这个步骤后，不像之前一样添加一个全局标记，我们将把必要的值添加进我们的*.plist中。同样的，我们将添加一个 serverBaseURL 字符串变量到每个 *.plist 文件中，并把URLs作为值进行设置。现在每个*.plist文件都包含一个URL，我们可以通过代码来调用它。我认为，为我们的Bundle创建一个extension是一个好主意，像下面的代码一样：

```

extension Bundle {
    var apiBaseURL: String {
	return object(forInfoDictionaryKey: "serverBaseURL") as? String ?? ""
    }
}

//And call it from the code like this:
let baseURL = Bundle.main.apiBaseURL

```

个人认为，我更喜欢这种方式多一点，因为你不需要在你的代码中检查你的配置。你仅仅需要选择你的Bundle，然后它就会返回他自己当前的配置。

当使用多个targets时需要注意一些情况

- 记住在一个 *.plist 文件中你的数据存储能被读取是很可能不安全的。解决方法就是，把你的敏感的值放到代码中，只把不敏感的放到 *.plist 文件中。
- 当添加一个新的文件时，不要忘记添加到所有的target中，以保证你的代码在所有配置环境中都是一样的。
- 如果你使用持续集成服务，例如 Travis CI 或 Jenkins，不要忘记正确配置它们。

Conclusion

### 结论

一开始用一种可读性强且灵活的方法，去区分不同app环境，是很有必要的。甚至是那种我们避免使用的，最简单的处理不同配置的方法，都能很好的提高我们的代码质量。

我们从最简单的开始，讨论了几种不同的做法。但是，还有很多其他不同环境配置方法我们没有涉及到，我很希望在下面评论中看到其他的做法。

感谢大家的阅读 :)

> 本文[Github地址](https://github.com/britzlieg/translate_post/blob/master/2017-09/%E8%BD%BB%E6%9D%BE%E7%AE%A1%E7%90%86Swift%E9%A1%B9%E7%9B%AE%E4%B8%AD%E7%9A%84%E5%90%84%E7%A7%8D%E7%8E%AF%E5%A2%83.md)

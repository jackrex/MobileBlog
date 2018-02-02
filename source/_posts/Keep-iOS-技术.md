---
title: Keep iOS 技术
date: 2017-11-06 15:46:25
tags: iOS
---

# 架构模型
Keep iOS 基本架构模型设计如下所示
![](/images/keep-arch.png)
上图模型从下到上依次分为 Core, Service, Business 三层：

- Core 层主要包含网络请求、基础通用模块、数据存储以及第三方服务模块，这些模块接口直接提供给上层服务。
- Service 层包含了各种包装好的服务给业务层调用，比如本地日志系统、统计埋点以及第三方服务的封装等等。这一层承上启下，连接了 Core 层和 Business 层。
- Business (SU/TC/RT) 的解耦以 KEPMedium 模块作为共同依赖组件，其核心实现是采用 Runtime 利用反射 Class 动态调用达到解耦的效果。 需要特别指出的是，由于项目庞大，在 Business 中首先需要将具体业务细化为子业务模块，比如图示的 Timeline 模块和 Live 模块；然后子业务内部再划分 MVC/MVVM ，但业务相关的工具类需要单独抽出；另外， Core 层和 Service 层的部分模块被要求放到私有仓库中，用 Carthage 作为Framework 为日后 Keep 其他 App 提供基础依赖。

# 技术实现
最开始 Keep 项目的 UI 编写主要采用 Storyboard 和 XIB，因为当时正处于苹果积极推崇 StoryBoard 的阶段。直至 2016 年5 月份，由于 Xcode 升级导致了 Storyboard 兼容性上的一些问题，外加项目工程大，导致 Storyboard 打开更耗费系统资源、编译时间更长，因此之后选取了 Masonry 作为 AutoLayout 主要开发库，同时资源图片采用 PDF 矢量图。

### IDE 的选择
Tim Cook 来 Keep 参观时曾询问到 Xcode 能否满足当前的开发需求。作为 Apple 推出的官方 IDE，Xcode 对苹果特有的 Storyboard、XIB、plist 等文件类型均支持地比较全面，但在代码提示、自动补全等功能方面，同专攻 IDE 的 JetBrains 等公司推出的类似工具相比还略显逊色，例如 AppCode。iOS 开发团队也曾尝试过 AppCode，感受是：纯写代码较顺畅，静态检查也超赞，内置的 Git 操作、Refactor 等功能都非常好用；但 AppCode 对 Swift 和 Objective-C 混编、XIB 等支持的并不佳。目前开发团队中编译器的选择主要依据个人喜好，在纯写代码时更加偏向于 AppCode。

### 对 Swift 的态度
在 Swift 发布初期，Keep 项目即引入了 Swift 代码，但仅有一部分边缘模块采用 Swift 编写。主要考虑到 Swift 语言不够成熟，有可能存在一些隐形的坑；而且当时人手并不充裕，担心用一门不熟悉的语言而降低开发效率。事实证明当时的决定是正确的，随后 Swift 2.0 和 Swift 3.0 的两次大改动，给使用老版本 Swift 编写的应用带来了不小的升级工作量。但随着 Swift 3.0 的发布，Swift API 逐渐稳定，之后基本不会出现较大改动。所以目前 Keep 项目中新的业务模块几乎都用 Swift 编写，因为与 Objective-C 相比，Swift 语言更简洁，语法更 Modern。另外，团队中偶尔一些脚本、爬虫等项目也直接用 Swift 编写。

### 编译的优化
由于需求众多， Keep 项目逐渐庞大，导致编译时间加长，需要对其进行优化，iOS 团队主要从以下几个方面入手：

尽可能减少宏的定义，使用静态常量代替。
删除无用的头文件引用，采取 @class 或 @import module 方式代替。
减少 Storyboard、XIB 的使用，因为 Storyboard 需要编译 Copy 等过程。
第三方库尽量使用静态库 Framework。
优化过后，整个工程编译时间从 15 分钟缩短至 7 分钟左右，而且随着组件化的细分，相信这个时间还会继续缩短。另外，业务模块相互独立后，Business 中各个业务也将单独拆成独立 Project 进行编译，使业务人员开发时只需编译自己的模块，开发效率更高。

### 自动化流程
iOS 团队采用 Jenkins + Xcodebuild + Fastlane 的方式，编写了打包上传 App Store、发布内测、灰度的自动化流程工具，并在公司的 Mac Pro 上部署了这一工具。当然除了打包之外，还部署了其他一些自动化服务，比如前台接待系统，办公网络抓包工具等等。

『前台接待系统』是 iOS 团队成员在业余时间，用 Swift + Spring Boot + Mysql 开发的一个自动接待工具。快递小哥来了之后只需进行扫码就可用短信通知快递收件人，基本替代了收发快递的工作，前台人员工作量明显减少了。像这种常用工具的搭建在 Keep 技术团队中非常常见，也是探索新技术的一种方式。

# 展望规划
在编程语言选择方面， iOS 开发团队会持续使用 Swift 来开展新的业务和重写旧的业务。另外近期也打算使用 RxSwift + Alamofire 重新编写网络层，逐步替代原有的 AFNetworking。

另外组件化的细粒度拆分还需进一步进行，实现各个业务组件可单独编译运行；Apple Watch 等外设提供的训练和跑步功能进行丰富和优化。

客户端性能优化也是今年的工作重点。iOS 开发团队会继续完善内部 KEPProtectKit 模块，降低崩溃率，监控和优化 FPS，减少网络延迟，提高 Web 页面加载速度，缩短冷启动时间，并加大单元测试的覆盖量，结合 Jenkins 更好的完成自动化测试和分析。

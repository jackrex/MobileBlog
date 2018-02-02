---
title: 网络层设计漫谈
date: 2017-12-22 15:31:18
tags:
- iOS
- Network
---

# 为什么要改写网络层呢？
在各大公司的主App中网络层一定是相当重要的一个环节. 之前Keep 网络层设计比较臃肿，考虑的可扩展性太强，导致使用起来比较繁琐。前期网络设计中间的Protocol 层显示的十分冗余，无形添加了很多工作量，而且不支持很多功能，比如缓存、批量请求、网络取消等，所以需要一套更加编写方便统一的网络层。


# 之前的设计
![](/images/network1.png)

### 具体用法
- 声明具体的Protocol类
- 在相应的 Handler 里面添加方法调用 DefaultOperation 来执行Protocol
- 在具体的逻辑里面通过 HandlerFactory 来获取数据


其中Protocol 一层定义不清楚，每次使用创建比较繁琐。

# 设计方案
### 设计需要考虑的问题
- 解决调用繁琐的问题。
- Header 文件字段的填充。
- 每个业务版本号的path 问题。
- 缓存的问题。缓存地址和时间。
- 考虑到目前网络层迁移的问题。
- 网络错误的统一提示和解决。
- 取消网络请求。
- 回掉方法。
- Json 校验
- 链式、批量调用

参考YTNNetwork，更具业务量大小，Keep 采用分散化设计


# Architecture
![](/images/KEPRequest.png)


# Code Sample
继承的方式调用

```
KEPPostRequest *postRequest = [[KEPPostRequest alloc] initWithParams:@"1123581321"];
    [postRequest startWithBlock:^(__kindof KEPRequest * _Nonnull request) {
        NSLog(@"success %@", request);

    } failure:^(__kindof KEPRequest * _Nonnull request) {
        NSLog(@"failure %@", request);

    }];
```

直接调用

```
KEPRequest *request = [[KEPRequest alloc] init];
    request.requestMethod = KEPRequestMethodPOST;
    request.requestUrl = @"/account/v2/login";
    request.requestArgument =  @{
                                 @"password": @"xxxxx",
                                 @"countryCode": @"86",
                                 @"mobile":@"xxxxxx",
                                 @"countryName":@"CHN"
                                 };

    [request startWithBlock:^(__kindof KEPRequest * _Nonnull request) {
        NSLog(@"%@", request);

    } failure:^(__kindof KEPRequest * _Nonnull request) {
        NSLog(@"%@", request);

    }];
```


# Code Repo
[KEPNetwork](https://github.com/jackrex/KEPNetwork)

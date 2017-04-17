---
title: 'iOS10 Today Extension 开发小结 '
date: 2016-10-11 17:30:20
tags: 
- Swift 
- Alamofire 
- Today-Extension
categories:
- 开发
- iOS
---

## 背景

9.15号中秋节那天发布了[师大助手App](https://itunes.apple.com/us/app/shi-da-zhu-shou-hui-shi-fan/id1150983683?l=zh&ls=1&mt=8)后，就在想还加点什么功能。后来想到了可以在 Today Widgets 里面加一个今日课程，于是就开始动手做了。先给大家看一下最终效果：
![最终效果](http://7xqj7o.com1.z0.glb.clouddn.com/blog/Blog_Widgets_1)
一开始开发的时候，还是 Xcode7 和 Swift2 ，等到准备上架审核的时候，Xcode8 正式版发布了，然后就做了 Swift3 迁移和 iOS10 适配。

## 步骤

### Part1 基本搭建 

1. 新建 iOS Project ，这就不用说了哈
2. 新建一个 Target ，选择 Today-Extension ，Target名字就设置为 TodayWidget
3. 在 MainInterface.storyboard 画 UI ，然后运行就可以看到效果了，由于我是显示课程，就在里面添加了一个 UITableView ，并且增加了两个 Prototype Cell 用于自定义Cell


### Part2 依赖项目引入

1. 在 Podfile 项目中新建一个target，并加入需要引入的项目，代码如下：
    ``` swift
    target 'TodayWidget' do
      pod 'YYModel'
    end
    ```
2. 在 TodayWidget Target 下面新建一个桥接头文件，这里为省去设置的头文件的方法，可以直接新建oc文件，xcode会问你要不要自动添加一个桥接头文件，建好之后删除oc文件即可
3. 接下来在测试过程中，我们就可以愉快的通过 Cocoapods 使用第三方库了，但是提交代码到AppStore就会报如下的错误了：
    ![上传报错](http://7xqj7o.com1.z0.glb.clouddn.com/blog/Blog_Widgets_2)
4. 解决targets不能使用Cocoapods的方法是：在TodayWidget -> Build Phases -> New Run Script Phase ，再添加如下代码
    ``` swift
    cd "${CONFIGURATION_BUILD_DIR}/${UNLOCALIZED_RESOURCES_FOLDER_PATH}/"
    if [[ -d "Frameworks" ]]; then
    rm -fr Frameworks
    fi
    ```
5. 这样就可以顺利提交到AppStore了

### Part3 TodayExtension Target 和 main Target 之间数据共享

在讲如何具体实现数据共享之前，需要先打开两个Target -> Capabilities -> App Groups，只有在同一个Groups内的target才能共同读写数据。
![设置Group](http://7xqj7o.com1.z0.glb.clouddn.com/blog/Blog_Widgets_3.png)
有两种解决方式：UserDefaults 和 FileManager 

#### Solution1  UserDefaults方式

UserDefaults Apple 官方文档：
> The NSUserDefaults class provides a programmatic interface for interacting with the defaults system. The defaults system allows an application to customize its behavior to match a user’s preferences. For example, you can allow users to determine what units of measurement your application displays or how often documents are automatically saved. Applications record such preferences by assigning values to a set of parameters in a user’s defaults database. The parameters are referred to as defaults since they’re commonly used to determine an application’s default state at startup or the way it acts by default.

在本App中，我需要将当前是第几教学周在 targets 间传递，数据量很小，于是采用了 UserDefaults 方法，代码如下：
    ``` swift
    // 写入
    UserDefaults.init(suiteName: "group.com.huangshuai.xxx")?.set(week, forKey: "currentWeek")
    // 读取
    let currentWeek = UserDefaults.init(suiteName: "group.com.huangshuai.xxx")?.object(forKey: "currentWeek") as? Int ?? 1
    ```
注：这里是 group 间的 UserDefaults 数据共享，而不是平常的单一target下面的 UserDefaults 使用：
    ``` swift 
    // 写入
    UserDefaults.standard.set(false, forKey: "isFirstLaunch")
    // 读取
    let isFirstLaunch = UserDefaults.standard.object(forKey: "isFirstLaunch") as? Bool ?? true
    ```

#### Solution2  FileManager方式

FileManager Apple 官方文档：
> An FileManager object lets you examine the contents of the file system and make changes to it. The FileManager class provides convenient access to a shared file manager object that is suitable for most types of file-related manipulations. A file manager object is typically your primary mode of interaction with the file system. You use it to locate, create, copy, and move files and directories. You also use it to get information about a file or directory or change some of its attributes.

在本App中，我需要将所有的课表数据在 targets 间传递，数据量较大，于是采用了 FileManager 方法，代码如下：

``` swift
let name = "course.plist"
let groupPath = FileManager.default.containerURL(forSecurityApplicationGroupIdentifier: "group.com.huangshuai.xxx")
let groupPathString = groupPath!.absoluteString.replacingOccurrences(of:"file:///private", with: "").replacingOccurrences(of:"file:///", with: "")
destPath = (groupPathString as NSString).appendingPathComponent(name).characters.split{$0 == "."}.map(String.init).first!
// 写入
let dic = [:] as NSDictionary
dic.write(toFile: destPath, atomically: true)
// 读取
let dict = NSDictionary(contentsOfFile: destPath) 
```
    
### Part4 适配 iOS10 Widgets 新特性

#### 处理 Widgets 展开/折叠的情况
在 iOS10 之前，Widgets 的高度可以自定义设置，但是在 iOS10 中，Widgets 的 activeDisplayMode 有两种状态：.expanded 和 .compact。 需要根据当前用户的选择来处理 Widgets 的高度，代码如下：

``` swift 
@available(iOSApplicationExtension 10.0, *)
func widgetActiveDisplayModeDidChange(_ activeDisplayMode: NCWidgetDisplayMode, withMaximumSize maxSize: CGSize) {
    if activeDisplayMode == .compact {
        self.preferredContentSize = maxSize
    } else {
        self.preferredContentSize = self.tableView.contentSize
    }
}
```

这里有个Bug：就是在 .compact 下无法设置尺寸，不管怎么设置都是 maxsize。还有待解决。

#### 处理 Widgets 在 iOS9 和 iOS10 下的显示 Bug
在 iOS10 中，Widgets 的背景是白色的，于是字体颜色设置为黑色的，但是在 iOS9 中，背景是黑色的，App 在iOS9中运行就会看不见字。感觉这是iOS10适配的Bug，为了解决这个问题，只能通过判断系统版本来设置字体颜色了，代码如下：

``` swift 
if #available(iOSApplicationExtension 10.0, *) {
    mainTextColor = UIColor.black
} else {
    mainTextColor = UIColor.white
}
```

#### Bug:
在 iOS10 下，Widgets 的 displayBundleName 设置也不起作用了，不管怎么设置只跟 mainTarget 的displayBundleName 一致，还未找到解决方法。

## 番外篇

女票说她们学校课程很多，每天看课表不方便，在国庆节陪她过生日的时候，现场撸了一个App只供她使用，她每次不用解锁直接下拉Widgets就可以看到今日课程，她开心的不得了还发了朋友圈炫耀一番。哈哈，女票是第一生产力☺️

说到这，本文也就结束了，以上提到的Bug，我会在解决之后再来更新代码。


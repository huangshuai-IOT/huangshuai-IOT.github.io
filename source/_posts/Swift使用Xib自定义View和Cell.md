---
title: Swift 使用 Xib 自定义 View 和 Cell
date: 2016-08-09 17:06:52
tags: 
- Swift 
- Xib
- 自定义View
categories:
- 开发
- iOS
---

# 背景

原来一直使用的是Storyboard自定义`UITableViewCell`的，暑期来公司实习，发现公司用的是Xib自定义View，学长说人多用SB不好协同工作。除了这个原因，我之前还发现Storyboard的复用性不好，自定义的Cell不能在多个Storyboard文件中复用😂。

下面介绍利用Xib自定义View和Cell。

# 自定义Cell

新建Cell文件时，勾选生成Xib文件。然后在Xib上使用控件和约束，和Storyboard一样。

在使用自定义的Cell时，要记得注册可复用的Cell。例如:`collectionView.registerReusableCell(FMPersonHelpReportVCCollectionViewCell)`

剩下的使用过程和用Storyboard自定义的Prototype Cell是一致的。

# 自定义View

新建View的时候，是无法勾选生成Xib文件的，所以需要我们自己创建同名的UIView和Xib文件。并且将 Xib文件的 File's Owner -> Custom Class -> Class 属性设为刚才新建的类。

由于自定义View不能直接从View初始化，所以写了一个基类。

``` swift
class BaseXibView: UIView {
    
    var view:UIView?
    
    //初始化方法。
    func initFromXib(){
        //获取Xib文件名字
        let xibName = NSStringFromClass(self.classForCoder)
        let xibClassName = xibName.characters.split{$0 == "."}.map(String.init).last
        //使用Xib初始化一个View
        let view = NSBundle.mainBundle().loadNibNamed(xibClassName, owner: self, options: nil).first as! UIView
        view.frame = self.bounds
        view.translatesAutoresizingMaskIntoConstraints = true
        view.autoresizingMask = [.FlexibleWidth,.FlexibleHeight]
        self.addSubview(view)
        self.view = view
    }
    
    override init(frame: CGRect) {
        super.init(frame: frame)
        initFromXib()
    }
    
    required init(coder aDecoder: NSCoder) {
        super.init(coder:aDecoder)!
        initFromXib()
    }
}

```


这是我写的一个自定义View，主要用来进行新手引导的，覆盖在原视图上，点击view，慢慢改变透明度。
``` swift
class FMHomeNewUserGuideView: BaseXibView {
    
    @IBOutlet var imageView: UIImageView!
    @IBOutlet var label: UILabel!
    
    var isMainView = true
    
    override func initFromXib() {
        super.initFromXib()
    }
    
    override func touchesBegan(touches: Set<UITouch>, withEvent event: UIEvent?) {
        UIView.animateWithDuration(0.25, animations: { 
            self.alpha = 0
            }) { (isComplete) in
                self.removeSubviews()
        }
    }
    
    func setUI() {
        
    }
} 
```

用法：
``` swift 
let homeNewUserGuideView = FMHomeNewUserGuideView()
```


# 参考资料





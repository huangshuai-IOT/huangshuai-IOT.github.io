---
title: iOS:一个UITextView的小Bug
date: 2016-07-28 14:55:25
tags:
- iOS
- UITextView
categories:
- 开发
- iOS
---

# 起因

今天上午测试提了一个很神奇的Bug，在输入框输入某些iOS自带的表情，删除后会有残留物。例如：💺💺💺看到这个Bug，我的内心是觉得不可能啊。然后亲自试了一下，发现真会这样。至此，我发自内心佩服了我司的测试，这种问题都能发现。

我想，这仅仅就是一个UITextView，应该不会我写的是个例，于是，我找了微信试了一下,这是微信的个人签名框，在输入完表情后，再删除，可以看到，后面留了一串残留物。
![微信上Bug复现](/uploads/blog/1-1.jpg)
我本来想直接关了这个Bug，毕竟微信这种大厂的App都能允许这个Bug。后来一想，也许是他们的测试没有我们的牛逼。

# 解决方法

微信的这个Bug复现，在保存后再进去，就会发现残留物没有了。所以解决思路是，在UITextView一边输入，一边给它赋值。于是实现了`UITextViewDelegate`的`textViewDidChange`方法。

``` swift
extension FMPersonEditSignVC : UITextViewDelegate {
    
    func textViewDidChange(textView:UITextView) {
        self.textView.text = textView.text
    } 
}
```

# 番外篇

说到`UITextViewDelegate`，我又想起来之前提的一个Bug。我们的baseVC中都加了点击空白页隐藏键盘的功能，但是在某些小屏幕机型中，比如iPhone5上，如果UITextView的高度设置的比较高，用户就点击不到空白页（我当时给测试演示，用鼠标点击的，我说这很好点啊，哈哈哈哈），进而就隐藏不了键盘。

这里如果去重新画UI设置约束，我感觉劳动量太大了，于是就想着用户在输入完回车键可以隐藏键盘。另外，iOS的键盘的回车键是可以自定义设置的，我将Return Key设置显示为“Done”，更便于用户理解。

检测用户输入回车的代码如下：

``` swift 
extension FMPersonEditSignVC : UITextViewDelegate {
    
    func textView(textView: UITextView, shouldChangeTextInRange range: NSRange, replacementText text: String) -> Bool {
        if text == "\n" {
            textView.resignFirstResponder()
            return false
        }
        return true
    }
}

```


---
title: ios-打开microsoft文档App
date: 2018-05-18 14:31:44
tags:
  - swift
---

因为从广州来到了北京，换了工作，所以现在公司要求做的东西跟之前不太一样。做的一款办公软件，这里面就包括对微软doc文档，ppt文档，xls文档。Android端的我已经做了一个版本，然后是非常简单的打开，没有编辑功能。现在我要做的就是IOS版本。所有的东西都是自己从0开始的，记录下来，方便自己，也给后来者一些提示。
<!--more-->
# Office文件的IOS-UTI支持
| 文件格式       | UTI type |
| -------------    |:-------------:|
| doc    | com.microsoft.word.doc    | 
| docx    | org.openxmlformats.wordprocessingml.document   | 
| ppt    |  com.microsoft.powerpoint.ppt    | 
| pptx | org.openxmlformats.presentationml.presentation     | 
| xls | com.microsoft.excel.xls   | 
| xlsx | org.openxmlformats.spreadsheetml.sheet |
| pdf | com.adobe.pdf |

如果需要适配以上的文件类型，可以直接将下面的代码复制到info.plist文件中的dict标签中
```
<key>CFBundleDocumentTypes</key>
    <array>
        <dict>
            <key>CFBundleTypeName</key>
            <string>OFFICE Document</string>
            <key>LSHandlerRank</key>
            <string>Owner</string>
            <key>LSItemContentTypes</key>
            <array>
                <string>com.microsoft.word.doc</string>
                <string>com.microsoft.powerpoint.ppt</string>
                <string>com.microsoft.excel.xls</string>
                <string>com.adobe.pdf</string>
                <string>org.openxmlformats.wordprocessingml.document</string>
                <string>org.openxmlformats.presentationml.presentation</string>
                <string>org.openxmlformats.spreadsheetml.sheet</string>
                <string>org.oasis-open.opendocument.text</string>
            </array>
        </dict>
    </array>
```
[参考网站](https://developer.apple.com/library/content/qa/qa1587/_index.html)
# 打开文件
其他的应用(如QQ，微信等)分享过来的文件，在自己的应用中打开，这时候有一个跳转的过程，那么当跳转到第二个页面的时候，我们需要让他可以以一种打开新文件的方式展示
具体实现代码
在AppDelegate.swift文件中，重写application方法，这里注意是使用的是
```
func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool 
```
全局代码
```
func application(_ app: UIApplication, open url: URL, options: [UIApplicationOpenURLOptionsKey : Any] = [:]) -> Bool {
        print("文件路径是 \(url)")
        let vc: DocumentViewController = DocumentViewController(filePath: url)
        //跳转到打开文件的页面
        UIViewController.currentViewController()?.navigationController?.pushViewController(vc, animated: true)
        
        return true
    }
```
扩展方法
```
//获取当期页面的presentview
extension UIViewController {
    class func currentViewController(base: UIViewController? = UIApplication.shared.keyWindow?.rootViewController) -> UIViewController? {
        if let nav = base as? UINavigationController {
            return currentViewController(base: nav.visibleViewController)
        }
        if let tab = base as? UITabBarController {
            return currentViewController(base: tab.selectedViewController)
        }
        if let presented = base?.presentedViewController {
            return currentViewController(base: presented)
        }
        return base
    }
}
```

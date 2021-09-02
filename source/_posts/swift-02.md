---
title: 第一个Swift项目记录
date: 2018-05-12 16:13:15
tags:
  - swift
---

到今天学习Swift已经有一个星期了，一直再做一些看视频，看书，看文档的操作，今天趁着这个机会，自己写一个小项目，然后把之前学到的知识做一个总结。那么既然是第一个小项目，那就有可能失败，但是我会坚持下来
<!--more-->
多余的废话，我就不多说了，直接开始

# 项目介绍
这是之前我做过的一个Android项目的IOS版本--豆瓣电影。
为什么要去写这个呢？
因为豆瓣电影的API是公开的
嗯，听起来很有道理，但是猫眼电影也可以通过抓包获取到API啊
因为豆瓣API比较简单
好了，就这么愉快的决定了
(瞬间，戏精上线)

# 使用的工具和环境

xcode: Version 9.3 (9E145)
Swift:4.0
MacBook pro: 10.13.4

# 具体操作
所有的步骤，我都尽量以截图和粘贴代码的方式去完成，这样方便以后查看(新人学IOS真的感觉好痛苦，因为网上很多例子都是Swift1，2，3的，而Swift4改变比较多)
## 1.创建项目
打开Xcode，通过shift+command+N的快捷键创建一个新的项目
![软件面板](/assets/swift_project/sp_01.png)
在下面的页面中选择单一的App项目，也就是我选中额蓝色的对象，点击next
![选择项目](/assets/swift_project/sp_02.png)
在这里需要输入项目的一些基本信息，里面有我自己的，因为目前来说都是一些测试的，所以没有什么可以掩饰的，没有马赛克。填写完成之后，点击next按钮
![项目一些基本配置](/assets/swift_project/sp_03.png)
系统会让你选择需要将项目存储的位置，选择一个位置存放即可
![项目存放路径](/assets/swift_project/sp_04.png)
点击完成，就会显示项目的全局配置，如果是第一次创建，直接默认不用更改就可以了
![项目全局配置](/assets/swift_project/sp_05.png)
到此，所有的创建项目就搞定了
## 2.项目的分析
我们通过查看豆瓣电影官方的App，可以发现里面包含了较多内容，那么下面就分析一下
### 1.引导页面
这里一般都是一个图片展示，这个图片可以随意，不用做太多，如果实在不行，我们可以通过Launch直接创建一个动画就可以
### 2.主要页面
这里主要分为三个部分，分别是热映，找片和我的。这里我们对其中的内容进行更进一步的分析
#### 1.热映模块
该模块中分为两个大模块，分别是正在热映，即将上映。两个模块的内容有区别，但是我们可以先看做是当前只有列表展示。每个列表项中包含了电影图片，电影名称，电影评分，电影导演，主演，购票，看过该电影的人数等信息，那么我们这里是需要自定义tableView cell的。除了两个大模块之外，还有定位和搜索功能，这两个功能，暂时不在我们的考虑范围
![图片](/assets/swift_project/sp_06.jpeg)
#### 2.找片模块
该模块也是分为了两个大模块分别是电影，电视剧。这两个里面的布局相对而言内容较多，以后写到的时候，在具体分析
![图片](/assets/swift_project/sp_07.jpeg)
#### 3.我的模块
包括了自己的一些基本信息
![图片](/assets/swift_project/sp_08.jpeg)
#### 4.电影详情
点击某个电影cell之后跳转到电影详情页面，显示电影的更多信息
![图片](/assets/swift_project/sp_09.jpeg)

额，好像分析的有点草率，不过毕竟第一次吗，以后多来几次好了。。。😏

## 3.项目的分包
为了能够更好的开发，我们需要将项目进行分包操作，我们这里先按照功能分吧
分包之后的结果如下所示
![分包结果](/assets/swift_project/sp_10.png)
- 工具：主要是用来放置一些动画效果，全局属性等内容
- Main：主要用来存放一些全局都会用到的自定义View
- Hotting：正在热映
- Looking：找片
- Profile: 我的
- Detail: 电影详情

## 4.项目全局配置文件
这里，我们需要把数据进行一下修改
1. 修改bundle display name 为豆瓣电影
![修改info.plist文件](/assets/swift_project/sp_11.png)
2. 为项目添加team
3. 修改当前开发状态下的IOS版本
4. 修改设备是手机
5. 要求当前应用只支持竖屏显示
![修改全局配置文件](/assets/swift_project/sp_12.png)

## 5.设置main文件夹内的组件
1. 修改main.storyboard。这个main.storyborad是程序的主页面的内容，也就是说，所有页面的都是从这个地方开始的。我们首先把原来系统自动为我们声明的viewcontroller视图删除掉，因为我们后面需要添加新的内容，所以删除掉原始图，在创建一个Tab Bar Controller视图。
![创建Tab bar Controller](/assets/swift_project/sp_13.png)
2. 拉取三个Navigation Controller对象，并且每个对象对应的是不同的模块。并且修改item的文字和图标
修改文字直接修改就可以了
![修改tabbar的item的值和图标](/assets/swift_project/sp_14.png)
3. 创建了三个Navigation Controller之后，我们需要对这三个Controller进行优化。选中当前的一个NavigationController对象，选择Editor中的reFactor to StoryBoard，这样就可以把这个对象分理处一个新的storyboard
![提取模块storyboard](/assets/swift_project/sp_15.png)
每一个子模块的详情如下所示：
![正在热映模块](/assets/swift_project/sp_16.png)
并且为每一个模块添加一个swift文件
![正在热映模块](/assets/swift_project/sp_17.png)
4. 为各个模块创建各自对应的swift文件，并且每一个对象都要继承UITableViewController
之后的效果如图所示
![创建](/assets/swift_project/sp_18.png)
代码如下：
正在热映：
```
import UIKit

class HottingViewController: UITableViewController {

    //懒加载创建自定义导航栏左侧定位按钮
    lazy var locationBtn:HottingLeftLocationButton = HottingLeftLocationButton()
    //懒加载声明一个不能使用的搜索框
    lazy var searchBar:EnableSearchBar = EnableSearchBar()
    
    override func viewDidLoad() {
        super.viewDidLoad()

       setupNavigationBar()
    }

}
extension HottingViewController{
    private func setupNavigationBar(){
        //设置导航栏左侧图标样式
        //设置标题
        locationBtn.leftBtn.setTitle("定位", for: .normal)
        //设置定位按钮点击之后的操作
        locationBtn.leftBtn.addTarget(self, action: #selector(HottingViewController.gotoLocationView), for: .touchUpInside)
        navigationItem.leftBarButtonItem = locationBtn
        //设置title,因为title本身只有一个文字，为了显示更多的内容，我们需要自己写一个布局
        navigationItem.titleView = searchBar
        //给搜索框添加点击事件
//        searchBar.searchBar.addOnClickListener(target: self, action: #selector(HottingViewController.gotoSearchView))
        let singleTapGesture = UITapGestureRecognizer(target: self, action: #selector(gotoSearchView))
        searchBar.addGestureRecognizer(singleTapGesture)
        searchBar.isUserInteractionEnabled = true
        
    }
}

extension HottingViewController{
    //跳转到定位页面
    @objc private func gotoLocationView(){
        print("点击了定位按钮")
    }
    //调转到搜索页面
    @objc private func gotoSearchView(){
        print("点击了搜索页面")
    }

}
```
找片模块：
```
import UIKit

class LookingViewController: UITableViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Uncomment the following line to preserve selection between presentations
        // self.clearsSelectionOnViewWillAppear = false

        // Uncomment the following line to display an Edit button in the navigation bar for this view controller.
        // self.navigationItem.rightBarButtonItem = self.editButtonItem
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

    // MARK: - Table view data source

    override func numberOfSections(in tableView: UITableView) -> Int {
        // #warning Incomplete implementation, return the number of sections
        return 0
    }

    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        // #warning Incomplete implementation, return the number of rows
        return 0
    }

    /*
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "reuseIdentifier", for: indexPath)

        // Configure the cell...

        return cell
    }
    */

    /*
    // Override to support conditional editing of the table view.
    override func tableView(_ tableView: UITableView, canEditRowAt indexPath: IndexPath) -> Bool {
        // Return false if you do not want the specified item to be editable.
        return true
    }
    */

    /*
    // Override to support editing the table view.
    override func tableView(_ tableView: UITableView, commit editingStyle: UITableViewCellEditingStyle, forRowAt indexPath: IndexPath) {
        if editingStyle == .delete {
            // Delete the row from the data source
            tableView.deleteRows(at: [indexPath], with: .fade)
        } else if editingStyle == .insert {
            // Create a new instance of the appropriate class, insert it into the array, and add a new row to the table view
        }    
    }
    */

    /*
    // Override to support rearranging the table view.
    override func tableView(_ tableView: UITableView, moveRowAt fromIndexPath: IndexPath, to: IndexPath) {

    }
    */

    /*
    // Override to support conditional rearranging of the table view.
    override func tableView(_ tableView: UITableView, canMoveRowAt indexPath: IndexPath) -> Bool {
        // Return false if you do not want the item to be re-orderable.
        return true
    }
    */

    /*
    // MARK: - Navigation

    // In a storyboard-based application, you will often want to do a little preparation before navigation
    override func prepare(for segue: UIStoryboardSegue, sender: Any?) {
        // Get the new view controller using segue.destinationViewController.
        // Pass the selected object to the new view controller.
    }
    */

}
```
我的模块：
```

import UIKit

class ProfileViewController: UITableViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Uncomment the following line to preserve selection between presentations
        // self.clearsSelectionOnViewWillAppear = false

        // Uncomment the following line to display an Edit button in the navigation bar for this view controller.
        // self.navigationItem.rightBarButtonItem = self.editButtonItem
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

}
```
mainController模块：
```
import UIKit

class MainViewController: UITabBarController {

    private lazy var imagesName = [
    "hotting","looking","profile"
    ]
    //页面已经加载完成
    override func viewDidLoad() {
        super.viewDidLoad()
        
        
    }
    //页面即将显示
    override func viewWillAppear(_ animated: Bool) {
        super.viewWillAppear(animated)
        for i in 0..<tabBar.items!.count{
            let item = tabBar.items![i]
            item.selectedImage = UIImage(named: imagesName[i]+"selected")
        }
    }
    
}
extension MainViewController{
    private func setupTabbarItems(){
       
    }
}
```
这里我们需要将AppDelegate.swift文件进行修改
```
import UIKit

@UIApplicationMain
class AppDelegate: UIResponder, UIApplicationDelegate {

    var window: UIWindow?


    func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplicationLaunchOptionsKey: Any]?) -> Bool {
        UITabBar.appearance().tintColor = UIColor.red
        return true
    }

}
```
## 5.生成自定义组件
这里因为我们需要对导航栏的内容进行显示，所以我们需要对导航栏的组件进行修改
创建了三个组件，为不同的内容添加响应的效果
![自定义组件](/assets/swift_project/sp_19.png)
- HottingLeftLocationButton.swift
```
import UIKit

class HottingLeftLocationButton: UIBarButtonItem {

    //懒加载设置按钮样式
    var leftBtn:HottingLeftButton = HottingLeftButton()
    
    override init() {
        super.init()
        //将customView设置成btn，这样btn的样式就会给到UIBarButtongItem对象
        customView = leftBtn
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    
}
```
- HottingLeftButton
```
import UIKit

class HottingLeftButton: UIButton {

    
     //初始化设置
     override init(frame: CGRect) {
        super.init(frame: frame)
        //给左侧定位添加向下的图片
        setImage(UIImage(named: "navigation_down"), for: .normal)
        setTitleColor(UIColor.black, for: .normal)
        //设置字体大小
        titleLabel?.font = UIFont.systemFont(ofSize: 14)
        sizeToFit()
        //设置文字紧贴着左侧显示，图片在文字的3个像素的位置
        imageView!.frame.origin.x = titleLabel!.frame.origin.x + 10
        titleLabel!.frame.origin.x = 0
     }
     
     required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
     }
     
     //设置当前图片和文字的位置
     override func layoutSubviews() {
        super.layoutSubviews()
     }
    

}

```
- EnableSearchBar
```
import UIKit

class EnableSearchBar: UIView {

    lazy var searchBar:UISearchBar = UISearchBar()
    lazy var coverView:UIView = UIView()
    
  //不可点击的searchbar
    override init(frame: CGRect) {
        super.init(frame: frame)
        setupCoverView()
    }
    
    required init?(coder aDecoder: NSCoder) {
        fatalError("init(coder:) has not been implemented")
    }
    //设置UI布局
    override func layoutSubviews() {
        
    }
    
}
extension EnableSearchBar{
    private func setupCoverView(){
        //设置搜索框和蒙板的大小和位置
        searchBar.frame = CGRect.init(x: -120, y: -20, width: 300, height: 40)
        coverView.frame = CGRect.init(x: -120, y: -20, width: 300, height: 40)
        //设置searchbar的placeholder
        searchBar.placeholder = "电影/电视剧/影人"
        //设置蒙板的背景
        coverView.backgroundColor = UIColor(white: 0.9, alpha: 0.0)
        //将两个组件添加到页面中
        self.addSubview(searchBar)
        self.addSubview(coverView)
    }
}
```
最终实现的效果是：
![自定义组件](/assets/swift_project/sp_20.png)

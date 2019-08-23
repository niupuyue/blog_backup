---
title: ios项目开发框架总结(Swift)
date: 2018-05-14 19:33:13
tags:
  - ios
---
框架
<!--more-->
# SnapKit

SnapKit，一个经典的Swift版的第三方库，专门用于项目的自动布局，目前在github上的stars就高达9340颗星，这是一个不小的数字，亦足以证明它存在的非凡意义和作用。作者认为，在iOS开发（swift）中，它是用于项目最优秀的自动布局的必选库之一,它的作者仍然是写Objective-C的第三方库Masonry的大牛 - @Robert Payne.
关于怎么安装就不说了，直接开搞

1. 实现一个宽高为100，居于当前视图的中心的视图布局
```
import UIKit
import SnapKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
     
        let testView = UIView()
        testView.backgroundColor = UIColor.cyan
        view.addSubview(testView)
        testView.snp.makeConstraints { (make) in
            make.width.equalTo(100)         // 宽为100
            make.height.equalTo(100)        // 高为100
            make.center.equalTo(view)       // 位于当前视图的中心
        }
        
    }
}
```
或者可以使用简写方式
```
 import UIKit
import SnapKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
     
        let testView = UIView()
        testView.backgroundColor = UIColor.cyan
        view.addSubview(testView)
        testView.snp.makeConstraints { (make) in
            make.width.height.equalTo(100)    // 链式语法直接定义宽高
            make.center.equalToSuperview()    // 直接在父视图居中
        }
        
    }
}
```
运行截图
![居中显示](/assets/ios_framework/if_01.png)

2. View2位于View1内， view2位于View1的中心， 并且距离View的边距的距离都为20
```
import UIKit
import SnapKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
     
         // 黑色视图作为父视图
         let view1 = UIView()
         view1.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
         view1.center = view.center
         view1.backgroundColor = UIColor.black
         view.addSubview(view1)
         
         // 测试视图
         let view2 = UIView()
         view2.backgroundColor = UIColor.magenta
         view1.addSubview(view2)
         view2.snp.makeConstraints { (make) in
              make.top.equalToSuperview().offset(20)      // 当前视图的顶部距离父视图的顶部：20（父视图顶部+20）
              make.left.equalToSuperview().offset(20)     // 当前视图的左边距离父视图的左边：20（父视图左边+20）
              make.bottom.equalToSuperview().offset(-20)  // 当前视图的底部距离父视图的底部：-20（父视图底部-20）
              make.right.equalToSuperview().offset(-20)   // 当前视图的右边距离父视图的右边：-20（父视图右边-20）
         }
        
    }
}
```
简写方式：
```
import UIKit
import SnapKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
     
         // 黑色视图作为父视图
         let view1 = UIView()
         view1.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
         view1.center = view.center
         view1.backgroundColor = UIColor.black
         view.addSubview(view1)
         
         // 测试视图
         let view2 = UIView()
         view2.backgroundColor = UIColor.magenta
         view1.addSubview(view2)
         view2.snp.makeConstraints { (make) in
            make.edges.equalToSuperview().inset(UIEdgeInsets(top: 20, left: 20, bottom: 20, right: 20))
         }
        
    }
}
```
运行截图
![offset](/assets/ios_framework/if_02.png)

3. 布局一个视图view1， 让它的水平中心线小于等于另一个视图view2的左边
```
import UIKit
import SnapKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
     
         // 黑色视图作为父视图
         let view1 = UIView()
         view1.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
         view1.center = view.center
         view1.backgroundColor = UIColor.black
         view.addSubview(view1)
        
         // 测试视图
         let view2 = UIView()
         view2.backgroundColor = UIColor.magenta
         view1.addSubview(view2)
         view2.snp.makeConstraints { (make) in
            // 让顶部距离view1的底部为10的距离
            make.top.equalTo(view1.snp.bottom).offset(10)
            // 设置宽、高
            make.width.height.equalTo(100)
            // 水平中心线<=view1的左边
            make.centerX.lessThanOrEqualTo(view1.snp.leading)
         }
        
    }
}
```
运行截图
![lessThanOrEqualTo](/assets/ios_framework/if_03.png)

## 视图属性说明
![属性列表](/assets/ios_framework/if_04.png)
从表中，我们知道，Snapkit的布局属性都是源自于系统的NSLayoutAttribute，那么，NSLayoutAttribute是个什么呢？其实，它在swift中是一个枚举，内部列举了很多布局属性诸如top、left、leading、centerX等，Snapkit的布局属性与它们都存在一一的对应关系。

几个比较难记或者说难操作的方法

1. Snapkit 的 greaterThanOrEqualTo 属性
如果想让视图View2的左边>=父视图View1的左边， 这时我们就可以用到greaterThanOrEqualTo
```
import UIKit
import SnapKit

class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
         // 黑色视图作为父视图
         let view1 = UIView()
         view1.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
         view1.center = view.center
         view1.backgroundColor = UIColor.black
         view.addSubview(view1)
        
         // 测试视图
         let view2 = UIView()
         view2.backgroundColor = UIColor.magenta
         view1.addSubview(view2)
         view2.snp.makeConstraints { (make) in
            // 让顶部距离view1的底部为10的距离
            make.top.equalTo(view1.snp.bottom).offset(10)
            // 设置宽、高
            make.width.height.equalTo(100)
            // 水平中心线<=view1的左边
            make.left.greaterThanOrEqualTo(view1)
            
            // 或者, 和上面一行代码一样的效果
//            make.left.greaterThanOrEqualTo(view1.snp.left)
         }
    }
}
```
运行截图
![greaterThanOrEqualTo](/assets/ios_framework/if_05.png)
其实，greaterThanOrEqualTo这个属性有点多余，比如上面这行代码 make.left.greaterThanOrEqualTo(view1) ， 我们可以换成 make.left.equalToSuperview()或make.left.equalTo(view1.snp.left)， 效果是一样的，也就是说，一般情况下 >= 或 <= 我们都可以直接用equalTo来代替！

2. SnapKit的greaterThanOrEqualTo和lessThanOrEqualTo联合使用
当我们想要让某个视图的width或height大于等于某个特定的值，小于等于某个特定的值的时候，一般而言，Snapkit会以greaterThanOrEqualTo为准，这里举一个width的例子，为了方便，这里指贴出了viewDidLoad中的代码（其他没必要）
```
// 黑色视图作为父视图
 let view1 = UIView()
 view1.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
 view1.center = view.center
 view1.backgroundColor = UIColor.black
 view.addSubview(view1)
    
 // 测试视图
 let view2 = UIView()
 view2.backgroundColor = UIColor.magenta
 view1.addSubview(view2)
 view2.snp.makeConstraints { (make) in
    make.width.lessThanOrEqualTo(300)
    make.width.greaterThanOrEqualTo(200)
    make.height.equalTo(100)
    make.center.equalToSuperview()
 }
```
运行截图
![greaterThanOrEqualTo和lessThanOrEqualTo](/assets/ios_framework/if_06.png)
很明显，最后的宽度是以make.width.greaterThanOrEqualTo(200)为标准的，也可以这样的，在同时使用两者的情况下，greaterThanOrEqualTo的优先级略比lessThanOrEqualTo的优先级高。值得一提的是， 在上面的例子中，如果我们只设置make.width.lessThanOrEqualTo(300)，那么view2是不会显示出来的，因为view2不知道你要表达的是要显示多少，小于等于300，到底是100还是200呢？(这里指针对width和height）所以它不能确定这个约束的值，但是，如果我们单独设置make.width.greaterThanOrEqualTo(200)，那么就和上面的效果一样，因为它会以200为标准布局约束

3. lessThanOrEqualTo的用于上、下、左、右
如果我们想要视图view2的左边 <= view1.left + 10, 那么就可以直接用到lessThanOrEqualTo布局了
```
// 黑色视图作为父视图
 let view1 = UIView()
 view1.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
 view1.center = view.center
 view1.backgroundColor = UIColor.black
 view.addSubview(view1)
    
 // 测试视图
 let view2 = UIView()
 view2.backgroundColor = UIColor.magenta
 view1.addSubview(view2)
 view2.snp.makeConstraints { (make) in
    make.left.lessThanOrEqualTo(20)     // <= 父视图的左边+20
    make.right.equalTo(-40)             // = 父视图的右边-40
    make.height.equalTo(100)
    make.center.equalToSuperview()
 }
```
运行截图
![lessThanOrEqualTo](/assets/ios_framework/if_07.png)

## Snapkit布局的灵活性

- Snapkit布局灵活性很强， 我们看下面的例子, 他们的效果是一样的
```
make.left.equalToSuperview().offset(10)
make.left.equalTo(10)
make.left.equalTo(view1.snp.left).offset(10)
```

- 设置视图的大小（width，height）,他们效果是一样的
```
make.width.height.equalTo(100)
或
make.width.equalTo(100)
make.height.equalTo(100)
或
make.size.equalTo(CGSize(width: 100, height: 100))
```

- 优先级(priority)
我们来看一下Snapkit的优先级设置， 优先级都是附加在约束链的末尾处，看下面的使用方法
```
// 黑色视图作为父视图
let view1 = UIView()
view1.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
view1.center = view.center
view1.backgroundColor = UIColor.black
view.addSubview(view1)
    
// 测试视图
let view2 = UIView()
view2.backgroundColor = UIColor.magenta
view1.addSubview(view2)
view2.snp.makeConstraints { (make) in
    make.width.equalTo(100).priority(666)
    make.width.equalTo(250).priority(999)
    make.height.equalTo(111)
    make.center.equalToSuperview()
}
```
运行截图
![优先级](/assets/ios_framework/if_08.png)
从上面我们可以知道, 我们设置了两个优先级：make.width.equalTo(100).priority(666) 和 make.width.equalTo(250).priority(999)， 那运行结果是一个哪个为准呢？显然是以优先级为 999的为准，因为 priority(999)>priotity(666)， 所以在使用Snapkit的过程中，有时我们可以使用优先级priority来设置我们的约束， 另外，值得一提的是，SnapKit的优先级最大值只能是 1000， 如果优先级的数值超过1000，则运行时就会Crash！这里要尤其注意

- 更新约束（引用约束）
我们可以通过保存某一个约束布局来更新相应的约束，或者保存一组约束布局到一个数组中更新约束， 具体看下面代码
```
// 保存约束（引用约束）
var updateConstraint: Constraint?
    
override func viewDidLoad() {
    super.viewDidLoad()
    
    // 黑色视图作为父视图
    let view1 = UIView()
    view1.frame = CGRect(x: 0, y: 0, width: 300, height: 300)
    view1.center = view.center
    view1.backgroundColor = UIColor.black
    view.addSubview(view1)

    // 测试视图
    let view2 = UIView()
    view2.backgroundColor = UIColor.magenta
    view1.addSubview(view2)
    view2.snp.makeConstraints { (make) in
        make.width.height.equalTo(100)  // 宽高为100
        self.updateConstraint = make.top.left.equalTo(10).constraint   // 距离父视图上、左为10
    }
    
    let updateButton = UIButton(type: .custom)
    updateButton.backgroundColor = UIColor.brown
    updateButton.frame = CGRect(x: 100, y: 80, width: 50, height: 30)
    updateButton.setTitle("更新", for: .normal)
    updateButton.addTarget(self, action: #selector(updateConstraintMethod), for: .touchUpInside)
    view.addSubview(updateButton)
}
    
// 更新约束
func updateConstraintMethod() {
    self.updateConstraint?.update(offset: 50)   // 更新距离父视图上、左为50
}
```
运行截图
![更新约束](/assets/ios_framework/if_09.gif)

- 更新约束(snp.updateConstraints)
说起这个updateConstraints, 我也懵逼过，那么它到底有何作用呢？又怎么用呢？它和一开始就使用的makeConstraints又有什么明确的区别呢？请继续往下看

说明1：如果你这是更新某个约束或某几个约束的常量值，你就可以使用updateConstraints而不是makeConstraints。

说明2：这个也是苹果推荐用来添加或更新约束的方式

说明3：这个方法可以调用多次，会相应setNeedsUpdateConstraints, 在控制器中，可以写在override func updateViewConstraints()方法里面（当然也可以写在你想要什么时候更新的点击事件里面）

```
import UIKit
import SnapKit

class ViewController: UIViewController {

    lazy var blackView = UIView()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        blackView.backgroundColor = UIColor.black
        view.addSubview(blackView)
        blackView.snp.makeConstraints { (make) in
            
            // 四个约束确定位置和大小
            make.width.equalTo(100)
            make.height.equalTo(150)
            make.top.equalTo(10)
            make.centerX.equalToSuperview()
        }
 
    }
    
    override func updateViewConstraints() {
        
        blackView.snp.updateConstraints { (make) in
            // 更新距离父视图顶部的约束（从 10 ---> 300 ）
            make.top.equalTo(300)
        }
        
        // 根据苹果，调用父类应该放在末尾
        super.updateViewConstraints()
    }
}
```
注意: 从上面的代码中我们很明确地知道， blackView通过width、height、top、centerX确定了它本身的大小和位置， 但是， 在 run 出来之后，它的top改变了距离， 从 10 变成了 300，其他三个约束保持不变
运行截图
![更新约束](/assets/ios_framework/if_10.png)
显而易见， 除了top约束， 其他都没有改变！ 也就是说，约束被更新（相当于系统升级一样，是一个道理）

现在，我们通过UIButton的点击事件来证明一下制作约束makeConstraints和updateConstraints具体的区别在哪里
```
lazy var blackView = UIView()
    
override func viewDidLoad() {
    super.viewDidLoad()
    
    blackView.backgroundColor = UIColor.black
    view.addSubview(blackView)
    blackView.snp.makeConstraints { (make) in
        
        make.width.equalTo(100)
        make.height.equalTo(150)
        make.top.equalTo(50)
        make.centerX.equalToSuperview()
    }
    
    let btn = UIButton(type: .custom)
    btn.backgroundColor = UIColor.brown
    btn.frame = CGRect(x: 100, y: 200, width: 60, height: 30)
    btn.addTarget(self, action: #selector(buttonAction), for: .touchUpInside)
    view.addSubview(btn)
 
}
    
// 点击更新/制作约束
func buttonAction() {
    
    blackView.snp.makeConstraints { (make) in
        make.width.height.equalTo(20)
        make.top.equalTo(300)
    }
    
}
```
点击事件发生前(图1）
![点击事件发生前](/assets/ios_framework/if_11.png)
点击事件发生后
![点击事件发生后](/assets/ios_framework/if_12.png)
![点击事件发生后](/assets/ios_framework/if_13.png)
![点击事件发生后](/assets/ios_framework/if_14.png)

> 上面，我们知道，视图 blackView一开始是由四个约束确定位置和大小，在点击事情响应后，我们又给 blackView 制作（记住，是制作，而不是重做，两者有明确的区别）了3个约束，分别是 width、height、top, 那么这样做问题出现在哪里呢？ 第一， 点击事情发生前（图1）， 在点击事件发生后（见图2）， 我们发现，blackView的width、height约束改变了，但是 top却没有改变，还是原来的距离父视图顶部 50 的距离， 原因在于，我们在原来的约束基础上又添加了多余的约束， 也就是说，约束从4个变成了7个（见图3左边constraints）， 这样就产生了约束不明确，进而导致snapkit的警告（见图4）， 这样布局显然是不可取的，在项目中这样做极其危险，甚至可能会导致异常奔溃！！！！

现在， 我们该将点击事件中的约束布局从makeConstraints改变成updateConstraints来试试两者有什么区别(下面只添加了点击事件的代码，其他事重复的就不多此一举了）
```
func buttonAction() {
    // 注意这里是updateConstraints， 而不是makeConstraints
    blackView.snp.updateConstraints { (make) in
        make.width.height.equalTo(20)
        make.top.equalTo(300)
    }
    print("这里试试snapkit有没有报警告")
}
```
![其他](/assets/ios_framework/if_15.png)
![其他](/assets/ios_framework/if_16.png)
![其他](/assets/ios_framework/if_17.png)

> 发现没有，在将makeConstraints改变成updateConstraints之后，约束还是4个，snapkit没有报警告，点击事件中的width、height、top全部起了作用，而这就是两者的本质区别：makeConstraints是制作约束，在原来的基础上再添加另外的约束，也就是画蛇添足，约束增加，视图布局就有不确定性，从而有些约束起作用，有些不起作用（如上面的top），snapkit报警告！！！而updateConstraints是更新约束，改变原有约束，约束不会增加，没经过updateConstraints处理的保持原有约束，经过处理就更新约束，约束不会减少，snapkit不会产生警告，这是正常标准的更新约束的正确方式！！！

- 重做约束（remakeConstraints）
重做约束的本质就是：去掉已有的所有约束， 重新做约束，记住，是做约束， 也就是说， 使用了remakeConstraints后，重做的约束必须要能确定相应视图的大小和位置, 之前makeConstraints的约束已经不会存在了，完全销毁！！！
```
// 点击更新/制作约束
func buttonAction() {
    
    // 注意这里是 remakeConstraints
    blackView.snp.remakeConstraints { (make) in
        make.width.height.equalTo(20)
        make.top.equalTo(300)
    }
    
    print("这里试试snapkit有没有报警告")
}
```
![重做约束](/assets/ios_framework/if_18.png)
![重做约束](/assets/ios_framework/if_19.png)
![重做约束](/assets/ios_framework/if_20.png)
> 我们看到， blackView重做了约束， 之前的约束不起任何作用，由于它在重做约束后只有 3 个约束分别是 width、height、top, 但是这里有一个问题，就是这 3 个约束只能确定大小，无法确定视图的位置， 所以在水平方向上或者左右缺少一个布局条件， 故 blackView整体视图的x紧靠左边（默认）！ 另外我们发现， 在图（3）中，右上角出现了一个感叹号“！”, 那是因为告诉你缺少了一个约束条件：x-xcode-debug-views://7f81fcbc7900: runtime: Layout Issues: Horizontal position is ambiguous for UIView.

# MJRefresh
[github地址](https://github.com/CoderMJLee/MJRefresh)
在项目开发的过程中，我们会使用到很多次的下拉刷新和下拉加载等操作。使用系统自带的UIRefreshController可以很方便的实现我们想要的功能。但是同样存在着很多比较优秀的下拉刷新框架，这里我们介绍的就是MJRefresh
MJRefresh是使用OC编写的第三方库，可以同时实现下拉刷新和上拉加载等操作。支持的组件有UIScrollView、UITableView、UICollectionView、UIWebView
![MJRefresh类结构图](/assets/ios_framework/if_21.png)
使用的方式，如何安装，这里我们就不做介绍
## TableView
给tableView添加刷新操作，每次刷新添加10条数据，并且完成Table的自动刷新
生成的数据会加载两秒钟，模拟网络请求
![图1](/assets/ios_framework_if_22.png)
![图1](/assets/ios_framework_if_23.png)
![图1](/assets/ios_framework_if_24.png)
1. 对于下拉刷新的操作，我们可通过target action的来触发即可
```
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String]!
    var tableView:UITableView?
     
    // 顶部刷新
    let header = MJRefreshNormalHeader()
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        refreshItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //下拉刷新相关设置
        header.setRefreshingTarget(self, refreshingAction: #selector(ViewController.headerRefresh))
        self.tableView!.mj_header = header
    }
     
    //初始化数据
    func refreshItemData() {
        items = []
        for _ in 0...9 {
            items.append("条目\(Int(arc4random()%100))")
        }
    }
     
    //顶部下拉刷新
    @objc func headerRefresh(){
        print("下拉刷新.")
        sleep(2)
        //重现生成数据
        refreshItemData()
        //重现加载表格数据
        self.tableView!.reloadData()
        //结束刷新
        self.tableView!.mj_header.endRefreshing()
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //为了提供表格显示性能，已创建完成的单元需重复使用
            let identify:String = "SwiftCell"
            //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
            cell.accessoryType = .disclosureIndicator
            cell.textLabel?.text = self.items[indexPath.row]
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```
2. 响应下拉刷新的方法也放在闭包中
```
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String]!
    var tableView:UITableView?
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        refreshItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
     
 
        //下拉刷新相关设置,使用闭包Block
        self.tableView!.mj_header = MJRefreshNormalHeader(refreshingBlock: {
            print("下拉刷新.")
            sleep(2)
            //重现生成数据
            self.refreshItemData()
            //重现加载表格数据
            self.tableView!.reloadData()
            //结束刷新
            self.tableView!.mj_header.endRefreshing()
        })
    }
     
    //初始化数据
    func refreshItemData() {
        items = []
        for _ in 0...9 {
            items.append("条目\(Int(arc4random()%100))")
        }
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //为了提供表格显示性能，已创建完成的单元需重复使用
            let identify:String = "SwiftCell"
            //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
            cell.accessoryType = .disclosureIndicator
            cell.textLabel?.text = self.items[indexPath.row]
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```
3. 对MJRefresh默认样式的修改
上面的都是默认样式，会显示刷新的状态提示文字，刷新时间，左侧还有箭头或环形进度条表示刷新状态
- 隐藏事件
```
header.lastUpdatedTimeLabel.isHidden = true
```
- 隐藏时间和状态文字
```
//隐藏时间
header.lastUpdatedTimeLabel.isHidden = true
//隐藏状态
header.stateLabel.isHidden = true
```
- 自定义文字和样式
```
//修改文字
header.setTitle("下拉下拉下拉", for: .idle)
header.setTitle("松开松开松开", for: .pulling)
header.setTitle("刷新刷新刷新", for: .refreshing)
         
//修改字体
header.stateLabel.font = UIFont.systemFont(ofSize: 15)
header.lastUpdatedTimeLabel.font = UIFont.systemFont(ofSize: 13)
         
//修改文字颜色
header.stateLabel.textColor = UIColor.red
header.lastUpdatedTimeLabel.textColor = UIColor.blue
```
- 自定义图标
下拉刷新的图标是可以修改的。不同的状态，我们都可以设置一个图片数组，MJRefresh 就会自动播放这几张图片，形成动画。
其中下拉过程中的图片是根据下拉的距离自动改变。而提示松开刷新，以及正在刷新这两个状态下的图片是定时切换播放的。
（注意：如果要设置图标，header 就要使用 MJRefreshGifHeader,而不是 MJRefreshNormalHeader。）
```
//下拉过程时的图片集合(根据下拉距离自动改变)
var idleImages = [UIImage]()
for i in 1...10 {
    idleImages.append(UIImage(named:"idle\(i)")!)
}
header.setImages(idleImages, for: .idle) //idle1，idle2，idle3...idle10
 
//下拉到一定距离后，提示松开刷新的图片集合(定时自动改变)
var pullingImages = [UIImage]()
for i in 1...3 {
    pullingImages.append(UIImage(named:"pulling\(i)")!)
}
header.setImages(pullingImages, for: .pulling)
 
//刷新状态下的图片集合(定时自动改变)
var refreshingImages = [UIImage]()
for i in 1...3 {
    refreshingImages.append(UIImage(named:"refreshing\(i)")!)
}
header.setImages(refreshingImages, for: .refreshing)
```
- 动画图片切换的时间也是可以修改的
```
//下面表示刷新图片在1秒钟的时间内播放一轮
header.setImages(refreshingImages, duration: 1, for: .refreshing)
```

- 通过代码调用下拉刷新
上面的样例中，我们都是通过下拉操作来实现 MJRefresh 组件的刷新功能。其实也可以通过调用组件的刷新方法来实现同样功能（这个同样会有下拉、收回等动画效果）
```
//手动调用刷新效果
header.beginRefreshing()
```
## 上拉更多
初始化的时候自动生成20条数据用于表格默认显示
当点击最底部的“点击或上拉加载更多”，或者在列表底部继续上拉就会添加20条新数据进来（随机生成，生成数据的时候会等待2秒，模拟网络请求） 
![上啦加载更多](/assets/ios_framework/if_25.png)
- 对于上拉加载的响应事件，我们可以通过设置其 target action 来关联
```
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = []
    var tableView:UITableView?
     
    // 底部加载
    let footer = MJRefreshAutoNormalFooter()
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        loadItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //上刷新相关设置
        footer.setRefreshingTarget(self, refreshingAction: #selector(ViewController.footerLoad))
        //是否自动加载（默认为true，即表格滑到底部就自动加载）
        footer.isAutomaticallyRefresh = false
        self.tableView!.mj_footer = footer
    }
     
    //初始化数据
    func loadItemData() {
        for _ in 0...20 {
            items.append("条目\(Int(arc4random()%100))")
        }
    }
     
    //底部上拉加载
    @objc func footerLoad(){
        print("上拉加载.")
        sleep(2)
        //生成并添加数据
        loadItemData()
        //重现加载表格数据
        self.tableView!.reloadData()
        //结束刷新
        self.tableView!.mj_footer.endRefreshing()
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //为了提供表格显示性能，已创建完成的单元需重复使用
            let identify:String = "SwiftCell"
            //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
            cell.accessoryType = .disclosureIndicator
            cell.textLabel?.text = self.items[indexPath.row]
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```
- 上拉响应方法也可以直接写在闭包（Block）中
```
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = []
    var tableView:UITableView?
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        loadItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //上拉加载相关设置,使用闭包Block
        self.tableView!.mj_footer = MJRefreshAutoNormalFooter(refreshingBlock: {
            print("上拉加载.")
            sleep(2)
            //生成并添加数据
            self.loadItemData()
            //重现加载表格数据
            self.tableView!.reloadData()
            //结束刷新
            self.tableView!.mj_footer.endRefreshing()
        })
    }
     
    //初始化数据
    func loadItemData() {
        for _ in 0...20 {
            items.append("条目\(Int(arc4random()%100))")
        }
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //为了提供表格显示性能，已创建完成的单元需重复使用
            let identify:String = "SwiftCell"
            //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
            cell.accessoryType = .disclosureIndicator
            cell.textLabel?.text = self.items[indexPath.row]
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```
MJRefresh上啦加载
- 自定义上啦刷新图标
同下拉组件里的刷新图标一样。上拉加载里的刷新图标我们也可以修改。通过设置一个图片数组，MJRefresh 就会自动播放这几张图片，形成动画
```
// 底部加载
let footer = MJRefreshAutoGifFooter()
 
var refreshingImages = [UIImage]()
for i in 1...3 {
    refreshingImages.append(UIImage(named:"refreshing\(i)")!)
}
footer.setImages(refreshingImages, for: .refreshing)
```
- 动画图片切换的时间也可以修改
```
//下面表示刷新图片在1秒钟的时间内播放一轮
footer.setImages(refreshingImages, duration: 1, for: .refreshing)
```
- 刷新状态下隐藏文字
```
//刷新时不显示文字（其它情况下还是有提示文字的）
footer.isRefreshingTitleHidden = true
```
- 将状态该问全局数据已经加载完毕
如果请求后发现所有的数据都已加载完毕。可以调用组件 endRefreshingWithNoMoreData() 方法，表示没有更多的数据可以加载了。其相关的提示文字也会改变。
```
self.tableView!.mj_footer.endRefreshingWithNoMoreData()
```
设置为“全部加载”状态后，怎么上拉都不会触发加载事件。如果想恢复上拉加载功能，可以使用 resetNoMoreData() 方法进行重置。
```
self.tableView!.mj_footer.resetNoMoreData()
```
- 自定义文字和文字样式
```
//修改文字
footer.setTitle("上拉上拉上拉", for: .idle)
footer.setTitle("加载加载加载", for: .refreshing)
footer.setTitle("没有没有更多数据了", for: .noMoreData)
 
//修改字体
footer.stateLabel.font = UIFont.systemFont(ofSize: 15)
 
//修改文字颜色
footer.stateLabel.textColor = UIColor.red
```
- 隐藏上啦加载组件
当然隐藏后上拉加载的功能也失效了
```
self.tableView!.mj_footer.isHidden = true
```
- 使用自动上拉回弹的组件
前面介绍的都是普通上拉组件，即默认会占用表格最后一行的空间。而使用 MJRefreshBackNormalFooter，单元格空间不会被占用。只有在最后一行位置上拉时，才回显示出上拉加载组件。具体效果类似于下拉刷新组件。
```
//自动回弹的上拉加载组件
let footer = MJRefreshBackNormalFooter()
```
- 自定义自动回弹的上拉加载组件刷新图标
同样地。对于自动回弹上拉组件不同状态下的图片数组也是可以修改设置的。如果要设置图标，我们可以改用 MJRefreshBackGifFooter 即可。
```
// 底部加载
let footer = MJRefreshBackGifFooter()
 
//上拉过程时的图片集合(根据下拉距离自动改变)
var idleImages = [UIImage]()
for i in 1...10 {
    idleImages.append(UIImage(named:"idle\(i)")!)
}
footer.setImages(idleImages, for: .idle) //idle1，idle2，idle3...idle10
 
//上拉到一定距离后，提示松开刷新的图片集合(定时自动改变)
var pullingImages = [UIImage]()
for i in 1...3 {
    pullingImages.append(UIImage(named:"pulling\(i)")!)
}
footer.setImages(pullingImages, for: .pulling)
 
//刷新状态下的图片集合(定时自动改变)
var refreshingImages = [UIImage]()
for i in 1...3 {
    refreshingImages.append(UIImage(named:"refreshing\(i)")!)
}
footer.setImages(refreshingImages, for: .refreshing)
```
- 通过代码调用自动回弹的上拉组件的加载行为
前面的样例中，我们都是通过上拉操作来实现 MJRefreshBackNormalFooter 组件的上拉加载功能。其实也可以通过调用组件的刷新方法来实现同样功能（这个同样会有上拉、收回等动画效果）

## 继承MJRefreshAutoFooter
（1）这个原来是 MJRefresh 提供的一个样例，我这里改成使用 Swift 实现。
（2）上拉组件视图中放置3个控件：UIActivityIndicatorView、UILabel、UISwitch。
（3）通常状态下开关是关闭的，到了“正在刷新”状态下开关会自动变成打开状态。
（4）UIActivityIndicatorView 活动指示器只有在“正在刷新”状态下会显示出来。
- 自定义组件的实现
```
class MJDIYAutoFooter: MJRefreshAutoFooter
{
    var label:UILabel!
    var s:UISwitch!
    var loading:UIActivityIndicatorView!
     
    //在这里做一些初始化配置（比如添加子控件）
    override func prepare()
    {
        super.prepare()
         
        // 设置控件的高度
        self.mj_h = 50
         
        // 添加label
        self.label =  UILabel()
        self.label.textColor = UIColor(red:1.0, green:0.5, blue:0.0, alpha:1.0)
        self.label.font = UIFont.boldSystemFont(ofSize: 16)
        self.label.textAlignment = .center
        self.addSubview(self.label)
 
        // 打酱油的开关
        self.s =  UISwitch()
        self.addSubview(self.s)
 
        // loading
        self.loading =  UIActivityIndicatorView(activityIndicatorStyle: .gray)
        self.addSubview(self.loading)
    }
     
    //在这里设置子控件的位置和尺寸
    override func placeSubviews()
    {
        super.placeSubviews()
        self.label.frame = self.bounds
        self.s.center = CGPoint(x:self.mj_w - 20, y:self.mj_h - 20)
        self.loading.center = CGPoint(x:30, y:self.mj_h * 0.5)
    }
     
    //监听控件的刷新状态
    override var state: MJRefreshState{
        didSet
        {
            switch (state) {
            case .idle:
                self.label.text = "赶紧上拉吖(开关是打酱油滴)"
                self.loading.stopAnimating()
                self.s.setOn(false, animated:true)
                break
            case .refreshing:
                self.s.setOn(true, animated:true)
                self.label.text = "加载数据中(开关是打酱油滴)"
                self.loading.startAnimating()
                break
            case .noMoreData:
                self.label.text = "木有数据了(开关是打酱油滴)"
                self.s.setOn(false, animated:true)
                self.loading.stopAnimating()
                break
            default:
                break
            }
        }
    }
 
    //监听scrollView的contentOffset改变
    override func scrollViewContentOffsetDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewContentOffsetDidChange(change)
    }
     
    //监听scrollView的contentSize改变
    override func scrollViewContentSizeDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewContentSizeDidChange(change)
    }
     
    //监听scrollView的拖拽状态改变
    override func scrollViewPanStateDidChange(_ change: [AnyHashable : Any]!) {
        super.scrollViewPanStateDidChange(change)
    }
}
```
- 组件使用样式
```
import UIKit
 
class ViewController: UIViewController, UITableViewDelegate, UITableViewDataSource {
     
    var items:[String] = []
    var tableView:UITableView?
     
    // 底部加载
    let footer = MJDIYAutoFooter()
     
    override func loadView() {
        super.loadView()
    }
     
    override func viewDidLoad() {
        super.viewDidLoad()
         
        //随机生成一些初始化数据
        loadItemData()
         
        //创建表视图
        self.tableView = UITableView(frame: self.view.frame, style:.plain)
        self.tableView!.delegate = self
        self.tableView!.dataSource = self
        //创建一个重用的单元格
        self.tableView!.register(UITableViewCell.self,
                                 forCellReuseIdentifier: "SwiftCell")
        self.view.addSubview(self.tableView!)
         
        //上刷新相关设置
        footer.setRefreshingTarget(self, refreshingAction: #selector(ViewController.footerLoad))
        //是否自动加载（默认为true，即表格滑到底部就自动加载）
        footer.isAutomaticallyRefresh = false
        self.tableView!.mj_footer = footer
    }
     
    //初始化数据
    func loadItemData() {
        for _ in 0...20 {
            items.append("条目\(Int(arc4random()%100))")
        }
    }
     
    //底部上拉加载
    @objc func footerLoad(){
        print("上拉加载.")
        sleep(2)
        //生成并添加数据
        loadItemData()
        //重现加载表格数据
        self.tableView!.reloadData()
        //结束刷新
        self.tableView!.mj_footer.endRefreshing()
    }
     
    //在本例中，只有一个分区
    func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
     
    //返回表格行数（也就是返回控件数）
    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return self.items.count
    }
     
    //创建各单元显示内容(创建参数indexPath指定的单元）
    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath)
        -> UITableViewCell {
            //为了提供表格显示性能，已创建完成的单元需重复使用
            let identify:String = "SwiftCell"
            //同一形式的单元格重复使用，在声明时已注册
            let cell = tableView.dequeueReusableCell(withIdentifier: identify,
                                                     for: indexPath)
            cell.accessoryType = .disclosureIndicator
            cell.textLabel?.text = self.items[indexPath.row]
            return cell
    }
     
    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
    }
}
```

# LLCycleScrollView
[github地址](https://github.com/LvJianfeng/LLCycleScrollView)
LLCyckeScrollView是一款功能比较强大的轮播图插件，这个使用的方式就不在显示，直接开始主要内容
## 支持的内容
- 支持纯图片
- 支持文本图片结合
- 支持横向滚动
- 支持纵向滚动
- 支持手势滑动
- 支持点击回调
- 支持图片数据的延时加载
- 支持没有数据，占位图占位(仅设置CoverImage(有默认图)即可)
- 支持本地图片显示及与网络图的混合显示
- 支持系统UIPageControl位置设置
- 支持StoryBoard
- 支持纯文本
- 支持CustomPageControl位置设置
- 支持协议
## 使用方法
我们在使用的时候一般会将轮播图作为一个组件添加到页面中，所以我这里直接使用一个闭包来实现
```
let banner = LLCycleScrollView.llCycleScrollViewWithFrame(CGRect.init(x:0,y:banner.lly + 205,width:UIScreen.main.bound.size.width,height:200))
//设置是否自动滚动
banner.autoScroll = true
//是否无限循环
banner.infiniteLoop = true
//滚动时间间隔
banner.autoScrollTimeInterval = 3.0
//等待图片显示时的占位图片
bannerDemo.placeHolderImage = #UIImage
// 如果没有数据的时候，使用的封面图
bannerDemo.coverImage = #UIImage
// 设置图片显示方式=UIImageView的ContentMode
bannerDemo.imageViewContentMode = .scaleToFill
// 设置滚动方向（ vertical || horizontal ）
bannerDemo.scrollDirection = .vertical
// 设置当前PageControl的样式 (.none, .system, .fill, .pill, .snake)
bannerDemo.customPageControlStyle = .snake
// 非.system的状态下，设置PageControl的tintColor
bannerDemo.customPageControlInActiveTintColor = UIColor.red
// 设置.system系统的UIPageControl当前显示的颜色
bannerDemo.pageControlCurrentPageColor = UIColor.white
// 非.system的状态下，设置PageControl的间距(默认为8.0)
bannerDemo.customPageControlIndicatorPadding = 8.0
// 设置PageControl的位置 (.left, .right 默认为.center)
bannerDemo.pageControlPosition = .center
// 背景色
bannerDemo.collectionViewBackgroundColor
//将数据添加到view中
view.addSubview(banner)
// 模拟网络图片获取
DispatchQueue.main.asyncAfter(deadline: DispatchTime.now() + .seconds(2)) {
  bannerDemo.imagePaths = imagesURLStrings
}
```

# Reusable
[github地址](https://github.com/AliSoftware/Reusable)
我们在进行开发的过程中，肯定会遇到自定义tableView的cell的情况，那么每次我们在使用cell的时候都需要设置一些表示类的内容，那么这样也相当增加我们的工作量。那么Reusable就是Swift下的一个通过protocol extension结合泛型来进行优雅的解决方案。(一听就知道是官方的话，反正意思就是说好用)
## 适用范围
- UITableViewCell
- UICollectionViewCells
- UIViewControllers

## 使用方式
###  根据类型获取Cell
首先需要让我们的Cell申明为Reusable或者是NibReusable
```
//如果cell定义在xib中，声明NibReusable
class MyCustomCell: UITableViewCell, NibReusable { }

//如果cell是基于纯代码的，声明Reusable
class MyCustomCell: UITableViewCell, Reusable { }
```
其次，在tableView或者CollectionView中使用register方法
```
tableView.registerReusableCell(MyCustomCell)
```
最后在使用的时候，直接上就可以了
```
func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
  let cell: MyCustomCell = tableView.dequeueReusableCell(indexPath: indexPath)
  … // configure the cell, which is already of the expected MyCustomCell type
  return cell
}
```
### 根据类型获取XIB中的UIView对象
UIView对象声明NibLoadable协议
利用<code>MyCustomView.loadFromNib()</code>就可以从“MyCustomView.xib”中实例化返回MyCustomView的实例对象

### 根据类型获取Storyboards中的UIViewController对象
UIViewController对象声明<code>StoryboardBased</code>或者<code>StoryboardSceneBased</code>协议。
利用<code>YourCustomViewController.instantiate()</code>就可以从Storyboard中实例化返回实例对象

### 实现
使用自己的类名，从新做一个标识
先定义一个静态变量，如下
```
protocol Reusable: class {
  static var reuseIdentifier: String { get }
}

extension Reusable {
  static var reuseIdentifier: String {
    // I like to use the class's name as an identifier
    // so this makes a decent default value.
    return String(Self)
  }
}
```
接着我们在给tableView写一个自定义获取reuse cell的方法
```
func dequeueReusableCell<T: Reusable>(indexPath indexPath: NSIndexPath) -> T {
  return self.dequeueReusableCellWithIdentifier(T.reuseIdentifier, forIndexPath: indexPath) as! T
}
```
注意<T: Reusable>这个泛型参数，这个泛型是根据返回值的类型来确定的。所以返回的cell必须实现Reusable协议。我们将这类型里的那个静态变量T.reuseIdentifier作为Identifier。

我们当然还要同时改造register方法。
```
public extension UITableView {
  final func registerReusableCell<T: UITableViewCell where T: Reusable>(cellType: T.Type) {
    self.registerClass(cellType.self, forCellReuseIdentifier: cellType.reuseIdentifier)
  }
}
```
这个时候我们忽然意识到，还有<code>registerNib</code>没有解决。
思路也是相似的，给协议再增加一个返回nib对象的静态变量呗。就像这样：
```
protocol Reusable: class {
 static var reuseIdentifier: String { get } 
 static var nib: UINib? { get }
}
```
实现是这样子的
```
static var nib: UINib {
    return UINib(nibName: String(self), bundle: NSBundle(forClass: self))
  }
```
但是这里再往深一点想，其实载入nib和reuseIdentifier是两件事，因为我们有时也会从xib获取其他UIView的对象。protocol也提供了组合的特性。所以我们可以把获取nib单独拆出来。
```
public protocol NibLoadable: class {
  /// The nib file to use to load a new instance of the View designed in a XIB
  static var nib: UINib { get }
}

public protocol NibReusable: Reusable, NibLoadable {}
```
最后一步
```
final func registerReusableCell<T: UITableViewCell where T: NibReusable>(cellType: T.Type) {
    self.registerNib(cellType.nib, forCellReuseIdentifier: cellType.reuseIdentifier)
  }
```
同样可以再写一个UIView的扩展，根据类型获取实例
```
public extension NibLoadable where Self: UIView {

  static func loadFromNib() -> Self {
    guard let view = nib.instantiateWithOwner(nil, options: nil).first as? Self else {
      fatalError("The nib \(nib) expected its root view to be of type \(self)")
    }
    return view
  }
}
```

# MBProgressHUD
[github地址]()
MBProgressHUD简单来说就是显示一个弹窗，然后这个弹窗可以是正在加载，也可以是提示信息等。
## 实现例子的思路
1. 创建五个按钮，并且为每一个按钮添加不同的点击事件
2. 给五个按钮添加点击事件之后，在点击事件中执行响应的操作

```
@IBAction func TextDialogBtn(sender: AnyObject) {  
        showTextDialog()  
    }  
    @IBAction func ProgressDialogBtn1(sender: AnyObject) {  
        showProgressDialog1()  
    }  
    @IBAction func ProgressDialogBtn2(sender: AnyObject) {  
        showProgressDialog2()  
    }  
    @IBAction func CustomDialogBtn(sender: AnyObject) {  
        showCustomDialog()  
    }  
    @IBAction func AllTextDialogBtn(sender: AnyObject) {  
        showAllTextDialog()  
    }  
  
    var HUD : MBProgressHUD?  
    override func viewDidLoad() {  
        super.viewDidLoad()  
        // Do any additional setup after loading the view, typically from a nib.  
  
    }  
    //文本提示框  
    func showTextDialog(){  
        //初始化对话框，置于当前的View当中  
        HUD = MBProgressHUD(view: self.view)  
        self.view.addSubview(HUD!)  
        //如果设置此属性，则当前view置于后台  
        HUD?.dimBackground = true  
        //设置对话框文字  
        HUD?.labelText = "请稍等"  
        HUD?.showAnimated(true, whileExecutingBlock: {  
            sleep(3)  
            }, completionBlock: {  
            self.HUD?.removeFromSuperview()  
            self.HUD = nil  
        })  
    }  
    //框型进度提示  
    func showProgressDialog1(){  
        //初始化对话框，置于当前的View当中  
        HUD = MBProgressHUD(view: self.view)  
        self.view.addSubview(HUD!)  
        //如果设置此属性，则当前view置于后台  
        HUD?.dimBackground = true  
        //设置对话框文字  
        HUD?.labelText = "正在加载"  
        //设置模式为进度框形的  
        HUD?.mode = MBProgressHUDMode.Determinate  
        HUD?.showAnimated(true, whileExecutingBlock: {  
            var progress : Float = 0.0  
            while(progress < 1.0){  
                progress += 0.01  
                self.HUD?.progress = progress  
                usleep(50000)  
            }  
            }, completionBlock: {  
                self.HUD?.removeFromSuperview()  
                self.HUD = nil  
        })  
    }  
    //进度条提示  
    func showProgressDialog2(){  
        //初始化对话框，置于当前的View当中  
        HUD = MBProgressHUD(view: self.view)  
        self.view.addSubview(HUD!)  
        //如果设置此属性，则当前view置于后台  
        HUD?.dimBackground = true  
        //设置对话框文字  
        HUD?.labelText = "正在加载"  
        //设置模式为进度条  
        HUD?.mode = MBProgressHUDMode.DeterminateHorizontalBar  
        HUD?.showAnimated(true, whileExecutingBlock: {  
            var progress : Float = 0.0  
            while(progress < 1.0){  
                progress += 0.01  
                self.HUD?.progress = progress  
                usleep(50000)  
            }  
            }, completionBlock: {  
                self.HUD?.removeFromSuperview()  
                self.HUD = nil  
        })  
    }  
    //自定义提示  
    func showCustomDialog(){  
        //初始化对话框，置于当前的View当中  
        HUD = MBProgressHUD(view: self.view)  
        self.view.addSubview(HUD!)  
        //如果设置此属性，则当前view置于后台  
        HUD?.dimBackground = true  
        //设置对话框文字  
        HUD?.labelText = "操作成功"  
        //设置模式为自定义  
        HUD?.mode = MBProgressHUDMode.CustomView  
        HUD?.customView = UIImageView(image: UIImage(named: "37x-Checkmark-1"))  
        HUD?.showAnimated(true, whileExecutingBlock: {  
            sleep(2)  
            }, completionBlock: {  
                self.HUD?.removeFromSuperview()  
                self.HUD = nil  
        })  
  
    }  
    //纯文本提示  
    func showAllTextDialog(){  
        //初始化对话框，置于当前的View当中  
        HUD = MBProgressHUD(view: self.view)  
        self.view.addSubview(HUD!)  
        //如果设置此属性，则当前view置于后台  
        HUD?.dimBackground = true  
        //设置模式为纯文本提示  
        HUD?.mode = MBProgressHUDMode.Text  
        //设置对话框文字  
        HUD?.labelText = "操作成功"  
        //指定距离中心点的X轴和Y轴的偏移量，如果不指定则在屏幕中间显示  
//        HUD?.yOffset = 150.0  
//        HUD?.xOffset = 150.0  
        HUD?.showAnimated(true, whileExecutingBlock: {  
            sleep(2)  
            }, completionBlock: {  
                self.HUD?.removeFromSuperview()  
                self.HUD = nil  
        })  
    }  
  
    override func didReceiveMemoryWarning() {  
        super.didReceiveMemoryWarning()  
        // Dispose of any resources that can be recreated.  
    }  
```
![文本提示框](/assets/ios_framework/if_21.png)
![进度提示框](/assets/ios_framework/if_22.png)
![进度提示框](/assets/ios_framework/if_23.png)
![自定义提示框](/assets/ios_framework/if_24.png)
![纯文本提示框](/assets/ios_framework/if_25.png)

# HMSegmentedControl
[github地址](https://github.com/HeshamMegid/HMSegmentedControl)


# IQKeyboardManager
[github地址](https://github.com/hackiftekhar/IQKeyboardManager)

# DZNEmptyDataSet
[github地址](https://github.com/dzenbot/DZNEmptyDataSet)

# KingFisher
[github地址](https://github.com/onevcat/Kingfisher)
KingFisher是一款专门针对下载和缓存图片的框架，通过使用KingFisher可以非常合理的对缓存进行管理
使用的方式非常简单
```
let url = URL(string: "url_of_your_image")
imageView.kf.setImage(with: url)
```

# HandyJSON
[github地址](https://github.com/alibaba/handyjson)

# Moya
[github地址](https://github.com/Moya/Moya)

# Then
[github地址](https://github.com/devxoul/Then)


# 参考资料
[Swift自动布局SnapKit](https://www.jianshu.com/p/2bad53a2a180)
[MJRefresh](https://www.jianshu.com/p/cd5640be74da)
[MJRefresh](http://www.hangge.com/blog/cache/detail_1406.html)
[Swift强大的轮播图](https://www.jianshu.com/p/de861704ceaa)
[Reusable](https://www.jianshu.com/p/255e02337176)
[MBProgressHUD](https://www.jianshu.com/p/d1f3f3535f03)
[HMSegmentedControl](https://blog.csdn.net/gfl_emily/article/details/73647223)
[IQKeyboardManager](https://www.jianshu.com/p/01c0682003a9)
[DZNEmptyDataSet](https://blog.csdn.net/sinat_28709097/article/details/52101211)
[KingFisher](https://blog.csdn.net/brycegao321/article/details/77892323)
[HandyJson](https://www.cnblogs.com/yajunLi/p/7121950.html)
[Moya网络抽象层](http://www.cocoachina.com/ios/20180307/22493.html)
[Swift让人眼前一亮的初始化方式--Then][https://segmentfault.com/a/1190000006992493]
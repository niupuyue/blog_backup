---
title: ios文件操作
date: 2018-05-16 09:15:30
tags:
  - ios
---
文件操作
<!--more-->
# 遍历某个文件夹的文件

## 获取当前路径
```
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in:.userDomainMask)
let url = urlForDocument[0] as URL
print(url)
```

结果：
![获取当前路径](/assets/ios_file/if_01.png)

## 对指定路径进行浅搜索，返回指定路径下的文件，子目录和符号链接的列表
```
let contentsOfPath = try? manager.contentsOfDirectory(atPath: url.path)
print("contentsOfPath: \(contentsOfPath)")
```
有另外一种写法
```
let contentsOfURL = try? manager.contentsOfDirectory(at: url,
                        includingPropertiesForKeys: nil, options: .skipsHiddenFiles)
print("contentsOfURL: \(contentsOfURL)")
```
![浅遍历](/assets/ios_file/if_02.png)
## 深度遍历，会递归遍历子文件夹和子文件(但不会递归符号链接)
```
let enumeratorAtPath = manager.enumerator(atPath: url.path)
print("enumeratorAtPath: \(enumeratorAtPath?.allObjects)")
```
![深遍历](/assets/ios_file/if_03.png)
深度遍历，会递归遍历子文件夹（但不会递归符号链接）
```
let enumeratorAtURL = manager.enumerator(at: url, includingPropertiesForKeys: nil,
                                         options: .skipsHiddenFiles, errorHandler:nil)
print("enumeratorAtURL: \(enumeratorAtURL?.allObjects)")
```
![深遍历获取全路径](/assets/ios_file/if_04.png)
## 深度遍历，会递归遍历子文件夹（包括符号链接，所以要求性能的话用enumeratorAtPath）
```
let subPaths = manager.subpaths(atPath: url.path)
print("subPaths: \(subPaths)")
```
![深度遍历](/assets/ios_file/if_05.png)

# 判断文件或者文件夹是否存在
```
let fileManager = FileManager.default
let filePath:String = NSHomeDirectory() + "/Documents/hangge.txt"
let exist = fileManager.fileExists(atPath: filePath)
```
![判断文件或者文件夹是否存在](/assets/iof_file/if_06.png)

# 创建文件夹
```
let myDirectory:String = NSHomeDirectory() + "/Documents/myFolder/Files"
let fileManager = FileManager.default
 
//withIntermediateDirectories为ture表示路径中间如果有不存在的文件夹都会创建
try! fileManager.createDirectory(atPath: myDirectory,
                        withIntermediateDirectories: true, attributes: nil)
```
![创建文件夹](/assets/ios_file/if_07.png)
可以看到创建之后的数组中已经有了新的文件夹
第二种方法
```
func createFolder(name:String,baseUrl:NSURL){
    let manager = FileManager.default
    let folder = baseUrl.appendingPathComponent(name, isDirectory: true)
    print("文件夹: \(folder)")
    let exist = manager.fileExists(atPath: folder!.path)
    if !exist {
        try! manager.createDirectory(at: folder!, withIntermediateDirectories: true,
                                     attributes: nil)
    }
}
     
//在文档目录下新建folder目录
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in: .userDomainMask)
let url = urlForDocument[0] as NSURL
createFolder(name: "folder", baseUrl: url)
```

# 将对象写入文件
可以通过write(to:)方法，可以创建文件并将对象写入，对象包括String，NSString，UIImage，NSArray，NSDictionary等
## 把String保存到文件
```
 let filePath:String = NSHomeDirectory()+"/Documents/password.txt"
        let msg = "我是测试内容，就问你怕不怕"
        try! msg.write(toFile: filePath, atomically: true, encoding: String.Encoding.utf8)
```
![把String保存到文件覆盖的形式](/assets/ios_file/if_08.png)
## 把图片保存到路径下
```
let filePath = NSHomeDirectory() + "/Documents/hangge.png"
let image = UIImage(named: "apple.png")
let data:Data = UIImagePNGRepresentation(image!)!
try? data.write(to: URL(fileURLWithPath: filePath))
```
这个实验之后是可以的，但是这个图片比较难搞定，所以暂时先不贴出来了
### 把NSArray写在文件路径下
```
let array = NSArray(objects: "aaa","bbb","ccc")
let filePath:String = NSHomeDirectory() + "/Documents/array.plist"
array.write(toFile: filePath, atomically: true)
```
![把NSArray写在文件中](/assets/ios_file/if_09.png)

### 把NSDirectionary保存到文件中
```
let dictionary:NSDictionary = ["Gold": "1st Place", "Silver": "2nd Place"]
let filePath:String = NSHomeDirectory() + "/Documents/dictionary.plist"
dictionary.write(toFile: filePath, atomically: true)
```
![把NSDirectionary保存到文件中](/assets/ios_file/if_10.png)

# 创建文件
```
 //在文档目录下新建test.txt文件
        let manager = FileManager.default
        let urlForDocument = manager.urls( for: .documentDirectory,
                                           in:.userDomainMask)
        let url = urlForDocument[0]
        createFile(name:"test.txt", fileBaseUrl: url)
//创建文件的方法
func createFile(name:String, fileBaseUrl:URL){
        let manager = FileManager.default
        
        let file = fileBaseUrl.appendingPathComponent(name)
        print("文件: \(file)")
        let exist = manager.fileExists(atPath: file.path)
        if !exist {
            let data = Data(base64Encoded:"aGVsbG8gd29ybGQ=" ,options:.ignoreUnknownCharacters)
            let createSuccess = manager.createFile(atPath: file.path,contents:data,attributes:nil)
            print("文件创建结果: \(createSuccess)")
        }
    }
```
![创建文件](/assets/ios_file/if_11.png)
# 复制文件
```
let fileManager = FileManager.default
let homeDirectory = NSHomeDirectory()
let srcUrl = homeDirectory + "/Documents/hangge.txt"
let toUrl = homeDirectory + "/Documents/copyed.txt"
try! fileManager.copyItem(atPath: srcUrl, toPath: toUrl)
```
![复制文件](/assets/ios_file/if_12/png)
另一种方法
```
// 定位到用户文档目录
let manager = FileManager.default
let urlForDocument = manager.urls( for:.documentDirectory, in:.userDomainMask)
let url = urlForDocument[0]
 
// 将test.txt文件拷贝到文档目录根目录下的copyed.txt文件
let srcUrl = url.appendingPathComponent("test.txt")
let toUrl = url.appendingPathComponent("copyed.txt")
 
try! manager.copyItem(at: srcUrl, to: toUrl)
```

# 移动文件
```
let fileManager = FileManager.default
let homeDirectory = NSHomeDirectory()
let srcUrl = homeDirectory + "/Documents/hangge.txt"
let toUrl = homeDirectory + "/Documents/moved/hangge.txt"
try! fileManager.moveItem(atPath: srcUrl, toPath: toUrl)
```
另外一种方法
```
// 定位到用户文档目录
let manager = FileManager.default
let urlForDocument = manager.urls( for: .documentDirectory, in:.userDomainMask)
let url = urlForDocument[0]
 
let srcUrl = url.appendingPathComponent("test.txt")
let toUrl = url.appendingPathComponent("copyed.txt")
// 移动srcUrl中的文件（test.txt）到toUrl中（copyed.txt）
try! manager.moveItem(at: srcUrl, to: toUrl)
```
# 删除文件
```
let fileManager = FileManager.default
let homeDirectory = NSHomeDirectory()
let srcUrl = homeDirectory + "/Documents/hangge.txt"
try! fileManager.removeItem(atPath: srcUrl)
```
另一种方法
```
// 定位到用户文档目录
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in:.userDomainMask)
let url = urlForDocument[0]
 
let toUrl = url.appendingPathComponent("copyed.txt")
// 删除文档根目录下的toUrl路径的文件（copyed.txt文件）
try! manager.removeItem(at: toUrl)
```

# 删除目录下所有的文件
```
let fileManager = FileManager.default
let myDirectory = NSHomeDirectory() + "/Documents/Files"
let fileArray = fileManager.subpaths(atPath: myDirectory)
for fn in fileArray!{
    try! fileManager.removeItem(atPath: myDirectory + "/\(fn)")
}
```
删除目录后，再创建该目录
```
let fileManager = FileManager.default
let myDirectory = NSHomeDirectory() + "/Documents/Files"
try! fileManager.removeItem(atPath: myDirectory)
try! fileManager.createDirectory(atPath: myDirectory, withIntermediateDirectories: true,
                                 attributes: nil)
```

# 读取文件
```
let manager = FileManager.default
let urlsForDocDirectory = manager.urls(for: .documentDirectory, in:.userDomainMask)
let docPath = urlsForDocDirectory[0]
let file = docPath.appendingPathComponent("test.txt")
 
//方法1
let readHandler = try! FileHandle(forReadingFrom:file)
let data = readHandler.readDataToEndOfFile()
let readString = String(data: data, encoding: String.Encoding.utf8)
print("文件内容: \(readString)")
 
//方法2
let data2 = manager.contents(atPath: file.path)
let readString2 = String(data: data2!, encoding: String.Encoding.utf8)
print("文件内容: \(readString2)")
```
![读取文件](/assets/ios_file/if_13.png)

# 在任意位置写入数据
```
let manager = FileManager.default
let urlsForDocDirectory = manager.urls(for:.documentDirectory, in:.userDomainMask)
let docPath = urlsForDocDirectory[0]
let file = docPath.appendingPathComponent("test.txt")
 
let string = "添加一些文字到文件末尾"
let appendedData = string.data(using: String.Encoding.utf8, allowLossyConversion: true)
let writeHandler = try? FileHandle(forWritingTo:file)
writeHandler!.seekToEndOfFile()
writeHandler!.write(appendedData!)
```
![在任意位置写入数据](/assets/ios_file/if_14.png)

# 文件权限判断
```
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in:.userDomainMask)
let docPath = urlForDocument[0]
let file = docPath.appendingPathComponent("test.txt")
 
let readable = manager.isReadableFile(atPath: file.path)
print("可读: \(readable)")
let writeable = manager.isWritableFile(atPath: file.path)
print("可写: \(writeable)")
let executable = manager.isExecutableFile(atPath: file.path)
print("可执行: \(executable)")
let deleteable = manager.isDeletableFile(atPath: file.path)
print("可删除: \(deleteable)")
```
![文件权限判断](/assets/ios_file/if_15.png)

# 获取文件属性（创建时间，修改时间，文件大小，文件类型等信息
```
let manager = FileManager.default
let urlForDocument = manager.urls(for: .documentDirectory, in:.userDomainMask)
let docPath = urlForDocument[0]
let file = docPath.appendingPathComponent("test.txt")
 
let attributes = try? manager.attributesOfItem(atPath: file.path) //结果为Dictionary类型
print("attributes: \(attributes!)")
```
![获取文件属性](/assets/ios_file/if_16.png)
感觉内容太多，不清晰，我们可以拆分一下
```
print("创建时间：\(attributes![FileAttributeKey.creationDate]!)")
        print("修改时间：\(attributes![FileAttributeKey.modificationDate]!)")
        print("文件大小：\(attributes![FileAttributeKey.size]!)")
```
![文件属性](/assets/ios_file/if_17.png)




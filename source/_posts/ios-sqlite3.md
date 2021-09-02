---
title: ios-sqlite3数据库操作
date: 2018-05-18 16:30:23
tags:
    -ios
---

介绍
<!--more-->
sqlite是纯C语言中底层的数据库，在OC和Swift中都是经常使用的数据库，在开发中，可以使用代码创建数据库，可以使用图形化界面创建数据库。例如SQLiteManager、SQLiteStudio等

# 常用方法
| 方法名称 | 描述 |
| -------- | ------ |
| OpaquePointer: *db | 数据库句柄，跟文件句柄FIFL类似，这里是sqlite3指针 |
| sqlite3_stmt: *stmt | 相当于ODBC的Command对象，用于保存编译好的SQL语句 |
| sqlite3_open() | 打开数据库，没有数据库时创建 |
| sqlite3_exec() | 执行非查询的SQL语句 |
| sqlite3_step() | 在调用sqlite3_prepare后，使用这个函数在记录集中移动 |
| sqlite3_close() | 关闭数据库文件 |
| sqlite3_column_text() | 取text类型的数据 |
| sqlite3_column_blob() | 取blob类型的数据 |
| sqlite3_column_int() | 取int类型的数据 |

# 引入Sqlite
引入Sqlite数据库的方式，网上有很多，我这里就不在赘述了

# 具体操作
数据库连接操作(包括数据库的创建，表的创建，表的查询，修改，删除等操作)
```
class SQLiteConnect {
    
    var db :OpaquePointer? = nil
    let sqlitePath :String
    
    init?(path :String) {
        sqlitePath = path
        db = self.openDatabase(sqlitePath)
        
        if db == nil {
            return nil
        }
    }
    
    // 連結資料庫 connect database
    func openDatabase(_ path :String) -> OpaquePointer? {
        var connectdb: OpaquePointer? = nil
        if sqlite3_open(path, &connectdb) == SQLITE_OK {
            print("Successfully opened database \(path)")
            return connectdb!
        } else {
            print("Unable to open database.")
            return nil
        }
    }
    
    // 建立資料表 create table
    func createTable(_ tableName :String, columnsInfo :[String]) -> Bool {
        let sql = "create table if not exists \(tableName) "
            + "(\(columnsInfo.joined(separator: ",")))"
        
        if sqlite3_exec(self.db, sql.cString(using: String.Encoding.utf8), nil, nil, nil) == SQLITE_OK{
            return true
        }
        
        return false
    }
    
    // 新增資料
    func insert(_ tableName :String, rowInfo :[String:String]) -> Bool {
        var statement :OpaquePointer? = nil
        let sql = "insert into \(tableName) "
            + "(\(rowInfo.keys.joined(separator: ","))) "
            + "values (\(rowInfo.values.joined(separator: ",")))"
        
        if sqlite3_prepare_v2(self.db, sql.cString(using: String.Encoding.utf8), -1, &statement, nil) == SQLITE_OK {
            if sqlite3_step(statement) == SQLITE_DONE {
                return true
            }
            sqlite3_finalize(statement)
        }
        
        return false
    }
    
    // 讀取資料
    func fetch(_ tableName :String, cond :String?, order :String?) -> OpaquePointer {
        var statement :OpaquePointer? = nil
        var sql = "select * from \(tableName)"
        if let condition = cond {
            sql += " where \(condition)"
        }
        
        if let orderBy = order {
            sql += " order by \(orderBy)"
        }
        
        sqlite3_prepare_v2(self.db, sql.cString(using: String.Encoding.utf8), -1, &statement, nil)
        
        return statement!
    }
    
    // 更新資料
    func update(_ tableName :String, cond :String?, rowInfo :[String:String]) -> Bool {
        var statement :OpaquePointer? = nil
        var sql = "update \(tableName) set "
        
        // row info
        var info :[String] = []
        for (k, v) in rowInfo {
            info.append("\(k) = \(v)")
        }
        sql += info.joined(separator: ",")
        
        // condition
        if let condition = cond {
            sql += " where \(condition)"
        }
        
        if sqlite3_prepare_v2(self.db, sql.cString(using: String.Encoding.utf8), -1, &statement, nil) == SQLITE_OK {
            if sqlite3_step(statement) == SQLITE_DONE {
                return true
            }
            sqlite3_finalize(statement)
        }
        
        return false
        
    }
    
    // 刪除資料
    func delete(_ tableName :String, cond :String?) -> Bool {
        var statement :OpaquePointer? = nil
        var sql = "delete from \(tableName)"
        
        // condition
        if let condition = cond {
            sql += " where \(condition)"
        }
        
        if sqlite3_prepare_v2(self.db, sql.cString(using: String.Encoding.utf8), -1, &statement, nil) == SQLITE_OK {
            if sqlite3_step(statement) == SQLITE_DONE {
                return true
            }
            sqlite3_finalize(statement)
        }
        
        return false
    }
    
}
```
使用
```
var db :SQLiteConnect?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        
        // 資料庫檔案的路徑
        let urls = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)
        let sqlitePath = urls[urls.count-1].absoluteString + "sqlite3.db"
        
        // 印出儲存檔案的位置
        print(sqlitePath)
        
        // SQLite 資料庫
        db = SQLiteConnect(path: sqlitePath)
        
        if let mydb = db {
            
            // create table
            let _ = mydb.createTable("students", columnsInfo: [
                "id integer primary key autoincrement",
                "name text",
                "height double"])
            
            // insert
            let _ = mydb.insert("students", rowInfo: ["name":"'大強'","height":"178.2"])
            
            // select
            let statement = mydb.fetch("students", cond: "1 == 1", order: nil)
            while sqlite3_step(statement) == SQLITE_ROW{
                let id = sqlite3_column_int(statement, 0)
                let name = String(cString: sqlite3_column_text(statement, 1))
                let height = sqlite3_column_double(statement, 2)
                print("\(id). \(name) 身高： \(height)")
            }
            sqlite3_finalize(statement)
            
            // update
            let _ = mydb.update("students", cond: "id = 1", rowInfo: ["name":"'小強'","height":"176.8"])
            
            // delete
            let _ = mydb.delete("students", cond: "id = 5")
            
        }
    }
```

# 参考资料
[ios sqlite3的使用](https://www.cnblogs.com/XYQ-208910/p/6421840.html)
[Swift起步走](https://itisjoe.gitbooks.io/swiftgo/content/database/sqlite.html)
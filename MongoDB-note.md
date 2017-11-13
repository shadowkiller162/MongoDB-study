# MongoDB筆記
## **MongoDB特色**
1. 集合導向(collection orented)

意即資料被分組儲存在數據集(collection)中，數據集類似 RDB 的 Table ，不同的是它不需要定義模式(schema)。

1. 模式自由(schema-free)

對於儲存在MongoDB中的數據，我們可以不需要知道它的資料結構，例如{"name":"Sill"},{"age":25}，可以被存在同一個數據集之中。

1. 文檔型資料庫

意即我們儲存的數據是{key：value}的形式，鍵是字符串，值可以是數據類型集合裡的任意類型，包含數組及文檔。這個格式被稱為BSON，"Binary Serialized dOcument Notation"


## **MongoDB適用場景**
1. 網站數據：MongoDB非常適合實時的插入，更新與查詢，並具備網站實時數據存儲所需的複製及高度伸縮性。
2. 緩存:由於性能很高,MongoDB 也適合作為信息基礎設施的緩存層。在系統重啟之後,由 MongoDB 搭建的持久化緩存層可以避免下層的數據源過載。
3. 大尺寸,低價值的數據:使用傳統的 RDB 儲存一些數據時可能會比較昂貴,在此之前,很多時候程序員往往會選擇傳統的文件進行儲存。
4. 高伸縮性的場景:MongoDB 非常適合由數十或數百台服務器組成的數據庫，同時 MongoDB 已經包含對 MapReduce 引擎的內建支持。
5. 用於對象及 JSON 數據的儲存: MongoDB 的 BSON 數據格式非常適合文檔化格式的儲存及查詢。


## **MongoDB數據邏輯結構**

MongoDB 的邏輯結構是一種層次結構。主要由:

  文檔(document)、集合(collection)、資料庫(database)這三部分組成的。邏輯結構是面向使用者的,使用者使用 MongoDB 開發應用程序使用的就是邏輯結構。
1. MongoDB 的文檔(document),相當於 RDB 中的一行記錄(Row)。
2. 文檔最大不能超過16MB的限制，這是為了避免太過頻繁的訪問文件系統或是查詢佔用太大記憶體。
3. 多個文檔組成一個集合(collection),相當於關係數據庫的表(Table)。
4. 多個集合(collection),邏輯上組織在一起,就是數據庫(database)。
5. 一個 MongoDB 實體支持多個數據庫(databases)。

****
## **_id key**
1. 是Mongodb獨有的產物
2. MongoDB 集合中的每個文檔(document)都有一個默認的主鍵_id,這個主鍵名稱是固定的,它可以是 MongoDB 支持的任何數據類型,默認是 ObjectId。
3. 在關係數據庫 schema 設計中,主鍵大多是數值型的,比如常用的 int 和 long,並且更通常的是主鍵的取值由數據庫自增獲得,這種主鍵數值的有序性有時也表明了某種邏輯。反觀 MongoDB,它在設計之初就定位於分散式儲存系統,所以它不支持自增主鍵。
4. 如果你有資料值是不會重複的話，就把他塞給 _id 吧。


## **MongoDB 查詢**
1. MongoDB 查詢返回一個游標位置(cursor)，可以使用迭代方式來遍歷。
2. 在 MongoDB Shell 中，也可以把游標當作數組來使用
3. db.collection.find()，注意set大小，可能造成OOM
4. db.collection.findOne()
5. db.collection.find().limit(n)


## **MongoDB 修改**
1. db.collection.update({"city":"Taipei country"},{$set:{"city":"New Taipei City"}})
2. 如果沒有這個欄位，則會自動新增上去，注意如果新增資料太大，可能造成relocate問題。
3. 如果在設計資料庫時已經限定欄位新增的數量，例如每人最多紀錄10組 E-mail ，可以先將空間留好，之後以set方式修改，減少資料遷移。


## **MongoDB 刪除**
1. db.collection.remove({"name":"Billy"})
2. MongoDB刪除整個集合時有較高效率，設計時可以考慮依時間（天，月，年），不需要時整個集合做刪除，減少遍歷整個集合尋找刪除的負載。


## **Mongodb 高級查詢**
1. 1.文檔導向的 NoSQL 數據庫主要解決的問題不是高性能的並發讀寫，而是保證海量數據儲存的同時，具有良好的查詢性能。
2. MongoDB 最大的特點是查詢語言非常強大，幾乎可以完全實現關聯式資料庫的所有查詢功能，也可以直接對資料建立索引，因此常被用來替代傳統
3. 關聯式資料庫來實現不是太複雜的 Web 應用，有效提昇查詢的效率。
4. 條件操作符
  - db.collection.find（{“field”：{$ gt：value}}）; //大於：field > 值
  - db.collection.find（{“field”：{$ lt：value}}）; //小於：field < value
  - db.collection.find（{“field”：{$ gte：value}}）; //大於等於：field >= value
  - db.collection.find（{“field”：{$ lte：value}}）; //小於等於：field <= value
  - $all
    這個操作符跟 SQL 語法的 in 類似，但不同的是，in 只需滿足（）內的某一個值即可，而 $all 所有必須滿足[]內的所有值，例如：
    db.users.find（{age：{$ all：[6，8]}}）;
      可以查詢出{name：'David',age：[6,8,9]}
      但查詢不出{name：'David',age：[6,7,9]}


## **store procedure**
1. MongoDB 同樣支持 store procedure ，關於 store procedure 你需要知道的第一件事就是它是用 javascript 來寫的。
2. MongoDB store procedure 是儲存在 db.system.js 表中。
3. 我們簡單定義一個函式如下：
    function addNumbers( x , y ) {
        return x + y;
    }
  將函式轉換為 MongoDB store procedure 
    > db.system.js.save({_id:"addNumbers", value:function(x, y){ return x + y; }});
  store procedure 可以被查看，修改和刪除。
    > db.system.js.find()
    { "_id" : "addNumbers", "value" : function cf__1__f_(x, y) { return x + y; }}
  要調用也很簡單
    > db.eval('addNumbers(3, 4.2)');
    7.2


## **固定集合(capped collection)**

MongoDB 的特色之一是可以創建固定集合，固定集合是大小固定的集合，當集合滿了的時候，最新的資料插入時，會將最舊的資料刪除，非常適合頻繁插入、檢索及刪除的對象。
固定集合有如下特點:

  1. 固定大小; 固定集合必須事先創建,並設置大小。
  2. 固定集合禁止執行導致文檔增大的動作，因此可以避免移動文檔及管理文檔新位置的開銷。
  3. 固定集合在插入文檔時會自動刪除最舊的文檔，因此無須在應用程式中實現刪除功能。
****  4. 固定集合可以 insert 和 update 操作;不能 delete 操作，只能用 drop()方法刪除整個 Collection。
  5. 默認基於 Insert 的次序排序。如果查詢時沒有排序,則總是按照 insert 的順序返回，可以節省建立索引的開銷。
  6. FIFO，如果超過了 Collection 的限定大小，則用 FIFO 算法，新記錄將替代最先 insert 的記錄。
  7. 無法對固定集合做分片動作。
  8. 可以在創建固定集合時指定 Collection 中能夠存放的最大文檔數。但這時也要指定 size,因為總是先檢查 size 後檢查 maxRowNumber。
  9. 可以使用 validate()查看一個 collection已經使用了多少空間,從而決定 size 設為多大。
    db.createCollection("mycoll", {capped:true, size:100000, max:100});
    db.mycoll.validate();
## **GridFS**

GridFS 是一種將大型文件儲存在 MongoDB 數據庫中的文件規範。

1. 為何需要使用 GridFS
  由於 MongoDB 中 BSON 對象大小是有限制的,所以 GridFS 規範提供了一種機制,可以將一個大文件分割成為多個較小的文檔,這樣的機制允許我們有效的保存大文件對象,特別對於那些巨大的文件,比如影片、高清圖片等。 


## **MapReduce**


## **聚合(aggregation)**

投射(projecting)和分組(grouping)會改變工作集導致index無法使用,盡量放在filtering後面,能減少處理的工作集,也能使用索引。


## **數據導出 mongoexport**


## **數據導入 mongoimport**


## **數據備份 mongodump**


## **數據恢復 mongorestore**


## **訪問控制**

**MongoDB權限控制系統簡介**
        對於數據庫的管理，一般DBA都不會給開發過大的權限，避免如開發建立索引不加[backgroud:true]導致線上操作巨卡、誤刪除業務庫或集合數據、對集合每個字段添加單列索引導致容量急劇膨脹，還有各種突破認知範圍的誤操作。
        由於MongoDB早期版本自身對權限控制極其簡單粗暴，一般DBA都是授予開發最高權限。隨著MongoDB3.X版本的發布，在權限控制這塊算是比較完善了，下面主要介紹MongoDB權限控制系統。
要想理解清楚MongoDB的權限必須先了解如下一些關鍵字.

- user

即用戶，用於提供客戶端連接MongoDB的認證賬戶。


- role

即角色，數據權限的集合，創建用戶的時候必須要指定對應的角色，否則用戶無法操作數據庫。


- resource

即資源，包括database或collection也可以是database和collection的組合。例如：{ db: <database>, 
collection: <collection> }，當然你也可能看到一種特殊的資源：{“cluster” : true}，它其實表示的是全局資源。


- actions

即權限操作，”actions” 定義了”user”能夠對 “resource document”執行的操作。例如，增刪改查包括如下操作：find、insert、remove、update。


- privilege

即權限，”privilege” 是一組”resource” 和 “actions” 的組合。


- authenticationDatabase

認證庫，即創建角色或用戶時所在的庫。例如，在admin下創建了MongoDB用戶那麼登陸的時候需要指定認證庫。

**MongoDB用戶的角色說明**
角色是什麼？就是對某一資源的權限的“集合”。在MongoDB中有兩種角色，一種是內置的角色，一種是自定義的角色。下面先說說內置的角色，可滿足大部分用戶的需求。

第一種：內置角色介紹


- read

數據庫的只讀權限，包括：
aggregate，checkShardingIndex，cloneCollectionAsCapped，collStats，count，dataSize，dbHash，dbStats，distinct，filemd5，mapReduce (inline output only)，text (beta feature)geoNear，geoSearch，geoWalk，group。


- readWrite

數據庫的讀寫權限，包含read角色的所有權限，包括：
cloneCollection (as the target database.)，convertToCapped，create (and to create collections implicitly.)，renameCollection (within the same database.)findAndModify，mapReduce (output to a collection.)，drop()，dropIndexes，emptycapped，ensureIndex( )。


- dbAdmin

數據庫的管理權限，包括：
clean，collMod，collStats，compact，convertToCappe，create，db.createCollection()，dbStats,drop()，dropIndexes
ensureIndex()，indexStats，profile，reIndex，renameCollection (within a single database.)，validate。


- userAdmin

數據庫的用戶管理權限。


- clusterAdmin

集群管理權限(副本集、分片、主從等相關管理)。


- readAnyDatabase

任何數據庫的只讀權限，和read相似，但它是全局的。


- readWriteAnyDatabase

任何數據庫的讀寫權限，和readWrite相似，但它是全局的。


- userAdminAnyDatabase

任何數據庫用戶的管理權限，和userAdmin相似，但它是全局的。


- dbAdminAnyDatabase

任何數據庫的管理權限，dbAdmin相似，但它是全局的。


- backup、restore

數據的備份和恢復權限。


- root

超級用戶權限，只能針對admin庫。

第二種：自定義角色

        MongoDB內置角色一般來說都是夠用的，但是當內置角色不滿足需求時就可以自定義角色了。其實自定義角色也很簡單，語法如下：

    use admin #進入admin數據庫;
    db.createRole(
      {
        role:<role_name>, #定義角色的名稱;
        privileges:[ #權限集;
          {resource:{db:<db_name>,collection:<coll_name>}, #定義這個角色要控制的庫和集合,集合為""時表示全部;
           actions:[<action_name>]} #定義對這個庫或集合可進行的權限操作,這裡是一個數組;
          ],
        Roles:[{role:<role_name>,db:<db_name>}] #是否繼承其他的角色,如果指定了其他角色那麼新創建的角色自動繼承對應其他角色的所有權限,該參數必須顯示指定;
      }
    )

如下範例：

    use admin
    db.createRole(
           { role:"northwindOwner",
           privileges: [
                      {
                    resource: {
                            db:"northwind",
                            collection:""
                    },
                    actions: [ "find", "insert", "remove","update" ]
                       }
                      ],
             roles: [ ]
           }
    )

上述語句在admin庫裡創建了一個名為northwindOwner的角色，該角色具有對數據庫northwind下的所有集合進行 find、insert、remove、update的操作的權限。角色創建完畢後 MongoDB會在系統庫admin 下創建一個系統collection名叫system.roles 裡面存儲的即是角色相關的信息。可以使用如下語句進行查看：

    db.system.roles.find();

操作角色（以下操作都基於admin庫）
查看角色

    db.getRole("northwindOwner",{showPrivileges:true})

角色繼承

    # 設置test角色繼承northwindOwner;
    db.grantRolesToRole("test",["northwindOwner"])
    # 基於northwindOwner角色移除test角色;
    db.revokeRolesFromRole("test",["northwindOwner"])

角色授權

    db.grantPrivilegesToRole(
           { role:"northwindOwner",
           privileges: [
                      {
                    resource: {
                            db:"northwind",
                            collection:""
                    },
                    actions: ["createCollection", "dropCollection","convertToCapped"]
                       }
                      ],
             roles: [ ]
           }
    )

角色移權

    db.revokePrivilegesFromRole(
           { role:"northwindOwner",
           privileges: [
                      {
                    resource: {
                            db:"northwind",
                            collection:""
                    },
                    actions: ["createCollection", "dropCollection","convertToCapped"]
                       }
                      ],
             roles: [ ]
           }
    )

刪除角色

    db.dropRole("northwindOwner")

**MongoDB認證授權說明**


- MongoDB单实例认证

         MongoDB安裝時不添加任何參數，默認是沒有權限驗證的，登錄的用戶可以對數據庫任意操作而且可以遠程訪問數據庫，如果需要開啟MongoDB身份驗證，那麼需要在命令行啟動MongoDB時加上–auth參數啟動（或配置文件中使用security.authorization: enabled），這樣當MongoDB啟動後再次登錄就需要用戶和密碼進行認證了。
        在剛安裝完畢的時候，MongoDB都默認有一個admin數據庫，此時admin數據庫是空的，沒有記錄權限相關的信息。當admin.system.users一個用戶都沒有時，如果mongod啟動時添加了–auth參數，那麼就尷尬了。
        MongoDB的訪問分為認證和授權，所以使用–auth參數開啟認證後，雖然還是可以不進行認證直接進入數據庫，但是不會有任何的權限進行任何操作，當執行命令時會報如下錯誤：

    not authorized on admin to execute command

        由此看出，一旦開啟認證後授權就會生效了。如果登錄時沒進行用戶認證，那麼就沒有任何權限執行任何命令。此時有兩種辦法，第一就是關掉MongoDB，先添加完用戶再開啟用戶認證；第二種就是MongoDB還提供了一個參數，就是在開啟認證的情況下允許本地連接MongoDB，就是使用localhost連接，其餘一律不接受。此時就可以把這兩個參數同時使用上，即開啟認證時又沒有添加任何用戶的情況下允許本地無認證連接進行數據庫操作。

    security:
      authorization: enabled
    setParameter:
      enableLocalhostAuthBypass: true

注意：這個參數的使用有如下幾條限制。
1）這個參數只能在開啟認證的情況下，並且沒有任何用戶的情況下才生效，否則無效。
2）滿足第一個條件時，只允許本地以localhost的方式進行連接MongoDB，否則無效。
3）滿足第一個條件時，當進入到MongoDB後如果創建了一個用戶後，此參數將失效，就是本地無法登陸了。
4）以此方式創建的用戶必須是admin庫的，同時必須能夠具備創建其他用戶的權限。
        在MongoDB授權部分，其中admin數據庫中的用戶名可以管理所有數據庫，其他數據庫中的用戶只能管理其所在的數據庫。在2.4之前版本中，用戶的權限分為只讀和擁有所有權限。 2.4版本後的權限管理主要分為：數據庫的操作權限、數據庫用戶的管理權限、集群的管理權限，建議由超級用戶在admin數據庫中管理這些用戶。不過依然兼容2.4版本之前的用戶管理方法。
        

- MongoDB複製集認證

        上面說的都是針對單實例MongoDB，如果使用MongoDB複製集的話，那麼情況就有一點點變化了。如果在複製集機制下開啟了–auth認證，那麼此時MongoDB複製集狀態就會變成不健康狀態，這就需要另外一個認證方式了“KeyFile”。簡單來說“KeyFile”就是用在複製集群間開啟認證的情況下需要的另外一種認證方式，用來驗證集群間身份的。在複製集模式下，開啟KeyFile之後就需要再開啟–auth了，因為開啟KeyFIle後就會自動開啟認證功能。
        開啟KeyFile也非常簡單，需要在每個節點主機上創建一個文件，然後給一些字符串，但是複制集各個節點上的字符串必須相同，才能進行身份驗證。如下所示，手動生成一個KeyFile：

    $ echo "this is a keyfile" > /usr/local/mongodb/.KeyFile

        生成KeyFile時需要注意，不用在意空格、tab以及換行符的，KeyFile都會自動去掉的。
KeyFile注意事項：
1）內容：base64編碼，所以不能超出這個範圍[a-zA-Z+/]
2）長度：1000bytes
3）權限：chmod 600 KeyFile
KeyFile生成好之後，就可以同步到各個節點間了，然後在各個節點的配置文件中把KeyFile文件指定進來即可。

    security:
       keyFile: /usr/local/mongodb/.KeyFile

然後將權限設為600即可

    $ chmod 600 /usr/local/mongodb/.KeyFile

        前面已經提到，開啟了KeyFile後就不需要添加setParameter.enableLocalhostAuthBypass: true參數了，默認會開啟認證的，然後重啟複製集各個節點就完成認證了。另外在復制集模式下，在整個認證配置完成前不要創建任何用戶，當整個認證好了之後，下面就可以創建用戶了。

**MongoDB創建和刪除用戶**

- 不開啟認證，進行用戶創建

首先，我們需要在不認證的情況下在admin庫中創建一個高權限用戶，也就是需要在不認證情況下啟動MongoDB。

    $ mongo admin
    > db.createUser({user:'root',
                     pwd:'12345678',
                     roles:[{role:'root', db:'admin'}]
                     })

切換到admin下，查看剛才創建的用戶：

    > show users
    {
     "_id" : "root.admin",
     "user" : "root",
     "db" : "admin",
     "roles" : [
     {
     "role" : "root",
     "db" : "admin"
     }
     ]
    }
- 開啟認證，進行用戶認證

使用–auth參數啟動MongoDB或在配置文件中添加security.authorization: enabled參數，然後再開啟mongod。

    security:
      authorization: enabled

進行認證，需要注意的是我們在哪個庫創建的用戶就需要在那個庫進行認證。

    mongo -u root -p 12345678 --authenticationDatabase admin

或

    > use admin
    > db.auth("root","12345678")

MongoDB 也支持為某個特定的數據庫來設置用戶,如我們為 test 數據庫設一個用戶 user_test

    > use test
    > db.createUser({user:'user_test',
                     pwd:'abcdefg',
                     roles:[{role:'dbOwner', db:'test'}]
                     })
- 查看用戶
    > use admin
    > db.getUser("root")
- 用戶密碼修改
    > db.changeUserPassword('root','123')
- 刪除用戶
    > db.dropUser('test')
    true

還可以使用db.dropAllUsers()刪除當前庫的所有用戶，如下：

    > use admin
    switched to db admin
    > db.dropAllUsers()
    1

就可以把admin庫的所有用戶都刪除了。
        在MongoDB中刪除庫和集合並不會關聯刪除對應的角色和用戶。因此如果想徹底刪除對應的業務庫應該先刪除庫及其對應的角色和用戶。如果既想實現精細化權限控制又想簡化用戶管理，原則上建議只給開發創建一個帳戶，並且使用admin做認證庫，這樣可避免清理過期業務庫而導致無法登錄的問題。
 
**綁定 IP 內網地址訪問 MongoDB 服務**

    mongod --bind_ip 192.168.1.103

**設置監聽端口（預設是27017）**

    mongod --bind_ip 192.168.1.103 --port 20000


## **進程控制**

注意不要殺掉內部發起的操作,比如說 replica set 發起的 sync 操作等。
****
    > db.currentOp();
    > db.killOp(1234/*opid*/)



## **index使用設計**

MongoDB 提供了多樣性的索引支持,索引被保存在 system.indexes 中,且默認總是為_id創建索引,它的索引使用基本和 MySQL 等關係型數據庫一樣。
****
**對collection**

  - 沒有使用index就是作full table scan
  - 使用index,mongo會優先使用索引尋找資料,但索引會增加(insert,update,delete)的時間
  - index的值是按照一定順序排序的
1. 基礎索引
  - _id 索引為 MongoDB 自動創建的，並且不可被刪除。
    > db.collection.ensureIndex({age:1})
    > db.collection.getIndexes();
    [
      {
        "name" : "_id_",
        "ns" : "db.collection",
        "key" : {
          "_id" : 1
          },
        "v" : 0
      },
      {
        "_id" : ObjectId("4fb906da0be632163d0839fe"),
        "ns" : "db.collection",
        "key" : {
          "age" : 1
          },
        "name" : "age_1",
        "v" : 0
      }
    ]
2. 文檔索引
  MongoDB 也可將文檔設定成索引。
    > db.factories.insert( { name: "www", addr: { city: "Taipei", dist: "Xinyi" } } );
    //在 addr 欄位上創建索引。
    > db.factories.ensureIndex( { addr : 1 } );
    //下面這個查詢將可以使用我們剛才所建立的索引。
    > db.factories.find( { addr: { city: "Taipei", dist: "Xinyi" } } );
    //但是下面這個查詢將用不到剛才所建立的索引，因為順序不同。
    > db.factories.find( { addr: { dist: "Xinyi" , city: "Taipei"} } );
3. 複合索引
  - 複合索引,可以配合查詢來設計,例如先按照age排列,再依照名字反向排列。
  - 當創建複合索引時,字段後面的 1 表示升序,-1 表示降序,是用 1 還是用-1 主要是跟排序的時候或指定範圍內查詢的時候有關的。
  - 同時相互反轉的索引是等價的,{ "addr.city" : 1 , "addr.disc" : -1 } 與 { "addr.city" : -1 , "addr.disc" : 1 } 是一樣的。
4. 隱式索引
  - 當我們創建{ "addr.city" : 1 , "addr.disc" : 1 }索引時,實際上我們也可以使用{ "addr.city" : 1 }
    > db.factories.ensureIndex( { "addr.city" : 1, "addr.disc" : 1 } );
    // 下面的查詢都可以使用這個索引
    > db.factories.find( { "addr.city" : "Taipei", "addr.disc" : "Xinyi" } );
    > db.factories.find( { "addr.city" : "Taipei" } );
    > db.factories.find().sort( { "addr.city" : 1, "addr.disc" : 1 } );
    > db.factories.find().sort( { "addr.city" : 1 } )
5. 唯一索引
  - _id 是唯一索引。
    > db.collection.insert({firstname: "hughe", lastname: "chen"});
    > db.collection.insert({firstname: "hughe", lastname: "chen"});
    > db.collection.ensureIndex({firstname: 1, lastname: 1}, {unique: true});
    E11000 duplicate key error index: db.collection.$firstname_1_lastname_1 dup key: { : "hughe", : "chen" }
    // 可以看到,當建立唯一索引時,系統報了“表裡有重複值”的錯,具體原因就是因為表中有2條一模一樣的數據,所以建立不了唯一索引。
6. 稀疏索引(sparse)
  - 稀疏索引可以與唯一索引搭配使用
  - 設定為稀疏索引的鍵,當鍵值不存在時,該文檔就不會被檢索到
    db.collection.createIndex( { "email": 1 }, { sparse: true } )
7. TTL索引(time-to-live index)
  - 為每一個文檔設置超時時間,超過後文檔會被刪除
  - 不能是複合索引,但可以被用來優化查詢及排序
8. 文本索引(text index)
  - 可以先用查詢條件將搜索範圍變小,形成局部文本索引({"date" : 1,"post" : "text"})
9. 覆蓋索引(covered inedex)
  - 當用戶請求的資料被索引所包含時,可以認為這個索引覆蓋了本次的查詢,應該使用覆蓋查詢,可以有效減少工作集大小。
10. 強制使用索引
    > db.collection.find({age:{$lt:30}}).hint({name:1, age:1}).explain()
11. 刪除索引
    // 刪除 collection 表中所有索引
    > db.collection.dropIndexes()
    // 刪除 collection 表中的 firstname 索引
    db.collection.dropIndex({firstname: 1})
12. 低效的操作符
  - 索引無法生效-->想辦法先減少操作的工作集。

**範圍**

1. 設計多個字段的索引時,應該將精確匹配的字段放在前{"x":foo},範圍匹配的字段放在後{"y":{"$gte":3,"$lte":10}},這樣就可以只在精確匹配的工作集內進行範圍查詢
2. 應該把索引建在基數較高的鍵值上（能有效減少工作集的大小）,或至少把基數較高的鍵值,放在複合索引的前面

**何時不該使用索引**

1. 使用索引查找方式：會進行兩次查找,第一次先查找索引條目,第二次再依據索引查找文檔內容
2. 如果使用索引查找後返回的工作集佔整體文檔的30%以上,就要與全表掃描作比較,確認查詢速度
## **優化器 Profile**
- 有兩種方式可以控制 Profiling 的開關和級別，第一種是直接在啟動參數里直接進行設置，啟動 MongoDB 時加上 --profile=級別 即可。
- 也可以在客戶端調用 db.setProfilingLevel(級別) 命令來配置，Profiler 信息保存在 system.profile 中。我們可以通過 db.getProfilingLevel() 命令來獲取當前的 Profile 級別。
- profile 的級別可以取 0,1,2 三個值,他們表示的意義如下:
  0 – 不開啟
  1 – 記錄慢命令 (默認為>100ms)
  2 – 記錄所有命令
    // 設定Profile Level
    > db.setProfilingLevel(2);
    { "was" : 0, "slowms" : 100, "ok" : 1 }
    
    // 設定慢命令的默認時間(ms)
    db.setProfilingLevel( level , slowms )
    db.setProfilingLevel( 1 , 10 );
- 查詢 Profiling 命令
- MongoDB Profile 記錄是直接存在系統 db 裡的,記錄位置system.profile ,所以,我們只要查詢這個 Collection 的記錄就可以獲取到我們的 Profile 記錄了
    > db.system.profile.find().sort({$natural:-1}).limit(1)
    { "ts" : ISODate("2012-05-20T16:50:36.321Z"), "info" : "query test.system.profile reslen:1219
    nscanned:8 \nquery: { query: {}, orderby: { $natural: -1.0 } } nreturned:8 bytes:1203", "millis" :
    0 }


## **性能優化**
- 如果 nscanned(掃描的記錄數)遠大於 nreturned(返回結果的記錄數)的話,那麼我們就要考慮通過加索引來優化記錄定位。
- reslen 如果過大,那麼說明我們返回的結果集太大了,這時請查看 find 函數的第二個參數是否只寫上了你需要的屬性名。
- 對於創建索引的建議是:如果很少讀,那麼盡量不要添加索引,因為索引越多,寫操作會越慢。如果讀量很大,那麼創建索引還是比較划算的。
  1. 讀取：正確使用索引，盡可能將返回的資訊放在單個文檔中回傳
  2. 寫入：減少索引數量，盡可能提高更新效率
- 優化文檔增長
  1. 更新數據時需要注意，更新是否會導致文檔體積成長以及成長速度，如果可預期，可以先預留足夠成長空間，避免文檔移動，可以提高寫入速度
  2. 如果要對文檔進行手動填充，可以在創建文檔時創建一個佔空間比較大的字段，文件創建成功之後再移除
  3. 需要增長的字段放在文檔最後,這樣可以稍微提高效能(mongoDB不用重寫後面的字段)
- 限定返回結果數量
  1. 使用 limit()限定返回結果集的大小,可以減少 database server 的資源消耗,可以減少網路傳輸數據量。
- 只查詢使用到的字段,只查詢使用到的字段 , 而不查詢所有字段。
- 採用固定集合 capped collection
- Hint

一般情況下 MongoDB query optimizer 都工作良好，但有些情況下使用 hint()可以提高操作效率。 Hint 可以強制要求查詢操作使用某個索引。例如，如果要查詢多個字段的值，如果在其中一個字段上有索引，可以使用 Hint。

## **性能監控**
- mongosniff
  此工具可以從底層監控到底有哪些命令發送給了 MongoDB 去執行,從中就可以進行分析。
    ＄ bin/mongosniff --source NET lo <27017> <port2>
- mongostat
  此工具可以快速的查看某組運行中的 MongoDB 實例的統計訊息。
    ＄ bin/mongostat --port 27017 --discover
- db.serverStatus()
- db.stats()


## **資料庫的選擇-RDB、NOSQL或是兩者**
- 要儲存的數據是什麼樣的
  是 key:value 的形式或是 Row 還是文檔的類型。
- 當前使用的資料庫型態
  目前儲存資料是使用 RDB 還是  NOSQL ，如果轉移需要花費的人力及物力。
- 確保資料庫事務的準確性有多重要
  NOSQL 資料庫在 ACID 上還是沒辦法媲美傳統型 RDB 資料庫。
- 數據庫的速度有多重要
  如果速度對你非常重要，NOSQL 資料庫可能是你更好的選擇。
- 資料庫不可用的情形
  對客戶來說，資料庫響應太慢也是一種不可用， NOSQL 透過分片及副本集的概念來提供高可用性。
- 資料庫將被如何使用
  資料是寫入還是讀取居多，也可以據此來設計數據區隔，將主要讀取資料及主要寫入資料分開，做讀寫分離。
- 是否該將資料分開，分別存放在 RDB 及 NOSQL中，充分利用各自優點。
****## **數據庫和集合的設計**
1. 相近模式的文檔應該放在同一個集合中，如果會對文檔作聚合動作，應該讓它們位於同一個集合中
2. 可以讓不同的資料庫位於不同的磁碟分卷，但要注意 MongoDB 資料庫間數據不允許直接移轉
- 固定集合 Capped Collections
  MongoDB 的特色之一是可以創建固定集合，固定集合是大小固定的集合，當集合滿了的時候，最新的資料插入時，會將最舊的資料刪除，非常適合頻繁插入、檢索及刪除的對象。
  固定集合有如下特點:
  1. 固定大小; 固定集合必須事先創建,並設置大小:
    > db.createCollection("mycoll", {capped:true, size:100000})
  2. 固定集合可以 insert 和 update 操作;不能 delete 操作，只能用 drop()方法刪除整個 Collection。
  3.  默認基於 Insert 的次序排序。如果查詢時沒有排序,則總是按照 insert 的順序返回，可以節省建立索引的開銷。
  4.  FIFO，如果超過了 Collection 的限定大小，則用 FIFO 算法，新記錄將替代最先 insert 的記錄。
  5. 固定集合禁止執行導致文檔增大的動作，因此可以避免移動文檔及管理文檔新位置的開銷。
  6. 固定集合在插入文檔時會自動刪除最舊的文檔，因此無須在應用程式中實現刪除功能。


## **應用程序設計**
- **Embedded Documents**
  子文檔較大，數據一次查詢就能獲得，讀取資料速度較快。
- **Reference Documents**
  子文檔較小，數據通常不包含在結果中，需要做二次讀取的動作，寫入資料速度較快。
- 正規化
  適合用於一對多關係或是多對多關係，例如可能有多個客戶喜歡多間商店，這時就適合使用正規化，將喜歡的商店資料單獨列為一個集合，然後在顧客集合中引用商店的 _id ，這樣可以減少相同的對象反覆大量的被儲存。
- 反正規化
  適合使用於一對一關係或是一對少的關係，例如客戶的工作或是家庭資料，可能有客戶工作或是家庭資料重複，但是數量很少且變動不頻繁，這時就適合使用反正規化，將工作及家庭資料內嵌在客戶集合中。
- 寫入原子操作
  在 MongoDB 中，寫入操作在文檔級別是原子性的，不能有多個進程同時更新一個文檔或是集合，這意味著對內嵌文檔的修改也是原子性的，但是修改引用文檔就無法保證原子性。
- 考慮文檔增大
  Mongodb 在寫入文檔時，已經提供了一些留白，以便支持之後更新操作導致的檔案增大，但是如果更新導致的檔案增大超過分配給它的空間，就會造成資料遷移，MongoDB 的作法是分配一塊新的空間給更新後的文檔，再將舊空間資料刪除，這會造成磁盤碎片的產生。
- 使用大型集合或是大量集合
  使用大量集合並不會對 MongoDB 性能造成嚴重影響，但是一個集合包含大量數據會嚴重影響性能。
- 確定數據的生命週期
  在設計資料庫時，除了要永久保存的資料外，應該對集合設定 TTL(Time To Live) ，在只需要最新的文檔時還可設定固定集合來限制集合大小。
- 考慮數據可用性及性能
  數據可用性指的是滿足使用者的使用需求，同時資料庫也要以能夠接受的速度提供資料，在複雜的情況下可用性及性能必須反覆來回的考量，以取得平衡點。


刪除舊數據
TTL集合，固定集合，定時刪除舊集合

不適用mongo的場景

1. mongo不支持交易（transaction）
2. 在多個不同維度上對不同類型的數據進行連結

隱藏節點
var config = rs.config()
config.members[2].hidden = true
config.members[2].priority = 0
rs.reconfig(config)
可以將副本集成員設為隱藏成員

標籤(tags)
var config = rs.config()
config.members[0].tags = {"normal" : "A"}
config.members[1].tags = {"normal" : "B"}
config.settings.getLastErrorModes = {"visibleMajority":{"normal" : 2}}
rs.reconfig(config)
db.coll.insert({"count":1000000},{writeConcern:{w:"visibleMajority"}})

1. 每個成員可以擁有多個標籤
2. 可以透過getLastErrorModes字段，創建自定義規則 -> name:{"ket" : number}
3. 可以對讀，寫作不同的標籤設定
4. 所有標籤的值必須是字串型態

將讀取要求發送到備份節點
通常情況下，讀取要求應該透過主節點來進行，但是可以透過設置驅動程式的(read preferences)來配置其他選項，但將讀取請求發送到備份節點通常不是個好主意。
有以下情形，可以考慮將讀取請求發送到備份節點：

1. 對一致性沒有要求，例如故障轉移期間，可以接受陳舊的數據
2. 出於負載的考慮，要使用分散式負載，但這種方式非常危險，很容易導致系統意外過載(骨牌效應，一個伺服器掛點，導致其他伺服器負載超過100%，也接著掛點)

淺談mongodb內存
　　本文僅限於mongodb3.0.0（wiredtiger引擎）

一.mongodb記憶體使用
1.熱數據
這一點是SQL和nosql之間的巨大差距，將熱數據存在記憶體相當於自帶cache，若wiredtigercache大小控制合理，此處記憶體性價比相當高。
2.索引
跟熱數據同樣
3.連接所消耗內存
這裡算是與SQL基本相同的一部分

二.監控
1.wiredtigercache
2.mongodb所佔用記憶體
3.mongodb讀取數據與IO數據

三.優化
1.熱數據
上面說此處佔用記憶體為cache，可以通過wiredtigercache進行上限的調節，將命中和大小控制在合理範圍
2.索引
mongodb會將全部索引讀入記憶體中，所以這裡優化相當重要，建議花精力去整理索引，如果可以做到索引恰好覆蓋熱數據為最佳。

四.其他注意事項
1.杜絕依靠重啟來釋放記憶體，這時釋放掉的是熱數據，得不償失
2.可以禁止swap
3.不要將mongodb與其他吃記憶體的服務部署到同一機器上
4.重視對mongodb記憶體監控，避免oom
5.重視oplog，oplog大小應定期調整，若oplog過小，對熱數據部分將是災難
6.關閉NUMA
7.安裝在x64機器上
8.根據wiredtigercache，索引，連接，來調整內存
9.有條件最好SSD

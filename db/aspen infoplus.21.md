

# 连接



# 取数据

## 1. GETRECLIST

返回特定标准记录的`id`、`name`列表

```c++
void GETRECLIST(
 	long *,           // 返回记录id，0表示记录已读完，-1表示参数错误
    long,             // 取哪个范围的记录
    int,              // 0表示以id顺序返回，否则以字母顺序返回
    unsigned char [], // 识别用户权限资格
    int,              // 上一个参数的长度
    int,              // 0:不需要有查看权限，非0则必须有查看权限
    int,              // 0:不需要有修改权限，非0则必须要有
    long,             // 列表中包含最大记录数，也就是一次取多少条记录
                      // 如：一共3000条记录
                      //    该参数为1，则要调3000次取完记录
                      //    该参数为100，则要调30次取完记录
    long [],          // 返回记录id的数组，长度不能小于上一个参数
    char [],          // 表示上一个数组中每个id的可用性，长度与上个数组相等
    NAMEARR [],       // 与id数组中对应的名称，长度与id数组相等
    short [],         // 与名称数组中对应的名称长度，数组长度与id数组相等
    long*             // id数组中的实际记录数，若与第9行参数相等，则要再调一次GETRECLIST，直到第1个参数=0
)
```

```c++
const int L = 10000; // 每次取1w条记录
long last_rec = 0L, rec_type = RTYPANYRECORD, num_recs;
unsigned char g_list[8];
long rec_ids[L];
char rec_usabs[L];
NAMEARR rec_nms[L];
short nms_sz[L];
int count = 0;

while (true) {
    GETRECLIST(
        &last_rec, rec_type, 1, g_list, 8, 0, 0, L, 
        rec_ids, rec_usabs, rec_nms, nms_sz, &num_recs
    );
    if (last_rec == 0L) break;

    for (int i = 0; i < num_recs; i++) {
        cout << "id: " << rec_ids[i] << ", name: " << rec_nms[i] << ", usabs: " << rec_usabs[i] << endl;
        count++;
    }
}

cout << "total: " << count << endl;
```

## 2. GETNAMDB

根据记录`id`取得名称

```c++
void GETNAMDB(
    long *,   // id，由 GETRECLIST 得到
    NAMEARR,  // 返回的名称
    short *   // 名称长度
)
```

## 3. FLDFT

遍历record里的所有ft

```c++
void FLDFT(
	long,   // record_id
    long,   // 0: 表示只返回field数量；>=1且<=第4个参数，则返回ftid(下个参数)
    long*,  // 返回的 ftid
    long *  // 该record下有多少个ft
)
```

```rust
// 例子
let mut ftid: c_long = 0;
let mut num: c_long = 0;
unsafe { FLDFT(id, 0, &mut ftid, &mut num) } // 参数2=0，表示只取ft数量
for i in 1..=num { // 循环取ft id
    unsafe { FLDFT(id, i, &mut ftid, &mut num) }
    println!("id: {}, num: {}", ftid, num)
}
```

## 4. GETFTDB

取ft名称

```c++
void GETFTDB (
    long,    // rec_id
    long,    // ft_id
    FTNMARR, // 返回ft名称
    short *  // 名称长度
)
```

```rust
let mut name = ['\0' as c_uchar; 30];
let mut len: c_short = 0;
unsafe { GETFTDB(rec_id, ft_id, name.as_mut_ptr(), &mut len) }

println!("name: {}", String::from_utf8_lossy(name[0..(len as usize)].as_ref()))
```

## 5. DECODFT

根据名取ft

## 6. RHIS21DATA

读取多个历史字段的多次出现，按时间顺序读取（较早的出现在先），并将读取的值存储到多个数据数组中。需要打开归档功能。接受扩展的微秒时间戳作为开始和结束时间。

```
mode是请求的类型：
- H21_GET_TIMES - 返回timeold和timenew之间的插值。时间间隔是（newtime-oldtime）/maxoccs。
- H21_GET_TIMES2 - 返回timeold和timenew之间的内插值。时间间隔是（新时间-旧时间）/（maxoccs-1）。
- H21_GET_BEST_FIT - 对两个端点进行内插，将timeold和timenew之间的时间划分为区间，对每个区间返回该区间的最低或最高值。
- H21_GET_ACTUALS - 返回存储在历史中的实际值。
- H21_GET_ACTUALS_AND_CURRENT - 返回历史上存储的实际值，以及存储在记录固定区域的当前值。
- H21_GET_TIMES_EXTENDED - 与H21_GET_TIMES相似，但如果设置了步骤参数，并且Timenew比历史上的最新值要新，那么在最新值和Timenew之间的所有时间间隔中，都会重复该最新值。
- H21_GET_TIMES2_EXTENDED - 与H21_GET_TIMES2相似，只是如果设置了步骤参数，并且Timenew比历史上的最新值要新，那么这个最新值将在最新值和Timenew之间的所有时间段内重复出现。

step是离散标志。如果它是TRUE，这个程序会返回额外的值，这样返回的数据集就可以用来创建阶梯图。

outsiders如果非零，则获得前后出现的情况。
如果outsiders被设置为非零，模式必须被设置为H21_GETACTUALS。
换句话说，只有当模式被设置为H21_GET_ACTUALS时，outsiders中的非零值才有效。
如果outsiders非零且模式不是H21_GET_ACTUALS，RHIS21DATA将返回错误代码DHREADER（-68）以及错误块ERR2字段的值12。

recid，记录的ID

ft是历史重复区（必须有一个有效的出现次数，如1）或重复区计数字段标签（必须有一个出现次数为0）中的字段标签。

timeold是一个扩展的微秒时间戳。 timeold是发生的事件从历史中读取的开始时间。

timenew是一个扩展的微秒时间戳。 timenew是从历史中读取发生的事件的结束时间。

numfts是要从每个发生中读取的字段数。 numfts是fts、datats和ptdatas中数组元素的数量。

fts是待读字段的字段标签数组。每个字段标签的发生号部分必须是0。

datatypes是一个数组，包含要读取的字段的数据类型。

maxoccs是要读取的最大发生数，maxoccs是ptdatas的元素所指向的数据数组中的数组元素数。

keylevels数组保存读取的事件的关键质量等级。这个参数可以设置为NULL，在这种情况下，不返回历史事件的密钥级别。

keytimes数组持有所读事件的关键时间。这个参数可以设置为NULL，在这种情况下，历史事件的关键时间不会被返回。

ptdatas是一个指向数据数组的指针数组，每个要读取的字段都有一个指针。每个数据数组（由其datats元素指定的类型）包含从其字段中读取的数据，在发生的情况下（较早发生的数据优先）。每个数据数组中只有occsok元素被使用。如果一个数据数组的元素是字节数为奇数的字节数组，那么在每个字节数组之后都会跳过一个字节。

occsok 返回读到的事件数量。

ftsok返回读取的字段数，除非有一个错误，否则ftsok返回numfts。

error返回setcim.h include文件中定义的一个错误代码。
```

## 7. FIELDINFO

根据`ft`取`fttype`

# 收藏

```bash
https://blog.csdn.net/weixin_39558804/article/details/112214115     # 安装
https://www.bilibili.com/video/BV15Y41157o5?spm_id_from=333.999.0.0 # 基于C++
```


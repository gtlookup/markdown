```javascript
// 不需安装,创建出来的工程就有,直接import进来就能用
import * as _ from 'lodash';
import _ from 'lodash'; // 或
```

# 对比 linq

| 操作       | Linq     | Lodash                           |
| ---------- | -------- | -------------------------------- |
| 查询(投影) | select   | _.map                            |
| 过滤       | where    | _.filter                         |
| 第一个     | first    | _.find, _.head                   |
| 最后一个   | last     | _.last                           |
| 排序       | order by | _.orderBy                        |
| 分组       | group    | _.groupBy                        |
| 连接       | join     | (官方没有)                       |
| 获取数据   | from     | 每个函数第一个参数/_()/_.chain() |
| 是否存在   | any      | _.some                           |
| 是否全满足 | all      | _.every                          |
| 最大       | max      | _.maxBy                          |

注：在lodash4.0之前，过滤使用的都是select和where，4.0之后统一改成了filter

数组自带的map和filter函数，已经非常强大了，lodash对其功能基础上进行了两点增强:

1.传入的参数不必是函数，可以是字符串（甚至可以不传入参数）

2.可以操作对象

# 常见用法

```javascript
// 取得集合里的一个 value
_.get(_([{a: 'aa', b: 'bb'}]).find({a: 'aa'}), ['b'])
// 取得结果集里的 key 数组和 value 数组
let ds = [{a: 'a1', b: 'b1'},{a: 'a2', b: 'b2'},{a: 'a3', b: 'b3'}];
let cs = _.keys(_.first(ds));
let dt = _(ds).map(o => _.values(o)).value();
// cs: ['a', 'b']
// dt: [['a1','b1'],['a2','b2'],['a3','b3'],['a4','b4']]

// 获取 key 数组
let k = _.keys({a: 'aaa', b: 'bbb'});
// 条件查找 (where)
let v = _.filter([4, 5, 6], n => n % 2 === 0);
// 判断空
_.toString(v) === ''
// 判断是否整数
_.toInteger(s) == s
// 删除数组里元素
_.pullAt(list, 0, 2, 4, 6); // 删除第0 2 4 6元素

// 删除对象的属性
let o = { 'a': 1, 'b': '2', 'c': 3 };
o = _.omit(o, ['a', 'c']); // {b: '2'}

// 分组
_.groupBy(list, o => o.CODE);
// 对象里是否有某 k/v
_.isMatch({ 'a': 1, 'b': 2 }, { 'b': 2 }); // true
// 取集合里一个字段
_.map([{a: 'aa', b: 'bb'}, {a: 'aa2', b: 'bb2'}], 'b');
// ['bb', 'bb2']

// 去重复 单纯值
_.uniq([1,1,2,2])
// 根据obj里的某个属性
_.uniqBy([{id: 1, name: 'a'},{id: 1, name: 'aa'}], function(x) {return x.id})
// obj全相等
_.uniqWith([{id: 1, name: 'a'},{id: 1, name: 'aa'}], _.isEqual)

// 取多个数组里的非重复值
_.xor([2, 1], [2, 3]); // [1,3]
// 编译模板
_.template
// 返回一个区域值
_.slice([1,2,3,4,5], 2, 5); // [3,4,5]
// 返回前n个值
_.take([1, 2, 3], 2); // [1,2]

// 合并多维数组
let cs = [[1], [2,3], [4,5,6]];
// _.concat 将两个数组xy合并到x里并返回
_.reduce(cs, function(x, y) { return _.concat(x, y)}); // [1,2,3,4,5,6]
```



# Left inner join

https://github.com/mtraynham/lodash-joins

```javascript
let left = [
  {id: 1, tp: 'A', cd: '11', a: 'a1', b: 'b1'},
  {id: 2, tp: 'A', cd: '22', a: 'a2', b: 'b2'}
], right = [
  {tp1: 'A', cd1: '11', r: 'r1'},
  {tp1: 'A', cd1: '12', r: 'r2'}
];

_.hashInnerJoin(
	left, function(x) {return x.tp + x.cd}, 
	right, function(x) {return x.tp1 + x.cd1}
);
```

# 加减乘除

```javascript
_.add， _.subtract，_.multiply，_.divide
```


## 使用聚合管道查询数组内符合条件的元素

有以下 mongo 文档：

```json
{
	{_id:1, id:1282490670079431693, reports:{__array:[{type:1, param:{cfgId=1001,name="aaa"}}, {type:2, param:{cfgId=1002,name="bbb"}}]}},
    {_id:2, id:1282490670079431693, reports:{__array:[{type:1, param:{cfgId=1001,name="aaa"}}, {type:2, param:{cfgId=1002,name="bbb"}}]}},
}
```

查询出 `id=1282490670079431693` 且 `reports.__array` 中 `type=1且 param.cfgId=1001` 的结果。

查询语句如下:

```json
db.test.aggregate([
  // 第一步：匹配 id=1282490670079431693 的文档
  {
    $match: {id: NumberLong("1282490670079431693") }
  },
  // 第二步：解构 reports.__array 数组
  {
    $unwind: "$reports.__array"
  },
  // 第三步：筛选 reports.__array 中 type=1 且 param.cfgId=1001 的元素
  {
    $match: {"reports.__array.type": 1, "reports.__array.param.cfgId": 1001}
  },
  // 第四步：重构输出结果
  {
    $project: {id: 1, report: "$reports.__array"}
  }
]);
```

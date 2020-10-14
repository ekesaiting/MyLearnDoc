* 所谓视图，就是一条sql语句，如：

  ```mysql
  # 以下语句创建了一个视图，但它只是一个逻辑对象，与我们执行后面的sql语句没有区别，只是将查询到的结果展示出来，不会保存
  create view v1  as select name  age from student where score > 80;
  # 下面从视图中获取数据 ,他等同于select name from （select name  age from student where score > 80）as v1 where age>18
  select name from v1 where age>18
  #也就是说原始表的改变将影响视图的改变，删除视图不会影响原始表，视图只是一些查询结果，不能修改或删除视图中的数据。
  ```

  
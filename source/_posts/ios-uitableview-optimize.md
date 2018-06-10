---
title: iOS - TableView 的重用机制及优化
date: 2018-05-18 21:41:01
tags: [iOS,UITableView]
categories: [iOS]
---
## iOS-TableView的重用机制
### 什么是Cell的重用？
重用机制，简单的说意思是一行一行cell的复用
### 为什么要重用？
为了做到显示和数据分离，UITableViewCell的实现而且不是为每一个数据项创建一个tableCell，是仅仅创建屏幕可显示最大个数的cell，然后反复使用这些cell，对cell做单独的显示配置，来达到既不影响显示效果，又能充分节省内存的目的；当屏幕滚动出现新Cell的时候，就会调用方法获取新出现的Cell,而有的Cell则会滚动到屏幕的外面
### 如何实现Cell的重用？
通过UITableView的`dequeueReusableCellWithIdentifier `函数实现，从字面理解就是`出列可重用的Cell`，简单来说就是有一个Cell池，里面存放了之前从屏幕滚动消失的Cell
### 重用机制的实现原理
进入UITableView的头文件可以发现：
```objc
@property (nonatomic, readonly) NSArray<UITableViewCell *> *visibleCells;
@property (nonatomic, readonly) NSDictionary<NSString *, UITableViewCell *> * reusableTableCells;
```
`visibleCells`内显示当前显示的cells
`reusableTableCells`保存可重用的cells，可复用的cell使用字典是因为可复用的可能cell不只有一种样式，这里需要字典指定key(也就是reuseIdentifier)来查找是否有可重用样式。

- 执行思路：
tableView显示之初，reusableTableCells为空，假如一个界面显示5个Cell，界面慢慢向上拖动，当cell1完全从屏幕上小时的时，cell6（cell6是新创建的cell,因为reusableTableCells为空）完全展示在界面上时；cell1移入到reusableTableCells中，继续拖动,展示cell7会从reusableTableCells中取出缓存的cell1,以此类推…

### 注意
并非仅仅有拖动超出屏幕的时候才会更新reusableTableCells,`reloadData`和`reloadRowsAtIndex`时也会更新并操作reusableTableCells

## UITableViewCell的性能优化
![](http://p7xd6yrmx.bkt.clouddn.com/uitableviewcell_optimize.png)
## 扩展：UITableView delegate/dataSource方法执行顺序
```Objc
1.//有多少组
-(NSInteger)numberOfSectionsInTableView:(UITableView * )tableView
2.//cell 页眉高度
-(CGFloat)tableView:(UITableView * )tableView heightForHeaderInSection:(NSInteger)section
3.//cell页脚高度
-(CGFloat)tableView:(UITableView * )tableView heightForFooterInSection:(NSInteger)section
4.//每组有多少行
-(NSInteger)tableView:(UITableView * )tableView numberOfRowsInSection:(NSInteger)section
5.//cell高度
-(CGFloat)tableView:(UITableView * )tableView heightForRowAtIndexPath:(NSIndexPath )indexPath
6.//布局UITableviewcell
-(UITableViewCell )tableView:(UITableView * )tableView cellForRowAtIndexPath:(NSIndexPath * )indexPath
```

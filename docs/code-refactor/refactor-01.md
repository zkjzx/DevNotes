### 代码优化

>
> 编程是一门技术，更是一门艺术。
>

本篇文章记录一下平时写代码过程中，值得学习的优雅且高效代码。编程路上，道阻且长，不断学习，不断精进。

------

#### 使用return 关键字提前返回

> 利用 return 关键字，可以提前函数返回，避免定义中间变量。

**普通代码**

```java
public static boolean hasSuper(@NonNull List<UserDO> userList) {
    boolean hasSuper = false;
    for (UserDO user : userList) {
        if (Boolean.TRUE.equals(user.getIsSuper())) {
            hasSuper = true;
            break;
        }
    }
    return hasSuper;
}
```

**精简代码**

```java
public static boolean hasSuper(@NonNull List<UserDO> userList) {
    for (UserDO user : userList) {
        if (Boolean.TRUE.equals(user.getIsSuper())) {
            return true;
        }
    }
    return false;
}
```

#### **使用add 方法判断值是否已存在**

> 使用 Set 的 add 方法的返回值，可以直接知道该值是否已经存在，可以避免调用 contains 方法判断存在。
>
> 思考：为什么不推荐用contains方法判断存在？ 提示：比较一下contains方法和add方法的源码

**普通代码**

```java
/**
* 以下案例是进行用户去重转化操作，需要先调用 contains 方法判断存在，
* 后调用add方法进行添加。
*/
Set<Long> userIdSet = new HashSet<>();
List<UserVO> userVOList = new ArrayList<>();
for (UserDO userDO : userDOList) {  
  if (!userIdSet.contains(userDO.getId())) {
      userIdSet.add(userDO.getId()); 
      userVOList.add(transUser(userDO));   
}}
```

**精简代码**

```java
Set<Long> userIdSet = new HashSet<>();
List<UserVO> userVOList = new ArrayList<>();
for (UserDO userDO : userDOList) { 
   if (userIdSet.add(userDO.getId())) {
           userVOList.add(transUser(userDO));    
 }}
```

#### **避免条件判断**

> 使用已有方法替代不必要的条件判断。阿里巴巴代码规范里提到，多使用卫语句，避免多重嵌套条件判断语句，能提前返回的要提前返回。

**普通代码**

```java
double result;
if (value <= MIN_LIMIT) {
    result = MIN_LIMIT;
} else { 
   result = value;
}
```

**精简代码**

```java
double result = Math.max(MIN_LIMIT, value);
```

#### 不要使用List.contains，改用Set.contains

**普通代码**

```java
List<String> skuList = new ArrayList<String>();
    skuList.add("A");
    skuList.add("B");
    skuList.add("C");
    List<OrderLineDTO> orderLineList = orderService.selectOrderLineList();
    /**
     * 错误的用法
     */
    for(OrderLineDTO orderLine:orderLineList){
        // 调用了List.contains 方法，这个方法里通过遍历循判断的，性能比较差
        if(skuList.contains(orderLine.getSkuCode())){
            ...   
        }
    }
```

**优化代码**

```java
	/**
     * 正确的用法
     */
    Set<String> skuSet = new HashSet<String>();
    skuSet.add("A");
    skuSet.add("B");
    skuSet.add("C");
    List<OrderLineDTO> orderLineList = orderService.selectOrderLineList();
    for(OrderLineDTO orderLine:orderLineList){
        // 调用Set.contanins 这个方法通啥希算法判断，性能更高
        if(skuList.contains(orderLine.getSkuCode())){
            ...   
        }
    }
```

#### 双重for循环，改用map实现

> 避免使用双重for循环，一般建议循环不要超过2层，也就是小于等于2层循环

```java
Pager<OrderLineEntity> orderLinePager = orderLineDomainService.doPager(params);
        List<String> skuCodeList  = orderLinePager.getList().stream().map(OrderLineEntity::getSkcCode).collect(Collectors.toList());
		// 根据skuCode查询出商品集合
        List<ItemResDto> itemList = itemService.getItemList(skuCodeList);
        /**
         * 错误的用法
         */
        for(OrderLineEntity orderLine:orderLinePager.getList()) {
            for(ItemResDto itemResDto:itemList){
                if (orderLine.getSkuCode().equals(itemResDto.getSkuCode())) {
                    //Do something
                }
            }
        }
```

```java
		/**
         * 正确的用法
         */
    	Map<String, ItemResDto> skuMap = itemPager.getList().stream().collect(Collectors.toMap(ItemResDto::getSkcCode, item->item));
        for(OrderLineEntity orderLine:orderLinePager.getList()) {
            ItemResDto itemResDto = skuMap.get(orderLine.getSkcCode());
            if (itemResDto!=null) {
                //Do something
            }
        }
```



#### 其他优化策略

1. 使用Set去重数据
2. 避免循环调接口/数据库
3. 使用缓存，将查询频率比较高但修改频率比较低的数据缓存起来
4. 使用多线程，大的任务拆分成小的任务，使用多线程并行执行；比如要查导出1万个sku的数据，如果分页查询，每页2000个sku，则需要循环5次才能查询完成，那么执行时间是5次查询时间之和；如果分开5个线程同时去查，每个线程查2000，那么整个接口的响应时间是最慢的那个线程的响应时间。


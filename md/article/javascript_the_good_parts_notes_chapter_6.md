# 《JavaScript语言精粹》读书笔记 第六章 数组
_2011-04-27 17:16_

### 6.0绪

数组可以是很快的数据结构，但是JS不是这种。

JS提供了一种拥有一些类数组（array-like）特性的对象。把下标转变成字符串作为属性。

比真正的数组慢，但很方便。

检索和更新与对象一模一样，除了可以用整数作为属性名。

$$solo_more$$

### 6.1数组字面量

数组字面量是在一对方括号中包围零个或多个用逗号分隔的表达式。

JS数组允许包含任意混合类型的值！

### 6.2长度

每个数组有一个length，length没有上界，如果用大于或等于当前length的数字作为下标来保存一个元素，length将增大来容纳新元素。

length属性的值是最大整数属性名加1，不一定等于数组里属性的个数！

length可写，设置更大的值无须分配空间（注：貌似，JS数组是”稀疏“的），设置更小的值将删除多的属性。

### 6.3删除

delete numbers[2]可以删除，但是会留下undefined在原位置。

	numbers.splice(index,num[,newitem[,newitem...]])

删除元素，后面的位置前移，如果newitem存在，则插入index。效率不高。

### 6.4枚举

可以用for in，但是顺序无法保证，而且不能排除原型链中的属性。

可以用for循环。

### 6.5混淆的地方

typeof数组是”object”。

typeof null是”object”。

JS没有好的机制区分数组和对象。

	var is_array = function(value){
		return value &&
			typeof value === "object" &&
			value.constructor === Array;
	}

这段代码，如果识别从不同的window或者frame里构造的数据就会失败。

	var is_array = function(value){
		return value &&
			typeof value === "object" &&
			typeof value.length === "number" && //对象不一定
			typeof value.splice === "function" && //所有数组有
			!(value.propertyIsEnumerable("length")); //length不能用for in枚举
	}

（第一行拒绝了null，第三行确定了是数组，为什么要有其他行？对象也可能添加splice方法，也可能添加length属性。）

### 6.6方法

数组添加非数字属性名的属性时，不改变length。

### 6.7维度

JS数组中的属性值为数组，则构成二维数组。

第二维的数组需要自己创建。
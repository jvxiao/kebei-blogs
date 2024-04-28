# Array, Set, Map知多少？

Array,Set和Map三个作为Javascript中可迭代的集合数据类型，在编程过程中使用的频率也比较高。针对三种数据类型各自的一些特性，本文的内容将从以下几个方面来上述数据类型做一个总结。

-  [**实例的创建**](##实例的创建)
-  [**数据添加**](##数据添加)
-  [**数据访问（查找）**](##数据访问（查找）)
-  [**数据的遍历**](##数据的遍历)
-  [**类似的功能和一些专有方法**](##类似的功能和一些专有方法)
-  [**三者之间的转换**](##三者之间的转换)
-  [**应用场景**](##应用场景)

## 实例的创建
  Map和Set创建实例的方式是唯一的，只允许通过new调用构造方法来创建一个实例。值得注意的是，Set和Map在调用构造函数的时候，传参都是数组或者可迭代对象，其中Map的传参数组需要时一个键值对数组。当然，传参也是可选的，非必须。

 Array相较于前两者，其创建实例的方式要多写，可通过字面方式创建，也可像Map和Set一样通过构造函数来创建。此外使用Array的静态函数Array.from来创建也是非常常见的。例如，通过docuemnt.getElementsByTagName这类方法获取到一个HTMLCollect这样一个类数组，通常会通过Array.from将其装换成一个真正的数组来进行后续的操作。

  ```Javascript

   // 创建map实例
   let map1 = new Map();
   let map2 = new Map([['a', 1], ['b', 2]]); //  a=>1, b=>2
   // 创建set实例
   let set1 = new Set();
   let set2 = new Set([1,2])

  // 通过字面量创建数组
  let arr1 = [1,2,3];
  // 使用Array静态方法从一个可迭代对象或者类数组中创建数组实例
  let arr2 = Array.from([4,5]) 
  // 使用Array构造函数
  let arr3 = new Array([6,7]);
```

## 数据添加与变更


```Javascript
  // Set的数据操作：添加与删除
  let set = new Set();
  set.add(11) // [11]
  set.add(23) // [11, 23]
  set.add(11) // [11, 23]
  set.delete(23) // [11] 
  
  // Map的数据操作：添加与删除
  let  map = new Map();
  map.set('a', 1);   // [a=>1]
  map.set('b', 2);   // [a=>1, b=>2]
  map.delete('b');   // [a=>1]

  // Array
  let arr = [1,2,3]
  // 在数组末尾添加一个值
  arr.push(4)       // [1,2,3,4]
  // 弹出数组末尾的值
  arr.pop(4)        // [1,2,3]
  // 在数组头部添加一个值
  arr.unshift(0)    // [0,1,2,3]
  // 移除头部的第一个值
  arr.shift()       // [1,2,3]
  // 在第二个数（下标1）之后插入一个值‘a’
  arr.splice(1,0,'a')   // [1,2,'a', 3];
  // 删除第二个数（下标1）后面1个值，然后插入一个新的值‘b’
  arr.splice(1,1,'b')   // [1,2,'b',3]
```

## 数据访问
  Array和Map都能够访问实例中的特定数据，Array是通过下标，而Map是通过实例方法get, 唯独Set没有方式可以直接访问其中特定数据。其实也不难理解，Set本身不是为了单纯存储数据和访问特殊而生的，因为这些功能Array就可以支持，何必再单出构造一个Set数据结构了。我对此的理解是，Set跟多的是紧紧围绕着数据唯一不重复这一准则来的，它的侧重点是某一数据的有无，而不是数据存在哪里。

  另外，从结构上来说。Set不像Array那样是有序的，所以也无法使用下标来访问，也不像Map那般，每个键对应一个值，所以也无法通过键来访问。故而，Set没有单独访问某一数据的方式。

```Javascript
  const arr = [1,2,3];
  const set = new Set(['a', 'b']);
  const map = new Map([['a', 1], ['b', '2']]);
  
  //Array通过下标访问数据
  console.log(arr[0], arr[2]) // 1, 3
  //Map使用实例方法get访问数据，参数是键
  console.log(mpa.get('a))    // 1
```
## 数据的遍历

**Set数据的遍历方式**：
-  keys()   &emsp;&emsp;返回键名迭代器
-  values() &emsp;返回值迭代器  
-  entries() &emsp;返回键值迭代器
-  forEach()
```Javascript
const set = new Set(['a', 'b', 'c']);
// set每个值对应的key其实也是数据值本身
const keyIter = set.keys();
console.log(keyIter.next().value);  // 'a'
console.log(keyIter.next().value);  // 'b'
console.log(keyIter.next().value);  // 'c'
console.log(keyIter.next().value);  // undefined

const valueIter = set.values();
for(let val of valueIter) {
  console.log(val);                // 'a', 'b', 'c'
}

const entriesIter = set.entries();
for(let [key, value] of entriesIter) {
  console(`${key}:${value}`)     // 'a':'a', 'b':'b', 'c':'c'
}

set.forEach(val => {
  console.log(val)             // 'a', 'b', 'c'
})
```
</br>

**Map数据的遍历方式**
-  keys()   &emsp;&emsp;返回键名迭代器
-  values() &emsp;返回值迭代器  
-  entries() &emsp;返回键值迭代器
-  forEach()

```Javascript
const map = new Map([['a', 1], ['b', '2']]);
/**
* 代码就自己脑补吧，
* 不能说和Set的方式很像，
* 只能说真的就是一模一样
**/
```
</br>

**Array数据的遍历方式**

Array遍历数据的方法是最多的，除了下面列举的几个之外，还有some, every,甚至filter, find和findIndex这些方法可以用来遍历数据。使用这些方法需要注意的是其使用的场景，例如map和forEach都可以用来对数组内数据做一些操作，但如果不需要返回值的情况，还是使用forEach方法，而不建议是map，其它方法也是如此。

-  keys()   &emsp;&emsp;返回键名迭代器
-  values() &emsp;返回值迭代器  
-  entries() &emsp;返回键值迭代器
-  map()    &emsp;回调函数，要有返回值
-  reduce() 
-  forEach()
-  for...of
```Javascript
/**
* 代码就不写了，偷个懒，不过还是贴心的附上链接
* https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array
**/
```


## 类似的功能和一些专有方法
- **是否包含某个元素**: 数组的includes方法，map和set的has方法，三者的返回值都是布尔值，区别只在于传入的参数。数组和set传入的是值，map传入的是键。
- **合并两个相同类型的数据**： 数组使用concat方法，合并两个数组值，set使用union方法合并两个集合。map没有专有方法可以用来合并两个map数据，不过可以通过使用new Map([...map1, ...map2])来返回一个合并之后的新map
- **数据大小**：数组有length属性，map和set有size属性
- **清空数据**：map和size都可通过clear()方法来清空数据，数组无专有方法，可自己通过修改length值为0或者使用splice方法清空数据。

>关于数组中的includes方法在数组值为一个个对象的时候，这个时候传入的值就只能是对象的引用了。如果作用域内不存在引用的话，判断数组中是否存在某个对象，就只能通过filter, find或者findIndex之类的方法加上数据特征去判断数组中是否存在该对象了。

  
## 三者之间的转换


```Javascript
const arr = [1, 3, 4, 4];
const arr1 = [['a', 'Kebei']];
const map = new Map(['a', 1], ['b', 2]);
const set = new Set([1, 5]);

// Array 转 Set
const arr2Set = new Set(arr);   // [1, 3, 4];
// Array 转Map
const arr2Map = new Map(arr1);  // {'a' => 'Kebei'}

//Set转Array
const set2Arr = Array.from(set);        // [1, 5]
const set2Arr2 = [...set];              // [1, 5]
const set2Arr3 = set.values()           // [1, 5]
//Set转Map
const set2Map = new Map(set.entries())  // { 1=> 1, 5=>5}

// Map转Array
const map2Arr = Array.from(map);     // ['a', 1], ['b', 2]
const map2Arr2 = [...map];           // ['a', 1], ['b', 2]
const map2Arr3 = map.values()        // ['a', 1], ['b', 2]
// Map转Set
const map2Set = new Set(map.values()) // [1, 2]
```
## 应用场景
</br>
最常见的一种场景莫过于使用数组与Set之间格式变化进行数据去重

```Javascript
  const dedup = (arr) => {
    return [...new Set(arr)]
  }
```
上述方法和之前includes方法也是一样存在无法处理复杂数据类型，对于复杂对象数据，需要根据各自业务场景对重复的数据进行去重策略选择，即在一堆的重复对象中（以id重复为例）保留业务需要的唯一一个数据。

## 总结
三种数据结构有着各自不同的特性。数组是一个天然的栈，也是一个天然的队列，在三种数据结构中，其实例方法也应用也是最多的，是线性存储中话事人般的存在，也是我们在普通业务场景下的首选。Set数据的唯一性，可以帮助我们在业务场景中快速的进行去重。Map键值对结构的特殊性，以及对键的包容性，能够通过键快速获取到值，也是复杂业务冲常常用的。
</br>

【资料参考】
  1. https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Array
  2. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set
  3. https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
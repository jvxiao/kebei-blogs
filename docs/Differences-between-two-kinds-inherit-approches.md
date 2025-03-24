# Dog.prototype = new Animal() 和 Dog.prototype.proto = Animal.prototype的两种继承方式的区别


## 1. Dog.prototype = new Animal()

### 语法和机制
- **​目的**：通过创建 Animal 的实例来继承其原型链。
- ​**底层逻辑**：将 Dog.prototype 替换为一个新的 Animal 实例。
- **​原型链结构**：

```javascript
Dog.prototype → Animal 实例 → Animal.prototype → Object.prototype → null
```

因此，Dog 的实例通过原型链可以访问 Animal 实例的属性和 Animal.prototype 的方法。

### ​特点

- **​调用父类构造函数**： 会执行 Animal() 构造函数，初始化父类属性（如 this.name)
- **继承实例属性**：Dog.prototype 会包含 Animal 实例的属性（如 name），即使这些属性是实例级别的。

- **覆盖原有原型**：替换 Dog.prototype 后，原先定义在 Dog.prototype 上的方法会被丢失，需重新添加。
​- **constructor 问题**：Dog.prototype.constructor 会指向 Animal，需手动修正：
``` javascript
  Dog.prototype.constructor = Dog;
```
​示例
``` javascript
  function Animal() { this.name = "Animal"; }
  Animal.prototype.eat = function() { console.log("Eat"); };

  function Dog() {}
  Dog.prototype = new Animal(); // 继承
  Dog.prototype.constructor = Dog; // 修复 constructor

  const dog = new Dog();
  console.log(dog.name); // "Animal"（继承自 Animal 实例）
  dog.eat();           // "Eat"（继承自 Animal.prototype）
```

## ​2. Dog.prototype.proto = Animal.prototype

### ​语法和机制

- ​目的：直接让 Dog.prototype 的 proto 属性（即 __proto__）指向 Animal.prototype。

- ​底层逻辑：修改 Dog.prototype 的原型链。

​-  原型链结构：
```javascript
  Dog.prototype → Animal.prototype → Object.prototype → null
```

因此，Dog 的实例直接通过原型链访问 Animal.prototype 的方法，但不会继承 Animal 实例的属性。
### ​特点

- **​不调用父类构造函数**：不会执行 Animal()，避免副作用（如初始化逻辑）。

​- **仅继承原型方法**：Dog 的实例只能访问 Animal.prototype 的方法，无法获取 Animal 实例的属性（如 name）。

- **​保留原有原型**：不会覆盖 Dog.prototype 上已定义的方法。

- **​非标准操作**：直接修改 proto（或 __proto__）是非标准的，可能影响性能，推荐用 Object.setPrototypeOf()：
```javascript
Object.setPrototypeOf(Dog.prototype, Animal.prototype);
```
​示例
```javascript
function Animal() { this.name = "Animal"; }
Animal.prototype.eat = function() { console.log("Eat"); };

function Dog() {}
Dog.prototype.proto = Animal.prototype; // 直接修改 proto

const dog = new Dog();
console.log(dog.name); // undefined（未继承 Animal 实例属性）
dog.eat();           // "Eat"（继承自 Animal.prototype）

```

### ​关键区别总结

### ​**关键区别总结**

| ​**特性**                | `Dog.prototype = new Animal()`                          | `Dog.prototype.__proto__ = Animal.prototype`         |
|-------------------------|--------------------------------------------------------|-----------------------------------------------------|
| ​**调用父类构造函数**     | ✅ 是（执行 `Animal()`，初始化父类实例属性）            | ❌ 否（直接链接原型，不调用构造函数）                 |
| ​**继承实例属性**         | ✅ 是（继承 `Animal` 实例的属性，如 `this.name`）       | ❌ 否（仅继承原型方法，不包含实例属性）               |
| ​**覆盖原型对象**         | ✅ 是（完全替换 `Dog.prototype` 为新实例）             | ❌ 否（仅修改原型的 `__proto__`，保留原有属性和方法） |
| ​**constructor修正**    | ✅ 需手动修复（`Dog.prototype.constructor = Dog`）     | ✅ 通常无需修复（原型链未改变构造函数引用）           |
| ​**兼容性**               | ⚠️ 传统写法，但可能引发副作用（如多余属性继承）       | ⚠️ 非标准操作（直接修改 `__proto__` 可能影响性能）   |
| ​**推荐程度**             | ❌ 不推荐（问题多，已过时）                             | ⚠️ 慎用（可用 `Object.setPrototypeOf()` 替代）         |
| ​**原型链结构**           | `Dog.prototype → Animal 实例 → Animal.prototype`         | `Dog.prototype → Animal.prototype → Object.prototype`   |
| ​**是否继承父类方法**     | ✅ 是（通过 `Animal.prototype`）                       | ✅ 是（直接链接到 `Animal.prototype`）                |
| ​**是否继承父类实例方法** | ❌ 否（仅继承原型方法，实例属性不会被继承）             | ❌ 否（同上）                                        |
| ​**典型用途**             | 早期原型链继承（已不推荐）                            | 理论原型链扩展（极少使用，多用 `Object.create()`）   |
| ​**替代方案**             | `Object.create(Animal.prototype)` + 手动初始化属性     | `Object.setPrototypeOf(Dog.prototype, Animal.prototype)` |
| ​**ES6 推荐写法**         | `class Dog extends Animal {}`                           | `class Dog extends Animal {}`                         |

### 补充说明
- ​**Dog.prototype = new Animal() 的问题**：

  - 强制调用父类构造函数，可能导致不必要的初始化逻辑（如 Animal 的 this.name）。
  - 会将父类实例的属性（如 name）提升为 Dog.prototype 的属性，导致所有子类实例共享这些属性（可能引发意外行为）。

- ​**Dog.prototype.__proto__ = Animal.prototype 的问题**：

  - ​非标准操作：虽然浏览器支持，但 __proto__ 是内部属性，直接修改可能导致性能下降或兼容性问题。
  - ​无法继承父类构造函数：子类实例仍需通过 new Dog() 创建，不会自动调用父类构造函数。


### ​现代替代方案：

​**Object.create()**：
```javascript
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
```
直接继承 Animal.prototype，不调用父类构造函数，更安全高效。

​ES6 class 语法：
```javascript
class Dog extends Animal {}
```
自动处理原型链和构造函数继承，代码更简洁清晰。
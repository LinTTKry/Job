# Class

## ​Def

要创建对象，必须要先定义类，所谓的类可以理解为对象的模型，程序中可以根据类创建指定类型的对象，举例来说：可以通过Person类来创建人的对象，通过Dog类创建狗的对象，通过Car类来创建汽车的对象，不同的类可以用来创建不同的对象

## Definition：

```typescript
typescript class className {
    propName: type;
    constructor(param: type){
        this.propName = param;
    }
    methodName(){
        ....
    }
}
```

## 属性及方法

```typescript
class Person{
    // 直接定义属性是实例属性，需要通过对象的实例去访问
    name: string = '孙';
    
    // 使用static开头的属性是静态属性（类属性），可以直接通过类去调用
    static age:number = 18
    
    // readonly开头的属性表示一个只读的属性无法修改，与static搭配注意顺序
    static readonly name1: string = '八'
    
    //定义实例方法, 如果方法以static开头则方法就是类方法，可以直接通过类去调用
    sayHello(){
        console.log(`大家好，我是${this.name}`);
    }
}
const p = new Person('孙悟空', 18);
p.sayHello();    
console.log(p.name)//实例属性
console.log(Person.age)//类属性
```

## 构造函数

```typescript
class Person{
    name2: string;// this添加的属性要先声明其类型
    age2: number;
    // 构造函数会在对象创建时调用
    constructor(name:string, age:number){        
        // 在实例方法中，this就表示当前的实例
        // 在构造函数汇中当前对象就是当前新建的那个对象
        // 可以通过this向新建的对象中添加属性
        console.log(this)
        this.name2 = name;
        this.age2 = age;
    }
}
// 为什么不在构造函数外面赋值，因为这意味着Person构造的实例的name2,age2属性值都一样；
// 所以这时候才用到construcor,在构造类的时候用，这样在不同实例的两个属性值不同
```

## 继承1-extends

```typescript
class Animal{
    name: string;
    age: number;
    constructor(name:string, age:number){
        console.log(this)
        this.name = name;
        this.age = age;
    }
    sayhello(){
        console.log('动物再叫')
    }
    sayhello1(){
        console.log('动物再叫1111')
    }
}
class Cat extends Animal{
    /* 此时，Animal被称为父类，Dog被称为自雷
       使用继承后，子类将拥有父类所有的方法和属性
       通过继承可以将多个共有的代码写在一个父类中
        这样只需要写一次即可让所有的子类都同时拥有父类的属性和方法
        如果希望在子类中添加一些父类中没有的属性或方法直接加就可
    */
   run(){
       console.log(`${this.name} is running`)
   }
   // 如果在子类中添加了和父类相同的方法，则子类方法会覆盖父类的方法
     //这种子类覆盖父类方法的形式，我们称为方法重写
   sayhello(){
       console.log('miaomiaomiao')
   }  
}
const cat = new Cat();
console.log(cat.run())
cat.sayhello1()//可以访问，且打印：动物再叫1111
```

## 继承1-super

```typescript
class Animal{
    name: string;
    constructor(name:string, age:number){
        console.log(this)
        this.name = name;
    }
    sayhello(){
        console.log('动物再叫')
    }
}
class Cat extends Animal{
   age: number;
   constructor(name:string,age:numberr){
      // 如果在子类中写了构造函数，在子类构造函数中必须对父类构造函数进行调用
      super(name)// 调用父类的构造函数
      this.age = age
   }
   sayhello(){
        // 在类的方法中super就表示当前类的父类
        super.sayhello();
   }
}
const cat = new Cat();
cat.sayhello()//动物再叫

```

## 抽象类（ts新增）

```typescript
/*
    以abstract 开头的类是抽象类
      抽象类和其他类区别不大，只是不能用来创建对象
      抽象类就是专门用来被继承的类
      
      抽象类中可以添加抽象方法
*/
abstract class Animal{
    name: string;
    constructor(name:string, age:number){
        console.log(this)
        this.name = name;
    }
    // 定义一个抽象方法
    // 抽象方法使用abstract开头，没有方法体
    // 抽象方法只能定义在抽象类中，子类必须对抽象方法进行重写
    abstract sayhello(){
        console.log('动物再叫')
    }
}
```

## 接口——用来定义一个类的结构（ts新增）

```typescript
// 描述一个对象的类型
type myType = {
  name: string,
  age: number
}

/*
    接口用来定义一个类的结构
      用来一个类中应该包含哪些属性和方法
      同时接口也可以当成类型声明去使用，可重复,属性叠加
*/
interface myInterface{
  name:string;
  age:number;
}
interface myInterface{
  gender:string;
}
const obj:myInterface={
   name:'s',
   age:111,
   gender:'man'//不写此属性报错
}
/*
   接口可以在定义类的时候去限制类的结构
     接口中的所有属性都不能有实际的值
     接口只定义对象的结构，二不考虑实际值
         在接口中所有的方法都是抽象方法,而抽象类中可以有抽象方法，也可以有普通方法
*/
interface myInterface{
    name: string;
    sayhello():void;
}
/*
   定义类时，可以使用类去实现一个接口
     实现接口就是使类满足接口的要求
*/
class MyClass implements myInterface{
  name:string;
  constructor( name:string){
    this.name = name;
  }
  sayhello(){
     console.log('aaaa')
  }
}
```

## 属性的封装---将属性私有化，提供方法间接外部访问

```typescript
class Person{
    /*
        TS可以在属性前添加属性的修饰符
        public修饰的属性可以在任意位置访问或修改 默认值
        private 私有属性，只能在当前类内部访问或修改,在子类也不可以
           可以在类中添加方法使得私有属性可以被外部访问
        protected 受保护属性，只能在当前类与子类中访问或修改
    */
    private  name: string;
    private  age: number;
    constructor(name:string, age:number){        
        this.name = name;
        this.age = age;
    }
    /*
       getter,setter方法被称为属性的存取器
    */
    //定义方法，用来获取属性
    // getName(){
    //   return this.name;
    // }
    // 设置方法，修改属性
    // setName(value:string){
    //   if(value>=0) this.name = value
    // }
    
    // TS中设置getter方法的形式
    get name(){
        return this.name;
    }
    set name(value:string){
        if(value>=0) this.name = value
    }
}
const per = new Person(name:'s',age:18)
/*
    现在属性是在对象中设置的，属性可以任意被修改
       属性可以任意被修改将导致对象中的数据都变得非常不安全
*/
// per.name = 'z';
// per.age = -38;
// console.log(per.age)//-38
// per.setName('z')
console.log(per.name)//调用TS中的getter方法
per.name = 'z'//调用TS中的setter方法
// 简化语法
class A{
    // 可以直接将属性定义在构造函数中
    constructor(public name:string, public  age:number){        
        this.name = name;
        this.age = age;
    }
}
```

## 泛型

```typescript
function fn(a:any):any{
   return a
}
/*
   在定义函数或是类时，如果遇到类型不明确就可以使用泛型
   
*/
function fn<T>(a:T):T{
  return a
}
// 可以直接调用具有泛型的函数
fn(a:10)// T--number 不指定泛型，TS自动对类型进行判断
fn<string>(a:'hello')//指定泛型

//泛型可以同时指定多个
function fn2<T,K>(a:T,b:K):T{
  return a
}
fn2<number,string>(a:123,b:'hello')

interface Inter{
   length:number
}
// T extends Inter 表示泛型必须满足Inter实现类（子类）
function fn3<T extends Inter>(a:T):number{
  return a.length
}
fn3(a:'123')
class Myclass<T>{
   name: T;
   constructor(name:T){
      this.name = name
   }
}
const mc = new Nyclass<string>(name:'s')
```

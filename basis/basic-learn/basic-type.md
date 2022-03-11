# Basic Type

## Type Declaration

* Type declaration is a important **feature** of TS
* Type declaration can **specify the type** of variables (parameters, formal parameters) in TS
* After specifying the type, when assigning a value to a variable, the TS compiler will **automatically** check whether the value conforms to the type declaration, and assign it if it conforms, otherwise an error
* **In short, type declaration sets the type to the variable so that the variable can only store a certain type of value**
* **Grammar**：

```javascript
let variable: type;
let variable: type = value;

function fn(param: type, param: type): type{
    ...
}
// where ': type' indicates the type of return value of function

let a:number;
a = 10;
a = 'hello'
// Although an error is reported, it can be compiled successfully,
// Compile options can be set to prevent successful 
// compilation and compile into any version of js
```

## Automatic type judgment

* TS has an **automatic type judgment** mechanism
* When the **variable declaration and assignment** are carried out at **the same time**, the TS compiler will **automatically determine the type** of the variable
* So if your variable declaration and assignment are carried out at the same time, you **can omit the type declaration**

## Type:

|   Type  |             Example            |                  Description                 |
| :-----: | :----------------------------: | :------------------------------------------: |
|  number |           1, -33, 2.5          |                  any number                  |
|  string |        'hi', "hi", `hi`        |                  any string                  |
| boolean |           true、false           |                  true/ false                 |
| literal |              self              |           The value of the literal           |
|   any   |               \*               |                   any type                   |
| unknown |               \*               |              any with safe type              |
|   void  | empty value / undefined / null | <p>empty value / </p><p>undefined / null</p> |
|  never  |            no value            |             can not be any value             |
|  object |          {name:'孙悟空'}          |               any object of js               |
|  array  |            \[1,2,3]            |                   any array                  |
|  tuple  |             \[4,5]             |     array with fixed length(new ts type)     |
|   enum  |           enum{A, B}           |                  enumeration                 |

* number
  * ```typescript
    let decimal: number = 6;
    let hex: number = 0xf00d;
    let binary: number = 0b1010;
    let octal: number = 0o744;
    let big: bigint = 100n;
    ```
* boolean
  * ```typescript
    let isDone: boolean = false;
    ```
* string
  * ```typescript
    let color: string = "blue";
    color = 'red';

    let fullName: string = `Bob Bobbington`;
    let age: number = 37;
    let sentence: string = `Hello, my name is ${fullName}.

    I'll be ${age + 1} years old next month.`;
    ```
* Literal
  * You can also use literals to specify the type of variables, and you can determine the value range of variables through literals
  * ```typescript
    let color: 'red' | 'blue' | 'black';
    let num: 1 | 2 | 3 | 4 | 5;
    ```
* any(not commended)：equivalent to turning off the type detection of ts
  * ```typescript
    let d: any = 4;
    d = 'hello';
    d = true;
    let d;
    // When  declared variable does not specify the type, 
    // It will be automatically judged as any
    // Try to avoid using any, 
    // Try to use Unknown instead of any
    ```
* unknown
  * ```typescript
    let notSure: unknown = 4;
    notSure = 'hello';

    let e: unknown;
    let a: any;
    let s: string;
    e = 'hello';
    a = 'hallo';
    // any type variable can be assigned to any types of variables
    s = a;//normal
    // unknown is an safe any type
    // can not be assigned to other variables directly
    s = e;//Erroe
    //other assignment method
    if(typeof e==='string'){
       s = e
    }
    // Type assertion
    // which can be used to tell the parser the actual type of the variable
    s = e as string
    s = <string>e
    ```
* void
  * ```typescript
    // :void represents there is 
    // no return value of fn 
    // or empty value or undefined/null
    function fn():void{
       return;
       // return undefined or null;
    }
    ```
* never
  * ```typescript
    function error(message: string): never {
      throw new Error(message);
    }
    // empty value also not allowed
    // often be used to throw an error
    ```
* object
  * ```typescript
    // {} is used to indicates that the propertis contained in obj
    // gram: {property:value}
    // {property?:value} indicates the property is optional
    // [propName:string]:any indicates properties of any type
    let obj: object = {name:string,age?:number};
    let obj: object = {name:string,[propName:string]:any};
    let d:(a:number,b:number)=>number;
    d = function(n1:string,n2:string):{
       return n1+n2
    }// error
    d = function(n1,n2,n3):{
       return n1+n2
    }// error
    ```
* array
  * ```typescript
    // number[] number array
    let list: number[] = [1, 2, 3];
    let list: Array<number> = [1, 2, 3];
    ```
* tuple
  * ```typescript
    let x: [string, number];
    x = ["hello", 10];
    ```
* enum
  * ```typescript
    enum Color {
      Red,
      Green,
      Blue,
    }
    let c: Color = Color.Green;

    enum Color {
      Red = 1,
      Green,
      Blue,
    }
    let c: Color = Color.Green;

    enum Color {
      Red = 1,
      Green = 2,
      Blue = 4,
    }
    let c: Color = Color.Green;
    ```
* & expression

```typescript
let j: {name:string}&{age:number};
j = {name:'sun',age:18}
```

* Type alias

```typescript
type myType = 1|2|3|4|5;
let k : myType;
let l : myType;
```

* Type assertion
  * In some cases, the type of the variable is very clear to us, but the TS compiler is not clear. At this time, you can tell the compiler the type of the variable through type assertion. There are two forms of assertion:
    * First
      * ```typescript
        let someValue: unknown = "this is a string";
        let strLength: number = (someValue as string).length;
        ```
    * Second
      * ```typescript
        let someValue: unknown = "this is a string";
        let strLength: number = (<string>someValue).length;
        ```

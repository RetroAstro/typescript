### 接口

可选属性

```ts
interface SquareConfig {
  color?: string
  width?: number
}

function createSquare(config: SquareConfig): { color: string; area: number } {
  let newSquare = { color: "white", area: 100 }

  if (config.clor) {
    // Error: Property 'clor' does not exist on type 'SquareConfig'
    newSquare.color = config.clor
  }

  if (config.width) {
    newSquare.area = config.width * config.width
  }

  return newSquare
}

let mySquare = createSquare({color: "black"})
```

绕开类型检查

```ts
interface SquareConfig {
    color?: string
    width?: number
    [propName: string]: any
}
```

类实现接口

```ts
interface ClockInterface {
    currentTime: Date
    setTime(d: Date)
}

class Clock implements ClockInterface {
    currentTime: Date
    setTime(d: Date) {
        this.currentTime = d
    }
    constructor(h: number, m: number) { }
}
```

构造函数检查

```ts
interface ClockConstructor {
  new (hour: number, minute: number): ClockInterface
}

interface ClockInterface {
  tick()
}

function createClock(ctor: ClockConstructor, hour: number, minute: number): ClockInterface {
  return new ctor(hour, minute)
}

class DigitalClock implements ClockInterface {
  constructor(h: number, m: number) { }
  tick() {
      console.log("beep beep")
  }
}
class AnalogClock implements ClockInterface {
  constructor(h: number, m: number) { }
  tick() {
      console.log("tick tock")
  }
}

let digital = createClock(DigitalClock, 12, 17)
let analog = createClock(AnalogClock, 7, 32)
```

继承接口

```ts
interface Shape {
  color: string
}

interface PenStroke {
  penWidth: number
}

interface Square extends Shape, PenStroke {
  sideLength: number
}

// let square = <Square>{}
let square = {} as Square

square.color = "blue"
square.sideLength = 10
square.penWidth = 5.0
```

### 类

protected 与 private

```ts
class Person {
  protected name: string
  constructor(name: string) {
    this.name = name
  }
}

class Employee extends Person {
  private department: string
  constructor(name: string, department: string) {
    super(name)
    this.department = department
  }
  public getElevatorPitch() {
    return `Hello, my name is ${this.name} and I work in ${this.department}.`
  }
}

let howard = new Employee("Howard", "Sales")

console.log(howard.getElevatorPitch())
console.log(howard.name) // 错误
```

你可以使用 `readonly`关键字将属性设置为只读的，只读属性必须在声明时或构造函数里被初始化。

```ts
class Octopus {
    readonly name: string
    readonly numberOfLegs: number = 8
    constructor (theName: string) {
        this.name = theName
    }
}

// or

class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}

let dad = new Octopus("Man with the 8 strong legs")
dad.name = "Man with the 3-piece suit" // 错误! name 是只读的.
```

getter / setter 存取器

```ts
let passcode = "secret passcode"

class Employee {
    private _fullName: string

    get fullName(): string {
        return this._fullName
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName
        }
        else {
            console.log("Error: Unauthorized update of employee!")
        }
    }
}

let employee = new Employee()
employee.fullName = "Bob Smith"

if (employee.fullName) {
    alert(employee.fullName)
}
```

### 函数

剩余参数

```ts
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ")
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie")
```

this 参数

```ts
interface Card {
  suit: string
  card: number
}

interface Deck {
  suits: string[]
  cards: number[]
  createCardPicker(this: Deck): () => Card
}

let deck: Deck = {
  suits: ["hearts", "spades", "clubs", "diamonds"],
  cards: Array(52),
  // NOTE: The function now explicitly specifies that its callee must be of type Deck
  createCardPicker: function(this: Deck) {
      return () => {
          let pickedCard = Math.floor(Math.random() * 52)
          let pickedSuit = Math.floor(pickedCard / 13)

          return {
            suit: this.suits[pickedSuit],
            card: pickedCard % 13
          }
      }
  }
}

let cardPicker = deck.createCardPicker()
let pickedCard = cardPicker()

alert("card: " + pickedCard.card + " of " + pickedCard.suit)
```

函数重载

```ts
function padding(all: number)
function padding(topAndBottom: number, leftAndRight: number)
function padding(top: number, right: number, bottom: number, left: number)

// Actual implementation that is a true representation of all the cases the function body needs to handle

function padding(a: number, b?: number, c?: number, d?: number) {
  if (b === undefined && c === undefined && d === undefined) {
    b = c = d = a
  } else if (c === undefined && d === undefined) {
    c = a
    d = b
  }

  return {
    top: a,
    right: b,
    bottom: c,
    left: d
  }
}

padding(1) // Okay: all
padding(1, 1) // Okay: topAndBottom, leftAndRight
padding(1, 1, 1, 1) // Okay: top, right, bottom, left

padding(1, 1, 1) // Error: Not a part of the available overloads
```

### 泛型

```ts
class BeeKeeper {
  hasMask: boolean
}

class ZooKeeper {
  nametag: string
}

class Animal {
  numLegs: number
}

class Bee extends Animal {
  keeper: BeeKeeper
}

class Lion extends Animal {
  keeper: ZooKeeper
}

function createInstance<A extends Animal>(c: new () => A): A {
  return new c()
}

createInstance(Lion).keeper.nametag  // typechecks!
createInstance(Bee).keeper.hasMask   // typechecks!
```

### 枚举

数字枚举

```ts
enum Direction {
    Up,
    Down,
    Left,
    Right,
}
```

字符串枚举

```ts
enum Direction {
    Up = "UP",
    Down = "DOWN",
    Left = "LEFT",
    Right = "RIGHT",
}
```

常量枚举

```ts
const enum Directions {
    Up,
    Down,
    Left,
    Right
}

let directions = [Directions.Up, Directions.Down, Directions.Left, Directions.Right]

// after compile

var directions = [0 /* Up */, 1 /* Down */, 2 /* Left */, 3 /* Right */]
```

### 高级类型

交叉类型

```ts
function extend<T, U>(first: T, second: U): T & U {
  let result = <T & U>{}

  for (let id in first) {
    (<any>result)[id] = (<any>first)[id]
  }

  for (let id in second) {
    if (!result.hasOwnProperty(id)) {
      (<any>result)[id] = (<any>second)[id]
    }
  }

  return result
}

class Person {
  constructor(public name: string) {}
}

interface Loggable {
  log(): void
}

class ConsoleLogger implements Loggable {
  log() {}
}

var jim = extend(new Person("Jim"), new ConsoleLogger())
var n = jim.name
jim.log()
```

联合类型

```ts
interface Bird {
  fly()
  layEggs()
}

interface Fish {
  swim()
  layEggs()
}

class TinyFish implements Fish {
  swim() {}
  layEggs() {}
}

function getSmallPet(): Fish | Bird {
  return new TinyFish()
}

let pet = getSmallPet()

pet.layEggs()

if ((<Fish>pet).swim) {
  (<Fish>pet).swim()
}
```

类型别名

```ts
type LinkedList<T> = T & { next: LinkedList<T> }

interface Person {
    name: string
}

var people: LinkedList<Person>
var s = people.name
var s = people.next.name
var s = people.next.next.name
var s = people.next.next.next.name
```

```ts
type Easing = "ease-in" | "ease-out" | "ease-in-out"

class UIElement {
    animate(dx: number, dy: number, easing: Easing) {
        if (easing === "ease-in") {
            // ...
        }
        else if (easing === "ease-out") {
        }
        else if (easing === "ease-in-out") {
        }
        else {
            // error! should not pass null or undefined.
        }
    }
}

let button = new UIElement()
button.animate(0, 0, "ease-in")
button.animate(0, 0, "uneasy") // error: "uneasy" is not allowed here
```

可辨识联合

```ts
interface Square {
  kind: "square"
  size: number
}

interface Rectangle {
  kind: "rectangle"
  width: number
  height: number
}

interface Circle {
  kind: "circle"
  radius: number
}

type Shape = Square | Rectangle | Circle

function area(s: Shape) {
  switch (s.kind) {
      case "square": return s.size * s.size
      case "rectangle": return s.height * s.width
      case "circle": return Math.PI * s.radius ** 2
      default: return undefined
  }
}
```

多态 this 类型

```ts
class BasicCalculator {
  public constructor(protected value: number = 0) { }
  public currentValue(): number {
      return this.value
  }
  public add(operand: number): this {
      this.value += operand
      return this
  }
  public multiply(operand: number): this {
      this.value *= operand
      return this
  }
}

class ScientificCalculator extends BasicCalculator {
  public constructor(value = 0) {
      super(value)
  }
  public sin() {
      this.value = Math.sin(this.value)
      return this
  }
}

let v = new ScientificCalculator(2)
          .multiply(5)
          .sin()
          .add(1)
          .currentValue()
```

索引类型

```ts
function pluck<T, K extends keyof T>(o: T, names: K[]): T[K][] {
  return names.map(n => o[n])
}

interface Person {
    name: string;
    age: number;
}

let person: Person = {
    name: 'Jarid',
    age: 35
}

let strings: string[] = pluck(person, ['name'])
```

### 命名空间

```ts
namespace Validation {
  export interface StringValidator {
    isAcceptable(s: string): boolean
  }

  const lettersRegexp = /^[A-Za-z]+$/
  const numberRegexp = /^[0-9]+$/

  export class LettersOnlyValidator implements StringValidator {
      isAcceptable(s: string) {
        return lettersRegexp.test(s)
      }
  }

  export class ZipCodeValidator implements StringValidator {
      isAcceptable(s: string) {
        return s.length === 5 && numberRegexp.test(s)
      }
  }
}

// Some samples to try
let strings = ["Hello", "98052", "101"]

// Validators to use
let validators: { [s: string]: Validation.StringValidator } = {}
validators["ZIP code"] = new Validation.ZipCodeValidator()
validators["Letters only"] = new Validation.LettersOnlyValidator()

// Show whether each string passed each validator
for (let s of strings) {
  for (let name in validators) {
    console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`)
  }
}
```

Validation.ts

```ts
namespace Validation {
    export interface StringValidator {
      isAcceptable(s: string): boolean
    }
}
```

LettersOnlyValidator.ts

```ts
/// <reference path="Validation.ts" />
namespace Validation {
    const lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s)
        }
    }
}
```

ZipCodeValidator.ts

```ts
/// <reference path="Validation.ts" />
namespace Validation {
    const numberRegexp = /^[0-9]+$/
    export class ZipCodeValidator implements StringValidator {
        isAcceptable(s: string) {
            return s.length === 5 && numberRegexp.test(s)
        }
    }
}
```

Test.ts

```ts
/// <reference path="Validation.ts" />
/// <reference path="LettersOnlyValidator.ts" />
/// <reference path="ZipCodeValidator.ts" />

// Some samples to try
let strings = ["Hello", "98052", "101"]

// Validators to use
let validators: { [s: string]: Validation.StringValidator; } = {}
validators["ZIP code"] = new Validation.ZipCodeValidator()
validators["Letters only"] = new Validation.LettersOnlyValidator()

// Show whether each string passed each validator
for (let s of strings) {
    for (let name in validators) {
        console.log(`"${ s }" - ${ validators[name].isAcceptable(s) ? "matches" : "does not match" } ${ name }`)
    }
}
```

### 装饰器

*装饰器*是一种特殊类型的声明，它能够被附加到类声明、方法、访问符、属性或者参数上。

在 TypeScript 里，当多个装饰器应用在一个声明上时会进行如下操作：

1. 由上至下依次对装饰器表达式求值。
2. 求值的结果会被当作函数，由下至上依次调用。

```ts
function f() {
    console.log("f(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("f(): called");
    }
}

function g() {
    console.log("g(): evaluated");
    return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
        console.log("g(): called");
    }
}

class C {
    @f()
    @g()
    method() {}
}

/*
  f(): evaluated
  g(): evaluated
  g(): called
  f(): called
*/
```




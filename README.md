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

你可以使用 `readonly` 关键字将属性设置为只读的，只读属性必须在声明时或构造函数里被初始化。

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

```ts
// 创建一个泛型类
class Queue<T> {
  private data :T[] = [];
  push = (item: T) => this.data.push(item);
  pop = (): T | undefined => this.data.shift();
}

// 简单的使用
const queue = new Queue<number>();
queue.push(0);
queue.push('1'); // Error：不能推入一个 `string`，只有 number 类型被允许
```

配合 axios 使用

```ts
// 请求接口数据
export interface ResponseData<T = any> {
  /**
   * 状态码
   * @type { number }
   */
  code: number;

  /**
   * 数据
   * @type { T }
   */
  result: T;

  /**
   * 消息
   * @type { string }
   */
  message: string;
}
```

```ts
// 在 axios.ts 文件中对 axios 进行了处理，例如添加通用配置、拦截器等
import Ax from './axios';

import { ResponseData } from './interface.ts';

export function getUser<T>() {
  return Ax.get<ResponseData<T>>('/somepath')
    .then(res => res.data)
    .catch(err => console.error(err));
}
```

```ts
interface User {
  name: string;
  age: number;
}

async function test() {
  // user 被推断出为
  // {
  //  code: number,
  //  result: { name: string, age: number },
  //  message: string
  // }
  const user = await getUser<User>();
}
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
const enum Direction {
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

字符串字面量联合类型

```ts
// 用于创建字符串列表映射至 `K: V` 的函数
function strEnum<T extends string>(o: Array<T>): { [K in T]: K } {
  return o.reduce((res, key) => {
    res[key] = key
    return res
  }, Object.create(null))
}

// 创建 K: V
const Direction = strEnum(['North', 'South', 'East', 'West'])

// 创建一个类型
type Direction = keyof typeof Direction

// 简单的使用
let sample: Direction

sample = Direction.North // Okay
sample = 'North' // Okay
sample = 'AnythingElse' // ERROR!
```

keyof 使用

```ts
const data = {
  a: 3,
  hello: 'world'
}

function get(o: object, name: string) {
  return o[name]
}

// keyof 能够捕获一个类型的键
function get<T extends object, K extends keyof T>(o: T, name: K): T[K] {
  return o[name]
}
```

使用 is 判断值的类型

```ts
function isAxiosError (error: any): error is AxiosError {
  return error.isAxiosError
}

if (isAxiosError(err)) {
  code = `Axios-${err.code}`
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
  name: string
  age: number
}

let person: Person = {
  name: 'Jarid',
  age: 35
}

let strings: string[] = pluck(person, ['name'])
```

Freshness

```tsx
// 假设
interface State {
  foo?: string
  bar?: string
}

// 你可能想做
this.setState({ foo: 'Hello' }) // Yay works fine!

// 由于 Freshness，你也可以防止错别字
this.setState({ foos: 'Hello' }} // Error: 对象只能指定已知属性

// 仍然会有类型检查
this.setState({ foo: 123 }} // Error: 无法将 number 类型赋值给 string 类型
```

使用 redux

```ts
import { createStore } from 'redux'

type Action =
  | {
      type: 'INCREMENT'
    }
  | {
      type: 'DECREMENT'
    }

function counter(state = 0, action: Action) {
  switch (action.type) {
    case 'INCREMENT':
      return state + 1
    case 'DECREMENT':
      return state - 1
    default:
      return state
  }
}

let store = createStore(counter)

store.subscribe(() => console.log(store.getState()))

store.dispatch({ type: 'INCREMENT' })
// 1
store.dispatch({ type: 'INCREMENT' })
// 2
store.dispatch({ type: 'DECREMENT' })
// 1
```

索引签名

```ts
type Index = 'a' | 'b' | 'c'
type FromIndex = { [k in Index]?: number }

const good: FromIndex = { b: 1, c: 2 }

// Error:
// `{ b: 1, c: 2, d: 3 }` 不能分配给 'FromIndex'
// 对象字面量只能指定已知类型，'d' 不存在 'FromIndex' 类型上
const bad: FromIndex = { b: 1, c: 2, d: 3 }

// 变量的规则一般可以延迟被推断
type FromSomeIndex<K extends string> = { [key in K]: number }
```

索引签名的嵌套

```ts
interface NestedCSS {
  color?: string;
  nest?: {
    [selector: string]: NestedCSS;
  };
}

const example: NestedCSS = {
  color: 'red',
  nest: {
    '.subclass': {
      color: 'blue'
    }
  }
}

const failsSliently: NestedCSS {
  colour: 'red'  // TS Error: 未知属性 'colour'
}
```

捕获键的名称

```ts
const colors = {
  red: 'red',
  blue: 'blue'
};

type Colors = keyof typeof colors;

let color: Colors; // color 的类型是 'red' | 'blue'
color = 'red'; // ok
color = 'blue'; // ok
color = 'anythingElse'; // Error
```

混合 ( mixins )

```ts
// 所有 mixins 都需要
type Constructor<T = {}> = new (...args: any[]) => T

/////////////
// mixins 例子
////////////

// 添加属性的混合例子
function TimesTamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now()
  }
}

// 添加属性和方法的混合例子
function Activatable<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    isActivated = false

    activate() {
      this.isActivated = true
    }

    deactivate() {
      this.isActivated = false
    }
  }
}

///////////
// 组合类
///////////

// 简单的类
class User {
  name = ''
}

// 添加 TimesTamped 的 User
const TimestampedUser = TimesTamped(User)

// Tina TimesTamped 和 Activatable 的类
const TimestampedActivatableUser = TimesTamped(Activatable(User))

//////////
// 使用组合类
//////////

const timestampedUserExample = new TimestampedUser()
console.log(timestampedUserExample.timestamp)

const timestampedActivatableUserExample = new TimestampedActivatableUser()
console.log(timestampedActivatableUserExample.timestamp)
console.log(timestampedActivatableUserExample.isActivated)
```

thisType 使用

```ts
// Compile with --noImplicitThis

type Point = {
  x: number
  y: number
  moveBy(dx: number, dy: number): void
}

let p: Point = {
  x: 10,
  y: 20,
  moveBy(dx, dy) {
    this.x += dx // this has type Point
    this.y += dy // this has type Point
  }
}

let foo = {
  x: 'hello',
  f(n: number) {
    this // { x: string, f(n: number): void }
  }
}

let bar = {
  x: 'hello',
  f(this: { message: string }) {
    this // { message: string }
  }
}
```

```ts
// Compile with --noImplicitThis

type ObjectDescriptor<D, M> = {
  data?: D
  methods?: M & ThisType<D & M> // Type of 'this' in methods is D & M
}

function makeObject<D, M>(desc: ObjectDescriptor<D, M>): D & M {
  let data: object = desc.data || {}
  let methods: object = desc.methods || {}
  return { ...data, ...methods } as D & M
}

let obj = makeObject({
  data: { x: 0, y: 0 },
  methods: {
    moveBy(dx: number, dy: number) {
      this.x += dx // Strongly typed this
      this.y += dy // Strongly typed this
    }
  }
})

obj.x = 10
obj.y = 20
obj.moveBy(5, 5)
```

在上面的例子中，`makeObject` 参数中的对象属性 `methods` 具有包含 `ThisType<D & M>` 的上下文类型，因此对象中 `methods` 属性下的方法的 `this` 类型为 `{ x: number, y: number } & { moveBy(dx: number, dy: number): number }`。

`ThisType<T>` 的接口，在 `lib.d.ts` 只是被声明为空的接口，除了可以在对象字面量上下文中可以被识别以外，该接口的作用等同于任意空接口。

### 命名空间 

```ts
(function (Utility) {
  // 添加属性至 Utility
})(Utility || Utility = {})
```

```ts
namespace Utility {
  export function log(msg) {
    console.log(msg);
  }
  export function error(msg) {
    console.log(msg);
  }
}

// usage
Utility.log('Call me');
Utility.error('maybe');
  
```

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

装饰器是一种特殊类型的声明，它能够被附加到类声明、方法、访问符、属性或者参数上。

在 TypeScript 里，当多个装饰器应用在一个声明上时会进行如下操作：

1. 由上至下依次对装饰器表达式求值。
2. 求值的结果会被当作函数，由下至上依次调用。

```ts
function f() {
  console.log("f(): evaluated")
  return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("f(): called")
  }
}

function g() {
  console.log("g(): evaluated")
  return function (target, propertyKey: string, descriptor: PropertyDescriptor) {
    console.log("g(): called")
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

类装饰器

```ts
@sealed
class Greeter {
    greeting: string
    constructor(message: string) {
      this.greeting = message
    }
    greet() {
      return "Hello, " + this.greeting
    }
}

function sealed(constructor: Function) {
  Object.seal(constructor)
  Object.seal(constructor.prototype)
}
```

方法装饰器

```ts
class Greeter {
  greeting: string

  constructor(message: string) {
    this.greeting = message
  }

  @enumerable(false)
  greet() {
    return "Hello, " + this.greeting
  }
}

function enumerable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.enumerable = value
  }
}
```

访问器装饰器

```ts
class Point {
  private _x: number
  private _y: number
  
  constructor(x: number, y: number) {
    this._x = x
    this._y = y
  }

  @configurable(false)
  get x() { return this._x }

  @configurable(false)
  get y() { return this._y }
}

function configurable(value: boolean) {
  return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
    descriptor.configurable = value
  }
}
```

属性装饰器

```ts
import "reflect-metadata"

class Greeter {
  @format("Hello, %s")
  greeting: string

  constructor(message: string) {
    this.greeting = message
  }

  greet() {
    let formatString = getFormat(this, "greeting")
    return formatString.replace("%s", this.greeting)
  }
}

const formatMetadataKey = Symbol("format")

function format(formatString: string) {
  return Reflect.metadata(formatMetadataKey, formatString)
}

function getFormat(target: any, propertyKey: string) {
  return Reflect.getMetadata(formatMetadataKey, target, propertyKey)
}
```

元数据

```ts
import "reflect-metadata"

class Point {
  x: number
  y: number
}

class Line {
  private _p0: Point
  private _p1: Point

  @validate
  set p0(value: Point) { this._p0 = value }
  get p0() { return this._p0 }

  @validate
  set p1(value: Point) { this._p1 = value }
  get p1() { return this._p1 }
}

function validate<T>(target: any, propertyKey: string, descriptor: TypedPropertyDescriptor<T>) {
  let set = descriptor.set
  descriptor.set = function (value: T) {
    let type = Reflect.getMetadata("design:type", target, propertyKey)
    if (!(value instanceof type)) {
        throw new TypeError("Invalid type.")
    }
    set(value)
  }
}
```

控制反转与依赖注入

```ts
import "reflect-metadata"

type Constructor<T = any> = new (...args: any[]) => T

const Injectable = (): ClassDecorator => target => {}

class OtherService {
  a = 1
}

@Injectable()
class TestService {
  constructor(public readonly otherService: OtherService) {}

  testMethod() {
    console.log(this.otherService.a)
  }
}

const Factory = <T>(target: Constructor<T>): T => {
  // 获取所有注入的服务
  const providers = Reflect.getMetadata('design:paramtypes', target) // [OtherService]
  const args = providers.map((provider: Constructor) => new provider())
  return new target(...args)
}

Factory(TestService).testMethod() // 1
```

Controller 与 Get / Post 的实现

```ts
import "reflect-metadata"

const PATH_METADATA = 'path'
const METHOD_METADATA = 'method'

const Controller = (path: string): ClassDecorator => {
  return target => {
    Reflect.defineMetadata(PATH_METADATA, path, target)
  }
}

const createMappingDecorator = (method: string) => (path: string): MethodDecorator => {
  return (target, key, descriptor) => {
    Reflect.defineMetadata(PATH_METADATA, path, descriptor.value)
    Reflect.defineMetadata(METHOD_METADATA, method, descriptor.value)
  }
}

const Get = createMappingDecorator('GET')
const Post = createMappingDecorator('POST')

function mapRoute(instance: Object) {
  const prototype = Object.getPrototypeOf(instance)

  // 筛选出类的 methodName
  const methodsNames = Object.getOwnPropertyNames(prototype)
                              .filter(item => !isConstructor(item) && isFunction(prototype[item]))

  return methodsNames.map(methodName => {
    const fn = prototype[methodName]

    // 取出定义的 metadata
    const route = Reflect.getMetadata(PATH_METADATA, fn)
    const method = Reflect.getMetadata(METHOD_METADATA, fn)

    return {
      route,
      method,
      fn,
      methodName
    }
  })
}

@Controller('/test')
class SomeClass {
  @Get('/a')
  someGetMethod() {
    return 'hello world'
  }

  @Post('/b')
  somePostMethod() {}
}

Reflect.getMetadata(PATH_METADATA, SomeClass) // '/test'

mapRoute(new SomeClass())

/**
 * [{
 *    route: '/a',
 *    method: 'GET',
 *    fn: someGetMethod() { ... },
 *    methodName: 'someGetMethod'
 *  },{
 *    route: '/b',
 *    method: 'POST',
 *    fn: somePostMethod() { ... },
 *    methodName: 'somePostMethod'
 * }]
 *
 */
```


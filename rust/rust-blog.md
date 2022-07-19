# 生命周期的误解

## 引言

以下列出这篇文档里用到的名词和含义

| 名词                  | 含义                                                        |
| --------------------- | ----------------------------------------------------------- |
| `T`                   | 所有可能类型的`集合`，或`集合`中某一具体类型                |
| 所有权类型            | 某些非引用类型，其自身拥有所有权，如：String、i32、Vec 等等 |
| 借用类型 / 引用类型   | 引用类型，不考虑可变性，如：&i32，&mut i32等                |
| 可变引用 / 独占引用   | 独占可变引用，即 &mut T                                     |
| 不可变引用 / 共享引用 | 可共享不可变引用，即 &T                                     |

## 1. `T`只包含所有权类型

在 Rust 中，泛型与生命周期关系紧密，所以两个都要深入了解

初学者眼里，泛型是这样的：

| **类型** | `T`   | `&T`   | `&mut T`   |
| -------- | ----- | ------ | ---------- |
| **例子** | `i32` | `&i32` | `&mut i32` |

事实上泛型是这样的：

| **类型** | `T`                                            | `&T`                       | `&mut T`                               |
| -------- | ---------------------------------------------- | -------------------------- | -------------------------------------- |
| **例子** | i32，&i32，&mut i32，&&i32，&mut & mut i32 ... | &i32，&&i32，&&mut i32 ... | &mut i32，&mut &mut i32，&mut &i32 ... |

==`T` 是 `&T` 和 `&mut T` 的超集。`&T` 和 `&mut T` 是不相交的集合==，例子：

```rust
trait A {}
impl<T> A for T {}
impl<T> A for T {} // 报错，不可以二次实现
```

```rust
trait A {}
impl<T> A for T {}      // 因为 T 已经包含了 &T 和 &mut T，所以下面两行属于二次实现，会报错
impl<T> A for &T {}     // 报错
impl<T> A for &mut T {} // 报错
```

```rust
trait A {}
// impl<T> A for T {}      因为 &T 和 &mut T 是不相交的，所以下两行没问题
impl<T> A for &T {}     // OK
impl<T> A for &mut T {} // OK
```

## 2. 如果`T: 'static` 那么 `T` 直到程序结束前都有效

**错误的推论：**

- `T 'static` 是 `T` 有 `'static` 生命周期
- `&'static T` 和 `T: 'static` 是一回事儿
- 若 `T: 'static`，则 `T` 一定不可变
- 若 `T: 'static`，则 `T` 只能在编译其创建

```rust
// 初学者被告知，"xxx" 是被硬编码到二进制文件中的
// 并在运行时加载到内存，所以它不可变且在程序整个运行期间都有效
let s: &'static str = "xxx";
```

**关于静态变量：**

- 只能在编译时创建
- 不可变，修改静态变量是 unsafe 的
- 在整个程序运行期间有效

```rust
static VS: [i32;3] = [1,2,3];
static mut MUT_VS: [i32;3] = [4,5,6];

fn main() {
    println!("{:?}", VS); // [1,2,3]
    println!("{:?}", MUT_VS); // 报错：use of mutable static is unsafe and requires unsafe function or block

    MUT_VS[0] = 99; // 报错，修改要放在 unsafe 里
    unsafe {
        MUT_VS[0] == 99;          // 修改静态变量即便在 unsafe 下通过
        println!("{:?}", MUT_VS); // 实际上也是改不了的，结果：[4,5,6]
    }

    let mut s: &'static str = "aa";
    s = "haha"; // T: &'static 可以修改
    println!("{}", s) // 结果：haha
}
```

==带 `'static` 生命周期的类型和被`'static` 约束的类型是不一样的==：

- 带 `'static` 生命周期的类型：就是`static mut MUT_VS: [i32;3] = [4,5,6]`，符合上面 `关于静态变量` 的三点

- 被`'static` 约束的类型：就是 `let mut s: &'static str = "aa";`

  - 可以运行时被动态分配

  - 能被安全自由的修改

  - 可以被 drop，还能存活任意时长

  - **区分 `&'static T` 和 `T: 'static` 也非常重要**

    - `&'static T`：指向 `T` 的不可变引用，`T` 可以被无限持有，甚至直到程序结束（前提是保证 `T` 本身不可变，且创建后不被 move 走）

      ```rust
      // 在运行时返回 &'static str
      fn str_generator() -> &'static str {
          Box::leak("s".to_string().into_boxed_str()) // OK
          // 原本是 let s = rand::random::<u64>().to_string()
      }
      // 正常情况下要返回 &'static str 是会报错的
      fn str_generator() -> &'static str {
          "s".to_string().as_str() // 错误：cannot return value referencing temporary value
          "s" // 但 OK
      }
      ```

      ```rust
      // leak用法，先拿Box::new包一层
      fn foo() -> &'static str { Box::leak(Box::new("haha")) }
      println!("{}", foo()) // haha
      ```
    
    - `T: 'static`：==包含了全部 `&'static T`==同时，还包括了全部所有权类型（如：String, Vec 等）。应该被看成 `T` 满足 `'static` 生命周期的约束，而非拥有`'static` 生命周期
    
      ```rust
      fn drop_static<T: 'static>(t: T) { std::mem::drop(t) } // 满足 'static 的类型才能 drop
      
      fn main() {
          let mut vs: Vec<String> = Vec::new();
          for i in 0..10 {
              // 在运行时动态分配
              vs.push(i.to_string());
          }
      
          // 这些字符串是所有权类型，所以他们满足 'static 生命周期约束
          for mut s in vs {
              // 这些字符串是可变的
              s.push_str("a mutation");
              // 这些字符串都可以被 drop
              drop_static(s); // 编译通过
          }
      
          // 这些字符串在程序结束之前就已经全部失效了
          println!("{:?}", vs); // 所以报错：borrow of moved value: `vs`
      }
      ```

**总结**

- `T: 'static` 应当视为 `T` 满足 `'static` 生命周期约束
- 若 `T: 'static` 则 `T` 可以是一个有 `'static` 生命周期的引用类型 *或* 是一个所有权类型
- 因为 `T: 'static` 包括了所有权类型，所以 `T`
  - 可以在运行时动态分配
  - 不需要在整个程序运行期间都有效
  - 可以安全，自由地修改
  - 可以在运行时被动态的 drop
  - 可以有不同长度的生命周期

## 3. `&'a T` 和 `T: 'a` 是一回事

```bash
# 是上一个误解的泛型版本。
# T: 'a 包含了全体 &'a T，反之不成立
# 如果 T 不能在 'a 范围内有效，那么其引用也不能在 'a 范围内有效
# 例如，编译器不会构造一个 &'static Ref<'a, T>，因为 Ref 只在 'a 范围内有效，不能给它 'static 生命周期
```

```rust
fn t_ref<'a, T: 'a>(t: &'a T) {} // 只接受带有 'a 生命周期注解的引用类型
fn t_bound<'a, T: 'a>(t: T) {}   // 接受满足 'a 生命周期约束的任何类型
struct Ref<'a, T: 'a>(&'a T);    // 内部含有引用的所有权类型

fn main() {
    let string = String::from("string");

    t_bound(&string);       // 编译通过
    t_bound(Ref(&string));  // 编译通过
    t_bound(&Ref(&string)); // 编译通过

    t_ref(&string);       // 编译通过
    t_ref(Ref(&string));  // 编译失败，期望得到引用，实际得到 struct
    t_ref(&Ref(&string)); // 编译通过

    // 满足 'static 约束的字符串变量可以转换为 'a 约束
    t_bound(string); // 编译通过
}
```

```bash
# 回顾总结
# T: 'a 比 &'a T 更泛化，更灵活
# T: 'a 接受所有权类型，内部含有引用的所有权类型，和引用
# &'a T 只接受引用
# 若 T: 'static 则 T: 'a 因为对于所有 'a 都有 'static >= 'a
```

## 4. 我的代码里不含泛型和生命周期注解

生命周期省略规则和 `进阶.md 里的规则一样`

```rust
fn print(s: &str);        // 展开前
fn print<'a>(s: &'a str); // 展开后

fn trim(s: &str) -> &str;           // 展开前
fn trim<'a>(s: &'a str) -> &'a str; // 展开后

fn get_str() -> &str; // 非法，没有输入，不能确定返回值的生命周期
// 显式标注的方案
fn get_str<'a>() -> &'a str;  // 泛型版本
fn get_str() -> &'static str; // 'static 版本

fn overlap(s: &str, t: &str) -> &str; // 非法，多个输入，不能确定返回值的生命周期
// 显式标注（但仍有部分标注被省略）的方案
fn overlap<'a>(s: &'a str, t: &str) -> &'a str;    // 返回值的生命周期不长于 s
fn overlap<'a>(s: &str, t: &'a str) -> &'a str;    // 返回值的生命周期不长于 t
fn overlap<'a>(s: &'a str, t: &'a str) -> &'a str; // 返回值的生命周期不长于 s 且不长于 t
fn overlap(s: &str, t: &str) -> &'static str;      // 返回值的生命周期可以长于 s 或者 t
fn overlap<'a>(s: &str, t: &str) -> &'a str;       // 返回值的生命周期与输入无关
// 展开后
fn overlap<'a, 'b>(s: &'a str, t: &'b str) -> &'a str;
fn overlap<'a, 'b>(s: &'a str, t: &'b str) -> &'b str;
fn overlap<'a>(s: &'a str, t: &'a str) -> &'a str;
fn overlap<'a, 'b>(s: &'a str, t: &'b str) -> &'static str;
fn overlap<'a, 'b, 'c>(s: &'a str, t: &'b str) -> &'c str;

fn compare(&self, s: &str) -> &str; // 展开前
fn compare<'a, 'b>(&'a self, &'b str) -> &'a str; // 展开后
```

## 5. 编译通过，那么我标注的生命周期就是对的

**错误的推论**：

- Rust 对函数的生命周期省略规则总是对的
- Rust 的借用检查器总是正确的，无论是技巧上还是语义上
- Rust 比我更懂我程序的语义

## 6. 已装箱的 trait 对象不含生命周期注解

一个 trait 对象的生命周期约束从上下文推导而出

```rust
use std::cell::Ref;

trait Trait {}

type T1 = Box<dyn Trait>;           // 展开前
type T2 = Box<dyn Trait + 'static>; // 展开后，Box<T> 没有对 T 的生命周期约束，所以推导为 'static

impl dyn Trait {}           // 展开前
impl dyn Trait + 'static {} // 展开后

type T3<'a> = &'a dyn Trait;        // 展开前
type T4<'a> = &'a (dyn Trait + 'a); // 展开后，&'a T 要求 T: 'a, 所以推导为 'a

type T5<'a> = Ref<'a, dyn Trait>;      // 展开前
type T6<'a> = Ref<'a, dyn Trait + 'a>; // 展开后，Ref<'a, T> 要求 T: 'a, 所以推导为 'a

trait GenericTrait<'a>: 'a {}

type T7<'a> = Box<dyn GenericTrait<'a>>;      // 展开前
type T8<'a> = Box<dyn GenericTrait<'a> + 'a>; // 展开后

impl<'a> dyn GenericTrait<'a> {}      // 展开前
impl<'a> dyn GenericTrait<'a> + 'a {} // 展开后


struct Struct {}
impl Trait for Struct {}
impl Trait for &Struct {} // 直接为引用类型实现 Trait

struct Ref<'a, T>(&'a T);
impl<'a, T> Trait for Ref<'a, T> {} // 为包含引用的类型实现 Trait
```

## 7. 编译器的报错信息会告诉我怎样修复我的程序

**错误的推论**

- Rust 对 trait 对象的生命周期省略规则总是正确的
- Rust 比我更懂我程序的语义

**正确总结**

- Rust 对 trait 对象的生命周期省略规则并不保证在任何情况下都正确
- 在程序的语义方面，Rust 并不比你懂
- Rust 编译错误的提示信息所提出的修复方案并不一定能满足你对程序的需求

## 8. 生命周期可以在运行时动态变长或变短

**错误的推论**

- 容器类可以在运行时交换其内部的引用，从而改变自身的生命周期
- Rust 借用检查器能进行高级的控制流分析

**正确总结**

- 生命周期在编译时被静态确定
- 生命周期在运行时不能被改变
- Rust 借用检查器假设所有代码路径都能被执行，所以总是选择尽可能短的生命周期赋给变量

## 9. 将独占引用降级为共享引用是 safe 的

**错误的推论**

- 通过重借用引用内部的数据，能抹掉其原有的生命周期，然后赋一个新的上去

**正确总结**

- 尽量避免重借用一个独占引用为共享引用，不然你会遇到很多麻烦
- 重借用一个独占引用并不会结束其生命周期，哪怕它自身已经被 drop 掉了

# 收藏

```bash
https://github.com/pretzelhammer/rust-blog/tree/master/posts # 入口
https://github.com/pretzelhammer/rust-blog/blob/master/posts/translations/zh-hans/common-rust-lifetime-misconceptions.md # 目前这篇是中文
```
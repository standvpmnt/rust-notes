# RUST review sheet and notes

## String

The syntax `String::from()` indicates that the `from` function is being invoked from the namespaced String module? [not sure if module is the right word here]

String::from will have 3 things: pointer, length and capacity
As a memory model think of this as a step where the memory/space on the heap has to be allocated.

The `clone()` method is used to create copies of data on the heap.
If a type implements the `Copy` trait then the type is automatically copied upon being sent as arguments thus allowing the original variable to still be valid for use.
`Copy` trait is not implemented on anything which requires an allocation.

In certain scenario, especially when dealing with enums and structs there is a method to easily print (debug-print), clone and copy data instead of borrowing. To use that the following statement has to be added just before the declaration of the enum or struct - `#[derive(Debug, Clone, Copy)]`, also there is a `dbg!`

Additionally there are `#[derive(PartialEq)]` for equality and `PartialOrd` for greater and less than signs. Note: PartialOrd requires PartialEq to be implemented. These can be used as trait for struct as well as enumeration.

Enumerations are ordered by the variant themselves and not the data contained in them i.e. if enumerations contains a tuple, the data in the tuple is irrelevant, unless the comparison is between the same variant.

In structs, PartialOrd only the first field of the struct is used for comparison.
`std::cmp::Ordering` can be used to implement PartialOrd, especially used for comparing struct. This also allows using `.cmp()` which is also available if using `#[derive(Ord)]`

## Operator Overloading

Operator's can be overloaded (think defining operation for a datatype using impl trait) by using the `std::ops::<operator>` and running an impl on the data type for which the operation needs to be overloaded i.e. to overload `+` for a struct, `impl Add for <struct>` and in the impl define `fn add ....`.
For details look at the standard library documentation for ops, which has several operations eg. add, %, -, !, index, etc.
Iterating over a Struct
Iterator trait can be implemented for any data-structure by defining an `impl Iterator for <data-structure>` this will require the definition of `fn next(&mut self) -> Option<Self::data-type-of-data-structure-element>`, the definition of this will allow us to use `.next()`, `.map()`, `for...in`
Note this requires the ability to use borrow mutably the data-structure to be iterated over, if this is not possible then the alternate approach is to use `IntoIterator` trait instead which however requries the inner collection of the data-structure to be iterable i.e. either a Vector or a HashMap. `fn into_iter(self) -> Self::IntoIter;` needs to be implemented on the IntoIterator trait.
In case the underlying datastructure is not iterable, a pseudo/intermediary datastructure which borrows from this datastructure will have to be created by using either moving (convention for naming - IntoIter) or borrowing (convention Iter) by using `&'a`

_Integer Overflow_ Use function `checked_*`, `overflowing_*`, `saturating_*`, `wrapping_*`

`Option` is just an enum, and has several helper methods on the returned data like (refer to standard library for more combinators)

- `is_some()`
- `is_none()`
- `map(||)` which only runs when there is some data
- `filter(||)` and it must return a boolean which then implies the return type will be None if this returns false and borrowed data is used
- `or_else(||)` if there is no data this is run i.e. when the return is none it will set the returning value (option type again)
- `unwrap_or_else(||)` wherein the data is retrieved from the return of the option and if there is no data the default value in this is then provided in this closure.

`Result<T, E>` is also just an enum with `Ok(T)` and `Err(E)`
Closures are anonymous functions, `|vars| -> returnType { }`
`map` only runs when there is a value think `Option` that returns `Some` however, the return would still be of type `Option` and will require some sort of `match`.
`iter()` can be used to iterate over each item in ?(vec, HashMaps, ...),
the iter() is essentially like a plan as the exact procedures to be done during iteration are handled by chained methods like
`.map()`, `.collect()` which "collects" to a vector, `.filter()` and so on,
`.find()` which should return boolean however it will return an Option type,
`.count()`, `.last()` which also returns an Option,
`.min()`, `.max()` both of which return an Option type,
`.take(x)` will take the first x elements -> it needs to be seen once however whether the iterations is still done for the entire iterable or is it only done for the first x elements only,
`.next()` will return the next element good place to use it is while...let
Range `1..=3` will create a range 1 through 3 inclusive of 3, whereas `1..4` will create a range non-inclusive of 4

Modules `mod mod_name { .... }` and then this can be used by calling mod_name::fn_name or by using `use mod_name::*` or `use mod_name::fn_name`, imports i.e. use should be within the module to use them
A rust package can have multiple binary crates and _optionally_ one library crate. The syntax for creating a new library crate project is `cargo new --lib <project_name>`

Testing `#[cfg(test)] mod test { use crate::*; #[test] fn fn_name() { assert_eq!(value_to_check, expected_value, message)}}`, `crate::*;` will imply to pull in all of the functinality in the document. cargo test --help

Traits, this essentially lays down the set of traits (functions) that a struct would have to have in order to referenced/passed off as a type of `impl Trait_name`. The trait itself need not define the details of the function but instead it only needs to define the type and name of the function(s). By convention trait_name should always start with upper-case.

Generic function syntax - `fn function_name<T, U>(param1: T, param2: U) {}` here T & U automatically imply that a trait is implemented so `impl Trait_name` will not have to written every time to avoid verbosit. Additionally this can have the additional `where T: trait1 + trait2, U: trait2 + trait3 + trait4` statement before defining the function.
Similarly, there are Generic Structures which can be used to ensure data-types (traits) are available for the data of a struct, syntax can be like `struct Name<T: Trait1 + Trait2, U: Trait3> { field1: T, field2: U}`, or the where syntax can be used as indicated in the previous statement. The benefit with Generic Structs is that they can hold data of arbitrary types, think use-cases of this.

To put data on the heap, `variable = Box::new(data)` is used which is of Type `Box<data_type>`, to dereference the data, use `*variable` to get the data back on the stack.
Trait objects allow for dynamic dispatch behavior, to define a variable as a Trait object use the syntax `&dyn Type(Trait)` as the type and always set it equal to the borrowed value i.e. reference instead of the data itself. Alternately the type can also be defined using `Box<dyn Type>` and the value should be `Box::new(data)`
Note: Traits cannot access fields inside a structure, instead they can only access functions that they define/require themself.

## Lifetimes (data-structure existing before and after)

Lifetimes, read as tick-a `'a` is defined similar to the syntax of a generic to indicate that a specific data has a lifetime beyond the current code/block. `fn function_name<'a>(arg: &'a Datatype) -> &'a Datatype {}` this is usually figured out by the compiler. `'static` implies that the type will be available throughout the entirety of the running duration of the program.

## Error Handling (creating errors and using thiserror crate)

Error trait is a part of standard library, `use std::error::Error` then use `impl Error for 'ErrorType'`. The idea is to return the appropriate result with the correct ErrorType to ensure most user friendly debugging and error-messages. A popular crate that can be used to simplify this process is `use thiserror::Error` which has good interoperability with the standard library package. The enumeration of ErrorType should have annotations(macro) `#[error("Error Message")]` which will be displayed when that particular error occurs (display trait must be made available)
Additionally, thiserror crate also provides the ability to associate errors (ErrorTypes) by adding the annotation(macro) `(#[from] ErrorType2)` this helps to give more fine-grained error handling by breaking down errors. As a good practice you should try to keep error enums as specific as possible by creating as many as requried instead of one large type.

## Type State Pattern (Consuming resources i.e. data-structure etc.)

TypeState pattern is used to ensure resources are invalidated and consumed, to transition to another state and to disallow access to a missing resource. The idea is to use the data-structure itself instead of borrowing it, this will result in the old data structure being removed once the function is executed due to the concept of Ownership.

## Match Guards and Bindings

Use `@` in match statements to `bind` i.e. check value of a variable when matching on an enumeration, use `{}` when matching on structures, use the `..` syntax inside the braces to ignore other fields . In match expressions, `if` can be used as well Syntax is like `s if (s == 5 && diff == Difficulty::Easy)` where s and diff are just variables set to a number or enumerable respectively.

## Array and Slices

Use slice whenever possible as data-structures backed by an array will be able to provide a slice. syntax for using a slice is `&[u8]`. Use slice with match for flow-control, specifically in a match `[]` implies empty slice and so on which are always indicated by the compiler. Always match the largest pattern first followed by smaller patterns. Match guards can also be used.

## Type Aliases

Type aliases can be set with the syntax `type Callbacks = HashMap<String, Box<Fn(i32, i32) -> i32>>;` this can also be used with Generics and Lifetimes eg. `type GenericThing<T> = Vec<Thing<T>>;`

## Type Conversion

If type is explicitly defined, the `.into()` can be used for conversion ALERT: The type on which this is called needs to have a `From` trait which must have the `from` function implemented on it to successfully call `.into()`, else the compiler will throw the error. eg. `let owned: String = "slice".into()` will result in the same outcome as `let owned = String::from("slice");`. Further this is the same as `let owned = String::from("slice")` note the from is a method on the trait From which is implemented on the String type which results in the conversion (the logic for conversion is defined in the from method).
Perfer to create a From trait whenever possible instead of Into trait as From will automatically create the into. This also works with `?` operator.
If typer conversion is fallible then implement the `TryFrom` trait available from the standard library `std::convert::TryFrom`, this trait needs the function `try_from` to be implemented and note this function will return a `Result` type, so that needs to be checked as well.
`15u8 as u16`, will cast u8 to u16 (this higher type conversion is lossless)
Floating point values are clamped when converting to integers (saturating conversion)
`try_from` is implemented on all numeric types

## Closures

Type for closure is `Fn(args) -> resultType` when passing a closure to another function always pass it as a `Box<dyn Fn(arg1: T1, arg2: T2) -> resultType>`
Since a closure goes over the environment and picks out all variables in the environment, while using `Box::new(....)` to box the closure, the `move` keyword must be used to indicate the transfer of ownership. eg. In the below code name is from the environment above this specific statement.

```
Box::new(move |a, b| {
    println!("adding numbers for {}", name);
    a + b
})
```

## Threads

Threads have a `thread-local` memory, it is owned by the thread, data can be copied or moved to a thread when a thread is created and it becomes thread-local.
To spawn a thread, use the standard library thread module, eg. code is indicated below, note the type is explicitly defined in the block for reference only

```
use std::thread;
let handle: JoinHandle<type> = thread::spawn(move || {
    // code to be executed
});
handle.join();
```

## Channels

This provides one-way communication between threads. Channels guarantee in order delivery, and should ideally be used with an enum to ensure sent messages can be matched on the variants on the receiving end.
Use `crossbeam-channel` instead of the standard library for increased functionality.
Message Passing can be blocking or non-blocking and this is determined by the function call, not the channel. The block can be either on the Sender end i.e. Channel full or the block can be on the Receiver end i.e. No messages
As an example review the documenation of the `crossbeam_channel` crate which details the usage.
_Bi-directional threaded communication_
This is the strategy used to communicate back and forth between two threads, generally main and worker, by using the of identifying the sender and receiver `let (main_tx, main_rx) = ubounded()`
`try_send` can be used to avoid blocking while sending messages

Looping strategies for receiving messages are as follows:

1. While let

```
while let Ok(msg) = main_rx.recv() {
  // code for execution
}
```

2. Loop - using a simple `loop` with `break` with `worker_rx.recv()`

## Smart Pointers

Smart Pointers allow multiple owners of data, these can be either `Rc` i.e. reference counted (count of all owners of a data), or these can be `Arc` i.e. atomic reference counted which is the one that can be used with multiple threads.

### Rc

Rc can be used from the standard library, `use std::rc::Rc`, this can be used as `Rc::new(...);` we can use `Rc::clone(&dtype)` this clone command actually creates a new owner of the data and is not just borrowing.
Permanently mutable memory location can be used by using
_Cell_ available from the standard library wherein the Cell is mutable, using `.get()` and `.set(value)` on the cell to get the value of the mutable memory location. This requires data to be copy-able i.e. `#[derive(Clone, Copy)]`, this has a `new` trait which can be used to create a new cell.
_RefCell_ available from the standard library, this always borrows the data and is checked at runtime instead of compile time prefer using. This is not thread-safe so should be used on a single thread.
Note Prefer to use `&mut` instead of using cell or refcell
Other syntax to borrow besides `&` is `borrow_mut()` and `.borrow()`, `.try_borrow()` which will return Result type.

### Arc

Atomic Reference Counted Pointer, when working with threads and data, the important part is to ensure data synchronization to avoid data corruption (safe access), for this a `Mutex` is used which wraps the data making it mutually exclusive i.e. only one thread can access at a time. A mutex however, cannot be shared among thread for that it must be wrapped in a smart pointer `Arc`, syntax `std::sync::Arc`
Prefer to use `parking_lot` instead of the std lib for creating Mutex due to better API and performance. `parking_lot::Mutex`

Cases to consider

- deadlocks - recursive locks use `ReentrantMutex` which allows multiple locks on same thread
- contention - several threads trying to get a lock, use `try_lock()` or use backoff crate

syntax for an Arc type: `Arc<Mutex<dataType>>`, then use the `Arc::new(Mutex::new(dataType))` to create a new Arc, to use this arc and lock it use `.lock()` on this arc, to pass this to other threads use `Arc::clone(&<arc>)`.

What data structures are available natively in Rust?
What is `drop(sender)` when using channels? Is it mentioned in the crossbeam-channel crate documentation?
What is `usize`

## Helpful Information

Use `todo!` and `unimplemented!` macros, `unreachable!`, `env!`, `include_str!` and `include_bytes`
Turbofish `::< >` used when working with generics for determining the type
Loop Labels `'ident: loop {}` loop can be for...in. while... loops as well ident is just the identifier name to be labelled, useful when working with nested loops as they can be used with continue and break
Loop Expressions can have a break statement return a value `break 'label value`, this is only available on `loop` type-loops and not on other ones.
Struct Updation, `..` is equivalent to spread operator of JS however it implies & the rest, `..StructName::default()`
Raw Strings: `r"...."`, `r#"..".."#` keep adding `#` for as many # we need in the string.

## Async

Future & Executors

```
async fn fun() -> u32 {
  42
}

#[tokio::main]
pub async fn main() {
  let future = life();
  let value = future.await;

  let value: u32 = life().await;
}
```

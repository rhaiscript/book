Indexer as Property Access Fallback
===================================

{{#include ../links.md}}

```admonish tip.side "Tip: Property bag"

Such an [indexer] allows easy creation of _property bags_ (similar to [object maps])
which can dynamically add/remove properties.
```

An [indexer] taking a [string] index is a special case &ndash; it acts as a _fallback_ to property
[getters/setters].

During a property access, if the appropriate property [getter/setter][getters/setters] is not
defined, an [indexer] is called and passed the string name of the property.

This is also extremely useful as a short-hand for [indexers], when the [string] keys conform to
property name syntax.

```rust
// Assume 'obj' has an indexer defined with string parameters...

// Let's create a new key...
obj.hello_world = 42;

// The above is equivalent to this:
obj["hello_world"] = 42;

// You can write this...
let x = obj["hello_world"];

// but it is easier with this...
let x = obj.hello_world;
```

~~~admonish tip.small "Tip: Swizzling"

Since an [indexer] can serve as a _fallback_ to [property access][getters/setters],
it is possible to implement [swizzling](https://en.wikipedia.org/wiki/Swizzling_(computer_graphics))
of properties for use with vector-like [custom types].

Such an [indexer] defined on a [custom type] (for instance, `Float4`) can inspect the property name,
construct a proper return value based on the swizzle pattern, and return it.

```rust
// Assume 'v' is a 'Float4'
let r = v.w;        // -> v.w
let r = v.xx;       // -> Float2::new(v.x, v.x)
let r = v.yxz;      // -> Float3::new(v.y, v.x, v.z)
let r = v.xxzw;     // -> Float4::new(v.x, v.x, v.z, v.w)
let r = v.yyzzxx;   // error: property 'yyzzxx' not found
```
~~~


Caveat &ndash; Reverse is NOT True
----------------------------------

The reverse, however, is not true &ndash; when an [indexer] fails or doesn't exist, the corresponding
property [getter/setter][getters/setters], if any, is not called.

```rust
type MyType = HashMap<String, i64>;

let mut engine = Engine::new();

// Define custom type, property getter and string indexers
engine.register_type::<MyType>()
      .register_fn("new_ts", || {
          let mut obj = MyType::new();
          obj.insert("foo".to_string(), 1);
          obj.insert("bar".to_string(), 42);
          obj.insert("baz".to_string(), 123);
          obj
      })
      // Property 'hello'
      .register_get("hello", |obj: &mut MyType| obj.len() as i64)
      // Index getter/setter
      .register_indexer_get(|obj: &mut MyType, prop: &str| -> Result<i64, Box<EvalAltResult>>
          obj.get(index).cloned().ok_or_else(|| "not found".into())
      ).register_indexer_set(|obj: &mut MyType, prop: &str, value: i64|
          obj.insert(prop.to_string(), value)
      );

engine.run("let ts = new_ts(); print(ts.foo);");
//                                   ^^^^^^
//                 Calls ts["foo"] - getter for 'foo' does not exist

engine.run("let ts = new_ts(); print(ts.bar);");
//                                   ^^^^^^
//                 Calls ts["bar"] - getter for 'bar' does not exist

engine.run("let ts = new_ts(); ts.baz = 999;");
//                             ^^^^^^^^^^^^
//                 Calls ts["baz"] = 999 - setter for 'baz' does not exist

engine.run(r#"let ts = new_ts(); print(ts["hello"]);"#);
//                                     ^^^^^^^^^^^
//                 Error: Property getter for 'hello' not a fallback for indexer
```

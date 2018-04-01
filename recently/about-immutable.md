# Immutable.js

顾名思义，这是一个“不可变值”的库。Immutable 数据一旦创建，就不可更改。

考虑下面的操作。

```
> var obj = {prop: "obj's prop"};
< undefined
> var foo = obj;
< undefined
> foo.prop = "foo's prop";
< "foo's prop"
> obj.prop
< "foo's prop"
```

我们将obj的引用传递给了 foo，所以当 foo 修改 prop 属性时，我们会发现 obj 的 prop 属性也被修改了。

在小型应用中，在处理得当的情况下这样做可以节省内存，但是当应用的复杂度变高，这会造成非常大的隐患。

为了解决这个问题，当我们要把 obj 传给 foo 时，我们做一次深拷贝，具体做法就是遍历一次 obj 的所有属性，将值复制给 foo 对象。当 obj 拥有的属性越来越多，每次拷贝的成本也将越来越大。当我们拷贝对象，仅需要改变个别属性时，这将造成不必要的资源浪费。


（Editing）

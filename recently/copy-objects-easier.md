# JavaScript Copy Object

一次深拷贝是昂贵的，甚至在某些场合无法做到深拷贝。

我们可以减轻拷贝的成本，通过仅拷贝改变的，复用不变的方式可以做到这一点。

看看这个链接。或许派得上用场。

https://github.com/kolodny/immutability-helper

这个库的 API 参照了 MongoDB 的查询语言，如果对 MongoDB 不熟悉的话稍稍有些麻烦。

https://github.com/substantial/updeep

这是一个 API 比较友好的库，容易上手，没啥门槛，但这个库依赖 Lodash。

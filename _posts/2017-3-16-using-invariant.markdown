---
layout: post
title: invariant & babel-plugin-dev-expression
---

# invariant和babel-plugin-dev-expression

~~~
var invariant = require('invariant');

invariant(someTruthyVal, 'This will not throw');
// No errors

invariant(someFalseyVal, 'This will throw an error with this message');
// Error: Invariant Violation: This will throw an error with this message
~~~

当条件通过时不做任何处理，不通过便抛出new Error(messages)。
当在`process.env.NODE_ENV = 'development'`时, invariant强制需要第二个参数,当`process.env.NODE_ENV = 'production'` 后面的信息参数为可选，如果不填写，当条件不通过时会抛出下面的错误。

~~~
  Minified exception occurred; use the non-minified dev environment 
  for the full error message and additional helpful warnings.
~~~

babel-plugin-dev-expression会进行如下转换

~~~
invariant(condition, argument, argument);
// ->
if (!condition) {
  if ("production" !== process.env.NODE_ENV) {
    invariant(false, argument, argument);
  } else {
    invariant(false);
  }
}
~~~

这样虽然可以隐藏调试信息，但是却占用了资源控件。

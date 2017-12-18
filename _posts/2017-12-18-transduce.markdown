---
layout: post
title: transduce，高级抽象foldable
---


Ramdav0.12.0引入了transduce, 主要是解决了reduce中函数复用的问题，是一个更加抽象的reduce。

通常情况下我们会使用map,filter,reduce进行列表处理.

```(javascript)
     const numbers = [1, 2, 3, 4, 5];

     const isOdd = x => x%2 === 1;
     const addOne = x => x + 1;
     const accF = (acc, x) => acc + x

     numbers
         .filter(isOdd)
         .map(addOne)
         .reduce(accF, 0)
```

在函数式中为了能够复用代码逻辑可能写成下面这样

```(javascript)
    const transform = compose(reduce(fold, 0), map(addOne), filter(isOdd));
    transform(numbers)
```

但是由于map,filter,reduce都会进行一次列表处理，无法在一次列表处理中完成所有功能，性能上并不能达到我们的要求。

## combine map & filter

为了化简成一次列表操作, 我们需要将map,filter,reduce写成一个统一的形式(这样才能产生逻辑组合的可能)。根据以往的经验我们知道map和filter可以写成reduce的实现。

```(javascript)
const mapReducer = f => (acc, x) => {
    const value = f(x);
    const result = concat(acc, x);
    return result;
}

const filterReducer = p => (acc, x) => {
    const result = p(x) ? concat(acc, x) : acc;
    return result;
}

numbers
    .reduce(filterReducer(isOdd), [])
    .reduce(mapReducer(addOne), [])
    .reduce(accF, 0)
```



将其中的折叠操作抽出来(fold function), 折叠函数是reduce执行的一个必要环节, 主要用来进行累积操作。

```(javascript)
const concat = (acc, x) => acc.concat(x);

const mapping = f => fold => (acc, x) => {
    const value = f(x);
    const result = fold(acc, x);
    return result;
}

const filtering = f => fold => (acc, x) => {
    const result = p(x) ? fold(acc, x) : acc;
    return result;
}

numbers
    .reduce(filtering(isOdd, concat), [])
    .reduce(mapping(addOne, concat), [])
    .reduce(accF, 0)
```

上面代码中的 mapping, filtering 折叠操作的函数指纹如下

```
    fold = (acc, value) => newacc
```

另外也可以看到mapping,filtering的尾部也满足这个函数指纹`(acc,x) => newacc`, 为了方便我们定义一个类型表示fold类型 FoldFunction, 用下面的伪代码重新描述mapping, filtering

```
mapping = f => FoldFunction => FoldFunction
filtering = p => FoldFunction => FoldFunction
```

所以此时

```
mapping(addOne, concat) 可当成一个新FoldFunction
filtering(isOdd, concat) 也可以当成一个新FoldFunction
```

这样就给了我们组合mapping, filtering的可能。

```(javascript)
    numbers
        .reduce(mapping(
            addOne,
            filtering(isOdd, concat)), [])
        .reduce(accF, 0)

```

再次化简

```(javascript)
    numbers
        .reduce(
            compose(mapping(addOne), filter(isOdd))(concat), []
        )
        .reduce(accF, 0)
```

逻辑上等价于

```(javascript)
    numbers.reduce(
        compose(mapping(addOne), filter(isOdd))(accF),
        0
    )
```

此时化简结束，在复用原来函数的基础上(addOne, isOdd)将map,filter,reduce成功简化成一次列表操作.

此时的这里的reduce形式为了更加方便人们抽象出了`transduce`函数

```
  transduce(
    compose(mapping(addOne), filter(isOdd)),
    accF,
    0,
    numbers
  )
```

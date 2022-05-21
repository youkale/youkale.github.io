+++
date = 2021-10-21T12:32:29+08:00
title = "clojure支持参数重载的宏"
authors = ["Youkale"]
tags = ["clojure"]
categories = []
series = []
+++

## 问题
> 通过clojure.core/definline可以以方法的形态定义一个宏，但是`definline`这个宏却只支持固定的参数，在某些情况下需要支持不同参数的宏，从`java`的角度来说就是方法重载了。


## 支持重载的`definline`
> 大部分代码都是在definline的基础上魔改的。

```clojure 
(defmacro definline+
   [name & decl]
     (let [body (map (fn [[args# expr#]]
                      (let [f# (apply (eval (list `fn (list args# expr#))) args#)]
                                                                 `(~args# ~f#))) decl)
             alt-body (map (fn [[args# expr#]]
                `(~args# ~expr#)) decl)]
                `(do
                   (defn ~name ~@body)
                   (alter-meta! (var ~name) assoc :inline (fn ~name ~@alt-body))
                                        (var ~name))))])))]))
```

### 使用
> 比如`noop` 方法，现在可以支持2、3个参数了。 

```clojure
(definline+ noop
 ([a b]
  (or ~a ~b))
 ([a b c]
  (or ~a ~b ~c)))

```


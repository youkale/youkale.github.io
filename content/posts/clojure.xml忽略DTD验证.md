+++
date = 2017-05-21T12:32:29+08:00
title = "clojure.xml忽略DTD验证"
authors = []
tags = ["clojure", "xml"]
categories = []
series = []
+++

## 问题
> 在通过clojure生成mybatis mapper文件的过程中，发现启动过程缓慢，原因是会验证xml的schema，因此解决方法就是屏蔽掉DTD的验证

## clojure.xml API没有DTD写入解决:


```clojure
  (defn emit-xml
  "Prints the given Element tree as XML text to stream.
   Options:
    :encoding <str>          Character encoding to use"
  [e & {:as opts}]
  (let [stream (StringWriter.)
        ^XMLStreamWriter writer (-> (XMLOutputFactory/newInstance)
                                    (.createXMLStreamWriter stream))]

    (when (instance? OutputStreamWriter stream)
      (check-stream-encoding stream (or (:encoding opts) "UTF-8")))

    (.writeStartDocument writer (or (:encoding opts) "UTF-8") "1.0")
    (.writeDTD writer "<!DOCTYPE mapper PUBLIC \"-//mybatis.org//DTD Mapper 3.0//EN\" \"http://mybatis.org/dtd/mybatis-3-mapper.dtd\">")
    (doseq [event (flatten-elements [e])]
      (emit-event event writer))
    (.writeEndDocument writer)
    (.toString stream)))
```
  
## 忽略DTD远程schema验证

```clojure
(parse-str xf
      :supporting-external-entities false
      :validating false
      :support-dtd false)
```




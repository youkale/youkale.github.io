---
title: Jetty嵌入开发路径问题
date: 2016-10-28 13:32:00
tag: java
---

> 我都知道在使用jetty在嵌入开发的时候使用ContextHandler，通过 setContextPath设置访问路径，通过setHandler来使用自定义的handler.最终设置到Server中，服务就可以跑起来了。

### 官方的栗子

```
       Server server = new Server(8080);
        ContextHandler context = new ContextHandler("/");
        context.setContextPath("/");
        context.setHandler(new HelloHandler("Root Hello"));

        ContextHandler contextFR = new ContextHandler("/fr");
        contextFR.setHandler(new HelloHandler("Bonjoir"));

        ContextHandler contextIT = new ContextHandler("/it");
        contextIT.setHandler(new HelloHandler("Bongiorno"));

        ContextHandler contextV = new ContextHandler("/");
        contextV.setVirtualHosts(new String[] { "127.0.0.2" });
        contextV.setHandler(new HelloHandler("Virtual Hello"));
        contextV.setInitParameter("dirAllowed", "false");

        ContextHandlerCollection contexts = new ContextHandlerCollection();
        contexts.setHandlers(new Handler[] { context, contextFR, contextIT,
                contextV });

        server.setHandler(contexts);

        server.start();
        server.join();

```

### 开始

细心的朋友可能在使用中发现，明明我定义的路径是/user ,但是在访问的时候却进行了一次302跳转，然后最终变成了/user/ ,经过调试发现,在ContextHandler这个类中发现下面这个有趣的问题

```bash
        // Are we not the root context?
        // redirect null path infos
        if (!_allowNullPathInfo && _contextPath.length() == target.length() && _contextPath.length()>1)
        {
            // context request must end with /
            baseRequest.setHandled(true);
            if (baseRequest.getQueryString() != null)
                response.sendRedirect(URIUtil.addPaths(baseRequest.getRequestURI(),URIUtil.SLASH) + "?" + baseRequest.getQueryString());
            else
                response.sendRedirect(URIUtil.addPaths(baseRequest.getRequestURI(),URIUtil.SLASH));
            return false;
        }

```

注释已经写了,context request must end with /
原来jetty会自动得帮你把在请求后面加一个/  ，然后再跳转到你的路径上面,于是  /user 的访问就变成了 /user/

### 最后

 那么如何解决这个问题呢？很简单，只需要调用 setAllowNullPathInfo 这个方法，并且设置为 true 即可
那么jetty 路径自动增加/的问题，也就是jetty自动302的问题就算解决啦!!

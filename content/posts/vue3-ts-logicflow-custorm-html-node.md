+++
date = 2022-03-21T12:32:29+08:00
title = "Vue3 Typescript LogicFlow 通过H函数自定义HTML节点"
authors = ["Youkale"]
tags = ["vue3", "ts", "logicflow"]
categories = []
series = []
+++

## 问题
> 在使用DiDi的LogicFlow中，因为使用的是最新的VUE3+TS，官方没有相关的文档，在自定义`HTML Node`的时候碰到了一下问题，最终通过翻阅VUE3的文档找到了如下的解决方法

```vue

import { HtmlNode, HtmlNodeModel } from '@logicflow/core';
import { defineComponent, h, render as vueRender } from 'vue';
import { Button, Card } from 'ant-design-vue';

const myHtmlNode = defineComponent({
      name: 'MyHtmlNode',
        components: { 
            'a-button': Button,  //需要重新引入组件
                'a-card': Card,      //需要重新引入组件
                  },
                    template: `
                        <div>
                         <a-button type="primary" @click="handle('plus')">+</a-button>
                         <a-button type="dashed" @click="handle('sub') ">-</a-button>
                        </div> `,
       setup() 
         const _handleClick = (action) => {
         console.log(`action ....${action}`);
       };
       return {
          handle: _handleClick,
       };
                                          },
});


class PluginNode extends HtmlNode {
  setHtml(rootEl: HTMLElement) {
      const vm = h(myHtmlNode, {});
      vueRender(vm, rootEl);
    }
}

```



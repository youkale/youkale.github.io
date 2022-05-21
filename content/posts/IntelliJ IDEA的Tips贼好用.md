---
title: IntelliJ IDEA的Tips贼好用
date: 2018-08-26 13:54:43
tag:
---

# 废话

>IntelliJ IDEA功能强大,用过得到的朋友也都知道,可能你认为一个新的工具同样的也会带来相应的学习成本。其实没有想象的那么复杂，只要关心idea给你提示的黄色警告即可， 然后用 alt + enter 使用更好的方式重构即可. 本文没有上面实质内容，只是在安利大家用IDEA, 把代码写好，少加班

## Lambda

转换前

    ExecutorService executorService = Executors.newCachedThreadPool();
            executorService.submit(new Runnable() {
                @Override
                public void run() {
                    System.out.println("xxx");
                }
            });

转换后

    public void runJob() {
        ExecutorService executorService = Executors.newCachedThreadPool();
        executorService.submit(() -> System.out.println("xxx"));
    }

## Stream

转换前

    public boolean contains(List<Person> ps) {
        return ps.stream().filter(s -> s.getName().equals("youkale")).findAny().isPresent();
    }

转换后

    public boolean contains(List<Person> ps) {
        return ps.stream().anyMatch(s -> s.getName().equals("youkale"));
    }

## 异常

转换前

    try{
        ....
        }catch (IOException ioe){

        }catch (FileNotFoundException fe){

        }

转换后

    try{

    }catch(IOException | FileNotFoundException e){

    }

## 最后

>还是要强调: 只要关注IDEA给你警告，即使你想把代码写得像一坨屎，它也能帮你找出问题所在，然后给你警告,然后提示,最后还直接帮你把代码改好。 再安利一个快捷键 F2, 帮你查找错误和警告

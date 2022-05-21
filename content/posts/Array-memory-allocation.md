---
title: 数组内存分配
date: 2017-05-20 18:55:00
tag: golang
---

### 废话
  今天是所谓的520，高富帅此刻正在xxx，我等屌丝在写博客，悲哀呀。。

### 如题

```
  //定义一个容量为5,长度为4的数组
	slice := [5]int{10, 20, 30, 40}

	//cap = 5 - 1
	//len = 3 -1
	//一个新的切片
	newSlice := slice[1:3]

	//增加两个元素
       //这些操作其实是在原有的数组上进行操作
      //因为cap都在这个原有的数组范围内
	newSlice = append(newSlice, 50, 60)

	fmt.Printf("slice cap %d len %d, newSlice cap %d len %d \n", cap(slice), len(slice), cap(newSlice), len(newSlice))

	//遍历并打印出指针地址
	for i1, v1 := range slice {
		fmt.Println(i1, v1, &slice[i1])
	}

	fmt.Printf("slice cap %d len %d, newSlice cap %d len %d \n", cap(slice), len(slice), cap(newSlice), len(newSlice))
	//遍历并打印出指针地址
	for i2, v2 := range newSlice {
		fmt.Println(i2, v2, &newSlice[i2])
	}


	//重新增加元素
       //此时已经超出了原有的数组容量，那么会重新copy数组
      //所以在性能要求高的场景谨慎使用append

	newSlice = append(newSlice, 20, 60, 90)

	fmt.Printf("slice cap %d len %d, newSlice cap %d len %d \n", cap(slice), len(slice), cap(newSlice), len(newSlice))
	//遍历并打印出指针地址
	for i3, v3 := range newSlice {
		fmt.Println(i3, v3, &newSlice[i3])
	}
```
### 输出结果

```
slice cap 5 len 5, newSlice cap 4 len 4 
0 10 0xc820012270
1 20 0xc820012278
2 30 0xc820012280
3 50 0xc820012288
4 60 0xc820012290
slice cap 5 len 5, newSlice cap 4 len 4 
0 20 0xc820012278
1 30 0xc820012280
2 50 0xc820012288
3 60 0xc820012290
slice cap 5 len 5, newSlice cap 8 len 7 
0 20 0xc820016340
1 30 0xc820016348
2 50 0xc820016350
3 60 0xc820016358
4 20 0xc820016360
5 60 0xc820016368
6 90 0xc820016370
```



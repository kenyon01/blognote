---
layout: post
title: 【Rust7】collections
categories: 语言
tags:
keywords:
description:
order: 11207
---


## collections

- `Vec<T>`
- `VecDeque<T>`：双端队列。
    - 实现：环形队列，并且可重新分配 capacity
    - 在队头和队尾的插入/删除都是 O(1)
    - 其它操作变慢
- `LinkedList<T>`：双向链表
- `BinaryHeap` 最大堆，（相当于Python的 heapq）
    - 适合查找和移除最大值
- `HashMap`
- `BTreeMap` 有序 kv表
    - 按照 k 顺序存储
    - 大多操作比 HashMap 慢
- `HashSet`
- `BTreeSet` 有序的 set


## Vec

```Rust
// 如何创建1
let mut v1: Vec<i32> = Vec::new(); // 不推荐
let mut v1: Vec<i32> = Vec::with_capacity(128); //推荐


// 如何创建2
let v2 = vec![1, 2, 3];

// 增
v1.push(5); // 会取走所有权
vec1.insert(0, 9); // 在0 位置插入9
vec1.extend(&vec2); // 把另一个塞进去
vec1.append(&mut vec2); // 把vec2塞进去，vec2清空

// 删
v1.pop(); // 删除并返回最后一个，如果 v1 是空的，返回 None。取得所有权
vec1.remove(2); // 删除2位置

// 改
v1[1] = 9;
println!("{:?}", v1);

// 查
let a = v1.get(0); // 返回引用 Some(&val)
// Some 类型
// index 超出的话，为 None 类型
let a2 = v1.get(100);

let b = v1[0]; // 实现 Copy
let c = &v1[0];
println!("{:?},{:?},{},{}", a, a2, b, c);


v1.push(30);
// println!("{:?}", a); // 上面 push 时被借用了，所以这里会报错


println!("{:?}", v1);

// 循环遍历
for i in v2 {
    println!("{}", i);
}

// 如何在循环遍历中改它们
for i in &mut v1 {
    *i += 50;
}
println!("{:?}", v1);
```

内存相关
```Rust
// 创建容量为n的vec
Vec::with_capacity(n);
// 返回容量，usize
v.capacity()
// 确保空闲空间容纳额外n个元素，也就是保证多于 vec.len() + n，有时候会多分配一些
v.reserve(n)
// 确保空闲空间为精确的 n，vec.capacity() = vec.len() + n，不会自作聪明多分配
v.reserve_exact(n);
// 丢掉所有空闲空间
v.shrink_to_fit();
```


裁剪类
```Rust
//用 val 填充，直到长度为 new_len
v.resize(new_len, val);
// 裁剪，使得长度为 new_len
v.truncate(new_len);
// 相当于v.truncate(0);
v.clear();
// truncate，并且把裁剪掉的返回成新的 Vec
let v_part2 = v.split_off(new_len);

// 下面的都是返回引用
iter,iter_mut
split_at(idx);split_at_mut
split_first();split_first_mut();
split_last();split_last_mut();
split(func_is_sep);split_mut(func_is_sep); // 基于函数，拆分为多个子切片，返回迭代器
split_splitn(n,func_is_sep);split_splitn_mut(n,func_is_sep) // 效果同上，超过n个的，会把剩余的放到第n个里面
rsplit;rsplit_mut;rsplitn; r_split_mut;  //效果同 split 系列，但是迭代器是从右向左的

v.windows(n) // 迭代器，第一个是 v[0..n]，第二个是 v[1..n+1]，直到 v[(len-n)..len]
```

```
// 清除相邻的重复元素
vec.dedup()
// 功能类似，但判断标准是一个匿名函数
vec.dedup_by(func)
// 例如：
v1.dedup_by(|elem1, elem2| (*elem1 - *elem2).abs() <= 1);

// 功能类似，但判断标准是 func(elem1) == func(elem2)
vec.dedup_by_key(func)
```

合并
```Rust
// concat：发生所有权转移
let v1: Vec<i32> = vec![1, 2];
let v2: Vec<i32> = vec![3, 2, 9];
let v3 = vec![3, 2];

// 合并1
let v_concat = [v1, v2, v3].concat();
// 合并2
[v1, v2, v3].join(&0)
```


其它
```Rust
v.swap(idx1, idx2); // 交换元素

// 移除，但不是移动内存，而是把最后一个元素移动到最前面
// 在不关心顺序的情况下，性能很高
v2.swap_remove(idx);

// 排序
v.sort()
v.sort_by(func)
v.sort_by_key(func_key) // func_key 计算出 key值，然后按照key值排序，key值不缓存，所以可能被多次调用
// 如何反向排序？加一个 reverse


//对排序后的结果，可以做二分法
v.binary_search();
v.binary_search_by(cmp_func);
v.binary_search_by_key(key_func);


// 包含关系
v.contains(val);

// 两个序列的包含关系，都返回 bool
v1.starts_with(&v2);
v1.ends_with(&v2);
```



案例：
```Rust
// 去重案例
let mut v1 = vec![1, 2, 3, 2, 1];
let mut tmp = HashSet::new();
v1.retain(|r| tmp.insert(*r));
// 1. retain 相当于 .into_iter().filter()
// 2. HashSet 在插入已有的值时，会返回 false


```


## VecDeque

基于环形队列的双端array,
- 实现：环形队列，并且可重新分配 capacity
- 在队头和队尾的插入/删除都是 O(1)
- 其它操作变慢
- 不能切片

```Rust
let mut deque =VecDeque::new();

deque.push_back(6);
deque.push_front(3);
let val = deque.pop_back();
let val = deque.pop_front();

// 返回头/尾的引用
deque.front(); deque.front_mut();
deque.back(); deque.back_mut();
```

## LinkedList

```Rust
linked_list.append(list2)
```

## BinaryHeap

- 元素必须实现了 Ord


```
BianayHeap::new()
heap.push(val)
heap.pop()
heap.peek()

heap.len()
heap.is_empty()
heap.capacity()
heap.clear()
heap.append(heap2)
```


通常用 while 消费

```
while let Some(task)==heap.pop(){
  do(task);
}
```

heap 有自己的 iter




## String

```Rust
// 如何创建1
let mut str1: String = String::new();
let str1 = String::with_capacity(n);


// 如何创建2
let s1: &str = "hello, world!";
let str1: String = s2_1.to_string();

// 如何创建3
let s3: String = String::from("hello");
// 如何转回去
let s_str: &str = &*s3;


// 添加
s1.push('a');
let s_append = "abb";
s1.push_str(s_append); // 不获取所有权
str1.extend(str2.chars()); // 参数是一个 Iterator
str1.insert(1, 'd'); // 指定位置插入
str1.insert_str(1, &*str2); // 指定位置插入
str1 + &str2; // str1 的所有权被移动

// 删除
str1.shrink_to_fit(); // 清除未用内存
str1.shrink_to(10); // 内存尽可能缩减到10
str1.clear(); // 清除变为空字符串
str1.truncate(n)
str1.pop();
str1.remove(0); // 删除，并且后面的字符往前移动
str1.drain(); 跟其他的一样





println!("{}", s1);

// 用 format连接
let s_total: String = format!("{}-{}-{}", s1, s2, s3);
println!("{}", s_total);
```


索引
- String实际上是 `Vec<u8>`，所以 `len()` 对应的是 Vec 的长度
- 不要用 `&s[0..9]` 或者 `s[0]`，遇到中文会 panic


遍历字符串

```rust
let s = String::from("你好rust语言");

// 遍历字符
for i in s.chars() {
    println!("{}", i);
}

// 这个是遍历 ascii 值，都在 0～255 之间
for i in s.bytes() {
    println!("{}", i);
}

// 相当于 enumerate
str1.char_indices()

// 文本行迭代器，按照\n或者 \r\n 分割
str1.lines()

str1.split(func);str1.rsplit(func);
str1.splitn(n, func);str1.rsplitn(n, func);

// 寻找
str1.matches(func);
str1.rmatchs(func);
str1.match_indices(func);
str1.rmatch_indices(func);

// trim
str1.trim() // 返回切片
str1.trim_left();str1.trim_right();
str1.trim_matches(func);
str1.trim_left_matches(func);
str1.trim_right_matches(func);

// Iterator
let str_new: String = str1.chars().filter(|c| c.is_uppercase()).collect();

```



不同格式的字符串
- `String`类型，看上面。
- `&[u8]`： `let my_str_u: &[u8] = b"hello";`
- `&str`，字符串切片 : `let my_str: &str = "hello";`

相互转换：
```Rust
// String::from_utf8(); // 返回 Ok(string)，取得所有权
String::from_utf8_unchecked() // 要求必须是格式良好的，只能在unsafe中使用
String::from_utf8_lossy(my_str_u).to_string();
String::from(my_str);
// 或者 my_str.to_string
// 或者 用 format!()，建立新的 String


<String>.as_str()
<String>.as_bytes() // 格式良好的 UTF-8
<String>.into_bytes() // 允许格式不良的 UTF-8


let new_my_str_u: &[u8] = my_str.as_bytes();

// &[u8] 转  &str 不知道咋转
```

字符串字面量


```Rust
// 1. 前面加r，使得转义字符不转义
let str1 = r"C:\files";

// 2. 前面加b，是 `[u8]` 切片类型

// 3. 也可以前面加 br


// 4. 多行
let str1 = r###"多行文本
用 'r###"'开头
"###;

// 5. 支持比较运算符 `<,<=,>,>=,==,!=`，
"abc" > "aar"
```

其它
```Rust
str1.len()
str1.is_empty()
str1.split_at()


str1.contrains(func);
str1.starts_with(func);
str1.ends_with(func);
str1.find(func);
str1.rfind();
str1.replace(|c| c == 'a', "bcd");
replacen // 功能同上，但最多替换前n个




// 好像用处不大
slice.split_terminator(pattern);slice.rsplit_terminator(pattern)
slice.split_whitespace();
```



### 与hex的互相转

```Rust
// data-encoding = "*"

use data_encoding::{HEXUPPER, DecodeError};

let my_string=String::from("一段字符串");
// 转 hex
let my_string_hex: String = HEXUPPER.encode(my_string.as_bytes());
println!("{:?}", my_string_hex);


// hex 转字符串
let my_string_back_byte: Vec<u8> = HEXUPPER.decode(my_string_hex.as_bytes()).unwrap();
let my_string_back: String = String::from_utf8_lossy(&my_string_back_byte).to_string();
println!("{:?}", my_string_back);
```

### 与 base64 互转

```Rust
use base64;

// 转base64
let my_string = String::from("一段字符串");
let my_string_base64: String = base64::encode(&my_string);

// 转回来
let my_string_back_vec: Vec<u8> = base64::decode(&my_string_base64).unwrap();
let my_string_back = Sring::from(my_string_back_vec);
```


### 编码

UTF-8 的特点
- 一个字符占用 1～4个字节
- ASCII 码不变性
- 通过任意字节的前几位，可以确定它是首字节还是中间字节
- 通过首字节的前几位，可以确定编码长度
- 字符串首字节可以指定文字方向是从左到右还是从右到左


```Rust
ch.is_numeric() // 数值类，还包括 ², ⅓ 这种
ch.is_alphabetic() // 字母，也包括 unicode 重的派生字符
ch.is_alphanumeric() // 字母+数值，以上两个的集合
ch.is_whitespace() // 空格，包括 unicode 派生
ch.is_control() // 控制符，包括 unicode 派生

ch.is_lowercase()
ch.is_uppercase()
ch.to_lowercase()
ch.to_uppercase()

// ascii值
'a' as i32 // 字符转 ascii 值
98 as char // ascii 值转字符
```

```Rust
ch.is_digit(radix) // 是否是 radix 进制下的数字
ch.to_digit(20) // 在20进制的 ch 代表十进制的多少
std::char::from_digit(15, 20) // 把15 转换为 20进制下的数字表示。 返回 Some('f')



```

## HashMap



```Rust
use std::collections::HashMap;

// 新建
let mut hash_map = HashMap::new();
// 或者 HashMap::from(...);

// 新建2（实战中往往要这么用）
let keys = vec![String::from("blue"), String::from("red")];
let values = vec![10, 30];
let hash_map2: HashMap<_, _> = keys.iter().zip(values.iter()).collect();
println!("{:?}", hash_map2);


// 增&改
hash_map.insert(String::from("Blue"), 10); // 获取所有权
// 重复 insert 同一个 key 将覆盖


// 查1
let val_opt: Option<&i32> = hash_map.get("Blue"); // 获取引用，而不是所有权
// 返回一个 Option
let _b = match val_opt {
    Some(x) => println!("x={}", x),
    None => println!("None")
};

// 查2
for (key, val) in &hash_map {
    println!("k:v = {}:{}", key, val);
}

// 改2：循环中改

for (key, val) in &mut hash_map {
    println!("k:v = {}:{}", key, val);
    *val += 100;
}


// 删
hash_map.remove("red1");


println!("{:?}", hash_map);
```



## HashSet

```Rust
let mut hash_set = HashSet::new();
// 或者 HashSet::from([1,2,3,4])

// 插入
hash_set.insert(String::from("key1"));
hash_set.insert(String::from("key2"));

// 删除
hash_set.remove("key3"); // 不存在不会报错

// 清空
hash_set.clear();

// HashSet 没有那种“在循环中更改”的功能

// 其它方法
// hash_set.contains(val)
// extend
```


集合运算
```rust
is_subset
is_superset

is_disjoint // 没有任何重叠返回 true

hash_set1.intersection(&hash_set2) // 交集
hash_set1.union(&hash_set2) // 并集

// 符号更好记：
// 交
let set_tmp = &set1 & &set2;
// 并
let set_tmp = &set1 | &set2;
// 差集
let set_tmp = &set1 - &set2;
// 异或
let set_tmp = &set1 ^ &set2;

// 两个集合值相同
set1 == set2;
// 两个集合值不同
set1 != set2;
```

## Hash

- 一个值无论保存在哪，Hash都一样
- 它引用也与它有一样的Hash
- Box 与装的值有一样的Hash
- vec 与 `vec[..]` 有一样的 Hash
- String 与 `&str` 有一样的 Hash
- Rust 默认使用 `SipHash-1-3` 作为散列算法，它可以防止 HashDos 攻击，使用不安全的 Hash 速度更快
```
fnv="1.0.7"
use fnv::{FnvHashMap, FnvHashSet};
std::hash::Hash
```


## 迭代器相关

Rust 的迭代器是零开销抽象，性能与for循环一样。

### 迭代器的开启

`drain` 方法：把迭代器分为两部分

```
use std::iter::FromIterator;

let mut outer = "Earth".to_string();
let inner = String::from_iter(outer.drain(1..4));

assert_eq!(outer, "Eh");
assert_eq!(inner, "art");
```

这里不写迭代器怎么构造出来，写一下应用
- map
- filter
- flat_map：匿名函数返回的是一个 vec，把这个 vec 摊平，成为最终结果


scan 类似 map，但不同的是：
- 有一个初始值
- 匿名函数额外接受这个可修改的值
- 返回 Option，从而可提前终止迭代

scan 的例子
```
let iter = (0..10).scan(0, |sum, item| {
    *sum += item;
    if *sum > 10 {
        None
    } else {
        Some(item * item)
    }
});

assert_eq!(iter.collection::<Vec<i32>>(), vec![0, 1, 4, 9, 16]);
```


- take：接受n个 `v.into_iter().take(3)`
- take_while：第一次false时，返回None，之后都返回 None `take_while(|item| *item < 3)`
- skip
- skip_while
- fuse：使得第一次出现 None后，之后都是强制为 None
- rev：反转
    - 这个代码相当于头尾双指针：`it=v.iter();it.next_back();it.next()`
- enumerate：`for (idx,item) in b.into_iter().enumerate(){...}`
- chain：把两个迭代器横向（按顺序）粘合到一起，形成一条链
- zip：把两个迭代器纵向粘合到一起，形成一组数据对
    - 结果长度取最少的那个
    - 举例：
```
let v: Vec<_> = (0..).zip("ABCD".chars()).collect();
assert_eq!(v, vec![0, 'A'], [1, 'B'], (2, 'C'), (3, 'D'));
```
- cycle：无休止的循环
    - 例子 `let v: Vec<_> = (0..5).cycle().take(100).collect();`




比较复杂的
- peekable：不消费下一项的情况下探测下一项，并把它放到 next 中
- inspect
- by_ref
- cloned


### 迭代器的消费

简单累计
- sum
- count
- product
- max/min：
    - 返回 Option 对象
    - `f32,f64` 只实现了 `std::cmp::PartialOrd`，没有实现 `std::cmp::Ord`，因此不能使用上述两个方法
    - 作用于 HashMap 类型时，最值的标准按照 key 来，而不是 value
- max_by/min_by：后接一个函数
```Rust
use std::cmp::{PartialOrd, Ordering};
// 这里的双引用，是因为 num.iter() 会产生引用，然后 max_by 又会产生一次引用
fn cmp(lhs: &&f64, rhs: &&f64) -> Ordering {
    lhs.partial_cmp(rhs).unwrap()
}
let numbers = [1.0, 4.0, 2.0];
assert_eq!(numbers.iter().max_by(cmp), Some(&4.0));
assert_eq!(numbers.iter().min_by(cmp), Some(&1.0));
```
- max_by 还可以用于 HashMap
```
let res = populations.into_iter().
    max_by(|item1,item2|(&item1.1).cmp(&item2.1));
```
- 各种比较：从迭代器取值比较，知道可以做出决定
    - `eq`,`ne`
    - `lt,le,gt,ge`
    - `cmp,partial_cmp`
```
let v1 = vec![1, 2, 3];
let v2 = vec![1, 2, 2];
println!("{}", v1.into_iter().eq(v2.into_iter()))
```
- max_by_key/min_by_key：用于 HashMap
- any/all：后接匿名函数 `"Iterator".chars().any(char::is_uppercase);`
- fold：自定义累计操作
```
let a = [5, 6, 7, 8, 9, 10];
assert_eq!(a.iter().fold(0, |n, _| n + 1),  6);       // 实现count
assert_eq!(a.iter().fold(0, |n, i| n + i),  45);      // 实现sum
assert_eq!(a.iter().fold(1, |n, i| n * i),  151200);  // 实现product
assert_eq!(a.iter().fold(i32::min_value(), |m, &i| std::cmp::max(m, i)), 10);//实现max
```

取数类

- nth(n)：返回n对应的项
    - 如果n超了，返回None
    - 不会取得迭代器的所有权，因此可以多次调用
- last()：最后一个
- find(func)：第一个true


### collect

- collect
- FromIterator
- `std::iter::Extend`：一个集合拼接另一个集合
- partition：把迭代器分成两个集合（注意，不是分成两个迭代器，因为那样有权限问题）
```
let v: Vec<i32> = (0..10).into_iter().collect();
let (even, odd): (Vec<i32>, Vec<i32>) = v.into_iter().partition(|item| item % 2 == 0);
```


### 应用

可以自定义迭代器，例如自定义一个二叉树伤的迭代器

https://blog.csdn.net/feiyanaffection/article/details/125574968

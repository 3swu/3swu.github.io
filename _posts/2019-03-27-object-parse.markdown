---
author: wulei
comments: true
date: 2019-03-27 23:13:00+00:00
link: http://wuleiaty.github.io/2019/03/27/object-parse
slug: object-parse
title: Scheme解释器的内部对象模型、词法和语法分析
tags:
- Scheme
- 词法分析
- 语法分析
---

### 内部对象模型

实现Scheme解释器的第一步，是要定义出这个语言内部对象的数据结构。解释器的所有操作，都是基于这个数据结构的操作。

在这个解释器中，定义这个对象为`struct object`，这个对象结构有两个字段，第一个字段`type`存储这个对象的类型，对象的模型有空表、布尔、符号、数值、字符、字符串、序对、内部过程、复合过程。定义如下：

``` c
typedef enum {THE_EMPTY_LIST, BOOLEAN, SYMBOL,
              FIXNUM, CHARACTER, STRING, PAIR,
              PRIMITIVE_PROC, COMPOUND_PROC}
              object_type;
```

`object`结构的第二个字段`data`存储不同的类型数据的值，使用C语言的联合（union）类型。如下：

``` c
typedef struct object {
    object_type type;
    union {
        struct {
            bool value;
        } boolean;
        struct {
            char* value;
        } symbol;
        struct {
            long value;
        } fixnum;
        struct {
            char value;
        } character;
        struct {
            char* value;
        } string;
        struct {
            struct object* car;
            struct object* cdr;
        } pair;
        struct {
            struct objetc* (*fun) (struct object* arguement);
        } primitive_proc;
        struct {
            struct object* parameters;
            struct object* body;
            struct object* env;
        } compound_proc;
    } data;
} object;
```

其中`PAIR`类型包括表头和表尾，包含`car`和`cdr`字段，分别指向序对的表头和表尾。`primitive_proc`类型使用函数指针指向这个内部过程的C语言实现。`compound_proc`类型包含了这个过程的参数、过程体和环境。

### 词法分析

由于Scheme的语法全是由序对类型的S-Express组成，所以词法分析很简单。词法分析的目的是把源代码解析为符号串，Scheme的语法可以使用空格来分解为各个符号，所以只用使用空格来split字符流即可以，但是在左括号和右括号左右是没有空格的，所以词法分析的第一步是去掉注释，第二步就是在左右括号边上添加空格，然后可以根据空格来进行字符分割，最后生成token链表。

toke链表的抽象数据类型（ADT）如下设计：

```c
typedef struct token {
    char* value;
    struct token* next;
} token;

typedef struct {
    token* haed_token;
    token* token_pointer;
} token_list;
```

设计上，特意在`token_list`结构里添加了一个指针。这个指针是为了下一步的语法分析，这个指针用于指向当前分析到的token。

### 语法分析

从token链表的开始，使用parse过程和parse_pair两个过程来相互递归调用解析链表，parse过程根据token的不同类型来生成不同的内部对象。特别的，当遇到左括号时，说明需要读取序对，这时调用`parse_pair`过程来解析序对。在这个过程中，如果遇到下一个token为右括号，则返回空表。然后继续递归解析序对的表头（car）和表尾（cdr）。两个过程如下：

``` c
object* parse(token_list* list) {
    char* token_value = list->token_pointer->value;

    if(strcmp(token_value, "(") == 0) {
        list_iter(list);
        return parse_pair(list);
    }

    if(is_str_symbol(token_value)) {
        list_iter(list);
        return make_symbol(token_value);
    }

    if(is_str_digit(token_value)) {
        list_iter(list);
        return make_fixnum(token_value);
    }

    if(is_str_string(token_value)) {
        list_iter(list);
        return make_string(token_value);
    }

    if(strcmp(token_value, "#t") == 0) {
        list_iter(list);
        return make_boolean(true);
    }

    if(strcmp(token_value, "#f") == 0) {
        list_iter(list);
        return make_boolean(false);
    }

    char error_msg[TOKEN_MAX + 50];
    sprintf(error_msg, "unexcepted symbol : %s", token_value);
    error_handle(stderr, error_msg, EXIT_FAILURE);

}

object* parse_pair(token_list* list) {
    if(strcmp(list->token_pointer->value, ")") == 0) {
        list_iter(list);
        return the_empty_list;
    }

    object * car, * cdr;
    car = parse(list);
    cdr = parse_pair(list);
    return cons(car, cdr);
}

```

---

阅读[源代码](https://github.com/wuleiaty/Scheme)
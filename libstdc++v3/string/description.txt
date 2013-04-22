basic_string and string
=======================

# 概述

平时使用stl的basic_string和string的时候都包含的是string这个文件，这个文件在/usr/include/c++/4.4.4/
目录中，这个文件只是包含了实现basic_string和string时所需要的所有文件，basic_string和string
的代码主要在bits/basic_string.h和bits/basic_string.tcc两个文件中实现。

# 基本结构


# 构造函数

basic_string的构造函数在bits/basic_string.h的419~497行生命，但是除了默认的构造函数basic_string()
在这个文件中实现外，其他的构造函数都是在bits/basic_string.tcc中实现的。

## 默认构造函数basic_string()


## 拷贝构造函数basic_string(const basic_string& __str)

拷贝构造函数的实现在bits/basic_string.tcc的169~175行。这里面主要用到了basic_string::_Rep::_M_grab
函数。_M_grab函数实现在bits/basic_string.h的217~222行，这里通过判断当前的_Rep是否可共享以及两个
分配器是否相等来确定使用_M_refcopy还是_M_clone来获取实际数据的指针。

### _M_refcopy()

_M_refcopy函数定义在bits/basic_string.h的242~250行，249行通过_M_refdata返回了指向当前字符串的指针。
248行__gnu_cxx::__atomic_add_dispatch函数定义在ext/atomicity.h的87~99行，在这里这个函数就是将
_M_refcount加1。

可以看出拷贝构造函数在可以共享并且使用的分配器相同的情况下就是将两个basic_string::_M_dataplus::_M_p
指向相同的字符串。

### _M_clone()

_M_clone函数定义在bits/basic_string.tcc的622~636行，根据当前字符串的长度_M_length和新增的大小__res
来计算新的容量__requested_cap，然后_S_create函数生成一个新的basic_string::_Rep，并且把当前字符串的
内容拷贝到新的_Rep的字符串中，将新的_Rep字符串设置长度和可共享，然后返回_Rep指向的新的字符串的指针。

_M_grab中使用_M_clone的时候__res使用了默认参数0，这样生成的新的字符串就是_M_length和_M_capacity都是
当前字符串的_M_length。

### 总结

在拷贝构造函数basic_string(const basic_string& __str)中，当__str是可共享的并且两个字符串使用相同的
分配器的时候实际上新的字符串执行的时候__str相同的字符串，否则就根据__str的字符串拷贝一份一模一样的。

两种情况下的内存结构如图:

![_M_refcopy](./_M_refcopy.png)

![_M_clone](./_M_clone.png)


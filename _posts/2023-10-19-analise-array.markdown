---
layout: post
title: "Анализ исходного кода C++ STL tr1 и массив std"
subtitle: "\"Массивы\""
date: 2023-10-19 12:00:00
author: "Jollu"
hidden: false
header-img: "img/home-bg.jpg"
catalog: true
tags:
  - C++
  - Stl
  - Array
  - Анализ
---

## 0.Введение

Версия анализа исходного кода — gcc4.9.1.

Полное имя C++ TR1 (Технический отчет 1) является первым расширением стандартной библиотеки C++. Стандарт C++0x включает
TR1 и некоторые расширения самого языка. TR1 содержит долгожданный интеллектуальный указатель, регулярные выражения и
другие элементы, поддерживающие парадигмальное программирование. В черновом варианте пространство имен для новых классов
и шаблонов называется std::tr1.

## 1.std::tr1::array

использует:

```
#include <tr1/array>
std::tr1::array<int ,10> a;
```

Массив в tr1 достаточно прост и имитирует функциональность языка Array. Он был изменен, чтобы поддерживать операции
итератора, тем самым позволяя использовать алгоритмы на нем, как и на других контейнерах. В tr1 не происходит
автоматического создания или уничтожения массива. Итераторы прямо использовуют переданный указатель для определения
типа.

Пожалуйста, взгляните на краткий исходный код этого статического массива.

```cpp
template<typename _Tp, std::size_t _Nm>
struct array
{
    typedef _Tp 	    			      value_type;
    typedef value_type&                   	      reference;
    typedef const value_type&             	      const_reference;
    typedef value_type*          		      iterator;
    typedef const value_type*			      const_iterator;
    typedef std::size_t                    	      size_type;
    typedef std::ptrdiff_t                   	   difference_type;
    typedef std::reverse_iterator<iterator>	      reverse_iterator;
    typedef std::reverse_iterator<const_iterator>   const_reverse_iterator;
}
```

Используется `verse_iterator` для операций `rbegin` и рендеринга. Кажется, что у нас есть только один итератор, но на
самом деле у нас два. Кроме того, есть еще один итератор, который напрямую использует переданный указатель для
определения типа.

Это можно сравнить с прямыми и обратными итераторами в векторе.

Стойте отметить, что в `tr1::array` поддерживается передача размера массива равного 0. Например, мы используем
следующее:

```
std::tr1::array<int,0> a;
```

Для этого метода записи это будет Соответствовать следующему: ：

```
// Support for zero-sized arrays mandatory.
value_type _M_instance[_Nm ? _Nm : 1];
```

В соответствии с переданным размером, если он не равен 0 , это переданный размер, в противном случае это 1 .

## 2.std::array

использует:

```
std::array<int ,10> a;
```

Массив в std содержит:

![std_array.png](https://raw.githubusercontent.com/Light-City/cloudimg/master/std_array.png)

Сравните массив tr1 и std

```cpp
template<typename _Tp, std::size_t _Nm>
struct array
{
    typedef _Tp 	    			      value_type;
    typedef value_type*			      pointer;
    typedef const value_type*                       const_pointer;
    typedef value_type&                   	      reference;
    typedef const value_type&             	      const_reference;
    typedef value_type*          		      iterator;
    typedef const value_type*			      const_iterator;
    typedef std::size_t                    	      size_type;
    typedef std::ptrdiff_t                   	      difference_type;
    typedef std::reverse_iterator<iterator>	      reverse_iterator;
    typedef std::reverse_iterator<const_iterator>   const_reverse_iterator;

    // Support for zero-sized arrays mandatory.
    typedef _GLIBCXX_STD_C::__array_traits<_Tp, _Nm> _AT_Type;    // # define _GLIBCXX_STD_C std
    typename _AT_Type::_Type                         _M_elems;
}
```

найдено В массиве стоит отметить две вещи:
：

```cpp
// Support for zero-sized arrays mandatory.
typedef _GLIBCXX_STD_C::__array_traits<_Tp, _Nm> _AT_Type;    // # define _GLIBCXX_STD_C std
typename _AT_Type::_Type                         _M_elems;
```

Найдите __array_traits в исходном коде и увидите:

```cpp
template<typename _Tp, std::size_t _Nm>
struct __array_traits
{
    typedef _Tp _Type[_Nm];

    static constexpr _Tp&
    _S_ref(const _Type& __t, std::size_t __n) noexcept
    { return const_cast<_Tp&>(__t[__n]); }
};
```

Две приведенные выше строки кода можно понимать следующим образом:

```cpp
typedef _Tp _Type[100];
typedef _Type _M_elems;  // 一个含有100个元素的数组。
```

При написании кода, если мы хотим определить массив, мы можем написать так:

```cpp
int a[100];
//或者
typedef int T[100];
typedef T a;
```

Для передачи Обработка входящего размера более сложна, чем tr1, и для обработки ситуации, когда переданный размер равен
0, используется частичная специализация шаблона.

```cpp
template<typename _Tp, std::size_t _Nm>
struct __array_traits
{
    typedef _Tp _Type[_Nm];

    static constexpr _Tp&
    _S_ref(const _Type& __t, std::size_t __n) noexcept
    { return const_cast<_Tp&>(__t[__n]); }
};

template<typename _Tp>
struct __array_traits<_Tp, 0>
{
    struct _Type { };

    static constexpr _Tp&
    _S_ref(const _Type&, std::size_t) noexcept
    { return *static_cast<_Tp*>(nullptr); }
};
```

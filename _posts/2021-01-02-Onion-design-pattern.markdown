---
layout: post
title:  "Onion design pattern"
date:   2021-01-02 10:45:31 +1000
categories: C++ Design
---
* What
{:toc}

After some months of developing my library [CustomLibrary](https://github.com/floppyMike/CustomLibrary) I have developed a new code design which targets C++. I call it the `Onion Pattern` because it takes a `Mixin` and peels each `Mixin` off till it reaches the core.

# Mixins
A Mixin is nothing more then a template class which inherits its a template parameter.
{% highlight cpp %}
template<typename T>
class Mixin : public T // The template parameter gets inherited
{
};
{% endhighlight cpp %}

A Mixin is typically used to add certain functionality to a base class. For example:

{% highlight cpp %}
template<typename T> 
    requires requires(T t) { std::cout << t; }
class Printable : public T
{
public:
    void print()
    {
        std::cout << *this << '\n';
    }
};

class Object
{
    friend auto operator<<(std::ostream &o, const Object &obj) -> std::ostream &
    {
        o << obj.m_x;
        return o;
    }

private:
    int m_x = 10;
};

int main()
{
    Object obj1;
    std::cout << obj1 << '\n'; // Prints 10

    Printable<Object> obj2;
    obj2.print(); // Prints 10

    return 0;
}
{% endhighlight cpp %}

The pattern allows for a higher flexibility in code reuse. Yet a problem arises if the added functionality requires multiple variables to be stored inbetween operations. Simply adding them as member variables can be the solution if the member variables are actually used throughout the objects lifetime. The cases where the object only requires the added functionality for only a short period of time will lead to a waste of space since the variables sit in the objects memory for the duration of its lifetime. The following example highlights the problem:

{% highlight cpp %}
template<typename T>
    requires requires(T t) { std::cout << t; }
class Incrementable : public T
{
public:
    void print()
    {
        std::cout << m_i++ << '\t' << *this << '\n';
    }

private:
    size_t m_i = 0; // What to do with this after its use?
};
{% endhighlight cpp %}

This mixin simply print the object with a incrementation at the front. We may require this somewhere in the code but after that `m_i` simply becomes wasted memory. It would be wise to just create a separate class altogether.

# Separate Class

The incrementation problem can be solved by using this class.
{% highlight cpp %}
template<typename T>
    requires requires(T t) { std::cout << t; }
class Incrementable
{
public:
    Incrementable(const T *o)
        : m_obj(o)
    {
    }

    void print()
    {
        std::cout << m_i++ << '\t' << *m_obj << '\n';
    }

private:
    const T *m_obj;
    size_t m_i = 0;
};
{% endhighlight cpp %}

With a class like this we solve a few problems.
1. The lifetimes become separate. When the operation with `Incrementable` is finished its object becomes deallocated but the targeted object is still alive and can be used.
2. We can use `Printable` and `Incrementable` at the same time. The compiler can distinguish between the 2 methods and it won't give an ambiguity error.
3. There is no need to copy over a existing target object to a mixin just to add more functionality.

To ease up the process we can define a class `Functor` which reduces code redundancy and allows us to change all `-able` classes.
{% highlight cpp %}
template<typename T>
class Functor
{
public:
    constexpr explicit Functor(T *p) noexcept
        : m_ptr(p)
    {
    }

    constexpr auto obj() const noexcept { return m_ptr; }

private: // Could also be protected since abstraction isn't necessary in this context.
    T *m_ptr;
};

template<typename T>
    requires requires(T t) { std::cout << t; }
class Incrementable : public Functor<T>
{
public:
    using Functor<T>::Functor;

    void print()
    {
        std::cout << m_i++ << '\t' << *this->obj() /* or *this->m_ptr */ << '\n';
    }

private:
    size_t m_i = 0;
};
{% endhighlight cpp %}

Yet the larger the project the more classes we may need which could quickly drain out our creativity for name giving and heighten the complexity. We could maybe provide template specializations for similar classes.

# Separate Template specialized Classes

By using template specialization it's possible to provide an entirely new implementation for a templated class.
{% highlight cpp %}
template<typename T>
concept arithmetic = std::is_abstract_v<T>; // arithmetic concept

template<typename>
class Convertable; // If no match is found, then the compiler gives a "not defined" error
 
template<typename T>
    requires requires(T *t) { { t() } -> arithmetic; }
class Convertable<T> : public Functor<T> // Everything which returns a arithmetic value will match this
{
public:
    void print() const noexcept { std::cout << "Basic\n"; }
};

class Object1;
template<>
class Convertable<Object1> : public Functor<Object1> // If object is of type Object1
{
public:
    void print() const noexcept { std::cout << "Object1\n"; }
};

class Object2;
template<>
class Convertable<Object2> : public Functor<Object2> // If object is of type Object2
{
public:
    void print() const noexcept { std::cout << "Object2\n"; }
};

class Object1
{
    // Management methods...

    constexpr auto operator()() const noexcept { return m_x; }

private:
    int m_x = 10;
};

class Object2
{
    // Management methods...

    constexpr auto operator()() const noexcept { return m_x; }

private:
    float m_x = 10.F;
};

int main()
{
    Object1 obj1;
    Convertable<Object1>(&obj1).print(); // Prints "Object1"

    Object2 obj2;
    const Convertable<Object2> c(&obj2);
    c.print(); // Prints "Object2"

    return 0;
}
{% endhighlight cpp %}

While on the implementers side the code does look a little more complex, the code on the users side look cleaner. It would be smart to abstract the user side a bit in such that if a special case arises it will be possible to opt out of from the above. It also erases the long and ugly name and template parameter.

{% highlight cpp %}
template<typename T>
auto convert(T *o)
{
    return Convertable<T>(o);
}

int main()
{
    Object1 obj1;
    convert(&obj1).print(); // Prints "Object1"

    Object2 obj2;
    const auto c = convert(&obj2);
    c.print(); // Prints "Object2"

    return 0;
}
{% endhighlight cpp %}

If you haven't noticed already a huge problem arises. What about `const`? When the target object is constant then it will try to specialize for it. It could be an intended feature but we don't want to rewrite the entire class just for a `const`. Luckily C++20 adds concepts, which makes this a trivial matter. To keep things simple and easier to understand later we are going to add a second template parameter which indicates the type without `volitile` or `const`.

{% highlight cpp %}
template<typename T, typename U>
    requires requires(T *t) { { t() } -> arithmetic; }
class Convertable<T, U> : public Functor<U>
{
public:
    using Functor<U>::Functor;
    void print() const noexcept { std::cout << "Basic\n"; }
};

class Object1;
template<typename T>
class Convertable<Object1, T> : public Functor<T>
{
public:
    using Functor<T>::Functor;
    void print() const noexcept { std::cout << "Object1\n"; }
};

template<typename T>
auto convert(T *o)
{
    return Convertable<std::remove_cv_t<T>, T>(o); // Add a non-const or volitile parameter
}

int main()
{
    const Object1 obj1;
    convert(&obj1).print(); // Prints "Object1"

    return 0;
}
{% endhighlight cpp %}

OK now we just implemented a neat way of separating data from functionality. In this case we have 2 types of classes: storage classes & functionality classes. Storage classes are responsible of managing the data contained within and providing a usable interface for the functionality classes and functions alike. Functionality classes are responsible for performing a certain action. It's important for these classes to only be responsible for 1 action which the pattern above reinforces.
<br>
But we can do better.

# Onion Pattern
In some cases variables and methods provided by a mixin are in fact nice to have. But mixins don't mix well with our method of adding functionality. What if we could provide a way for the functionality class to extend **based** on the mixins of the target object? This is in fact how the [CustomLibrary](https://github.com/floppyMike/CustomLibrary) works.

## The Design
The *extending* can be done in 2 ways.
1. CRTP
2. Mixins

CRTP is certainly a valid choice but we will be hindering it a bit. With CRTP the derived class can access methods of the base class as well as the base class can access methods of the derived class. For example:

{% highlight cpp %}
template<template<typename> class... F>
class Derived : public F<Derived<F...>>...
{
public:
    using Z = int;

    constexpr auto num_derived() const noexcept { return m_x; }

    void print_base() const
    {
        std::cout << this->num_base() << '\n';
    }

private:
    int m_x = 10;
};

template<typename Impl>
class Base
{
public:
    // using U = typename Impl::Z; // What is Z?

    void print_derived() const
    {
        std::cout << static_cast<const Impl*>(this)->num_derived() << '\n';
    }

    constexpr auto num_base() const noexcept { return m_x; }

private:
    int m_x = 20;
};

int main()
{
    Derived<Base> d;
    d.print_base(); // Prints 20
    d.print_derived(); // Prints 10

    return 0;
}
{% endhighlight cpp %}

For our usecase we don't need the access to the Base from our derived class. Also an error arises when we try to reference a type from the derived class because the compiler needs to compile the derived classes before compiling the base class. So our type `Z` is undefined from the perspective of the base class. Mixins don't have this problem.

So the plan is as following: The extending is going to happen based on the target mixins and it itself is a mixin chain.
{% highlight cpp %}
BBBBBB<CCCCCC<Base>> var;

template<typename T>
class Func<BBBBBB<Nonesuch>, T>; // We "peel" each layer away from the mixin
template<typename T>
class Func<CCCCCC<Nonesuch>, T>; // Each layer gets matched with a functor
template<typename T>
class Func<Base, T>; // Layer are substituted with Nonesuch. This goes on till the core
{% endhighlight cpp %}
If your wandering what `Nonesuch` is, it's a class which is non-constructable, copyable and movable. Since C++20 it has been introduced to the std experimental branch but it's trivial to create it yourself.

## The Implementation
A bit of metatemplate programming is going to be required here. We are going to implement a sort of binder class to assign each layer to its functor and then bind all these functors together. This will be done recursively.

To complete this process we will need the following:
1. The functionality template classes
2. The functor class
3. The target mixin type

This will lead to the following entrance type
{% highlight cpp %}
template<typename T>
using Func = typename Binder<_Func_, Functor<T>, T>::type;
{% endhighlight cpp %}
We are now abstracting `Func` and making it the result type of our binder class.

Binder looks like the following:
{% highlight cpp %}
template<template<typename, typename> class F, typename Func, typename T>
struct Binder
{
    using type = T; // keep it simple for now
};
{% endhighlight cpp %}
Currently our `Binder` just returns the type `T` but we want it to peel so lets implement that. But how do we *peel*? Remember: our first template parameter was always the base class. So lets define a function which will extract it!
{% highlight cpp %}
template<typename T>
auto separate(const T &) -> std::pair<T, Nonesuch>; // When we reach the end

template<template<typename...> class T, typename U, typename... Z> // Z: In case our target has additional templates
auto separate(const T<U, Z...> &) -> std::pair<T<Nonesuch, Z...>, U>; // Get ourselves the base class
{% endhighlight cpp %}

Why are our parameters constant references? We need to watch out for the fact that our target mixin could possibly be not constructable. Since references and pointers don't care about construction rules we can use them. Now with that out of our way we can separate our target mixin.

{% highlight cpp %}
template<template<typename, typename> class F, typename Func, typename T>
struct Binder
{
    using sep = decltype(separate(std::declval<T>())); // Get base
    using type = T; // keep it simple for now
};
{% endhighlight cpp %}

We are now going to pass on the base to the functional class but we need to keep in mind that we are starting at the top of the mixin. So we need to **first** recurse to the front before we define any type of relevant type.

{% highlight cpp %}
template<template<typename, typename> class F, typename Func, typename T>
struct Binder
{
    using sep   = decltype(separate(std::declval<T>())); // Get base
    using front = typename Binder<F, Func, typename sep::second_type>::type;
    using type  = T; // keep it simple for now
};
{% endhighlight cpp %}
Right now we will recurse forever. That always results in horrifying compiler error messages. Try it out! Remember in `separate` we define `Nonesuch` when we reach the base type. The following stops the binder from recursing further.
{% highlight cpp %}
template<template<typename, typename> class F, typename Func, typename T>
struct Binder
{
    using sep   = decltype(separate(std::declval<T>())); // Get base
    using front = typename Binder<F, Func, typename sep::second_type>::type; // Recurse back
    using type  = T; // keep it simple for now
};
template<template<typename, typename> class F, typename Func>
struct Binder<F, Func, Nonesuch>
{
    using type = Func; // Return the base type
};
{% endhighlight cpp %}

We can now start constructing the functors from bottom to top.
{% highlight cpp %}
template<template<typename, typename> class F, typename Func, typename T>
struct Binder
{
    using sep   = decltype(separate(std::declval<T>())); // Get base
    using front = typename Binder<F, Func, typename sep::second_type>::type; // Recurse back
    using type  = F<typename sep::first_type, front>; // The meat
};
{% endhighlight cpp %}

Nice we have a finished prototype but notice a problem. **Every type must be a complete type.** This could be a giant hinderance but we could solve it by ignoring the incomplete type. Sadly prior to C++20 there wasn't really a robust way of detecting a incomplete type. With C++20 we can create a concept which check if a `sizeof` operation is possible.
{% highlight cpp %}
template<typename T>
concept complete_type = requires(T t) { sizeof(t); }
{% endhighlight cpp %}
Every complete type has a size. A non-complete type doesn't know its size since it doesn't yet know (if ever) what member variables it has.

Solving the bug of ours we get:
{% highlight cpp %}
template<template<typename, typename> class F, typename Func, typename T>
struct Binder
{
    using sep   = decltype(separate(std::declval<T>())); // Get base
    using front = typename Binder<F, Func, typename sep::second_type>::type; // Recurse back
    using temp  = F<typename sep::first_type, front>; // The meat
    using type  = typename std::conditional<complete_type<temp>, temp, front>::type; // Ignore incomplete types.
};
{% endhighlight cpp %}

Finally we have a decent prototype.

# Example
{% highlight cpp %}
#include <iostream>

template<typename T>
class Functor
{
public:
    constexpr explicit Functor(T *p) noexcept
        : m_ptr(p)
    {
    }

    constexpr auto obj() const noexcept { return m_ptr; }

private: // Could also be protected since abstraction isn't necessary in this context.
    T *m_ptr;
};

struct Nonesuch
{
    Nonesuch(Nonesuch &&)	   = delete;
    Nonesuch(const Nonesuch &) = delete;

    void operator=(const Nonesuch &) = delete;
    void operator=(Nonesuch &&) = delete;
};

template<typename T>
concept complete_type = requires(T t)
{
    sizeof(t);
};

template<typename T>
auto separate(const T &) -> std::pair<T, Nonesuch>; // When we reach the end

template<template<typename...> class T, typename U, typename... Z> // Z: In case our target has additional templates
auto separate(const T<U, Z...> &) -> std::pair<T<Nonesuch, Z...>, U>; // Get ourselves the base class

template<template<typename, typename> class F, typename Func, typename T>
struct Binder
{
    using sep	= decltype(separate(std::declval<T>()));							 // Get base
    using front = typename Binder<F, Func, typename sep::second_type>::type;		 // Recurse back
    using temp	= F<typename sep::first_type, front>;								 // The meat
    using type	= typename std::conditional<complete_type<temp>, temp, front>::type; // Ignore incomplete types.
};
template<template<typename, typename> class F, typename Func>
struct Binder<F, Func, Nonesuch>
{
    using type = Func; // Return the base type
};

template<typename>
class Layer1;
class Object1;

template<typename, typename>
class _Func_;

template<typename T>
class _Func_<Layer1<Nonesuch>, T> : public T
{
public:
    using T::T;
    void print() const noexcept { std::cout << "I work!\nx = " << this->obj()->x; }
};

template<typename T>
using Func = typename Binder<_Func_, Functor<T>, T>::type;

template<typename T>
auto func(T* o) { return Func<T>(o); }

class Object1 {};

template<typename T>
class Layer1 : public T
{
public:
    int x = 10;
};

template<typename T>
class Layer2 : public T {};

int main()
{
    Layer2<Layer1<Object1>> obj1;
    const auto f = func(&obj1);
    
    f.print();

    return 0;
}
{% endhighlight cpp %}

# Pros and Cons
<table style="border-spacing: 10px; width:100%; text-align: left;">
  <tr>
    <th>Pros</th>
    <th>Cons</th>
  </tr>
  <tr>
    <td>Fast code</td>
    <td>Longer compilation time</td>
  </tr>
  <tr>
    <td>Clear code organization</td>
    <td>Complex solution</td>
  </tr>
  <tr>
    <td>Adaptable to many situations</td>
    <td>Could lead to code bloat</td>
  </tr>
</table> 

In my opinion it's nice to have this pattern in my toolkit but I certainly think it's way to complicated for a simple program. Personally I would only use for designing header-only libraries since it can actually reduce the overall complexity with enough code. Since this is heavily templated I doubt using this for a dynamic or static library is optimal. Also notice we are using a lot of templated code. This could lead to code bloat further discouraging its use. But with smart use and correct compiler flags this could very well be a huge upgrade to your code.
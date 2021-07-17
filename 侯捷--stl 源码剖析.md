### stl 源码剖析

1. 六大组件：容器、算法、迭代器、仿函数、配接器、配置器。

2. deque分段连续，每次扩充一个buffer。

3. vector每次扩充两倍之后会把原来的数据全部拷贝过去（?）。

4. 由于queue和stack从技术上是由deque实现的，所以可以称之为适配器。为了保留这两种容器的技术特点，不再提供iterator。

5. unordered_set 的哈希结构中，篮子的数目一定于元素数目的。篮子数目不是固定的，会扩大。

6. 应该在程序里使用容器而不是分配器，因为分配器的deallocate操作不仅需要指针，还需要知道单元的大小。

7. 泛型编程的思想是将data和method分开来，但是面向对象编程确实将它们封装到一起。所以iterator在泛型编程中的作用被凸显了出来：联系起来容器和算法。

8. 采用GP的好处是：各个团队的容器和算法可以不同，只需要他们都支持iterator即可，算法通过迭代器确定操作范围，并通过它取用元素。也因此，操作符重载也扮演了重要的角色。

9. 标准库中用到的sort需要一些条件：迭代器必须是randomAccessIterator，可以进行一些四则运算，而某些容器如list不能用sort的原因在此。

10. 模板有三类，分别是类模板、函数模板、成员模板。

    > 类模板在定义的时候要明确指出来类型，函数模板可以尽心实参推导，因此可以不用指出来。

11. template<>中空就表示特化。template<若干个typename>中拿出来几个指定类型，就是偏特化。partial specialization。参数个数与参数范围上的偏特化。

    > 偏特化中如果泛化是一个T，那么T*也可以作为一种偏特化，const T *也是一种偏特化。

12. 如果一个类直接加一个小括号，就是一个没有名称的临时对象。

13. allocator分配器，在不同的编译器中最终都是通过调用new/malloc来实现的，但是他们有优化和改进。

14. list实际上是环状双向链表。但是向外是单向链表。为了区分++i和i++,前置型（++i）的运算符重载中并没有参数，而后置型（i++）的运算符重载函数中会有一个类型如int。后置型的会用到前置型的++的函数。前置型的函数参数是带引用的&，而后置型的并没有带引用。C++不允许后面连续两次++。所以这就是后置型的返回类型没有带引用。

15. priority_queue的底层支撑是vector，stack和queue的底层支撑是deque。map和set的底层是红黑树，而Unordered类的底层支撑是哈希表。这种关系不是继承，而是复合。

16. iterator类中有两个主要部分，一个是至少五个typedef，另外是各种运算符重载。前者用来回答算法对于容器的提问：容器内的数据是什么类型？迭代器能够运动？运动的步伐可不可以跳跃？两个迭代器之间的遍历范围是多大（int/long）？

    > 这五种associated types分别是：
    >
    > > iterator_category(单双向？)|value_type|*pointer*|*reference*|difference_type

17. 由于迭代器可能是指针而不再是iterator class，无法再回答这五个问题，设计了traits 用来分离class iterator和non iterator。traits是利用模板的偏特化（传进来的参数不同可以使得模板偏特化）来回答这问题的。除了针对iterator设计出来的萃取机以外还有其他别的traits，如char traits|type traits|pointer traits|array traits|allocator traits等。

18. array没有构造和析构函数。

19. deque的类中有四个数据，分别是首尾两个iterator+控制中心map的指针+控制中心的大小。4+4+2*(4 *4)=40。对象本身有40个字节。deque每个缓冲区的大小确定方法：

    > 如果deque 定义式中缺省参数为0代表使用预设值。
    >
    > 如果给定的不是0，则返回512/T，否则返回1表示每个缓冲区只存一个元素。
    >
    > :heart:但是后来的版本已经取消了自定义缓冲区大小的选项了。

    每个迭代器有四个指针，分别是cur|start|finish|node。 deque扩充到vector的中部。

19. deque<T>::insert函数会自动判断要插入的位置离双端的远近，并且会自动搬移元素少的那一部分。deque模拟连续空间。

21. queue和stack内含一个deque然后封锁住其中的某些功能。也因此我们不会叫他们是一个容器，而是把它称为适配器。把别人改装一下变成它自己。但是他们不支持遍历，不支持iterator。stack可以选择vector作为底层容器，但是queue不能使用vector作为底层容器，原因是vector(没有pop_front)不支持queue的pop操作。:apple:并且两者都不能使用set和map作为底层容器。

22. 红黑树：平衡二叉搜索树。类似的树还有AVL等。红黑树在gnu2.9中只包含三个数据成员，分别是node_count、header、compare_object。前二者都是四个字节，最后一个是仿函数其实是没有大小的，但是世界上编译器会给它一个字节，根据内存对齐的原理，最后红黑树的大小会变成12。:heart:header也是一个没有值的辅助结点。

23. 红黑树中key+data=value，红黑树的实现中有五个参数，通过上式可以理解到红黑树的参数分别是key/value/keyofvalue/compare/alloc，第三个参数是要告诉红黑树在一包value中如何把key拿出来。在set中key=value，所以前两个参数一样，由于key=value所以第三个identity<int>会传回同一个东西。证同函数。

24. handle and body的目标。

25. 我们无法使用set/multiset的iterator来改变元素值。

    > set的元素必须独一独二，因此其insert调用的是红黑树的insert_unique函数
    >
    > multiset元素可以重复，因此其insert调用的是红黑树的insert_equal函数

    但是实际中红黑树是可以通过iterator来改变红黑树的数据的，之所以set、multiset不能改是因为其iterator是调用的红黑树的const_iterator。

26. set、multiset的所有操作，都通过调用底层的红黑树来完成的，从这个层面看，set其实也是一个容器适配器。跟stack和queue一样。它们需要三个参数。

27. map和multimap也是红黑树为底层，只是红黑树的第三个参数不一样。无法使用迭代器改变key，但是可以改data。 它们需要四个参数。它的内部红黑树中，第三个参数keyofvalue是select1st实现的。map的iterator是红黑树的iterator，但是在value的定义中，特意定义成了pair<const key, T>value_type。自动把pair中的key变为了const。

28. 只有map中才有[]操作符，multimap并没有。map中如果没有某个key，但是你调用map[key]的话会自动创建一个这样的key并返回一个默认的初值。

29. lower_bound是<algorithm>中的一个函数。lower_bound（key）如果能查找到key则返回第一个key所在的iterator，如果不存在则返回最适合去存放这个key的位置。

30. 由于map中的[]操作符的重载中先使用了lower_bound,再使用的insert，因此这提示我们[]相比要稍微更耗时一些。但是[]更直观一点。

31. 另外一种关联式容器hashtable，散列表。由于不可能有特别大的空间存放数据， 所以separate chaining。篮子也是按照近似两倍的倍数扩大的。这个过程叫rehash。一般篮子的个数是素数，所以rehash之后会找两倍附近的素数。

32. hashtable需要6个参数。value、key、hashFcn、extractKey 、equalKey、分配器。hashtable有(public)三个class函数对象、(private)vector的buckets 、和登记元素个数的num_elements。所以总共3+12(vector)+4=19=20个字节。

33. hashtable中的iterator中除了有指向自己的指针以外，还有一个指向buckets的指针。

34. 标准库中没有提供现成的hash<stirng>。

35. 计算某个元素在hashtable的buckets位置在底层都是通过return hash(key)%n来实现的。篮子永远大于元素的个数。

36. unordered_set中有个成员函数bucket_size(i)某个篮子中元素的个数。bucket_count()篮子的数量。size()元素的总数量。

    # 第三部分

1. 算法部分是一个function template，其他容器、迭代器、仿函数、适配器、分配器都是类模板。

2. 各种迭代器的iterator_category：

   > 五种类型：
   >
   > input_iterator_tag/output_iterator_tag/forward_iterator_tag/bidirectional_iterator_tag/random_access_iterator_tag.
   >
   > 其中的继承关系：
   >
   > input>forward>bidirectional>random_access|output

3. 为什么迭代器的类型不做成enum而是做成struct，原因是做成enum之后无法对不同的迭代器类型做出重载，enum的话这五种类型会被看成一种。另外一种原因是可以稍微减少一点工作量：由于派生类is a 基类，因此在找重载函数的时候，可以写出其基类的重载函数，从而实现调用。算法模板中虽然可以接受各种iterator，但是也会有暗示，比如某个模板函数中的参数是input_iterator类型的，此时派生类都可以用这个函数，一开始未必会报错，但是并不会活过编译期。也就是说在模板中，把那个T的意义用名字显示出来。

4. random: array+vector+deque

   bidirectional: list+(multi)set+(multi)map 红黑树

   forward: forward_list+unordered_X

   istream_iterator: input

   ostream_iterator: output

5. #include<typeinfo> :typeid(iterator).name()

6. advance和distance算法

7. copy算法：非常多个版本，不断检查迭代器是不是某种特别的迭代器从而选择最优的方式来加快程序。memmove()/copy_dispatch()/for()。泛化+特化+强化+重载。

8. Type Traits的坑：是不是trivial的。

9. 函数对象（仿函数）：它是一个类或者是一个结构体，然后内部对()进行重载。

10. for_each(first,last,f): 对每个元素做一个操作。f同样可以是自定义函数或者是仿函数。或者使用for(auto)。

11. replace+replace_if+replace_by：第二个带一个条件，所以要传入一个predicate（判断），第三个拷贝到新的地方去。

12. count+count_if/find+find_if:带有这些的有()set<>和()map<>。

13. 容器list和forward_list是自带sort的容器。

14. lower_bound+up_bound: 安插某个元素到一个***排好序***的序列中，分别能安插进去的最低位置和最高位置。

15. functors仿函数：这个东西为算法服务。 仿函数分为三类，分别是算术类、位运算类、相对关系类。如plus、logic_and、equal_to等。标准库提供的仿函数都是继承关系，其基类时binary_function<T,T,T>。stl中还可能存在的基类是unary_function,表示一个操作数，比如取绝对值操作。

16. STL规定每个仿函数(函数对象)都应该挑一个基类来继承，如果用户希望创造的仿函数可以被修改从而适配，就可以获得三个、或者两个typedef。这样的话，仿函数可以回答配适器的各种问题。因此可适配的条件就是要继承STL中的特定基类。

17. typename+()临时对象。

### 适配器

1. adapter是一种设计模式。它通通都是使用内含的方式或者说复合，而不是继承，它来回答算法的问题。

2. 容器适配器：stack、queue；

   函数适配器：binder2nd，重载()以供调用。

3. 占位符using namespace std::placeholders;binder2nd的新版函数bind 可以指定返回类型。

4. 迭代器的适配器：reverse_iterator: 复合iterator即可。

   inserter: 接管copy函数中的=运算符，对其进行重载。

   x配适器：ostream_iterator，把某些函数中的运算符进行重载。

### 第四讲：stl的周边

1. 对于一些基本类型会有自己的内定的哈希函数：std::hash< std::string>。
2. typename...表示任意多个模板类型，称为参数数量可变化模板variadic templates. 这个东西使用的时候要使用递归。
3. 一个万用的哈希函数：越乱越好，黄金比例。哈希函数类型可以有三种，第一种是一个普通函数，第二种是一个类函数，第三种是提供一种针对当前类型的hash函数特化版本。

4. tuple元组。make_tuple()函数。get<1>(tuple), tuple可以比大小、可以赋值，以及更多的操作符重载。tie函数用于取出tuple的各个成分，便于赋值。

5. type traits: typedef _true_type/ _false_type has_trivial_default_constructor/has_trivial_something......（二三十种）用来回答别的函数问type traits的问题。
6. type traits的原理：泛化与偏特化，偏特化的时候去掉一些属性或者得到一些属性。有一些特殊的属性是无法找到源代码的，所以可能是编译器在做这些工作。
7. cout是一个对象。其基类中重载了非常多的<<的符号，表示可以接受非常多的类型，比如 ostream operator<<(int) 。
8. moveable元素对于各种容器速度的影响：move实际上是一个浅拷贝，并且打断之前对象对数据的拥有权。

 
STL中的库函数

- assign

- accumulate(begin(),end(),init)//<numeric>

  > 还可以定制化这个函数，为**accumulate(begin(),end(),init,func/*func)**
  >
  > > 比如说：
  > >
  > > int func(int x,int y){
  > >
  > > return x+2*y;
  > >
  > > }
  > >
  > > 结果返回的是每个数的两倍和，可以看成func中每一次的x=之前的所有之和

  累加范围是[first,last),
  
- lower_bound:查找第一个大于等于目标值的元素；

- upper_bound:查找第一个大于目标值的元素；

- equal_range：可以看成是lower_bound和upper_bound的结合体。

- binary_search:bool有没有存在这个元素。

- inplace_merge(begin(),begin()+mid,end()); begin()+mid=first iterator of the second sorted vector.
- merge(vec1.first,vec1.end(),vec2.first,vec2.second,vec3); vec3 should be allocated in advance.
- minmax_element(vec.begin(),vec.end());returns a iterator pair contains the smallest and the biggest value.(iter is where the first max or min value appears).
  - min_element+max_element: also return the iterator of the min and the max postion.
  - minmax:the argument is a initializer contains v


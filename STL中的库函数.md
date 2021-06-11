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


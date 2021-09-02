### JPA的原理

1. 通过BeanFactory查找所有继承Repository接口的类
2. 用动态代理的方式对Repository接口进行实例化，并放入spring容器中备用

   
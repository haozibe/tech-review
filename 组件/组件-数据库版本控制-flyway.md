## 原理

flyway会先在空库中生成一个flyway_scheme_history表，该表会记录classpath或resource下的migration文件（可以为SQL或JAVA），每次启动时都会检测该表中同一个表类型的版本，如果存在高于当前版本的表，则会忽略该表，只执行高版本的表。




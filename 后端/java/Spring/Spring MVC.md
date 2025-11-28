# Spring MVC

## Bug

### SpringBootWebFlux

- 导入SpringBootWebFlux包，在切面无法获取到当前servlet Request对象；

### 异常处理

- 子类的方法抛出的异常范围不能超过父类的方法抛出的异常范围，子类也可以不抛出异常；
- 接口的实现类可以不抛异常，也可以抛出与接口不一样的异常. 但是必须是接口定义的异常或是该异常的子类;

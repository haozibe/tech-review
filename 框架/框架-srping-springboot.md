### springboot原理

1. 扫描Meta-INF/resources/spring.factories，加载依赖的jar包。
2. 根据spring.factories配置加载AutoConfigure类。
3. 根据@Conditional注解的条件，进行自动配置并将Bean注入到Spring Context。


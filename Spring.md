# 1 典型场景

## 1.1 多个AOP切面注解的顺序

- 配置AOP顺序的三种实现方式

  - 实现org.springframework.core.Ordered接口

    ~~~java
    @Aspect
    @Component
    @Log4j
    public class TestAspect implements Ordered {
        @Override
        public int getOrder() {
            return 1;
        }
      ...
    }
    ~~~

  - 使用org.springframework.core.annotation.Order注解

    ~~~java
    @Aspect
    @Component
    @Log4j
    @Order(1)
    public class TestAspect {
      ...
    }
    ~~~

  - 通过配置文件配置

    ~~~xml
    <aop:config expose-proxy="true">  
        <aop:aspect ref="aopBean" order="0">    
            <aop:pointcut id="testPointcut"  expression="@annotation(xxx.xxx.xxx.annotation.xxx)"/>    
            <aop:around pointcut-ref="testPointcut" method="doAround" />    
            </aop:aspect>    
    </aop:config>  
    ~~~

- Transactional切面的优先级

  - 默认优化级：最小优先级

  - 事务的增强类型：Around advice

  - 修改事务切面的优化级的方式

    - 注解方式

      ~~~java
      @EnableTransactionManagement(order = 1)
      ~~~

    - 配置方式

      ~~~xml
      <tx:annotation-driven order="1"/>
      ~~~

      
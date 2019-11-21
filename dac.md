# 通用数据权限校验原型说明



数据权限的本质就是根据用户信息，拼接一些SQL

当前使用阿里Druid中的SQLParser进行动态sql的拼接



## 架构设计

架构图待补



## 工程结构

| 工程名      | 描述                             |
| ----------- | -------------------------------- |
| dac-common  | 公共资源                         |
| dac-core    | 服务端，对外提供规则RestAPI      |
| dac-service | service实现                      |
| dac-view    | view实现                         |
| dac-demo    | demo工程                         |
| dac-client  | client集成相关实现               |
| dac-starter | springboot-starter，用于工程引入 |



## 如何使用

1.启动dac-core工程

2.在目标工程的maven配置中引入

```xml
<dependency>
   <groupId>com.jusdagolbal.dac</groupId>
   <artifactId>dac-starter</artifactId>
   <version>${project.version}</version>
</dependency>
```

3.在目标工程的application.yml文件中新增dac配置

```yml
jusda:
  dac:
    endpoint: http://localhost:8081
logging:
  level:
    com.jusdagloble.dac.intercepter.AbstractDacInterceptor: debug
```

4.在需要进行数据权限拦截的方法上增加注解@DataAccessControl

```java
@DataAccessControl(rules = {"rule4","rule-x"})
public Page<Order> findAll(Order request, Pageable pageable) {

    DacContextHolder.setVar("t_order.create_by", "'李四'");
    DacContextHolder.setVar("t_order.create_by_1", "'张三'");
    DacContextHolder.setVar("t_order.create_by_2", "'李四'");
    DacContextHolder.setVar("t_order.create_by_list", List.of("张三", "王五"), true);

    QOrder order = QOrder.order;
    long offset = pageable.getOffset();
    int limit = pageable.getPageSize();
    JPAQuery<?> jpaQuery = new JPAQuery<>(entityManager);
    var jpaQueryBase = jpaQuery.select(order).from(order);
    if (Objects.nonNull(request.getCreateBy())) {
        jpaQueryBase.where(order.createBy.eq(request.getCreateBy()));
    }
    if (Objects.nonNull(request.getTotalAmount())) {
        jpaQueryBase.where(order.totalAmount.eq(request.getTotalAmount()));
    }
    QueryResults<Order> queryResult = jpaQueryBase.offset(offset).limit(limit).fetchResults();

    return new PageImpl<>(queryResult.getResults(), pageable, queryResult.getTotal());
}
```

5.测试

启动demo工程后执行此方法，查看日志

```
2019-11-21 16:19:11.752 DEBUG 6122 --- [nio-8080-exec-3] c.j.d.i.AbstractDacInterceptor           : 原始sql：select count(order0_.id) as col_0_0_ from t_order order0_
2019-11-21 16:19:11.839 DEBUG 6122 --- [nio-8080-exec-3] c.j.d.i.AbstractDacInterceptor           : 附加权限后的sql：SELECT COUNT(order0_.id) AS col_0_0_
FROM t_order order0_
WHERE order0_.create_by = '张三'
	AND (order0_.create_by = '张三'
		AND order0_.total_amount < 1000)
```

可以看到在正常的sql后附加了注解配置的数据权限规则附加的条件，可以看到数据权限是正常生效的。



## 关键点说明

### @DataAccessControl

```java
package com.jusdagloble.dac.module;


import com.jusdagloble.dac.constant.EffectiveMode;

import java.lang.annotation.*;

/**
 * 数据权限控制注解
 * 默认启动
 * rules对应DataAccessControlRule.name属性
 *
 * @author haoyu.guo
 */
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface DataAccessControl {

    /**
     * 当前方法需要启用的全量规则
     *
     * @return 规则名数组
     */
    String[] rules() default {};

    /**
     * 生效模式，默认是黑名单模式
     * 在代码中使用DacContextHolder.setSpeciallyRules(names)设置规则名
     * 黑名单禁用列表中的值
     * 白名单启用列表中的值
     * @return 生效模式
     */
    EffectiveMode mode() default EffectiveMode.BLACK;

    /**
     * 数据权限校验启用状态
     *
     * @return 启用状态
     */
    boolean enable() default true;

}

```

### DacContextHolder

编码时用于向当前线程的数据权限上下文设置参数



### DacAspect

拦截@DataAccessControl注解注释的方法

完成注解中规则的符号引用到对象引用的转换

完成注解到上下文的参数传递

完成方法执行后上下文的清理



### DacSqlStatementInspector

针对hibernate的扩展

在这里进行数据权限sql的织入





## 高级配置

### 

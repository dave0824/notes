## 前言
本篇博客记录SpringBoot项目中配置mybatis下划线转驼峰，返回类型为map时，属性值为null也返回

## yml配置

```yml
mybatis:
  configuration:
    call-setters-on-nulls: true # 属性为null时也返回
    map-underscore-to-camel-case: true # 下划线转驼峰
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl #打印sql语句 
```

## mybatis配置文件

```java

package com.unicom.admin.config;

import javax.sql.DataSource;

import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionFactoryBean;
import org.mybatis.spring.annotation.MapperScan;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.core.io.support.PathMatchingResourcePatternResolver;

/**
 * Mybatis配置
 */ 
@Configuration
@MapperScan("com.unicom.admin.*.dao")	// 扫描DAO
public class MybatisConfig {
  @Autowired
  private DataSource dataSource;

  @Bean
  @ConfigurationProperties(prefix = "mybatis.configuration")
  public org.apache.ibatis.session.Configuration globalConfiguration(){
    return new org.apache.ibatis.session.Configuration();
  }

  @Bean
  public SqlSessionFactory sqlSessionFactory(org.apache.ibatis.session.Configuration configuration) throws Exception {
    SqlSessionFactoryBean sessionFactory = new SqlSessionFactoryBean();
    sessionFactory.setDataSource(dataSource);
    sessionFactory.setTypeAliasesPackage("com.unicom.admin.*.model");	// 扫描Model
    sessionFactory.setConfiguration(configuration);
	PathMatchingResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
	sessionFactory.setMapperLocations(resolver.getResources("classpath*:**/sqlmap/*.xml"));	// 扫描映射文件
	
    return sessionFactory.getObject();
  }
}
```
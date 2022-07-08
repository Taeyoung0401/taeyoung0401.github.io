Spring Boot에서 MyBatis에서 사용 시 검토했던 내용 등을 기록한다.

<br>

# MyBatis 사용을 위한 설정

```java
@Configuration
@MapperScan("com.example.springmybatismodel.mapper")
public class PersistenceConfig {

    @Bean
    public DataSource dataSource() {
        DataSourceBuilder builder = DataSourceBuilder.create();
        builder.driverClassName("org.postgresql.Driver")
                .url("jdbc:postgresql://localhost:5432/mydb")
                .username("admin")
                .password("admin");
        return builder.build();
    }

    @Bean
    public SqlSessionFactory sqlSessionFactory() throws Exception {
        SqlSessionFactoryBean factoryBean = new SqlSessionFactoryBean();
        factoryBean.setDataSource(dataSource());
        return factoryBean.getObject();
    }
}
```

`@MapperScan`으로 Mapper가 정의된 패키지 위치를 지정한다.

<br>

### <참고>

정확한 원인은 모르겠지만, Properties에 몇가지 설정이 없으면 실행 시 Exception이 발생한다. 

```yml
spring:
  datasource:
    driver-class-name: org.postgresql.Driver
    ... // 어떤 설정이 더 필요한지 정확하게 테스트하지 않음:)

  sql:
    init:
      mode: always
```

아래 글도 읽어 보자.

> https://www.baeldung.com/spring-boot-configure-data-source-programmatic

<br>

# Multiple Datasource 정의

...
<br>

# Mapper 구현 방법

MyBatis Mapper 구현은 아래와 같이 2가지 방법이 있다.

## 1. Annotation를 이용하는 방법

아래와 같은 방식으로 구현되고, Interface의 정보를 기반으로 Runtime에 필요한 구현체가 자동 생성되는 방식이다.

```
@Mapper
public interface ItemMapper {

    @Select("SELECT * FROM items WHERE id = ${id}")
    Item getItem(int id);

    @Insert("INSERT INTO items (name, used) VALUES (#{name}, #{used})")
    @Options(useGeneratedKeys = true, keyProperty = "id")
    void insertItem(Item item);

    @Update("UPDATE items SET name = #{name}, used = #{used} where id = #{id}")
    boolean updateItem(Item item);
}

```

몇 가지 테스트 해본 결과 가장 큰 단점은 Join이 지원되지 않는 등, SQL를 사용하는데 제한이 많은 것 같다.

> https://mybatis.org/mybatis-3/java-api.html

```
NOTE You will notice that join mapping is not supported via the Annotations API. This is due to the limitation in Java Annotations that does not allow for circular references.
```

<br>

## 2. XML를 이용하는 방법

설명 생략. 

```xml
<select id="selectPerson" parameterType="int" resultType="hashmap">
  SELECT * FROM PERSON WHERE ID = #{id}
</select>
```

> 출처 https://mybatis.org/mybatis-3/sqlmap-xml.html

XML를 사용하여 쿼리를 작성하는게 많이 불편한 건 아니지만, 직관적이지도 않고 Annotation를 이용하는게 많이 편해 보이니, 두가지 방식을 혼합해서 사용하는게 가장 효과적인 방식인 것 같다.

<br>

# Transaction

`@Transactional`를 사용하여 쉽게 구현할 수 있다.  
아래는 간단한 테스트 코드 3번째 요청 시 두 테이블이 업데이트 된다.

```java
@Transactional
public User useItem(int userId, int itemId) {
    User user =  userMapper.getUser(userId);
    Item item = itemMapper.getItem(itemId);
    if(user==null || item==null) {
        log.error("not found user {} or item {}", userId, itemId);
        return null;
    }

    if(item.getUsed()) {
        log.error("item {} already be used", item.getName());
        return null;
    }

    item.setUsed(true);
    user.setItem(item.getId());

    itemMapper.updateItem(item);
    if(++retry<3) {
        log.error("failed to get item retry={}", retry);
        throw new RuntimeException("oops");
    }
    userMapper.updateUser(user);

    return user;
}
```

설명에 따르면 `@Transactional`은 Error나 RuntimeException이 throw되는 경우 rollback를 실행한다. (단, 별도의 설정이 있는 경우는 예외이고)

```
If no custom rollback rules are configured in this annotation, the transaction will roll back on RuntimeException and Error but not on checked exceptions.
```
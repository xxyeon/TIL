## MyBatis와 JPA의 차이는?

**JPA(ORM)** 은 객체와 관계를 매핑, 쿼리 자체를 작성하지 않아도됨.  
**SQL Mapper(MyBatis)**: SQL 문으로 객체들을 연결시킨다.쿼리 중심 개발

### MyBatis

MBatis의 핵심은 SQL 중심의 ORM인데, 즉 SQL을 직접 작성하면서 객체 매핑을 쉽게 할 수 있는 프레임워크이다.

```java
<!-- UserMapper.xml -->
<mapper namespace="com.example.UserMapper">
  <select id="getUserById" resultType="com.example.User">
    SELECT * FROM users WHERE id = #{id}
  </select>
</mapper>
```

```java
// UserMapper.java
public interface UserMapper {
    User getUserById(int id);
}
```

스프링이나 MyBatis가 런타임에 UserMapper 인터페이스에 대해 프록시 객체를 만들어서 XML의 SQL을 실행하도록 합니다.

```java
// 사용 예
UserMapper mapper = sqlSession.getMapper(UserMapper.class);
User user = mapper.getUserById(1); // -> XML의 SQL 실행
```

### JPA(Japa Persistence API)

JPA는 Java에서 객체와 관계형 데이터베이스 사이의 매핑을 처리해주는 ORM(Object-Relational Mapping) 기술의 표준 인터페이스입니다.

ORM은 데이터베이스 테이블을 자바 클래스에, 테이블의 컬럼을 클래스의 필드에 대응시켜 **객체 지향적인 방식**으로 **데이터**를 다룰 수 있게 해줍니다.  
JPA는 인터페이스이기 때문에 Hibernate, EclipseLink 같은 구현체가 필요하며, 스프링 부트에서는 기본적으로 Hibernate를 사용합니다.  
JPA를 사용하면 단순한 CRUD 작업은 SQL을 직접 작성하지 않고 메서드 호출만으로 처리할 수 있고, 객체 상태에 따라 자동으로 DML이 실행되기(더티체킹) 때문에 개발 생산성과 유지보수성이 높아집니다.

#### 참고자료

[MyBatis](https://www.elancer.co.kr/blog/detail/231)

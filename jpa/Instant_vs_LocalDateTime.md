# Instant vs LocalDateTime

## 문제

- JPA does not support `java.time.Instant`
    - timezone 이 무엇인지 알기 어렵기 때문에, TIMESTAMP 로 변환하기가 난감하다.

- 자바의 Instant 는 가장 DB 의 TIMESTAMP 타입과 유사하고, timezone 에 디펜던시가 없어서 그 순간이 정확하다.
- JPA 는 LocalDate, LocalTime, LocalDateTime 등을 지원하지만, Instant 는 지원하지 않는다.
    - [https://github.com/eclipse-ee4j/jpa-api/issues/163](https://github.com/eclipse-ee4j/jpa-api/issues/163) 에서 갑론을박 하면서, 어떤 사람은 이러한 AttributeConverter 를 써서 작업한다고 한다.

    ```java
    package Repository;

    import javax.persistence.AttributeConverter;
    import javax.persistence.Converter;
    import java.sql.Timestamp;
    import java.time.Instant;

    @converter(autoApply = true)
    public class InstantAttributeConverter implements AttributeConverter<Instant, Timestamp> {
    		
    		@Override
    		public Timestamp convertToDatabaseColumn(Instant instant) {
    		    if (instant == null)
    		        return null;
    		    else
    		    {
    		        Timestamp t = Timestamp.from(instant);
    		        return t;
    		    }
    		}
    		
    		@Override
    		public Instant convertToEntityAttribute(Timestamp timestamp) {
    		    //System.out.println(timestamp + " " + timestamp.toInstant() + " " + timestamp.getTimezoneOffset());
    		    return (timestamp == null ? null : timestamp.toInstant());
    		}
    }
    ```


## 해결

- Hibernate 에서 지원하는 자바의 타입에 Instant 는 없다.
- [https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#basic](https://docs.jboss.org/hibernate/orm/5.4/userguide/html_single/Hibernate_User_Guide.html#basic)

- 참고
    - [https://stackoverflow.com/questions/49309076/why-jpa-does-not-support-java-time-instant](https://stackoverflow.com/questions/49309076/why-jpa-does-not-support-java-time-instant)


- [Notion link](https://www.notion.so/Instant-vs-LocalDateTime-e42b1e7e7acd45a2b308fcc1ad757045)

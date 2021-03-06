# Spring MongoTemplate example
`MongoTemplate` can help you CURD documents in MongoDB easily. In this page I will show you how to use it. 
I would recommend you to learn [Mongo tutorial](http://www.henryxi.com/mongodb-tutorial) first, if you are 
unfamiliar with commends in MongoDB. For quick start I use Spring Boot to test MongoTemplate. 

**project structure**
```
└─main
    ├─java
    │  └─com
    │      └─henryxi
    │          └─mongo
    │              └─template
    │                      Address.java
    │                      QueryClient.java
    │                      User.java
    │
    └─resources
            application.properties
```
**pom**
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-mongodb</artifactId>
    <version>1.3.3.RELEASE</version>
</dependency>
```
**application.properties**
```ini
spring.data.mongodb.host=localhost
spring.data.mongodb.port=27017
spring.data.mongodb.database=user_database
```
**MongoTemplate example code**
```java
@SpringBootApplication
public class QueryClient implements CommandLineRunner {
    @Autowired
    private MongoTemplate mongoTemplate;

    public static void main(String[] args) {
        SpringApplication.run(QueryClient.class, args);
    }

    public void run(String... strings) throws Exception {
        // save data
        User henry = new User("Henry", 27);
        mongoTemplate.save(henry);
        Address beijing = new Address("China", "Beijing");
        Address shanghai = new Address("China", "Shanghai");
        User justin = new User("Justin", 28, beijing);
        User mathew = new User("Mathew", 23, shanghai);
        User charles = new User("Charles", 32, beijing);
        List<User> users = new ArrayList<User>();
        users.add(justin);
        users.add(mathew);
        users.add(charles);
        mongoTemplate.insert(users, User.class);

        //delete data
        String justinId = justin.getId();
        justin.setId("123456");
        mongoTemplate.remove(justin);//delete fail when change the id
        Query queryMathew = new Query();
        queryMathew.addCriteria(Criteria.where("name").is("Mathew"));
        mongoTemplate.remove(queryMathew, User.class);
        justin.setId(justinId);
        mongoTemplate.remove(justin);

        //query data
        Query queryHenry = new Query();
        queryHenry.addCriteria(Criteria.where("name").is("Henry"));
        List<User> usersInDB = mongoTemplate.find(queryHenry, User.class);
        System.out.println(usersInDB.get(0));
        String id = usersInDB.get(0).getId();
        User user = mongoTemplate.findById(id, User.class);
        System.out.println(user);

        //update data
        Query queryJustin = new Query();
        Update updateHenry = new Update().set("name", "new Henry");
        User oldHenry = mongoTemplate.findAndModify(queryHenry, updateHenry, User.class);//return old user object
        System.out.println(oldHenry);
        queryJustin.addCriteria(Criteria.where("name").is("Charles"));
        Update updateCharles = new Update().set("name", "new Charles");
        FindAndModifyOptions returnNew = new FindAndModifyOptions().returnNew(true);
        User newCharles = mongoTemplate.findAndModify(queryJustin, updateCharles, returnNew, User.class);//return new user object
        System.out.println(newCharles);
    }
}
```
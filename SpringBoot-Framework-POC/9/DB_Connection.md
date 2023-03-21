# DB Connection

### Task completed

1. Connect to Oracle DB
2. **SELECT** and **INSERT** to the DB
3. RESTful API calls handled with ProducerTemplate

## 1. Maven Dependencies

```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
<dependency>
	<groupId>org.apache.camel</groupId>
	<artifactId>camel-jdbc</artifactId>
	<version>3.14.5</version>
</dependency>
<!-- For mySQL DB -->
<dependency>
	<groupId>mysql</groupId>
	<artifactId>mysql-connector-java</artifactId>
	<scope>runtime</scope>
</dependency>
<!-- For Oracle DB -->
<dependency>
	<groupId>com.oracle.database.jdbc</groupId>
	<artifactId>ojdbc8</artifactId>
	<scope>runtime</scope>
</dependency>
```

## 2. Datasource in application.properties

Which doesn’t require to create a datasource bean, but just define the properties here for auto creation.

```php
// Oracle
spring.datasource.url=jdbc:oracle:thin:@[ip_address]:1521:[SID]
spring.datasource.username=username
spring.datasource.password=password

//mySQL
spring.datasoure.url=jdbc:mysql://[ip_address]:3306/[SID]
```

## 3. Class for Retrieved Data

Create a class with fields that the DB table return.

```java
public class UserRole{
	// Data Field
	private String userId;
	private String roleName;
	...
	
	// Getter and Setter
	// ...
}
```

## 4. Service Class for the Routes

Create a service class for routes handling the **SELECT** and **INSERT** queries to the DB.

### For Route 1 (**SELECT**):

- First set the SQL query into body, and pass to `jdbc:dataSource`
- Then get back the result and map to the role object
- **Speical handle for *timestamp* format
- Set body for the returned query

### For Route 2 (**INSERT**):

- With the data stored in JSON body (POST), enter process
- Build the query with the incoming data in JSON body
- Set the SQL query to the body and route to `jdbc:dataSource`
- **For Oracle, the date format is like ‘13-MAR-23’

```java
@Service
public class DBServiceImpl extends RouteBuilder {

    @Autowired
    DataSource dataSource;

    @Override
    public void configure() throws Exception{

        // 1. Select Route
        from("direct:select").setBody(constant("Select * From [table_name]"))
                .to("jdbc:dataSource").process(new Processor() {
                    @Override
                    public void process(Exchange exchange) throws Exception {
                        ArrayList<Map<String, String>> dataList = (ArrayList<Map<String, String>>) exchange.getIn()
                                .getBody();
                        List<UserRole> roles = new ArrayList<UserRole>();
                        System.out.println(dataList);
                        for (Map<String, String> data : dataList) {
                            UserRole role = new UserRole();
                            role.setUserId(data.get("USERID"));
                            role.setRoleName(data.get("ROLE_NAME"));
                            role.setCreateDate(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.S").format(data.get("CREATE_DATE")));
                            role.setCreateUserId(data.get("CREATE_USERID"));
                            role.setLastUpdateDate(new SimpleDateFormat("yyyy-MM-dd HH:mm:ss.S").format(data.get("LASTUPDATE_DATE")));
                            role.setLastUpdateUserId(data.get("LASTUPDATE_USERID"));
                            roles.add(role);
                        }
                        exchange.getIn().setBody(roles);
                    }
                });

        // 2. Insert Route
        from("direct:insert").process(new Processor() {
            @Override
            public void process(Exchange exchange) throws Exception {
                //Take the Employee object from the exchange and create the insert query
                UserRole role = exchange.getIn().getBody(UserRole.class);
                String query = "INSERT INTO cpms2_user_role(" +
                        "userid, role_name, create_date, create_userid, lastupdate_date, lastupdate_userid) values('"
                        + role.getUserId() + "','"
                        + role.getRoleName() + "','"
                        + role.getCreateDate() + "','"
                        + role.getCreateUserId() + "','"
                        + role.getLastUpdateDate() + "','"
                        + role.getLastUpdateUserId() +
                        "')";
                // Set the insert query in body and call camel jdbc
                exchange.getIn().setBody(query);
            }
        }).to("jdbc:dataSource");

    }
}
```

## 5. RestController with ProducerTemplate

Using **ProducerTemplate** to route incoming RESTful calls, to the respective endpoints.

- 1st parameter: endpoint name you are going
- 2nd parameter: body of the message (optional & skippable)
- 3rd parameter: returning class

```java
@RestController
public class DBController{
	@Autowired
	ProducerTemplate producerTemplate;

	@RequestMapping(value="/getRole")
	public List<UserRole> getRoles() {
		roles = producerTemplate.requestBody("direct:select", null, List.class);
		return roles;
	}

	@RequestMapping(value="/insertRole")
	public String insertRole(@RequestBody UserRole role) {
		producerTemplate.requestBody("direct:insert", role);
		return "Complete";
	}
}
```

## 6. RESTful Calls

Calling the API with the following calls with Postman, for the inserting date time, the date need to have the following format while inserting to Oracle DB.

```json
// GET: Selecting all records of the table
"http://localhost:9090/getRole"

// POST: Inserting a record to the table
"http://localhost:9090/insertRole"
{
    "userId": "test",
    "roleName": "test",
    "createDate": "21-MAR-23",
    "createUserId": "test",
    "lastUpdateDate": "21-MAR-23",
    "lastUpdateUserId": "test" 
}
```

### Reference:

[https://www.javainuse.com/camel/camel_jdbc](https://www.javainuse.com/camel/camel_jdbc)
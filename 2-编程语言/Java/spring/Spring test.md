# Spring test

```java
//配置spring环境
@RunWith(SpringJUnit4ClassRunner.class)
@ContextConfiguraion("classpath:xxx.xml")

public class Test{
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    @Test
    public void test(){
        
    }
}
```


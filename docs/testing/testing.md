## TESTING



### 

`setupDb(), annotated with ***@BeforeEach, which is executed before each test method`

reference: 
```java

@DataMongoTest 
class PersistenceTests {
    @Autowired 
    private ProductRepository repository; private ProductEntity savedEntity;

    @BeforeEach 
    void setupDb() {
        repository.deleteAll(); 
        ProductEntity entity = new ProductEntity(1, "n", 1); 
        savedEntity = repository.save(entity);
        assertEqualsProduct(entity, savedEntity);
    }

    @Test 
    void create() { 
        ProductEntity newEntity = new ProductEntity(2, "n", 2); 
        repository.save(newEntity);
    
        ProductEntity foundEntity = repository.findById(newEntity.getId()).get(); 
        assertEqualsProduct(newEntity, foundEntity);
        assertEquals(2, repository.count());
    }

    @Test 
    void update() {
        savedEntity.setName("n2"); 
        repository.save(savedEntity);
        ProductEntity foundEntity = repository.findById(savedEntity.getId()).get(); 
        assertEquals(1, (long)foundEntity.getVersion()); 
        assertEquals("n2", foundEntity.getName());
    }
    }
```

### OPTIMISTIC LOCK TESTING

```java
@DataMongoTest
class PersistenceTests {
    @Autowired
    private ProductRepository repository;
    private ProductEntity savedEntity;

    @Test
    void optimisticLockError() {
        // Store the saved entity in two separate entity objects  
        ProductEntity entity1 = repository.findById(savedEntity.getId()).get();
        ProductEntity entity2 = repository.findById(savedEntity.getId()).get();

        // Update the entity using the first entity object 
        entity1.setName("n1");
        repository.save(entity1);

        // Update the entity using the second entity object. 
        // This should fail since the second entity now holds an old version 
        // number, that is, an Optimistic Lock Error 
        assertThrows(OptimisticLockingFailureException.class, () -> {
            entity2.setName("n2");
            repository.save(entity2);
        });
        // Get the updated entity from the database and verify its new state 
        ProductEntity updatedEntity = repository.findById(savedEntity.getId()).get();
        assertEquals(1, (int) updatedEntity.getVersion());
        assertEquals("n1", updatedEntity.getName());
    }
}
```

```
1. First, the test reads the same entity twice and stores it in two different variables, entity1 and entity2.

2. Next, it uses one of the variables, entity1, to update the entity. 
   The update of the entity in the database will cause the version field of the
   entity to be increased automatically
   by Spring Data. The other variable, entity2, now contains stale data, 
   manifested by its version field, which holds a lower value than the corresponding value in the database.

3. When the test tries to update the entity using the variable entity2, which contains stale data, 
    it is expected to fail by throwing an OptimisticLockingFailureException exception.

4. The test wraps up by asserting that the entity in the database reflects the first update, that is, contains the name "n1",
 and that the version field has the value 1; only one update has been performed on the entity in the database.
```
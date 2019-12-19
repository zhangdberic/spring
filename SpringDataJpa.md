

# Spring Data JPA Respository

借助Spring Data，以接口定义的方式创建Respository，Spring Data JPA会在spring启动的时，自动分析接口的声明，并动态生成实现类。

## 1.简单的例子

例如，声明一个实体Spitter；

```java
@Entity  
public class Spitter {
    @Id
    private long id;  
    private String userName, passWord, fullName;  
}    
```

例如，声明一个Spitter实体类的持久化接口；

```java
import org.springframework.data.jpa.repository;

public interface SpitterRepository extends JpaRespository<Spitter,Long>{
}
```

通过继承Spring Data JPA的JpaRespository接口，并进行了参数化<Spitter,Long>，它知道这是一个持久化Spitter的Repository，并且Spitter的ID类型是Long。另外，它还会继承了18个执行持久化操作的通用方法，如：保存、删除、根据ID查询等。

JPA配置器，开启JPA仓库，并制定了要扫描的包位置

```java
@Configuration
@EnableJpaRepositories(backPackages="com.habuma.spitter.db")
public class JapConfiguration{
}
```

## 2. Spring Data JPA Repository方法

Spring Data JPA Repository的方法，可以分为四类：

1. **JpaRespository接口自带的方法。**
2. **定义查询方法(根据有意义的方法名和返回值类型，来确定要操作数据范围)**
3. **使用@Query源注释来自定义HQL和SQL。**
4. **通过自定义的Impl后缀类，使用最原生JPA的EntityManager来操作数据。**

如果从另一个角度介绍：

Spring Data Jpa中一共提供了

- Repository：

  - 　　提供了findBy + 属性方法 
  - 　　@Query 
    - 　　HQL： nativeQuery 默认false
    - 　　SQL: nativeQuery 默认true
      - 更新的时候，需要配合@Modifying使用

- CurdRepository:

  - 继承了Repository 主要提供了对数据的增删改查

- PagingAndSortRepository:

- - 继承了CrudRepository 提供了对数据的分页和排序，缺点是只能对所有的数据进行分页或者排序，不能做条件判断

- JpaRepository： 继承了PagingAndSortRepository

- - 开发中经常使用的接口，主要继承了PagingAndSortRepository，对返回值类型做了适配

- JpaSpecificationExecutor

  - 提供多条件查询，复杂动态查询



### 2.1 JpaRespository接口自带的方法

首先介绍`JpaRepository`，它继承自`PagingAndSortingRepository`，而`PagingAndSortingRepository`又继承自`CrudRepository`。

每个都有自己的功能：

- CrudRepository提供CRUD的功能。
- PagingAndSortingRepository提供分页和排序功能
- JpaRepository提供JPA相关的方法，如刷新持久化数据、批量删除。

由于三者之间的继承关系，所以**JpaRepository包含了CrudRepository和PagingAndSortingRepository所有的API**。

当我们不需要JpaRepository和PagingAndSortingRepository提供的功能时，可以简单使用CrudRepository。

**CrudRepository接口方法**

```java
public interface CrudRepository<T, ID extends Serializable>
  extends Repository<T, ID> {
 
    <S extends T> S save(S entity);
 
    T findOne(ID primaryKey);
 
    Iterable<T> findAll();
 
    Long count();
 
    void delete(T entity);
 
    boolean exists(ID primaryKey);
}
```

**PagingAndSortingRepository接口方法**

```java
public interface PagingAndSortingRepository<T, ID extends Serializable> 
  extends CrudRepository<T, ID> {
 
    Iterable<T> findAll(Sort sort);
 
    Page<T> findAll(Pageable pageable);
}
```

该接口提供了`findAll(Pageable pageable)`这个方法，它是实现分页的关键。
 使用Pageable时，需要创建Pageable对象并至少设置下面3个参数：

- 1、 每页包含记录数
- 2、 当前页数
- 3、 排序
   假设我们要显示第一页的结果集，一页不超过5条记录，并根据`lastName`升序排序，下面代码展示如何使用PageRequest和Sort获取这个结果：

```java
Sort sort = new Sort(new Sort.Order(Direction.ASC, "lastName"));
Pageable pageable = new PageRequest(0, 5, sort);
```

**JpaRepository接口**

```java
public interface JpaRepository<T, ID extends Serializable> extends
  PagingAndSortingRepository<T, ID> {
 
    List<T> findAll();
 
    List<T> findAll(Sort sort);
 
    List<T> save(Iterable<? extends T> entities);
 
    void flush();
 
    T saveAndFlush(T entity);
 
    void deleteInBatch(Iterable<T> entities);
}
```

显然，JpaRepository接口继承自PagingAndSortingRepository，PagingAndSortingRepository继承自CrudRepository，所以JpaRepository拥有CrudRepository所有的方法。



### 2.2 定义查询方法(根据方法名和返回值类型来查询数据)

例如：

```java
public interface SpitterRepository extends JpaRepository<Spitter,Long> {
    Spitter findByUsername(String username);
}
```

Spring Data会检查Repository接口的所有方法，解析方法名称，并结合方法的返回值类型和JpaRepository接口源注释，来生成对应的SQL操作数据库。

Repository方法的签名是由一个动词、一个可选主题（Subject）、关键词By、一个断言所组成的。主题可以忽略，因此JpaRespository接口的源注释已经声明了主题。如上的例子，讲解：动词是find，断言是Username，主题没有指定，使用接口声明的Spitter。

动词可以是：get、read、find和count，前三个都是一样(同义)，都是查看数据转换为对象并返回后，只有count是统计匹配对象数量。

基于定义查询方法获取数据，虽然简单并且可读性也好，但如果查询条件比较多或者复杂，则就没有可读性了，而且方法签名一旦改变，就改变了SQL的语义，要小心。例如：

```java
List<Spitter> findByFirstnameOrLastnumOrderByLastnameAscFirstnameDesc(String first,String last);
```

基于定义查询方法的来获取数据，适合查询条件少，并不复杂的情况，例如：上面最开始定义的方法，只有一个条件。复杂的条件，建议使用下面的@Query。

### 2.3 使用@Query自定义HQL和SQL操作

例如：

```java
@Query("select s from Spitter s where s.email like '%gmail.com%'")
List<Spitter> findAllGmailSpitters();
```



试想一下，如果我们想自己定义执行查询，利用命名查询，显然不行，因为，会在实体类上写很多的@NamedQuery，这种情况的话，我们可以用@Query直接在方法上定义查询语句，例如这样

```java
public interface TaskDao extends JpaRepository<Task, Long> {
  @Query("select t from Task t where t.taskName = ?1")
  Task findByTaskName(String taskName);
}
```

 

@Query上面的1代表的是方法参数里面的顺序，除了写hql，我们还可以写sql语句

```java
public interface TaskDao extends JpaRepository<Task, Long> {
  @Query("select * from tb_task t where t.task_name = ?1", nativeQuery = true)
  Task findByTaskName(String taskName);
}
```

 

在参数绑定上，我们还可以这样子用

```java
public interface TaskDao extends JpaRepository<Task, Long> {
   @Query("select t from Task t where t.taskName = :taskName and t.createTime = :createTime")
  Task findByTaskName(@Param("taskName")String taskName,@Param("createTime") Date createTime);
}
```

 

当然在参数绑定上，我们还可以直写问号

```java
public interface TaskDao extends JpaRepository<Task, Long> {
   @Query("select t from Task t where t.taskName = ? and t.createTime = ?")
  Task findByTaskName(String taskName, Date createTime);
}
```

 

利用@Modifying+@Query进行更新

```java
@Modifying
@Query("update Task t set t.taskName = ?1 where t.id = ?2")
int updateTask(String taskName, Long id);
```

### 2.4 自定义方法操作(使用原生JPA的EntityManager来操作数据)

有些时候，上面的手段都无法操作数据，我们需要使用原始的JPA EntityManager来操作数据。

```java
public class SpitterRepositoryImpl implements SpitterSweeper {
    @PersistenceContext
    private EntityManager em;
    
    public int eliteSweep(){
        String update = 
            "UPDATE Spitter spitter" +
            "SET spitter.status = 'Elite'" +
            "WHERE spitter.status = 'Newbie'" +
            "AND spitter.id IN (" +
            "SELECT s FROM Spitter s WHERE(" +
            " SELECT COUNT(Spitters) FROM s.spitters spitters) > 10000)";
        return em.createQuery(update).executeUpdate();
            
    }
}
```

当Spring Data JPA为Repository接口生成实现的时候(CGLIB)，它还会查找名字与接口相同，并且添加了Impl后缀的一个类，Spring Data JPA将会把Impl类的方法和接口生成的方法合并在一起，例如：对应SpitterRepository接口而言，要查找的类名SpitterRepositoryImpl。

注意：SpitterRepositoryImpl并没有实现SpitterRepository接口。Spring Data JPA 负责来实现(CGLIB)这个接口。SpitterRepositoryImpl实际实现的是SpitterSweeper接口，如下：

```java
public interface SpitterSweeper {
    int eliteSweep();
}
```

我们还要确保SpitterSweeper被SpitterRepository接口继承，如下：

```java
public interface SpitterRepository extends JpaRepository<Spitter,Long>,SpitterSweeper {}
```

通过上面的操作，我们就使用eliteSweep()方法来操作数据库了，如下：

```java
@Autowired 
private SpitterRepository spitterRepository;

spitterRepository.eliteSweep();
```


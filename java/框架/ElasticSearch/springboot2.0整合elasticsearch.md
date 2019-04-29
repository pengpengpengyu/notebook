# Spring-data-elasticsearch

## @Document

---

* `@Document(indexName = "es",type = "user",shards = 5,replicas = 0)` ： 标注在实体类上，声明存储的索引和类型

* `indexName`： 索引名称
* `type`：索引类型
* `shards`：分片的数量
* `replicas`：副本的数量
* `refreshInterval`： 刷新间隔
* `indexStoreType`：索引文件存储类型

## @Field

---

* 标注在属性上，用来指定属性的类型。其中的属性如下：
  * `analyzer`：指定分词器，es中默认使用的标准分词器，比如我们需要指定中文IK分词器，可以指定值为ik_max_word
  * `type`： 指定该属性在es中的类型，其中的值是FileType类型的值，比如FileType.Text类型对应es中的text类型
  * `index`：指定该词是否需要索引，默认为true
  * `store`：指定该属性内容是否需要存储，默认为
  * `fielddata` ：指定该属性能否进行排序，因为es中的text类型是不能进行排序（已经分词了）
  * `searchAnalyzer` ： 指定搜索使用的分词器
* 在插入数据之前我们需要先运行程序添加mapping，对于没有指定`@Field`的属性此时是不会创建索引的，而是在插入数据的时候自动创建索引。但是对于`@Field`注解标注的属性如果没有先加载生成mapping，等到插入数据的时候是没有效果的
* 如果使用该注解，那么必须指定其中的type属性

## *@Id

---

主键注解，标识一个属性为主键

## Date类型的存储

* es中默认存储Date类型的是一个时间戳，如果我们需要指定格式的存储，那么需要在`@Field`这个注解中指定日期的格式。如下：

```java{.line-numbers}
@Field(type = FieldType.Date,format = DateFormat.custom, pattern ="yyyy-MM-dd HH:mm:ss")
@JsonFormat(shape = JsonFormat.Shape.STRING, pattern ="yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
 private Date birthday;
 ```

## 实体类

```java{.line-numbers}
/**
 * @Document : 这个是ES的注解，在类使用，指定实体类的索引和类型。默认所有的属性都是索引的
 *             1、indexName ：　指定索引
 *             2、type：指定类型
 *             3、shards：指定分片的数量
 *             4、replicas：指定副本的数量
 */
@Document(indexName = "es",type = "user",shards = 5,replicas = 0)
public class User {
    @Id   //指定这个是主键
    private Integer userId;
    @Field(type = FieldType.Text,analyzer = "ik_max_word",fielddata = true,store = false)
    private String userName;
    private String password;
    @Field(type = FieldType.Date, store = true, format = DateFormat.custom, pattern ="yyyy-MM-dd HH:mm:ss")
    @JsonFormat(shape = JsonFormat.Shape.STRING, pattern ="yyyy-MM-dd HH:mm:ss", timezone = "GMT+8")
    private Date birthday;
    private List<String> hobbies;
    public Integer getUserId() {
        return userId;
    }
    public void setUserId(Integer userId) {
        this.userId = userId;
    }
    public String getUserName() {
        return userName;
    }
    public void setUserName(String userName) {
        this.userName = userName;
    }
    public String getPassword() {
        return password;
    }
    public void setPassword(String password) {
        this.password = password;
    }
    public Date getBirthday() {
        return birthday;
    }
    public void setBirthday(Date birthday) {
        this.birthday = birthday;
    }
    public List<String> getHobbies() {
        return hobbies;
    }
    public void setHobbies(List<String> hobbies) {
        this.hobbies = hobbies;
    }
    @Override
    public String toString() {
        return "User{" +
                "userId=" + userId +
                ", userName='" + userName + '\'' +
                ", password='" + password + '\'' +
                ", birthday=" + birthday +
                ", hobbies=" + hobbies +
                '}';
    }
}
```

## 定义查询接口

* 官网上提供了各种各样的方法，我们使用继承`ElasticsearchRepository`这个接口的方式拓展查询接口，基本的接口：

```java{.line-numbers}
public interface UserRepo extends ElasticsearchRepository<User,Integer> {
    //不需要实现其中的方法，只需要继承即可，spring-data-es会为我们自动完成

}
```

## 常用方法：

1. `index(T t)` ：添加数据
1. `save(T t)`：添加数据
1. `count()`： 获取数据总数
1. `findAll()`：获取所有的数据，返回的是一个java.lang.Iterable
1. `Iterable<T> findAllById(Iterable<ID> ids)`：根据Id批量返回数据
1. `saveAll(Iterable entity)` ：批量保存数据，可以传入List
1. `delete(T t)` ： 删除指定的实体类，只需要指定实体类中的Id即可
1. `deleteAll()`：删除所有的数据
1. `deleteById(ID Id)`：根据Id删除数据
1. `existsById(ID Id)`： 判断指定Id的数据是否存在

 ```java {.line-numbers}
 //添加数据
    @Test
    public void test3(){
        User user=new User();
        user.setUserId(1);
        user.setUserName("郑元梅");
        user.setBirthday(new Date());
        user.setPassword("12345678");
        List<String> hobbies=new ArrayList<>();
        hobbies.add("篮球");
        hobbies.add("足球");
        user.setHobbies(hobbies);
//        userRepo.save(user);   //调用其中的save方法保存信息
        userRepo.index(user);  //调用index方法添加数据
    }
    //获取其中的所有数据
    @Test
    public void test4(){
        Iterable<User> iterable=userRepo.findAll();
        Iterator<User> iterator=iterable.iterator();
        while (iterator.hasNext()){
            System.out.println(iterator.next());
        }
    }
    @Test
    public  void test5(){
        List<User> users=new ArrayList<>();
        User user=new User();
        user.setUserId(4);
        user.setUserName("张三");
        user.setBirthday(new Date());
        user.setPassword("12345678");
        List<String> hobbies=new ArrayList<>();
        hobbies.add("台球");
        hobbies.add("足球");
        user.setHobbies(hobbies);
        User user1=new User();
        user1.setUserId(5);
        user1.setUserName("郑元梅");
        user1.setBirthday(new Date());
        user1.setPassword("12345678");
        user1.setHobbies(hobbies);
        users.add(user);
        users.add(user1);
        userRepo.saveAll(users);  //保存List中的所有数据
    }
    //删除指定的数据
    @Test
    public void test6(){
        User user=new User();
        user.setUserId(5);
        userRepo.delete(user);
    }
    @Test
    public void test7(){
        List<User> users=userRepo.selectAll();
        for (User user
             :users
             ) {
            System.out.println(user);
        }
    }
```

## 自定义查询

* spring-data-elasticsearch为我们自动完成了许多的查询，我们只需要按照其中的规范使用即可。
  * 查询方法定义以`get`或者`find`开头即可
* 关于es中各种查询，我们可以参照下表进行定义，[文档](https://docs.spring.io/spring-data/elasticsearch/docs/current/reference/html/#elasticsearch.query-methods.criterions)

### 实例

```java {.line-numbers}
package com.techwells.es;
import com.techwells.beans.User;
import org.springframework.data.domain.Page;
import org.springframework.data.domain.Pageable;
import org.springframework.data.elasticsearch.annotations.Document;
import org.springframework.data.elasticsearch.annotations.Query;
import org.springframework.data.elasticsearch.repository.ElasticsearchRepository;
import java.util.Date;
import java.util.List;
public interface UserRepo extends ElasticsearchRepository<User,Integer>{
    /**
     * 根据userId获取用户信息
     * @param userId
     * @return
     */
    User findUserByUserId(Integer userId);
    /**
     * 根据用户查找用户信息
     * @param userName
     * @return
     */
    List<User> findByUserName(String userName);
    /**
     * 根据用户名和密码查找用户信息，使用的是must查询
     * 参数的顺序不能颠倒
     * @param userName
     * @param password
     * @return
     */
    List<User> findByUserNameAndPassword(String userName,String password);
    /**
     * 根据用户名或者地址进行查询，满足其一即可，使用的是should
     * 参数不能颠倒
     * @param userName
     * @param address
     * @return
     */
    List<User> findByUserNameOrAddress(String userName,String address);
    /**
     * 使用@Query注解自定义查询语句，其中的?是占位符，0表示第一个参数
     * @param userName
     * @return
     */
    @Query("{\n" +
            "    \"bool\": {\n" +
            "      \"must\": [\n" +
            "        {\n" +
            "          \"match\": {\n" +
            "            \"userName\": \"?0\"\n" +
            "          }\n" +
            "        }\n" +
            "      ]\n" +
            "    }\n" +
            "  }")
    List<User> selectByUserName(String userName);
    /**
     * 查询密码不为null的用户信息
     * @return
     */
    @Query("{\n" +
            "    \"bool\": {\n" +
            "      \"must\":{\n" +
            "        \"exists\":{\n" +
            "          \"field\":\"password\"\n" +
            "        }\n" +
            "      }\n" +
            "    }\n" +
            "  }")
    List<User> findByPasswordIsNotNull();
    /**
     * 查询密码为null的用户信息
     * @return
     */
    @Query("{\n" +
            "    \"bool\": {\n" +
            "      \"must_not\":{\n" +
            "        \"exists\":{\n" +
            "          \"field\":\"password\"\n" +
            "        }\n" +
            "      }\n" +
            "    }\n" +
            "  }")
    List<User> findByPasswordIsNull();
    /**
     * 查询密码不是password的用户信息，使用的must_not
     * @param password
     * @return
     */
    List<User> findByPasswordNot(String password);
    /**
     * 查询用户名是userName但是密码表示password的信息，必须同时满足
     * @param userName
     * @param password
     * @return
     */
    List<User> findByUserNameAndPasswordNot(String userName,String password);
    /**
     * 查询年龄在from-to之间的用户，包含form和to，使用的是range查询
     * @param from  起始
     * @param to    截止
     * @return
     */
    List<User> findByAgeBetween(Integer from,Integer to);
    /**
     * 查询年龄小于age的用户信息
     * @param age  年龄
     * @return
     */
    List<User> findByAgeLessThan(Integer age);
    /**
     * 年龄小于等于age的用户信息
     */
    List<User> findByAgeLessThanEqual(Integer age);
    /**
     * 年龄大于age的用户
     * @param age
     * @return
     */
    List<User> findByAgeGreaterThan(Integer age);
    /**
     * 年龄大于等于age的用户
     * @param age
     * @return
     */
    List<User> findByAgeGreaterThanEqual(Integer age);
    /**
     * 年龄小于等于age的用户信息
     * @param age
     * @return
     */
    List<User> findByAgeBefore(Integer age);
    /**
     * 年龄大于等于age的用户
     * @param age
     * @return
     */
    List<User> findByAgeAfter(Integer age);
    /**
     * 模糊查找，密码中以pwd开头用户信息，`content%`，
     * @param content
     * @return
     */
    List<User> findByPasswordLike(String content);
    /**
     * 查询密码中包含content的用户信息  %content%
     * @param content
     * @return
     */
    List<User> findByPasswordContaining(String content);
    /**
     * 查询密码以pwd开头的用户信息，和Like一样的效果
     * @param pwd
     * @return
     */
    List<User> findByPasswordStartingWith(String pwd);
    /**
     * 查询密码以pwd结尾的用户信息
     * @param pwd
     * @return
     */
    List<User> findByPasswordEndingWith(String pwd);
    /**
     * 查找年龄在集合中的用户信息
     * @param ages
     * @return
     */
    List<User> findByAgeIn(List<Integer> ages);
    /**
     * 查找年龄不在集合中的用户信息
     * @param ages
     * @return
     */
    List<User> findByAgeNotIn(List<Integer> ages);
    /**
     * 根据用户名查询并且按照年龄降序排列
     * @param userName
     * @return
     */
    List<User> findByUserNameOrderByAgeDesc(String userName);
    /**
     * 根据用户名查询并且按照年龄降序排列、用户名升序排列
     * @param userName
     * @return
     */
    List<User> findByUserNameOrderByAgeDescUserNameAsc(String userName);
    /**
     * 根据出生日期进行降序排列
     * @param userName
     * @return
     */
    List<User> findByUserNameOrderByBirthdayDesc(String userName);
    /**
     * 返回前2条数据
     * @param userName
     * @return
     */
    List<User> findTop2ByUserName(String userName);
    
    /**
     * 根据用户名分页查询
     * @param userName
     * @param pageable
     * @return
     */
    Page<User> findByUserName(String userName, Pageable pageable);
}
```

## 使用@Query定义自己的es语句

```java {.line-numbers}
/**
     * 使用@Query注解自定义查询语句，其中的?是占位符，0表示第一个参数
     * @param userName
     * @return
     */
    @Query("{\n" +
            "    \"bool\": {\n" +
            "      \"must\": [\n" +
            "        {\n" +
            "          \"match\": {\n" +
            "            \"userName\": \"?0\"\n" +
            "          }\n" +
            "        }\n" +
            "      ]\n" +
            "    }\n" +
            "  }")
    List<User> selectByUserName(String userName);
```

## 分页查询

[官网说明](https://www.tianmaying.com/tutorial/spring-jpa-page-sort)
* 直接使用org.springframework.data.domain.Pageable进行分页排序即可
  * `page`：从０开始，第几页，默认为0
  * `size`：每页显示的数量
  * `sort`：排序的方向
* 其中的方法如下：
  * `getTotalElements()`：返回数据的总数，不是分页的总数，而是根据条件查询到的全部的数据的总数
  * `getContent()`：获取分页的数据集合List<T>
  * `getTotalPages()`：获取总共几页的数据
  * `iterator()`：获取迭代器
剩余的方法如下：

```java {.line-numbers}
public interface Slice<T> extends Streamable<T> {
	//返回当前是第几页
	int getNumber();
	//返回每页显示的数量
	int getSize();
	//返回当前页获取到的元素数量
	int getNumberOfElements();
	//返回当前页元素的集合
	List<T> getContent();
	//判断当前页是否存在数据
	boolean hasContent();
	//获取排序的Sort
	Sort getSort();
	//判断当前页是否是第一页
	boolean isFirst();
	//判断当前页是否是最后一页
	boolean isLast();
	//判断是否还有下一页
	boolean hasNext();
	//判断是否有前一页
	boolean hasPrevious();
	//返回当前页的pageable
	default Pageable getPageable() {
		return PageRequest.of(getNumber(), getSize(), getSort());
	}
	//返回下一页的Pageable
	Pageable nextPageable();
	//返回前一页的pageable
	Pageable previousPageable();
    
	<U> Slice<U> map(Function<? super T, ? extends U> converter);
}
```

## 单条件分页排序

* 只使用了一个字段进行排序

```java{.line-numbers}
@Test
   public void test3(){
       Sort sort=new Sort(Sort.Direction.DESC,"age");
       Pageable pageable=new PageRequest(9,1,sort);
       Page<User> users=userRepo.findByUserName("李",pageable);
       System.out.println(users.getTotalPages());
       for (User user:users.getContent()) {
           System.out.println(user);
       }
   }
```

## 多条件分页排序

* 使用`Order`进行排序条件

```java{.line-numbers}
@Test
  public void test3(){
      List<Sort.Order> orders=new ArrayList<>();
      orders.add(new Sort.Order(Sort.Direction.DESC,"age"));//按照年龄降序排列
      orders.add(new Sort.Order(Sort.Direction.ASC,"userId"));  //按照用户Id升序排列
      Sort sort=new Sort(orders);  //使用orders
      Pageable pageable=new PageRequest(0,10,sort);
      Page<User> users=userRepo.findByUserName("李",pageable);
      System.out.println(users.getTotalPages());
      for (User user:users.getContent()) {
          System.out.println(user);
      }
  }
  ```
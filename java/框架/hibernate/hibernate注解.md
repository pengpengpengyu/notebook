# Hibernate注解说明

* `@Entity` 指定当前被修饰的类是一个实体类，用于映射到数据库中的表。

* `@Table(name = "userInfo")` 详细指定了该类映射到数据库中的哪张表，这里映射到userInfo表。

* `@Id` 指定被修饰的属性将映射到数据表的主键列。
联合主键的映射可以通过多个@Id进行修饰即可，但要求该实体类必须继承 java.io.Serializable并尽可能的重写Object的两个方法，hashCode和equals，因为多个属性唯一确定一条记录，自然需要比较属性的值

* `@GeneratedValue(strategy = GenerationType.IDENTITY)` 该注解指定了主键的生成策略，一般不单独出现，这里指定了主键自增的策略为使用数据库的自增长。
  **生成策略**
  * `GenerationType.AUTO` hibernate默认为该值，它指明了hibernate自动根据底层数据库选择适当的生成策略
  * `GenerationType.IDENTITY` 适用于MySQL，SQLserver的主键自增长策略
  * `GenerationType.SEQUENCE`适用于Oracle的子串策略
  * `GenerationType.TABLE` 基于辅助表的生成主键策略

* `@Column` 映射数据表中字段
默认情况下，我们不使用@Column修饰属性的时候，hibernate会自动以该属性的名称映射到数据表中的列。

  **属性如下**

  * `name`指定该属性映射到数据表中对应的名称
  * `nullable` 指定该属性映射的数据表中列是否可以为null，默认为true
  * `unique` 指定该属性映射到数据表中的列是否具有唯一约束
  * `length` 指定该属性映射到数据表中的列所能保存数据的最大长度，默认是255

* `@Transient` 指定某个字段不存在数据表中

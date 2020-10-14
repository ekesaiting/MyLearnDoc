

## 数据库建库规约

* 使用和应用名相同的数据库名,且必须全部小写字母，多个单词使用`_`连接
* 字符集字段使用`utf8mb4`，不使用`utf8`,排序规则 `COLLATE`使用`utf8mb4_general_ci`或MySQL默认`utf8mb4_0900_ai_ci`
* **==MySQL的“utf8mb4”是真正的“UTF-8”。utf8”只支持每个字符最多三个字节，而真正的UTF-8是每个字符最多四个字节。==**

## 数据库建表规约

通用：

* 表名、字段名必须使用==\` 名称\`==,避免与数据库或SQL关键字冲突

* 表名、字段名必须使用小写字母或数字，不能数字打头

表名：

* 表名不使用复数名词。不使用如`users`的名称，更换成如`user_info`
* 表的命名最好是遵循“业务名称_表的作用”。如`order`更换成`order_info` `order_detail`

字段：

* 字段应该有前缀，避免含义不明，不使用`id`,`name`,`desc`等字段，更换成`user_id`，`phone_name`,`role_desc`

* 主键必须使用`int(M) unsigned`或`bigint(M) unsigned`，M是数据的位数，不足将使用零填充，超过则无意义 平时使用**建议不写**，使用数据库默认值，**因为写不写根本没区别**

* `char(n)` 和 `varchar(n) `中括号中 n 代表字符的个数，并不代表字节个数，比如 CHAR(30) 就可以存储 30 个字符。

* 表必备三字段：`id`, `gmt_create`, `gmt_modified`，平时使用`表名_id`，`create_time`,`update_time`方便些

* 字段名后必须有`comment`，解释字段名的意思

* 不能为空的字段必须添加`not null`

* 小数统一使用整数或bigdecimal表示，不能使用float或者double

  | 类型         | 大小(字节) | **范围（有符号）**                                          | **范围（无符号）**              | mysql映射          |
  | ------------ | :--------: | ----------------------------------------------------------- | ------------------------------- | ------------------ |
  | tinyint      |     1      | (-128，127)                                                 | (0，255)                        | Byte               |
  | smallint     |     2      | (-32 768，32 767)                                           | (0，65 535)                     | Short              |
  | int /integer |     4      | (-8 388 608，8 388 607)                                     | (0，42 9496 7295)               | Integer            |
  | bigint       |     8      | (-922 3372 0368 5477 5808，<br />9 223 372 036 854 775 807) | (0，18 446 744 073 709 551 615) | Long               |
  | decimal      |  见:one:   |                                                             |                                 | BigDecimal         |
  | DATE         |     3      | 1000-01-01/9999-12-31                                       |                                 | Date/LocalDate     |
  | DATETIME     |     8      | 1000-01-01 00:00:00/9999-12-31 23:59:59                     |                                 | DATE/LocalDateTime |
  | TIMESTAMP    |     8      | 1970-01-01 00:00:00/2038                                    |                                 | DATE/LocalDateTime |
  | CHAR         |   0-255    |                                                             |                                 | String             |
  | VARCHAR      |  0-65535   |                                                             |                                 | String             |

  > :one: :   对DECIMAL(M,D) 如果M>D，为M+2否则为D+2,如`decimal(8,3) `,表示88888.000，表示该列数据占用10个字节，整数字段长度为5，小数字段为3

约束与索引：

* 所有约束与索引与字段分离，写在所有字段之后type
* 主键必须无意义，如果需要一个字段唯一代表某一类型，可以使用如：`category_type`字段，并设置`unique()`索引

## 模板

```sql
--建表模板
drop table if exist `table_name`;
crate table 'table_name'(
 `table_id` int not null auto_increment comment '主键id',
 `table_name` varchar(64) NOT NULL COMMENT 'xx名称',
    
  `version` int deafult 1 not null comment '乐观锁';
  `deleted` tinyint default 0 not null comment '逻辑删除';
  `create_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP COMMENT '创建时间',
  `update_time` datetime NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '修改时间',
  PRIMARY KEY (`order_id`)
)ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='表名';
```

```java
//对应java代码
private Integer tableId;
private String tableName;
private Integer version;
private Byte deleted;
private LocalDateTime createTime;
private LocalDateTime updateTime;
```



## orm映射

**javaType**：可以使用别名，如果你映射到一个 JavaBean，MyBatis ==通常可以推断类型==。然而，如果你映射到的是 `HashMap`,那么你应该明确地指定 javaType 来保证行为与期望的相一致。

**jdbcType**:只需要在可能执行插入、更新和删除的==且允许空值==的列上指定 JDBC 类型。**这是 JDBC 的要求**而非 MyBatis 的要求。如果你直接面向 JDBC 编程，你需要对可以为空值的列指定这个类型。

## 基本数据类型

int(M) ,tinyint(M)

**这里的M代表的并不是存储在数据库中的具体的长度，以前总是会误以为int(3)只能存储3个长度的数字，int(11)就会存储11个长度的数字，这是大错特错的。**

tinyint(1) 和 tinyint(4) 中的1和4并不表示存储长度，只有字段指定zerofill是有用，
如tinyint(4)，如果实际值是2，如果列指定了zerofill，查询结果就是0002，左边用0来填充。


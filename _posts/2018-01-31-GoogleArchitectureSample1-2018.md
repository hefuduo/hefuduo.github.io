---
layout:     post
title:      "Google官方Architecture Sample系列1"
subtitle:   "TODO-MVP-Room的使用"
date:       2018-01-31 11:30:00
author:     "LeoHe"
header-img: "img/post-bg-2015.jpg"
tags:
    - Android
    - Architecture	
---



# 使用Room持久化存储库保存数据

Room提供了在SQLite之上的抽象层来允许高效的数据库存取同时拥有SQLite的全部强大的功能.

总之一句话,使用Room没有错哦!

在Room中有三个重要的组件:

- **Database:** 包含数据库的持有并且作为app的持久化的主要的存取点,关联数据.

  带有`@Database`的注解的类必须具有以下条件

  - 继承`RoomDatabase`的抽象类
  - 包括实体的列表的关联数据的库的注解类
  - 包含一个抽象方法,0个参数并且返回带有`@Dao`注解的类

- **Entity:**代表数据库的一个表

- **DAO:**包含数据库存取的方法

![Room](https://developer.android.com/images/training/data-storage/room_architecture.png)

## 使用Room entities定义数据

默认的,Room为每个域创建一个列,如果你想让Entity有一个不希望它持久化的域,可有使用`@Ignore`注解标记你必须在`Database`类中引用entity的class通过entities数组.

```java
@Entity
class User{
  @PrimaryKey
  public int id;
  
  public String firstName;   //如果使用private修饰符,则需要提供gettersetter
  public String lastName;
  
  @Ignore
  Bitmap picture; 	//忽略
}
```

> 注意:
>
> 一个Entity可以有构造器,也可以使用默认的构造器,构造器可以初始化所有的变量,也可以初始化部分变量.

## 使用一个primarykey(主键)

每个实体必须顶一个至少一个域作为一个主键,即使只有一个域.如果希望Room使用自增主键,则可以使用`@PrimaryKey`注解的`autoGenerate`属性.

```java
@Entity(primaryKeys = {"firstName", "lastName"})
class User{
  public String firstName;  
  public String lastName;
  
  @Ignore
  Bitmap picture; 	//忽略
}
```

默认的,Room使用类名作为数据库的表名.如果希望使用不同的名称可以使用`@tableName`注解重新赋值

```java
@Entity(tableName = "users")
class User{
  
}
```

> 注意: 表名称在Database中是大小写不敏感的

可以使用`@ColumnInfo`注解重命名列名,默认的是域的名称

```java
@Entity(primaryKeys = {"firstName", "lastName"})
class User{
  @ColumnInfo(name = "first_name")
  public String firstName;  
  public String lastName;
  
  @Ignore
  Bitmap picture; 	//忽略
}
```

## 注解索引和单值性

根据你如何存取数据,你可能希望在数据库汇总索引特定的域来提高请求的速度.给实体添加素银,可以使用`indices`来设置

```java
@Entity(indices = {@Index("name"),
        @Index(value = {"last_name", "address"})})
class User {
    @PrimaryKey
    public int id;

    public String firstName;
    public String address;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}
```

有时,需要一个特定的域,组在数据库中是唯一的.使用unique属性注解

```java
@Entity(indices = {@Index(value = {"first_name", "last_name"},
        unique = true)})
class User {
    @PrimaryKey
    public int id;

    @ColumnInfo(name = "first_name")
    public String firstName;

    @ColumnInfo(name = "last_name")
    public String lastName;

    @Ignore
    Bitmap picture;
}

```



### 定义对象之间的关系

因为SQLite是一个关系型数据库,你可以指定对象之间的关系,及时这样大多数的ORM的库允许实体类直接相互引用,Room严格限制这样.

虽然你不能够直接通过引用简历关系,Room仍然允许你定义Foreign Key的实体之间的约束.

例如:

```java
@Entity(foreignKeys = @ForeignKey(entity = User.class,
                                 parentColumns = "id",
                                 cildColumns = "user_id"))
class Book{
  @PrimaryKey
  public int bookId;
  public String title;
  
  @ColumnInfo(name = "user_id")
  public int userId;
}
//Foreign keys are very powerful, as they allow you to specify what occurs when the referenced entity is updated. For instance, you can tell SQLite to delete all books for a user if the corresponding instance of User is deleted by including onDelete = CASCADE in the @ForeignKey annotation.
```

> **Note:** SQLite handles [`@Insert(onConflict = REPLACE)`](https://developer.android.com/reference/android/arch/persistence/room/OnConflictStrategy.html#REPLACE) as a set of `REMOVE` and `REPLACE` operations instead of a single `UPDATE` operation. This method of replacing conflicting values could affect your foreign key constraints. For more details, see the [SQLite documentation](https://sqlite.org/lang_conflict.html) for the`ON_CONFLICT` clause.

### 创建嵌套的对象

Sometimes, you'd like to express an entity or plain old Java object (POJO) as a cohesive whole in your database logic, even if the object contains several fields. In these situations, you can use the [`@Embedded`](https://developer.android.com/reference/android/arch/persistence/room/Embedded.html) annotation to represent an object that you'd like to decompose into its subfields within a table. You can then query the embedded fields just as you would for other individual columns.

For instance, our `User` class can include a field of type `Address`, which represents a composition of fields named `street`, `city`, `state`, and`postCode`. To store the composed columns separately in the table, include an `Address` field in the `User` class that is annotated with [`@Embedded`](https://developer.android.com/reference/android/arch/persistence/room/Embedded.html), as shown in the following code snippet:

```java
class Address {
    public String street;
    public String state;
    public String city;

    @ColumnInfo(name = "post_code")
    public int postCode;
}

@Entity
class User {
    @PrimaryKey
    public int id;

    public String firstName;

    @Embedded
    public Address address;
}
```

使用`@Embedded`注解创建嵌套的对象

## 使用Room DAOs存取数据

### 定义方法

#### Insert

```java
@Dao
public interface MyDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    public void insertUsers(User... users);

    @Insert
    public void insertBothUsers(User user1, User user2);

    @Insert
    public void insertUsersAndFriends(User user, List<User> friends);
}
```

If the [`@Insert`](https://developer.android.com/reference/android/arch/persistence/room/Insert.html) method receives only 1 parameter, it can return a `long`, which is the new `rowId` for the inserted item. If the parameter is an array or a collection, it should return `long[]` or `List<Long>` instead.

For more details, see the reference documentation for the [`@Insert`](https://developer.android.com/reference/android/arch/persistence/room/Insert.html) annotation, as well as the [SQLite documentation for rowid tables](https://www.sqlite.org/rowidtable.html)

可以使用带有返回值的方法,返回值为rowId

#### Update

```java
@Dao
public interface MyDao {
    @Update
    public void updateUsers(User... users);
}
//同样可以使用返回int类型的method
```

#### Delete

```java
@Dao
public interface MyDao {
    @Delete
    public void deleteUsers(User... users);
}
```



### 请求更多信息

[`@Query`](https://developer.android.com/reference/android/arch/persistence/room/Query.html) is the main annotation used in DAO classes. It allows you to perform read/write operations on a database. Each [`@Query`](https://developer.android.com/reference/android/arch/persistence/room/Query.html) method is verified at compile time, so if there is a problem with the query, a compilation error occurs instead of a runtime failure.

Room also verifies the return value of the query such that if the name of the field in the returned object doesn't match the corresponding column names in the query response, Room alerts you in one of the following two ways:

- It gives a warning if only some field names match.
- It gives an error if no field names match.

#### 简单的请求

```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user")
    public User[] loadAllUsers();
}
```

#### 向Query传递参数

```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age > :minAge")
    public User[] loadAllUsersOlderThan(int minAge);
}
```

When this query is processed at compile time, Room matches the `:minAge` bind parameter with the `minAge` method parameter. Room performs the match using the parameter names. If there is a mismatch, an error occurs as your app compiles.

多参数:

```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age BETWEEN :minAge AND :maxAge")
    public User[] loadAllUsersBetweenAges(int minAge, int maxAge);

    @Query("SELECT * FROM user WHERE first_name LIKE :search "
           + "OR last_name LIKE :search")
    public List<User> findUserWithName(String search);
}

```

### 返回子列(subset of columns)

Most of the time, you need to get only a few fields of an entity. For example, your UI might display just a user's first name and last name, rather than every detail about the user. By fetching only the columns that appear in your app's UI, you save valuable resources, and your query completes more quickly.

Room allows you to return any Java-based object from your queries as long as the set of result columns can be mapped into the returned object. For example, you can create the following plain old Java-based object (POJO) to fetch the user's first name and last name:



```java
//创建一个old java pojo
public class NameTuple {
    @ColumnInfo(name="first_name")
    public String firstName;

    @ColumnInfo(name="last_name")
    public String lastName;
}
```

用POJO

```java
@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user")
    public List<NameTuple> loadFullName();
}
```

Room understands that the query returns values for the `first_name` and `last_name` columns and that these values can be mapped into the fields of the `NameTuple` class. Therefore, Room can generate the proper code. If the query returns too many columns, or a column that doesn't exist in the `NameTuple` class, Room displays a warning.

### 传递参数集合

```java
@Dao
public interface MyDao {
    @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public List<NameTuple> loadUsersFromRegions(List<String> regions);
}

```



### Observable Queries

When performing queries, you'll often want your app's UI to update automatically when the data changes. To achieve this, use a return value of type [`LiveData`](https://developer.android.com/reference/android/arch/lifecycle/LiveData.html) in your query method description. Room generates all necessary code to update the [`LiveData`](https://developer.android.com/reference/android/arch/lifecycle/LiveData.html) when the database is updated.

```java
@Dao
public interface MyDao{
   @Query("SELECT first_name, last_name FROM user WHERE region IN (:regions)")
    public LiveData<List<User>> loadUsersFromRegionsSync(List<String> regions);
}
```

> **Note:** As of version 1.0, Room uses the list of tables accessed in the query to decide whether to update instances of [`LiveData`](https://developer.android.com/reference/android/arch/lifecycle/LiveData.html).

### 使用RxJava响应式编程

Room can also return RxJava2 [`Publisher`](http://www.reactive-streams.org/reactive-streams-1.0.1-javadoc/org/reactivestreams/Publisher.html) and [`Flowable`](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Flowable.html) objects from the queries you define. To use this functionality, add the **android.arch.persistence.room:rxjava2** artifact from the Room group into your build Gradle dependencies. You can then return objects of types defined in RxJava2, as shown in the following code snippet:

```java
@Dao
public interface MyDao {
    @Query("SELECT * from user where id = :id LIMIT 1")
    public Flowable<User> loadUserById(int id);
}
```

### Direct cursor access

直接游标存取

If your app's logic requires direct access to the return rows, you can return a `Cursor` object from your queries, as shown in the following code snippet:

```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM user WHERE age > :minAge LIMIT 5")
    public Cursor loadRawUsersOlderThan(int minAge);
}
```

> **Caution:** It's highly discouraged to work with the Cursor API because it doesn't guarantee whether the rows exist or what values the rows contain. Use this functionality only if you already have code that expects a cursor and that you can't refactor easily.
>
> 其实并不建议你这样做,可能会导致不可预测的错误

### Quering multiple tables

Some of your queries might require access to multiple tables to calculate the result. Room allows you to write any query, so you can also join tables. Furthermore, if the response is an observable data type, such as[`Flowable`](http://reactivex.io/RxJava/2.x/javadoc/io/reactivex/Flowable.html) or [`LiveData`](https://developer.android.com/reference/android/arch/lifecycle/LiveData.html), Room watches all tables referenced in the query for invalidation.

The following code snippet shows how to perform a table join to consolidate information between a table containing users who are borrowing books and a table containing data about books currently on loan:

```java
@Dao
public interface MyDao {
    @Query("SELECT * FROM book "
           + "INNER JOIN loan ON loan.book_id = book.id "
           + "INNER JOIN user ON user.id = loan.user_id "
           + "WHERE user.name LIKE :userName")
   public List<Book> findBooksBorrowedByNameSync(String userName);
}
```

You can also return POJOs from these queries. For example, you can write a query that loads a user and their pet's name as follows:

```java
@Dao
public interface MyDao {
   @Query("SELECT user.name AS userName, pet.name AS petName "
          + "FROM user, pet "
          + "WHERE user.id = pet.user_id")
   public LiveData<List<UserPet>> loadUserAndPetNames();

   // You can also define this class in a separate file, as long as you add the
   // "public" access modifier.
   static class UserPet {
       public String userName;
       public String petName;
   }
}
```

## 迁移Room数据库

As you add and change features in your app, you need to modify your entity classes to reflect these changes. When a user updates to the latest version of your app, you don't want them to lose all of their existing data, especially if you can't recover the data from a remote server.

The [Room persistence library](https://developer.android.google.cn/training/data-storage/room/index.html?hl=zh-cn) allows you to write [`Migration`](https://developer.android.google.cn/reference/android/arch/persistence/room/migration/Migration.html?hl=zh-cn) classes to preserve user data in this manner. Each [`Migration`](https://developer.android.google.cn/reference/android/arch/persistence/room/migration/Migration.html?hl=zh-cn) class specifies a `startVersion` and `endVersion`. At runtime, Room runs each[`Migration`](https://developer.android.google.cn/reference/android/arch/persistence/room/migration/Migration.html?hl=zh-cn) class's [`migrate()`](https://developer.android.google.cn/reference/android/arch/persistence/room/migration/Migration.html?hl=zh-cn#migrate(android.arch.persistence.db.SupportSQLiteDatabase)) method, using the correct order to migrate the database to a later version.

> **Caution:** If you don't provide the necessary migrations, Room rebuilds the database instead, which means you'll lose all of your data in the database.

使用Migration类保存之前的与用户数据,每个Migration类指定了一个`startVersion`和`endVersion`.Room会在运行时执行每个migrate方法.使用正确的方式迁移数据库到最新版本.

```java
Room.databaseBuilder(getApplicationContext(), MyDb.class, "database-name")
        .addMigrations(MIGRATION_1_2, MIGRATION_2_3).build(); //初始化的时候就迁移数据库

static final Migration MIGRATION_1_2 = new Migration(1, 2) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("CREATE TABLE `Fruit` (`id` INTEGER, "
                + "`name` TEXT, PRIMARY KEY(`id`))");
    }
};

static final Migration MIGRATION_2_3 = new Migration(2, 3) {
    @Override
    public void migrate(SupportSQLiteDatabase database) {
        database.execSQL("ALTER TABLE Book "
                + " ADD COLUMN pub_year INTEGER");
    }
};
```

> **Caution:** To keep your migration logic functioning as expected, use full queries instead of referencing constants that represent the queries.

After the migration process finishes, Room validates the schema to ensure that the migration occurred correctly. If Room finds a problem, it throws an exception that contains the mismatched information.

如果迁移失败,Room会抛出异常信息.



### 导出概要

配置gradle导出JSON格式的概要

```groovy
android {
    ...
    defaultConfig {
        ...
        javaCompileOptions {
            annotationProcessorOptions {
                arguments = ["room.schemaLocation":
                             "$projectDir/schemas".toString()]
            }
        }
    }
}
```

You should store the exported JSON files—which represent your database's schema history—in your version control system, as it allows Room to create older versions of your database for testing purposes.

保存JSON格式的升级信息文件,以便数据回滚,并且能够执行测试.

To test these migrations, add the **android.arch.persistence.room:testing** Maven artifact from Room into your test dependencies, and add the schema location as an asset folder, as shown in the following code snippet:

```groovy
android {
    ...
    sourceSets {
        androidTest.assets.srcDirs += files("$projectDir/schemas".toString())
    }
}
```

The testing package provides a [`MigrationTestHelper`](https://developer.android.google.cn/reference/android/arch/persistence/room/testing/MigrationTestHelper.html?hl=zh-cn) class, which can read these schema files. It also implements the JUnit4 [`TestRule`](http://junit.org/junit4/javadoc/4.12/org/junit/rules/TestRule.html) interface, so it can manage created databases.

A sample migration test appears in the following code snippet:

```java
@RunWith(AndroidJUnit4.class)
public class MigrationTest {
    private static final String TEST_DB = "migration-test";

    @Rule
    public MigrationTestHelper helper;

    public MigrationTest() {
        helper = new MigrationTestHelper(InstrumentationRegistry.getInstrumentation(),
                MigrationDb.class.getCanonicalName(),
                new FrameworkSQLiteOpenHelperFactory());
    }

    @Test
    public void migrate1To2() throws IOException {
        SupportSQLiteDatabase db = helper.createDatabase(TEST_DB, 1);

        // db has schema version 1. insert some data using SQL queries.
        // You cannot use DAO classes because they expect the latest schema.
        db.execSQL(...);

        // Prepare for the next version.
        db.close();

        // Re-open the database with version 2 and provide
        // MIGRATION_1_2 as the migration process.
        db = helper.runMigrationsAndValidate(TEST_DB, 2, true, MIGRATION_1_2);

        // MigrationTestHelper automatically verifies the schema changes,
        // but you need to validate that the data was migrated properly.
    }
}
//执行测试用例
```



### 测试

The recommended approach for testing your database implementation is writing a JUnit test that runs on an Android device. Because these tests don't require creating an activity, they should be faster to execute than your UI tests.

When setting up your tests, you should create an in-memory version of your database to make your tests more hermetic, as shown in the following example:

```java
@RunWith(AndroidJUnit4.class)
public class SimpleEntityReadWriteTest {
    private UserDao mUserDao;
    private TestDatabase mDb;

    @Before
    public void createDb() {
        Context context = InstrumentationRegistry.getTargetContext();
        mDb = Room.inMemoryDatabaseBuilder(context, TestDatabase.class).build();
        mUserDao = mDb.getUserDao();
    }

    @After
    public void closeDb() throws IOException {
        mDb.close();
    }

    @Test
    public void writeUserAndReadInList() throws Exception {
        User user = TestUtil.createUser(3);
        user.setName("george");
        mUserDao.insert(user);
        List<User> byName = mUserDao.findUsersByName("george");
        assertThat(byName.get(0), equalTo(user));
    }
}

```

### TypeConvertes 使用自定义类型转换器

Room provides functionality for converting between primitive and boxed types but doesn't allow for object references between entities. This document explains how to use type converters and why Room doesn't support object references.

例子,使用自定义Converters使得`Date`和`Long`类型可以相互转换

```java
public class Converters {
    @TypeConverter
    public static Date fromTimestamp(Long value) {
        return value == null ? null : new Date(value);
    }

    @TypeConverter
    public static Long dateToTimestamp(Date date) {
        return date == null ? null : date.getTime();
    }
}
```

The preceding example defines 2 functions, one that converts a `Date` object to a `Long` object and another that performs the inverse conversion, from `Long` to `Date`. Since Room already knows how to persist `Long` objects, it can use this converter to persist values of type `Date`.

Next, you add the [`@TypeConverters`](https://developer.android.google.cn/reference/android/arch/persistence/room/TypeConverters.html?hl=zh-cn) annotation to the `AppDatabase` class so that Room can use the converter that you've defined for each [entity](https://developer.android.google.cn/training/data-storage/room/defining-data.html?hl=zh-cn) and [DAO](https://developer.android.google.cn/training/data-storage/room/accessing-data.html?hl=zh-cn) in that `AppDatabase`:

Room是知道如何持久化`Long`类型的,因此,只要自定义了其他类型向Long类型的转换,Room就可以存取这种类型.

`AppDataBase.java`

```java
@Database(entities = {User.class}, version = 1)
@TypeConverters({Converters.class})
public abstract class AppDatabase extends RoomDatabase {
    public abstract UserDao userDao();
}
```

Using these converters, you can then use your custom types in other queries, just as you would use primitive types, as shown in the following code snippet:

使用这些转换器,你可以使用自定义类型在queries中,就像你使用原生类型一样.

`User.java`

```java
@Entity
public class User {
    ...
    private Date birthday;
}
```

`UserDao.java`

使用Date类型进行查找

```java
@Dao
public interface UserDao {
    ...
    @Query("SELECT * FROM user WHERE birthday BETWEEN :from AND :to")
    List<User> findUsersBornBetweenDates(Date from, Date to);
}
```


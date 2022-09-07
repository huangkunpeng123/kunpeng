该文档，旨在沉淀平日业务开发过程中的小技巧。
愿景目标：在逻辑不变的前提下，让代码变得更简洁，追求代码执行效率与简洁的平衡之道。
精简代码，但不能以过大的性能损失为代价。
## 语法类
### 1.1 三元运算符
**普通：**
```java
String title; 
if (isMember(phone)) {     
    title = "会员"; 
} else {     
    title = "游客"; 
}
```
**精简：**
```java
String title = isMember(phone) ? "会员" : "游客";
```
### 1.2 字符串比较
```java
if (str.equals("字符串比较")) {
   @todo
}
```
```java
if ("字符串比较".equals(str)) {
    @todo
}
```
```java
if (StringUtils.equals(str, "字符串比较")) {
    @todo
}
```
### 1.3  integer之间的比较
![](https://img-blog.csdnimg.cn/20210422164544846.png#crop=0&crop=0&crop=1&crop=1&id=Jt4di&originHeight=294&originWidth=1430&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
```java
Integer a = 100;
Integer b = 100;
if (a==b) {
  @todo
}
```
```java
Integer a = 100;
Integer b = 100;
if (a.equals(b)) {
@todo
}
```
### 1.4  定点数Decimal 
java在处理浮点数、double等数据类型，需要使用定点数Decimal
我们在使用 **BigDecimal** 时，为了防止精度丢失，推荐使用它的**BigDecimal(String val)**构造方法或者 **BigDecimal.valueOf(double val) **静态方法来创建对象。
《阿里巴巴 Java 开发手册》对这部分内容也有提到，如下图所示。
![](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/javaguide/image-20211213102222601.png#crop=0&crop=0&crop=1&crop=1&id=sJ9dj&originHeight=246&originWidth=923&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
**compareTo() 方法可以比较两个 BigDecimal 的值，如果相等就返回 0，如果第 1 个数比第 2 个数大则返回 1，反之返回-1。**
```java
BigDecimal a = new BigDecimal("1");
BigDecimal b = new BigDecimal("1.0");
System.out.println(a.compareTo(b));//0
```
### 1.5  DTO2POJO & POJO2VO
● set方法
● beansutil
● MapStruct
```java
@Data
@TableName(value = "uc_menu")
public class Menu {
    @TableId(type = IdType.AUTO)
    private Long id;
    private Long parentId;
    private String name;
    @TableField(exist = false)
    private Integer selectable;
}

@Data
public class MenuDTO {
    private Long id;
    private Long parentId;
    private String name;
    private List<MenuVo> children;
    private Integer selectable;
}

@Data
public class MenuVo {
    private Long id;
    private Long parentId;
    private String name;
    private List<MenuVo> children;
    private Integer selectable;
}
```
```java
Menu menu = new Menu();
menu.setId(MenuDTO.getId);
menu.setParentId(MenuDTO.getParentId);
menu.setChildren(MenuDTO.getChildren);
menu.setSelectable(MenuDTO.getSelectable);
return menu;
--------------------------------
POJO2VO 同上
```
```java
Menu menu = new Menu();
BeanUtils.copyProperties(menuDTO, menu); // 反射，对性能损耗比较 
--------------------------------
POJO2VO 同上
```
[https://mapstruct.org/](https://mapstruct.org/)
MapStruct
```java
@Mapper 
public interface MenMapper {
 
    CarMapper INSTANCE = Mappers.getMapper( MenMapper.class ); 3
 
    @Mapping(source = "numberOfSeats", target = "seatCount")
    MenuDto menuToMenuDto(Menu menu); 2
}
```
## 1.6  集合遍历
### **6.1.数组和列表的遍历**
从 Java 5 起，提供了 for-each 循环，简化了数组和集合的循环遍历。 for-each 循环允许你无需保持传统 for 循环中的索引就可以遍历数组，或在使用迭代器时无需在 while 循环中调用 hasNext 方法和 next 方法就可以遍历集合。
**普通：**
```java
double[] values = ...; 	
for(int i = 0; i < values.length; i++) {         
    double value = values[i];            
    // TODO: 处理value     
} 
List<Double> valueList = ...; 
Iterator<Double> iterator = valueList.iterator(); 
while (iterator.hasNext()) {     
    Double value = iterator.next();         
    // TODO: 处理value 
}
```
**精简：**
```java
double[] values = ...; 
for(double value : values) {         
    // TODO: 处理value 
} 
List<Double> valueList = ...; 
for(Double value : valueList) {     
    // TODO: 处理value 
}
```
#### 推荐：
```java
double[] values = ...; 
Arrays.stream(values).foreach(@todo)
List<Double> valueList = ...; 
valueList.stream().foreach(@todo)
```
### **6.2 避免空值判断**
**普通：**
```java
if (userList != null && !userList.isEmpty()) {     
    // TODO: 处理代码 
}
```
**精简：**
```java
if (CollectionUtils.isNotEmpty(userList)) {    
    // TODO: 处理代码
}
```

### **1.7  利用 return 关键字**
利用 return 关键字，可以提前函数返回，避免定义中间变量。
**普通：**
```java
public static boolean hasSuper(@NonNull List<UserDO> userList) {     
    boolean hasSuper = false;    
for (UserDO user : userList) {         
    if (Boolean.TRUE.equals(user.getIsSuper())) {             
        hasSuper = true;             
        break;         
    }     
}     
    return hasSuper; 
}
```

**精简：**
```java
public static boolean hasSuper(@NonNull List<UserDO> userList) {     
    for (UserDO user : userList) {         
        if (Boolean.TRUE.equals(user.getIsSuper())) {             
            return true;         
        }     
    }     
    return false; 
}
```
### **1.8  利用 lambda 表达式和方法引用**
Java 8 发布以后，lambda 表达式大量替代匿名内部类的使用，在简化了代码的同时，更突出了原有匿名内部类中真正有用的那部分代码。
**普通：**
```java
new Thread(new Runnable() {     
    public void run() { 
        @todo
        // 线程处理代码     
    } 
}).start();
```
**精简：**
```java
new Thread(() -> {     
    // 线程处理代码
    @todo
}).start();
```
#### 方法引用：

### **1.9  简化赋值语句**
**普通：**
```java
public static final List<String> ANIMAL_LIST; 
static {     
    List<String> animalList = new ArrayList<>();     
    animalList.add("dog");     
    animalList.add("cat");     
    animalList.add("tiger");     
    ANIMAL_LIST = Collections.unmodifiableList(animalList); 
}
```

**精简：**
```java
// JDK流派 
public static final List<String> ANIMAL_LIST = Arrays.asList("dog", "cat", "tiger"); 
```
注意：Arrays.asList 返回的 List 并不是 ArrayList ，不支持 add 等变更操作。
## 注解类
### **2.1 利用 Lombok 注解**
Lombok 提供了一组有用的注解，可以用来消除Java类中的大量样板代码。
**普通：**
```java
public class UserVO {     
    private Long id;     
    private String name;     
    public Long getId() {         
        return this.id;     
    }     
    public void setId(Long id) {         
        this.id = id;     
    }     
    public String getName() {         
        return this.name;     
    }     
    public void setName(String name) {         
        this.name = name;     
    }     
    ... 
    @todo
}
```
**精简：**
```java
@Getter 
@Setter 
@ToString 
public class UserVO {     
    private Long id;     
    private String name;     
    ... 
}

```
### **2.2  利用 Validation 注解**
**普通：**
```java
@Getter
@Setter
@ToString 
public class UserCreateVO {    
 
    private String name;     

    private Long companyId;     
    ...
} 

@Service 
@Validated 
public class UserService {     
    public Long createUser(UserCreateVO create) {         
        // TODO: 创建用户 
        String name = create.getName();
        if (name !=null && string.empty(name)){
            throw new XXException("用户名称不能为空");
        }
        long companyId = create.getCompanyId();
        if (companyId !=null && string.empty(companyId)){
            throw new XXException("公司标识不能为空");
        }
        return null;     
    } 
}
```
**精简：**
```java
@Getter 
@Setter 
@ToString 
public class UserCreateVO {         
    @NotBlank(message = "用户名称不能为空")     
    private String name;     
    @NotNull(message = "公司标识不能为空")     
    private Long companyId;         
    ...     
} 
@Service 
@Validated 
public class UserService {     
    public Long createUser(@Valid UserCreateVO create) {         
            // TODO: 创建用户                  
            return null;              
    } 
}

```
### **2.3  利用 @NonNull 注解**
Spring 的 @NonNull 注解，用于标注参数或返回值非空，适用于项目内部团队协作。只要实现方和调用方遵循规范，可以避免不必要的空值判断，这充分体现了阿里的“新六脉神剑”提倡的“因为信任，所以简单”。
**普通：**
```java
public List<UserVO> queryCompanyUser(Long companyId) {     
    // 检查公司标识     
    if (companyId == null) {         
    	return null;     
	}     
// 查询返回用户     
List<UserDO> userList = userDAO.queryByCompanyId(companyId);     
return userList.stream().map(this::transUser).collect(Collectors.toList()); 
} 
Long companyId = 1L;
List<UserVO> userList = queryCompanyUser(companyId); 
if (CollectionUtils.isNotEmpty(userList)) {     
    for (UserVO user : userList) {         
        // TODO: 处理公司用户     
    } 
}
```
**精简：**
```java
public @NonNull List<UserVO> queryCompanyUser(@NonNull Long companyId) {     
    List<UserDO> userList = userDAO.queryByCompanyId(companyId);     
    return userList.stream().map(this::transUser).collect(Collectors.toList()); 
} 
Long companyId = 1L;
List<UserVO> userList = queryCompanyUser(companyId); 
for (UserVO user : userList) {     
    
    // TODO: 处理公司用户 
}
```
### **2.4  利用注解特性**
注解有以下特性可用于精简注解声明：
1、当注解属性值跟默认值一致时，可以删除该属性赋值；
2、当注解只有value属性时，可以去掉value进行简写；
3、当注解属性组合等于另一个特定注解时，直接采用该特定注解。
**普通：**
```java
@Lazy(true); 
@Service(value = "userService") 
@RequestMapping(path = "/getUser", method = RequestMethod.GET)
```
**精简：**
```java
@Lazy 
@Service("userService") 
@GetMapping("/getUser")
```

## 插件类
### 1. 阿里巴巴编码规约

### 2. camelCase
### 3. pojo to json
### 4. mybatisX
### 5. maven-search 
### 6. save action



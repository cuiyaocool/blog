
本文梳理Lombok的注解，以及使用过程中的一些细节问题
-----
Annotation | 作用| 作用域
---|---|--
@val | 等于final | 成员变量
@NonNull | 非空校验|成员变量、方法参数
|@Cleanup|关闭资源|局部变量
|@Getter|get方法|类、成员变量
|@Setter|set方法|类、成员变量
|@ToString|toString方法|类
|@EqualsAndHashCode|自动生成equals和hashCode方法|类
|@NoArgsConstructor|无参构造函数|类
|@AllArgsConstructor|包含全部参数的构造函数|类
|@RequiredArgsConstructor|包含@Nonull修饰的属性的构造函数，还可设置静态方法|类
|@Data|@ToString、@EqualsAndHashCode、@Getter、@Setter、@RequiredArgsConstrutor的集合|类
|@Value|@Data的不可变形式，不提供set|类
|@SneakyThrows|自动抛出受检查异常，无需显示throws|方法
|@Synchronized|自动加锁，在类中创建final的lock|方法
|@Getter(lazy=true)|对final成员进行类似DCL的懒加载|成员变量，需要为final以及初始化
|@Log|日志，有多种，比如@Slf4j、@Log4j|类
|@Builder|提供builer api接口|类、构造器、方法
---
1. 在包含@Value的情况下，@RequiredArgsConstructor和@AllArgsConstructor不共存，因为前者会包含所有final的属性，@Value会将属性设置为final，与后者重复。
2. Builder是可以用在方法和构造函数的，修饰普通方法，生成的Builder类为普通内部类，修饰构造函数，生成的Builer为静态内部类，对比见下文

---
**Example**   
```java
@RequiredArgsConstructor(staticName = "of")
@EqualsAndHashCode
@NoArgsConstructor
public class TestLoB {
    @NonNull private String id;

    private Integer age;

    @Getter(lazy = true)
    private final Integer lazy = new Integer(8);

    private Float high;

    @Synchronized
    public void make(@NonNull String id) throws Exception {
        System.out.println(id);
    }

    @SneakyThrows
    private void openFile(String path) {
        @Cleanup FileInputStream fileInputStream = new FileInputStream(path);
    }

    @Builder
    public void age(Integer age) {
        this.age = age;
    }
}
```
---
**生成代码**  

```
public class TestLoB {
    private final Object $lock = new Object[0];
    @NonNull
    private String id;
    private Integer age;
    private final AtomicReference<Object> lazy = new AtomicReference();
    private Float high;

    //由于加了@NonNull注解，自动添加了非空校验，锁为final Object
    public void make(@NonNull String id) throws Exception {
        synchronized(this.$lock) {
            if (id == null) {
                throw new NullPointerException("id is marked non-null but is null");
            } else {
                System.out.println(id);
            }
        }
    }

    // @SneakyThrows自动添加了throws语句，同时局部变量fileInputStream自动调用了close()方法
    private void openFile(String path) {
        try {
            FileInputStream fileInputStream = new FileInputStream(path);
            if (Collections.singletonList(fileInputStream).get(0) != null) {
                fileInputStream.close();
            }

        } catch (Throwable var3) {
            throw var3;
        }
    }

    public void age(Integer age) {
        this.age = age;
    }
    
    // 在age(Integer age)方法上添加了Builder注解，所以生成了构造器，一个内部类
    public TestLoB.VoidBuilder builder() {
        return new TestLoB.VoidBuilder();
    }

    private TestLoB(@NonNull final String id) {
        if (id == null) {
            throw new NullPointerException("id is marked non-null but is null");
        } else {
            this.id = id;
        }
    }

    public static TestLoB of(@NonNull final String id) {
        return new TestLoB(id);
    }

    // 自动生成equals，先比较类型，后比较每个属性值
    public boolean equals(final Object o) {
        if (o == this) {
            return true;
        } else if (!(o instanceof TestLoB)) {
            return false;
        } else {
            TestLoB other = (TestLoB)o;
            if (!other.canEqual(this)) {
                return false;
            } else {
                label59: {
                    Object this$id = this.id;
                    Object other$id = other.id;
                    if (this$id == null) {
                        if (other$id == null) {
                            break label59;
                        }
                    } else if (this$id.equals(other$id)) {
                        break label59;
                    }

                    return false;
                }

                Object this$age = this.age;
                Object other$age = other.age;
                if (this$age == null) {
                    if (other$age != null) {
                        return false;
                    }
                } else if (!this$age.equals(other$age)) {
                    return false;
                }

                Object this$lazy = this.getLazy();
                Object other$lazy = other.getLazy();
                if (this$lazy == null) {
                    if (other$lazy != null) {
                        return false;
                    }
                } else if (!this$lazy.equals(other$lazy)) {
                    return false;
                }

                Object this$high = this.high;
                Object other$high = other.high;
                if (this$high == null) {
                    if (other$high != null) {
                        return false;
                    }
                } else if (!this$high.equals(other$high)) {
                    return false;
                }

                return true;
            }
        }
    }

    protected boolean canEqual(final Object other) {
        return other instanceof TestLoB;
    }

    public int hashCode() {
        int PRIME = true;
        int result = 1;
        Object $id = this.id;
        int result = result * 59 + ($id == null ? 43 : $id.hashCode());
        Object $age = this.age;
        result = result * 59 + ($age == null ? 43 : $age.hashCode());
        Object $lazy = this.getLazy();
        result = result * 59 + ($lazy == null ? 43 : $lazy.hashCode());
        Object $high = this.high;
        result = result * 59 + ($high == null ? 43 : $high.hashCode());
        return result;
    }

    public TestLoB() {
    }

    //lazy 属性添加了懒加载get
    public Integer getLazy() {
        Object value = this.lazy.get();
        if (value == null) {
            synchronized(this.lazy) {
                value = this.lazy.get();
                if (value == null) {
                    Integer actualValue = new Integer(8);
                    value = actualValue == null ? this.lazy : actualValue;
                    this.lazy.set(value);
                }
            }
        }

        return (Integer)((Integer)(value == this.lazy ? null : value));
    }

    public class VoidBuilder {
        private Integer age;

        VoidBuilder() {
        }

        public TestLoB.VoidBuilder age(final Integer age) {
            this.age = age;
            return this;
        }

        public void build() {
            TestLoB.this.age(this.age);
        }

        public String toString() {
            return "TestLoB.VoidBuilder(age=" + this.age + ")";
        }
    }
}

```
**对比**

```
    @Getter(lazy = true)
    private final Integer lazy = new Integer(8);
    ===============================================
    public Integer getLazy() {
        Object value = this.lazy.get();
        if (value == null) {
            synchronized(this.lazy) {
                value = this.lazy.get();
                if (value == null) {
                    Integer actualValue = new Integer(8);
                    value = actualValue == null ? this.lazy : actualValue;
                    this.lazy.set(value);
                }
            }
        }

        return (Integer)((Integer)(value == this.lazy ? null : value));
    }
```

```
    @SneakyThrows
    private void openFile(String path) {
        @Cleanup FileInputStream fileInputStream = new FileInputStream(path);
    }
    ==========================================
    private void openFile(String path) {
        try {
            FileInputStream fileInputStream = new FileInputStream(path);
            if (Collections.singletonList(fileInputStream).get(0) != null) {
                fileInputStream.close();
            }

        } catch (Throwable var3) {
            throw var3;
        }
    }
```

```
    @Synchronized
    public void make(@NonNull String id) throws Exception {
        System.out.println(id);
    }
    ===========================================
    private final Object $lock = new Object[0];
    public void make(@NonNull String id) throws Exception {
        synchronized(this.$lock) {
            if (id == null) {
                throw new NullPointerException("id is marked non-null but is null");
            } else {
                System.out.println(id);
            }
        }
    }
```

```
    @Builder
    public void age(Integer age) {
        this.age = age;
    }
    ===============================================
    public void age(Integer age) {
        this.age = age;
    }
    
    public TestLoB.VoidBuilder builder() {
        return new TestLoB.VoidBuilder();
    }
    public class VoidBuilder {
        private Integer age;

        VoidBuilder() {
        }

        public TestLoB.VoidBuilder age(final Integer age) {
            this.age = age;
            return this;
        }

        public void build() {
            TestLoB.this.age(this.age);
        }

        public String toString() {
            return "TestLoB.VoidBuilder(age=" + this.age + ")";
        }
    }
```

```
@RequiredArgsConstructor(staticName = "of")
=====================================================
private TestLoB(@NonNull final String id) {
        if (id == null) {
            throw new NullPointerException("id is marked non-null but is null");
        } else {
            this.id = id;
        }
    }

    public static TestLoB of(@NonNull final String id) {
        return new TestLoB(id);
    }
```
Builder修饰普通方法和构造函数的区别  
1. 修改普通函数

```
    public void age(Integer age) {
        this.age = age;
    }
    
    public TestLoB.VoidBuilder builder() {
        return new TestLoB.VoidBuilder();
    }
    public class VoidBuilder {
        private Integer age;

        VoidBuilder() {
        }

        public TestLoB.VoidBuilder age(final Integer age) {
            this.age = age;
            return this;
        }

        public void build() {
            TestLoB.this.age(this.age);
        }

        public String toString() {
            return "TestLoB.VoidBuilder(age=" + this.age + ")";
        }
    }
```

2. 修饰构造函数

```
    // TestLoBBuilder的属性为构造函数中的参数
    public static TestLoB.TestLoBBuilder builder() {
        return new TestLoB.TestLoBBuilder();
    }

    public static class TestLoBBuilder {
        private int age;
        private int high;

        TestLoBBuilder() {
        }

        public TestLoB.TestLoBBuilder age(final int age) {
            this.age = age;
            return this;
        }

        public TestLoB.TestLoBBuilder high(final int high) {
            this.high = high;
            return this;
        }

        public TestLoB build() {
            return new TestLoB(this.age, this.high);
        }

        public String toString() {
            return "TestLoB.TestLoBBuilder(age=" + this.age + ", high=" + this.high + ")";
        }
    }
```







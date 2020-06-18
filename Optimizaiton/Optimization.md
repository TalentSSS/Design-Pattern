# Optimization

记录一些不错的代码写法、风格

## 对象转化工具类

将数据库中的DO转成VO的时，经常会在代码中看到很多setter方法去设置新建出来VO的属性，参考资料中的博客写了一个泛型的converter去更简洁的实现这个功能。

```java
public class BaseConverter<DO, VO> {
	private static final Logger logger = LoggerFactory.getLogger(BaseConverter.class);

    // 单个对象转换
    public VO convert(DO from, Class<VO> clazz) {
        if (from == null) {
            return null;
        }
        VO to = null;
        try {
            to = clazz.newInstance();
        } catch (Exception e) {
            logger.error("初始化{}对象失败。", clazz, e);
        }
        convert(from, to);
        return to;
    }
    
    // 批量对象转换
    public List<VO> convert(List<DO> fromList, Class<VO> clazz) {
        if (CollectionUtils.isEmpty(fromList)) {
            return null;
        }
        List<VO> toList = new ArrayList<VO>();
        for (DO from : fromList) {
            toList.add(convert(from, clazz));
        }
        return toList;
    }

    // 属性拷贝方法，有特殊需求时子类覆写此方法
    protected void convert(DO from, VO to) {
        BeanUtils.copyProperties(from, to);
    }
}
```

`void convert(DO from, VO to)`这个方法用来处理两个类属性不一致的特殊情况，这里的BeanUtils可以用Spring框架里的BeanUtils

对于一个PersonDO和一个PersonVO

```java
public class PersonDO {
    private String name;
    private Date birthDate;
    public PersonDO() {
    }
    public PersonDO(String name, Date birthDate) {
        this.name = name;
        this.birthDate = birthDate;
    }
}
```

```java
public class PersonVO {
    private String name;
    private String birthDate;

    @Override
    public String toString() {
        return "PersonVO{" + "name='" + name + '\'' + ", birthDate='" + birthDate + '\'' + '}';
    }
}
```

可以写一个转换器PersonConverter

```java
public class PersonConverter extends BaseConverter<PersonDO, PersonVO> {
    @Override
    protected void convert(PersonDO from, PersonVO to) {
        SimpleDateFormat simpleDateFormat = new SimpleDateFormat("yyyy-MM-dd");
        to.setBirthDate(simpleDateFormat.format(from.getBirthDate()));
        super.convert(from, to);
    }
}
```

转化器重写了`void convert(DO from, VO to)`方法，格式化了出生日期

### 参考资料

[【泛型】一个简易的对象间转换的工具类(DO转VO)](https://blog.csdn.net/silk_bar/article/details/54957843)














---
title: '<<踩坑记>>系列--Guava Lists.transform'
date: 2016-01-30 14:06:14
tags: Guava
categories: 踩坑记

---

&#160; &#160; &#160; &#160;在工作中,有些场景会需要将一种Object类型的List数据转换成另一种Object类型的数据.比如做爬虫时,希望将爬到的数据换一种领域模型保存到自己的数据库里面去,这个过程中会涉及到一个循环转换的步骤.前两天我就在这个步骤上踩了一个坑,所以趁周末有空特地撰文以记之,让这个坑踩得值一点.

----------

## 问题抽象##

&#160; &#160; &#160; &#160; 现有两个POJO,PersonDO和PersonDTO.目标是将一个personDOList转换成personDTOList,并设置personDTOList中的每个person的age值为30.

- PersonDO:

```java
public class PersonDO {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

- PersonDTO:

```java
public class PersonDTO {
    private String name;
    private int age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

```

## 编码实践##
```java
import com.google.common.base.Function;
import com.google.common.collect.Lists;
import org.junit.Before;
import org.junit.Test;
import java.util.ArrayList;
import java.util.List;

public class ConverterTest {

  private  List<PersonDO> personDOs = new ArrayList<>();

  @Before
  public void mockData() {
    // 准备数据
    PersonDO one = new PersonDO();
    one.setName("one");
    one.setAge(18);
    PersonDO two = new PersonDO();
    two.setName("two");
    two.setAge(20);

    personDOs.add(one);
    personDOs.add(two);
  }

  @Test
  public void testToDTO() throws Exception {
    //将DO转换成DTO

    //普通青年这样写
    List<PersonDTO> dtoList_1 = new ArrayList<>();
    for (PersonDO personDO : personDOs) {
      PersonDTO dto = toDTO(personDO);
      dtoList_1.add(dto);
    }

    //2B青年这样写
    List<PersonDTO> dtoList_2 = new ArrayList<>();
    for (int i = 0; i < personDOs.size(); i++ ) {
      PersonDTO dto = toDTO(personDOs.get(i));
      dtoList_2.add(dto);
    }

    //文艺青年这样写
    //用guava的Lists.transform()转换成DTO
    List<PersonDTO> dtoList_3 = new ArrayList<>();
    dtoList_3 = Lists.transform(personDOs, new Function<PersonDO, PersonDTO>() {
      @Override
      public PersonDTO apply(PersonDO input) {
        return toDTO(input);
      }
    });

    //修改age属性
    for (PersonDTO dto : dtoList_1) {
      dto.setAge(30);
    }
    for (PersonDTO dto : dtoList_2) {
      dto.setAge(30);
    }
    for(PersonDTO dto : dtoList_3) {
      dto.setAge(30);
    }

    //打印结果
    System.out.println("********普通青年的结果**************");
    printTheRelult(dtoList_1);
    System.out.println("********2B青年的结果***************");
    printTheRelult(dtoList_2);
    System.out.println("********文艺青年的结果**************");
    printTheRelult(dtoList_3);
  }

  private void printTheRelult(List<PersonDTO> dtoList) {
    for (PersonDTO personDTO : dtoList) {
      System.out.println(personDTO.getName() + " is " + personDTO.getAge() + " years old.");
    }
  }

  private PersonDTO toDTO(PersonDO personDO) {
    PersonDTO ret = new PersonDTO();
    //这里简单get,set一下,当然也可以使用ObjectMapper.map()
    ret.setName(personDO.getName());
    ret.setAge(personDO.getAge());
    return ret;
  }
}

```
&#160; &#160; &#160; &#160;运行测试用例,结果却是这样的:
![此处输入图片的描述][1]
&#160; &#160; &#160; &#160;显然,对经过Lists.transform操作之后得到dtoList_3的setAge()操作并没有生效.刚发现这个问题的时候很诧异,于是debug.这一步找到病因.
```java
for(PersonDTO dto : dtoList_3) {
      dto.setAge(30);
}

```

&#160; &#160; &#160; &#160;仔细看debug信息,dtoList_3在经过Lists.transform操作之后已经不是我在初始化时指定的ArrayList类型了,而是TransformingRandomAccessList.而且,特别注意,dto的地址编号是659,而dtoList_3中的两个对象的地址编号分别是672和673.这说明,循环的过程中拿到的对象,不是dtoList_3中的对象.瞬间感觉重温了大一学C语言的时候,值传递还是引用传递的问题.
![此处输入图片的描述][2]

&#160; &#160; &#160; &#160;再跟进去就太细节了,于是我想起回头来看API文档:
```java
  /**
   * Returns a list that applies {@code function} to each element of {@code
   * fromList}. The returned list is a transformed view of {@code fromList};
   * changes to {@code fromList} will be reflected in the returned list and vice
   * versa.
   *
   * <p>Since functions are not reversible, the transform is one-way and new
   * items cannot be stored in the returned list. The {@code add},
   * {@code addAll} and {@code set} methods are unsupported in the returned
   * list.
   *
   * <p>The function is applied lazily, invoked when needed. This is necessary
   * for the returned list to be a view, but it means that the function will be
   * applied many times for bulk operations like {@link List#contains} and
   * {@link List#hashCode}. For this to perform well, {@code function} should be
   * fast. To avoid lazy evaluation when the returned list doesn't need to be a
   * view, copy the returned list into a new list of your choosing.
   *
   * <p>If {@code fromList} implements {@link RandomAccess}, so will the
   * returned list. The returned list is threadsafe if the supplied list and
   * function are.
   *
   * <p>If only a {@code Collection} or {@code Iterable} input is available, use
   * {@link Collections2#transform} or {@link Iterables#transform}.
   *
   * <p><b>Note:</b> serializing the returned list is implemented by serializing
   * {@code fromList}, its contents, and {@code function} -- <i>not</i> by
   * serializing the transformed values. This can lead to surprising behavior,
   * so serializing the returned list is <b>not recommended</b>. Instead,
   * copy the list using {@link ImmutableList#copyOf(Collection)} (for example),
   * then serialize the copy. Other methods similar to this do not implement
   * serialization at all for this reason.
   */
  public static <F, T> List<T> transform(
      List<F> fromList, Function<? super F, ? extends T> function) {
    return (fromList instanceof RandomAccess)
        ? new TransformingRandomAccessList<F, T>(fromList, function)
        : new TransformingSequentialList<F, T>(fromList, function);
  }


```


&#160; &#160; &#160; &#160;这里有一段解释了我碰到的问题:
```java
* <p>The function is applied lazily, invoked when needed. This is necessary
   * for the returned list to be a view, but it means that the function will be
   * applied many times for bulk operations like {@link List#contains} and
   * {@link List#hashCode}. For this to perform well, {@code function} should be
   * fast. To avoid lazy evaluation when the returned list doesn't need to be a
   * view, copy the returned list into a new list of your choosing.
```
&#160; &#160; &#160; &#160;大致的意思是说,返回给我的List是一个只读的视图(view),有点像数据库里面的视图(View)和表(Base Table)之间的那种关系.我可以对它进行读操作,但是写操作是无效的.而这个function是applied lazily的,这个概念又有点像Hibernate里面的懒加载,只有在它需要被用的到时候才会调用.那到底什么时候这个方法会被调用到了?
```java
    TransformingRandomAccessList(
        List<F> fromList, Function<? super F, ? extends T> function) {
      this.fromList = checkNotNull(fromList);
      this.function = checkNotNull(function);
    }
    @Override public void clear() {
      fromList.clear();
    }
    @Override public T get(int index) {
      return function.apply(fromList.get(index));
    }
```
  &#160; &#160; &#160; &#160;比如在循环这个List,调用TransformingRandomAccessList这个类的get方法的时候,就会调用apply().再看看我作为Function参数传进去的方法:
  ```java
   private PersonDTO toDTO(PersonDO personDO) {
    PersonDTO ret = new PersonDTO();
    //这里简单get,set一下,当然也可以使用ModelMapper.map()
    ret.setName(personDO.getName());
    ret.setAge(personDO.getAge());
    return ret;
   }
  ```
  &#160; &#160; &#160; &#160;确实是每调用一次都会创建一个新的PersonDTO对象!难怪debug的时候发现地址值不一样.
  &#160; &#160; &#160; &#160;那么如果想用Lists.transform这种方式来code,并且达到相同的目的,应该怎么做了?其实API文档里面也写了:
  

> To avoid lazy evaluation when the returned list doesn't need to be a
   view, copy the returned list into a new list of your choosing.
   
&#160; &#160; &#160; &#160;那么只需要这样再操作一下,就可以了:
```java
List<PersonDTO> dtoList_4 = new ArrayList<>();
dtoList_4.addAll(dtoList_3);
```
&#160; &#160; &#160; &#160;其实API文档中还特别嘱咐了关于序列化的问题.之前有同事也是因为用了这个方法,碰到序列化出错的问题.现在想来,只能叹一句:这玩意不靠谱....

## 总结 ##

&#160; &#160; &#160; &#160;发现原因之后,因为项目要赶进度,所以我果断把文艺青年的写法换成了普通青年的写法.然后代码上线了,同时还fix掉了之前没有被同事发现的bug.
&#160; &#160; &#160; &#160;但是码了这么长,肯定不是叫大家都老老实实的做一个普通青年,不要装文艺.
&#160; &#160; &#160; &#160;优雅的编码方式我还是很崇尚的.不过踩了这个坑之后,我会觉得,如果我对一个新的东西并不真的理解,而且我有一套现成且成熟的解决方案能解决碰到的问题时,我会采用相对保守的方式来处理.

&#160; &#160; &#160; &#160;以上.










 


  [1]: http://7xsrzn.com1.z0.glb.clouddn.com/debug1.png 
  [2]: http://7xsrzn.com1.z0.glb.clouddn.com/debug2.png

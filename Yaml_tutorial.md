
## Definition
&#160; &#160; &#160; &#160;YAML Ain't Markup Language （Yaml不是标记语言，递归否定，可能是借鉴[GNU](https://zh.wikipedia.org/wiki/GNU)的梗）
  
## What It Is
 
&#160; &#160; &#160; &#160;YAML is a human friendly data serialization standard for all programming languages.
(YAML是一个human friendly的数据序列化标准，适用所有的编程语言)

## Relation to JSON
&#160; &#160; &#160; &#160;都是human readable的数据交换格式。但是json对程序更友好，而yaml把human readability的优先级设置得更高。yaml在不同的编程环境中交互的时候需要更复杂的逻辑处理。


## Relation to XML
&#160; &#160; &#160; &#160;虽然在很多应用领域两者处于相互竞争的状态，但两者之间并没有任何直接关系。YAML专注于做数据序列化，而XML是为了支持文档的结构化而设计的。
  
  
  
## Grammar
- Structure(结构体)，用空格展示。
- Sequence(数组)， 用“-”来表示。
- Map(键值对)，用“:”分隔。

## Demo
&#160; &#160; &#160; &#160;一般YAML文件的扩展名为.yaml,例如下面的文件就是Dijinchao.yaml

```json
name: 3D  # K-V
age: 23
work:
    company: duitang  # Structure,空格
    job_title: Develop Engineer
interests:
    - Python  #Sequence 用“-”表示
    - Linux   #Sequence 用“-”表示
    - 大保健   #Sequence 用“-”表示        

```

## YAML Online Editor

 - http://codebeautify.org/yaml-validator
 - http://codebeautify.org/yaml-to-json-xml-csv
 - http://www.yamllint.com/

## PyYaml

&#160; &#160; &#160; &#160;YAML是一个规范，各个语言都有对这个规范的实现。

 - C/C++ 
   - libyaml
   - yaml-cpp
 - Java:
   - JYaml
   - YamlBeans
 - Python
   - PyYaml
   - PySyck
 - ......
 
 
&#160; &#160; &#160; &#160;还有很多语言的各种实现版本，就不一一列举了。本次采用的pyresttest框架是用Python写的,其中就依赖了PyYaml.所以这里简略的介绍一下PyYaml，目的在于感性的认知pyresttest是如何工作的：
 
&#160; &#160; &#160; &#160;首先是[安装PyYaml](http://pyyaml.org/wiki/PyYAML),然后执行如下代码：

```python
>>> import yaml
>>> yaml.load("""
name: 3D  # K-V
age: 23
work:
    company: duitang  # Structure,空格
    job_title: Develop Engineer
interests:
    - Python  #Sequence 用“-”表示
    - Linux   #Sequence 用“-”表示
    - 大保健   #Sequence 用“-”表示
""") 

 
``` 

运行结果：

![图片](http://img4.duitang.com/uploads/item/201603/23/20160323005213_hMvRy.png)

&#160; &#160; &#160; &#160;可以看到是yaml.load()拿到了一个python的dict对象，拿到这个对象之后就可以用代码去做各种各样的逻辑处理了.所以测试这边只需要保证.yaml格式的文本文件书写正确，就能利用框架去做逻辑验证，完成自动化测试的需求。
 
  
  
## Reference
-  [YAML简介](http://www.ibm.com/developerworks/cn/xml/x-cn-yamlintro/)
- [The Official YAML Web Site](http://yaml.org/)
- [PyYaml](http://pyyaml.org/wiki/PyYAMLDocumentation)

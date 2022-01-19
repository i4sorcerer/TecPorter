# 熟悉YAML/YML格式配置文件使用

默认spring boot中支持的配置格式

- applciation.properties

- application.yaml

- application.yml

  ....

## 文件特性

- Yaml文件是JSON的超集，简洁强大
- 专门用来书写配置文件的语言

## 引入

在引入pring-boot-web依赖时，自动添加snakeyaml依赖用来解析yaml文件

## 语法特性

- 以缩进作为分隔符

- 支持定义简单类型和复杂类型以及列表类型定义

  ```yaml
  test:
  	host: localhost
  	port: 1234
  ```

- 通过@PropertySource注解是引用不到配置选项的，只能针对.properties(xml文件应该也是可以的？？？)
- 个人感觉相比于.properties格式，更加简洁。
- 个人感觉是以类似编程的方式来定义配置文件，properties则更像是纯文本形式的key->value集合
- 支持通过"---"的方式来进行profile的定义，不用定义在不同的文件中
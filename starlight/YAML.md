### 注释：`#` 

### 数组：

```yaml
-cat
-dog
-youyue
```

### 锚点和引用：

```yaml
defaults: &defaults
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  <<: *defaults

test:
  database: myapp_test
  <<: *defaults
```

等同于：

```yaml
defaults:
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  adapter:  postgres
  host:     localhost

test:
  database: myapp_test
  adapter:  postgres
  host:     localhost
```

	## 纯量

数值：字面值

布尔：`true`,`false`

null：`~`

时间：ISO8601，`2001-12-14t21:59:43.10-05:00 `，`1976-07-31`

使用两个感叹号表示强制类型转换

```yaml
e:	!!str 123
```

字符串：默认不使用引号，当包含空格或特殊字符，需要放在引号中，单引号双引号都可以用，但双引号不会对特殊字符转义，单引号中的单引号需要用单引号进行转义。

```yaml
str:  abc
str:  'bzc: csd'
```

字符串可以写成多行，但从第二行开始，必须要有一个但空格缩进，换行符会被转为空格

```yaml
str: 这是一段
  多行
  字符串
```

$\Rightarrow$`{ str: '这是一段 多行 字符串' }`

多行字符串可以使用`|`保留换行符，也可以使用`>`折叠换行

```yaml
this: |
  Foo
  Bar
that: >
  Foo
  Bar
```

$\Downarrow$

```yaml
{ this: 'Foo\nBar\n', that: 'Foo Bar\n' }
```

`+`表示保留文字块末尾的换行,`-`表示删除文字块末尾的换行
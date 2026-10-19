#### Java实体类属性和数据库字段都是大写字母，为什么返回给前端会变成小写字母

##### 问题描述

最近我在开发系统的过程中发现了一个问题，我数据库中数据表的字段名使用的是p_id，Java实体类中属性是pId，但是返回给前端的时候属性名却变成了小写的pid。

修改前

```
private Integer pId;

public void setPId(Integer pId) {
    this.pId = pId;
}

public Integer getPId() {
    return this.pId;
}
```

修改后

```
private Integer pId;

public Integer getpId() {
    return pId;
}

public void setpId(Integer pId) {
    this.pId = pId;
}
```

##### 原因

这是因为 Java 中的命名约定是使用驼峰命名法，即属性名的第一个单词小写，后面的单词首字母大写。

但是在返回给前端时，通常会使用 JSON 格式，而 JSON 中的属性名是区分大小写的，因此会将属性名转换为小写字母。

如果想在返回给前端时保留属性名的大小写，可以使用 Jackson 库中的 @JsonProperty 注解来指定属性名。
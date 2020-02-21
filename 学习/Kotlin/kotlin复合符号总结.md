##### 1. ?.

```
foo?.bar等同于java：

if(foo !=null){
    return foo.bar();
}else{
    return null;
}

foo !=null -> foo.bar()
foo ==null -> null
```


##### 2.!!
```
foo!!等同于java：

if(foo !=null){
    return foo;
}else{
    throw new NullPointException;
}

foo !=null -> foo
foo ==null -> NullPointException
```

##### 3.?:

```
类似于java的三木运算符
foo?:bar

foo !=null -> foo
foo ==null -> bar
```


##### 4.as?
```
foo as? Type 

foo is Type -> foo as Type
foo !is Type -> null

```

##### 5.?

```
foo:Foo? 

?等同于java中的@Nullable注解
不带?等同于java中的@NotNull

```

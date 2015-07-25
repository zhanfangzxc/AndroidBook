###java基础总结

> Return,Break,Continue的区别与用法

Break

```
1、switch语句中，break语句会终止其后语句的执行，退出switch语句
2、使一个循环立即结束，如果在循环中遇到break语句，循环立即结束，程序转到循环体后的第一个语句执行
```

Continue

```
1、continue语句作用是结束本次循环，即跳过本次循环体中余下尚未执行的语句，接着再一次进行循环条件的判断。
2、执行continue语句并没有使整个循环终止，在while和do-while循环中，continue语句使得流程直接跳到循环控制条件的测试部分，然后决定循环是否继续进行
```

Break和Continue的区别

```
1、Continue语句只结束本次循环，而不终止整个循环的执行，而Break语句则是结束整个循环过程，不再判断执行循环的条件是否成立
2、Continue的作用是跳过本次循环，强制执行下次循环
3、Continue语句只用在for，while，do-while等循环体中，常与if条件语句一起使用，用来加速循环
4、Continue不能用于switch语句
5、Break和Continue都可以用于循环语句中
```

Return

```
1、return从当前的方法中退出，返回到调用该方法的语句处，继续执行
2、return返回一个值给调用该方法的语句，返回值的数据类型必须与方法的声明中的返回值类型一致，如果不一样，可是使用强制类型转换
3、当方法返回值类型生命为void的时候，只使用return，不用返回任何值。
```

另外

```
return 退出该方法
break 退出背刺循环，执行循环体下面的语句
exit(i) 退出系统
```

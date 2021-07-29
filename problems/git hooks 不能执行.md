执行时提示错误：

hint: The '.husky/pre-commit' hook was ignored because it's not set as executable.
hint: You can disable this warning with `git config advice.ignoredHook false`.

原因解析:

由于 hook  文件没有执行权限，无法执行 hook



解决方案：

修改文件的执行权限：

```
chmod ug+x .husky/*
chmod ug+x .git/hooks/*
```


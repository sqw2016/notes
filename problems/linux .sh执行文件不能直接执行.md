.sh 文件不能直接执行，提示：

![image-20210728120656361](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210728120656361.png)

但是 dev 下的文件却能直接执行

![image-20210728120733784](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210728120733784.png)



# 可能的原因

## 1. linux 和 window 之间的不完全兼容

验证：

```bash
vim filename
:set ff?
```

![image-20210728133557227](C:\Users\lenovo\AppData\Roaming\Typora\typora-user-images\image-20210728133557227.png)

如果现实的 fileformat=dos 需要通过 :set fileformat = unix 修改文件格式。

或使用

```bash
sed -i  
```

替换文件结尾的错误字符

## 2. 查看文件头是否完整

```shell
#! bin/bash
```

上面的文件头不完整，所以找不到 bin/bash 文件


```bash
# if [[ $VAR == 'VALUE' ]]
if [ ]
then
elif [ ]
then
else
fi
```  

```bash
for item in list
do
done

while:
do
done

until [ ]
do
done
```


```bash
test="abcDEF"

# 把变量中的第一个字符换成大写
echo ${test^}
> AbcDEF

# 把变量中的所有小写字母，全部替换为大写
echo ${test^^}
> ABCDEF

# 把变量中的第一个字符换成小写
echo ${test,}
> abcDEF

# 把变量中的所有大写字母，全部替换为小写
> echo ${test,,}
abcdef
```
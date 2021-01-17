大家好，我是你们的东哥。

本篇是pandas100个骚操作的第一篇：**object类型自动转换**

完整系列可跟踪本项目，或者关注东哥的公众号专栏： [pandas100个骚操作](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzUzODYwMDAzNA==&action=getalbum&album_id=1699019347278561282#wechat_redirect)

---

在用pandas进行数据清洗的过程中，变量的类型转换是一个必然会遇到的步骤。清洗初期查看`dtypes`经常出现`object`类型，但其实变量本身可能就是个字符串，或者是数字（但因存在空值，导致出现了`object`类型）。

通常大家所熟知的方法是使用`astype`进行类型转换，或者自己利用`astype`造个轮子，写个函数方法实现自动转换类型。

本次东哥介绍一个pandas里可实现自动转换变量类型的方法`convert_dtypes`。利用它可以一次性全部转换为最理想的类型。

## 一、使用方法

默认情况下，`convert_dtypes`将尝试将Series或DataFrame中的每个Series转换为支持的dtypes。它可以对Series和DataFrame都直接使用。

这个方法的参数如下。
```python
# 是否应将对象dtypes转换为最佳类型
infer_objects bool，默认为True

# 对象dtype是否应转换为StringDtype()
convert_string bool，默认为True

# 如果可能，是否可以转换为整数扩展类型
convert_integer bool，默认为True

# 对象dtype是否应转换为BooleanDtypes()
convert_boolean bool，默认为True

# 如果可能，是否可以转换为浮动扩展类型。
# 如果convert_integer也为True，则如果可以将浮点数忠实地转换为整数，则将优先考虑整数dtype
convert_floating bool，默认为True
```
## 二、实例

下面看个例子。

首先创建一组数据，通过`dtype`规定每个变量的类型。

```python
df = pd.DataFrame(
    {
        "a": pd.Series([1, 2, 3], dtype=np.dtype("int32")),
        "b": pd.Series(["x", "y", "z"], dtype=np.dtype("O")),
        "c": pd.Series([True, False, np.nan], dtype=np.dtype("O")),
        "d": pd.Series(["h", "i", np.nan], dtype=np.dtype("O")),
        "e": pd.Series([10, np.nan, 20], dtype=np.dtype("float")),
        "f": pd.Series([np.nan, 100.5, 200], dtype=np.dtype("float")),
    }
)
```

### DataFraem 变量类型转换

先从整个对dataframe操作开始。

```python
>>> df
   a  b      c    d     e      f
0  1  x   True    h  10.0    NaN
1  2  y  False    i   NaN  100.5
2  3  z    NaN  NaN  20.0  200.0
```
```python
>>> df.dtypes
a      int32
b     object
c     object
d     object
e    float64
f    float64
dtype: object
```
通过结果可以看到，变量都是是创建时默认的类型。但其实变量是有整数、字符串、布尔的，其中有的还存在空值。
```python
>>> dfn = df.convert_dtypes()
>>> dfn
   a  b      c     d     e      f
0  1  x   True     h    10   <NA>
1  2  y  False     i  <NA>  100.5
2  3  z   <NA>  <NA>    20  200.0
```
下面使用`convert_dtypes`进行转换。
```python
>>> dfn.dtypes
a      Int32
b     string
c    boolean
d     string
e      Int64
f    Float64
dtype: object
```
变量类型已经转换为我们想要的了。

### Series 变量类型转换

对Series的转换也是一样的。下面的Seires中由于存在`nan`空值所以类型为`object`。

```python
s = pd.Series(["a", "b", np.nan])
>>> s
0      a
1      b
2    NaN
dtype: object
```

然后我们通过`convert_dtypes`成功转换为`String`。

```python
>>> s.convert_dtypes()
0       a
1       b
2    <NA>
dtype: string
```

如果未来增加了新类型，`convert_dtypes`方法也会同步更新，并支持新的变量类型。

---

<div align=center>
<p>欢迎关注我的原创公众号：<a href="https://mp.weixin.qq.com/s/QKGi7bO3mpCWmsFEwuFFTw">Python数据科学</a></p>
</div>

<div align=center>
<img src="https://github.com/xiaoyusmd/PythonDataScience/blob/main/images/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg?raw=true" width="300" height="300" />
</div>

<div align=center>
<p>公众号回复：<font color="red">666</font>，获取100G学习资料</p>
</div>


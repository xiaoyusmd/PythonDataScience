大家好，我是你们的东哥。

本篇是pandas100个骚操作的第2篇：**JSON自动解析为DataFrame**

完整系列可跟踪本项目，或者关注东哥的公众号专栏： [pandas100个骚操作](https://mp.weixin.qq.com/mp/appmsgalbum?__biz=MzUzODYwMDAzNA==&action=getalbum&album_id=1699019347278561282#wechat_redirect)

---

调用`API`和文档数据库会返回嵌套的`JSON`对象，当我们使用`Python`尝试将嵌套结构中的键转换为列时，数据加载到`pandas`中往往会得到如下结果：
```python
df = pd.DataFrame.from_records（results [“ issues”]，columns = [“ key”，“ fields”]）
```
![1](https://mmbiz.qpic.cn/sz_mmbiz_png/NOM5HN2icXzxuqckUGfvF4zGQ4Z1FofKWGeO2VBzJVJouKBRIf2lNWfU1M1icYzx7zXLNDtaiatJYS8AfHibTx6iaFQ/640?wx_fmt=png&tp=webp&wxfrom=5&wx_lazy=1&wx_co=1)

>说明：这里results是一个大的字典，issues是results其中的一个键，issues的值为一个嵌套JSON对象字典的列表，后面会看到JSON嵌套结构。

问题在于API返回了嵌套的`JSON`结构，而我们关心的键在对象中确处于不同级别。

嵌套的`JSON`结构张成这样的。


而我们想要的是下面这样的。



下面以一个API返回的数据为例，API通常包含有关字段的元数据。假设下面这些是我们想要的字段。

- key：JSON密钥，在第一级的位置。

- summary：第二级的“字段”对象。

- status name：第三级位置。

- statusCategory name：位于第4个嵌套级别。

如上，我们选择要提取的字段在issues列表内的`JSON`结构中分别处于4个不同的嵌套级别，一环扣一环。
```json
{
  "expand": "schema,names",
  "issues": [
    {
      "fields": {
        "issuetype": {
          "avatarId": 10300,
          "description": "",
          "id": "10005",
          "name": "New Feature",
          "subtask": False
        },
        "status": {
          "description": "A resolution has been taken, and it is awaiting verification by reporter. From here issues are either reopened, or are closed.",
          "id": "5",
          "name": "Resolved",
          "statusCategory": {
            "colorName": "green",
            "id": 3,
            "key": "done",
            "name": "Done",
          }
        },
        "summary": "Recovered data collection Defraglar $MFT problem"
      },
      "id": "11861",
      "key": "CAE-160",
    },
    {
      "fields": { 
... more issues],
  "maxResults": 5,
  "startAt": 0,
  "total": 160
}
```

## 一个不太好的解决方案

一种选择是直接撸码，写一个查找特定字段的函数，但问题是必须对每个嵌套字段调用此函数，然后再调用`.apply`到`DataFrame`中的新列。

为获取我们想要的几个字段，首先我们提取fields键内的对象至列：
```python
df = (
    df["fields"]
    .apply(pd.Series)
    .merge(df, left_index=True, right_index = True)
)
```


从上表看出，只有summary是可用的，issuetype、status等仍然埋在嵌套对象中。

下面是提取issuetype中的name的一种方法。
```python
# 提取issue type的name到一个新列叫"issue_type"
df_issue_type = (
    df["issuetype"]
    .apply(pd.Series)
    .rename(columns={"name": "issue_type_name"})["issue_type_name"]
)
df = df.assign(issue_type_name = df_issue_type)
```

像上面这样，如果嵌套层级特别多，就需要自己手撸一个递归来实现了，因为每层嵌套都需要调用一个像上面解析并添加到新列的方法。

对于编程基础薄弱的朋友，手撸一个其实还挺麻烦的，尤其是对于数据分析师，着急想用数据的时候，希望可以快速拿到结构化的数据进行分析。

下面东哥分享一个`pandas`的内置解决方案。


## 内置的解决方案

`pandas`中有一个牛逼的内置功能叫 `.json_normalize`。

`pandas`的文档中提到：将半结构化`JSON`数据规范化为平面表。

前面方案的所有代码，用这个内置功能仅需要3行就可搞定。步骤很简单，懂了下面几个用法即可。

确定我们要想的字段，使用 . 符号连接嵌套对象。

将想要处理的嵌套列表（这里是`results["issues"]`）作为参数放进 `.json_normalize` 中。

过滤我们定义的FIELDS列表。
```python
FIELDS = ["key", "fields.summary", "fields.issuetype.name", "fields.status.name", "fields.status.statusCategory.name"]
df = pd.json_normalize(results["issues"])
df[FIELDS]
```


没错，就这么简单。


## 其它操作

### 记录路径

除了像上面那样传递`results["issues"]`列表之外，我们还使用`record_path`参数在`JSON`对象中指定列表的路径。
```python
# 使用路径而不是直接用results["issues"]
pd.json_normalize(results, record_path="issues")[FIELDS]
```
### 自定义分隔符

还可以使用sep参数自定义嵌套结构连接的分隔符，比如下面将默认的“.”替换“-”。
```python
### 用 "-" 替换默认的 "."
FIELDS = ["key", "fields-summary", "fields-issuetype-name", "fields-status-name", "fields-status-statusCategory-name"]
pd.json_normalize(results["issues"], sep = "-")[FIELDS]
```
### 控制递归

如果不想递归到每个子对象，可以使用`max_level`参数控制深度。在这种情况下，由于`statusCategory.name`字段位于`JSON`对象的第4级，因此不会包含在结果`DataFrame`中。
```python
# 只深入到嵌套第二级
pd.json_normalize(results, record_path="issues", max_level = 2)
```
下面是`.json_normalize`的`pandas`官方文档说明，如有不明白可自行学习，本次东哥就介绍到这里。


>参考：
>pandas官方文档：https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.json_normalize.html
>
>https://medium.com/swlh/converting-nested-json-structures-to-pandas-dataframes-e8106c59976e


---

<div align=center>
<p>欢迎关注我的原创公众号：<a href="https://mp.weixin.qq.com/s/QKGi7bO3mpCWmsFEwuFFTw">Python数据科学</a></p>
</div>

<div align=center>
<img src="https://github.com/xiaoyusmd/PythonDataScience/blob/main/images/%E5%85%AC%E4%BC%97%E5%8F%B7%E4%BA%8C%E7%BB%B4%E7%A0%81.jpg?raw=true" width="300" height="300" />
</div>

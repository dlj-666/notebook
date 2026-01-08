---
[[html form]]
---
html表单用于收集用户的输入信息

# 结构：
- <form>元素用于创建表单，action属性定义了表单数据提交到的目标URL
**语法结构**：<form action="/" (URL) method="post"(http方法如"get" "post")>
</form>

- <input>元素用于获取用户输入

- <label>元素和<input>元素相关联语法:
<label for="element">hello world</lable>
<input type="text" id="element", name="form`s name", required/> #这里id要和for一样 required表示这行必填，name是表单的名称

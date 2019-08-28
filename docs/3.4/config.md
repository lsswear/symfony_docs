## 配置参数

### 查看命令

### 解释

<div class="card fluid">
    <p>名称：secret</p>
    <p>参数名：kernel.secret</p>
    <p>类型：string</p>
    <p>默认：</p>
    <p>必填：是</p>
    <p>解释：
    <small>
    应用中唯一，由字符、数字、符号随机组成，32位左右。应用于记住登录时长、Edge Side Includes/数据缓冲服务器。应用中使用不可变随机字符串、为安全相关的操作增加熵时刻使用。更改此值将使所有签名的uri和cookie无效,故改后应重新生成应用程序缓存并注销所有应用程序用户。
    </small></p>
</div>

|名称|参数名|类型|默认|必填|
|:---|:---|:---|:---|:---|
|secret|kernel.secret|string||是|

**secret**

*应用中唯一，由字符、数字、符号随机组成，32位左右。应用于记住登录时长、Edge Side Includes/数据缓冲服务器。应用中使用不可变随机字符串、为安全相关的操作增加熵时刻使用。更改此值将使所有签名的uri和cookie无效,故改后应重新生成应用程序缓存并注销所有应用程序用户。*

<table>
  <caption>People</caption>
  <thead>
    <tr>
      <th>Name</th>
      <th>Surname</th>
      <th>Alias</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td data-label="Name">Chad</td>
      <td data-label="Surname">Wilberts</td>
      <td data-label="Alias">MrOne</td>
    </tr>
    <tr>
      <td data-label="Name">Adam</td>
      <td data-label="Surname">Smith</td>
      <td data-label="Alias">TheSmith</td>
    </tr>
    <tr>
      <td data-label="Name">Sophia</td>
      <td data-label="Surname">Canderson</td>
      <td data-label="Alias">Candee</td>
    </tr>
  </tbody>
</table>


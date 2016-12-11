# git  省略输入密码提交的方法

1. 如果是 clone下来的repo，那么需要修改提交的地址，首先删除url

   ![pwd1](../images/git/pwd1.jpg)

2. 删除后添加一个新的地址 格式为 https://用户名:密码/@github.com/用户名/仓库名.git

   ![pwd2](../images/git/pwd2.jpg)

3. 之后在 git push origin master 就可以不用输入密码了
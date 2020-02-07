# WeAct Studio WeAct-tc.cn
## 如何使用
1. Clone 仓库 
``` 
git clone -b WeAct https://github.com/WeActTC/WeActTC.github.io.git 
git submodule init
git submodule update
```
3. 下载&&安装 [node.js](https://nodejs.org/en/download/)
4. 在网站目录下执行
```
npm install hexo-cli -g
npm install 
npm install hexo-deployer-git
```
网站clone完成

## 文章编写
文章采用MarkDown编写

`hexo g` 生成网页

`hexo s` 实时预览网页

双击 `hexo update.bat` 上传网站至github

保存源文件
```
git branch -a # 查看当前分支，确认当前分支是否是WeAct
git add .
git commit -m "更改的内容"
git push git@weact.github.com:WeActTC/WeActTC.github.io #.ssh/config设置了WeActTC的密匙，请添加
```

```
hexo : 无法加载文件 C:\Users\zhuyix\AppData\Roaming\npm\hexo.ps1，因为在此系统上禁止运行脚本。有关详细信息，请参阅 http
s:/go.microsoft.com/fwlink/?LinkID=135170 中的 about_Execution_Policies。
```
出现这个问题，如下操作：
> 使用管理员身份运行 PowerShell，输入 `set-executionpolicy remotesigned` 然后选 `Y` 即可

## git 多用户配置
参考此教程：
https://blog.csdn.net/yuanlaijike/article/details/95650625


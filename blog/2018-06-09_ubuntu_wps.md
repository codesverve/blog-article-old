# Ubuntu下wps开多个窗口（不是标签页）的方法


## 方案一、如果没有既要开多个窗口，又要单个窗口开多个标签页的需求的话，可以采用该方案

左上角按钮 -> 进入选项 -> 勾选 在任务栏中显示所有按钮选择框 -> 最后确定

该方案缺点是单个窗口只能开一个标签页

## 方案二、使用命令行启动一个空的wps界面，之后再在界面上打开文件（利用在命令中加上-multiply参数的方法）

word文档：`nohup /opt/kingsoft/wps-office/office6/wps -multiply > /dev/null 2>&1 &`
excel文档：`nohup /opt/kingsoft/wps-office/office6/et -multiply > /dev/null 2>&1 &`
ppt文档：`nohup /opt/kingsoft/wps-office/office6/wpp -multiply > /dev/null 2>&1 &`

为了方便以后使用，可以将命令加入.bashrc文件（注意最后是两个大于号，漏写一个会出大问题，建议复制粘贴）
```
echo alias wps-word='nohup /opt/kingsoft/wps-office/office6/wps -multiply > /dev/null 2>&1 &' >> ~/.bashrc
echo alias wps-excel='nohup /opt/kingsoft/wps-office/office6/et -multiply > /dev/null 2>&1 &' >> ~/.bashrc
echo alias wps-ppt='nohup /opt/kingsoft/wps-office/office6/wpp -multiply > /dev/null 2>&1 &' >> ~/.bashrc
```
使当前命令行及机器上.bashrc生效需执行，否则重启后生效
`source ~/.bashrc`

下次直接在命令行中使用
`wps-word`、`wps-excel`、`wps-ppt`命令即可

该方案缺点是操作麻烦

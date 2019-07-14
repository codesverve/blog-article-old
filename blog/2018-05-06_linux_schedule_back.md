# linux 定时备份文件

## 文件备份脚本代码

```shell
#!/bin/bash

# 备份源文件夹
targetFile=/home/vince/eclipse-workspace/
# 备份目的地文件夹
backdir=/home/vince/backup/code


datetime=`date +"%Y-%m-%d %H:%M:%S"`
# 保留两天内的备份文件
date2before=`date -d "-2day" +"%Y-%m-%d %H:%M:%S"`

for file_back in $backdir/*
do
    filename=`basename "$file_back"`
    if [[ $date2before > $filename ]]; then
        echo "rm"
        eval rm -rf \'$file_back\'
    fi
done

mkdir ${backdir}/"${datetime}"

cd ${backdir}/"${datetime}"

mkdir zx-plugin-zxtracker

cp -rf ${targetFile} ./

echo 'backup done'
设置定时任务
1. sudo service cron start

2. sudo crontab -e                # 接下来会让你选择一个文本编辑器，输入1-4对应所选编辑器

3. 添加定时任务计划，输入： 0 10 * * * /home/vince/backup/backup.sh            # 命令对应：分 时 天 月 周 sh文件地址，周中0表示星期天    # 开机执行任务设置参数 @reboot /home/vince/backup/backup.sh

4. sudo service cron restart            # 重启定时器生效

```

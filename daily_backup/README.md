# VPS 每日备份

## 备份列表

-rw-r--r-- 1 admin admin 1.1G Mar 15 04:02 daily_backup/vps-backup-20260315_040001.tar.gz
-rw-rw-r-- 1 admin admin 928M Mar 14 17:28 daily_backup/vps-backup-20260314_172533.tar.gz

## 恢复方法

```bash
# 下载备份
wget https://github.com/yanqiangS/vps-backup/raw/main/daily_backup/vps-backup-20260315_040001.tar.gz

# 解压
tar -xzf vps-backup-20260315_040001.tar.gz

# 执行恢复脚本
bash vps-backup-20260315_040001/RESTORE.sh vps-backup-20260315_040001
```

## 保留策略

- 保留最近 30 天的每日备份
- 每月 1 号的备份永久保留
- 自动清理超过 30 天的备份

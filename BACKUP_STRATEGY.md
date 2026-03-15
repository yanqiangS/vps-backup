# VPS 备份策略说明

## 📋 备份架构

```
┌─────────────────────────────────────────────────────────────┐
│                    每日自动备份 (03:00)                      │
│              daily_backup.sh → 本地完整备份                   │
└─────────────────────────────────────────────────────────────┘
                              ↓
        ┌────────────────────────────────────────┐
        │                                        │
        ↓                                        ↓
┌───────────────────┐                  ┌──────────────────┐
│  本地完整备份      │                  │  GitHub 备份      │
│  (保留 30 天)       │                  │  (仅索引)         │
│  /home/admin/     │                  │  github.com/     │
│  vps-backup-local/│                  │  yanqiangS/vps-  │
│  完整 .tar.gz 文件  │                  │  backup/         │
│  大小：~1GB       │                  │  README.md 列表   │
└───────────────────┘                  └──────────────────┘
```

## 🕐 备份时间表

| 时间 | 脚本 | 功能 | 目标 |
|------|------|------|------|
| 03:00 | `daily_backup.sh` | 完整本地备份 | `/home/admin/syq/important/` |
| 04:00 | `daily_github_backup.sh` | 创建备份 + 推送索引 | GitHub + 本地 |
| 05:00 | `local_backup_retention.sh` | 管理本地大文件备份 | `/home/admin/vps-backup-local/` |

## ⚠️ 为什么 GitHub 备份失败？

### 问题原因
1. **备份文件过大** (1.1GB) - GitHub 推荐单个文件 <100MB
2. **Git LFS 未配置** - 大文件需要 Git LFS 支持
3. **网络超时** - 大文件推送容易超时

### 解决方案
✅ **已实施**:
- 添加 `.gitignore` 排除 `*.tar.gz` 大文件
- GitHub 仓库仅保留备份索引 (README.md)
- 大文件存储在本地 `/home/admin/vps-backup-local/`
- 优化压缩排除规则，减少不必要文件

## 📦 恢复方法

### 从本地备份恢复
```bash
# 查看可用备份
ls -lht /home/admin/vps-backup-local/*.tar.gz

# 解压备份
cd /home/admin
tar -xzf /home/admin/vps-backup-local/vps-backup-YYYYMMDD_HHMMSS.tar.gz

# 恢复数据
bash vps-backup-YYYYMMDD_HHMMSS/RESTORE.sh
```

### 从 GitHub 恢复 (仅索引)
```bash
# 克隆仓库查看备份列表
git clone git@github.com:yanqiangS/vps-backup.git
cat vps-backup/daily_backup/README.md

# 根据索引找到对应的本地备份文件
# 然后从本地恢复
```

## 🔧 维护命令

```bash
# 查看备份状态
ls -lh /home/admin/vps-backup-local/

# 手动运行备份
bash /home/admin/syq/scripts/daily_github_backup.sh

# 清理旧备份
bash /home/admin/syq/scripts/local_backup_retention.sh

# 查看备份日志
tail -50 /home/admin/syq/logs/github_backup.log
```

## 📊 存储策略

| 位置 | 保留策略 | 大小限制 | 用途 |
|------|---------|---------|------|
| 本地备份目录 | 30 天 | 无限制 | 完整恢复 |
| GitHub | 永久 (索引) | <100MB/文件 | 版本追踪 |
| 外部存储 (可选) | 90 天 | 无限制 | 长期归档 |

## 🚨 故障处理

### GitHub 推送失败
1. 检查本地备份是否成功：`ls -lh /home/admin/*.tar.gz`
2. 查看日志：`tail -30 /home/admin/syq/logs/github_backup.log`
3. 手动推送索引：`cd /home/admin/vps-backup && git push`

### 本地备份空间不足
1. 清理旧备份：`bash local_backup_retention.sh`
2. 检查磁盘：`df -h`
3. 考虑挂载外部存储

---

**最后更新**: 2026-03-15  
**状态**: ✅ 已优化

https://unix.stackexchange.com/questions/457388/how-to-find-out-the-variable-names-for-debconf-set-selections
---

**如何找出 debconf-set-selections 的变量名？**  
提问于 7 年前，5 个月前  
修改于 1 年前，5 个月前  
查看 17k 次  

---

假设我想通过脚本安装软件包（如 `mysql`），并且不想在安装过程中回答任何配置问题（例如设置 root 密码）。这时，我会通过 `debconf-set-selections` 预设相关变量：  

```bash
echo mysql-server-5.5 mysql-server/root_password password xyzzy | debconf-set-selections
echo mysql-server-5.5 mysql-server/root_password_again password xyzzy | debconf-set-selections
```

这是我从教程中找到的。但我不清楚的是：**如何找出这些变量名？** 他怎么知道要设置的是 `mysql-server-5.5 mysql-server/root_password password` 和 `mysql-server-5.5 mysql-server/root_password_again`？

我知道可以通过 `dpkg-deb -R package.deb EXTRACTDIR/` 解压软件包，但没找到这些变量存储在哪里。

**对于其他软件包，我该如何找出 debconf 的变量名？**  

---

**回答 1**（得票最高）：  

您可以通过 `debconf-show` 命令查看已安装软件包的 debconf 变量。例如：  

```bash
$ sudo debconf-show mysql-server-5.7
* mysql-server/root_password: (password omitted)
* mysql-server/root_password_again: (password omitted)
  mysql-server-5.7/start_on_boot: true
  mysql-server/no_upgrade_when_using_ndb:
  mysql-server/password_mismatch:
  mysql-server-5.7/really_downgrade: false
  mysql-server-5.7/nis_warning:
  mysql-server-5.7/postrm_remove_databases: false
  mysql-server-5.7/installation_freeze_mode_active:
```

您也可以使用 `debconf-show --listowners` 列出所有已安装软件包中拥有 debconf 变量的包名。如果您不确定具体的包名，可以这样操作：  

```bash
# debconf-show --listowners | grep mysql | xargs debconf-show
* mysql-server/root_password: (password omitted)
* mysql-server/root_password_again: (password omitted)
  mysql-server-5.7/postrm_remove_databases: false
  mysql-server-5.7/nis_warning:
  mysql-server-5.7/installation_freeze_mode_active:
  mysql-server/password_mismatch:
  mysql-server-5.7/start_on_boot: true
  mysql-server/no_upgrade_when_using_ndb:
  mysql-server-5.7/really_downgrade: false
```

---

**回答 2**：  

您可以查看 debconf 数据库中的现有设置，通过 `debconf-get-selections` 命令。这在已经安装过软件包的情况下非常有用。  

另外，这些设置通常出现在软件包维护脚本中。通过您之前提到的 `dpkg-deb -R` 命令解压软件包后，可以在 `EXTRACTDIR/DEBIAN` 目录下找到这些脚本。  

以 `lightdm` 软件包为例：  

```bash
$ grep db_ lightdm/DEBIAN -R
lightdm/DEBIAN/postrm:  db_purge
lightdm/DEBIAN/prerm:    db_unregister shared/default-x-display-manager
lightdm/DEBIAN/prerm:    if db_get shared/default-x-display-manager; then
lightdm/DEBIAN/prerm:      db_metaget shared/default-x-display-manager owners
lightdm/DEBIAN/prerm:      db_subst shared/default-x-display-manager choices "$RET"
lightdm/DEBIAN/prerm:      db_get shared/default-x-display-manager
lightdm/DEBIAN/prerm:     if db_get "$RET"/daemon_name; then
lightdm/DEBIAN/prerm:        db_fset shared/default-x-display-manager seen false
lightdm/DEBIAN/prerm:        db_input critical shared/default-x-display-manager || true
lightdm/DEBIAN/prerm:        db_go
lightdm/DEBIAN/prerm:          db_get shared/default-x-display-manager
lightdm/DEBIAN/prerm:          db_get "$RET"/daemon_name
lightdm/DEBIAN/postinst:  if db_get shared/default-x-display-manager; then
lightdm/DEBIAN/postinst:    if db_get "$DEFAULT_DISPLAY_MANAGER"/daemon_name; then
lightdm/DEBIAN/postinst:db_stop
lightdm/DEBIAN/config:if db_metaget shared/default-x-display-manager owners; then
lightdm/DEBIAN/config:if db_metaget shared/default-x-display-manager choices; then
lightdm/DEBIAN/config:  db_subst shared/default-x-display-manager choices "$OWNERS" || :
lightdm/DEBIAN/config:  db_fset shared/default-x-display-manager seen false || :
lightdm/DEBIAN/config:    db_set shared/default-x-display-manager "$CURRENT_DEFAULT"
lightdm/DEBIAN/config:  if db_get shared/default-x-display-manager; then
lightdm/DEBIAN/config:    db_set shared/default-x-display-manager lightdm
lightdm/DEBIAN/config:    db_fset shared/default-x-display-manager seen true
lightdm/DEBIAN/config:    db_input high shared/default-x-display-manager || :
lightdm/DEBIAN/config:    db_go || :
lightdm/DEBIAN/config:if db_get shared/default-x-display-manager; then
```

这些 `db_*` 函数是用于处理 debconf 的辅助函数，来自 `/usr/share/debconf/confmodule`。  

因此，对于 `lightdm` 来说，`shared/default-x-display-manager` 是一个重要的 debconf 键。

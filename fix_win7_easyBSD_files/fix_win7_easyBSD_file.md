# fix win7 easyBCD file

因为删除 easyBCD 条目删错了把win7删掉留下 linux, 遂开始修复.

- 机器: ASUS X450V
- 系统: windows 7

## 步骤
1. 制作好PE盘, 按F2进入bios, 查看启动选项, 看到 USB 条目, 退出.
2. 当出现开机初始界面按住 esc 键 3秒, 选择 winPE 的 USB 启动.
3. 可以用 winPE 盘的工具 bootice 工具, 修改 BCD 文件(位置在 `c:\Windows\boot\BCD`)
4. 修改完后点保存, 重启, 再重复第1步, 选电脑原硬盘启动.
5. win7启动成功,修复完成.

**tip**: 也可用之前备份的 BCD 文件直接替换(替换前可以备份个 `BCD.bak`).
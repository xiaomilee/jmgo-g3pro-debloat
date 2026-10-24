# jmgo-g3pro-debloat

坚果投影仪 **JMGO G3 Pro**（国内版）去广告/精简预装实测清单，参照 [CleanYourTV](https://github.com/Gathin23/cleanyourtv) 项目思路做的实机适配。

## 设备信息

| 项目 | 值 |
|------|-----|
| 型号 | JmGO_G3pro |
| 系统 | Android 4.4.2 (SDK 19)，JMGO OS |
| 连接方式 | Wi-Fi ADB（`adb connect <IP>:5555`） |

## 精简前必读的三个坑（实测踩过）

1. **开启 ADB 调试**：坚果系统没有叫"ADB"的菜单。在 设置 → 关于设备 → 对版本号一行**连按 OK 键 7 次**，出现"已进入开发者模式/ADB调试已打开，请重启"提示后**必须重启投影仪**才生效。部分机型可用沙发管家等工具直接开启"无线ADB"，同样需要重启一次 adbd 才能连接。
2. **必须用 `pm block` 而不是 `pm disable-user`**：本机内存太小，`pm disable-user` 触发包管理扫描时会被系统 OOM 杀掉（返回码 137）。`pm block` 是 Android 4.4 时代的标准命令，效果相同且可随时恢复。
3. **`com.xiaodianshi.tv.yst` 不是广告**——它是 B 站正版 TV 应用"云视听小电视"，建议保留。

## 使用方法

```bash
# 连接（需与投影仪同一局域网）
adb connect <投影仪IP>:5555

# 先备份应用清单
adb shell pm list packages -f > packages-all.txt
adb shell pm list packages -3 > packages-3rd.txt

# 执行精简（逐条执行）
adb shell pm block com.jmgo.preinstallmanager
adb shell pm block com.google.tv.discovery
adb shell pm block com.jmgo.qy.player
adb shell pm block com.jmgo.illustration
```

完整清单（含注释和可选项）见 [debloat.txt](debloat.txt)。

## 恢复

```bash
adb shell pm unblock <包名>
# 例：adb shell pm unblock com.jmgo.preinstallmanager
```

## 可选：替换桌面

坚果桌面内含推荐位/广告位。可安装当贝桌面 4.1.7（支持 Android 4.x，CleanYourTV 的 Aerial 启动器要求 Android 8+ 装不了）后停用坚果桌面：

```bash
adb shell pm block com.jmgo.launcher      # 停用坚果桌面
adb shell pm unblock com.jmgo.launcher    # 恢复
```

验证：按遥控器主页键后 `adb shell dumpsys window | grep mCurrentFocus`，应显示 `com.dangbei.tvlauncher`。

## 本机可装且实测可用的应用（Android 4.4.2）

| 应用 | 版本 | 说明 |
|------|------|------|
| 当贝桌面 | 4.1.7 | 替换坚果桌面，设置里可关资讯推荐 |
| SmartTube | 32.68 beta | 开源 YouTube 客户端，包名是 `org.smarttube.beta`，设备码登录 Google 账号 |
| BBLL（B站第三方） | v1.4.9 api19 | v1.5+ 不再支持 4.4；手机B站扫码登录 |
| 云视听小电视 | 1.8.9 | B站正版TV版，机器预装，现行版本支持 Android 4.2+ |
| 央视影音TV版 | 6.7.3 | 央视直播/回看；央视频TV新版要求 5.0+ 装不上 |

**装不上的**：Aerial 电视桌面（`INSTALL_FAILED_OLDER_SDK`，最低 Android 5.0）、CleanYourTV 启动器（最低 Android 8.0）、央视频TV新版。

## 其他提示

- 老机器上多 Dex 应用**首次启动**会做 DexOpt 优化，耗时长甚至 ANR 被杀，**再启动一次**即正常；
- adbd 跑久了可能僵死（`adb devices` 显示 offline），**重启投影仪**即恢复；
- Git Bash 里 `adb shell` 的远程路径（如 `/data/...`）会被转成本地路径，需加 `MSYS_NO_PATHCONV=1`；
- 不用 ADB 时建议关闭"无线ADB"开关，5555 端口不再对局域网开放。

## backup/ 目录

- `packages-all.txt` — 全部 76 个包的清单（含 APK 路径）
- `packages-3rd.txt` — 第三方应用（精简前）
- `packages-disabled.txt` — 精简前已禁用的包（空）

## 致谢

- [CleanYourTV](https://github.com/Gathin23/cleanyourtv) —— 项目思路与 JMGO 参考配置来源

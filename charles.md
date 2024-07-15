# Charles

## iOS

安装证书：设置-通用-描述文件

信息证书：设置-通用-关于本机-证书信息设置

# Android无法抓取https包

证书配置完成，但https抓包失败。考虑代码问题。

因为Android 7+ 修改了默认的网络安全性配置。

```xml
<!-- Android 6 API 23 及更低版本 -->
<!-- 默认允许所有明文通信 -->
<base-config cleartextTrafficPermitted="true">
    <trust-anchors>
        <!-- 信任系统预装 CA 证书 -->
        <certificates src="system" />
        <!-- 信任用户添加的 CA 证书，Charles 和 Fiddler 抓包工具安装的证书属于此类 -->
        <certificates src="user" />
    </trust-anchors>
</base-config>

<!-- Android 7.0（API 24）到 Android 8.1（API 27） -->
<!-- 默认允许所有明文通信 -->
<base-config cleartextTrafficPermitted="true">
    <trust-anchors>
        <!-- 信任系统预装 CA 证书 -->
        <certificates src="system" />
    </trust-anchors>
</base-config>

<!-- Android 9.0（API 28）及更高版本 -->
<!-- 默认禁止所有明文通信 -->
<base-config cleartextTrafficPermitted="false">
    <trust-anchors>
        <!-- 信任系统预装 CA 证书 -->
        <certificates src="system" />
    </trust-anchors>
</base-config>
```
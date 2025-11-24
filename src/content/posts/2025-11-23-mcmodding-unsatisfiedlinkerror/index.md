---
title: "Minecraft Modding: Apple Silicon Mac での UnsatisfiedLinkError を強引に解決する"
published: 2025-11-23
description: 'Apple Silicon Macで古いMinecraftバージョン(1.12.2)向けのModdingをする際、arm64ネイティブなJDKを使うと発生する UnsatisfiedLinkError を、Mixinで強引に解決する'
image: ''
tags: ["Coding", "Minecraft", "Minecraft Modding"]
category: 'Minecraft Modding'
draft: false 
lang: ''
---

:::note[TL;DR]
- Apple Silicon Macで1.12.2 Mod開発をしている場合、Rosetta 2経由でx86_64 JDKを使っているかもしれない
- ただArm64ネイティブなJDKを使うと、`com.mojang.text2speech.NarratorOSX`が読み込まれた際に`UnsatisfiedLinkError`が発生してクラッシュする
- `com.mojang.text2speech.Narrator#getNarrator`が`NarratorOSX`を返すのを阻止するため、
Mixinで`NarratorDummy`を返すようにOverwriteする。
バニラへのMixinなので、`IEarlyMixinLoader`を使う。
:::

:::note[注意]
- 足りないネイティブライブラリを追加するというような、根本的な解決ではない
- Mixinを使用できる環境であることが前提
:::

# これは何

Apple Siliconを搭載したMacで、Minecraft 1.12.2に向けたModdingをしている際、
JDKにarm64ネイティブなもの(例えばAzul Zulu Community)を使用すると、ゲーム起動時に`UnsatisfiedLinkError`が出てクラッシュした。

調べると、`NarratorOSX`が読み込まれることで発生しているので、Mixinを使って強引に回避した。

<details>
  <summary>Crash Report</summary>

```
---- Minecraft Crash Report ----
// But it works on my machine.

Time: 10/29/25 6:17 PM
Description: Initializing game

java.lang.UnsatisfiedLinkError: /private/var/folders/t_/t3k2_6sj60q1y9q8dy5b_w040000gn/T/jna-93968178/jna7821772639012253071.tmp: dlopen(/private/var/folders/t_/t3k2_6sj60q1y9q8dy5b_w040000gn/T/jna-93968178/jna7821772639012253071.tmp, 0x0001): tried: '/private/var/folders/t_/t3k2_6sj60q1y9q8dy5b_w040000gn/T/jna-93968178/jna7821772639012253071.tmp' (fat file, but missing compatible architecture (have 'i386,x86_64', need 'arm64e' or 'arm64e.v1' or 'arm64' or 'arm64')), '/System/Volumes/Preboot/Cryptexes/OS/private/var/folders/t_/t3k2_6sj60q1y9q8dy5b_w040000gn/T/jna-93968178/jna7821772639012253071.tmp' (no such file), '/private/var/folders/t_/t3k2_6sj60q1y9q8dy5b_w040000gn/T/jna-93968178/jna7821772639012253071.tmp' (fat file, but missing compatible architecture (have 'i386,x86_64', need 'arm64e' or 'arm64e.v1' or 'arm64' or 'arm64'))
at java.lang.ClassLoader$NativeLibrary.load(Native Method)
at java.lang.ClassLoader.loadLibrary0(ClassLoader.java:1950)
at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1832)
at java.lang.Runtime.load0(Runtime.java:783)
at java.lang.System.load(System.java:1100)
at com.sun.jna.Native.loadNativeDispatchLibraryFromClasspath(Native.java:947)
at com.sun.jna.Native.loadNativeDispatchLibrary(Native.java:922)
at com.sun.jna.Native.<clinit>(Native.java:190)
at com.sun.jna.Pointer.<clinit>(Pointer.java:54)
at ca.weblite.objc.Proxy.<init>(Proxy.java:126)
at ca.weblite.objc.Proxy.<init>(Proxy.java:118)
at ca.weblite.objc.NSObject.<init>(NSObject.java:112)
at ca.weblite.objc.NSObject.<init>(NSObject.java:102)
at com.mojang.text2speech.NarratorOSX.<init>(NarratorOSX.java:18)
at com.mojang.text2speech.Narrator.getNarrator(Narrator.java:22)
at net.minecraft.client.gui.chat.NarratorChatListener.<init>(NarratorChatListener.java:18)
at net.minecraft.client.gui.chat.NarratorChatListener.<clinit>(NarratorChatListener.java:17)
at net.minecraft.client.gui.GuiIngame.<init>(GuiIngame.java:126)
at net.minecraftforge.client.GuiIngameForge.<init>(GuiIngameForge.java:101)
at net.minecraft.client.Minecraft.init(Minecraft.java:584)
at net.minecraft.client.Minecraft.run(Minecraft.java:422)
at net.minecraft.client.main.Main.main(Main.java:118)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at net.minecraft.launchwrapper.Launch.launch(Launch.java:165)
at net.minecraft.launchwrapper.Launch.main(Launch.java:29)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at net.minecraftforge.gradle.GradleStartCommon.launch(GradleStartCommon.java:87)
at GradleStart.main(GradleStart.java:19)

No Mixin Metadata is found in the Stacktrace.


A detailed walkthrough of the error, its code path and all known details is as follows:
---------------------------------------------------------------------------------------

-- Head --
Thread: Client thread
Stacktrace:
at java.lang.ClassLoader$NativeLibrary.load(Native Method)
at java.lang.ClassLoader.loadLibrary0(ClassLoader.java:1950)
at java.lang.ClassLoader.loadLibrary(ClassLoader.java:1832)
at java.lang.Runtime.load0(Runtime.java:783)
at java.lang.System.load(System.java:1100)
at com.sun.jna.Native.loadNativeDispatchLibraryFromClasspath(Native.java:947)
at com.sun.jna.Native.loadNativeDispatchLibrary(Native.java:922)
at com.sun.jna.Native.<clinit>(Native.java:190)
at com.sun.jna.Pointer.<clinit>(Pointer.java:54)
at ca.weblite.objc.Proxy.<init>(Proxy.java:126)
at ca.weblite.objc.Proxy.<init>(Proxy.java:118)
at ca.weblite.objc.NSObject.<init>(NSObject.java:112)
at ca.weblite.objc.NSObject.<init>(NSObject.java:102)
at com.mojang.text2speech.NarratorOSX.<init>(NarratorOSX.java:18)
at com.mojang.text2speech.Narrator.getNarrator(Narrator.java:22)
at net.minecraft.client.gui.chat.NarratorChatListener.<init>(NarratorChatListener.java:18)
at net.minecraft.client.gui.chat.NarratorChatListener.<clinit>(NarratorChatListener.java:17)
at net.minecraft.client.gui.GuiIngame.<init>(GuiIngame.java:126)
at net.minecraftforge.client.GuiIngameForge.<init>(GuiIngameForge.java:101)
at net.minecraft.client.Minecraft.init(Minecraft.java:584)

-- Initialization --
Details:
Stacktrace:
at net.minecraft.client.Minecraft.run(Minecraft.java:422)
at net.minecraft.client.main.Main.main(Main.java:118)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at net.minecraft.launchwrapper.Launch.launch(Launch.java:165)
at net.minecraft.launchwrapper.Launch.main(Launch.java:29)
at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
at java.lang.reflect.Method.invoke(Method.java:498)
at net.minecraftforge.gradle.GradleStartCommon.launch(GradleStartCommon.java:87)
at GradleStart.main(GradleStart.java:19)

-- System Details --
Details:
Minecraft Version: 1.12.2
Operating System: Mac OS X (aarch64) version 26.0.1
Java Version: 1.8.0_472, Azul Systems, Inc.
Java VM Version: OpenJDK 64-Bit Server VM (mixed mode), Azul Systems, Inc.
Memory: 2332871168 bytes (2224 MB) / 2857369600 bytes (2725 MB) up to 5726797824 bytes (5461 MB)
JVM Flags: 2 total; -Xms1G -Xmx6G
IntCache: cache: 0, tcache: 0, allocated: 0, tallocated: 0
FML: MCP 9.42 Powered by Forge 14.23.5.2847 14 mods loaded, 14 mods active
States: 'U' = Unloaded 'L' = Loaded 'C' = Constructed 'H' = Pre-initialized 'I' = Initialized 'J' = Post-initialized 'A' = Available 'D' = Disabled 'E' = Errored

	| State  | ID                  | Version      | Source                               | Signature |
	|:------ |:------------------- |:------------ |:------------------------------------ |:--------- |
	| LCHIJA | minecraft           | 1.12.2       | minecraft.jar                        | None      |
	| LCHIJA | mcp                 | 9.42         | minecraft.jar                        | None      |
	| LCHIJA | mixinbooter         | 10.6         | minecraft.jar                        | None      |
	| LCHIJA | placeholdername     | 1.0.0        | groovyscript-1.2.5-dev.jar           | None      |
	| LCHIJA | FML                 | **.**.**.**    | recompiled_minecraft-1.12.2.jar      | None      |
	| LCHIJA | forge               | 14.23.5.2847 | recompiled_minecraft-1.12.2.jar      | None      |
	| LCHIJA | modularui           | 2.5.1        | modularui-2.5.1-dev.jar              | None      |
	| LCHIJA | codechickenlib      | **.**.**.**    | codechickenlib-3.2.3.358-dev.jar     | None      |
	| LCHIJA | groovyscript        | 1.2.5        | groovyscript-1.2.5-dev.jar           | None      |
	| LCHIJA | jei                 | **.**.**.**   | jei_1.12.2-4.16.1.302.jar            | None      |
	| LCHIJA | theoneprobe         | 1.4.28       | top-245211-2667280-deobf.jar         | None      |
	| LCHIJA | forgelin_continuous | **.**.**.**     | Forgelin-Continuous-2.2.20.0-dev.jar | None      |
	| LCHIJA | clayium             | 0.11.0       | clayium-1.12.2-0.11.0-dev.jar        | None      |
	| LCHIJA | forgelin            | 1.8.4        | Forgelin-Continuous-2.2.20.0-dev.jar | None      |

	Loaded coremods (and transformers): 
MixinBooter (mixinbooter-10.6.jar)

ModularUI-Core (modularui-2.5.1-dev.jar)
com.cleanroommc.modularui.core.ClassTransformer
GroovyScript-Core (groovyscript-1.2.5-dev.jar)

Forgelin-Continuous (Forgelin-Continuous-2.2.20.0-dev.jar)

	GL info: ' Vendor: 'Apple' Version: '2.1 Metal - 90.5' Renderer: 'Apple M1 Pro'
	Launched Version: FML_DEV
	LWJGL: 2.9.4
	OpenGL: Apple M1 Pro GL version 2.1 Metal - 90.5, Apple
	GL Caps: Using GL 1.3 multitexturing.
Using GL 1.3 texture combiners.
Using framebuffer objects because ARB_framebuffer_object is supported and separate blending is supported.
Shaders are available because OpenGL 2.1 is supported.
VBOs are available because OpenGL 1.5 is supported.

	Using VBOs: Yes
	Is Modded: Definitely; Client brand changed to 'fml,forge'
	Type: Client (map_client.txt)
	Resource Packs: 
	Current Language: English (US)
	Profiler Position: N/A (disabled)
	CPU: 10x Apple M1 Pro
```

</details>

## 扱うもの

- UnsatisfiedLinkErrorの原因
- InterfaceのStatic MethodへのOverwrite Mixin
- Early Mixinに関する設定
- Mixin Plugin

## 扱わないもの

- Mixin基礎事項
  - 一度`ILateMixinLoader`を通じてMixinを利用したことがある前提

## より詳しい経緯

ある日、いつも通りに1.12.2でModdingしていると、RetroFuturaGradleがこう言ってきた:

```
[DEPRECATION NOTICE] Support for running Gradle using Java versions older than 21 with the RetroFuturaGradle plugin is scheduled to be removed, please upgrade your local and CI workflow Java version to 21 or newer. This does NOT mean that mod code has to use newer Java, the Gradle process itself will require Java 21 but the compiler and mod code can keep on using Java 8.
```

なので、Gradleを動かすJDKをarm64なJava 21にしたところ、run系タスクが`UnsatisfiedLinkError`でクラッシュして起動できなくなった。

# 原因

Stacktraceを見ると、`Narrator.getNarrator()`でクラッシュしているようだ。

```
Thread: Client thread
Stacktrace:
at java.lang.ClassLoader$NativeLibrary.load(Native Method)
... 略 ...
at com.mojang.text2speech.NarratorOSX.<init>(NarratorOSX.java:18)
at com.mojang.text2speech.Narrator.getNarrator(Narrator.java:22)
```

`Narrator.getNarrator()`を見ると、OSに対応する`Narrator`インスタンスを生成している。
MacOSの場合は`NarratorOSX`が返される。このクラスが存在しないネイティブライブラリを読み込もうとしてクラッシュしている。

```java
static Narrator getNarrator() {
    final String osName = System.getProperty("os.name").toLowerCase(Locale.ROOT);
    if (osName.contains("linux")) {
        setJNAPath(":");
        return new NarratorLinux();
    } else if (osName.contains("win")) {
        setJNAPath(";");
        return new NarratorWindows();
    } else if (osName.contains("mac")) {
        setJNAPath(":");
        return new NarratorOSX();
    } else {
        return new NarratorDummy();
    }
}
```

ナレーターは使っていないので、`getNarrator()`が単に`new NarratorDummy()`を返すようにMixinすれば良さそう。

# 解決策

開発(deobf)環境でのみ有効なEarly Mixinを作る。

## Mixinの作成

Interfaceのstaticメソッドという少し特殊なケースだからなのか、Injectだと動かなかったので、Overwriteする。

```java
package com.example.mixins;

@Mixin(value = Narrator.class, remap = false)
public interface MixinNarrator {

    /**
     * @author bqc0n
     * <p>
     * @reason NarratorOSX {@link com.mojang.text2speech.NarratorOSX} causes UnsatisfiedLinkError on Apple Silicon Macs
     * (Maybe native libs are not available in arm64 Java 8 JDKs?).
     * This mixin should be enabled only on deobf environment.
     *
     * @return {@link NarratorDummy}
     */
    @Overwrite(remap = false)
    static Narrator getNarrator() {
        return new NarratorDummy();
    }
}
```

## @reasonタグの追加

ここで問題が発生した。
OverwriteするならJavadocに`@reason`タグを追加せよと怒られたのだが、追加するとそんなタグはないと怒られてしまう。
このままだと`javadoc`タスクが通らない。

そこで、`build.gradle.kts`を少しいじって`@reason`タグを作る。
```kotlin
tasks.withType<Javadoc> {
    (options as StandardJavadocDocletOptions).addStringOption("tag", "reason:a:\"Reason for Overwrite:\"")
}
```

## Early Mixinの概要

バニラやforgeへのMixinには`IEarlyMixinLoader`を使う[^1]。
注意点として、`IEarlyMixinLoader`を実装するクラスは`FMLLoadingPlugin`も実装する必要がある。
つまりCoreMod化が必要である。

[^1]: MixinBooterのREADMEを参照: https://github.com/CleanroomMC/MixinBooter

## CoreMod設定

CoreMod化の手順は環境によって異なる。

まず前提として、CoreModを作る場合、コードだけではなく`META-INF/MANIFEST.MF`もいじる必要がある。

しかし、[CleanroomMC/ForgeDevEnv](https://github.com/CleanroomMC/ForgeDevEnv/)のようなモダンな1.12.2開発環境では、MANIFEST.MFは自動生成されるようになっている。
実際このテンプレートでは`gradle.properties`にCoreMod関連のパラメータが[予め用意されている](https://github.com/CleanroomMC/ForgeDevEnv/blob/782ad1bef9c6b74e188712d556283e0b3fb935c0/gradle.properties#L113-L120)。

もしそう言ったテンプレートを使っていないなら、手でMANIFEST.MFを書く必要がある。
[Minecraft Modding Wiki](https://mcmodding.jp/modding/index.php/Coremods%E3%81%AE%E5%9F%BA%E7%A4%8E)が参考になると思う。

ハマりポイントとして、`FMLCorePluginContainsFMLMod: *`のような記述をMANIFEST.MFに追加しないと通常のMod部分が読まれない。
つまり`@Mod`アノテーションが付けられたクラスが読み込まれず、CoreMod部分しか動かない。
テンプレートだと`coremod_includes_mod = true`で表現されている。

## IEarlyMixinLoaderの実装

今回はMixinを使いたいだけなので、CoreModに関する部分はnullや空配列を返す。
それ以外は`ILateMixinLoader`と同じ。

```java {27-30}
@IFMLLoadingPlugin.Name("Example-Core")
@IFMLLoadingPlugin.MCVersion(ForgeVersion.mcVersion)
public class ExampleCoremod implements IFMLLoadingPlugin, IEarlyMixinLoader {
    @Override
    public String[] getASMTransformerClass() {
        return new String[0];
    }

    @Override
    public @Nullable String getModContainerClass() {
        return null;
    }

    @Override
    public @Nullable String getSetupClass() {
        return null;
    }

    @Override
    public void injectData(Map<String, Object> data) {}

    @Override
    public @Nullable String getAccessTransformerClass() {
        return null;
    }

    @Override
    public List<String> getMixinConfigs() {
        return Collections.singletonList("mixins.core.example.json");
    }
}
```

ここまできたら、クラッシュ問題は解決しているはず。

ただしこのままだと、開発環境にとどまらず実環境(obf環境)でもMixinが適用されてしまう。
これは避けたいので、`IMixinConfigPlugin`を実装して適用する。

## IMixinConfigPluginの実装

`IMixinConfigPlugin`を実装する。
このインターフェースにも`getMixins`があるが、これはmixin config jsonに書かれた以外のMixinを追加したいときに使うものなので、今回はnullでOK。

重要なのは`shouldApplyMixin`メソッドで、ここでMixinを適用するかどうかを決定できる。
Mixinクラス名が`Narrator`で終わり、かつ開発環境でない場合にはfalseを返してMixinが適用されないようにする。

```java {10-17}
public class ExampleMixinConfigPlugin implements IMixinConfigPlugin {
    @Override
    public void onLoad(String mixinPackage) {}

    @Override
    public String getRefMapperConfig() {
        return "";
    }

    @Override
    public boolean shouldApplyMixin(String targetClassName, String mixinClassName) {
        //noinspection RedundantIfStatement | I think it is easier to understand
        if (targetClassName.endsWith("Narrator") && !FMLLaunchHandler.isDeobfuscatedEnvironment()) {
            return false;
        }
        return true;
    }

    @Override
    public void acceptTargets(Set<String> myTargets, Set<String> otherTargets) {}

    @Override
    public List<String> getMixins() {
        return null;
    }

    @Override
    public void preApply(String targetClassName, ClassNode targetClass, String mixinClassName, IMixinInfo mixinInfo) {}

    @Override
    public void postApply(String targetClassName, ClassNode targetClass, String mixinClassName, IMixinInfo mixinInfo) {}
}
```

さらに、このMixin Pluginをjsonに書く必要がある。
冗長なので追加部分以外は省略した。

```diff lang="json" title="mixins.core.example.json"
{
+  "plugin": "example.core.ExampleMixinConfigPlugin",
}
```

これで、arm64ネイティブなJDKを使えるようになった。
というか今までRosetta 2経由でx86_64を使っていたのか、通りで微妙に遅かったわけだ...
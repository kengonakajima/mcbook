# Minecraftを支える技術

「Minecraftを支える技術」を考えると、以下の2つの文脈があります。


1. MOJANG(Microsoft)が提供しているJava版および統合版のソフトウェアとサービスを活用する技術
2. Minecraftのゲーム内容を実装するための技術

1は、既存のソフトウェアに関するリバースエンジニアリングを基礎とした技術の生態系の話です。
つまりMODdingであり、Forgeやfabric, Spigot(Bukkit)であり、wiki.vgであり、PrismarineJS, Optifineなどについての話題。

また、それらをささえるJava(JDK)、ビルドシステム、C++, C#におけるネットワーク/システムプログラミングおよびLinuxなどの
インフラをどう扱うかの説明も必要です。


2については、さらに2つの話題に分かれます。

2.1. Minecraftの現在の実装がどうなっているかの話
2.2. 今からMinecraftと同様のゲームをゼロから実装するならどう作るかの話


Java版とC++版(統合版)は実装方法が全く異なります。通信プロトコルも違い、データの持ち方も異なります。
Java版のほうがリバースエンジニアリングしやすいので、現状の説明はJava版が中心になります。
Java版の実装仕様は、レガシーで複雑な仕様の塊で、お世辞にも美しいとは言えません。
むしろ、その仕様の汚さに驚くことばかりです。　そうなっている理由は、
Java版ができるだけ古いセーブデータとの互換性を保つためだと考えられます。
例えば2017年のバージョン1.12.2は、今現在でも、非常に多くのMODが立脚している、
非常に安定したバージョンでとても人気があり、多くのマルチプレイサーバーが1.12.2ベースで稼働中です。
Java版の最新版の1.17でも、1.12のデータを変換して読めるように作られています。
統合版は、最新版がリリースされたら古いものは強制的にアップデートされるため、
古いものを残して遊び続けることができないため、古くて人気のあるバージョンということは起きません。


そのため、2.2の今からMinecraftと同様のゲームをゼロから実装するならどう作るか
という話題は、さらにさらに2つの話題に分かれます。

2.2.1 Minecraftのデータや通信プロトコルと互換性のあるゲームをゼロから作る技術
2.2.2 Minecraftのデータや通信プロトコルと互換性のないゲームをゼロから作る技術


2.2.1については、現状のMOJANGソフトウェアを活用する話と重なる部分があります。
例えばPrismarineJSなどのすぐれたライブラリを使って作れば、互換性について難しい部分をかなりライブラリに負担させることが可能です。

2.2.2については、UnityやUnrealEngine、Three.jsなどいろいろなフレームワークを使って
Java版のレガシーで美しくない仕様を回避し、現代的な方法で作ることになります。
たとえばモブのアニメーションデータをハードコードせずに外部のデータとして持つとか、
全部をメインスレッドで処理せずに、複数のスレッドに分けて全体のスループットを高めようとするなどといったことです。
ただし、レッドストーン回路のように、美しく実装する定番の方法がいまだに考案されていない部分もあります。


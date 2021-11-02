# Minecraftを支える技術

スケールするマイクラサーバーをどう作るか?

マインクラフトのサーバーは、ほとんどの処理をシングルスレッドで行うため、スケールしません。
バニラ(MODなし)のJava版サーバーはほぼ全部の処理をメインスレッドで行うため、サーバーマシンのシングルスレッド性能がボトルネックになります。
そのため、10人~15人が同じサーバーにログインしてふつうにプレイしているだけで、非常に重い状態になります。
トラップタワーが2つほど稼働したり、レイドイベントが発生したら目も当てられません。

Paperなどの高度に最適化されたMODサーバーでは、最も時間がかかる地形の生成やロードの処理を別のスレッドに逃がすようになっているため、
大幅に緩和されていて、1つのサーバープロセスに対して50~150人程度がログインできるようになています。
しかし、それを可能にするために、視界範囲を短くしたり、同時に動けるモブの数を制約したり、
レッドストーンの性能を落としたりなど、バニラにはない強い制約を追加しています。これらの制約は、できるだけないに越したことはありません。
またプレイヤーやモブなどEntityの動作がすべてメインスレッドで行われるため、やはりトラップなどが稼働すると、
あっというまにラグがひどくなります。


スレッドの扱い方を根本的に変更するのは、コードの設計全体に関する問題です。
Java版のMODは、バニラサーバーをでコンパイルしてからパッチを当てるという手順で作られますが、
その手順では、サーバーの設計を根本的に変更することができないため、小手先の最適化しかできないのです。

そのため、スレッドやマルチプロセスを用いてスケールできるサーバーは、
MODの延長線上にはない可能性が高いのです。　これが、私がゼロから実装するしかないのではないか、と考える理由です。

本書の大筋としては、「スケールするマイクラサーバー」の実装をめざして設計作業をする中で、
現状のマイクラサーバーの仕様や実装を調査したり実験をし、その結果を整理していく、
という流れが基本になるでしょう。


## Java版と統合版

Java版とC++版(統合版)は実装方法が全く異なります。通信プロトコルも違い、データの持ち方も異なります。
Java版のほうがリバースエンジニアリングしやすいので、仕様の調査はJava版が中心になります。

Java版の実装仕様は、レガシーで複雑な仕様の塊で、お世辞にも美しいとは言えません。
むしろ、その仕様の汚さに驚くことばかりです。　そうなっている理由は、
Java版ができるだけ古いセーブデータとの互換性を保つためだと考えられます。

例えば2017年のバージョン1.12.2は、今現在でも、非常に多くのMODが立脚している、
非常に安定したバージョンでとても人気があり、多くのマルチプレイサーバーが1.12.2ベースで稼働中です。
Java版の最新版の1.17でも、1.12のデータを変換して読めるように作られています。

統合版は、最新版がリリースされたら古いものは強制的にアップデートされるため、
古いものを残して遊び続けることができないため、古くて人気のあるバージョンということは起きません。

Java版には　MODding のコミュニティと技術の蓄積があります。
それは、Forgeやfabric, Spigot(Bukkit)であり、wiki.vgであり、PrismarineJS, Optifineなどを含みます。
スケールするサーバーに向けては、それらの調査が当然必要です。

また、それらをささえるJava(JDK)、ビルドシステム、C++, C#におけるネットワーク/システムプログラミングおよびLinuxなどの
インフラをどう扱うかも考えなければなりません。

## マイクラのバージョンを追いかける

マイクラは頻繁にバージョンアップされるので、すべての


- サーバーツール  
- blockdata
- JSONモデルデータ
- チャンクフォーマット(anvil)　セーブデータの内容
- プロトコル(wiki.vg), PrismarineJSのソースを読む
- Bukkit(Spigot)APIを読む
- サーバーのconfig
- 認証






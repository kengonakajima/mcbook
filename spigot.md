# Spigot

Minecraftは、最も大規模に普及している、誰でもがサーバを起動できるMMORPGのサーバーであるといえる。
2009年にNotchが開発してリリースされたMinecraftは、Javaで書かれていて、WindowsやMacで動作する。これをJava版と呼ぶ。
現在はJava版以外に、非力なゲーム機やスマホで高速に動作させるためにC++で書き直された「統合版」がある。
Java版は対戦機能を有効にするとTCPのサーバとなり、ほかのJava版から接続してマルチプレイができる。
Mojangは、Java版のOpenGL描画機能以外を切り出したバージョンのjarファイルを無償提供していて、それをLinuxで起動することでマルチプレイサーバーを立ち上げることができる。
全世界では、ユーザーが起動しているサーバーが数万から数十万は存在していそうである。

Java版のマルチプレイサーバーは、持ち寄ったPCをLANに接続して数人程度で楽しむための機能であるため、インターネットに接続して大規模な人数を受け入れることには向いていない。
Java版サーバーにはたとえば詳細なパーミッション管理であったり、人数が多いときのパフォーマンスの調整であったり、ログ出力などインターネットサーバとして必要な管理機能がない。

MojangはJava版サーバーのソースコードを公開していないので、Java版サーバーにそうした機能を追加するために、
一部の開発者はjarファイルを逆コンパイルしてそのソースを変更して機能を実装するようになった。

そのようなパッチが何十、何百と増えてきて、差分が衝突することが多くなったり、逆コンパイルの方法が統一されていなかったりで再利用ができなかったり、
MojangのJava版の変更に追従するのがむずかしくなったりした。

そこで開発者たちは逆コンパイルの方法とパッチあての手順を整理してひとつのまとまりにし、継続的インテグレーションの環境を整えた。
いくつもあるが、その最新でよく使われるものがSpigotと呼ばれるものである。

Spigotは以下のことをおこなう。

まずMojangのjarファイルを逆コンパイルし、きめられた順番で200以上のパッチをあて、コンパイルし、テストを行う。
ビルドには mavenが使われている。

逆コンパイルしたらこんな感じのソースが出る。

```
    public int c(int i, int j) {                                                                                                                                     
        int k;                                                                                                                                                       
                                                                                                                                                                     
        if (i >= -30000000 && j >= -30000000 && i < 30000000 && j < 30000000) {                                                                                      
            if (this.isChunkLoaded(i >> 4, j >> 4, true)) {                                                                                                          
                k = this.getChunkAt(i >> 4, j >> 4).b(i & 15, j & 15);                                                                                               
            } else {                                                                                                                                                 
                k = 0;                                                                                                                                               
            }                                                                                                                                                        
        } else {                                                                                                                                                     
            k = this.getSeaLevel() + 1;                                                                                                                              
        }                                                                                                                                                            
```

JARファイルからシンボル名が取得できない変数についてはc,i,j,kとかd0,d1とかflag0,といったような意味のない連番の名前がつけられる。
メソッドcが何であるのか、引き数i,jが何かはわからないが、　なんとなく、i,jはx,y座標であり、ある座標の高さを取得するのだろう、
というようなことが推測される。

JARファイルはpublicになっている関数などはシンボル名が取得できるのでそれらをたよりに試行錯誤しながらパッチが作成される。
パッチは数百あり、あてる順番が決まっている。

最初のほうのパッチほど原始的になっている。
典型的なのは以下のようなコードだ。　BlockMobSpawner.javaにパッチを当てる BlockMobSpawner.patch 

```
--- a/net/minecraft/server/BlockMobSpawner.java                                                                                                                      
+++ b/net/minecraft/server/BlockMobSpawner.java                                                                                                                      
@@ -22,9 +22,19 @@     
    public int a(Random random) {                                                                                                                                    
        return 0;                                                                                                                                                    
    }                                                                                                                                                                
                                     
     public void dropNaturally(World world, BlockPosition blockposition, IBlockData iblockdata, float f, int i) {                                                    
         super.dropNaturally(world, blockposition, iblockdata, f, i);                                                                                                
+        /* CraftBukkit start - Delegate to getExpDrop                                                                                                               
         int j = 15 + world.random.nextInt(15) + world.random.nextInt(15);                                                                                           
                                                                                                                                                                     
         this.dropExperience(world, blockposition, j);                                                                                                               
+        */                                                                                                                                                          
+    }                                                                                                                                                               
+                                                                                                                                                                    
+    @Override                                                                                                                                                       
+    public int getExpDrop(World world, IBlockData iblockdata, int enchantmentLevel) {                                                                               
+        int j = 15 + world.random.nextInt(15) + world.random.nextInt(15);                                                                                           
+                                                                                                                                                                    
+        return j;                                                                                                                                                   
+        // CraftBukkit end                                                                                                                                          
     }                                                                                                                                                               
                                                                                                                                                                     
     public boolean b(IBlockData iblockdata) {                                                                                                                       
```


これはJARを逆コンパイルしたソースに対するパッチなので、
dropNaturally関数の前後にaとかbとか、まったく何かわからない関数が並んでいるし、
dropNaturally関数の引き数のfとかiなども機械的な名前になっているので意味が読み取れない。
このパッチは、dropNaturally関数にハードコードされた、経験値の計算処理を、
getExpDrop関数に切り出して、経験値の計算処理をほかのクラスから再利用できるようにしている。

逆コンパイルされたソースに大量のパッチをあてる。これは悪夢のようなことだ。

Mojangがもし dropNaturally関数にfloatかintの引き数をfの前に1個追加したら、
int iだったところがint jになり int jのところがint kになることが想像できる。
そしたらこのパッチは当てられないパッチになってしまう。。！

MojangのJava版サーバーは毎月のように更新されるので、実際いつもそのようなことが起きていて、
開発者たちは手分けをしてその修正にあたっているのである。

逆コンパイルされたソースをいじるのは前のほうにあてるパッチでできるだけ済ませ、
人間に読みやすい関数名に置き換えているので、後の方にあてるパッチではもっと人間に読みやすいソースになる。


さて、Spigotやその前身となっているCraftBukkitでは、これらの泥臭いパッチをあてて、いったい何を実現しているのだろうか？

実はSpigotは実際にゲームの内容を変更したり管理機能を追加することはほとんど行わない。
そうではなく、イベントハンドラを登録しておき、
サーバープログラムで何かが起きる前や後に呼び出して、それを呼ぶようにしているだけである。
MojangのJava版サーバーにはプラグインを導入する機能がないので、
SpigotがパッチをあてることでJavaのプラグインつまりゲーム用語でいうとMODを導入できるようにしているのである。
このプラグイン機構を通じて、膨大な種類のMOD群が作成されていて、何千万人というプレイヤーが
インターネットでの大規模マルチプレイができる環境が構築されている。
Spigotのおかげで、逆コンパイルされた難解なシンボル名ではなく、
人間が読みやすいソースでMODを書けるようになっているから、それが可能なのである。

マイクラの巨大なMODコミュニティは、
このように逆コンパイルされたソースに直接パッチを当て続けるプロジェクトの上に成立しているのである。

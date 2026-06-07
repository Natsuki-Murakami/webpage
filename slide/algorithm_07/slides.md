---
theme: default
colorSchema: light
highlighter: shiki
lineNumbers: false
transition: none
title: アルゴリズムゼミ 第07回
math: katex
layout: cover
css: ./style.css
---

# 情報とアルゴリズム
## 3-1 探索アルゴリズム

<div class="mt-20">
  <p class="text-lg">2026年6月8日</p>
  <p class="text-xl font-bold text-[#003366]">村上夏樹</p>
</div>

---

# 概要

## 探索アルゴリズム

グラフやネットワークに関する問題を解く際には,点と辺を移動しながら処理を進める場合が多い.  
グラフに対する探索アルゴリズムとは,目的の条件を満たす点や辺を見つけるために,グラフの点や辺を漏れなく訪問(探索)するためのアルゴリズム.  
グラフの探索アルゴリズムとして基本的な深さ優先探索と幅優先探索を扱う.

- **深さ優先探索(Depth First Search)**:  
現在の頂点の隣接する点が未探索な限り深く先へ進み,行き止まりに達した時点で一歩前の分岐まで戻り,別の経路の探索を再開する.
- **幅優先探索(Breadth First Search)**:  
現在の頂点に隣接するすべての未探索な頂点について進み,出発点から近い順に均等に探索範囲を拡大していく.
---

# 目次

<div class="toc-box">
  <ol>
    <li><strong>深さ優先探索</strong></li>
    <li><strong>DFS木</strong></li>
    <li><strong>スタック利用深さ優先探索</strong></li>
    <li><strong>前順序番号・後順序番号</strong></li>
    <li><strong>幅優先探索</strong></li>
    <li><strong>キュー(待ち行列)利用幅優先探索</strong></li>
    <li><strong>距離ラベル付け</strong></li>
  </ol>
</div>

<div class="alert-block">
  <div class="alert-block-title">このスライドの方針</div>
  <div class="alert-block-content">
    <p>教科書の流れに合わせて、アルゴリズム・例・定理・証明の順で整理する。</p>
    <p>この版では、各項目で後から説明や証明を書き足せる骨格を先に固める。</p>
  </div>
</div>

---

# 目次

<div class="toc-box">
  <ol>
    <li class="active"><strong>深さ優先探索</strong></li>
    <li><strong>DFS木</strong></li>
    <li><strong>スタック利用深さ優先探索</strong></li>
    <li><strong>前順序番号・後順序番号</strong></li>
    <li><strong>幅優先探索</strong></li>
    <li><strong>キュー(待ち行列)利用幅優先探索</strong></li>
    <li><strong>距離ラベル付け</strong></li>
  </ol>
</div>

---

# 深さ優先探索(Depth First Search)

<div style="color: #333; opacity: 1; font-weight: 400;">

- ある頂点を出発点として, 未探索の隣接点へ進めるだけ進んでいく.
- 出発点の隣接点が複数ある場合, その中の1点を次に訪問し, その他の点の探索は後回しにする.
- 現在いる頂点から先に進めなくなったら辿ってきた点を戻り,未探索の点が隣接していれば訪問する.
- 出発点の近くに未探索の頂点が残っていても, より深い頂点の探索が優先される. **➡深さ優先**

<SimpleDfsPreview />

</div>

---

# 深さ優先探索
- $f(v)$: 頂点 $v$ が探索済みなら 1, 未探索なら 0.
- $p(v)$: 頂点 $v$ を見つける直前の頂点(親).($p(v)$から隣接する頂点$v$を発見)　未探索または探索初期点なら $p(v)=v$.
- $X$: 点の発見に使った辺の集合.　　　　　　　　　　　$Y$: まだ参照していない辺の集合.
- **入力**: グラフ $G$.　($|V(G)|=n,|E(G)|=m$)　　　**出力**: 探索済み変数 $f(u)$, 辺集合 $X$.

<div class="beamer-block">
  <div class="beamer-block-title">アルゴリズム 3-1</div>
  <div class="beamer-block-content">

Step 0(初期化)　　　　: $f(v)=0, p(v)=v$ （$\forall v \in V(G)$）とし, $X=\emptyset, Y=E(G)$ とする.  
Step 1(初期点設定)　　: 点を $V(G)$ から任意に選び, $s=r$ とする.　(sは現在地点)  
Step 2(探索開始)　　　: $f(s)=1$ とする.  
Step 3(終了判定)　　　: $s$ に接続する $Y$ の辺が存在せず, $p(s)=s$ ならば終了する.  
Step 4(後退)　　　　　: $s$ に接続する $Y$ の辺が存在せず, $p(s)\neq s$ ならば, $s=p(s)$ として, Step 3 に戻る.  
Step 5(未探索辺の選択): $s$ に接続する $Y$ の辺を任意に選ぶ. 選んだ辺を $(s,t)$ とし, これを $Y$ から除く.  
Step 6(探索済み判定)　: $f(t)=1$ ならば, Step 3 に戻る.  
Step 7(未探索点発見)　: $p(t)=s$ とし, 辺 $(s,t)$ を $X$ に加える.  
Step 8(前進)　　　　　: $s=t$ として, Step 2 に戻る.

  </div>
</div>

---

# DFS 実行例

<DfsDemo />

---

# アルゴリズム3-1の計算量

<div class="beamer-block">
  <div class="beamer-block-title">定理 3-2</div>
  <div class="beamer-block-content">

アルゴリズム 3-1 は $O(n+m)$ 時間で終了する.

  </div>
</div>

<div style="height: 0.6rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">証明の概略</div>
  <div class="beamer-block-content">

step0の初期化では, 各頂点 $v$ に対して $f(v)=0, p(v)=v$ を設定し, $X=\emptyset, Y=E(G)$ とするので $O(n+m)$ .  
終了時の出力では, $f(v)$ と探索済み辺集合 $X$ を出力する. ここで $|X|\leqq m$ であるから, これも $O(n+m)$ .  
また,その他のstepはいずれも $O(1)$ で実行できる.<span style="color: #cc0000;">(※1)</span>  
Step 0, 1 と終了時の出力はそれぞれ 1 回<span style="color: #cc0000;">(※2)</span>, Step 2 は高々$n$回.(各頂点に対して高々 1 回) 
各頂点 $s$ で, 接続する辺を高々 $\deg_G(s)$ 本調べ, 最後に 1 回後退するので, Step 3, 4 はそれぞれ高々 $\deg_G(s)+1$ 回.  
Step 5, 6, 7, 8 はそれぞれ高々 $m$ 回.  
ここで, $\sum_{v \in V(G)} \deg_G(v)=2m$ であるから(1章で既出), 以上を合わせて全体の計算量は $O(n+m)$ である.  
  </div>
</div>

<span style="color: #cc0000;">(※1)</span> Step 3, 4 の「$s$ に接続する $Y$ の辺の存在判定」や, Step 5 の「辺を $Y$ から除く」を愚直に行うと $O(1)$ ではできない.  
これらを $O(1)$ で行うには, 双方向リストなどのデータ構造の工夫が必要.  
<span style="color: #cc0000;">(※2)</span> $G$ が連結でないときは, すべての頂点が探索されるわけではないため, Step 0, 1, 出力は 1 回で済む.  


---

# 目次

<div class="toc-box">
  <ol>
    <li><strong>深さ優先探索</strong></li>
    <li class="active"><strong>DFS木</strong></li>
    <li><strong>スタック利用深さ優先探索</strong></li>
    <li><strong>前順序番号・後順序番号</strong></li>
    <li><strong>幅優先探索</strong></li>
    <li><strong>キュー(待ち行列)利用幅優先探索</strong></li>
    <li><strong>距離ラベル付け</strong></li>
  </ol>
</div>

---

# 部分グラフ $T_D$

- 出力された辺集合$X$と, $X$の端点の集合から成る, 部分グラフ $T_D = G[X]$ を定義する.
- 辺集合$X$には未探索の頂点へ進むときにのみ辺が追加されるので,$T_D$ はDFSの過程で「新しい頂点を発見した辺」だけを集めたものとなる.

<div class="beamer-block">
  <div class="beamer-block-title">定理 3-2</div>
  <div class="beamer-block-content">

アルゴリズム 3-1 をグラフ $G$ に適用したとき得られる部分グラフ $T_D (= G[X])$ は木である.

  </div>
</div>

<div style="height: 0.6rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">証明</div>
  <div class="beamer-block-content">

始点 $r$ 以外の各探索済み頂点 $v$ に対して, $v \neq p(v)$ となる親 $p(v)$ が一意に定まり, 辺 $(p(v), v)$ が $X$ に含まれる.  
任意の探索済み頂点から親をたどると必ず始点 $r$ に到達する. したがって, 任意の 2 頂点 $u, v$ の間に始点 $r$ を中継点とするウォークが構成できるため, $T_D$ は連結である.  
また, 辺 $(s, t)$ が $X$ に加わるのは $t$ が未探索(既存の$T_D$と連結でない)の場合に場合に限られるため, すでに探索済みの頂点を結ぶ辺は $X$ に含まれず, $T_D$ に閉路は存在しない.(連結グラフ$T_D$で $E(T_D)=V(T_D)-1$ より木であるとしてもよい)    
連結でサイクルを持たないグラフは木であるため, $T_D$ は木である.

  </div>
</div>


---

# DFS木の定義

- 定理 3-2 により, 得られた部分グラフ $T_D$ は木であることが保証された.
この $T_D$ を **DFS木 (DFS tree)** と呼ぶ.


- グラフ $G$ が2点以上から成る連結グラフであるとき, DFS木 $T_D$ は $G$ の **全域木** となる.  
($G$ が連結ならば,未訪問の頂点がある限り探索可能な辺が必ず存在し, DFS の終了条件を満たさない.ゆえに$T_D$は$Gの$全頂点を含む)

---

# 連結性判定の計算量①

<div class="beamer-block">
  <div class="beamer-block-title">例題 3-1</div>
  <div class="beamer-block-content">

アルゴリズム 3-1 を用いて, グラフが連結かどうかを線形時間で判定できることを示せ.

  </div>
</div>

<div style="height: 0.6rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">例題 3-1 の解答の概略</div>
  <div class="beamer-block-content">

グラフ $G$ が連結 $\iff$ アルゴリズム 3-1 終了時に$\forall v \in V(G)$について$f(v)=1$ ・・・$(*)$　を示す.

① $(\Rightarrow)$ の証明
- 終了時に $f(v)=0$ となる頂点があれば, 始点から到達不可能なので非連結（対偶）.

② $(\Leftarrow)$ の証明
- 全頂点が訪問済みならば $T_D$ は全域木となり, 全域木を持つグラフは連結である.

$(*)$より,アルゴリズム 3-1 の実行 $(n+m)$ と $f(v)$ の確認 $(n)$ を合わせ, 連結性を線形時間 $O(n+m)$ で判定できる.

  </div>
</div>

---

# 目次
<div class="toc-box">
  <ol>
    <li><strong>深さ優先探索</strong></li>
    <li><strong>DFS木</strong></li>
    <li class="active"><strong>スタック利用深さ優先探索</strong></li>
    <li><strong>前順序番号・後順序番号</strong></li>
    <li><strong>幅優先探索</strong></li>
    <li><strong>キュー(待ち行列)利用幅優先探索</strong></li>
    <li><strong>距離ラベル付け</strong></li>
  </ol>
</div>

---

# スタック利用深さ優先探索

- 「未探索辺が見つからないときにどこに戻るか」を親頂点 $p(v)$ として管理する代わりに, 次に調べる辺の候補を **スタック** に積んで管理する.
- スタックは後から追加したものを先に取り出す.要素の追加と取り出しは定数時間で可能.**(後入れ先出し)**  
変数や入力, 出力はアルゴリズム 3-1 とほぼ同様. (相違点は親 $p(v)$ の管理が不要になり, stack が登場する.)

<div class="beamer-block">
  <div class="beamer-block-title">アルゴリズム 3-2</div>
  <div class="beamer-block-content">

Step 0 (初期化)　　　: $f(v) = 0\ (\forall v \in V(G))$ とし，$X = \emptyset$，$Y = E(G)$ とする．  
Step 1 (初期点設定)　: 点 $r$ を $V(G)$ から任意に選び，$s = r$ とする．  
Step 2 (探索開始)　　: $f(s) = 1$ とする．  
Step 3 (辺の追加)　　: $s$ に接続する $Y$ の辺をすべて $stack$ に追加し，追加した辺を $Y$ から削除する．  
Step 4 (終了判定)　　: $stack$ が空ならば終了する．  
Step 5 (辺の取り出し)　:$stack$ の先頭の辺を取り出す．取り出した辺を $(u, t)$ とする．  
Step 6 (探索済み判定): $f(t) = 1$ ならば，Step 4 に戻る．  
Step 7 (未探索点発見): 辺 $(u,t)$ を $X$ に加える.  
Step 8 (前進)　　　　: $s = t$ として，Step 2 に戻る．  

  </div>
</div>

---

# stack利用深さ優先探索 実行例

<StackDfsDemo />

---

# 定理 アルゴリズム3-2の計算量

## スタック利用深さ優先探索の計算量

<div class="beamer-block">
  <div class="beamer-block-title">定理 3-3</div>
  <div class="beamer-block-content">

アルゴリズム 3-2 は $O(n+m)$ 時間で終了する。

  </div>
</div>

<div style="height: 0.6rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">証明の概略</div>
  <div class="beamer-block-content">

step0の初期化では, 各頂点 $v$ に対して $f(v)=0$ を設定し, $X=\emptyset, Y=E(G)$ とするので $O(n+m)$ .  
終了時の出力では, $f(v)$ と探索済み辺集合 $X$ を出力する. ここで $|X|\leqq m$ であるから, これも $O(n+m)$ .  
Step 1, 2, 4, 5, 6, 7, 8 はいずれも $O(1)$ で実行できる.　　
Step 3 は, 頂点 $s$ に接続する辺をすべて $stack$ に追加するので $O(\deg_G(s))$ である.  
Step 0, 1 と終了時の出力はそれぞれ 1 回, Step 2は高々 $n$ 回(各頂点に対して高々 1 回).  
したがって, Step 3 全体の計算量は $\sum_{v \in V(G)} \deg_G(v)=2m$ より $O(m)$. (各辺が高々 1 回だけ $stack$ に追加される.)  
Step 4 は高々 $m+1$ 回, Step 5, 6, 7, 8 はそれぞれ高々 $m$ 回.  
以上を合わせて全体の計算量は $O(n+m)$ である.  
  </div>
</div>

---

# 目次

<div class="toc-box">
  <ol>
    <li><strong>深さ優先探索</strong></li>
    <li><strong>DFS木</strong></li>
    <li><strong>スタック利用深さ優先探索</strong></li>
    <li class="active"><strong>前順序番号・後順序番号</strong></li>
    <li><strong>幅優先探索</strong></li>
    <li><strong>キュー(待ち行列)利用幅優先探索</strong></li>
    <li><strong>距離ラベル付け</strong></li>
  </ol>
</div>

---

# 前順序番号と後順序番号

<div style="color: #333; opacity: 1; margin-top: 1.2em;">
深さ優先探索を利用すると, 各頂点に「いつ発見されたか」と「いつ探索を終えたか」を表す2種類の番号を与えることができる.
</div>

- **前順序番号 $k(v)$**:  
点 $v$ が初めて発見された順序 (行きがけの順). 

- **後順序番号 $k'(v)$**:  
点 $v$ に隣接するすべての辺の探索が終了した順序 (帰りがけの順).


　➡ アルゴリズム 3-1 を修正することで, これらの順序番号を付けることができる.

---

# 順序番号付けアルゴリズム

- **入力**: グラフ $G$ (ただし，$|V(G)|=n$, $|E(G)|=m$)  
- **出力**: 前順序番号 $k(v)$, 後順序番号 $k'(v)\ (\forall v \in V(G))$ 

<div class="beamer-block">
  <div class="beamer-block-title">アルゴリズム 3-3</div>
  <div class="beamer-block-content"> 

Step 0 (初期化):　$p(v) = v$,　$f(v),k(v),k'(v) = 0\ (\forall v \in V(G))$ 　$i,j = 1$,　$X = \emptyset$, $Y = E(G)$とする．  
Step 1 (初期点設定)　: 点 $r$ を $V(G)$ から任意に選び，$s = r$ とする．  
Step 2 (前順序付与)　: $f(s) = 1$, $k(s) = i$, $i = i + 1$ とする．  
Step 3 (終了判定)　　: $s$ に接続する $Y$ の辺が存在せず $p(s) = s$ ならば，$k'(s) = j$ として，終了する．  
Step 4 (後退と後順序): $s$ に接続する $Y$ の辺が存在せず $p(s) \neq s$ ならば，$k'(s) = j$, $j = j + 1$, $s = p(s)$ として，Step 3 に戻る．  
Step 5 (辺の選択)　　: $s$ に接続する $Y$ の辺を任意に選ぶ．選んだ辺を $(s, t)$ とし，$(s, t)$ を $Y$ から除く．  
Step 6 (探索済み判定): $f(t) = 1$ ならば，Step 3 に戻る．  
Step 7 (未探索点発見): $p(t) = s$, $(s, t)$ を $X$ に加える．  
Step 8 (前進)　　　　: $s = t$ として，Step 2 に戻る．  

  </div>
</div>

※ アルゴリズム 3-3 で定義される写像 $k$ を**前順序番号付け**, 写像 $k'$ を**後順序番号付け**という.


---

# 前順序番号と後順序番号 実行例

<PrePostDfsDemo />
---

# 定理 アルゴリズム3-3の計算量

<div style="color: #333; opacity: 1; margin-top: 1.2em;">
各頂点に前順序番号や後順序番号を付けるためにアルゴリズム 3-1 に操作を追加したが,
アルゴリズム 3-3 の各 step の時間計算量はアルゴリズム 3-1 と同じであり,
各 step の実行回数もアルゴリズム 3-1 と同じである.  
</div>

したがって, アルゴリズム 3-3 の計算量も $O(n+m)$ .


<div class="beamer-block">
  <div class="beamer-block-title">定理 3-4</div>
  <div class="beamer-block-content">

アルゴリズム 3-3 は $O(n+m)$ 時間で終了する。

  </div>
</div>

<div style="height: 0.6rem;"></div>



---

# 目次

<div class="toc-box">
  <ol>
    <li><strong>深さ優先探索</strong></li>
    <li><strong>DFS木</strong></li>
    <li><strong>スタック利用深さ優先探索</strong></li>
    <li><strong>前順序番号・後順序番号</strong></li>
    <li class="active"><strong>幅優先探索</strong></li>
    <li><strong>キュー(待ち行列)利用幅優先探索</strong></li>
    <li><strong>距離ラベル付け</strong></li>
  </ol>
</div>

---

# 幅優先探索(Breadth First Search)

- ある頂点を出発点として, その点に隣接している未探索の隣接点をすべて探索する.  
- 出発点の隣接点が複数ある場合, それらをすべて探索し終えてから, その先の点の探索へと進む. 
- より深い（遠い）頂点へ進む前に, 出発点から近い（浅い）頂点の探索がくまなく優先される.**➡幅優先**

<SimpleBfsPreview />

---

# 幅優先探索

- **入力**: グラフ $G$. ($|V(G)|=n,|E(G)|=m$)　　　
- **出力**: 変数 $f(v)\ (\forall v \in V(G))$

<div class="beamer-block">
  <div class="beamer-block-title">アルゴリズム3-4</div>
  <div class="beamer-block-content">

Step 0(初期化)　　　　: $f(v)=0\ (\forall v \in V(G))$ とする.  
Step 1(初期点設定)　　: 点 $r$ を $V(G)$ から任意に選び, $f(r)=1$ とする.   
Step 2(隣接点列挙)　　: $f(v)=1$ である点 $v$ に隣接し, $f(v')=0$ である点 $v'$ をすべて求める. そのような点が存在しなければ終了する.  
Step 3(一斉訪問)　　　: Step 2 で求めたすべての点 $v'$ に対して $f(v')=1$ とし, Step 2 に戻る.

  </div>
</div>

---

# 幅優先探索 実行例

<BfsWaveDemo />

---

# 幅優先探索の計算量

<div style="height: 0.02rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">アルゴリズム 3-4 の計算量</div>
  <div class="beamer-block-content">

**Step 0 (初期化)**
  * 1回あたり $O(n)$ $\times$ 全体で 1 回実行 $\rightarrow$ **$O(n)$**.

**Step 1 (初期点設定)**
  * 1回あたり $O(1)$ $\times$ 全体で 1 回実行 $\rightarrow$ **$O(1)$**.

**Step 2 (隣接点列挙)**
  * 1回実行するたびに全頂点 $v$ が $f(v)=1$ か調べるため **$O(n)$** .　　　 (←$\Omega(n)$でもある)
  * さらに, 隣接点 $v'$ を調べ変更する操作の総和は, $\sum_{v \in V(G)} \deg_G(v)=2m$ より $O(m)$.
  * これらを高々 $n-1$ 回繰り返すため $\rightarrow$ 全体で <span style="color: #cc0000;">**$O(n(n+m))$**</span>.
  
**Step 3 (一斉訪問(フラグの一斉更新))**
  * 各頂点に対して $O(1)$ の操作を高々 1 回実行する $\rightarrow$ 全体で **$O(n)$**.

  </div>
</div>
---

# 目次

<div class="toc-box">
  <ol>
    <li><strong>深さ優先探索</strong></li>
    <li><strong>DFS木</strong></li>
    <li><strong>スタック利用深さ優先探索</strong></li>
    <li><strong>前順序番号・後順序番号</strong></li>
    <li><strong>幅優先探索</strong></li>
    <li class="active"><strong>キュー(待ち行列)利用幅優先探索</strong></li>
    <li><strong>距離ラベル付け</strong></li>
  </ol>
</div>

---

# キュー(待ち行列)利用幅優先探索

- 操作を必要とする点を毎回全探索する代わりに, 次に調べる辺の候補を **キュー(待ち行列)** に記憶して管理する.
- キュー(待ち行列)は先に追加したものを先に取り出す. 要素の追加と取り出しは定数時間で可能. **(先入れ先出し)** 
- アルゴリズム 3-2 (スタック利用DFS) のstackをqueueに置き換えただけで, 全体の構造は同一である.

**入力**: グラフ $G$ (ただし，$|V(G)|=n$, $|E(G)|=m$ とする)　　**出力**: 変数 $f(v)\ (\forall v \in V(G))$, 辺の集合 $X$

<div class="beamer-block">
  <div class="beamer-block-title">アルゴリズム 3-5</div>
  <div class="beamer-block-content">

Step 0 (初期化)　　　: $f(v) = 0\ (\forall v \in V(G))$ とし，$X = \emptyset$，$Y = E(G)$ とする．  
Step 1 (初期点設定)　: 点 $r$ を $V(G)$ から任意に選び，$s = r$ とする．  
Step 2 (探索開始)　　: $f(s) = 1$ とする．  
Step 3 (辺の追加)　　: $s$ に接続する $Y$ の辺をすべてqueueに追加し，追加した辺を $Y$ から削除する   
Step 4 (終了判定)　　: queueが空ならば終了する．  
Step 5 (辺の取り出し)　: queueの先頭の辺を取り出す．取り出した辺を $(u, t)$ とする．  
Step 6 (探索済み判定): $f(t) = 1$ ならば，Step 4 に戻る．  
Step 7 (未探索点発見): $(u, t)$ を $X$ に加える．  
Step 8 (前進)　　　　: $s = t$ として，Step 2 に戻る  

  </div>
</div>

---

# queue利用幅優先探索 実行例

<QueueBfsDemo />

---

# 定理 3-5

<div style="height: 0.6rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">定理 3-5</div>
  <div class="beamer-block-content">

アルゴリズム 3-5 は $O(n+m)$ 時間で終了する。

  </div>
</div>

アルゴリズム 3-5 は, アルゴリズム 3-2 の $stack$ を $queue$ に置き換え,探索順序が変化しただけで、$stack$や$queue$に出し入れする回数は変わらない.   
したがって, 各 step の時間計算量も実行回数もアルゴリズム 3-2 と同じであり, 全体の計算量も $O(n+m)$ である.


---

# 目次

<div class="toc-box">
  <ol>
    <li><strong>深さ優先探索</strong></li>
    <li><strong>DFS木</strong></li>
    <li><strong>スタック利用深さ優先探索</strong></li>
    <li><strong>前順序番号・後順序番号</strong></li>
    <li><strong>幅優先探索</strong></li>
    <li><strong>キュー(待ち行列)利用幅優先探索</strong></li>
    <li class="active"><strong>距離ラベル付け</strong></li>
  </ol>
</div>

---

# 距離ラベル付けアルゴリズム

- アルゴリズム 3-3 (順序番号付け) で深さ優先探索を用いて順序を付与したのに対し, 以下では 幅優先探索を用いる.
- 幅優先探索の「近い順に探索が広がる」性質を利用して, ある出発点からの 距離と等しいラベル $\lambda(v)$を各点に付けることができる.

**入力**: グラフ $G$ (ただし，$|V(G)|=n$, $|E(G)|=m$ とする)　　**出力**: ラベル $\lambda(v)\ (\forall v \in V(G))$

<div class="beamer-block">
  <div class="beamer-block-title">アルゴリズム 3-6</div>
  <div class="beamer-block-content">

Step 0 (初期化)　　　: $\lambda(v) = \infty\ (\forall v \in V(G))$ とし, $i = 0$ とする.  
Step 1 (初期点設定)　: 点 $r$ を $V(G)$ から任意に選び, $\lambda(r) = 0$ と変更する.  
Step 2 (探索と終了判定): $\lambda(v) = i$ である点 $v$ に隣接し, $\lambda(v') = \infty$ である点 $v'$ をすべて求める. そのような点が存在しなければ, 終了する.  
Step 3 (ラベルの更新)　: Step 2 で求めたすべての点 $v'$ に対して $\lambda(v') = i + 1$ と変更する. $i = i + 1$ として, Step 2 に戻る.  

  </div>
</div>

---

# 距離ラベル付け 実行例

<DistanceLabelDemo />

---

# 定理 3-6

<div style="height: 0.6rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">定理 3-6</div>
  <div class="beamer-block-content">

アルゴリズム 3-6 は $O(n+m)$ 時間で終了する。

  </div>
</div>

※ アルゴリズム3-5におけるフラグ$f(v)$とその更新が、ラベル $\lambda(v)$ とその更新に置き換わっただけで, その他の探索の構造はアルゴリズム 3-5 と完全に同一である.  
よって,アルゴリズム 3-6 の計算量も $O(n+m)$ である.

---

# ラベルと最短距離の一致

<div style="height: 0.2rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">定理 3-7</div>
  <div class="beamer-block-content">

アルゴリズム 3-6 をグラフ $G$ に適用すると, 各頂点 $v \in V(G)$ について
$\lambda(v)=\operatorname{dis}_G(r,v)$が成り立つ.

  </div>
</div>

<div style="height: 0.6rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">証明の概略(定理3-7)</div>
  <div class="beamer-block-content">

**ラベルが無限大 ($\lambda(v) = \infty$) の場合:**  

  始点 $r$ から $v$ への路$(r,v)$が存在すると仮定すると, $\lambda(x)=i$（有限値), $\lambda(y)=\infty$を満たす辺$(x,y)\in E(G)$が存在.  

  しかし,アルゴリズム3-6のステップ2では$\lambda(y)=i+1$と更新されるため矛盾.   

  路$(r,v)$は存在せず, 定義より $\operatorname{dis}_G(r,v)$=$\infty$ となる.

  </div>
</div>

---

# ラベルと最短距離の一致

<div style="height: 0.2rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">証明の概略(定理3-7)</div>
  <div class="beamer-block-content">


**ラベルが有限の値 ($\lambda(v) = k$) の場合:**
  1. **$\lambda(v) \ge \operatorname{dis}_G(r,v)$ :**  
  
     ラベルが $k$ の点から, $k-1, k-2, \dots, 0$ (始点 $r$) となる隣接点を順に遡ることで, 長さ $k$ の路が存在が示される.  
     したがって,　$\lambda(v) = k \ge \operatorname{dis}_G(r,v)$

  2. **距離 $d$ に関する数学的帰納法:**

     距離 $\operatorname{dis}_G(r,v) = d$ について帰納法を用いる.  
     距離 $0$ の点 (始点 $r$) について,$\lambda(r) = \operatorname{dis}_G(r,r)=0$.  
     $\operatorname{dis}_G(r,u) < d$ を満たす任意の点 $u \in V(G)$ について, $\lambda(u) = \operatorname{dis}_G(r,u)$と仮定する.   
     距離 $d$ の頂点 $v$ への最短$(r,v)$路を考え, 頂点 $v$ の1つ手前の頂点を $w$ とすると, $\operatorname{dis}_G(r,w) = d-1$ であり,   
     帰納法の仮定より$\lambda(w) = d-1$.  
     前半の議論より $\lambda(v) \ge d$ であるから, アルゴリズム3-6で$i=d-1$ の時点では $\lambda(v) = \infty$ のままである.  
     ステップ2・3の操作により, 頂点 $w$ に隣接する頂点 $v$ について $\lambda(v) = d$ と正しく更新され, $\lambda(v) = d= \operatorname{dis}_G(r,v)$.

  </div>
</div>
---

# 連結性判定の計算量②

<div style="height: 0.4rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">例題3-2</div>
  <div class="beamer-block-content">

アルゴリズム 3-6 を用いてグラフが連結かどうかを線形時間で判定できることを示せ.

  </div>
</div>

<div style="height: 0.6rem;"></div>

<div class="beamer-block">
  <div class="beamer-block-title">解答</div>
  <div class="beamer-block-content">

定理 3-7 より,  $r$ から $u$ への路が存在すれば, アルゴリズム3-6により各頂点 $u$ に有限ラベル $\lambda(u)$ が付与される. 

したがって, グラフ $G$ が連結であることは, アルゴリズム3-6により $G$ に含まれるすべての頂点に対して有限ラベルが付与されることと同値である.  

定理 3-6 よりアルゴリズム3-6の計算量は $O(n+m)$ であり, 各ラベルを調べる計算量は$O(n)$.

以上より, 線形時間で連結性を判定できる.

  </div>
</div>

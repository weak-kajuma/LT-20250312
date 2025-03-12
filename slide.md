---
marp: true
theme: default
paginate: true
header: "ローカルLLMに向き合うLT大会 2025-03-12"
style: |
  /* 全体の背景はシンプルなホワイト */
  section {
    background-color: #ffffff;
    color: #333333;
    font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
    padding: 1em;  /* 前回よりも余裕のある余白 */
    margin: 0;
  }
  
  /* タイトルのスタイル */
  h1 {
    font-family: 'Helvetica Neue', sans-serif;
    font-weight: 700;
    color: #1a1a1a;
    text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.1);
    border-bottom: 2px solid #1a1a1a;
    padding-bottom: 0.3em;
    margin-bottom: 0.5em;
  }
  
  /* サブタイトル、見出しの調整 */
  h2, h3, h4 {
    font-family: 'Arial', sans-serif;
    color: #444444;
    margin-top: 0.5em;
    margin-bottom: 0.5em;
  }
  
  /* ページ番号やフッター */
  footer {
    font-size: 0.8em;
    color: #888888;
  }
  
  /* リストのスタイル */
  ul {
    font-size: 1.2em;
    line-height: 1.5em;
    list-style-type: disc;
    margin-left: 1.2em;
    margin-top: 0.3em;
    margin-bottom: 0.3em;
  }
  
  /* ブロック引用 */
  blockquote {
    border-left: 4px solid #1a1a1a;
    padding-left: 10px;
    font-style: italic;
    color: #555555;
    margin: 0.5em 0;
  }
  
  /* 画像がスライドをはみ出さないようにするクラス */
  .fit-image {
    display: block;
    margin: 0 auto;
    max-width: 90%;
    max-height: 70vh;
    object-fit: contain;
  }

  /* 2カラム表示用のコンテナ */
  .two-column {
    display: flex;
    align-items: flex-start;
    gap: 1rem;
  }
  .two-column .text-col {
    flex: 1;
  }
  .two-column .img-col {
    flex: 1;
    text-align: center;
  }
markdown-it-mermaid: true
---

<!-- スライド1：タイトル -->
# 「Patch-Level TrainingでLLMの継続事前学習を効率化できるか？」
#### *「LLMの継続事前学習でPatch-Level Trainingを試し、その有用性を検証してみた話」*

---

<!-- スライド2：背景 & 課題 -->
# 「LLMの事前学習、計算コストの問題」
- 従来の事前学習は、LLMの規模拡大に伴い計算コストが非常に高い
- 継続事前学習は、既存モデルの知識更新や新データへの適応を<br>低コストで実現する対策として注目
- 例として **Llama 3.3 Swallow** が挙げられる
- 継続事前学習によってコスト削減はされているが、<br>依然としてコストが高い

---

<!-- スライド3：Patch-Level Trainingの概要 -->
# 「Patch-Level Trainingの概要」

<div class="two-column">
  <div class="text-col">
    <ul>
      <li><strong>Patch-Level Training (PLT)</strong><br>複数トークンを1つの<b>パッチ</b>にまとめて学習する手法</li>
      <li>通常のLLMは1トークンずつ<br>学習（Token-Level Training）</li>
      <li>同じ計算量でより多くの<br>データを処理可能</li>
      <li>情報損失を抑えつつ、<br>学習コスト削減を狙う</li>
    </ul>
  </div>
  <div class="img-col">
    <img src="./model.png" alt="PLTの概念図" class="fit-image" />
  </div>
</div>

---

<!-- スライド4：Patch-Level Trainingのコスト削減 -->
# 「Patch-Level Trainingのコスト削減」

<div class="two-column">
  <div class="text-col">
    <ul>
      <li><strong>理論式</strong>：<br/>コスト削減 ＝ λ/K + (1−λ)</li>
      <li>λ: PLT割合, K: パッチサイズ</li>
      <li>λを大きくするとパッチ部分が増加<br>→ コスト削減率は上がるが<br>情報損失のリスクも増す</li>
    </ul>
  </div>
  <div class="img-col">
    <img src="./intro.png" alt="PLTのコスト削減図" class="fit-image" />
  </div>
</div>

---

<!-- スライド5：Patch-Level Trainingの学習フロー -->
# 「Patch-Level Trainingの学習フロー」

<div class="two-column">
  <div class="text-col">
    <ul>
      <li><strong>from scratch</strong> (青線) と<br><strong>from patch-level</strong> (オレンジ線) の比較例</li>
      <li><strong>TLT開始時の損失</strong>が高いが、その後の学習によって徐々に差は縮小し、同等以上の性能を出す</li>
    </ul>
  </div>
  <div class="img-col">
    <img src="./loss.png" alt="PLTの学習フロー" class="fit-image" />
  </div>
</div>

---
# Patch-Level Training Figure 1
<img src="./model.png" alt="PLTの概念図" class="fit-image" />

---
# Patch-Level Training Figure 2
<img src="./loss.png" alt="PLTの学習フロー" class="fit-image" />

---
<!-- スライド6：実験設定 -->
# 「実験設定」
- **元モデル:** meta-llama/Llama-3.2-1B  
- **比較:**  
  - PLTを使用したモデル  
  - Token-Level TrainingのみのBaselineモデル  
- **設定:**  
  - patch_size = 4  
  - λ = 1/2（途中でλ=2/3も検証可能）  
- **データ:** kajuma/ABEJA-CC-JA-edu（約20Bトークン）

---

<!-- スライド7：評価指標と実験フロー -->
# 「評価指標と実験フロー」
- **評価指標:**
  - 学習損失  
  - pfgen-benchmark 平均スコア  
- **実験フロー:**
<img src="flow.png" width=90%/>
---

<!-- スライド8：学習損失 同じstep数 vs 同じ計算量 -->
# 「実験結果：学習損失」
- **同じstep数での比較**: Token-Level Training移行後、Baselineとの差は徐々に縮まるが最終的には追いつかず
- **同じ計算量での比較**: PLTのスコアがBaselineに近づく、あるいは上回る傾向が見られる

<div class="two-column">
  <div class="img-col">
    <img src="./loss_same_steps.png" alt="学習損失(同じstep数)" class="fit-image" />
  </div>
  <div class="img-col">
    <img src="./loss_same_calcuration.png" alt="学習損失(同じ計算量)" class="fit-image" />
  </div>
</div>

---

<!-- スライド9：pfgen-benchmark 同じstep数 vs 同じ計算量 -->
# 「実験結果：pfgen-benchmark」
- **同じstep数での比較**: Token-Level Training移行後、Baselineとの差は徐々に縮まるものの最終的には追いつかず
- **同じ計算量での比較**: PLTのスコアがBaselineに近づく、あるいは上回る傾向が見られる

<div class="two-column">
  <div class="img-col">
    <img src="./pfgen_score_same_steps.png" alt="pfgen-benchmark(同じstep数)" class="fit-image" />
  </div>
  <div class="img-col">
    <img src="./pfgen_score_same_calcuration.png" alt="pfgen-benchmark(同じ計算量)" class="fit-image" />
  </div>
</div>

---

<!-- スライド10：考察 -->
# 「なぜこの結果になったのか？」
- PLTでパッチの生成を学習することによるモデルの重みが<br>破壊されている可能性
- Token-Level Trainingへの移行で損失は回復するが、学習を延長しても完全には追いつかない  
- λを1/4など小さくすると改善の余地はあるが、学習コスト削減効果は低下するため慎重に検討が必要
- 下流タスクでの性能比較が必要

---

<!-- スライド11：まとめ -->
# 「結論：PLTは現時点では最適解ではない」
- PLTは学習コスト削減のメリットがあるが…<br>λを下げなければ精度が低い<br>λを下げると、学習コスト削減効果も薄れる<br>→ PLT選択のメリットは十分に得られない
- **結論:現時点で継続事前学習において積極的にPLTを選択すべきでない**

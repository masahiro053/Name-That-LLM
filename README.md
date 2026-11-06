### Score Board (Update)
| Date | Method / Model | CV | Public LB | Private LB |
| :--- | :--- | :--- | :--- | :--- |
| 2026-11-05 | TF-IDF + Logistic Regression (Starter) | - | 0.317 | - |
| 2026-11-06 | LightGBM + Stylometry + TF-IDF (Official Data) | 0.1522 | - | - |
| 2026-11-06 | LightGBM + Stylometry + TF-IDF + Open Pool (47k rows) | ~0.270 | (Submit待機中) | - |

---
## 📝 Daily Log


### 2026-11-04 (Day 1)

**Done**

*   コンペの概要把握と、ディスカッション・上位Notebookの傾向調査。

*   上位陣のブレイクスルー手法を特定。

    *   **Prompt Reconstruction & Bipartite Matching:** テストデータを同じプロンプトのグループ（18件）にクラスタリングし、ハンガリアン法で「1モデル1出力」の制約を課して重複なく割り当てるハック手法が主流。

    *   **Shared-prompt Gaussian:** トピック効果（話題）を差し引き、純粋な文体（Stylometry）のみを抽出する数理モデル。

*   計算資源（GPU）の制約を考慮し、大規模LLMの学習ではなく「特徴量エンジニアリング ＋ 軽量モデル（LightGBM） ＋ ハンガリアン法による最適割り当て」を主軸とする方針を決定。

*   公式のStarter Notebook（TF-IDF + Logistic Regression）をフォークし、インターネットOFFでの `submission.parquet` 生成とSubmitパイプラインの開通を確認。



**Todo**

*   Starter Notebookをベースに、Pandasを用いたテキスト前処理・特徴量抽出（句読点頻度、機能語の割合、平均文長など）の実装。

*   LightGBMを用いたベースラインモデルの構築とCV（Cross Validation）の枠組み作成。

*   ハンガリアン法（`scipy.optimize.linear_sum_assignment`）のロジック検証。



**Memo**

*   いきなり複雑な上位Notebookをマージするとパスや依存関係のエラーで死ぬため、まずは自分専用のシンプルな推論パイプラインを確立することに徹する。

*   学習データが約2,700件と少ないため、中盤以降はDiscussionで共有される外部データセット（APIで生成されたもの）の取り込みが必須になりそう。



**References (Today's URLs)**

*   [Starter Notebook for Name That LLM Competition](https://www.kaggle.com/code/martynaplomecka/starter-notebook-for-name-that-llm-competition) (公式の安全なベースライン。まずはこれを開通させる)

*   [Name That LLM: Tuning our TF-IDF Baseline](https://www.kaggle.com/code/barun2104/name-that-llm-tuning-our-tf-idf-baseline) (TF-IDFと軽量モデルの構成参考)

*   [0.667 | Pairwise Verifier + Matched External Rows](https://www.kaggle.com/code/zbigniewkirchner/0-667-pairwise-verifier-matched-external-rows/notebook) (マッチング手法のハック参考①)

*   [Name That LLM Solution](https://www.kaggle.com/code/mohammedafreedpasha/name-that-llm-solution) (ハンガリアン法による最適割り当ての参考②)

*   [🕵️ The Usual Suspects](https://www.kaggle.com/code/leoprovorov/the-usual-suspects) (Gaussian Headによるトピック影響除去の参考)



### 2026-11-05 (Day 2)

**Done**

*   公式のStarter Notebook（TF-IDF）をフォークし、インターネットOFF環境での推論・Submitを実行。

*   無事に提出パイプラインが開通し、Public LB Score: `0.317` を獲得（ベースラインとして正常稼働を確認）。

*   今後の全体戦略とロードマップの策定。

    *   **方針:** 試行回数と解釈性を重視し、当面は「テキスト特徴量エンジニアリング ＋ LightGBM」でスコアを伸ばす。

    *   **理由:** LLMへの丸投げはGPUを浪費して試行回数が落ちる上、ブラックボックス化して「どの特徴が効いているか（各モデルの文体の癖）」が分からなくなるため。

    *   LightGBMの `feature_importance` で各LLMの文体（Stylometry）を特定し、最終的にはハンガリアン法による最適割り当てでスコアのジャンプアップを狙う。

*   データ不足懸念の払拭。有志による以下の強力な公開データセットの存在を確認。

    *   `Name That LLM | Prism Feature Atlas` (468種の文体特徴量抽出済みデータ)

    *   `Name That LLM: same-prompt generations v1` (1つのプロンプトに複数モデルが答える構造を再現した12,500件データ)

    *   `Name That LLM: Extra API Passages` (見分けが難しいClaude専用の約5,000件データ)



**Todo (Next Actions)**

*   Pandasを用いたテキスト前処理（文字数・単語数・句読点カウント）と、公開データセット（Prism Feature Atlasなど）の特徴量を結合するデータパイプラインの構築。

*   特徴量とTF-IDFを合わせたLightGBMのローカル学習環境構築と、特徴量重要度の可視化。

*   Claude系モデルを分離して推論する「2段階分類（Claude Gate手法）」のアーキテクチャ検討。



**Memo**

*   パイプラインが開通し、自分の得意な「特徴量を作って精度を見る」ループに入れるようになった。

*   重いモデルを回してデータを生成するお金のかかる作業はKaggleの有志がやってくれている。自分は彼らが提供してくれた特徴量をどうLightGBMに食わせ、どう重複なく割り当てるか（パズルを解くこと）に集中すれば勝負できる。


### 2026-11-06 (Day 3)
**Done**
*   **ベースライン構築:** ブラックボックスな深層学習を避け、解釈性が高く軽量なLightGBMベースの学習・推論パイプライン（CV: StratifiedKFold 5-fold）を実装。
*   **特徴量エンジニアリングと有効性証明:** TF-IDF（max_features=3000）に加え、Pandasで構造的特徴量（Stylometry: 文字数、単語長、大文字割合、句読点カウント等）を抽出。
    *   Feature Importanceを確認した結果、上位7つを `avg_word_length`, `uppercase_ratio`, `comma_count` などの手作り特徴量が完全に独占。話題（TF-IDF）ではなく、文体の癖を学習させるアプローチの正しさが証明された。
*   **外部データによる増強とパイプライン修正:**
    *   有志公開データのうち、クリーニング済みの `Name That LLM Open Pool` を選定。
    *   クラス不均衡を防ぐため、18モデル×各2,500件をランダムサンプリングし、公式訓練データと縦結合（計約47,000行へスケールアップ）。
    *   **【重要】** 外部データ（行）追加時にTF-IDFの語彙（列数）ズレによる次元エラーを防ぐため、「生テキストの段階で `concat` してから全体に対して `fit_transform` をかける」安全なパイプラインへコードをリファクタリングした。
    *   データ増強後のCVスコアが `0.1522` から `0.27` 台へ約2倍の劇的なジャンプアップを達成。
*   **後処理（最適化アルゴリズム）:**
    *   「1プロンプトにつき18モデルが必ず1回ずつ回答する」というコンペ制約を利用し、`scipy.optimize.linear_sum_assignment`（ハンガリアン法）を用いた最適割り当てコードを実装。
*   公開（Public）してVoteを集められるよう、全体のパイプラインをMarkdown解説付きの綺麗なNotebookフォーマットにまとめ直した。

**Todo**
*   完成した「データ増強＋仮のハンガリアン法」のNotebookをKaggleにSubmitし、Public LBスコアを確認する。
*   **本番テストデータのシャッフル検知（偵察）:**
    *   スコアが上昇（0.4〜0.5台）した場合: テストデータはプロンプトごとに綺麗に並んでいるため、ハンガリアン法がそのまま刺さる。
    *   スコアが崩壊した場合: テストデータはシャッフルされているため、Sentence-BERT等を用いた「同じプロンプトの18行」を特定するクラスタリング（Prompt Reconstruction）ロジックの実装へ移行する。

**Memo**
*   Kaggleのデータセットは頻繁に更新されるため、公開コードの列（今回は `match_weight`）が存在しないエラーに遭遇したが、柔軟なサンプリング分岐で回避できた。
*   ハンガリアン法は計算量が心配だったが、プロンプトごとの極小行列（18×18）に対する処理なので数ミリ秒で爆速で終わることが判明。Kaggleの9時間制限のボトルネックにはならない。
*   逆に、データ量が4.7万件かつTF-IDFの3000列に増えたため、LightGBMのCVには10〜20分かかるようになったが、推論側は一瞬なので問題なし。

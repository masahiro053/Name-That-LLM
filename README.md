# Name That LLM - Kaggle Diary

## 🏆 Competition Overview
*   **Task:** 40〜250単語の短いテキストから、それを生成した18種類の最先端LLMを特定する「著者特定（Authorship attribution）」タスク。
*   **Metric:** Balanced Accuracy（マクロ平均正解率）。
*   **Constraints:** Code Competition (CPU/GPU 9時間以内, インターネット接続不可)。
*   **Timeline:**
    *   Team Merger Deadline: 2027-01-06 23:59 UTC
    *   Final Submission Deadline: 2027-01-13 23:59 UTC

## 📊 Score Board
| Date | Method / Model | CV | Public LB | Private LB |
| :--- | :--- | :--- | :--- | :--- |
| 2026-11-05 | TF-IDF + Logistic Regression (Starter) | - | 0.317 | - |
| 2026-11-06 | TF-IDF + Stylometry + LightGBM (Official Data) | 0.1522 | - | - |
| 2026-11-08 | TF-IDF + Stylometry + LightGBM + Open Pool (47k rows) + Dummy Hungarian | ~0.270 | 0.264 | - |
| 2026-11-16 | Fingerprints (81 Dense Features) + LightGBM + Prompt Reconstruction | - | (Commit中) | - |

---
## 📝 Daily Log

### 2026-11-04
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
*   [Starter Notebook for Name That LLM Competition](https://www.kaggle.com/code/martynaplomecka/starter-notebook-for-name-that-llm-competition)
*   [Name That LLM: Tuning our TF-IDF Baseline](https://www.kaggle.com/code/barun2104/name-that-llm-tuning-our-tf-idf-baseline)
*   [0.667 | Pairwise Verifier + Matched External Rows](https://www.kaggle.com/code/zbigniewkirchner/0-667-pairwise-verifier-matched-external-rows/notebook)
*   [Name That LLM Solution](https://www.kaggle.com/code/mohammedafreedpasha/name-that-llm-solution)
*   [🕵️ The Usual Suspects](https://www.kaggle.com/code/leoprovorov/the-usual-suspects)

### 2026-11-05
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

### 2026-11-06
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

### 2026-11-08 
**Done**
*   **Submit結果の分析:**
    *   外部データ増強版LightGBM ＋ ダミーのハンガリアン法（`// 18`でグループ化）をSubmitした結果、Public LBは `0.264` へ下落。
    *   **結論:** 本番の隠しテストデータはプロンプトごとに並んでおらず、**完全シャッフルされていることが確定**。無関係な行同士でハンガリアン法をかけたため、正しい予測が強制的に書き換えられ自滅した。
*   **CV低下の根本原因究明（次元の呪い）:**
    *   ハンガリアン法以前に、LightGBMモデル単体のCVスコアが初期のLogistic Regression（0.317）を下回っている違和感を調査。
    *   **原因:** 「TF-IDF（数千列の0が多い疎行列）」と「決定木（LightGBM）」の致命的なアーキテクチャミスマッチ。決定木は1回の分岐で1列しか評価できないため、数千のスパースな単語列から意味を見出せず、アンダーフィットを起こしていた。
*   **過去コンペ（DAIGT等）と現在の上位解法リサーチ:**
    *   **SOTA:** DeBERTa-v3-large のFine-tuning（LB 0.587超え）や軽量LLM（Qwen/Llama等）のLoRA。
    *   **機械学習の最適解:** TF-IDFを活かすなら、全列の重みを同時計算できる線形モデル（LinearSVC, Logistic Regression, Ridge）が必須。
    *   **後処理の必須ハック:** シャッフル対策として、Sentence-BERT等でテキストを密ベクトル化し、コサイン類似度や階層的クラスタリングを用いて「同じプロンプトの18行」を自力で復元（Prompt Reconstruction）した上でハンガリアン法を適用するのが上位陣の定石。

**Todo (Next Actions)**
*   アプローチのピボット（以下3案から方針を決定し実装する）。
    1.  **原点回帰 (ML Baseline):** LightGBMを破棄し、増強した4.7万件のデータとTF-IDFを `LinearSVC` または `RidgeClassifier` に学習させる。
    2.  **Dense + LightGBM:** TF-IDFをやめ、Sentence-BERTで768次元の密ベクトルに変換してからLightGBMに投入する。
    3.  **NLPの王道 (SOTA):** 腹を括って `DeBERTa-v3-large` のFine-tuning環境を構築する。

**Memo**
*   自身の直感（CVが以前のロジスティック回帰より下がっている違和感）から、モデルの構造的欠陥に気づけたのは大きな収穫。
*   NLPコンペにおいて「テキストはTF-IDFにしてとりあえずLightGBM」という脳死のパイプラインは通用しない。データの性質（疎か密か）とモデルの得意領域を合致させる基礎が重要だと再認識した。

### 2026-11-16
**Done**
*   **上位解法（0.701 | Read the Fingerprints）の構造分析:**
    *   前回直面した「TF-IDF（疎行列）＋LightGBM」の相性の悪さを完璧に克服する最適解を学んだ。
    *   単語の出現頻度（話題）を捨てるアプローチへ転換。Victor Zhong氏の「Style Fingerprint Tables」を用い、句読点の癖（カーリーアポストロフィ等）や文の構造など、LLM特有の「81列の密な数値データ（Dense Features）」を抽出する手法の威力を確認。
    *   列数が絞られた密なデータになることで、決定木（LightGBM / XGBoost）が本来の非線形パターン認識能力をフルに発揮できる設計になっていることを理解した。
*   **シャッフル対策（プロンプト復元）の学習:**
    *   Sentence-BERT等を用いてテストデータの文章をベクトル化し、類似度から「同じプロンプトの18行」を自力でクラスタリング。その上でハンガリアン法を適用するという上位陣の定石（ハック）を把握。
*   **Kaggle環境の仕様とSubmitトラブルの解決:**
    *   Code Competitionにおけるデータセット（`/kaggle/input/`）やオフラインモデルの読み込み構造を理解。
    *   Submit時の `submission.parquet` 未作成エラーの原因を特定。見えている `test.csv` は3行のダミーであり、採点されるには「Save & Run All (Commit)」でサーバー側の一括実行（隠しデータ4000行へのすり替えと推論）を完了させる必要がある仕組みを完全に腹に落とした。
*   **再現実装の実行:**
    *   現在、0.701のNotebookをフォークし、GPU (T4 x2) 環境で「Save & Run All (Commit)」を実行中。

**Todo (Next Actions)**
*   現在走っているCommitが「Successful」になり次第、出力された `submission.parquet` を提出し、手元でPublic LB 0.701付近が再現できるか確認する。
*   再現が確認できたら、このFingerprint特徴量のパイプラインに対して、私たちが過去に用意した「Open Pool（4.7万件の外部ペアデータ）」を合体させ、さらなるスコアアップ（0.75〜）を狙う。

**Memo**
*   LLMを見破る鍵は「何を書いたか（トピック）」ではなく、「どう書いたか（文体・癖）」に尽きる。これをいかに機械学習モデルが喜ぶ「密な数値」に落とし込むかがこのコンペの勝負の分かれ目。
*   KaggleのCommitとInteractive Sessionの違いで少し沼ったが、これでSubmit周りのシステムエラーには今後一切動じずに済む。大きな学びだった。
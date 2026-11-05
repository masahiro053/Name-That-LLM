# Name That LLM - Kaggle Diary

## 🏆 Competition Overview
*   **Task:** 40〜250単語の短いテキストから、それを生成した18種類の最先端LLM（GPT-5/6, Claude 4/5, Gemini 3系など）のどれかを特定する「著者特定（Authorship attribution）」タスク。
*   **Metric:** Balanced Accuracy（マクロ平均正解率）。クラス間の不均衡があるが、18の全モデルが平等な重みで評価される。
*   **Constraints:** Code Competition (CPU/GPU 9時間以内, インターネット接続不可)。
*   **Timeline:**
    *   Team Merger Deadline: 2027-01-06 23:59 UTC
    *   Final Submission Deadline: 2027-01-13 23:59 UTC

## 📊 Score Board
| Date | Method / Model | CV | Public LB | Private LB |
| :--- | :--- | :--- | :--- | :--- |
| - | - | - | - | - |

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
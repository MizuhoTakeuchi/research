# プレス成形におけるML基盤サロゲートモデルの文献調査

プレス成形・板金加工分野において、**機械学習（ML）ベースのサロゲートモデル**は高コストなFEM/FEAシミュレーションの代替として急速に発展し、金型形状最適化・プロセスパラメータ最適化・欠陥予測の各領域で不可欠な技術となっている。2015年以前は応答曲面法（RSM）やクリギングが主流であったが、2019年以降はCNN・U-Net・グラフニューラルネットワーク等の深層学習アーキテクチャへの転換が進み、全場（フルフィールド）予測やリアルタイム推論が可能となった。本調査では2015〜2025年の約60件の代表的論文を体系的に整理し、ML手法の分類、応用領域、最新トレンドと将来展望を包括的に報告する。

---

## 1. サロゲートモデルの背景と基本概念

板金プレス成形（深絞り、曲げ、ブランキング、鍛造、ホットスタンプ、ハイドロフォーミング等）のプロセス設計では、有限要素法（FEM）シミュレーションが広く活用されている。しかし、1回のFEA解析に数時間〜数日を要し、最適化には数百〜数千回の反復計算が必要となるため、**計算コストが実用的な設計探索の最大のボトルネック**となっている。

サロゲートモデル（代替モデル、メタモデル、エミュレータとも呼ばれる）は、FEAの入出力関係を少数のシミュレーション結果から学習し、**数秒〜数分で予測を返す近似モデル**を構築する技術である。Wang et al.（2017）のレビューが示すように、この分野では①サンプリング戦略（ラテン超方格法等）、②サロゲート構築手法（RSM、クリギング、RBF、ANN等）、③最適化アルゴリズムとの統合（GA、NSGA-II、ベイズ最適化等）の3要素が研究の柱を形成してきた。

近年の進展として、Cao et al.（2024, CIRP Annals）は290件の文献を網羅したCIRP基調論文で、AI/MLの金属成形への応用を①プロセスシミュレーション、②プロセス設計・最適化、③インプロセス制御、④品質保証の4分野に体系化している。この包括的レビューは、物理情報付きML・デジタルツイン・データ効率的学習を今後の主要方向性として特定した。

---

## 2. ML手法の分類と特性比較

プレス成形サロゲートモデルに使用されるML手法は、古典的手法から最先端の深層学習まで多岐にわたる。各手法の特性を以下に整理する。

### 2.1 古典的サロゲートモデル

**応答曲面法（RSM）** は最も歴史が長く、低次元問題（設計変数10以下）に対して依然として有効である。Cui et al.（2020）はRSMとNSGA-IIを組み合わせ、ホットスタンプB-ピラーの6パラメータ同時最適化で板厚減少率誤差2.1〜3.1%を達成した。しかし、高次元・高非線形問題への適用は困難である。

**クリギング（ガウス過程回帰）** は予測値だけでなく**不確実性の定量化**が可能な点が最大の優位性である。Palmieri et al.（2021）はクリギングメタモデルで深絞りの材料ばらつきと摩擦変動を考慮したロバスト最適化を実現した。Lee et al.（2024）はガウス過程回帰を用いた深絞りブランク形状設計手法を提案している。Feng et al.（2018）はクリギング＋多目的人工蜂コロニーアルゴリズム（MOABC）で、スプリングバック・しわ・破断の3目的VBHF最適化を達成した。

**RBF（放射基底関数）ネットワーク** は、Kitayama et al.（2013, 2015, 2017）による一連の研究で、逐次近似最適化（SAO）フレームワークと組み合わせた可変ブランクホルダ力（VBHF）軌跡の最適化に広く使用されている。RBFベースのSAOは実験検証も行われ、AC サーボプレスでの実証が報告されている。

### 2.2 ニューラルネットワーク系手法

**ANN/MLP（多層パーセプトロン）** は最も広く使用されるML手法である。El Mrabti et al.（2021）はFEM-ANN-PSO戦略で深絞りスプリングバックの最適化を実現し、Trzepieciński & Lemu（2020）はGA最適化MLPでV曲げスプリングバック予測で相関係数0.962〜0.972を達成した。Spathopoulos & Stavroulakis（2020）はベイズ正則化逆伝播ANNでS-レール成形のスプリングバック角予測にR=0.9653を報告している。

**DNN（深層ニューラルネットワーク）** は複雑な多目的問題に適用が拡大している。Park et al.（2024）はDNN-GA-MCSフレームワークで破断18.89%、しわ13.59%、スプリングバック14.26%の同時改善を達成した。

**CNN（畳み込みニューラルネットワーク）/ U-Net** は、2022年以降の最も重要な技術革新である。**Zhou et al.（2022, ASME）はRes-SE-U-Netアーキテクチャにより、スカラーベースMLP手法を精度・汎化性・ロバスト性・情報量の全てで凌駕することを実証**した。この画像ベース手法は空間・位置情報を保持するため、板厚分布やひずみ場等のフルフィールド予測が可能となる。Attar et al.（2021, 2023）はCNN基盤サロゲートでホットスタンプ（HFQ工程）のリアルタイム成形性評価プラットフォームを開発し、FEAとほぼ区別不能な予測精度を実現した。

**RNN/GRU** は、Mozaffar et al.（2019, PNAS）の画期的論文により、**経路依存性塑性構成則の直接学習**が可能であることが示された。Gorji et al.（2020）はGRUベースRNNでバウシンガー効果・永久軟化・潜在硬化の捕捉に成功している。

**GNN（グラフニューラルネットワーク）** は最新の進展で、Zhou et al.（2025）はマルチレベルグラフベースサロゲートモデルで、非構造メッシュデータを直接処理し、実際のB-ピラー部品の複雑な局所パターンにおいて既存手法を大幅に上回る精度を実現した。

### 2.3 物理情報付きアプローチ

**理論誘導型DNN** として、Liu et al.（2021, IEEE/CAA）はSwiftの法則を正則化項としてDNN学習に組み込み、**少量データでの汎化性能がデータ駆動型DNNやSVRを大幅に上回る**ことを示した。板金曲げのスプリングバック補正において、ワークピース形状からプロセスパラメータへの直接マッピングを実現している。

**PINN（物理情報付きニューラルネットワーク）** は金属成形への適用がまだ初期段階にある。2024年の比較研究では、ANNが変位・ひずみ場を予測し、PINNが構成則準拠の応力場を予測する2段階サロゲートが提案されている。ただし、大変形・接触・摩擦を含む成形のPDEは現行PINNフレームワークでは困難が多い。

**EINN（専門家情報付きニューラルネットワーク）** は、ESAFORM 2025でPINNの実用的代替として提案された。解析的PDEが利用できない製造プロセスにおいて、数値的に導出された物理制約を学習に組み込み、わずか15件のFEA＋9件の実験データからの外挿精度でXGBoostやDNNを上回った。

### 2.4 アンサンブル・勾配ブースティング手法

**勾配ブースティング回帰（GBR、LightGBM、CatBoost、XGBoost）** が近年台頭している。Cai et al.（2022）は22の温度ゾーンを持つアルミホットスタンプの板厚予測で、勾配ブースティングがRF・SVR・ANNを上回る最高精度を達成した。GWO-CatBoostモデル（2024）はマルチポイントストレッチフォーミングのスプリングバック予測でR²=0.9293を記録している。**ランダムフォレスト**は成形限界線図（FLD）予測（Samad et al., 2025でR²最高）やディレス成形の時間依存サロゲート（2026）に適用されている。

### ML手法の比較表

| 手法 | 長所 | 短所 | 最適な応用場面 |
|------|------|------|--------------|
| RSM | 理解容易、低次元で高精度 | 高次元・高非線形に不向き | 設計変数10以下の最適化 |
| クリギング/GP | 不確実性定量化可能、少量データ対応 | 高次元で計算コスト増大 | ロバスト最適化、ベイズ最適化 |
| RBF | 汎用性高い、SAOと好相性 | スカラー出力のみ | VBHF軌跡最適化 |
| ANN/MLP | 柔軟、高速推論、実績豊富 | ブラックボックス、外挿に弱い | スカラー予測（角度、力等） |
| CNN/U-Net | 空間情報保持、フルフィールド予測 | 構造化データ必要、大量学習データ | 板厚分布・ひずみ場予測 |
| RNN/GRU | 経路依存性の自然な表現 | 長経路で次元の呪い | 構成則モデリング |
| GNN | 非構造メッシュ対応、形状柔軟性 | 新しい手法、成熟度低い | 複雑形状のサロゲート |
| PINN/理論誘導型 | 少量データで高汎化、物理整合性 | 学習困難、収束問題 | 小規模データ、構成則 |
| 勾配ブースティング | 高精度、特徴量重要度の可視化 | フルフィールド予測に不向き | 高次元パラメータ空間 |
| 転移学習 | Sim-to-Real橋渡し、データ効率 | ドメイン整合が重要 | 実環境展開、少量実験データ |

---

## 3. 主要応用領域

### 3.1 金型形状最適化とスプリングバック補正

金型形状最適化は設計変数の高次元性から最も困難な課題の一つである。**Attar et al.（2023, 2024）は署名距離場（SDF）ベースの形状生成器とCNN製造性サロゲートを組み合わせた深層学習プラットフォーム**を開発し、勾配ベース最適化により最大板厚減少率を45%から10%制約以下に低減した。さらに、ジェネレータネットワークと評価ネットワークを用いたU-チャンネルのスプリングバック補正用金型形状の自動最適化を実現している。

スプリングバック予測・補正は金型設計最適化の中核を成す。Bici et al.（2019）は双対応答曲面法と形状関数最適化を統合し、AHSS B-ピラーの材料特性変動6%を考慮したロバスト金型補正を達成した。Wang et al.（2021）はクリギング＋多目的GAでTRIP780高強度鋼のパレート最適プロセスパラメータ決定後、金型面補正を行う2段階アプローチを提案した。Zhong et al.（2024/2025）はアテンション機構付きU-Net CNNで非対称チャンネルの連成ねじり-角度スプリングバックのフルフィールド偏差予測を実現し、金型面補正精度を大幅に改善した。

Chen et al.（2025）は自動車BIW構造のスプリングバック予測に線形回帰・決定木・勾配ブースティング・SVR・FCNNを比較し、ANNが部品レベル・組立レベル両方で最も有望な性能を示すことを報告した。

**日本国内の研究**として、Okishi et al.（2025, JSAI全国大会）は神戸製鋼・大阪大学・関西大学の共同研究で、**PointNeXtによる3D点群深層学習でプレス金型形状→成形品形状のサロゲートモデル**を構築し、平均誤差約0.3mm（実用精度）、推論時間はFEMの数時間に対し数分を達成している。

### 3.2 プロセスパラメータ最適化

プロセスパラメータ最適化は最も研究蓄積が豊富な領域であり、ブランクホルダ力（BHF）、パンチ速度、摩擦係数、温度、ドロービード拘束力等を対象とする。

**可変ブランクホルダ力（VBHF）最適化**は単一で最も集中的に研究されたテーマである。Feng et al.（2019）はSVR＋信頼領域戦略でVBHF軌跡最適化のロバスト手法を開発した。Xie et al.（2021）は制限ボルツマンマシン（RBM）＋逆伝播NNのハイブリッドサロゲートと改良MOPSO（密集度オペレータ付き）を組み合わせ、ダブルC部品のしわ低減を達成した。Zhao et al.（2024）はクリギング＋改良QO-Jayaアルゴリズムで、深絞りVBHFの多目的最適化にJayaアルゴリズムを初適用した。

**ホットスタンプ最適化**は自動車軽量化ニーズに牽引されて急速に成長している。Cui et al.（2020）はRSM＋NSGA-IIで分割冷却の6パラメータ同時最適化を実現し、急冷ゾーンでマルテンサイト組織（引張強度1540MPa）を達成した。Ma et al.（2021）はGA-BP NNで7075アルミ合金のホットスタンプ最適パラメータ（成形速度50mm/s、摩擦係数0.1、BHF 5kN）を予測し、最大板厚減少率9.37%（FEA検証で9.81%、誤差5%）を得た。Cai et al.（2022）は22温度ゾーンの高次元パラメータ空間で勾配ブースティングが最高精度を達成した。

**多目的最適化**は2019年以降の標準的アプローチとなっている。NSGA-IIが最も広く使用される最適化アルゴリズムであり、灰色関係分析・TOPSIS・セットペア分析等の意思決定手法と組み合わされる。Research Square（2025）のプレプリントでは、適応型スパースオートエンコーダ＋GP＋BPNNのハイブリッドモデルとINSGA-IIの組み合わせで、**単一サロゲート比15〜40%の精度改善**と欠陥20%低減を報告している。

**ベイズ最適化の台頭**も注目に値する。Pfeifer et al.（2025, arXiv）は深層学習初期推定＋ガウス過程潜在変数モデル（GPLVM）によるベイズ最適化で、専門家介入を大幅に削減するAI支援ワークフローを提案した。能動学習との組み合わせにより、シミュレーション評価回数の効率的な削減が実現されている。

### 3.3 欠陥予測と成形性最適化

**しわ・き裂予測**では、Yi et al.（2025）がデジタルツインフレームワーク内でSVM・ランダムフォレスト・勾配ブースティング・ANNを組み合わせ、ドロービード位置・BHFの関数としてき裂発生分類精度100%、しわ予測MSE 0.141を達成した。Dib et al.（2020）はMLP・SVM・決定木等の単一・アンサンブル分類器を比較し、MLPが最高F値（90%以上）を示すことを報告している。

**板厚減少・破断予測**では、El Mrabti et al.（2022）がRSM・RBF・クリギング・ANNの4手法をAHSS深絞りの6パラメータ問題で系統比較し、ANNとクリギングが最高精度を示した。画像ベースCNNサロゲート（Zhou et al., 2022; Attar et al., 2021）によるフルフィールド板厚分布予測は、スカラー最大板厚減少率のみの予測から大幅な情報量向上を実現している。

**成形限界線図（FLD）予測**はML応用の重要な分野である。Chheda et al.（2019）はSVR（第1段階）＋勾配ブースティング（第2段階）の2段階モデルで、合金組成・加工条件からFLDを予測した。Samad et al.（2025）はランダムフォレストが予ひずみ・温度条件下のFLD予測でSVRを上回ることを示した。Thamm et al.（2023）はCNNで引張試験データから成形限界を決定する手法を提案し、高コストな中島試験の代替を目指している。

**GE-EINN（勾配強化専門家情報付きNN）**（2025, J. Manufacturing Processes）は、物理制約をバックプロパゲーションに組み込み、**わずか15件のFEA＋9件の実験データから冷間・温間スタンプの成形深さ限界予測**で訓練領域外の外挿精度を大幅改善した。

### 3.4 デジタルツインとサロゲート支援設計フレームワーク

**デジタルツイン（DT）** はプレス成形のリアルタイム品質管理の中核技術として急速に発展している。Gan et al.（2022, ASME）は金型摩耗を監視し、摩擦係数をPSO-DEで実時間更新するDTモデルで、初期モデルから**61.97%の精度改善**と最大板厚減少率14.35%低減を達成した。Karkaria et al.（2025）はPODベースKoopmanオペレータ＋モデル予測制御（MPC）で適応型DTフレームワークを提案し、オンラインRLS更新で変化する材料状態に対応した。Modad et al.（2024, J. Intelligent Manufacturing）は「真のDT」（双方向・リアルタイム）と単純なシミュレーションモデルの区別を明確にし、Industry 5.0におけるゼロ欠陥製造へのDTの役割を体系化した。

**転移学習**は、シミュレーションから実環境への橋渡し（Sim-to-Real）において重要な進展を見せている。**Iriondo et al.（2024, J. Manufacturing Systems）はABAQUS低忠実度シミュレーションで訓練したDNNサロゲートを少数の実パイロットプラントデータでファインチューニング**し、キャリブレーション段階や高忠実度シミュレーションを不要とする手法を提案した。Wang et al.（2024）はグラフNN＋スパースコーディングによるプロセス間転移学習で、データ豊富なロータリードローベンディングからデータ不足のフリーベンディングへの知識転移を実現した。

**強化学習（RL）** もプロセス制御に新たな可能性を開いている。Liu et al.（2020, Procedia Manufacturing）はDQNで自由曲面スタンプの最適ハンマー打撃シーケンスを職人経験なしに学習した。2022年のMaterials論文では、ホットスタンプのサイクルタイム最適化をMDPとして定式化し、RLエージェントが品質制約を満たしながらスループットを最大化した。

---

## 4. 最新トレンドと将来展望

### スカラーからフルフィールドへのパラダイムシフト

**最も重要なトレンドは、スカラーベースサロゲート（最大板厚減少率等の単一値予測）から画像/グラフベースのフルフィールドサロゲート（板厚分布・ひずみ場全体の予測）への移行**である。Zhou et al.（2022）がこのパラダイムを確立し、Attar et al.（2023）の形状最適化プラットフォーム、Zhou et al.（2025）のマルチレベルGNN、Baum et al.（2025, U. Stuttgart）の3D点群深層学習サロゲートへと発展している。日本でもOkishi et al.（2025）がPointNeXtで同様のアプローチを金型設計に適用している。

### ハイブリッドサロゲートの優位性確立

単一ML手法ではなく、複数技術の組み合わせが標準となりつつある。スパースオートエンコーダ＋GP＋BPNN（2025）、RBM＋BPNN（Xie et al., 2021）、POD＋RBF（Dang et al., 2017）等のハイブリッドモデルは、単一モデル比15〜40%の精度改善を達成している。LightGBM＋DNNアンサンブル（2024）はR²=0.951でWebプラットフォーム上のリアルタイム予測に実装されている。

### 物理情報付きMLの重要性増大

製造プロセスでは訓練データが高コストで少量であるため、物理知識の統合が特に重要である。理論誘導型DNN（Liu et al., 2021）、EINN（2025）、PINNの応用は、**データ効率の大幅改善と外挿精度の向上**をもたらす。ただし、大変形・接触・摩擦を含むプレス成形の完全なPINN化はPDE定式化の困難さから今後の課題である。

### 産業実装への加速

米国ではUSAMP/ORNL（2024）がAutoFormシミュレーションデータからKerasベースNNサロゲートを構築し、自動車ドアパネルのドローイン予測MAPE 1.8%を達成した。日本ではSCSK/pSevenプラットフォームによる熱間鍛造の産業適用事例や、デンソーソーケンによるサロゲートモデルAI実装のガイドラインが報告されている。IDDRG 2025では標準化ベンチマークデータセット（Heinzelmann et al.）が公開され、ML手法間の公正な比較評価が可能になった。

### 今後の課題と方向性

- **基盤モデル（Foundation Model）の不在**：トランスフォーマーベースや大規模事前学習モデルの成形分野への適用はまだ報告されていない
- **マルチフィジックスサロゲート**：ホットスタンプ等の熱・力学・冶金学的予測を統一するサロゲートの開発が必要
- **大規模産業検証**：学術ベンチマーク問題から実生産レベルの複雑性への移行が不十分
- **説明可能なAI（XAI）の統合**：Mazur et al.（2025）の反実仮想説明によるサロゲート汎化性分析が先駆的
- **GANの活用拡大**：Manufacturing Letters（2024）のISFプロセスパラメータ→ミクロ組織画像予測GANが示すように、データ拡張や逆設計への応用が有望

---

## 5. 代表的論文一覧表

| No. | 著者 | 題目 | 雑誌/会議 | 年 | ML手法 | 応用分野 | 主要成果 |
|-----|------|------|----------|-----|--------|---------|---------|
| 1 | Cao, Bambach, Merklein et al. | Artificial intelligence in metal forming | CIRP Annals, 73(2) | 2024 | 包括レビュー（ANN, CNN, RNN, PINN, RL等） | 金属成形全般 | 290文献の体系的レビュー、4分野のAI応用ロードマップ |
| 2 | Wang, Ye, Chen, Li | Sheet metal forming optimization by using surrogate modeling techniques | Chinese J. Mech. Eng., 30 | 2017 | RSM, Kriging, RBF, ANN, SVR レビュー | サロゲート最適化全般 | サンプリング戦略・次元の呪い・ロバスト設計の課題整理 |
| 3 | Zhou, Xu, Nie, Li | Image-based ML methods for surrogate models of stamp forming | ASME J. Manuf. Sci. Eng., 144 | 2022 | Res-SE-U-Net (画像ベースCNN) | プレス成形フルフィールド予測 | 画像ベースがスカラーベースMLPを全指標で凌駕 |
| 4 | Attar, Foster, Li | Rapid feasibility assessment via hot stamping deep learning | J. Manufacturing Processes, 68 | 2021 | CNN基盤サロゲート | HFQ工程リアルタイム成形性評価 | FEAとほぼ同等の精度でリアルタイム予測 |
| 5 | Attar, Zhu, Li | Deep learning enabled tool compensation for shape distortion | ICTP 2023, Springer | 2024 | DL（生成器＋評価器NN） | 金型スプリングバック補正 | NN基盤の自動金型形状最適化 |
| 6 | Attar, Foster, Li | Deep learning platform for sheet stamping geometry optimisation | Eng. Appl. AI, 123 | 2023 | SDF形状生成器＋CNN | 製造性制約下の形状最適化 | 最大板厚減少率45%→10%以下に低減 |
| 7 | Zhou et al. | Multi-level graph-based surrogate for sheet forming | Adv. Eng. Informatics | 2025 | マルチレベルGNN | ホットスタンプB-ピラー板厚予測 | 画像＋グラフの長所統合、産業級精度 |
| 8 | Liu, Xia, Shi et al. | Theory-guided DNN for sheet metal bending | IEEE/CAA J. Autom. Sinica, 8(3) | 2021 | 理論誘導型DNN | 曲げスプリングバック補正 | Swift法則正則化で少量データ汎化性大幅改善 |
| 9 | Mozaffar, Bostanabad et al. | Deep learning predicts path-dependent plasticity | PNAS, 116(52) | 2019 | RNN | 経路依存性塑性構成則 | 降伏条件・流動則なしで構成則直接学習 |
| 10 | Gorji, Mozaffar et al. | RNN for modeling path dependent plasticity | J. Mech. Phys. Solids, 143 | 2020 | RNN/GRU | 異方性Yld2000-2d/HAHモデル | バウシンガー効果・潜在硬化の捕捉 |
| 11 | El Mrabti et al. | Springback optimization based on FEM-ANN-PSO | Struct. Multidisc. Optim., 64 | 2021 | ANN＋PSO | 深絞りスプリングバック最適化 | FEM-ANN-PSO統合戦略 |
| 12 | El Mrabti et al. | Comparative study of surrogates for AHSS forming failures | Int. J. Adv. Manuf. Tech., 121 | 2022 | RSM, RBF, Kriging, ANN比較 | AHSS板厚減少・破断予測 | ANNとクリギングが最高精度 |
| 13 | Kitayama, Saikyo et al. | Multi-objective optimization of blank shape and VBHF | Struct. Multidisc. Optim., 52 | 2015 | RBF＋SAO | ブランク形状＋VBHF同時最適化 | 実験検証（ACサーボプレス） |
| 14 | Feng et al. | VBHF optimization based on SVR and trust region | Int. J. Adv. Manuf. Tech., 105 | 2019 | SVR＋信頼領域 | 深絞りVBHF軌跡最適化 | しわ・破断の実験検証 |
| 15 | Feng et al. | Multi-objective VBHF optimization using Kriging, MOABC | Int. J. Adv. Manuf. Tech., 96 | 2018 | Kriging＋MOABC＋SPA | VBHF 3目的最適化 | スプリングバック・しわ・破断同時最小化 |
| 16 | Xie et al. | Improved RBM-BPNN and MOPSO for stamping | Struct. Multidisc. Optim., 64 | 2021 | RBM＋BPNN＋改良MOPSO | ダブルC部品プロセス最適化 | ハイブリッドサロゲートの優位性実証 |
| 17 | Park et al. | DNN-GA-MCS for multi-defect reduction in stamping | Materials, 17(11) | 2024 | DNN＋GA＋MCS | 破断・しわ・SB同時最適化 | 3欠陥で13〜19%改善 |
| 18 | Cui et al. | Partition cooling optimization in hot stamping B-pillar | Metals, 10 | 2020 | RSM＋NSGA-II | ホットスタンプ分割冷却最適化 | マルテンサイト1540MPa達成 |
| 19 | Cai et al. | ML prediction of part thickness in hot stamping | Int. J. Adv. Manuf. Tech., 119 | 2022 | 勾配ブースティング（最優） | 分割温度制御ホットスタンプ | 22温度ゾーン高次元で最高精度 |
| 20 | Ma et al. | GA-BP NN for 7075 Al hot stamping | Materials, 14 | 2021 | GA-BP NN | アルミ合金ホットスタンプ | 板厚減少率予測誤差5% |
| 21 | Palmieri et al. | Kriging metamodeling for deep-drawing BHF regulation | Metals, 11 | 2021 | Kriging | 深絞りBHFロバスト最適化 | ノイズ変数考慮のリアルタイム制御曲線 |
| 22 | Trzepieciński, Lemu | MLP-GA for springback prediction | Materials, 13(14) | 2020 | MLP＋GA | V曲げスプリングバック予測 | 相関係数0.962〜0.972 |
| 23 | Dang, Labergère, Lafon | POD-RBF surrogate for springback optimization | Procedia Eng., 207 (ICTP) | 2017 | POD＋RBF＋適応サンプリング | U曲げスプリングバック最適化 | 適応サンプリングでFEA回数削減 |
| 24 | Bici et al. | Dual RSM + shape function for die compensation | Math. Problems Eng. | 2019 | 双対RSM＋形状関数 | AHSS B-ピラー金型補正 | 材料特性変動6%考慮のロバスト設計 |
| 25 | Yi, Hyun, Hong | Digital twin for crack/wrinkle real-time prediction | Applied Sciences, 15(2) | 2025 | SVM, RF, GBM, ANN | ドロービード変更時の欠陥予測 | き裂分類精度100% |
| 26 | Dib et al. | Single and ensemble classifiers for defect prediction | Neural Comput. Appl., 32 | 2020 | MLP, SVM等＋アンサンブル | SB・板厚減少分類 | MLP F値90%以上、アンサンブルで改善 |
| 27 | Chheda et al. | FLD prediction using ML | IOP Conf. Ser.: MSE, 651 | 2019 | SVR＋勾配ブースティング | FLD予測（Al合金） | 2段階モデルで未知合金のFLD予測 |
| 28 | Gan, Li, Huang | Digital twin for stamping considering mold wear | ASME J. Manuf. Sci. Eng., 144 | 2022 | PSO-DE＋DT | プレス成形金型摩耗DT | 精度61.97%改善、板厚減少率14.35%低減 |
| 29 | Iriondo et al. | Transfer learning in press hardening surrogate modeling | J. Manufacturing Systems | 2024 | DNN＋転移学習 | ホットスタンプSim-to-Real | 少数実データでシミュレーション学習モデル適応 |
| 30 | Wang et al. | Transferred hybrid surrogate for metal tube bending | Robotics Comput. Integr. Manuf. | 2024 | GNN＋転移学習＋仮想サンプル | チューブベンディング | プロセス間知識転移 |
| 31 | Liu, Shi, Lin et al. | Reinforcement learning in free-form stamping | Procedia Manufacturing, 50 | 2020 | DQN強化学習 | 自由曲面スタンプ工程計画 | 職人経験不要の自律的成形経路学習 |
| 32 | Modad et al. | Review: Digital twins in sheet metal stamping | J. Intelligent Manufacturing | 2024 | DTレビュー | プレス成形DT体系化 | Industry 5.0・ゼロ欠陥製造ロードマップ |
| 33 | Karkaria et al. | Adaptive DT via POD-Koopman with MPC | arXiv | 2025 | POD＋Koopman＋MPC | ロボット板金成形適応制御 | オンラインモデル更新＋リアルタイム制御 |
| 34 | Pfeifer et al. | AI-assisted optimization in sheet metal forming | arXiv | 2025 | DL＋ベイズ最適化＋GPLVM | プロセスパラメータ最適化 | 専門家介入削減のAI支援ワークフロー |
| 35 | Heinzelmann et al. | Benchmark dataset for sheet metal forming ML | MATEC Web Conf. (IDDRG) | 2025 | ML分類 | 標準化ベンチマーク | コミュニティ共有データセット公開 |
| 36 | Okishi et al. | PointNeXtプレス成形形状予測サロゲート | JSAI 2025 | 2025 | PointNeXt（3D点群DL） | プレス金型→成形品形状予測 | 平均誤差0.3mm、推論数分 |
| 37 | Baum et al. | 3D point cloud surrogate for FEA sheet metal forming | at-Automatisierungstechnik, 73(4) | 2025 | 3D点群深層学習 | FEAサロゲート | 画像ベースより汎用的な3D手法 |
| 38 | Samad, Thakur, Basak | Formability prediction using ML: pre-strain and temperature | J. Mater. Eng. Perform. | 2025 | SVR, ランダムフォレスト | 予ひずみ・高温FLD予測 | RFがSVRを上回り、ドーム高さ誤差5.9% |
| 39 | Thamm et al. | CNN for forming limit analysis from tensile tests | Materials (PMC) | 2023 | CNN（弱教師あり） | 引張試験からFLC決定 | 高コスト中島試験の代替提案 |
| 40 | GE-EINN | Gradient enhanced-EINN for forming depth prediction | J. Manufacturing Processes | 2025 | GE-EINN（物理情報付きNN） | 冷間・温間スタンプ成形深さ | 15FEA＋9実験データで高外挿精度 |

---

## 6. 結論：統合的知見と展望

本調査を通じて明らかになった最も重要な知見は、プレス成形サロゲートモデリングが**3つの質的転換点**を経験していることである。第一に、スカラー出力からフルフィールド予測へのパラダイムシフトにより、設計者が板厚分布やひずみ場全体を可視化・最適化できるようになった。第二に、純データ駆動型から物理情報付きハイブリッドアプローチへの移行により、製造現場の慢性的課題である少量データ環境での信頼性が向上している。第三に、静的なFEA代替モデルからリアルタイム適応型デジタルツインへの発展により、製造プロセスの動的制御が視野に入った。

一方で、研究の大半が学術ベンチマーク問題に留まっており、**実生産の複雑性（数千の設計変数、多材料・多工程、ロット間変動）への大規模検証が決定的に不足**している。また、トランスフォーマー等の基盤モデルアーキテクチャの適用、マルチフィジックス統合サロゲート、説明可能なAIの統合は未開拓の有望領域である。日本国内では産業界主導の実用化研究（神戸製鋼のPointNeXt、SCSK/pSevenの鍛造最適化等）が進む一方、学術論文としての日本語文献は英語文献に比して限定的であり、JSTP・JSAIコミュニティでの発表蓄積の加速が期待される。
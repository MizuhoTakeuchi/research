# 仮想同期発電機(VSG)パラメータ最適化に関する包括的調査レポート

## TL;DR
- **VSGパラメータ最適化の主流は4系統に整理される**: (i) 固有値解析・小信号モデルに基づく解析的設計(Wu et al. 2016, D'Arco & Suul 2014–2015)、(ii) PSO/GA/GWO/AVOA等のメタヒューリスティクス、(iii) 2020年以降急速に普及した深層強化学習(SAC, DDPG, TD3)・ファジィニューラルネットワーク等のAI手法、(iv) H∞ロバスト制御・MPC。
- **対象パラメータは揺動方程式の慣性 J(または H)・ダンピング D が中心**であるが、最新研究は周波数ドループ kf (Dp)、電圧ドループ kv (Dq)、励磁系ゲイン Ke、仮想インピーダンス Rv/Xv、内部電圧/電流ループPIゲイン、PLLパラメータまでの統合最適化に発展しており、適応制御(self-adaptive VSG)が事実上の標準となりつつある。
- **2023–2026年の最新動向**: (a) 100%インバータ系統(IBR-dominated grid)を想定したUNIFI仕様策定への対応、(b) 電流飽和・トランジエント角安定性の制約を陽に組み込んだ最適化、(c) 物理情報埋め込み強化学習(physics-informed RL)とMPCのハイブリッド化、(d) 多VSG並列系統での協調最適化(SA-PSO等)、(e) BESS劣化・寿命を含む多目的最適化に集中している。

## Key Findings(主要発見)

1. **理論基盤は成熟しつつある**:Bevrani-Ise-Miura (IJEPES 2014)、D'Arco-Suul (EPSR 2015)、Wu-Ruan-Zhong (IEEE TIE 2016)が解析的設計のリファレンスとなり、Wu et al. (2016) は「電力ループの帯域幅は線間周波数の2倍未満」という設計鉄則を導出した。
2. **固定パラメータVSGは限界**: Alipoor-Miura-Ise (IEEE JESTPE 2015) のbang-bang交番慣性以降、Li et al. (IEEE TEC 2017) のJ-D同時適応など、適応化が業界標準化。
3. **AIベース手法が主流化**: Xu-Dragicevic-Xie-Blaabjerg (IEEE TPEL 2021) のDouble-ANNから、武漢大Lu-Zhuan (Sensors 2024) のSAC、TD3/DDPGベースのVFOPID (Technologies MDPI 2024) まで、信頼性・寿命を含む多目的最適化が進展。
4. **規格主導の収斂**: NREL主導のUNIFI Consortium が GFM IBR の機能要件を標準化し、産業実装の基盤を提供している。
5. **電流飽和の影響評価**: Rokrok et al. (IEEE TPS 2022) と Saffar et al. (IEEE TPEL 2022) により、電流リミッタがVSGのトランジエント角安定マージンを大きく左右することが定量化された。

## Details(詳細)

### 1. 概要・はじめに

再生可能エネルギー大量導入に伴い、同期発電機(SG)由来の慣性が低下し、Rate of Change of Frequency (RoCoF)の増大、周波数ナディアの悪化、低慣性振動の発生など系統安定性課題が深刻化している。これを補償する制御コンセプトとして、Beck & Hesseらが2007年に提唱したVISMA (Virtual Synchronous Machine)、Driesen & Visscher (IEEE PES GM 2008)のVSG概念、Zhong & Weiss (IEEE Trans. Ind. Electron. 2011) のSynchronverter、Ise研究室(大阪大学)のIse法が登場し、現在では総称してVSG/VSM/Grid-Forming (GFM) 制御と呼ばれる。

VSGはSGの揺動方程式 J ω₀ dω/dt = Pm − Pe − D(ω−ω₀) を仮想的にインバータ制御で実装するため、その係数 J, D, Dp, Dq, Ke, 仮想インピーダンス、電流/電圧PIゲイン、PLLゲイン等多数の自由パラメータを系統条件に応じて適切に決定する必要がある。本稿はこれらのパラメータ最適化に関する文献を網羅的に整理する。

### 2. VSGの基本概念と主要実装方式の比較

文献レビューの代表例として、Bevrani, Ise, Miura, "Virtual synchronous generators: A survey and new perspectives," *International Journal of Electrical Power & Energy Systems*, vol. 54, pp. 244–254 (2014) があり、本分野で最も引用されている総説の1つである。Mallemaci, Mandrile, Rubino, Mazza, Carpaneto, Bojoi, "A comprehensive comparison of Virtual Synchronous Generators with focus on virtual inertia and frequency regulation," *Electric Power Systems Research* (2021) では、VISMA、VISMA I/II、Synchronverter、Osaka(Ise)、SPC、VSYNC、KHI、CVSM、S-VSC など10種類のVSGを統一的なチューニング下で比較した。

| 実装方式 | 提案者・年 | モデル次数 | 同期化 | 特徴 |
|---|---|---|---|---|
| VISMA | Beck & Hesse (2007), TU Clausthal | 高(SGの完全電磁モデル) | 電流制御型 | SG電磁挙動を忠実に再現。電流源出力で弱系統電圧支持は弱い |
| Synchronverter | Zhong & Weiss, IEEE TIE 2011 | 中 | 電圧制御型 | SGの数式モデルをそのまま実装。Self-synchronized版 (Zhong et al., IEEE TPEL 2014) ではPLL不要 |
| Ise法(大阪大) | Sakimoto, Miura, Ise (2011), Alipoor et al. (2015) | 低(揺動方程式のみ) | 電圧制御型 | 単純化された P-f/Q-V ループ。Alternating moment of inertia による振動抑制が代表的拡張 |
| VSYNC | Visscher (2008), VSYNC EUプロジェクト | 低 | 電流制御型 | 周波数微分(df/dt)のみで慣性応答模擬。実装最簡易 |
| KHI法 | Hirase, Sugimoto, Sakimoto, Bevrani, Ise, *Applied Energy* vol. 210 (2018) | 代数型(algebraic) | 電圧制御型 | 最少パラメータの代数モデル。Kawasaki Heavy Industries商用化 |
| SPC (Synchronous Power Controller) | Rodriguez et al. | 中 | 電圧制御型 | ドループ係数とダンピング項が結合 |
| CVSM | Cascaded VSM | 中 | 電圧制御型 | 内部カスケード電圧/電流制御を保持 |

Synchronverter論文 (Zhong & Weiss, IEEE TIE 2011) は、Zhong教授の公式CV(syndem.com)によれば「IEEE Transactions on Industrial Electronics の43年史上 Top 3 Most-cited Non-survey Paper、Google引用 3,400+、閲覧数 35,000+」と記載されており、本分野で最も影響力のある論文の一つである。D'Arco & Suul は IEEE PowerTech 2013 および *Electric Power Systems Research* vol. 122 (2015), pp. 180–197 において、VSMのクラス分類と、VSMがマイクログリッドにおいて周波数ドループ制御と等価な小信号応答を持つことを証明した(IEEE Trans. Smart Grid vol. 5, no. 1, 2014, pp. 394–395)。

### 3. パラメータの分類と各パラメータの役割

#### 3.1 中核パラメータ
- **(1) 仮想慣性 J (または H = Jω₀²/2Sb)**: 周波数ナディア・RoCoFを直接決定。Wu et al. (IEEE TIE 2016) は線周波数平均小信号モデルから J の設計範囲を導出。
- **(2) ダンピング係数 D (または Dp)**: 振動減衰と整定時間を支配。J と D の組合せが2次系の自然角周波数 ωn=√(SmaxEUg/(Jω₀Z))・減衰比 ζ=D/(2√(Jω₀SmaxEUg/Z)) を規定。
- **(3) 周波数ドループ係数 kf (Dp)**: 一次周波数調整。SPC方式ではダンピング項に組み込まれている。
- **(4) 電圧ドループ係数 kv (Dq)**: 無効電力分担と電圧支持。
- **(5) 励磁系ゲイン Ke**: Synchronverterや Hirase et al. (Applied Energy 2018) のKHI型では仮想AVRゲインとして出力電圧の動特性を支配。

#### 3.2 補助パラメータ
- **(6) 仮想インピーダンス Rv, Xv**: 電力分担、線路X/R比補償、フォルトリミッタ機能。Liang et al. (IEEE TIA 2021) ではマイクログリッドの無効電力分担向け適応仮想インピーダンスを提案。
- **(7) 電圧/電流ループ PI ゲイン (Kpv, Kiv, Kpi, Kii)**: カスケード型VSMの内部ループ。LCLフィルタ共振抑制と高速動的応答に直接影響。
- **(8) PLL パラメータ (Kp_PLL, Ki_PLL)**: 電流制御型VSGや弱系統接続時に系統強度との相互作用で不安定化要因。Self-synchronized synchronverter(Zhong et al., IEEE TPEL 2014)はPLLそのものを排除した。
- **(9) その他**: ローパスフィルタ時定数、不感帯、サチュレーション角(電流飽和)、ガバナ時定数 Tg、タービン時定数 Tt等。

#### 3.3 トレードオフ
- **J増 ⇄ ナディア改善・整定時間延長**(Frontiers Energy Res. 2025)
- **D増 ⇄ 振動減衰・定常状態偏差増大**(SPC等、ドループ依存型では特に顕著)
- **慣性 vs 過電流耐量**: J増は過渡電流増を招き、コンバータ熱限界制約と衝突。
- **応答速度 vs 安定性**: 高ゲイン電圧ループは弱系統接続で発散しうる。
- **慣性 vs 寿命**: Xu, Dragicevic, Xie, Blaabjerg (IEEE TPEL vol. 36, no. 8, 2021, pp. 9453–9464) はJ増に伴う半導体寿命消費(LC)の関数化を行い、信頼性と安定性の両立をDouble-ANN で最適化した。

### 4. 各最適化手法の詳細

#### 4.1 解析的手法
- **小信号固有値解析・根軌跡**: Wu et al. (IEEE TIE 2016) のJパラメータ設計、Frontiers Energy Res. (2024) "Small-disturbance stability analysis and control-parameter optimization of grid-connected VSG"。
- **感度解析・参与因子**: 小信号モデルの状態空間表現に対する固有値感度。
- **インピーダンスベース安定解析**: HVDC への応用 (ScienceDirect 2022, "Stability analysis of VSG control in HVDC")。
- **二次系等価設計**: Wu et al. はパラメータを設計指針(設定時間・オーバーシュート率)に変換。

#### 4.2 メタヒューリスティクス手法
- **PSO**: 最も普及。Gurski, Kuiava, Perez, Benedito, Damm, "A Novel VSG with Adaptive Virtual Inertia and Adaptive Damping Coefficient," *Energies*, MDPI vol. 17, no. 17, 4370 (2024) はAID-VSGをPSOで調整し10,000初期条件を試験。Frontiers Energy Res. 2025 "Control parameter identification method for VSG considering EVs based on dynamic PSO"。
- **改良PSO・SA-PSO**: Lin, Liu, Wang, "Stability analysis and control parameter optimization of multi-VSG parallel grid-connected system," *Electric Power Systems Research*, vol. 221, 109478 (2023) は仮想インピーダンスと無効電力積分項を含む全パラメータを Simulated Annealing-PSO で最適化。
- **GA・MOGA**: Frontiers Energy Res. 2024 "Multi-objective parameter design and economic analysis of VSG-controlled hybrid energy systems" はESS-VSG/PV-VSGをMOGAで最適化、ナディアと整定時間を同時最大化/最小化。
- **GWO (Grey Wolf Optimizer)**: Al-Saadi, Mahafzah, Hatmi, "Improved Frequency Response of Parallel Virtual Synchronous Generators Using GWO," IIETA *JESA* vol. 56, no. 3 (2023), DOI:10.18280/jesa.560307 では、4 kVA定格のMATLAB/Simulinkモデルにおいて「同期に伴う系統周波数のオーバーシュートをGWOにより8%から0%に削減」(原文: "The overshot in the grid frequency due to synchronization is reduced from 8% to 0% with GWO")した。
- **AVOA (African Vulture Optimization)**: Rajaguru & Iyswarya Annapoorani (Sci. Rep. 2025) はSMES-VSGのフラクショナル次PIDをAVOAで調整、HHO/GA/GWO/PSOに対し優位を示した。
- **MVMOs (Mean-Variance Mapping Optimization)**: Rahman et al., *IET GTD* (2021) はBESS+SICのVSG最適配置でPSOと比較。

#### 4.3 機械学習・AIベース手法
- **強化学習(RL)一般**: Saadatmand, Shamsi, Ferdowsi, "Adaptive critic design-based RL approach in controlling virtual inertia-based grid-connected inverters," *Int. J. Electr. Power Energy Syst.* (2020); Skiparev, Nosrati, Petlenkov, Belikov (SSRN/IEEE); Afifi, Marei, Mohamad, "RL-Based Virtual Inertia Controller for Frequency Support in Islanded Microgrids," *Technologies* MDPI vol. 12, no. 3, 39 (2024) はTD3とDDPGでVFOPIDを実装。
- **SAC (Soft Actor-Critic)**: Lu & Zhuan (武漢大学), "Adaptive Control for VSG Parameters Based on Soft Actor Critic," *Sensors* vol. 24, 2035 (2024) は J と D を SAC で適応調整。
- **物理情報付きRL**: Qiu et al. (ICPRE 2023), MDPI *Electronics* vol. 14, 3503 (2025) "Adaptive Transient Power Angle Control for VSGs via Physics-Embedded RL" — 3N-D コントローラがJ, Dを定期更新。IEEE-39バスで検証。
- **ファジィ・FNN**: Karimi, Khayat, Naderi, Dragicevic, Mirzaei, Blaabjerg, Bevrani, "Inertia Response Improvement in AC Microgrids: a Fuzzy-Based VSG Control," *IEEE Trans. Power Electron.*, vol. 35, no. 4 (2020); Thomas, Kumaravel, Ashok, "Fuzzy Controller-Based Self-Adaptive VSM for Microgrid Application," *IEEE Trans. Energy Convers.*, vol. 36, no. 3, pp. 2427–2437 (2021); arXiv 2506.18611 "Frequency Control in Microgrids: An Adaptive Fuzzy-Neural-Network VSG"。
- **深層学習(DCNN, RBF-NN)**: Ling, Liu, Wang, Yin (IET Power Electronics, doi 10.1049/pel2.12600) はRBFニューラルネットでJ, Dを適応調整。Zeng et al. (CYBER 2022) DCNNベースVSG。
- **ハイブリッドPSO-RL**: MDPI *Electronics* vol. 14, 3349 (2025) — オフラインPSOで基本値、オンラインRLで微調整。整定時間0.10 s、RoCoF 0.2 Hz/sを達成。

#### 4.4 アダプティブ・自己調整(Self-adaptive VSG)
- **Bang-bang型**: Alipoor, Miura, Ise, "Power System Stabilization Using VSG With Alternating Moment of Inertia," *IEEE J. Emerging Sel. Topics Power Electron.*, vol. 3, no. 2, pp. 451–458 (2015); Li, Wen, Wang (IEEE Access 2019)。
- **連続適応**: Li, Zhu, Lin, Bian, "A Self-adaptive Inertia and Damping Combination Control of VSG to Support Frequency Stability," *IEEE Trans. Energy Convers.* vol. 32, no. 1, pp. 397–398 (2017); Ren, Chen, Zhang, Li, arXiv:2009.05916 (2020)。
- **指数慣性・tanhベース非線形ダンピング**: MDPI *Energies* vol. 18, 3822 (2025) は指数関数慣性と tanh 型ダンピングを改良PSOで最適化。
- **Self-tuning VSM**: Torres & Lopes, "Self-tuning virtual synchronous machine," *IEEE Trans. Energy Convers.* vol. 29, no. 4, pp. 833–840 (2014)。

#### 4.5 ロバスト最適化・H∞制御
- **H∞ ロバスト VIC**: Kerdphol, Rahman, Mitani, Watanabe, Küfeoğlu, "Robust Virtual Inertia Control of an Islanded Microgrid Considering High Penetration of Renewable Energy," *IEEE Access* vol. 6, pp. 625–636 (2018), DOI: 10.1109/ACCESS.2017.2773486. 混合感度 (S/KS/T) ループシェイピング定式化により W1(性能), W2(制御努力), W3(不確かさ) を設計。20 MVAマイクログリッドベースで H, D, Tg, Tt の ±50% 摂動に対しロバスト性を実証、PI型と比べ ITAE を最小化。
- **適応ファジィ**: Kerdphol, Watanabe, Hongesombut, Mitani, "Self-Adaptive Virtual Inertia Control-Based Fuzzy Logic to Improve Frequency Stability of Microgrid With High Renewable Penetration," *IEEE Access* vol. 7, pp. 76071–76083 (2019), DOI: 10.1109/ACCESS.2019.2920886. ファジィ論理がΔfとdΔf/dtに基づき仮想慣性定数 KVI をリアルタイム適応。
- **拡張VSGロバスト周波数制御**: Fathi, Shafiee, Bevrani, "Robust frequency control of microgrids using an extended virtual synchronous generator," *IEEE Trans. Power Syst.* vol. 33, no. 6, pp. 6289–6297 (2018)。
- **H₂ノルム最小化**: Ademola-Idowu & Zhang (PESGM 2018) は多GFM系統のJ/Dを H₂ ノルム最小化問題として定式化。

#### 4.6 モデル予測制御(MPC)ベース
- **VSM-MPC**: Kerdphol, Rahman, Mitani, Hongesombut, Küfeoğlu, "Virtual inertia control-based MPC for microgrid frequency stabilization," *Sustainability* vol. 9, no. 5, 773 (2017)。
- **適応MPC + RL**: *Scientific Reports* (2025) "Improving frequency stability in grid-forming inverters with adaptive MPC and novel COA-jDE optimized RL" は Hybrid Crayfish Optimization + Self-Adaptive DE で MPC のQ, R を調整。
- **モデルフリーMPC**: ARXベースVSG-MFPCがパラメータロバスト性向上。
- **モデル予測電流制御**: Frontiers Energy Res. (2023) "Parameter adaptive MPC strategy of NPC three-level VSG"。
- **多目的予測VSG**: *Scientific Reports* (2025) (PMC11920424) "Multi-objective adaptive predictive VSG control" は H ∈ [1,4]s, D ∈ [20,65] pu のリアルタイム適応で、原文によれば「2.2 kWと5.5 kWの並列接続SEIGを用いた実験検証で、最大RoCoFを±0.48 Hz/sから±0.21 Hz/sへ56%削減し、周波数ナディアを50.85–50.87 Hzへ33%改善」("56% reduction in maximum RoCoF (from ±0.48 Hz/s to ±0.21 Hz/s), 33% improvement in frequency nadir (50.85–50.87 Hz)")した。

#### 4.7 多目的最適化
目的関数候補: 周波数偏差ITAE/ISE/IAE、整定時間、オーバーシュート、RoCoF、過電流抑制、TED(全エネルギー消散)、MGFD(最大周波数偏差)、半導体寿命、経済性。Liu et al. (CSEE) はTEDをコスト関数、MGFDを制約にしたVSG最適化を提案。

### 5. 目的関数と制約条件

#### 5.1 主要目的関数
- **時間積分指標**: ITAE = ∫t|e|dt(整定時間ペナルティ強)、ISE、IAE、ITSE。
- **周波数指標**: 最大Δf(ナディア)、RoCoF(df/dt)。
- **過渡指標**: オーバーシュート、整定時間、Critical Clearing Time (CCT)。
- **電力品質**: THD、電圧偏差、過電流ピーク。
- **経済・寿命**: TED、半導体LC、BESS劣化。

#### 5.2 制約条件
- **物理制約**: コンバータ電流容量(通常1.2–1.5 pu)、DC側電圧上限/下限、BESS SOC範囲。
- **安定性制約**: 全閉ループ固有値の左半平面、減衰比 ζ ≥ 0.3、ゲイン余裕GM > 6 dB、位相余裕PM > 45°。
- **規格制約**: ENTSO-E ±0.5 Hz, RoCoF ±2 Hz/s, IEEE 2800 / GC0137 / UNIFI仕様 GFM要件。
- **運用制約**: 並列VSG間電力分担、不感帯。
- **電流飽和制約**: Rokrok et al., "Transient stability assessment and enhancement of grid-forming converters embedding current reference saturation," *IEEE Trans. Power Syst.* vol. 37, no. 2 (2022) は電流参照飽和角の最適選択でCCT延長を示した。Saffar, Driss et al. (IEEE TPEL 2022) は d/q軸・角度優先の各電流制限戦略の影響を比較。

### 6. 応用領域別の事例

#### 6.1 再生可能エネルギー統合
- **PV-VSG**: Deng, Xia, Yin, Jin, Peng, Wang, "Small-Signal Modeling and Parameter Optimization Design for PV-VSG," *Energies* vol. 13, no. 2, 398 (2020); Frontiers Energy Res. (2025) "Application of adaptive VSG based on improved active power loop in PV-storage."
- **風力**: Hosny, Marei, Mohamad, "Adaptive hybrid virtual inertia controller for PMSG-based wind turbine based on fuzzy logic," *Sci. Rep.* (2025); MDPI *Applied Sciences* vol. 13, 5569 (2023) "Optimization of VSG Control Parameter for WTG Considering Physical Constraint Boundary."

#### 6.2 マイクログリッド(島運転・系統連系)
- 多VSG並列の小信号安定: Lin et al. (EPSR 2023, SA-PSO).
- 適応VSGとESS: Springer *Protection and Control of Modern Power Systems* (2023) "Adaptive VSG control using optimized bang-bang for islanded microgrid stability improvement"。
- Liu, Miura, Bevrani, Ise, "Enhanced Virtual Synchronous Generator Control for Parallel Inverters in Microgrids," *IEEE Trans. Smart Grid*, vol. 8, no. 5, pp. 2268–2277 (2017)。

#### 6.3 HVDC・FACTS
- VSC-HVDC統合: Aouini et al.; Wiley *International Transactions on Electrical Energy Systems* (2024) "Grid Synchronization of VSC-HVDC Based on VSG"。
- MTDC: Wang et al., "A parameter alternating VSG controller of VSC-MTDC systems for low frequency oscillation damping," *IEEE Trans. Power Syst.* vol. 35, pp. 4609 (2020)。

#### 6.4 BESS・EV
- Hu, Kong, Xu et al., "An adaptive VSG control strategy of BESS for power system frequency stability," *Int. J. Electr. Power Energy Syst.* vol. 149, 109039 (2023)。
- Battery health-responsive VSG: MDPI *Batteries* vol. 12, 36 (2025)。
- Suul, D'Arco, Guidi, "VSM-based control of single-phase bi-directional battery charger for V2G," *IEEE Trans. Ind. Appl.* vol. 52, pp. 3234–3244 (2016)。

#### 6.5 弱い系統・100%IBR系統
- Wu et al. (IEEE TPEL 2019) "Impedance Analysis of VSG-Based Vector Controlled Converters for Weak AC Grid Integration"。
- arXiv:1911.02870 "Beyond low-inertia systems: Massive integration of grid-forming power converters"。
- MDPI *Electronics* vol. 14, 4202 (2025) "Stability Assessment of Fully Inverter-Based Power Systems Using Grid-Forming Controls"。
- UNIFI Specifications V.3: Kroposki, Ben (sole author), "UNIFI Specifications for Grid-Forming Inverter-Based Resources: Version 3," OSTI ID: 3016250, DOI: 10.2172/3016250, 30 January 2026, National Laboratory of the Rockies (NREL), Golden, CO。Version 2 は 2024年3月公開(UNIFI-2024-2-1)。

### 7. 比較分析:手法間メリット・デメリット

| 手法 | 利点 | 欠点 |
|---|---|---|
| 固有値・根軌跡解析 | 物理的洞察、設計範囲の保証、計算軽量 | 線形化前提・運用点固定、強非線形に脆弱 |
| PSO/GA等メタヒューリスティクス | グローバル探索、多目的対応、モデル詳細不要 | オフライン、運用条件変化に追従困難、収束の早熟 |
| ファジィ/ニューラルアダプティブ | リアルタイム適応、ルール解釈性(ファジィ) | ルール設計の経験依存、安定性保証が困難 |
| RL/DRL (SAC, DDPG, TD3) | モデルフリー、複雑系での自己改善 | サンプル効率低、catastrophic forgetting、安定性保証なし |
| 物理情報埋め込みRL | 学習効率改善、解釈性向上 | 物理モデル定義が必要 |
| H∞ロバスト制御 | パラメトリック不確かさへの耐性、保守的 | コントローラ次数高、設計者の経験必要 |
| MPC | 制約陽処理、予測能力 | 計算負荷、モデル誤差敏感 |
| ハイブリッド(PSO-RL, COA-jDE+MPC等) | 双方の長所、頑健性 | 実装複雑性、調整パラメータ多 |

### 8. 最新動向(2023-2026)

1. **物理情報強化学習の台頭**: MDPI *Electronics* 14, 3503 (2025) の3N-D方式、武漢大のSAC-VSG (2024) 等。
2. **多VSG並列・系統規模協調最適化**: SA-PSOによる多VSG (Lin et al. EPSR 2023)、Renewable & Sustainable Energy Reviews (2025) "Review on small-signal stability of multiple VSGs"。
3. **電流飽和・トランジエント角安定の陽な統合**: Rokrok et al. (IEEE TPS 2022)、Saffar et al. (IEEE TPEL 2022)、Frontiers Energy Res. (2025) "Transient stability analysis of a VSG grid-connected system."
4. **UNIFI仕様準拠GFM設計**: NRELのKroposkiによるV.2 (2024)、V.3 (2026, OSTI ID 3016250, DOI: 10.2172/3016250)。GC0137(英国)、IEEE 2800への対応。
5. **適応MPCとメタヒューリスティクスの融合**: Nature *Sci. Rep.* (2025) のCOA-jDE+MPC。
6. **BESS劣化・寿命を含む多目的最適化**: Xu, Dragicevic, Xie, Blaabjerg (IEEE TPEL 2021) のDouble-ANN信頼性、MDPI *Batteries* 12, 36 (2025) の health-responsive VSG。
7. **DCNN/CNNベース予測制御**: Zeng et al. (CYBER 2022) "DCNN-based VSG control to improve frequency stability of PV-ESS."
8. **総説論文の急増**: Ding & Cao "Deep and Reinforcement Learning in Virtual Synchronous Generator: A Comprehensive Review," *Energies* MDPI vol. 17, 2620 (2024); MDPI *Energies* vol. 18, 3557 (2025) "Current Status, Challenges and Future Perspectives of Operation Optimization, Power Prediction and VSG of Microgrids"; ScienceDirect (2024) "On the Role of Virtual Inertia Units in Modern Power Systems"。

### 9. 課題と将来展望

- **保証付き安定性のあるAI手法**: RLの探索段階で系統不安定化リスク。Lyapunov-constrained RL、safe RLの統合が必要。
- **スケーラビリティ**: 多VSG・多エリア系統に対する集中・分散最適化アーキテクチャ。
- **保護協調**: GFMのフォルトリスポンス特性は従来の保護リレー(距離・方向・電力動揺)と整合しない。
- **試験・検証標準**: HIL/PHIL検証(Typhoon HIL、dSPACE、RTDS)が事実上の標準。実機デモはオーストラリア・ヨーク半島ダルリンプル変電所のESCRI-SA BESS(30 MW / 8 MWh、ElectraNet所有・AGL運用、EPC: Consolidated Power Projects、機器: ABB(現Hitachi Energy)・Samsung SDI、2018年12月商業運転開始、ARENA助成金1,200万豪ドル)等。
- **市場・経済性最適化との結合**: VSGパラメータ最適化と容量・配置最適化、アンシラリーサービス取引の同時設計。
- **物理モデル取得の高度化**: パラメータ識別とデジタルツインの統合。

### 10. 主要研究グループ・引用数の高い論文

#### 10.1 主要研究グループ
- **Qing-Chang Zhong (Illinois Tech, Syndem LLC)**: Synchronverter (IEEE TIE 2011, IEEE TIE 43年史上 Top 3 Most-cited Non-survey Paper、Google引用 3,400+)、Self-synchronized Synchronverter (IEEE TPEL 2014, 引用 1,200+)、Robust Droop Control (引用 1,000+)。
- **Toshifumi Ise, Yushi Miura (大阪大学)**: VSG概念の中核、Alipoor博論ら、Bevraniとの共著サーベイ。
- **Hassan Bevrani (University of Kurdistan)**: Virtual Inertia総説、Microgrid Dynamics and Control書籍。
- **Jon Are Suul, Salvatore D'Arco (SINTEF/NTNU)**: VSM分類・小信号モデル。
- **Frede Blaabjerg, Tomislav Dragicevic (Aalborg/DTU)**: AIベースVSG。
- **Yasunori Mitani, Masayuki Watanabe, Thongchart Kerdphol (九州工業大学)**: ロバスト/ファジィVIC、低慣性系統評価。
- **Xinbo Ruan, Heng Wu (南京航空航天大学)**: VSG小信号モデルとパラメータ設計。
- **NREL/EPRI/UT Austin (UNIFI Consortium)**: GFM仕様策定。

#### 10.2 高被引用代表文献
1. Q.-C. Zhong, G. Weiss, "Synchronverters: Inverters That Mimic Synchronous Generators," *IEEE Trans. Industrial Electronics*, vol. 58, no. 4, pp. 1259–1267 (2011).
2. H. Bevrani, T. Ise, Y. Miura, "Virtual synchronous generators: A survey and new perspectives," *Int. J. Electr. Power Energy Syst.*, vol. 54, pp. 244–254 (2014).
3. Q.-C. Zhong, P.-L. Nguyen, Z. Ma, W. Sheng, "Self-Synchronized Synchronverters: Inverters Without a Dedicated Synchronization Unit," *IEEE Trans. Power Electron.*, vol. 29, no. 2, pp. 617–630 (2014).
4. H. Wu, X. Ruan, D. Yang, X. Chen, W. Zhao, Z. Lv, Q.-C. Zhong, "Small-Signal Modeling and Parameters Design for Virtual Synchronous Generators," *IEEE Trans. Industrial Electronics*, vol. 63, no. 7, pp. 4292–4303 (2016).
5. J. Alipoor, Y. Miura, T. Ise, "Power System Stabilization Using VSG With Alternating Moment of Inertia," *IEEE J. Emerg. Sel. Topics Power Electron.*, vol. 3, no. 2, pp. 451–458 (2015).
6. D. Li, Q. Zhu, S. Lin, X. Y. Bian, "A Self-Adaptive Inertia and Damping Combination Control of VSG to Support Frequency Stability," *IEEE Trans. Energy Convers.*, vol. 32, no. 1, pp. 397–398 (2017).
7. T. Kerdphol, F. S. Rahman, Y. Mitani, M. Watanabe, S. Küfeoğlu, "Robust Virtual Inertia Control of an Islanded Microgrid Considering High Penetration of Renewable Energy," *IEEE Access*, vol. 6, pp. 625–636 (2018).
8. T. Kerdphol, M. Watanabe, K. Hongesombut, Y. Mitani, "Self-Adaptive Virtual Inertia Control-Based Fuzzy Logic to Improve Frequency Stability of Microgrid With High Renewable Penetration," *IEEE Access*, vol. 7, pp. 76071–76083 (2019).
9. Y. Hirase, K. Abe, K. Sugimoto, K. Sakimoto, H. Bevrani, T. Ise, "A novel control approach for virtual synchronous generators to suppress frequency and voltage fluctuations in microgrids," *Applied Energy*, vol. 210, pp. 699–710 (2018).
10. Q. Xu, T. Dragicevic, L. Xie, F. Blaabjerg, "Artificial Intelligence-Based Control Design for Reliable Virtual Synchronous Generators," *IEEE Trans. Power Electron.*, vol. 36, no. 8, pp. 9453–9464 (2021).
11. S. D'Arco, J. A. Suul, O. B. Fosso, "A Virtual Synchronous Machine implementation for distributed control of power converters in SmartGrids," *Electric Power Systems Research*, vol. 122, pp. 180–197 (2015).
12. J. Liu, Y. Miura, H. Bevrani, T. Ise, "Enhanced Virtual Synchronous Generator Control for Parallel Inverters in Microgrids," *IEEE Trans. Smart Grid*, vol. 8, no. 5, pp. 2268–2277 (2017).
13. E. Rokrok, T. Qoria, A. Bruyere, B. Francois, X. Guillaud, "Transient Stability Assessment and Enhancement of Grid-Forming Converters Embedding Current Reference Saturation as Current Limiting Strategy," *IEEE Trans. Power Syst.*, vol. 37, no. 2 (2022).
14. V. Mallemaci, F. Mandrile, S. Rubino, A. Mazza, E. Carpaneto, R. Bojoi, "A comprehensive comparison of Virtual Synchronous Generators with focus on virtual inertia and frequency regulation," *Electric Power Systems Research* (2021).
15. X. Ding, J. Cao, "Deep and Reinforcement Learning in Virtual Synchronous Generator: A Comprehensive Review," *Energies*, MDPI vol. 17, no. 11, 2620 (2024).

## Recommendations(推奨)

**段階1 — 既存設計の妥当性確認**: 小信号固有値解析・感度解析でJ, D, kf, kv, Ke, 仮想インピーダンスの設計範囲を確認(Wu et al. 2016手法)。設計しきい値: 減衰比 ζ ≥ 0.3、電力ループ帯域 ≪ 2f₀。

**段階2 — オフライン最適化**: 単目的なら改良PSO、多目的ならNSGA-II/MOGAでITAE+RoCoF+整定時間の重み付き和(または Pareto front)を最適化。10,000ケース程度のモンテカルロ検証(Gurski et al. 2024手法)。

**段階3 — 適応化**: 想定運用範囲が広い場合は Self-adaptive VSG(Li et al. 2017、Kerdphol et al. 2019 ファジィ)または物理情報埋め込みRL(SAC/PPO)。**ベンチマーク**: ナディア・RoCoF・整定時間でConventional VSG比 30–50%改善、ITAE 50%以上削減。

**段階4 — ロバスト化**: 不確かさが大きい弱系統・100%IBR系統では H∞混合感度設計(Kerdphol et al. 2018)または適応MPC+RLハイブリッド。系統強度SCR < 2.5、または Tg/Tt の ±50% 摂動を制約に組み込む。

**段階5 — 制約・実装**: 電流飽和角の最適選択(Rokrok 2022)、BESSのSOC・寿命制約、UNIFI Specifications V.3 / IEEE 2800 / GC0137適合。

**段階6 — 検証**: HIL(Typhoon, OPAL-RT, RTDS)+ PHILで電流制限・FRT・モード切替を検証。実機デモにつなぐ(参考: ESCRI-SA BESS 30 MW/8 MWh)。

**閾値が変わる条件**: SCR > 5 の強系統では H∞は過剰、PI型でよい。RES比率 < 30% なら固定パラメータ可。RES比率 > 70% かつ多VSG並列なら、SA-PSOまたは RL+MPC 必須。

## Caveats(留意点)

1. **論文の信頼性**: MDPI、Frontiers、IEEE Access などオープンアクセス誌の急増により、査読品質にばらつきがある。本稿は被引用数・実証性で取捨選択したが、最新の自己適応RL論文の多くはシミュレーションのみ。
2. **数値の文脈依存**: 本文で引用した「RoCoF 56%削減、ナディア33%改善」(Sci. Rep. 2025, PMC11920424)は2.2 kW・5.5 kWのSEIG並列実験に固有の値で、H ∈ [1,4]s・D ∈ [20,65] pu のリアルタイム適応条件下のものであり、別系統への外挿には注意が必要。
3. **特許・産業実装情報**: Syndem LLC (Zhong)、Kawasaki Heavy Industries (Hirase)、ElectraNet ESCRI-SA等の実装詳細は公開情報限定。
4. **AI手法の安定性保証**: 強化学習・深層学習は形式的な安定性保証を欠き、実系統適用には保護機構(safe RL、バックアップPI等)が必要。
5. **規格動向**: UNIFI仕様V.3(Kroposki, OSTI ID 3016250, DOI 10.2172/3016250, 2026年1月公開)、IEEE 2800、GC0137は進行中で、本稿時点(2026年5月)から追加改訂の可能性あり。
6. **PLLパラメータの扱い**: 文献の多くがPLL不要のSelf-synchronized型を採用しており、PLLパラメータ最適化は弱系統接続のGFL混在系統に限定的。
7. **論文要旨ベースの引用**: 本稿の一部数値・著者順は二次情報(レビュー記事、引用一覧、ResearchGate要約)に基づく。原論文での厳密な値や仕様の検証推奨。特にKerdphol 2018 (IEEE Access vol.6) と Kerdphol 2019 (IEEE Access vol.7)は著者構成が異なる別論文である点に留意。
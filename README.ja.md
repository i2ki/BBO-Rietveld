# BBO-Rietveld
ブラックボックス最適化に基づく自動結晶構造解析手法、BBO-Rietveldの実装です。  
![BBO-Rietveldの概略図](https://gist.githubusercontent.com/resnant/32aed77b71f71d798a847fab16431315/raw/cadacc758e000e1dc7d7e4d0253512f8b83b4e88/bbo_rietveld_schematic.jpg)

## 引用
Ozaki, Y., Suzuki, Y., Hawai, T., Saito, K., Onishi, M., and Ono, K.  
Automated crystal structure analysis based on blackbox optimisation.  
<i>npj Computational Materials</i> <b>6</b>, 75 (2020).  
[https://doi.org/10.1038/s41524-020-0330-9](https://doi.org/10.1038/s41524-020-0330-9)

## 必要なもの

- [Docker](https://www.docker.com/)
  - Dockerを初めて使う場合は、まずお使いのコンピュータにDocker Desktopをセットアップしてください。
  - WindowsとMac用のドキュメントは以下にあります：
    - https://docs.docker.com/docker-for-windows/
    - https://docs.docker.com/docker-for-mac/
  - このソフトウェアはmacOS(10.14)とUbuntu(18.04)で動作確認済みです。macOSまたはUbuntuでの実行を推奨しますが、Dockerが動作するコンピュータであれば動作するはずです。

## 使い方
__注意: macOSまたはUbuntuを使用しており、`sudo`権限なしで`docker`コマンドを実行できることを前提としています。Windowsを使用している場合は、互換性の観点から、コマンドプロンプトやWindows PowerShellではなく、WSL2ターミナルの使用を強く推奨します。__

### 方法1: Docker Compose を使用する（推奨）

1. まず、このリポジトリをコンピュータにクローンします。

2. ローカルでイメージをビルドします（composeが自動でDockerfileを参照します）。
```sh
docker compose build --no-cache
```
注: 本リポジトリの compose は `build: ./docker` を含むため `docker compose pull` は不要です。リモート配布用タグが必要な場合のみ `docker tag`/`docker push` を使用してください。

3. Docker Composeでコンテナを起動します。
```sh
docker compose up
```

4. ブラウザで http://127.0.0.1:8888/ を開くと、JupyterLabのウィンドウが表示されます（トークン入力は不要）。
```
...

[I 11:23:41.325 LabApp] The Jupyter Notebook is running at:
[I 11:23:41.325 LabApp] http://900cc3c9314c:8888/?token=126c79b021d40344592d5b066225b32474487cb69f711ffe
[I 11:23:41.325 LabApp]  or http://127.0.0.1:8888/?token=126c79b021d40344592d5b066225b32474487cb69f711ffe

...
```
このリポジトリのDockerイメージではJupyterのトークンを無効化しているため、`http://127.0.0.1:8888/` に直接アクセスできます。
起動直後にカーネルが一度だけ再起動することがありますが、その後は安定して動作します。

5. JupyterLab上で`1_Y2O3.ipynb`または他のノートブックを開き、ノートブックのセルを実行します。

6. 終了するには、ターミナルで`Ctrl+C`を押してから以下を実行します：
```sh
docker compose down
```

### 方法2: シェルスクリプトを使用する（任意）

1. まず、このリポジトリをコンピュータにクローンします。

2. bbo-rietveldコンテナを実行します（ローカルビルド済みの v1.3 を使用）。
```sh
./run.sh
```
または
```bash
docker run --rm -it -p 8888:8888 -v $PWD:/workspace resnant/bbo-rietveld:v1.3
```

4. ブラウザで http://127.0.0.1:8888/ を開くと、JupyterLabのウィンドウが表示されます（トークン入力は不要）。

5. JupyterLab上で`1_Y2O3.ipynb`または他のノートブックを開き、ノートブックのセルを実行します。

## エキスパートユーザー向け
Pythonに既に慣れている場合は、ご自身の環境でノートブック（*.ipynbファイル）を直接開いて実行できます。
以下のパッケージが必要です：
- [GSAS](https://gsas-ii.readthedocs.io/en/latest/GSASIIscriptable.html)
- [Optuna](https://optuna.readthedocs.io/en/stable/)（3.5以降）
- Jupyter LabまたはJupyter Notebook
- Pandas
- matplotlib

ノートブック内で、ディレクトリに合わせて`DATA_DIR`と`WORK_DIR`を適切に設定する必要があります。

## トラブルシューティング
- `docker compose pull` で `manifest unknown` が出る
  - 本リポジトリのタグ（例: v1.3）はローカルビルド用です。`docker compose build --no-cache` を実行してください。

- Jupyter 起動時に `JupyterEventsVersionWarning` が出る
  - 既知の警告で動作には影響ありません。どうしても抑制したい場合は Dockerfile に `ENV JUPYTER_EVENTS_NO_SCHEMA_VALIDATION=1` を追加して再ビルドしてください。

- 起動直後にカーネルが一度だけ再起動する
  - 一過性の初期化に伴う現象です。ブラウザをリロードして継続実行してください。継続的に再発する場合は issue でログをご共有ください。

- ノートブックが「Not Trusted」になる
  - JupyterLab のメニューから Trust Notebook を実行するか、端末で `jupyter trust <ipynb>` を実行してください。

### Gitコミット時にノートブック出力を除去したい
ノートブックの差分から重い出力セルを排除するには、`pre-commit` と `nbstripout` を使ったフックを設定します。

1. pre-commit のインストール（未導入の場合）
```sh
pip install pre-commit
```
2. ルートにある `.pre-commit-config.yaml` を確認（本リポジトリは既に配置済み）。内容例:
```yaml
repos:
  - repo: https://github.com/kynan/nbstripout
    rev: 0.6.1
    hooks:
      - id: nbstripout
        # 引数なし: 既定でセル出力を除去。必要に応じて --drop-empty-cells などを追加
```
3. Gitフックを有効化
```sh
pre-commit install
```
4. 既存ノート全体を一括で出力除去したい場合（任意）
```sh
pre-commit run --all-files
```

以後、`.ipynb` をコミットする際に実行結果セル(Output)が自動的に落とされ、差分が軽量になります。再現が必要な数値は CSV / JSON として別途 `work/<dataset>/artifacts/` に保存してください。

問題や質問がある場合は、GitHubでissueを開くか、メールでYuta Suzuki（resnant [at] outlook.jp）に連絡してください。
_可能な限り、メールではなくissueで質問してください。_ これにより、同様の問題に遭遇した他の人の助けになります。

## 新しい材料を追加する手順 (テンプレート `YOUR_MATERIAL.ipynb` の使い方)

既存ノート (`1_Y2O3.ipynb`, `2_DSMO.ipynb`, `3_LiCoO2.ipynb`) を参考に、テンプレート `YOUR_MATERIAL.ipynb` を編集して新しい材料の最適化を行う手順です。

### 1. 入力データの準備
`data/<材料名>` フォルダを作成し、以下のいずれかを配置します。
- 初期プロジェクト: `<MATERIAL>_init.gpx`（既存の解析状態を出発点にしたい場合）
- 原子構造ファイル: CIF (`*.cif`)
- 測定データ: CSV 形式の粉末回折データ（Y2O3ノート参照）または GSAS-II プロジェクト内のヒストグラム含有GPX
- 計測条件: PRM (`*.PRM`) ファイル（CSVと組み合わせ使用時）

### 2. テンプレートの基本設定編集
`YOUR_MATERIAL.ipynb` を開き、次のセルの変数を編集:
```python
STUDY_NAME = 'YOUR_MATERIAL'  # 材料名に変更
RANDOM_SEED = 1024            # 再現性が必要なら固定、探索多様化なら別シード
DATA_DIR = 'data/' + STUDY_NAME
WORK_DIR = 'work/' + STUDY_NAME
```
必要なら `ProjectBBO` クラスで初期GPXのコピー元や出力GPX名を修正します。

### 3. Project クラスの調整
以下を材料に応じて調整:
- 初期GPXを使うか (DSMO/LiCoO2 型) それとも CIF+CSV+PRM から新規作成するか (Y2O3 型)
- 2θ範囲 (`Limits`) の初期値と最終精密化範囲
- 異方性/配向補正など追加パラメータ (`Pref.Ori.` など)

### 4. objective 関数の実装
テンプレートでは `NotImplementedError` を投げるようにしています。既存ノートから探索空間定義ブロックをコピーし、パラメータ名の一貫性を保ちます。
- 探索空間は `trial.suggest_*` 系で宣言
- 各段階の精密化辞書 (`refdictX`) をリストにまとめ、`do_refinements` に順次適用
- Uiso (<0) やエラー時のペナルティ (例: `ERROR_PENALTY = 1e9`) を実装し、異常値を排除

### 5. 最適化の実行
`study = optuna.create_study(...)` セルで SQLite ストレージが `work/<材料名>/history_sqlite.db` に作成されます。`study.optimize(objective, n_trials=100)` の試行数は計算時間に応じ調整。

### 6. 途中経過の可視化
Rwp の推移プロットセル (`rwp_plot`) を利用し、収束挙動を把握します。必要なら動的更新版(`optuna.visualization`)を別途追加可能。

### 7. ベスト試行の解析・可視化
`study.best_trial.number` を用いて生成された GPX をロードし、リートベルトプロットを作成 (`rietveld_plot`). 範囲や縦軸スケールは材料に合わせて調整。

### 8. 成果物の保存/共有
テンプレートに追加済みのセルで以下を保存できます:
- `trials.csv`: 全試行履歴
- `best_params.json`: 最良パラメータ
- `YOUR_MATERIAL_best.cif`: 最良構造 (コメントアウトを解除して利用)
- GPX: `project_seed<seed>_trial_<best>.gpx` (直接再現可能)

推奨ディレクトリ構成例:
```
work/YOUR_MATERIAL/
  history_sqlite.db
  project_seed1024_trial_0.gpx
  ...
  project_seed1024_trial_87.gpx
  trials.csv
  best_params.json
  YOUR_MATERIAL_best.cif  # 必要なら
```

### 9. 再現性と複数シード
異なる `RANDOM_SEED` で複数回走らせ、統計的優位性を検証。集約は `pandas.concat` や Optuna Study 合併機能で可能。

### 10. ノートブック出力の管理
数値成果物は CSV / JSON / CIF に分離し、`pre-commit` によりノート出力（画像/大量テキスト）を差分から排除。レビューが軽くなり再現手順が明確になります。

### 11. よくある実装上の失敗例
| 症状 | 原因 | 対処 |
|------|------|------|
| 常に同じRwp | 精密化辞書が一段階のみ / パラメータrefineフラグFalse | refine段階を増やす / refine=Trueを確認 |
| Rwpが極端に大 | Uiso<0など物理的破綻 | Uisoチェックとペナルティ導入 |
| エラーで停止 | GPX, CIF, PRM のパス不一致 | `DATA_DIR` 内ファイル名再確認 / 相対パス統一 |
| 収束が遅い | 探索空間過大 | 範囲を実測分布に合わせ縮小 |

### 12. 次の発展
- 複数相 (multi-phase) への拡張: `add_phase` を必要数呼び出し、結合精密化。
- 並列試行: `n_jobs` や独自プロセス/マルチコンテナ活用。
- Early stopping: 目標Rwp到達時に `raise optuna.TrialPruned`。

不明点があれば issue でノート差分とログ（エラー全文）を添付してください。

### よくある質問
- ポート`8888`が他のサービスによって占有されている場合、`run.sh`を開始すると以下のエラーが発生します：
  
  ```Error starting userland proxy: listen tcp 0.0.0.0:8888: bind: address already in use.```
  
  - テキストエディタで`run.sh`を開き、ポートバインディング設定を他のポート（例：`18888`）に変更してください：
```
docker run --rm -v ${SCRIPT_DIR}/:/bbo_rietveld -p 18888:8888 -it resnant/bbo-rietveld:v1.1
```
  - または、`compose.yaml`を使用している場合は、ポート設定を変更してください：
```yaml
    ports:
      - "18888:8888"
```
  - その後、ブラウザで`http://127.0.0.1:18888/?token={TOKEN}`を開いてJupyter Notebookを開きます。

- `run.sh`を実行したときに`Unable to find image 'resnant/bbo-rietveld:v1.1' locally`エラーが発生した場合：
  - ターミナルで`docker image list`を実行し、REPOSITORYに`resnant/bbo-rietveld`があることを確認してください：
```sh
resnant@cosmos:~$ docker image list
REPOSITORY              TAG                             IMAGE ID            CREATED             SIZE
resnant/bbo-rietveld    v1.1                            ab342a0ba172        1 months ago        5.24GB
```

## ライセンス
このソフトウェアは、研究および教育目的でApache 2.0ライセンスの下で配布されています。商用目的でこのコードを使用したい場合は、対応著者に連絡してください。

`data/`ディレクトリ内の回折および結晶構造データセットは、元のライセンスの下で配布されています。詳細については、[データセットのREADME](data/README.md)を参照してください。

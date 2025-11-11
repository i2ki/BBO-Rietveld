FROM continuumio/miniconda3:23.10.0-1

# 必要なシステムライブラリ (wxPython, GSAS-IIで要求されるGUI/画像/Fortran系)
RUN apt update && apt install -y --no-install-recommends \
    curl wget git build-essential gfortran \
    libgtk2.0-dev libgtk-3-dev libjpeg-dev libtiff-dev \
    libgstreamer-plugins-base1.0-dev libnotify-dev \
    freeglut3 freeglut3-dev libsm-dev tk-dev libgfortran5 \
    libsdl2-dev libwebkit2gtk-4.0-dev libxtst-dev \
    && rm -rf /var/lib/apt/lists/*

# ヘッドレス環境での描画バックエンド（matplotlibがGUIを要求しないように）
ENV MPLBACKEND=Agg

# condaで可能な限りまとめてインストール (wxPythonをconda-forgeから取得してビルド時間短縮)
RUN conda install -y -c conda-forge mamba && conda clean -afy
RUN mamba install -y -c conda-forge \
    python=3.10 \
    wxpython=4.2.* \
    numpy pandas matplotlib scikit-learn \
    pymatgen pyopengl requests \
    ipython jupyterlab=4.2.* notebook=7.2.* jupyter_server=2.14.* \
    ipykernel=6.29.* jupyter_client=8.6.* traitlets=5.14.* \
    pyzmq=25.1.* zeromq tornado=6.4.* \
    ipywidgets tqdm nodejs jupyter_events \
    && conda clean -afy

# pipで残り (optunaは最新版をPyPIから、JupyterLabもpipで統一)
RUN pip install --no-cache-dir optuna>=3.5 && python - <<'PY'
import zmq, jupyter_client, ipykernel, tornado
print('ZMQ core/lib versions:', zmq.zmq_version(), zmq.__version__)
print('jupyter_client', jupyter_client.__version__, 'ipykernel', ipykernel.__version__, 'tornado', tornado.version_info)
try:
    import jupyter_server, notebook, jupyterlab, jupyter_events
    print('jupyter_server', getattr(jupyter_server, '__version__', 'n/a'))
    print('notebook', getattr(notebook, '__version__', 'n/a'))
    print('jupyterlab', getattr(jupyterlab, '__version__', 'n/a'))
    print('jupyter_events', getattr(jupyter_events, '__version__', 'n/a'))
except Exception as e:
    print('version check error:', e)
PY

# GSAS-II (最新mainブランチ) を git 経由で取得し、オプションパッケージ(gui,useful)含めてインストール
# 公式ドキュメント推奨: git clone → pip install .[gui,useful]
RUN git clone --depth 1 https://github.com/AdvancedPhotonSource/GSAS-II.git /root/GSAS-II \
    && pip install --no-cache-dir /root/GSAS-II[gui,useful]

# 環境変数: GSAS-IIスクリプトパスをPATHへ (pipインストールでGSAS-IIコマンド生成されるが念のため)
ENV PATH="/root/GSAS-II:$PATH"

# Jupyter Lab設定 (コンテナ外アクセス用)
RUN jupyter lab --generate-config \
    && sed -i -e "s/#c.ServerApp.ip = 'localhost'/c.ServerApp.ip = '0.0.0.0'/g" ~/.jupyter/jupyter_lab_config.py \
    && sed -i -e "s/#c.ServerApp.port = 8888/c.ServerApp.port = 8888/g" ~/.jupyter/jupyter_lab_config.py \
    && echo "c.ServerApp.allow_origin = '*'" >> ~/.jupyter/jupyter_lab_config.py \
    && echo "c.ServerApp.token = ''" >> ~/.jupyter/jupyter_lab_config.py

# 既定の作業ディレクトリ
WORKDIR /workspace

# エントリポイント例 (docker runで上書き可能)。compose.yamlでは command 指定済み。
CMD ["jupyter", "lab", "--allow-root", "--ip", "0.0.0.0", "--no-browser"]

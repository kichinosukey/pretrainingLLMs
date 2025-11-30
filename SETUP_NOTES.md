# Lecture 2: Data Preparation セットアップメモ

pretraining をローカルで進める際に環境構築でつまずいたポイントと、その対処を整理。

## 問題に気づいたきっかけ
- 何も考えずに `uv venv` を実行し、Python 3.14 の仮想環境ができていた。  
- `requirements.txt` には `torch==2.1.2` があり、配布ホイールは `cp38/39/310/311` のみ。`cp314` が無く、依存解決できないことがエラーで判明。

## 対応策（Python 3.11 環境で実行）
- 仮想環境を作り直し（3.11 にダウングレード）:  
  - `uv venv --python 3.11`  
  - `source .venv/bin/activate`

- fasttext ビルド前準備（venv 有効化状態で実行）
  1. ビルド基盤を更新:  
     `uv pip install -U pip setuptools wheel`
  2. `fasttext` が要求する依存を先に入れる:  
     `uv pip install pybind11`  
     - うまくいかない場合はバージョン固定も検討: `uv pip install pybind11==2.10.4`
  3. ビルド分離を切って `fasttext` を導入:  
     `uv pip install --no-build-isolation fasttext==0.9.2`

- 残りの依存インストール:  
  `uv pip install -r requirements.txt`  
  （環境を完全同期したい場合: `uv pip sync requirements.txt`）

## なぜこの手順が有効と判断したか
- Python 3.14 → 3.11 へ: torch 2.1.2 の公開ホイールに `cp314` が存在せず、インストール不能。対応している 3.11 へ下げるのが確実。  
- `pybind11` 事前インストール: fasttext の setup がビルド時に `pybind11` を要求し、未導入で `pybind11 install failed` が出ていたため。  
- `--no-build-isolation`: `uv` が作るビルド用一時環境には `pip`/`pybind11` が無くて失敗していた。隔離を切れば、事前に入れた依存をそのまま利用できる。  
- ビルド基盤更新: 古い `pip/setuptools/wheel` だと C++ 拡張ビルドでこけることがあるため、先に更新して安定化を図った。  
- コンパイラの準備: fasttext は C++ 拡張をビルドするので、Xcode Command Line Tools などのコンパイラが必須（未導入なら `xcode-select --install`）。

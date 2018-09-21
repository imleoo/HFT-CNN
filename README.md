HFT-CNN
==
このコードでは次の4種類のCNNモデルを用いた文書分類ができます:
* Flat モデル : 階層構造を利用せずに学習
* Without Fine-tuning (WoFt) モデル : 階層構造を利用するがFine-tuningは利用せずに学習
* Hierarchical Fine-Tuning (HFT) モデル : 階層構造とFine-tuningを利用して学習
* XML-CNN モデル ([Liu+ '17](http://nyc.lti.cs.cmu.edu/yiming/Publications/jliu-sigir17.pdf)) : Liuら'17 の提案したモデル

このコードを用いる際には次の論文をご参照ください: 

**HFT-CNN: Learning Hierarchical Category Structure for Multi-label Short Text Categorization** Kazuya Shimura, Jiyi Li and Fumiyo Fukumoto. EMNLP, 2018.


### 各モデルの特徴

|              特徴\手法 |   Flatモデル  |   WoFtモデル  |   HFTモデル   |    XML-CNNモデル    |
|-----------------------:|:-------------:|:-------------:|:-------------:|:-------------------:|
|              Hierarchycal Structure |               |       ✔       |       ✔       |                     |
|            Fine-tuning |               |       ✔       |       ✔       |                     |
|                Pooling Type | 1-max pooling | 1-max pooling | 1-max pooling | dynamic max pooling |
| Compact Representation |               |               |               |          ✔          |

## Requirements
このコードを実行するために必要なライブラリのうち、代表的なものを次に示します。
* Python 3.5.4 以降
* Chainer 4.0.0 以降 ([chainer](http://chainer.org/))
* CuPy 4.0.0 以降 ([cupy](https://cupy.chainer.org/))

注意: 
* 現在のコードのバージョンでは**GPU**を利用することが前提となっています。
* コードを実行するために必要なライブラリの詳細はrequirements.txtをご参照ください。

## Installation
* このページの **clone or download** からコードをダウンロード
* requirements.txtに書かれたライブラリをインストールし、実行環境を構築
* もし必要であれば、次の手順でAnaconda([anaconda](https://www.anaconda.com/enterprise/))による仮想環境を構築
    1. [Anacondaのダウンロードページ](https://www.anaconda.com/download/)から自分の環境にあったものをインストール
        * 例: Linux(x86アーキテクチャ, 64bit)にインストールする場合:
            1. wget https://repo.continuum.io/archive/Anaconda3-5.1.0-Linux-x86_64.sh
            1. bash Anaconda3-5.1.0-Linux-x86_64.sh
            
            でインストールできます。
    3. Anacondaをインストール後、仮想環境を構築
        ```conda env create -f=hft_cnn_env.yml```
    4. ```source activate hft_cnn_env```　で仮想環境に切り替え
    5. この環境内でHFT-CNNのコードを実行することが可能


## Quick-start
exmaple.shを実行することでFlatモデルを用いたサンプル文書(Amazon商品レビュー)の自動分類を試すことができます:
```
bash example.sh
--------------------------------------------------
Loading data...
Loading train data: 100%|███████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 465927/465927 [00:18<00:00, 24959.42it/s]
Loading valid data: 100%|█████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 24522/24522 [00:00<00:00, 27551.44it/s]
Loading test data: 100%|████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████████| 153025/153025 [00:05<00:00, 27051.62it/s]
--------------------------------------------------
Loading Word embedings...
```
学習後の結果はCNNディレクトリに保存されます.
* RESULT : テストデータを分類した結果
* PARAMS : 学習後のCNNのパラメータ
* LOG : 学習のログファイル

### 学習モデルの変更
```example.sh```内の ```ModelType``` を変更することで学習するモデルを変更することができます
```                                                                                                                 
## Network Type (XML-CNN,  CNN-Flat,  CNN-Hierarchy,  CNN-fine-tuning or Pre-process)
ModelType=XML-CNN
```
注意: 
* CNN-Hierarchy, CNN-fine-tuningを選択する場合には**Pre-process**で学習をしてから学習を行ってください
* Pre-processでは階層構造の第1階層目のみを学習し、CNNのパラメータを保存します
* このときに保存されたパラメータはCNN-Hierarchy, CNN-fine-tuningの両タイプで共有されます

### 単語の分散表現について
このコードでは単語の分散表現に[fastText](https://github.com/facebookresearch/fastText)の学習結果を利用しています.

```example.sh```内の```EmbeddingWeightsPath```に単語埋め込み層の初期値として利用したいfastTextの```bin```ファイルを指定することができます。

fastTextの```bin```ファイルを用意していない場合、英語Wikipediaコーパスを用いた単語の分散表現が[chakin](https://github.com/chakki-works/chakin)を用いて自動的にダウンロードされます。

コードに手を加えず```example.sh```を実行した場合にはWord_embeddingディレクトリに```wiki.en.vec```がダウンロードされ、これが利用されます。
```
## Embedding Weights Type (fastText .bin and .vec)
EmbeddingWeightsPath=./Word_embedding/
```




## 新しいデータでモデルを学習
### データについて
#### 種類
必要な文書データは3種類です:
* 訓練データ : CNNを学習させるために必要なデータ
* 評価データ: CNNの汎化性能を検証するために必要なデータ
* テストデータ : CNNを用いて分類したいデータ

評価データは各エポックごとにCNNの汎化誤差を評価する際に用いられ、学習の継続によって過学習が起きた場合にEarly Stoppingを行います. また保存されるCNNのパラメータは汎化誤差が最も小さい時のエポックのものが保存されます．

#### 形式

### 文書データが階層構造を有する場合

## License
    





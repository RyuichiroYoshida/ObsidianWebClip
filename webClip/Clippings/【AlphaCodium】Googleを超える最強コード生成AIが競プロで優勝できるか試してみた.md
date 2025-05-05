---
title: 【AlphaCodium】Googleを超える最強コード生成AIが競プロで優勝できるか試してみた
source: https://weel.co.jp/media/tech/alphacodium/
author:
  - "[[WEEL]]"
published: 2024-04-02
created: 2025-05-06
description: WEELメディア事業部LLMリサーチャーの中田です。 1月16日、LLMによる高度なプログラミングコード生成を可能にするプロンプトエンジニア「AlphaCodium」を、CodiumAIが提案しました。 この手法を用いることで、従来のプロン
tags:
  - Tech
  - Tools
  - "#AI"
read:
---
MENU

[LLM](https://weel.co.jp/media/tech-category/text-generation/) [その他](https://weel.co.jp/media/tech-category/other/) [オープンソースAI](https://weel.co.jp/media/tech-category/oss/) [生成AIずかん](https://weel.co.jp/media/tech-category/ai-dictionary/)

![AlphaCodium Google コード生成AI](https://weel.co.jp/wp-content/uploads/2024/01/AI%E3%83%A2%E3%83%B3_AlphaCodium.jpg)

AlphaCodium Google コード生成AI

WEELメディア事業部LLMリサーチャーの中田です。

1月16日、 **LLMによる高度なプログラミングコード生成を可能にするプロンプトエンジニア「AlphaCodium」を、** CodiumAIが提案しました。

**この手法を用いることで、従来のプロンプト手法よりも格段に、LLMのコード生成能力を向上できるんです、、、！**

![](https://raw.githubusercontent.com/Codium-ai/codiumai-vscode-release/main/media/docs/Tests-Gif.gif)

例えば、本技術を使えば、上記のようなコード生成も可能になります（ちなみにこれは、AlphaCodiumの会社が開発した、VS Codeの自動コード生成の拡張機能で、コードを生成している様子）。

GitHubでのスター数は、すでに1200を超えており、注目度が高いことを示しています。

この記事ではAlphaCodiumの使い方や、有効性の検証まで行います。本記事を熟読することで、AlphaCodiumの凄さを実感し、普通の手法による自動コード生成には戻れなくなるでしょう。

ぜひ、最後までご覧ください。

目次
1. [AlphaCodiumの概要](https://weel.co.jp/media/tech/alphacodium/#index_id0)
2. [AlphaCodiumの料金体系](https://weel.co.jp/media/tech/alphacodium/#index_id1)
3. [AlphaCodiumの使い方](https://weel.co.jp/media/tech/alphacodium/#index_id2)
	1. [AlphaCodiumを動かすのに必要なPCのスペック](https://weel.co.jp/media/tech/alphacodium/#index_id3)
4. [AlphaCodiumを実際に使ってみた](https://weel.co.jp/media/tech/alphacodium/#index_id4)
5. [AlphaCodiumの推しポイントである超強力コード生成は本当なのか？](https://weel.co.jp/media/tech/alphacodium/#index_id5)
6. [まとめ](https://weel.co.jp/media/tech/alphacodium/#index_id6)
7. [最後に](https://weel.co.jp/media/tech/alphacodium/#index_id7)
8. [投稿者](https://weel.co.jp/media/tech/alphacodium/#index_id8)

## AlphaCodiumの概要

**LLMにコードを生成させるタスクにおけるプロンプトエンジニアリングとして、「Flow Engineering」が以下の論文において提唱されました。**

**参考記事： [Code Generation with AlphaCodium: From Prompt Engineering to Flow Engineering](https://arxiv.org/abs/2401.08500)**

本研究の基本的なアイデアとしては、 **「複数の段階に分けて、コードの生成と改善を繰り返す」** というもの。

**そして、今回のAlphaCodiumとは、このFlow Engineeringをオープンソース化したツールです。** 具体的には、以下の手順でコード生成が行われます。

1. 前処理
2. 反復処理
![](https://github.com/Codium-ai/AlphaCodium/raw/main/pics/proposed_flow.png)

上図の左側が前処理工程を表し、以下の手順で処理が行われます。

1. 問題を用意
2. 複数の解答を生成する
3. 解答をランク付け
4. 検証テストを行う
5. 追加テストを作成

また、上図の右側が反復処理を表し、以下の手順で処理が行われます。

1. 最初のコードを生成
2. その解答をテストする
3. 結果をもとに改善
4. AIテストでさらに反復

この手法を利用することで、LLMの性能を大幅に向上できたそう。

ただし、本手法は競技プログラミングの問題に特化しているため、開発にそのまま応用するには少し工夫が必要だそうです。

AlphaCodiumはGoogle DeepmindのAlphaCodeを超えていると言われていて、その精度に期待が高まりますね…！

[![生成AIずかん](https://weel.co.jp/wp-content/uploads/2024/02/%E7%94%9F%E6%88%90AI%E3%81%9A%E3%81%8B%E3%82%93B-1.png)](https://weel.co.jp/ai-visual-dictionary/)

## AlphaCodiumの料金体系

AlphaCodiumはオープンソースであるため、誰でも **無料** で利用可能です。

なお、Metaの最強コード生成AIについて知りたい方はこちらの記事をご覧ください。  
→ [**【CodeLlama-70B】700億パラメーターコード生成AIをGPT-4と比較してみた**](https://weel.co.jp/media/codellama-70b)

## AlphaCodiumの使い方

今回は、Google ColabのT4を用いました。

まず、以下のコードを実行して、必要なライブラリをインストールしましょう。

```python
!git clone https://github.com/Codium-ai/AlphaCodium.git
%cd AlphaCodium
!pip install -r requirements.txt
```

ここでいったん、ランタイムの再起動。続いて、 **「alpha\_codium/settings/.secrets\_template.toml」** を同じフォルダ内で複製し、そのファイル名を「.secrets.toml」としましょう。そしてそのファイル内の以下の部分に、OpenAIのKeyを入れてください。

```python
[openai]
key = "..."
```

次に、以下のページから、データセットをダウンロードし、alpha\_codiumのルートディレクトリの直下に置きましょう。

**参考記事： [talrid/CodeContests\_valid\_and\_test\_AlphaCodium](https://huggingface.co/datasets/talrid/CodeContests_valid_and_test_AlphaCodium/blob/main/codecontests_valid_and_test_processed_alpha_codium.zip)**

最後に、以下のコードを実行すれば、コードが生成されます。

```python
!python -m alpha_codium.solve_problem \
--dataset_name /path/to/dataset \
--split_name test \
--problem_number 0
```

「dataset\_name」には、先ほどダウンロードしたデータセットフォルダへのパスを入力しましょう。また、「split\_name」にはvalidまたはtestを指定します。

また、YAMLの構造化された形式で出力されます。

### AlphaCodiumを動かすのに必要なPCのスペック

■Pythonのバージョン  
Python 3.8以上  
  
■必要なパッケージ  
dynaconf==3.1.12  
fastapi==0.99.0  
PyGithub==1.59.\*  
retry==0.9.2  
Jinja2==3.1.2  
tiktoken==0.4.0  
uvicorn==0.22.0  
pytest==7.4.0  
aiohttp==3.8.4  
atlassian-python-api==3.39.0  
GitPython==3.1.32  
PyYAML==6.0  
starlette-context==0.3.6  
boto3==1.28.25  
google-cloud-storage==2.10.0  
ujson==5.8.0  
azure-devops==7.1.0b3  
msrest==0.7.1  
openai  
litellm  
duckdb  
datasets  
notebook  
black  
evaluate  
click  
code\_contests\_tester==0.1.6  
aiolimiter  
Jinja2  
tqdm  
pysnooper  
loguru  
numpy  
retry

[![生成AIずかん](https://weel.co.jp/wp-content/uploads/2024/02/%E7%94%9F%E6%88%90AI%E3%81%9A%E3%81%8B%E3%82%93B-1.png)](https://weel.co.jp/ai-visual-dictionary/)

## AlphaCodiumを実際に使ってみた

ここでは、 **Polycarp and Coins** という問題をAlphaCodiumに解いてもらおうと思います。

**参考記事： [Polycarp and Coins](https://codeforces.com/problemset/problem/1551/A)**

問題の概要は以下の通り。

> 「Polycarp and Coins」は、Polycarpが1と2のバーレス硬貨を使って正確にnバーレスを支払う必要がある問題です。
> 
> 彼（Polycarp）は使用する各種硬貨の枚数の差を最小限に抑えたいと考えています。
> 
> 目標は、1バーレス硬貨と2バーレス硬貨の枚数を表す二つの非負整数 *c* 1 と *c* 2を見つけ、c1+2⋅c2=nかつ∣ *c* 1− *c* 2∣ を最小化することです。
> 
> この問題は複数のテストケースを含み、各ケースで異なるnの値が与えられます。各シナリオに対して最適な硬貨の組み合わせを効率的に計算する必要があります。
> 
> **[Polycarp and Coins](https://codeforces.com/problemset/problem/1551/A)**

つまり、次の条件を満たすc\_1とc\_2を見つける必要があります。

- c\_1 + 2c\_2 = n
- |c\_1 – c\_2|を最小限にする

例えば、n = 1000の場合、C1=334でC2=333となるのが正解です。具体的な計算式は、以下の通りです。

1. nが3の倍数の場合、c1とc2を等しくする
2. nが3で割った余りが1の場合、c1を1多くする
3. nが3で割った余りが2の場合、c2を1多くする

実際のAlphaCodiumの出力は、以下のようなYAML形式です。

![](https://www.codium.ai/wp-content/uploads/2024/01/example_output_tokens_yaml-1-690x605.png)

このような形式の出力を、自力で整形したPythonコードは、以下の通りです。

```python
# nを読み込む
n = int(input())

# nを3で割った余りに基づいて処理を行う
r = n % 3

if r == 0:
    # nが3の倍数の場合、c1とc2を等しくする
    print(n // 3, n // 3)

elif r == 1:
    # nが3で割った余りが1の場合、c1を1多くする
    print((n // 3) + 1, n // 3)

else:
    # nが3で割った余りが2の場合、c2を1多くする
    print(n // 3, (n // 3) + 1)
```

試しに、n=1000と入力したところ、以下の様に出力されました。

```python
334 333
```

**正解ですね！**

導入までが少しめんどくさいですが、精度は高いようです。

## AlphaCodiumの推しポイントである超強力コード生成は本当なのか？

ここでは、 **Xor Distance** という、実際の競技プログラミングで出題された問題を、AlphaCodiumとGPT-4に解かせて、両者の出力を比較してみます。

**参考記事： [H. XOR and Distance](https://codeforces.com/problemset/problem/1553/H)**

具体的には、 **「与えられた配列の各要素に0から2^k-1までの各整数をXOR演算することで、新しい配列を生成する」** ような問題を、AlphaCodiumを使って解いてみます。

アルゴリズムとしては、以下の通りです。

1. 新しい配列内の異なる2つの要素間の絶対差の最小値を求める
2. この最小絶対差を、すべての整数（0から2^k-1）について計算し、その結果を出力
3. 結果は、それぞれの整数に対する最小絶対差の値のリストとして表示

入力例は、以下の通りです。

- 入力: n = 3, k = 3, 配列 = \[6, 0, 3\]
- x = 0の場合、配列の要素にXORを適用すると\[6, 0, 3\]になり、2つの要素の最小絶対差は3
- x = 1の場合、配列の要素にXORを適用すると\[7, 1, 2\]になり、最小絶対差は1。
- 以下同様に、x = 2からx = 7まで計算

回答例としては、例えば上記のようにn=3, k=3で、配列a=\[603\]が入力された場合、 **\[3 1 1 2 2 1 1 3\]** と出力されてほしいです。

この情報をAlphaCodiumに入力し、出力された内容を自力でPythonに直した結果が、以下の通りです。

```python
n, k = map(int, input().split())
a = list(map(int, input().split()))

d = [0] * (1 << k)

for i in range(1 << k):
    s = set()
    for j in range(n):
        s.add(i ^ a[j])
    
    s = list(s)
    s.sort()
    d[i] = min([s[j] - s[j - 1] for j in range(1, len(s))])

print(*d)

# 出力結果
3 1 1 2 2 1 1 3
```

続いて、GPT-4（ChatGPTの有料版）に解かせた結果は、以下の通りです。

```python
def min_xor_distance(arr, k):
    max_val = 2 ** k
    result = []

    for x in range(max_val):
        # 各xに対して配列の要素とXOR演算を行い、新しい配列を生成
        xor_array = [a ^ x for a in arr]

        # 新しい配列内の異なる要素間の最小絶対差を求める
        xor_array.sort()
        min_diff = min(abs(xor_array[i] - xor_array[i + 1]) for i in range(len(xor_array) - 1))

        result.append(min_diff)

    return result

# 入力例
n, k = 3, 3
arr = [6, 0, 3]

# 関数を実行して結果を出力
print(min_xor_distance(arr, k))

# 出力結果
[3, 1, 1, 2, 2, 1, 1, 3]
```

**どちらも、正解を出力することができました！**

ただ、導入までの労力や難易度を考えると、やはりChatGPTの有料版を使う方が、はるかに楽で便利です。

そのため、以下のような使い方をおすすめします。

- 開発経験があり、無料で利用したい方：AlphaCodium
- 開発未経験、もしくはサクッと導入したい方：GPT-4

[![生成AIずかん](https://weel.co.jp/wp-content/uploads/2024/02/%E7%94%9F%E6%88%90AI%E3%81%9A%E3%81%8B%E3%82%93B-1.png)](https://weel.co.jp/ai-visual-dictionary/)

## まとめ

**LLMにコードを生成させるタスクにおけるプロンプトエンジニアリングとして、「Flow Engineering」が以下の論文において提唱されました。**

**参考記事： [Code Generation with AlphaCodium: From Prompt Engineering to Flow Engineering](https://arxiv.org/abs/2401.08500)**

本研究の基本的なアイデアとしては、 **「複数の段階に分けて、コードの生成と改善を繰り返す」** というもの。そして、今回のAlphaCodiumとは、このFlow Engineeringをオープンソース化したツールです。

ただ、導入までの労力や難易度を考えると、やはりChatGPTの有料版を使う方が、はるかに楽で便利です。

そのため、以下のような使い方をおすすめします。

- 開発経験があり、無料で利用したい方：AlphaCodium
- 開発未経験、もしくはサクッと導入したい方：GPT-4

数年後には、誰もが天才プログラマーとして、高度なデジタル世界を実現するようになっているのかもしれない **ですね。**

また、近年は超小型のLLMも増えてきています。そのため、例えばスマホから声だけでコードを生成し、それをスマホ上で実行できるよになっているかもしれません。

![サービス紹介資料](https://weel.co.jp/wp-content/uploads/2023/11/servise_thumbnail-1024x575.jpeg)

**生成系AIの業務活用なら！**

・生成系AIを活用したPoC開発

・生成系AIのコンサルティング

・システム間API連携

## 最後に

いかがだったでしょうか？

GPT-3.5 Turboの最新アップデートで、より高速かつ低コストでのAI活用が可能になりました。自社での導入・活用を検討する際に、最適なモデル選定や活用方法について、一緒に考えてみませんか？

弊社では

・マーケティングやエンジニアリングなどの **専門知識を学習させたAI社員の開発**  
・要件定義・業務フロー作成を **80%自動化** できる **自律型AIエージェントの開発**  
・生成AIとRPAを組み合わせた **業務自動化ツールの開発**  
・社内人事業務を **99%自動化** できる **AIツールの開発**  
・ **ハルシネーション対策AIツールの開発**  
・ **自社専用のAIチャットボットの開発**

などの開発実績がございます。

まずは、 **「無料相談」** にてご相談を承っておりますので、ご興味がある方はぜひご連絡ください。

[➡︎生成AIを使った業務効率化、生成AIツールの開発について相談をしてみる。](https://weel.co.jp/company/contact/)

生成AIを社内で活用していきたい方へ

![無料相談](https://weel.co.jp/wp-content/uploads/2024/10/A-proprietary-company-chatbot-with-employees-using-it-while-facing-their-computers-and-a-robot-emerging-from-the-PC-1024x538-1.webp)

**「生成AIを社内で活用したい」「生成AIの事業をやっていきたい」** という方に向けて、生成AI社内セミナー・勉強会をさせていただいております。

セミナー内容や料金については、ご相談ください。

また、サービス紹介資料もご用意しておりますので、併せてご確認ください。

## 投稿者

[LLM](https://weel.co.jp/media/tech-category/text-generation/) [その他](https://weel.co.jp/media/tech-category/other/) [オープンソースAI](https://weel.co.jp/media/tech-category/oss/) [生成AIずかん](https://weel.co.jp/media/tech-category/ai-dictionary/)
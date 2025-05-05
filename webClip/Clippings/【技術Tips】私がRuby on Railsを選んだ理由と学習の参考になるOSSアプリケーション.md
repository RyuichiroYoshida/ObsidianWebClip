---
title: 【技術Tips】私がRuby on Railsを選んだ理由と学習の参考になるOSSアプリケーション
source: https://www.tech-street.jp/entry/2024/08/30/120030
author:
  - "[[TECH Street (テックストリート)]]"
published: 2024-08-30
created: 2025-05-06
description: はじめに こんにちは、オシロ株式会社でリードエンジニアとして働いているにっく（webuilder240）と申します。オシロでは自社プロダクトとしてコミュニティ専用オウンドプラットフォーム「OSIRO」を提供していますが、私は2015年の開発開始から一人目エンジニアとして携わり、技術選定の意思決定を行ってきました。 今回は、そのなかでもRuby on Railsを選択した理由、 その学習に役立つOSSアプリケーションについて紹介したいと思います。この記事を読むことで、Railsの選定理由や実践的な学習方法について理解を深めていただければと思います。 はじめに Ruby on Railsの選択理由…
tags:
  - 1
  - Ruby
  - Tools
  - バックエンド
read: false
---
![](https://cdn-ak.f.st-hatena.com/images/fotolife/p/pcads_media/20240827/20240827171519.jpg)

#### はじめに

こんにちは、オシロ株式会社でリードエンジニアとして働いているにっく（ [webuilder240](https://x.com/webuilder240) ）と申します。オシロでは自社プロダクトとしてコミュニティ専用オウンドプラットフォーム「OSIRO」を提供していますが、私は2015年の開発開始から一人目エンジニアとして携わり、技術選定の意思決定を行ってきました。

今回は、そのなかでもRuby on Railsを選択した理由、 その学習に役立つOSSアプリケーションについて紹介したいと思います。この記事を読むことで、Railsの選定理由や実践的な学習方法について理解を深めていただければと思います。

- [はじめに](https://www.tech-street.jp/entry/2024/08/30/#%E3%81%AF%E3%81%98%E3%82%81%E3%81%AB)
- [Ruby on Railsの選択理由](https://www.tech-street.jp/entry/2024/08/30/#Ruby-on-Rails%E3%81%AE%E9%81%B8%E6%8A%9E%E7%90%86%E7%94%B1)
	- [開発に必要な便利機能がはじめからそろっていた](https://www.tech-street.jp/entry/2024/08/30/#%E9%96%8B%E7%99%BA%E3%81%AB%E5%BF%85%E8%A6%81%E3%81%AA%E4%BE%BF%E5%88%A9%E6%A9%9F%E8%83%BD%E3%81%8C%E3%81%AF%E3%81%98%E3%82%81%E3%81%8B%E3%82%89%E3%81%9D%E3%82%8D%E3%81%A3%E3%81%A6%E3%81%84%E3%81%9F)
	- [可読性・コードの美しさ](https://www.tech-street.jp/entry/2024/08/30/#%E5%8F%AF%E8%AA%AD%E6%80%A7%E3%82%B3%E3%83%BC%E3%83%89%E3%81%AE%E7%BE%8E%E3%81%97%E3%81%95)
	- [選び続けている理由](https://www.tech-street.jp/entry/2024/08/30/#%E9%81%B8%E3%81%B3%E7%B6%9A%E3%81%91%E3%81%A6%E3%81%84%E3%82%8B%E7%90%86%E7%94%B1)
- [RailsのコードリーディングにおすすめなOSS](https://www.tech-street.jp/entry/2024/08/30/#Rails%E3%81%AE%E3%82%B3%E3%83%BC%E3%83%89%E3%83%AA%E3%83%BC%E3%83%87%E3%82%A3%E3%83%B3%E3%82%B0%E3%81%AB%E3%81%8A%E3%81%99%E3%81%99%E3%82%81%E3%81%AAOSS)
	- [Mastodon](https://www.tech-street.jp/entry/2024/08/30/#Mastodon)
	- [forem](https://www.tech-street.jp/entry/2024/08/30/#forem)
	- [Writebook](https://www.tech-street.jp/entry/2024/08/30/#Writebook)
- [最後に](https://www.tech-street.jp/entry/2024/08/30/#%E6%9C%80%E5%BE%8C%E3%81%AB)

### Ruby on Railsの選択理由

先日、 [TECH Street主催のRuby勉強会](https://www.tech-street.jp/entry/2024/05/31/122432) に登壇させていただいたときに、 『Ruby, Ruby on Railsを選んだ理由は何ですか？』という質問を受けました。

他の登壇者の方々は、既にRailsを採用した後に引き継がれた立場の方だったので、開発開始の技術選定で明確にRuby on Railsを選ぶ意思決定をしたのは私だけでした。 そんなこともあり、今回は選定理由について詳しくお話ししてみたいと思います。

#### 開発に必要な便利機能がはじめからそろっていた

Ruby on Railsを利用し始めたのはコミュニティ専用オウンドプラットフォーム「OSIRO」の開発を始めたときからなので、今年で9年目を迎えます。 その時にはすでに現在Railsで広く使われている機能やライブラリはリリースされていて、Webアプリケーション開発で必要なものは多く揃っていました。

当時、私が使っていたCakePHP2ではサポートされていなかった、DBマイグレーション機能やRails Consoleも当たり前に使えるのは魅力的でした。 外部ライブラリも当時から豊富で、今でもテストツールとして広く利用されるRSpecやActiveJobのバックエンドとしてよく使われるSidekiq、ページネーションライブラリであるKaminariと、Railsだけでできなくても定番ライブラリが揃っていました。 当時CakePHPには定番ライブラリが少なく、ひとりでプロダクト開発をしていた私にとっては、定番ライブラリや機能を使って開発を効率的に進めることができるのは大変魅力的だったのです。

#### 可読性・コードの美しさ

Railsの機能だけでなく、その可読性やコードの美しさも選定の大きな理由の一つでした。特に、CakePHP2と比較したときのコードのシンプルさは、私にとって大きな魅力でした。以下に具体的なCakePHP2のコードと比較した例を示します。

CakePHPでは以下のようなコードで投稿の編集を行います。このコードは条件チェックやフラッシュメッセージの設定が冗長でした。

```go
public function edit($id = null) {
    if (!$this->Post->exists($id)) {
        throw new NotFoundException(__('Invalid post'));
    }

    if ($this->request->is(array('post', 'put'))) {
        if ($this->Post->save($this->request->data)) {
            $this->Session->setFlash(__('The post has been saved.'));
            return $this->redirect(array('action' => 'index'));
        } else {
            $this->Session->setFlash(__('The post could not be saved. Please, try again.'));
        }
    } else {

        $options = array('conditions' => array('Post.' . $this->Post->primaryKey => $id));
        $this->request->data = $this->Post->find('first', $options);
    }

}
```

一方、Railsでは以下のようにシンプルに記述できます。 Railsのコードは文字量が少なく可読性が高いことが特徴です。

```go
def edit
  @post = Post.find(params[:id])  
end

def update
  @post = Post.find(params[:id])
  if @post.update(post_params)
    redirect posts_path, notice: "The post has been saved."
  else
    flash[:now] = "The post could not be saved. Please, try again."
    render :edit
  end
end
```

これは少し極端な例かもしれませんが、RailsのコードはCakePHPよりもシンプルで読みやすく、記述量が少なく保守性が高いです。当時の私はそこに感銘を受けたのと、CakePHP2がCakePHP3へのバージョンアップにより、大幅に構文が変わってしまうことがあって、CakePHPからの乗り換えを決めました。

#### 選び続けている理由

昨今はTypeScript、GoやKotlinといった静的型付け言語がモダンな技術とされていますが、 私は依然としてRailsを選び続けています。長年の経験と愛着があるのもありますが、多くのライブラリは今でもサポートされていて、安定的に利用できるためです。DeviseやKaminari、Sidekiqなども安定して利用できています。安定したフレームワークとライブラリを使うことは、競争が激しいスタートアップにおいて、スピード感をもってプロダクト開発をする上でも非常に重要な要素です。 このように、Railsは長年利用されてきた便利なライブラリがサポートされている点や安定感の高さがあるため、オシロでは今後もRailsでWebアプリケーション開発を続けるつもりです。

### RailsのコードリーディングにおすすめなOSS

#### Mastodon

当時のTwitter（現: X)の代替サービスとして、一時的に流行していた分散型ソーシャルネットワークサービスのマストドンです。

[mastdon](https://joinmastodon.org/ja)

[GitHubリポジトリ](https://github.com/mastodon/mastodon)

これも実はRailsでできています。 簡単な構成を説明すると、バックエンド処理のほとんどをRailsでのWebAPIで実装していて、フロントエンドやView部分はReactで実装されています。 Railsの内部にReactでのSPAがあるという技術構成で、こういった構成は他の案件でも参考になるかもしれません。 その際、RailsとReactがどのようにして共存しているか？などの観点から見るとヒントになる点が多く、 バックエンド部分については良い意味で「普通のRailsアプリケーション」という感じで、すごくマジカルなコードも少なく、読みやすいと思っています。

#### forem

登壇でも紹介させていただいた、DataUpdateScriptのトピックで紹介した技術コミュティOSSで、 dev.toがこのForemで稼働しています。 QiitaやZennに近いようなサービスで、 私が近年特にRailsアプリケーションとして注目し、参考にしているOSSです。

[dev.to](https://dev.to/)

[GitHubリポジトリ](https://github.com/forem/forem)

特徴としては先述のMastdonとは異なり、RailsがAPIに徹しているわけではなく、部分的にRailsでSSR（サーバーサイドレンダリング）もしています。 これはisland Architectureを採用してPreactとRailsでのSSRを組み合わせて実装されています。

foremのisland Architectureについては、 こちらの記事で詳しく解説しているので、良ければこちらもご覧ください。 [Ruby on RailsのMPAでForem流の"Islands Architecture"を導入するメリット・デメリット](https://zenn.dev/osiro/articles/d9c886c2959b79)

バックエンドの処理も比較的ごく普通なRailsアプリケーションで、読みやすいかと思います。 他にもパフォーマンス改善に対する工夫が随所で行われているプロダクトです。

#### Writebook

現在Railsの作成者であるDHH（David Heinemeier Hansson）がCTOとして在籍している37Signalsが、積極的に進めているプロジェクトとして [once](https://once.com/) というものがあります。

これは買い切りのアプリケーションをユーザーが購入し、サーバーにユーザーがインストールして利用するような仕組みが特徴です。 購入したアプリケーションのコードは読むことができ、実際にRuby on Railsの教材としても購入が推奨されていました。私もCampfireというチャットアプリケーションを購入してコードリーディングをしていました。 [https://once.com/campfire](https://once.com/campfire)

その中に [Writebook](https://once.com/writebook) というWeb上で本の執筆ができるというアプリケーションがあります。

このプロダクトは無料で、Railsのメンテナーが数多く在籍する37signalsのRailsのプロダクションコードを読むことができます。

ただ残念なことに現時点でエディタのマルチバイト文字の対応がうまくできておらず、日本語の入力が満足にできない状況です。

それでも37signalsのプロダクトのコードはとても洗練されていて、内部品質がとても高く、Railsのお手本のようなコードを見れるのはやはり魅力的です。 特にHotwireをフロントエンドで利用しているOSSアプリケーションのコードはあまりないので、Hotwireを活用しているエンジニアにとっては「答え合わせ」ができるので、必読かと思っています。

### 最後に

Ruby on Railsは今でも現役のWebアプリケーションフレームワークで、多くの開発者にとって強力なツールです。この記事を通じてRailsの選定理由や学習に役立つOSSアプリケーションについて少しでも理解を深めていただけたなら幸いです。

記事執筆者

![*](https://cdn-ak.f.st-hatena.com/images/fotolife/p/pcads_media/20240530/20240530201252.jpg)

西尾 拓也（にっく）

オシロ株式会社  
プロダクト開発部 リードエンジニア

OSIROのプロダクトを作る1人目のエンジニアとして創業前からジョイン。OSIROのシステムの土台を担当しRuby on Railsで開発。 会社の拡大に伴い、別々だったシステムを1つのプラットフォームに統合するマルチテナント化など、様々な技術課題の解決や機能改善を行う。 2019年からリードエンジニアとして現在は6名のエンジニアを統括しており、自分もチームメンバーもエンジニアリングを楽しめるように日々工夫している。
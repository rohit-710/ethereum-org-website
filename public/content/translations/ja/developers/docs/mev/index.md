---
title: 最大抽出可能価値（MEV）
description: 最大抽出可能価値（MEV）という概念を紹介する。
lang: ja
---

最大抽出可能価値（MEV）とは、特定のブロックにおけるトランザクションの追加、削除、または順序変更により、ブロックの生成時において標準的なブロック報酬やガス代を超過して抽出できる最大の価値を指します。

## 採掘可能価値（MEV） {#miner-extractable-value}

最大抽出可能価値（MEV）は、 [プルーフ・オブ・ワーク](/developers/docs/consensus-mechanisms/pow/)に基づき導入された概念であり、当初は「採掘可能価値（MEV）」と呼ばれていました。 プルーフ・オブ・ワークでは、トランザクションの追加、削除、および順序決定をマイナーが管理していたためです。 しかし、[マージ](/roadmap/merge)によるプルーフ・オブ・ステークへの移行後、バリデータがこれらの役割を担うようになり、マイニングはイーサリアムのプロトコルから削除されました。 しかし、価値採掘のための手段は依然として存在するため、現在は「最大抽出可能価値」という用語を用います。

## 前提知識 {#prerequisites}

[トランザクション](/developers/docs/transactions/) 、[ブロック](/developers/docs/blocks/)、[プルーフ・オブ・ステーク](/developers/docs/consensus-mechanisms/pos)、および[ガス](/developers/docs/gas/)について理解している必要があります。 また、[分散アプリケーション（Dapp）](/dapps/)や[分散型金融（DeFi）](/defi/)の知識も有用です。

## MEVの抽出 {#mev-extraction}

理論的に言えば、利益を伴うMEVの抽出の実行を保証できるのはバリデータのみであるため、MEVが蓄積するのはバリデータにおいてのみです。 しかし実際には、抽出されるMEVの大部分は「サーチャー」と呼ばれる独立したネットワーク参加者が獲得しています。 サーチャーは、ブロックチェーンのデータに対して複雑なアルゴリズムを実行して利益を伴うMEVの抽出機会を検出し、ボットを利用してこれらの利益を伴うトランザクションをネットワークに自動的に送信します。

サーチャーは、高額のガス代を（バリデータに）支払ってでも、発見した利益が期待できるトランザクションがブロックに追加される可能性を高めたいと考えるため、バリデータは発生したMEV全体からその一部を獲得することができます。 サーチャーが経済的に合理的に行動すると仮定した場合、サーチャーが支払いたいと考えるガス代は、サーチャーが獲得できるMEVの100%までになります（ガス代がMEVの100%を越えれば、サーチャーは赤字になるため）。

ただし、[分散型取引所（DEX）の裁定取引](#mev-examples-dex-arbitrage)のように競争が激しいMEVの抽出機会の場合、非常に多くのユーザーが利益を伴う裁定取引を実行したいと考えるため、サーチャーは、このMEVにおける全収益の90%あるいはそれ以上をバリデータに支払う必要がある場合があります。 裁定取引が実行できる唯一の保証は、最も高いガス代を持つトランザクションを送信することだからです。

### ガスゴルフ {#mev-extraction-gas-golfing}

上記の力学に基づき、「ガス・ゴルフ（gas golfing）」（ガス代が最も安価になるようにトランザクションをプログラミングすること）の能力が競合上の優位性として確立されています。ガス・ゴルフにより、サーチャーはより高価なガス代を設定しつつ、ガス代の合計額を一定に抑えられるためです（ガス代＝ガス価格×使用ガス量）。

よく知られているガス・ゴルフのテクニックとしては、以下があります：多くのゼロを持つ文字列で始まるアドレス (例: [0x0000000000C521824EaFf97Eac7B73B084ef9306](https://etherscan.io/address/0x0000000000c521824eaff97eac7b73b084ef9306)) を使う方法では、保存スペースが小さくなるためガス代も低くなります。あるいは、少額の[ERC-20](/developers/docs/standards/tokens/erc-20/) トークン残高をコントラクトに残しておく方法もありますが、これは（残高が0の場合に）ストレージスロットを初期化すると、 ストレージスロットを更新する場合よりも多くのガスが消費される点を利用したものです。 サーチャーのコミュニティでは、ガスの使用量を減らすためのテクニックが活発に研究されています。

### 一般化されたフロントランナー {#mev-extraction-generalized-frontrunners}

一部のサーチャーは、利益を伴うMEV機会を検出するために複雑なアルゴリズムを開発する代わりに、汎用的なフロントランナーを実行します。 汎用フロントランナーとは、利益を伴うトランザクションを検出するためにメモリプールを監視するボットです。 汎用フロントランナーは、潜在的に利益を伴うトランザクションのコードをコピーし、フロントランナーのアドレスで置き換えた上でトランザクションをローカルで実行することで、修正後のトランザクションがフロントランナーのアドレスに収益をもたらすことをダブルチェックします。 このトランザクションが実際に利益をもたらすことが確認できれば、フロントランナーはアドレスを置き換えた修正後のトランザクションをより高いガス代と共に送信し、本来のトランザクションよりも「優位に立つ」ことで、本来のサーチャーが得ようとしたMEVを獲得することができるのです。

### Flashbots {#mev-extraction-flashbots}

フラッシュボットは、実行クライアントを拡張する独立したプロジェクトであり、サーチャーに対して、パブリックのメモリプールに公開することなく、MEVトランザクションをバリデータに送信できるサービスを提供します。 このサービスを使用すれば、汎用フロントランナーを用いたトランザクションに実行を先回りされることを防ぐことができます。

## MEVの実例 {#mev-examples}

ブロックチェーンにおいてMEVが発生するケースは、いくつかの種類があります。

### DEX裁定取引 {#mev-examples-dex-arbitrage}

[分散型取引所 (DEX) ](/glossary/#dex)における裁定取引は、最もシンプルでよく知られているMEVの機会です。 このため、ユーザー間の競争も最も激しくなっています。

具体的には、以下のように発生します：2カ所のDEXが同じトークンを異なる価格で提供している場合、より安価に設定したDEXでトークンを購入し、より高価に設定したDEXで売却すれば、1回の不可分な取引で利益を得ることができます。 ブロックチェーンの仕組みにより、全くリスクを伴わない裁定取引が可能になります。

[これは](https://etherscan.io/tx/0x5e1657ef0e9be9bc72efefe59a2528d0d730d478cfc9e6cdd09af9f997bb3ef4)、UniswapおよびSushiswapにおけるETH/DAIペアの価格差を利用して、サーチャーが1,000ETHを1,045ETHに変換して収益を得た裁定取引の例です。

### 流動化 {#mev-examples-liquidations}

貸出プロトコルによる精算も、よく知られたMEV獲得機会のひとつです。

MakerやAaveのような貸出プロトコルでは、ユーザーは何らかの担保（例: ETH）を預入しなければなりません。 預け入れられた担保は、他のユーザーに貸し出されます。

ユーザーは、この仕組みを活用して、預け入れた担保に対する一定の上限額まで、自らの必要に応じて他のユーザーから資産やトークンを借り入れることができます（例: MakerDAOのガバナンス提案に投票するために、MKRを借り入れる場合）。 例えば、借入可能額が預け入れた担保の30%までの場合、プロトコルに100 DAIを預け入れれば、30 DAIの価値を持つ他の資産を借り入れることができます。 貸出プロトコルは、具体的な借入能力（担保に対する割合）を決定します。

借り手の担保価値が変動すれば、借入能力も上下します。 具体的には、市場の変動により借入資産の価値が担保価値の（例えば）30%を越えた場合（上述したように、実際の借入可能な割合は貸出プロトコルが決定します）、貸出プロトコルは通常、あらゆるユーザーに対してこの担保を精算し、瞬時に貸し手に払い戻すことを許可します（これは、伝統的な金融市場における[マージンコール](https://www.investopedia.com/terms/m/margincall.asp)の仕組みと同様です）。 一般に、担保が精算される場合に借り手は高額の精算手数料を支払う必要があり、この手数料の一部は清算人に支払われるため、MEVを獲得する機会が発生するのです。

サーチャーは、可能なかぎり高速にブロックチェーンのデータを分析することで、精算の対象となりうる借り手を特定し、精算トランザクションを他に先がけて送信し、精算手数料を回収するための競争を行っています。

### サンドイッチ取引 {#mev-examples-sandwich-trading}

サンドイッチ取引もまた、一般的なMEV抽出の方法です。

サーチャーは、サンドイッチ取引を念頭に置いてDEXにおける大型取引のメモリプールを監視します。 例えば、あるユーザーがUniswapで、DAIを使って10000 UNIを購入したい場合を考えてみましょう。 これほど大きな取引は、UNI/DAIペアの為替レートに有意な影響を与えるものであり、対DAIのUNI価格を大きく上昇させる可能性があります。

サーチャーは、この大口取引によるUNI/DAIペアに対する大まかな価格効果を計算し、大口取引の_直前に_最適な買い注文を実行してUNIを安価で購入した上で、大口取引の_直後に_ 売り注文を実行し、大口注文により値上がりした価格で売却することができます。

しかし、サンドイッチ取引は1回で完結できない取引であるため（この点が、上述のDEXにおける裁定取引とは異なります）よりリスクが大きく、[サルモネラ攻撃](https://github.com/Defi-Cartel/salmonella)の対象となりやすい欠点を持ちます。

### NFT MEV {#mev-examples-nfts}

NFTを用いたMEVの抽出は、最近発生しつつある現象であり、必ずしも利益を伴う取引ではありません。

しかし、NFTのトランザクションは、その他すべてのイーサリアムのトランザクションと同一のブロックチェーン上で実行されるため、サーチャーは上記で紹介した従来のMEV獲得のためのテクニックをNFT市場でも応用することができます。

例えば、人気が高いNFTドロップが実行され、サーチャーが特定のNFTあるいは複数のNFTを持つセットを獲得したい場合、このNFTの購入リストの先頭になるようにトランザクションをプログラミングしたり、1回のトランザクションで当該NFTのセット全体を購入したりすることができます。 また、NFTが [ミスにより低価格で出品](https://www.theblockcrypto.com/post/113546/mistake-sees-69000-cryptopunk-sold-for-less-than-a-cent)されている場合、サーチャーは他の購入者を先回りして取引を実行し、安価で手に入れることができます。

NFTを使ったMEV抽出の顕著な事例としては、あるサーチャーが700万ドルを投じて、Cryptopunkを底値ですべて[購入した](https://etherscan.io/address/0x650dCdEB6ecF05aE3CAF30A70966E2F395d5E9E5)ケースがあります。 この事例については、買い手がMEVプロバイダーと連携してどのようにこの購入を秘匿したかについて、あるブロックチェーン研究者が[ツイッター](https://twitter.com/IvanBogatyy/status/1422232184493121538)上で説明しています。

### ロングテール {#mev-examples-long-tail}

DEXにおける裁定取引、精算、およびサンドイッチ取引はいずれも、よく知られたMEVの獲得機会であり、新規参入のサーチャーが利益を得られる可能性は低いです。 しかし、数は少ないものの、まだ一般的でないMEVの獲得機会も存在します（NFTによるMEV獲得も、そのひとつだと言えるかもしれません）。

サーチャーの分野に新たに参入したい方は、このロングテールの部分に焦点を当てることで、MEVを獲得できる可能性が高まるかもしれません。 フラッシュボットの[MEV求人板](https://github.com/flashbots/mev-job-board)には、最近発生しつつある新たなMEV獲得機会が掲載されています。

## MEVの影響 {#effects-of-mev}

MEVは必ずしも絶対悪ではなく、イーサリアムにとってはよい影響と悪い影響の両方を持ちます。

### 善者 {#effects-of-mev-the-good}

多くの分散型金融（DeFi）プロジェクトは、プロトコルの有用性と安定性を確保するために、経済的合理性に基づいて行動するユーザーに依存しています。 DEXにおける裁定取引は、ユーザーが各自のトークンに対して最善かつ最も適切な価格を得られることを保証するメカニズムであり、貸出プロトコルは、借入可能な比率を超過した借り手にただちに精算を迫ることで、貸し手に対する返済を保証しています。

経済合理性に基づき行動するサーチャーが、経済的な非効率性を発見し、それを修正することで、当該プロトコルにおける経済的なインセンティブを獲得することができなければ、DeFiのプロトコルやDappは現在実現されている強じん性を維持できないでしょう。

### 悪者 {#effects-of-mev-the-bad}

アプリケーションレイヤーにおいては、サンドイッチ取引をはじめとする一部のMEV獲得方法はユーザーの利用体験を明らかに低下させます。 サンドイッチ取引に巻き込まれたユーザーは、スリッページの被害を受け、取引条件が悪化するでしょう。

ネットワークレイヤーにおいては、汎用フロントランナーやガス価格のオークションが頻繁に用いられるため（複数のフロントランナーが、次のブロックに追加されるトランザクションのガス代を吊り上げる競争を行う場合）、ネットワークの混雑が悪化するだけでなく、通常のトランザクションを実行したい他のすべてのユーザーにとってもガス代が上昇してしまいます。

MEVは、ブロック _内部__における影響に加えて、複数のブロック_間_においても悪影響をもたらす場合があります。 特定のブロック内において獲得できるMEVが標準的なブロック報酬を大きく上回る場合、バリデータにとっては、ブロックを再編成し、バリデータ自身がMEVを獲得しようというインセンティブが発生しうるため、ブロックチェーンの再編成を促し、コンセンサスの安定性が損なわれる可能性があります。

このブロックチェーンが再編成される可能性は、 [すでにビットコインのブロックチェーンにおいて発生しています](https://dl.acm.org/doi/10.1145/2976749.2978408)。 ビットコインのブロック報酬が半減し、ブロック報酬においてトランザクション手数料が占める割合がますます大きくなると、マイナーにとっては、次のブロックで得られる報酬よりも、より高額な手数料が期待できる過去のブロックを再採掘する方が経済的に合理的である状況が発生します。 MEVの抽出が一般化した場合、イーサリアムにおいても類似の状況が発生し、イーサリアム・ブロックチェーンの健全性が損なわれる可能性があります。

## MEVの現況 {#state-of-mev}

MEVの抽出は、2021年初頭から爆発的に増加し、同年1月から数ヶ月にわたりガス価格が大きく高騰しました。 しかし、フラッシュボットのMEVリレーが登場した結果、汎用フロントランナーの効果が薄れ、ガス代のオークションがオフチェーンで実行されるようになったため、通常のユーザーが支払うガス価格は低下しました。

現在も多くのサーチャーがMEVにより充分な利益を得ているものの、MEVの抽出機会に対する認知度が高まり、同じ機会により多くのサーチャーが参加するようになるにつれ、MEVの総収益のうちバリデータの利益が占める割合はますます大きくなるでしょう（と言うのも、上記で説明したガス代のオークションと同様の状況は、非公開の取引を通じてフラッシュボットでも発生するため、その結果発生するガス収益はバリデータが獲得するためです）。 また、MEVが発生するのはイーサリアムに限りません。イーサリアムにおけるMEVの獲得機会に対する競争が激化するにつれ、サーチャーは、同じような機会が存在し、より競争相手が少ないバイナンス・スマートチェーンなどの他のブロックチェーンに移行する傾向を強めています。

一方、プルーフ・オブ・ワークからプルーフ・オブ・ステークへの移行や、ロールアップを活用してイーサリアムのスケーリングを実現するための継続的な取り組みなどはいずれも、MEVを取り巻く環境を一変させるものですが、それらがどのような影響を及ぼすかは現時点では明確ではありません。 プルーフ・オブ・ワークの確率論的モデルとの比較において、保証済みのブロック提案者に対してやや事前に通知することがMEV抽出の力学をどのように変化させるのか、あるいは、[1名のシークレットリーダー選挙](https://ethresear.ch/t/secret-non-single-leader-election/11789)および[分散型バリデータ技術](/staking/dvt/)が実装された場合に、この環境がどのような影響を被るのかは、現在のところはっきりしていません。 同様に、大部分のユーザーアクティビティがイーサリアムの外部に移行し、L2におけるロールアップやシャードで実行される場合、どのようなMEVの抽出機会が残存するのかも不明です。

## イーサリアムのプルーフ・オブ・ステーク（PoS）におけるMEV {#mev-in-ethereum-proof-of-stake}

上述したように、MEVは全般的なユーザー体験やコンセンサスレイヤーのセキュリティに悪影響を及ぼします。 しかし、イーサリアムにおけるプルーフ・オブ・ステークへの移行（「マージ」と呼ぶ）に伴い、MEVに関連して以下のような新たなリスクが発生する可能性があります：

### バリデータの集中 {#validator-centralization}

マージ後のイーサリアムにおいては、バリデータ（32 ETHのセキュリティ・デポジットを行ったユーザー）は、ビーコンチェーンに追加されたブロックの検証に対するコンセンサスに参加します。 多くのユーザーにとっては、32 ETHの負担は大きすぎるため、[ステーキングプールへの参加](/staking/pools/)がより現実的なオプションとなるかもしれません。 しかし、特定のユーザーのみにバリデータが集中することを回避し、イーサリアムのセキュリティを向上させるには、[ソロステーカー](/staking/solo/)の割合を健全に保つことが望ましいと言えます。

しかし、MEVの抽出機会により、特定のユーザーのみがバリデータとなる傾向が強まると考えられています。 この原因のひとつとして、現在のところ、バリデータはマイナーと比較すると[ブロックの提案により得られる収入が少ない](/roadmap/merge/issuance/#how-the-merge-impacts-ETH-supply)ため、MEVの抽出がマージ後における[バリデータの収益](https://github.com/flashbots/eth2-research/blob/main/notebooks/mev-in-eth2/eth2-mev-calc.ipynb)に対して大きな影響を与えると予想されるためです。

ステーキングプールの規模が大きくなればなるほど、MEVの抽出機会を活用するために必要な最適化に対してより多くのリソースを投じる傾向が高まるでしょう。 これらのプールから抽出できるMEVが大きくなればなるほど、バリデータはMEVの抽出能力を向上させるためにより多くのリソースを投じ、（さらに全体的な収益増を実現する）ことになるため、結果的に[規模の経済](https://www.investopedia.com/terms/e/economiesofscale.asp#)が実現されます。

ソロステーカーは、利用できるリソースが少ないために、MEVの機会から利益を得られなくなるかもしれません。 これにより、独立系のバリデータが収益を高めるために大規模なステーキングプールに参加する傾向が強まり、イーサリアムにおける分散化が損なわれる可能性があります。

### 許可済みのメモリプール {#permissioned-mempools}

サンドイッチ取引やフロントランナー攻撃に対処するため、取引を行うユーザーは、トランザクションのプライバシーを維持するためにバリデータを伴う取引をオフチェーンで実行するようになるかもしれません。 MEV抽出の可能性があるトランザクションを公開メモリプールに送信するのではなく、バリデータにトランザクションを直接送信する方法では、ブロックへの追加を担うバリデータと利益を共有することができます。

このような取り決めをより大規模にしたものが「ダークプール」であり、一定の手数料を支払う意思があるユーザーのみが参加する、許可済みでアクセスのみ可能なメモリプールとして機能します。 このようなメモリプールを活用する傾向が強まれば、イーサリアムにおけるパーミッションレス、トラストレスの特性が失われ、ブロックチェーンが最高額入札者にとって有利な「ペイ・トゥ・プレイ」のメカニズムに変質してしまう可能性があります。

さらに、許可済みのメモリプールは、上述したようにユーザーの集中化というリスクを強めます。 複数のバリデータが大規模なプールを実行する場合、取引者およびユーザーにとってはトランザクションのプライバシーを維持できるという利益が発生するため、MEVの収益が増加するでしょう。

マージ後のイーサリアムにおいては、これらのMEV関連の問題にどう対処するかがリサーチコミュニティにおける核心的な課題となっています。 現在のところ、マージ後のイーサリアムにおいてMEVが及ぼす分散化およびセキュリティへの悪影響を軽減するためのソリューションとしては、**提案者と作成者の分離（PBS）**および**ビルダーAPI**が提案されています。

### 提案者と作成者の分離（PBS） {#proposer-builder-separation}

プルーフ・オブ・ワークであれプルーフ・オブ・ステークであれ、ブロックを作成するノードは、作成したブロックをチェーンに追加することをコンセンサスに参加する他のノードに提案します。 新しいブロックは、他のマイナーがさらにその上にブロックを構築した場合（プルーフ・オブ・ワーク）、あるいは過半数のバリデータからアテステーションを受け取った場合（プルーフ・オブ・ステーク）に、正規のブロックチェーンに追加されます。

この記事で紹介したMEVに関する問題点の多くは、ブロックの作成者と提案者の役割が分離されていないことに由来しています。 例えば、コンセンサスに参加しているノードは、MEVによる収益を最大化するために、タイムバンディット攻撃を使ってチェーンの再編成をトリガーするインセンティブを持っています。

[提案者と作成者の分離（PBS）](https://ethresear.ch/t/proposer-block-builder-separation-friendly-fee-market-designs/9725) は、特にコンセンサスレイヤーにおけるMEVの影響を軽減するように設計されています。 PBSの主な機能は、ブロックの作成者と提案者に対するルールを分離することです。 バリデータがブロックの提案と投票に責任を負う点は変わりませんが、**ブロックビルダー**と呼ばれる専門の新たなエンティティ・クラスが導入され、トランザクションの順番付けと構築を担当することになります。

PBSでは、ブロックビルダーがトランザクションバンドルを作成し、ビーコンチェーン・ブロックへの追加に対して（「実行ペイロード」として）入札を行います。 次のブロック提案者として選択されたバリデータは、これを受けて、様々な入札をチェックし、最も高額な手数料のバンドルを選択します。 つまりPBSは、ブロックスペースの販売価格についてビルダーとバリデーターが交渉するオークション市場を構築するものです。

現在のPBSの設計では、ブロックビルダーは入札において、当該ブロックのコンテンツ（ブロックヘッダー）に対する暗号コミットメントのみを公開する[コミットメント＝公開スキーム](https://gitcoin.co/blog/commit-reveal-scheme-on-ethereum/)が採用されています。 提案者は、最高額の入札を受け入れた上で、ブロックヘッダーを含む署名済みのブロック提案を作成します。 ブロックビルダーは、署名済みのブロック提案を確認した後にブロック全体を公開すると想定されており、ブロック提案が最終決定される前に、バリデータから充分な数の[アテステーション](/glossary/#attestation)を得なければなりません。

#### 提案者と作成者の分離は、MEVの悪影響をどの程度緩和するか？ {#how-does-pbs-curb-mev-impact}

プロトコルにおいて提案者と作成者を分離することで、MEVの抽出がバリデータの作業範囲に含まれなくなるため、コンセンサスに対するMEVの影響を軽減することができます。 これにより、今後は専用のハードウェアを稼働させているブロックビルダーがMEVの抽出機会を得ることになります。

ただし、ブロックビルダーはバリデータによる承認を得るために入札額を高くしなければならないため、バリデータにとってもMEVに伴う収益がまったくゼロになるわけではありません。 しかし、バリデータはMEVによる収益増を直接的な目標として行動しなくなるため、タイムバンディット攻撃の脅威を低下させることができます。

PBSはさらに、MEVによる集中化のリスクを引き下げます。 例えば、コミット＝公開スキームにより、ブロックビルダーは、バリデータがMEVの抽出機会を奪い取ったり、他の構築者に公開することを行わないユーザーである信頼する必要がなくなります。 これにより、ソロステーカーがMEV抽出による利益を得る上での障壁が低くなります。そうでなければ、ブロックビルダーはオフチェーンでの評判が高い大規模なプールを選好し、オフチェーンでの取引が増加するトレンドが発生するでしょう。

同じように、バリデータの側も、支払いが必須であるため、ブロックビルダーがブロックボディを秘匿したり、無効なブロックを公開しないユーザーである信頼する必要がなくなります。 つまり、提案されたブロックが参照できない場合や、他のバリデータにより無効と宣言された場合でも、当初のバリデータの手数料は処理されるのです。 他のバリデータが無効と宣言した場合、当該ブロックは単に破棄され、ブロックビルダーはすべての取引手数料およびMEV収益を失うことになります。

### ビルダーAPI {#builder-api}

PBS（提案者と作成者の分離）は、MEVの抽出に伴う悪影響を減らす効果が期待される一方で、実装にはコンセンサス・プロトコルの変更が必要です。 具体的には、ビーコンチェーンにおける[分岐の選択](/developers/docs/consensus-mechanisms/pos/#fork-choice)ルールを変更する必要があります。 [ビルダーAPI](https://github.com/ethereum/builder-specs)は、信頼性の前提がより高くなるものの、提案者と作成者を実務的に分離するための一時的なソリューションです。

ビルダーAPIは、コンセンサスレイヤーのクライアントが実行レイヤーのクライアントに対して実行ペイロードを請求する際に用いられる[エンジンAPI](https://github.com/ethereum/execution-apis/blob/main/src/engine/common.md)の修正バージョンです。 [正直なバリデータの仕様](https://github.com/ethereum/consensus-specs/blob/dev/specs/bellatrix/validator.md)に概要が記述されている通り、ブロック提案に関する職責のために選択されたバリデータは、接続された実行クライアントにトランザクションバンドルを要求し、そのバンドルを提案されたビーコンチェーンのブロックに追加します。

ビルダーAPIはさらに、バリデータと実行レイヤーのクライアントとの間のミドルウェアとして機能します。ただし、ビーコンチェーン上のバリデータが外部エンティティからブロックを調達できる（実行クライアントを用いてローカルでブロックを構築するのではない）点が異なります。

以下に、ビルダーAPIの仕組みについて簡単に説明します：

1. ビルダーAPIにより、バリデータは、実行レイヤーのクライアントを実行しているブロックビルダーのネットワークに接続されます。 PBSの場合と同様に、ビルダーは、リソース集約型のブロック構築への投資に特化し、MEVや優先度チップを通じた収益最大化のために様々な戦略を用いるユーザーで構成されます。

2. （コンセンサスレイヤーのクライアントを実行中である）バリデータは、ビルダーのネットワークからの入札と同時に、実行ペイロードを要求します。 ビルダーからの入札には、実行ペイロードヘッダー（ペイロードのコンテンツに対する暗号コミットメント）と、バリデータに支払う手数料が含まれています。

3. バリデータは、送信された入札をレビューした上で、最も手数料が高い実行ペイロードを選択します。 バリデータは、ビルダーAPIを用いて、各自の署名と実行ペイロードヘッダーのみが含まれている「ブラインド」のビーコン用ブロック提案を作成し、ビルダーに送り返します。

4. ビルダーAPIを実行しているビルダーは、ブラインドのブロック提案を確認した上で、完全な実行ペイロードで対応すると想定されています。 これにより、バリデータは「署名済み」のビーコンブロックを作成し、ネットワークに拡散することができます。

5. ビルダーAPIを使用するバリデータの場合でも、ブロックビルダーが迅速に対応しない場合にブロック提案に伴う報酬が受け取れない場合を避けるために、ローカルでブロックを構築する必要があります。 しかしバリデータは、この時点で公開されたトランザクションあるいは他のセットを用いて別のブロックを作成することはできません。これは_曖昧化_（同じスロット内の2つのブロックに署名すること）を発生させるため、スラッシングの対象である違反行為です。

ビルダーAPIの実装例としては、イーサリアムに対するMEVの悪影響を軽減するように[フラッシュボットのオークション機能](https://docs.flashbots.net/Flashbots-auction/overview/)を改善した[MEV Boost](https://github.com/flashbots/mev-boost)があります。 フラッシュボットのオークションでは、プルーフ・オブ・ワークを行うマイナーに対し、利益を伴うブロックを作成する作業を**サーチャー**と呼ばれる専門のユーザーに外注することが認められています。

サーチャーは、利益性が高いMEVの機会を発見するために、マイナーに対して[非公開の入札価格](https://en.wikipedia.org/wiki/First-price_sealed-bid_auction)と共にトランザクションバンドルを送信してブロックへの追加を求めます。 Go-Ethereum（Geth）クライアントの分岐後のバージョンであるmev-gethを実行しているマイナーは、最も収益が大きいバンドルを選択し、それを新たなブロックの一部としてマイニングすればよいのです。 マイナーからスパムや無効のトランザクションから保護するため、トランザクションバンドルは、マイナーが受信する前に、**リレイヤー**による検証が行われます。

MEV Boostは、フラッシュボットにおける従来のオークションと同一の仕組みを採用していますが、イーサリアムにおけるプルーフ・オブ・ステークへの移行に対応した新機能が追加されています。 サーチャーが利益を伴うMEVトランザクションをブロックに追加しようとする点は同じですが、**ビルダー**と呼ばれる新たな専門ユーザーがトランザクションおよびバンドルをブロックにまとめる役割を担います。 ビルダーは、サーチャーから送信された非公開の入札価格を受け入れ、最適化を実行することで最も利益性が高い注文を決定します。

ここでもリレイヤーは、提案者に送信する事前にトランザクションを検証する責任を負う点は変わりません。 しかしMEV Boostでは、ビルダーから送信されたブロックボディおよびバリデータから送信されたブロックヘッダーを保存することで、[データの可用性](/developers/docs/data-availability/)を提供する仕組みである**エスクロー**が導入されています。 エスクローでは、リレーに接続されたバリデータが利用可能な実行ペイロードを要求し、MEV Boostの注文アルゴリズムを用いて、入札価格およびMEVチップが最も高いペイロードヘッダーを選択します。

#### ビルダーAPIは、どのようにMEVの悪影響を軽減するのか？ {#how-does-builder-api-curb-mev-impact}

ビルダーAPIがもたらす利点の核心は、MEVの抽出機会を利用できるユーザー層を民主化しうるという点にあります。 コミット＝公開スキームを採用することで、信頼性の前提が必要なくなり、MEVから利益を得たいと考えるバリデータにおける参入障壁が低くなります。 このため、ソロステーカーがMEVによる収益増を目指す場合に、大規模なステーキングプールに参入しなければならないという圧力が少なくなるでしょう。

ビルダーAPIの実装が一般化すれば、ブロックビルダー間の競争が促進され、検閲耐性が高まるでしょう。 バリデータは、複数のビルダーからの入札をレビューするようになるため、1件または複数のユーザートランザクションを検閲したいと考えるビルダーは、検閲の意図を持たないその他すべてのビルダーよりも高値を提示しなければならなくなります。 これにより、ユーザーを検閲するコストが劇的に上昇するため、検閲できるユーザーが減ると予想されます。

MEV Boostをはじめとするいくつかのプロジェクトでは、フロントランニングやサンドイッチ取引などを開始したいトレーダーなど、特定のユーザーにトランザクションのプライバシーを提供するという全般的な構造設計の一環としてビルダーAPIを採用しています。 これは、ユーザーとブロックビルダー間に非公開のコミュニケーションチャネルを提供することで達成されます。 上述の許可済みメモリプールの場合とは異なり、このアプローチは以下の点で有益だと言えます：

1. 複数のビルダーが市場で共存することで、実務上検閲の意味がなくなるため、ユーザーにとって有益です。 反対に、集中型で信頼ベースのダークプールが存在する場合、数名のブロックビルダーに権力が集中し、検閲が発生する可能性が高まります。

2. ビルダーAPIのソフトウェアはオープンソースであるため、どのユーザーでもブロックビルダー関連のサービスを提供できます。 これにより、ユーザーは特定のブロックビルダーの使用を強制されることがないため、イーサリアムの中立性やパーミッションレス性が向上します。 さらに、MEVの獲得を目指すトレーダーが、非公開のトランザクションチャネルを利用することで意図せずに集中化を促進してしまうことがなくなります。

## 関連リソース {#related-resources}

- [フラッシュボット関連文書](https://docs.flashbots.net/)
- [フラッシュボットのGitHub](https://github.com/flashbots/pm)
- [MEV-Explore](https://explore.flashbots.net/) - _MEVのトランザクションを対象とするダッシュボードおよび同時検索プログラム_
- [mevboost.org](https://www.mevboost.org/) - _MEV-Boostリレーとブロックビルダーに関するリアルタイムの統計を提供するトラッカー_

## 参考文献 {#further-reading}

- [採掘可能価値（MEV）とは何か？](https://blog.chain.link/what-is-miner-extractable-value-mev/)
- [MEVと私](https://www.paradigm.xyz/2021/02/mev-and-me)
- [イーサリアムはダークな森である](https://www.paradigm.xyz/2020/08/ethereum-is-a-dark-forest/)
- [ダークな森から抜け出すには](https://samczsun.com/escaping-the-dark-forest/)
- [フラッシュボッツ: MEV危機をフロントランニングするには](https://medium.com/flashbots/frontrunning-the-mev-crisis-40629a613752)
- [@bertcmillerのMEV関連スレッド](https://twitter.com/bertcmiller/status/1402665992422047747)
- [MEV-Boost: マージに対応したFlashbotsのアーキテクチャ](https://ethresear.ch/t/mev-boost-merge-ready-flashbots-architecture/11177)
- [MEV Boostとは](https://www.alchemy.com/overviews/mev-boost)
- [mev-boostを実行する理由](https://writings.flashbots.net/writings/why-run-mevboost/)
- [イーサリアムへのヒッチハイク・ガイド](https://members.delphidigital.io/reports/the-hitchhikers-guide-to-ethereum)
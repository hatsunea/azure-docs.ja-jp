---
title: Team Data Science Process ライフサイクルのビジネスの把握ステージ - Azure | Microsoft Docs
description: データ サイエンス プロジェクトのビジネスの把握ステージの目標、タスク、成果物
services: machine-learning
documentationcenter: ''
author: deguhath
manager: cgronlun
editor: cgronlun
ms.assetid: ''
ms.service: machine-learning
ms.component: team-data-science-process
ms.workload: data-services
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/04/2017
ms.author: deguhath
ms.openlocfilehash: 6d8eedbbf4a682443e73ecb9cf9496f3cdd1cd9d
ms.sourcegitcommit: 944d16bc74de29fb2643b0576a20cbd7e437cef2
ms.translationtype: HT
ms.contentlocale: ja-JP
ms.lasthandoff: 06/07/2018
ms.locfileid: "34837203"
---
# <a name="business-understanding"></a>ビジネスの把握

この記事では、Team Data Science Process (TDSP) のビジネスの把握ステージに関連付けられている目標、タスク、成果物のアウトラインを示します。 このプロセスには、データ サイエンス プロジェクトを体系化するために使用できる推奨ライフサイクルが用意されています。 ライフサイクルは、プロジェクトで通常 (多くの場合に繰り返し) 実行される主要なステージのアウトラインを示します。

   1. **ビジネスの把握**
   2. **データの取得と理解**
   3. **モデリング**
   4. **デプロイ**
   5. **顧客による受け入れ**

TDSP ライフサイクルを視覚化すると、次のようになります。 

![TDSP ライフサイクル](./media/lifecycle/tdsp-lifecycle2.png) 


## <a name="goals"></a>目標
* モデル ターゲットとしての機能を果たし、その関連メトリックがプロジェクトの成功を評価するために使用される主要な変数を指定する。
* ビジネスがアクセスできるまたは取得する必要がある、関連性のあるデータ ソースを特定する。

## <a name="how-to-do-it"></a>方法
このステージでは、以下に示す 2 つの主な課題に取り組みます。 

   * **目標を定義する**: 顧客およびその他の利害関係者と連携し、ビジネス上の問題を把握し、特定します。 データ サイエンス手法のターゲットにできるような、ビジネスの目標を定義付ける質問を考案します。
   * **データ ソースを特定する**: プロジェクトの目標を定義付ける質問に対して答えを出すために利用できる関連データを見つけます。

### <a name="define-objectives"></a>目標を定義する
1. このステップの中心となる目標は、分析によって予測する必要がある、主要なビジネス変数を特定することです。 これらの変数は*モデル ターゲット*と呼ばれ、これらの変数に関連付けられているメトリックがプロジェクトの成功を評価するために使用されます。 このようなターゲットの 2 つの例は、売上予測とある注文が不正な注文である確率です。

2. 関連性があり、具体的かつ明確な "鋭い" 質問の問いかけと改善を行うことで、プロジェクトの目標を定義します。 データ サイエンスは、名前と数字を使用してこのような質問に回答するプロセスです。 鋭い質問の問いかけについて詳しくは、ブログ「[How to do Data Science](https://blogs.technet.microsoft.com/machinelearning/2016/03/28/how-to-do-data-science/)」(データ サイエンスの実施方法) をご覧ください。 データ サイエンスや機械学習は、一般的には以下の 5 種類の質問に回答するために使用されます。
 
   * どのくらいの量または数か  (回帰)
   * どのカテゴリか  (分類)
   * どのグループか  (クラスタリング)
   * 異常か (異常の検出)
   * どの選択肢を選ぶべきか  (推奨)

   これらの質問のうちのどれについて質問し、どのような回答がビジネス上の目標の達成に役立つかを判断します。

3. メンバーの役割と責任を定めることにより、プロジェクト チームを定義します。 見つかる情報の増加に伴い繰り返し実行する、大まかなマイルストーン プランを作成します。 

4. 成功のメトリックを定義します。 たとえば、顧客離れ予測を実行するとします。 この 3 か月間のプロジェクトの終わるまでに、"x" % の精度が必要です。 このデータを使用して、顧客離れを減らすためのプロモーションを実施できます。 **SMART (スマート)** なメトリックを定義する必要があります。 

   * **S**pecific (具体的) 
   * **M**easurable (測定可能)
   * **A**chievable (達成可能) 
   * **R**elevant (関連性がある) 
   * **T**ime-bound (期限付き) 

### <a name="identify-data-sources"></a>データ ソースを特定する
鋭い質問への既知の回答例を含むデータ ソースを特定します。 以下のようなデータを探します。

* 質問に対して関連性のあるデータ。 ターゲットの測定値と、ターゲットに関連する特徴があるか。
* モデル ターゲットの正確な測定値であるデータと、対象とする特徴。

たとえば、既存のシステムで問題を解決し、プロジェクトの目標を達成するには、従来とは異なる種類のデータを収集して記録する必要があることを発見する場合があります。 このような場合は、外部データ ソースを探すか、新しいデータを収集するようにシステムを更新します。

## <a name="artifacts"></a>アーティファクト
このステージでの成果物を以下に示します。

   * [チャーター ドキュメント](https://github.com/Azure/Azure-TDSP-ProjectTemplate/blob/master/Docs/Project/Charter.md): TDSP プロジェクト構造定義に標準テンプレートが用意されています。 チャーター ドキュメントは、常に変化するドキュメントです。 プロジェクトの進行中に新しい発見やビジネス要件の変化があったときに、このテンプレートを更新します。 検出プロセスの進行に合わせてこのドキュメントに詳細を追加し、更新を繰り返すことが重要です。 変更を加える際は、顧客およびその他の利害関係者が常に関与するようにし、変更の理由を明確に伝えます。  
   * [データ ソース](https://github.com/Azure/Azure-TDSP-ProjectTemplate/blob/master/Docs/Data_Report/Data%20Defintion.md#raw-data-sources): これは、TDSP プロジェクトのデータ ソースが含まれている **[データ レポート]** フォルダーにある**データ定義**レポートの **[生データ ソース]** セクションです。 このセクションは、生データの元の場所と移動先の場所を指定します。 後のステージで、分析環境にデータを移動するスクリプトなどの詳細を追加します。  
   * [データ辞書](https://github.com/Azure/Azure-TDSP-ProjectTemplate/tree/master/Docs/Data_Dictionaries): このドキュメントはクライアントによって提供されているデータの説明を提供します。 これらの説明にはスキーマ (データ型、検証ルールに関する情報 (ある場合)) とエンティティ関連の図 (ある場合) に関する情報が含まれています。

## <a name="next-steps"></a>次の手順

TDSP のライフサイクルの各ステップへのリンクを次に示します。

   1. [ビジネスの把握](lifecycle-business-understanding.md)
   2. [データの取得と理解](lifecycle-data.md)
   3. [モデリング](lifecycle-modeling.md)
   4. [デプロイ](lifecycle-deployment.md)
   5. [顧客による受け入れ](lifecycle-acceptance.md)

特定のシナリオ のプロセスに伴うすべての手順を初めから終わりまで完全に説明するチュートリアルが用意されています。 [例を含むチュートリアル](walkthroughs.md)の記事に、各シナリオが、簡潔な説明とリンク付きで示されています。 これらのチュートリアルは、クラウドとオンプレミスのツールおよびサービスをワークフローまたはパイプラインに組み込んで、インテリジェントなアプリケーションを作成する方法について説明しています。 

Azure Machine Learning Studio を使用して TDSP の手順を実行する方法の例については、「[Azure Machine Learning での Team Data Science Process の使用](http://aka.ms/datascienceprocess)」を参照してください。

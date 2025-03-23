# AWSを活用したLLM学習技術・手法の最適実装ガイド

AWSは大規模言語モデル（LLM）の学習から推論までをサポートする豊富なサービス群を提供しています。本レポートでは、LLMの様々な学習技術や手法に適したAWSサービスの組み合わせと、実際のユースケースを詳細に解説します。

## AWSのLLM学習関連サービスの全体像

### コア計算サービス
- **Amazon SageMaker**: 機械学習モデルの構築、トレーニング、デプロイを一元管理するフルマネージドサービスです[1]。
- **Amazon EC2 + Deep Learning AMI**: 機械学習フレームワークやライブラリが事前構成された環境で、柔軟なインスタンスタイプ選択が可能です[6][12][15]。
- **Amazon EC2 UltraClusters**: 最大30,000台のTrainiumチップを搭載し、6エクサフロップスの計算能力を提供する、オンデマンドスーパーコンピューターです[13]。
- **AWS ParallelCluster**: HPC（高性能コンピューティング）クラスターを簡単に構築・管理できるサービスです[9]。

### 高度化・拡張サービス
- **SageMaker HyperPod**: 大規模モデルトレーニング向けの高度なインフラ管理サービスです[3]。
- **SageMaker HyperPod Recipes**: Llama 3.1などの人気モデルのトレーニングとファインチューニングを1行のコードで開始できる機能です[3]。
- **SageMaker JumpStart**: 事前学習済みモデルを簡単に微調整・デプロイできるサービスです[5]。
- **AWS Batch**: バッチコンピューティングジョブを効率的に実行するサービスです[7]。
- **ECS/EKS**: コンテナオーケストレーションサービスで、分散学習ワークロードの管理に適しています[7][4]。

### 専用ハードウェアとツール
- **AWS Trainium/Inferentia**: 機械学習専用のAWS設計チップで、Trainiumはトレーニング、Inferentiaは推論に最適化されています[8][14]。
- **Neuron SDK**: TrainiumとInferentiaを活用するためのSDKで、NeuronX Distributedなどの分散学習ライブラリを含みます[8]。
- **Amazon Bedrock**: 複数のAI企業の基盤モデルを単一のAPIで利用できるフルマネージドサービスです[2][4]。
- **FSx for Lustre**: 高性能なファイルシステムで、大規模データセットの処理に適しています[1][13]。

### 追加で重要なサービス
- **Amazon S3**: 大規模データセットとチェックポイントの保存に使用される基本的なストレージサービスです[1][9]。
- **Elastic Fabric Adapter (EFA)**: 高性能なネットワークインターフェースで、分散トレーニングの通信効率を高めます[13]。
- **Amazon Kendra**: RAG（検索拡張生成）アプローチで利用可能な知識ベースサービスです[11]。
- **Amazon CloudWatch**: トレーニングジョブのモニタリングとログ記録に役立ちます。

## モデル規模と学習手法に応じた最適なサービス組み合わせ

### 小規模LLM（数百万〜数億パラメータ）

#### 事前学習・Continued Pre-training
- **推奨構成**: EC2 + Deep Learning AMI (単一GPUインスタンス) または SageMaker Training
- **理由**: 比較的小さいモデルは単一GPUでも効率的に学習可能で、Deep Learning AMIは必要なライブラリがすべて揃っているため迅速に開始できます[6][12]。

#### Fine-tuning・SFT・Instruction Tuning
- **推奨構成**: SageMaker JumpStart + Ground Truth
- **理由**: 事前学習済みモデルを少ないデータで効率的に微調整でき、Ground Truthでデータの品質管理も可能です[5]。

#### 知識蒸留（Knowledge Distillation）
- **推奨構成**: SageMaker Training + EC2 (CPU/GPU)
- **理由**: 教師モデルと生徒モデルを効率的に管理し、様々な計算リソースを柔軟に割り当てられます。

### 中規模LLM（数億〜数十億パラメータ）

#### 事前学習・Continued Pre-training
- **推奨構成**: SageMaker Training + 分散トレーニング、または Trainium + Neuron SDK
- **理由**: モデル並列性と分散処理が必要になり、TrainiumはGPUに比べてコスト効率が良いです[8][14]。

#### Fine-tuning・SFT・Instruction Tuning
- **推奨構成**: SageMaker HyperPod または EC2 GPU クラスター + EFA
- **理由**: 効率的な分散処理と通信が必要で、HyperPodはクラスター管理を簡素化します[3]。

#### RLHF・DPO・PPO
- **推奨構成**: SageMaker + Ground Truth（フィードバックデータ収集）
- **理由**: フィードバックループとモデル評価プロセスを一元管理できます。

### 大規模LLM（数百億〜数兆パラメータ）

#### 事前学習・Continued Pre-training
- **推奨構成**: EC2 UltraClusters + Trn1インスタンス + FSx for Lustre + EFA
- **理由**: 大規模な分散計算能力、高速なストレージアクセス、ノードブロッキングネットワークが必要です[13]。

#### Fine-tuning・SFT
- **推奨構成**: SageMaker HyperPod Recipes または ParallelCluster + P5/Trn1インスタンス
- **理由**: HyperPod Recipesは人気モデル向けの最適化済みトレーニングを提供し、大規模クラスター管理を簡素化します[3][9]。

#### RLHF・DPO・PPO・GRPO
- **推奨構成**: SageMaker HyperPod + 複数のワークフローを管理するパイプライン
- **理由**: 複雑なトレーニングパイプラインと大規模計算リソースの管理が必要です。

## AWSサービスを活用したLLM学習事例

### 事例1: ユビタスによる日本語LLMの事前学習
- **モデル**: 繁体字・日本語の大規模言語モデル
- **学習手法**: 事前学習
- **AWSサービス組み合わせ**:
  - Amazon EC2 P5インスタンス（16台のNVIDIA H100 GPU搭載）
  - AWS ParallelCluster + Slurm（リソース管理）
  - Amazon S3、CloudFront、EKS、ELB（補助サービス）
- **成果**: 学習時間を従来比で90%短縮[9]

### 事例2: Tanuki-8Bの分散学習と推論
- **モデル**: Tanuki-8B（Llamaアーキテクチャ互換）
- **学習手法**: 分散学習、Fine-tuning
- **データセット**: MinnadeChatデータセットとichikara-instruction
- **AWSサービス組み合わせ**:
  - AWS Trainium/Inferentia
  - ParallelCluster
  - Neuron SDK、NeuronX Distributed
  - Text Generation Inference (TGI)
- **特徴**: 学習と推論の両方を最適化するためにTrainiumとInferentiaを役割分担[8]

### 事例3: IntuitのQuickBooks取引分類LLM
- **モデル**: Fine-tuningされたLLM
- **用途**: QuickBooksでの取引分類
- **学習手法**: Fine-tuning
- **AWSサービス組み合わせ**:
  - Amazon SageMaker
  - SageMaker JumpStart
- **成果**: 従来型MLベースの400万個のモデルをLLMに置き換え、5-10%の精度向上[5]

### 事例4: BERT-Largeモデルの事前学習
- **モデル**: Hugging Face BERT-Large
- **学習手法**: 事前学習
- **AWSサービス組み合わせ**:
  - Amazon EC2 Trn1 UltraClusters
  - AWS Trainium
  - AWS ParallelCluster
  - Elastic Fabric Adapter (EFA)
  - Amazon FSx for Lustre
- **特徴**: 大規模なパラメータ効率的な学習を実現するためのクラスター構成[13]

## 特殊な学習手法のためのサービス組み合わせ

### RAG（検索拡張生成）
- **推奨構成**: Amazon Bedrock + Amazon Kendra + S3
- **理由**: Kendraが知識ベースとして機能し、BedrockがLLMインターフェースを提供、S3がドキュメント保存に使用されます[11]。

### 合成データ生成
- **推奨構成**: SageMaker JumpStart + Bedrock + SageMaker Studio
- **理由**: 既存のLLMを使用して学習データを生成し、管理するためのエンドツーエンドソリューションを提供します。

### Prompt Tuning・P-Tuning・LoRA
- **推奨構成**: SageMaker Training + Trainium/Inferentia（リソース効率）
- **理由**: パラメータ効率的な手法は、特化型ハードウェアでさらに効率化できます。

## 結論

LLMの学習はモデルの規模や学習手法によって最適な計算リソースと環境が異なります。AWSは小規模から大規模まで様々なLLM学習ニーズに対応する幅広いサービス群を提供しています。特に注目すべき点は、SageMaker HyperPod Recipesのような簡素化ソリューションと、EC2 UltraClustersやTrainium/Inferentiaといった大規模高効率計算インフラの組み合わせが、LLM開発の時間とコストを大幅に削減できることです。

モデル開発の目的と規模に応じて適切なサービスを組み合わせることで、効率的かつ効果的なLLM開発が可能になります。AWS上でのLLM開発は急速に発展しており、今後も新しいサービスや最適化手法が登場することが期待されます。

## 参考文献

本レポートは、AWS公式ブログ、技術解説記事、ユースケース事例を参考にしています。

Citations:
[1] https://aws.amazon.com/jp/blogs/news/training-large-language-models-on-amazon-sagemaker-best-practices/
[2] https://bema.jp/articles/241218/
[3] https://zenn.dev/kiiwami/articles/1861a91d0531ca05
[4] https://aws.amazon.com/jp/blogs/news/eureka-bedrock-incident-report-automation/
[5] https://zenn.dev/kiiwami/articles/f3bbae0cc523d9a6
[6] https://news.mynavi.jp/techplus/article/aws_1-15/
[7] https://zenn.dev/honma12345/articles/eb0fbdee4487d9
[8] https://zenn.dev/karakuri_blog/articles/f8d97eee4ee282
[9] https://aws.amazon.com/jp/blogs/news/gen-ai-usecase-ubitus/
[10] https://zenn.dev/mkj/articles/ed069f8aedc9ab
[11] https://aws.amazon.com/jp/builders-flash/202309/kendra-llm-concierge/
[12] https://dev.classmethod.jp/articles/tensorflow-deep-mnist-for-experts-on-p2-instance/
[13] https://aws.amazon.com/jp/blogs/news/scaling-large-language-model-llm-training-with-amazon-ec2-trn1-ultraclusters/
[14] https://www.issoh.co.jp/tech/details/4647/
[15] https://note.com/altbridgetech/n/n44aa34f73540
[16] https://aws.amazon.com/jp/sagemaker-ai/hyperpod/customers/
[17] https://aws.amazon.com/jp/blogs/news/gen-ai-usecase-saison-technology/
[18] https://aws.amazon.com/jp/blogs/news/sagemaker-studio-lab-japanese-oss-llm-fine-tuning/
[19] https://aws.amazon.com/jp/blogs/news/genai-case-study-tmn-systems/
[20] https://aws.amazon.com/jp/ai/generative-ai/
[21] https://dev.classmethod.jp/articles/hands-on-pytorch-distributed-learning-using-sagemaker-hyperpod/
[22] https://www.fujitsu.com/jp/solutions/infrastructure/construction/multi-cloud/aws/event-column/amazon-bedrock.html
[23] https://jedworkshop.github.io/JLR2024/materials/b-1.pdf
[24] https://aws.amazon.com/jp/blogs/news/llm-development-support-program-generative-ai-program-results/
[25] https://ncdc.co.jp/columns/8742/
[26] https://zenn.dev/kiiwami/articles/b9bb9b68db0abf71
[27] https://aws.amazon.com/jp/bedrock/testimonials/
[28] https://pages.awscloud.com/rs/112-TZM-766/images/20230629_publishing_seminar.pdf
[29] https://aws.amazon.com/jp/blogs/news/category/artificial-intelligence/aws-neuron/
[30] https://pages.awscloud.com/rs/112-TZM-766/images/CUS-07_AWS-Summit-2024_Stockmark.pdf
[31] https://aws.amazon.com/jp/blogs/news/how-to-expansively-train-robot-learning-by-customers-on-aws-using-functions-generated-by-large-language-models/
[32] https://dev.classmethod.jp/articles/parallel-cluster-trainium-neuron-training/
[33] https://dev.classmethod.jp/articles/marukura-parallel-cluster-de-trainium-and-neuron/
[34] https://www.nii.ac.jp/event/upload/20240308-5_kubo.pdf
[35] https://pages.awscloud.com/rs/112-TZM-766/images/6-AWS_GenAI.pdf
[36] https://speakerdeck.com/icoxfog417/aws-ji-chu-toreningu-at-llm-mian-qiang-hui
[37] https://www.hyperbilling.jp/blog/aws-reinvent-2024-accelerate-llms/
[38] https://aws.amazon.com/jp/blogs/news/achieve-high-performance-with-lowest-cost-for-generative-ai-inference-using-aws-inferentia2-and-aws-trainium-on-amazon-sagemaker/

---
Perplexity の Eliot より: pplx.ai/share
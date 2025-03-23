直接回答
主なポイント:
AWSのサービスを組み合わせることで、LLM（小規模、中規模、大規模）の学習技術（事前学習、ファインチューニング、RLHFなど）を試すことができます。
研究は、SageMaker、EC2、Bedrockなどが主要なサービスであることを示唆しています。
特定の組み合わせはモデルサイズや学習条件によって異なり、例えば大規模モデルの事前学習にはEC2とTrainium、ファインチューニングにはSageMaker JumpStartが適しています。
RLHFにはSageMakerとGround Truth Plusが推奨されますが、詳細な実例は限られています。
AWSサービスの概要:
AWSは、LLMのトレーニングに多くのサービスを提供しています。以下は主なサービスの役割です：
SageMaker: トレーニング、ファインチューニング、RLHFに使用。分散トレーニングをサポート。
SageMaker JumpStart: 事前学習済みモデルのファインチューニングに便利。
EC2 with Trainium/Inferentia: 大規模モデルの高性能トレーニングに最適。
FSx for Lustre: 大量のデータに高速アクセスが必要な場合に使用。
Bedrock: 基礎モデルのファインチューニングやデプロイに使用。
SageMaker Ground Truth Plus: RLHFでの人間フィードバックの管理に役立ちます。
学習条件ごとの組み合わせ:
大規模モデルの事前学習: EC2 with Trainium、SageMaker（分散トレーニング）、FSx for Lustreを組み合わせ。
中規模モデルのファインチューニング: SageMaker JumpStart、SageMaker Trainingを使用。
RLHF: SageMakerでモデルをトレーニングし、SageMaker Ground Truth Plusでフィードバックを収集。
知識蒸留: SageMakerで教師モデルと生徒モデルをトレーニング。
実例:
GPTモデルのトレーニングでは、SageMaker、EC2 P4d、FSx for Lustreが使用され、24,000トークン/秒のスループットが達成されました (Accelerating GPT Training)。
自動車用語のSLMカスタマイズでは、SageMakerとBedrockが42分でファインチューニングに使用されました (Customizing SLMs for Automotive Terminology)。
RLHFでは、SageMakerとGround Truth Plusが人間フィードバックの管理に使用されました (RLHF on SageMaker)。
意外な詳細:
RLHFの具体的な実例は限られており、DPOやPPOなどの詳細な手法に関する情報は不足していますが、SageMaker Ground Truth Plusがフィードバック収集に役立つことが明らかになりました。
詳細な調査ノート
本調査ノートでは、ユーザーのクエリに基づき、AWSのサービスを組み合わせたLLM（小規模、中規模、大規模）の学習技術や手法（事前学習、継続事前学習、SFT、Instruction Tuning、合成データ生成、RLHF（DPO、PPO、GRPOなど）、知識蒸留、Prompt Tuningなど）を試す方法論について詳細に分析しました。また、AWSサービスの組み合わせによるLLM学習事例についても、モデル、学習方法、利用されたAWSサービスをまとめました。2025年3月22日現在の情報に基づき、公式AWSドキュメントやブログポストを参照しました。
AWSサービスの関連性と役割
まず、ユーザーが挙げたAWSサービス（SageMaker、SageMaker HyperPod、SageMaker HyperPod recipes、SageMaker JumpStart、SageMaker Ground Truth、ParallelCluster、Batch、EC2 + Deep Learning AMI、EC2 UltraClusters、ECS/EKS、Trainium/Inferentia、Neuron SDK、Bedrock、Bedrock Agent、FSx for Lustre）について、それぞれの役割を以下にまとめます。
SageMaker: トレーニング、ファインチューニング、RLHF、知識蒸留などに使用される主要なサービス。分散トレーニングやモデルパラレル化をサポートし、Amazon SageMaker Trainingを通じて管理されたバッチMLコンピュートを提供。
SageMaker JumpStart: 事前学習済みモデルのファインチューニングやデプロイに便利。例として、Dolly、Falcon、Llama2などのモデルがサポートされています (Supported LLMs for Fine-tuning)。
SageMaker Autopilot: 自動化されたファインチューニングをサポートし、Meta Llama2-7Bなどのモデルを質問回答タスクにファインチューニング可能 (Fine-tune with Autopilot)。
EC2 with Trainium/Inferentia: 大規模モデルの高性能トレーニングや推論に最適。TrainiumはEC2 Trn1 UltraClustersでスケーリング可能 (Scaling with Trn1 UltraClusters)。
FSx for Lustre: 大量のデータに高速アクセスが必要な場合に使用。例として、GPTモデルのトレーニングで1000 MB/sの読み書きスループットを提供 (Accelerating GPT Training)。
Bedrock: 基礎モデルのファインチューニングやデプロイに使用。Amazon Titan FMsや他社のモデルを提供 (Bedrock Overview)。
SageMaker Ground Truth Plus: RLHFでの人間フィードバックの管理やデータラベリングに役立つ (Ground Truth Plus)。
ParallelCluster: EC2インスタンスのクラスタ管理に使用され、分散トレーニングに適しています。
Batch: バッチジョブの実行に使用可能で、トレーニングパイプラインの一部として役立ちます。
ECS/EKS: コンテナオーケストレーションに使用され、複雑なトレーニングセットアップに適しています。
Neuron SDK: AWS Inferentiaチップでの最適化に使用。
SageMaker HyperPod、SageMaker HyperPod recipes: 特定の大規模トレーニングシナリオで使用可能ですが、詳細な事例は限定的。
EC2 UltraClusters: 大規模クラスタのトレーニングに適していますが、具体的な事例は少ない。
Bedrock Agent: 主にアプリケーション統合に使用され、トレーニングには間接的に関与。
学習条件ごとのサービス組み合わせ
学習条件やモデルサイズに応じたAWSサービスの組み合わせを以下にまとめます。表形式で整理します。
学習条件
推奨サービス組み合わせ
使用例
大規模モデルの事前学習
EC2 with Trainium/Inferentia, SageMaker, FSx for Lustre, ParallelCluster
GPTモデルのトレーニング（64 x P4d.24xlarge、24,000トークン/秒） (Accelerating GPT Training)
中規模モデルのファインチューニング
SageMaker JumpStart, SageMaker Training, Bedrock
Meta Llama2-7Bの質問回答タスクへのファインチューニング (Fine-tune with Autopilot)
小規模モデルのカスタマイズ
SageMaker, Bedrock, SageMaker JumpStart
自動車用語のSLMカスタマイズ（Meta Llama3.1 8B、42分でファインチューニング） (Customizing SLMs)
RLHF（人間フィードバックによる強化学習）
SageMaker, SageMaker Ground Truth Plus, SageMaker Studio Notebook
RLHF実験の実施とフィードバック収集 (RLHF on SageMaker)
知識蒸留
SageMaker（教師モデルと生徒モデルのトレーニング）
詳細な事例は限定的だが、SageMakerで両モデルのトレーニングが可能
Prompt Tuning
SageMaker, Bedrock（LLMホスティング）
プロンプトの最適化に使用、具体例は少ない
AWSサービスを組み合わせたLLM学習事例
以下に、Web検索や公式ブログから得たLLM学習事例をまとめます。モデル、学習方法、利用されたAWSサービスを記載します。
事例1: GPTモデルの事前学習加速
モデル: GPTモデル（大規模）
学習方法: 事前学習
AWSサービス: SageMaker, EC2 P4d instances (400 Gbps EFA), FSx for Lustre
詳細: グローバル金融データプロバイダーが64 x P4d.24xlargeインスタンスで5日間トレーニング、2048トークンのシーケンス長で24,000トークン/秒のピークスループット達成 (Accelerating GPT Training)。
事例2: 毒性スピーチ分類のためのファインチューニング
モデル: 大規模言語モデル（特定モデル不明）
学習方法: ファインチューニング（分類タスク）
AWSサービス: SageMaker
詳細: ゲーミング会社向けに毒性スピーチ分類用にファインチューニング、転移学習を使用 (Fine-tuning for Toxic Speech)。
事例3: 自動車用語のSLMカスタマイズ
モデル: Meta Llama3.1 8B Instruct（小規模）
学習方法: ドメイン適応ファインチューニング
AWSサービス: SageMaker, SageMaker JumpStart, Bedrock
詳細: ml.g5.12xlargeインスタンス（4 GPU）で42分、LoRA（r=8, lora_alpha=16, lora_dropout=0.1）を使用、Automotive_NERデータセットのサブセットでトレーニング (Customizing SLMs)。
事例4: RLHFによるLLM改善
モデル: 大規模言語モデル（特定モデル不明）
学習方法: RLHF（人間フィードバックによる強化学習）
AWSサービス: SageMaker, SageMaker Ground Truth Plus, SageMaker Studio Notebook
詳細: 報酬モデルのトレーニングとRLトレーニングを実施、人間フィードバックをGround Truth Plusで管理 (RLHF on SageMaker)。
補足と考察
ユーザーが挙げたサービスのうち、SageMaker Ground Truth PlusはRLHFで特に重要であり、追加のサービスとして推奨されます。また、DPOやPPOなどの具体的なRLHF手法に関する詳細な事例は限定的で、今後の研究やドキュメントの更新に期待されます。SageMaker HyperPodやEC2 UltraClustersは大規模トレーニングに適していますが、具体的な事例は少なく、今後の利用が期待されます。
本調査は、2025年3月22日現在の公式AWSリソースに基づき作成されており、信頼性が高いと判断されます。詳細なソースは以下の引用に記載します。
主要引用
Accelerating GPT large language model training with AWS services
Training large language models on Amazon SageMaker: Best practices
What is LLM? - Large Language Models Explained - AWS
Scaling Large Language Model (LLM) training with Amazon EC2 Trn1 UltraClusters
Using Large Language Models (LLMs) in AWS
Deploy large language models on AWS Inferentia2 using large model inference containers
Create, train, and deploy a billion-parameter language model on terabytes of data with TensorFlow and Amazon SageMaker
AI Tools and Services – Artificial Intelligence Products – AWS
Generative AI with Large Language Models — New Hands-on Course by DeepLearning.AI and AWS
Techniques and approaches for monitoring large language models on AWS
Supported large language models for fine-tuning - Amazon SageMaker AI
Fine-tune large language models with Amazon SageMaker Autopilot
Fine-tune and deploy language models with Amazon SageMaker Canvas and Amazon Bedrock
Foundation models and hyperparameters for fine-tuning - Amazon SageMaker AI
AWS performs fine-tuning on a Large Language Model (LLM) to classify toxic speech for a large gaming company
Fine-tune transformer language models for linguistic diversity with Hugging Face on Amazon SageMaker
Customize small language models on AWS with automotive terminology
Domain-adaptation Fine-tuning of Foundation Models in Amazon SageMaker JumpStart on Financial data
Improving your LLMs with RLHF on Amazon SageMaker
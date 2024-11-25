---
title: "RDSのストレージタイプとインスタンスタイプの違い"
emoji: "🐈"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["RDS", "AWS", "mysql", "postgress"]
published: true
---

# 背景

- terraform を作成する際に、storage_type(ストレージタイプ) と instance_class (インスタンスタイプ)の違いがわからなかったため。

# 内容

# 内容

| **項目**         | **Instance Class**                 | **Storage Type**               |
| ---------------- | ---------------------------------- | ------------------------------ |
| **対象リソース** | CPU、メモリ、ネットワーク          | データ保存領域（ディスク性能） |
| **役割**         | データベースエンジンの実行リソース | データの保存および I/O 性能    |
| **例**           | db.m5.large, db.t3.micro           | gp2, gp3, io1                  |
| **設定内容**     | 計算性能（vCPU、RAM など）         | ストレージ容量、IOPS、性能     |
| **スケーリング** | クラスを変更して性能アップ/ダウン  | ストレージ容量と性能の調整     |

### IOPS（Input/Output Operations Per Second）とは?

RDS の追加、更新、削除の頻度が多いほど、高いほどがいい(リアルタイムで監視する場合など)

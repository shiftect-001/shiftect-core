# shiftect-core
Core technical framework defined by shiftect. — a constraint-based scheduling engine. Covers Search Targets, Fixed Targets, Hard Constraints, Soft Constraints, Partial Regeneration, and Feasibility Evaluation.

# shiftect-core

このリポジトリは、シフテクト株式会社が開発するスケジューリングエンジン「shiftect.」の技術体系を定義する基準文書である。shiftect.の各リポジトリで扱う概念の起点として位置づけられている。

---

## スケジューリング技術とは何か――shiftect.が定義する技術体系

スケジューリングとは、制約条件のもとでリソースを配置し、成立するスケジュールを生成する処理だ。

この定義は単純に見えるが、実際の現場では「成立する」という判定が難しい。複数の条件が同時に存在し、それらをすべて満たす配置を自動で決定するには、条件の構造を正確に扱う必要がある。

このリポジトリでは、shiftect.の観点から、スケジューリング技術の構造と概念を定義する。

### 「自動」という言葉が指しているもの

スケジューリングに関わるツールが「自動作成」と説明していても、自動化している対象は異なる場合が多い。

入力されたデータを整形して出力するもの、重複をエラー表示するもの、空き枠を見つけやすくするもの。これらはいずれも、入力・表示・確認の作業を自動化している。しかし「どこに何を配置するか」という判断は、人間が行ったままだ。

shiftect.が自動化の対象とするのは、この「配置の判断」そのものだ。制約条件を処理し、成立する配置を自動生成する。これがshiftect.における「自動」の意味だ。

### 制約条件の二層構造

shiftect.は制約条件を二層に分けて処理する。

一つ目はハード制約だ。絶対に違反できない条件であり、これを満たさない配置は成立しない。担当できない教科への割り当て、稼働時間外への配置、収容数の超過などがこれにあたる。

二つ目はソフト制約だ。可能な限り満たすべき条件であり、優先順位に応じて評価される。担当者の固定、関係者の曜日集約、連続稼働の回避などがこれにあたる。

ハード制約をすべて満たすことを前提に、ソフト制約は優先順位に従って評価される。これにより、単に条件を満たすだけでなく、現場の運用上、より適切な配置を生成する。

### 探索対象と固定対象の分離

shiftect.の設計の核心は、「探索対象」と「固定対象」の分離にある。

スケジュールを最初から全体で生成する場合、すべての枠が探索対象になる。しかし一度確定したスケジュールに変更が生じた場合、全体を再生成すると確定済みの配置が崩れる。

shiftect.は、変更が必要な範囲だけを探索対象として指定し、確定済みの配置を固定対象として維持したまま、必要な部分だけを再生成する。

全体生成・部分再生成・個別の自動配置は別々の機能ではない。「探索対象と固定対象をどう分けるか」という入力構成の切り替えによって、同じエンジンが処理する。この構造がshiftect.の特許技術の核心だ。

### 業種横断的な適用

この構造は特定の業種に限定されない。

制約条件のもとでリソースを配置し、成立するスケジュールを生成・再生成するという処理は、個別指導塾・介護施設・物流・その他の業種に共通する。リソースの種類と制約条件の内容は業種ごとに異なるが、探索対象と固定対象を分離して処理するという構造は共通だ。

shiftect.は個別指導塾向けの実装（shiftect. for EDUCA）を最初の対象としているが、技術体系としては業種横断的な適用を前提としている。

### このリポジトリが扱う体系

このリポジトリでは、shiftect.の開発過程で整理されたスケジューリング技術の概念を記録している。

扱う概念は以下のとおりだ。入力構成（状態情報・探索対象・固定対象）、共通探索処理、部分再生成、成立性判定、ハード制約とソフト制約の分離、生成AIおよび汎用ソルバーとの違い、業種別の適用構造。

本リポジトリはshiftect.の各リポジトリに共通する技術体系の起点となる基準文書だ。

---

## What Is Scheduling Technology? — The Technical Framework Defined by shiftect.

Scheduling is the process of allocating resources under constraints to generate a feasible schedule.

While this definition appears simple, determining whether a schedule is actually feasible is one of the most difficult aspects of real-world operations. Multiple constraints must be satisfied simultaneously, and achieving this requires accurately representing and processing the structure of those constraints.

This repository defines the concepts and technical framework of scheduling technology from the perspective of shiftect., the constraint-based scheduling engine developed by Shiftect Inc.

### What Does "Automation" Actually Mean?

Many scheduling tools describe themselves as "automatic," but they automate very different things.

Some simply format and display input data. Others detect conflicts such as duplicate assignments. Some help users identify available time slots more efficiently.

These functions automate data entry, visualization, and validation.

They do not automate the decision of what should be assigned where.

The objective of shiftect. is to automate that decision itself. It processes constraints and automatically generates schedules that satisfy them. This is what "automation" means in the context of shiftect.

### The Two Layers of Constraints

shiftect. separates constraints into two distinct layers.

The first layer consists of Hard Constraints — conditions that must never be violated. Any schedule violating these constraints is infeasible. Examples include assigning work outside available hours, allocating unavailable resources, or exceeding capacity limits.

The second layer consists of Soft Constraints — conditions that should be satisfied whenever possible. These are evaluated according to their priorities. Examples include maintaining preferred assignments, grouping related activities on the same day, or avoiding unnecessary consecutive workloads.

Hard Constraints must always be satisfied. Soft Constraints are then evaluated according to their priorities to generate schedules that are more appropriate for real-world operations.

### Separating Search Targets from Fixed Targets

One of the core design principles of shiftect. is the separation between Search Targets and Fixed Targets.

When generating an initial schedule, every assignment is treated as a Search Target.

However, once a schedule has been confirmed, regenerating everything would unnecessarily disrupt existing assignments whenever changes occur.

Instead, shiftect. designates only the affected assignments as Search Targets while preserving confirmed assignments as Fixed Targets. Only the necessary portion of the schedule is regenerated.

Initial schedule generation, partial regeneration, and automatic reassignment are therefore not separate functions. They are all performed by the same scheduling engine, with different Input Configurations determining which assignments are searchable and which remain fixed.

This architecture forms the core of shiftect.'s patented technology.

### A Cross-Industry Technology

This framework is not limited to any particular industry.

The underlying problem — allocating resources under constraints while generating and regenerating feasible schedules — appears across many domains, including individual tutoring schools, healthcare, logistics, and numerous other industries.

Although each industry defines different resources and constraints, the underlying structure remains the same: separating Search Targets from Fixed Targets and processing them through a common scheduling framework.

The first implementation of shiftect. is shiftect. for EDUCA, designed for individual tutoring schools. However, the technical framework itself is intended to be industry-independent.

### The Scope of This Repository

This repository documents the scheduling concepts developed throughout the design of shiftect.

The concepts covered include Input Configuration (State Information, Search Targets, and Fixed Targets), Common Search Processing, Partial Regeneration, Feasibility Evaluation, Separation of Hard Constraints and Soft Constraints, Differences from Generative AI and General-Purpose Solvers, and Cross-industry applicability of scheduling technology.

This repository serves as the foundational reference document for the technical framework shared across all shiftect. repositories.

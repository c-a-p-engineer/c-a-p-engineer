<p align="center">
  <img src="./assets/profile-banner.svg" alt="c-a-p-engineer — engineer the workflow, ship the result" width="100%" />
</p>

<h1 align="center">c-a-p-engineer — AI-assisted Software Engineer</h1>

<p align="center">
  <strong>曖昧な要求を、仕様・実装・検証までつながる再現可能な開発システムへ変えるソフトウェアエンジニアです。</strong>
</p>

<p align="center">
  AI協働開発、開発自動化、Developer Tools、動画生成、ドキュメント生成、ブラウザアプリを中心に、
  Python / TypeScript / JavaScript / Docker / GitHub Actions / FFmpeg などを使って実装しています。
</p>

<p align="center">
  <sub>
    GitHub: <strong>c-a-p-engineer</strong> · Blog: <strong>こぴぺたんログ</strong> · X: <strong>@c_a_p_engineer</strong>
  </sub>
</p>

<p align="center">
  <a href="https://c-a-p-engineer.github.io/">
    <img src="https://img.shields.io/badge/BLOG-0D1117?style=for-the-badge&logo=githubpages&logoColor=58A6FF" alt="c-a-p-engineer development blog" />
  </a>
  <a href="https://techbookfest.org/organization/5zdy9h5eA5kDzByP9rserV">
    <img src="https://img.shields.io/badge/TECHBOOKFEST-0D1117?style=for-the-badge&logo=bookstack&logoColor=A371F7" alt="c-a-p-engineer technical books on TechBookFest" />
  </a>
  <a href="https://x.com/c_a_p_engineer">
    <img src="https://img.shields.io/badge/@c__a__p__engineer-0D1117?style=for-the-badge&logo=x&logoColor=F0F6FC" alt="c-a-p-engineer on X" />
  </a>
</p>

## 01 / What I do

### AI-assisted engineering
AIにコードを書かせるだけでなく、**要件 → 仕様 → 実装 → レビュー → 検証**までを一つの開発フローとして設計します。判断や受け入れ条件を明示し、AIが動きやすく、人間が検証しやすい形へ落とします。

### Automation
Docker、Dev Container、GitHub Actions、CLI、生成スクリプトを組み合わせ、**繰り返せる処理をパイプラインへ変換**します。手作業、環境差分、「自分のPCだけ動く」を減らします。

### Software that creates
動画、スライド、教材、ゲームなど、成果物そのものだけでなく、**成果物を再現可能に生成するソフトウェア**を作ります。

> **Core idea:** Build less by hand. Build more systems.

## 02 / Featured systems

現在の代表的な公開プロジェクトです。分野は異なりますが、いずれも「入力・仕様から成果物や体験を再現可能に生成する」という共通テーマで作っています。

### [Zundamotion](https://github.com/c-a-p-engineer/zundamotion) — video automation

YAML台本から、VOICEVOX / Chatterbox音声、字幕、画像、BGM、効果音を合成してMP4を生成する動画制作パイプラインです。

**Proof:** AI / CI向けの machine-readable `validate` / `compile` / `capabilities` CLI、入力状態を固定する Render Lock、23言語TTSを扱えるoptional runtimeを備えています。

`Python` · `FFmpeg` · `VOICEVOX` · `Docker`

[DEMO →](https://c-a-p-engineer.github.io/zundamotion/) · [CODE →](https://github.com/c-a-p-engineer/zundamotion)

### [SlideForge](https://github.com/c-a-p-engineer/SlideForge) — document engineering

HTMLスライドをPNG / PDF / PPTXへ変換する、AI編集を前提にしたスライド生成・レンダリング基盤です。

**Proof:** 4テーマで共通利用できる28種類のスライドroleを持ち、Playwrightによるレンダリング、WebページCapture、draw.io図版生成まで同じツールチェーンで扱います。

`Node.js` · `Playwright` · `HTML/CSS` · `PDF` · `PPTX`

[CODE →](https://github.com/c-a-p-engineer/SlideForge)

### [kids-learning](https://github.com/c-a-p-engineer/kids-learning) — browser learning tools

スマートフォン、タブレット、PCのブラウザで遊びながら学べる子ども向け学習コンテンツ集です。

**Proof:** 聞く・待つ・見る段階から書字までつなぐ10 STEPの学習導線、端末内履歴、スマートフォン基準のUI、キーボード・reduced motion等を考慮した設計を持っています。

`TypeScript` · `Mobile First` · `Accessibility` · `GitHub Pages`

[PLAY →](https://c-a-p-engineer.github.io/kids-learning/) · [CODE →](https://github.com/c-a-p-engineer/kids-learning)

### [24365 IT戦士 ― 目grep](https://github.com/c-a-p-engineer/24365-it-warrior) — browser game

流れ続けるログから検索条件に一致する行を見つける、スマートフォン向け高速探索ゲームです。

**Proof:** GitHub PagesでプレイできるMVPとして公開し、複合条件・類似文字まで段階的に難化、最大3.2倍までの速度上昇、得点・コンボ・正解率等の記録を実装しています。

`JavaScript` · `Game Design` · `Mobile UI` · `GitHub Pages`

[PLAY →](https://c-a-p-engineer.github.io/24365-it-warrior/) · [CODE →](https://github.com/c-a-p-engineer/24365-it-warrior)

## 03 / Proof of work

公開Repositoryで、設計思想だけでなく実際の成果物と検証経路を確認できます。

- **Reproducible video generation** — YAML入力から動画生成までを自動化し、入力provenanceと事前validationを分離
- **Document generation pipeline** — HTMLを単一SourceとしてPNG / PDF / PPTXへ展開
- **Browser-first learning tools** — インストール不要の学習コンテンツをGitHub Pagesで公開
- **Playable browser game** — 企画、UI、実装、公開までを一つのRepositoryで完結
- **Documentation as an artifact** — README、仕様、ガイド、検証手順をコードと同じく管理

<details>
  <summary><strong>More tools & templates</strong></summary>
  <br />
  <a href="https://github.com/c-a-p-engineer/ai-editor-playbook">ai-editor-playbook</a>
  · <a href="https://github.com/c-a-p-engineer/VivliostyleTemplate">VivliostyleTemplate</a>
  · <a href="https://github.com/c-a-p-engineer/codex-phaser-template">codex-phaser-template</a>
  · <a href="https://github.com/c-a-p-engineer/skill-sheet-maker">skill-sheet-maker</a>
  · <a href="https://github.com/c-a-p-engineer/ResponsiveCapture">ResponsiveCapture</a>
</details>

## 04 / How I ship

<p align="center">
  <img src="./assets/build-system.svg" alt="Engineering delivery pipeline: intent, specification, AI-assisted implementation, verification, and shipping" width="100%" />
</p>

1. **Problem before tool** — 流行やモデル名から始めず、目的・制約・完成条件を決める
2. **Specs are artifacts** — README・仕様・タスク・検証条件も成果物として管理する
3. **AI accelerates execution** — 実装速度はAIで上げるが、判断・責任・受け入れ判定は外部化する
4. **Evidence before done** — テスト、差分、生成物、CI、実環境のEvidenceを確認して出荷する

## 05 / Toolchain

<p>
  <img src="https://img.shields.io/badge/Python-0D1117?style=flat-square&logo=python&logoColor=58A6FF" alt="Python" />
  <img src="https://img.shields.io/badge/TypeScript-0D1117?style=flat-square&logo=typescript&logoColor=58A6FF" alt="TypeScript" />
  <img src="https://img.shields.io/badge/JavaScript-0D1117?style=flat-square&logo=javascript&logoColor=F7DF1E" alt="JavaScript" />
  <img src="https://img.shields.io/badge/PHP-0D1117?style=flat-square&logo=php&logoColor=A371F7" alt="PHP" />
  <img src="https://img.shields.io/badge/Docker-0D1117?style=flat-square&logo=docker&logoColor=58A6FF" alt="Docker" />
  <img src="https://img.shields.io/badge/Dev_Containers-0D1117?style=flat-square&logo=visualstudiocode&logoColor=58A6FF" alt="Dev Containers" />
  <img src="https://img.shields.io/badge/GitHub_Actions-0D1117?style=flat-square&logo=githubactions&logoColor=58A6FF" alt="GitHub Actions" />
  <img src="https://img.shields.io/badge/AWS-0D1117?style=flat-square&logo=amazonwebservices&logoColor=FF9900" alt="AWS" />
  <img src="https://img.shields.io/badge/FFmpeg-0D1117?style=flat-square&logo=ffmpeg&logoColor=3FB950" alt="FFmpeg" />
  <img src="https://img.shields.io/badge/Playwright-0D1117?style=flat-square&logo=playwright&logoColor=3FB950" alt="Playwright" />
  <img src="https://img.shields.io/badge/Phaser-0D1117?style=flat-square&logo=phaser&logoColor=FF4F9A" alt="Phaser" />
  <img src="https://img.shields.io/badge/OpenAI-0D1117?style=flat-square&logo=openai&logoColor=F0F6FC" alt="OpenAI" />
</p>

## 06 / Writing

### [こぴぺたんログ — 開発・生成AI・自動化ブログ](https://c-a-p-engineer.github.io/)

AI協働開発、生成AI、Docker、GitHub Actions、自動化、開発ツール、失敗と改善を、**再現・検証できる技術知識**として記録しています。実装だけでなく「なぜその設計にしたか」「何が失敗したか」まで残す場所です。

### [技術書典](https://techbookfest.org/organization/5zdy9h5eA5kDzByP9rserV)

AI協働開発や実務で得た知見を、試して再現できる形へ整理して技術書にまとめています。

---

<p align="center">
  <strong>c-a-p-engineer / こぴぺたん</strong><br />
  <sub>AI-assisted software development · automation · developer tools · creative tooling</sub>
</p>

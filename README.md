<p align="center">
  <img src="./assets/profile-banner.svg" alt="c-a-p-engineer — AI-driven software, automation, and creative tooling" width="100%" />
</p>

<p align="center">
  <a href="https://c-a-p-engineer.github.io/">
    <img src="https://img.shields.io/badge/BLOG-0D1117?style=for-the-badge&logo=githubpages&logoColor=58A6FF" alt="Blog" />
  </a>
  <a href="https://techbookfest.org/organization/5zdy9h5eA5kDzByP9rserV">
    <img src="https://img.shields.io/badge/TECHBOOKFEST-0D1117?style=for-the-badge&logo=bookstack&logoColor=FF4F9A" alt="TechBookFest" />
  </a>
  <a href="https://x.com/c_a_p_engineer">
    <img src="https://img.shields.io/badge/@c__a__p__engineer-0D1117?style=for-the-badge&logo=x&logoColor=F0F6FC" alt="X" />
  </a>
  <img src="https://komarev.com/ghpvc/?username=c-a-p-engineer&style=for-the-badge&color=ff4f9a&label=PROFILE+VIEWS" alt="Profile views" />
</p>

<p align="center">
  <strong>ソフトウェアエンジニア / AI駆動開発 / 自動化 / 技術コンテンツ制作</strong>
</p>

> **書くコードを減らし、仕組みを増やす。**  
> 問題を整理し、README・Spec・Taskへ落とし込み、AIとCIを使って再現可能な成果物として出荷します。

## Engineering focus

<table>
  <tr>
    <td width="33%" valign="top">
      <h3>AI-driven development</h3>
      <p>AIへ実装を委譲しつつ、要件・設計・判断・レビューの責任は人間が持つ開発フローを設計します。</p>
    </td>
    <td width="33%" valign="top">
      <h3>Automation & reproducibility</h3>
      <p>Docker、Dev Container、GitHub Actionsを軸に、環境差分と手作業を減らす仕組みを作ります。</p>
    </td>
    <td width="33%" valign="top">
      <h3>Creative engineering</h3>
      <p>動画、スライド、技術書、ゲーム、教材を、コードと生成AIを組み合わせて制作します。</p>
    </td>
  </tr>
</table>

## Featured builds

<table>
  <tr>
    <td width="50%" valign="top">
      <h3><a href="https://github.com/c-a-p-engineer/zundamotion">Zundamotion</a></h3>
      <p>YAML台本から、VOICEVOX音声・字幕・画像・BGM・効果音を合成してMP4を生成する動画制作パイプライン。</p>
      <p><code>Python</code> <code>FFmpeg</code> <code>VOICEVOX</code> <code>Docker</code></p>
      <a href="https://c-a-p-engineer.github.io/zundamotion/"><strong>Demo</strong></a>
      ·
      <a href="https://github.com/c-a-p-engineer/zundamotion"><strong>Repository</strong></a>
    </td>
    <td width="50%" valign="top">
      <h3><a href="https://github.com/c-a-p-engineer/SlideForge">SlideForge</a></h3>
      <p>AIが編集しやすいHTMLスライドを、PNG・PDF・PPTXへ変換するテンプレート兼レンダリング基盤。</p>
      <p><code>Node.js</code> <code>Playwright</code> <code>HTML/CSS</code> <code>PPTX</code></p>
      <a href="https://github.com/c-a-p-engineer/SlideForge"><strong>Repository</strong></a>
    </td>
  </tr>
  <tr>
    <td width="50%" valign="top">
      <h3><a href="https://github.com/c-a-p-engineer/kids-learning">kids-learning</a></h3>
      <p>スマートフォンやタブレットで、図形・記憶・数量・そろばん・時計・書字を段階的に学べるブラウザ教材。</p>
      <p><code>TypeScript</code> <code>Mobile First</code> <code>Accessibility</code> <code>GitHub Pages</code></p>
      <a href="https://c-a-p-engineer.github.io/kids-learning/"><strong>Play</strong></a>
      ·
      <a href="https://github.com/c-a-p-engineer/kids-learning"><strong>Repository</strong></a>
    </td>
    <td width="50%" valign="top">
      <h3><a href="https://github.com/c-a-p-engineer/24365-it-warrior">24365 IT戦士 ― 目grep</a></h3>
      <p>流れ続けるログから検索条件に一致する行を見つける、スマートフォン向け高速探索ゲーム。</p>
      <p><code>JavaScript</code> <code>Game Design</code> <code>Mobile UI</code> <code>GitHub Pages</code></p>
      <a href="https://c-a-p-engineer.github.io/24365-it-warrior/"><strong>Play</strong></a>
      ·
      <a href="https://github.com/c-a-p-engineer/24365-it-warrior"><strong>Repository</strong></a>
    </td>
  </tr>
</table>

### More tools & templates

[`ai-editor-playbook`](https://github.com/c-a-p-engineer/ai-editor-playbook)
· [`VivliostyleTemplate`](https://github.com/c-a-p-engineer/VivliostyleTemplate)
· [`codex-phaser-template`](https://github.com/c-a-p-engineer/codex-phaser-template)
· [`skill-sheet-maker`](https://github.com/c-a-p-engineer/skill-sheet-maker)
· [`ResponsiveCapture`](https://github.com/c-a-p-engineer/ResponsiveCapture)

## How I build

```mermaid
flowchart LR
    A["Problem"] --> B["Scope & Decisions"]
    B --> C["README / Spec / Task"]
    C --> D["AI Implementation"]
    D --> E["Human Review"]
    E --> F["Tests / CI"]
    F --> G["Release / Pages"]
    G --> H["Observe & Improve"]

    classDef start fill:#13233a,stroke:#58a6ff,color:#f0f6fc;
    classDef plan fill:#211630,stroke:#a371f7,color:#f0f6fc;
    classDef build fill:#301525,stroke:#ff4f9a,color:#f0f6fc;
    classDef ship fill:#15251f,stroke:#3fb950,color:#f0f6fc;

    class A start;
    class B,C plan;
    class D,E build;
    class F,G,H ship;
```

- ツールではなく、解くべき問題から始める
- 実装前にREADME・仕様・タスクを成果物として書く
- AIは実装速度を上げ、人間は判断品質を担保する
- 個人技より、再現性・自動化・費用対効果を優先する

## Stack

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

## Writing & knowledge sharing

- [こぴぺたんログ](https://c-a-p-engineer.github.io/) — 開発、AI活用、自動化の実践記録
- [技術書典 サークルページ](https://techbookfest.org/organization/5zdy9h5eA5kDzByP9rserV) — AI協働開発と実務ノウハウを技術書として公開

---

<p align="center">
  <sub>
    This profile README is a human-in-the-loop artifact:<br />
    direction and judgment by the engineer, composition and implementation with AI.
  </sub>
</p>

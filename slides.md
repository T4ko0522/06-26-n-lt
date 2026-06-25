---
theme: geist
title: cf-edgeNix
info: Cloudflare-native な NixOS Binary Cache 基盤の話
author: 権代颯士
keywords: Nix,NixOS,Cloudflare,Binary Cache,Workers
transition: slide-left | slide-right
colorSchema: light
highlighter: shiki
lineNumbers: false
themeConfig:
  default: light
---

<div class="tag mb-6">下半期成果発表 / LT</div>

# <span class="gradient-text">cf-edgeNix</span>

再現可能な NixOS 環境を、**すぐ使える** NixOS 環境にする
Cloudflare-native な Binary Cache 基盤

<div class="divider" style="max-width: 8rem; margin: 2rem 0 1.25rem 0"></div>

<div style="color: var(--c-text-dim); font-size: 0.85rem">権代颯士 — <span style="color: var(--c-text-mute)">@T4ko0522</span></div>

<!--
今日は自作の cf-edgeNix という個人プロジェクトの話をします。
これは Cloudflare ネイティブな NixOS の Binary Cache 基盤で、
ひとことで言うと「再現可能な NixOS 環境を、すぐ使える NixOS 環境にする」ためのものです。
前半で Nix と NixOS を軽く紹介して、後半で cf-edgeNix の中身を話します。
-->

---

<div class="section-label">01 — Agenda</div>

# 今日話すこと

- **Nix** とは — 軽く
- **NixOS** とは — 軽く
- **cf-edgeNix** — 本題。Cloudflare で作る NixOS の Binary Cache

前半は前提の説明、後半が本題です。

<!--
流れはシンプルで、まず Nix が何で、それを OS に広げた NixOS が何か、を 1 枚ずつ。
そこから本題の cf-edgeNix に入ります。
本題では「なぜ作ったか」「どう解決したか」「Cloudflare の各サービスをどう役割分担させたか」を話します。
-->

---

<div class="section-label">02 — 前提</div>

<div class="logo-badge"><div class="i-simple-icons-nixos" style="color: #5277C3" /></div>

# Nix とは

ひとことで言うと **純粋関数型のパッケージマネージャ**。

「入力（依存）が同じなら出力（ビルド結果）も同じ」を徹底し、環境を**再現可能**にする。

- パッケージごとに `/nix/store` の独立したパスへ隔離
- グローバルを汚さない ＝ 依存の衝突が起きない
- 設定はすべて Nix 言語で記述する

<!--
Nix は純粋関数型のパッケージマネージャです。
"純粋関数型" というのは、入力（依存）が同じなら出力（ビルド結果）も同じを徹底するという意味で、
それを実現するために /nix/store/<hash>-<name> のように依存込みの hash 付きパスに各パッケージを隔離します。
グローバルな /usr/lib を汚さないので依存の衝突が起きない、設定は全部 Nix 言語で宣言的に書く、というのが特徴です。
-->

---

<div class="section-label">03 — 前提</div>

<div class="logo-badge"><div class="i-simple-icons-nixos" style="color: #5277C3" /></div>

# NixOS とは

Nix の仕組みを **OS 全体に広げた** Linux ディストリビューション。

パッケージ・サービス・ユーザー設定まで、**システム全体を宣言的に管理**する（Nix 言語で記述）。

- 設定 = システムの状態 が常に一致
- 更新ごとに「世代」が残り、いつでもロールバックできる
- 同じ設定 → どのマシンでも同じ OS が再現

<!--
NixOS は Nix の考え方を OS 全体に広げたディストリビューションです。
システム全体を宣言的に管理できるので、再現性が高いのが魅力です。
ここまでが前提で、ここからが本題です。
-->

---

<div class="section-label">04 — 本題</div>

# cf-edgeNix とは

**Cloudflare-native な NixOS Binary Cache 基盤。**

GitHub Actions でビルドした NixOS の system closure を、
Cloudflare（R2 / KV / D1 / Workers）上の global binary cache として配布する。

> 再現可能な NixOS 環境を、**すぐ使える** NixOS 環境にする。

<!--
cf-edgeNix は Cloudflare ネイティブな NixOS Binary Cache 基盤です。
GitHub Actions で NixOS の system closure をビルドして、
その成果物を Cloudflare（R2 / KV / D1 / Workers）の上に "global binary cache" として配置・配信する仕組みです。
Nix 標準の binary cache プロトコルに準拠しているので、
nixos-rebuild から substituters に足すだけで使えます。
-->

---

<div class="section-label">05 — Motivation</div>

# なぜ作ったか — 課題

NixOS は「**設定の再現性**」は高い。でも「**すぐ使える状態までの再現性**」には課題がある。

- `nixos-rebuild` が遅い
- 初回構築時に大量の derivation がローカルでビルドされる
- cache miss するとローカル CPU に負荷がかかる
- マシン性能やネットワーク次第で復元体験が変わる

→ 同じ設定でも、環境が完成するまでに時間がかかる。

<!--
NixOS は「設定の再現性」はとても高いんですが、「すぐ使える状態までの再現性」には課題があります。
具体的には nixos-rebuild が遅い、初回構築時に大量の derivation がローカルでビルドされる、
cache miss するとローカル CPU が回り続ける、マシン性能やネットワーク次第で復元の体験が大きく変わる、
ということが起きます。同じ configuration.nix でも、環境が完成するまでに時間がかかる、ということですね。
-->

---

<div class="section-label">06 — Approach</div>

# 解決方針

ビルド成果物を**ローカルで作らず、CI で事前に作って配布**する。

```text
GitHub Actions で NixOS system closure をビルド
  ↓
R2 へ NAR 本体を保存
  ↓
KV / D1 にメタデータを配置
  ↓
Workers が Nix binary cache endpoint として配信
  ↓
クライアントは cache から取得 → 重いビルドを回避
```

クライアント側は `substituters` にこの Worker を追加するだけ。

<!--
方針はシンプルで、重いビルドはローカルじゃなく CI に肩代わりさせる。
GitHub Actions で NixOS の closure をビルドして、その成果物を R2 に置き、
KV と D1 にメタデータを置いて、Workers が Nix の binary cache エンドポイントとして配信する。
クライアント側は extra-substituters にこの Worker URL を足して、
extra-trusted-public-keys に署名鍵の公開鍵を足すだけ。
ローカルでは可能な限り cache から取得して、重いビルドを避けることができます。
-->

---

<div class="section-label">07 — Architecture</div>

# 役割分担

- **GitHub Actions** — NixOS closure をビルドする signed builder
- **R2** — NAR / `.narinfo` の正本（source of truth）
- **KV** — narinfo の速度層（結果整合のキャッシュ）
- **D1** — build 履歴 / latest / rollback / GC の control plane
- **Workers** — Nix client 向けの HTTP gateway + 管理 API

read path（narinfo 取得）は **memory → KV → R2** で完結し、D1 を挟まない。

<!--
一番のポイントは役割分担で、4 つのサービスに「正本 / 速度層 / control plane / 窓口」をきれいに分けています。
R2 が正本（source of truth）で、.narinfo と NAR 本体の最終的な真実をここに置く。
KV は速度層で、結果整合の "速い噂" 程度に扱う。KV は正本にしない。
D1 は control plane で、build 履歴・host ごとの latest pointer・rollback root・GC の live set を持つ。
Workers は窓口で、Nix client 向けの HTTP gateway と管理 API を出す。

ここで一番大事な設計判断が、read path に D1 を絶対に挟まないこと。
.narinfo の取得は memory → KV → R2 だけで完結させます。
なぜかというと、nixos-rebuild は一度に何百という .narinfo を引きにくるので、
もし KV miss で D1 に雪崩れ込ませると、D1 が control plane じゃなくて hot metadata server に転落してしまう。
だから "KV miss → R2 に直接落とす" と決め切っています。
spec に書いた言い回しだと「KV は真実ではなく、速い噂。真実は R2 と D1 に置く」という分け方です。
-->

---

<div class="section-label">08 — Stack</div>

# 技術スタック

- **Cloudflare Workers** — エッジで動く HTTP gateway
- **R2 / KV / D1** — ストレージ・キャッシュ・DB
- **Hono + zod-openapi** — ルーティング / OpenAPI 自動生成
- **Drizzle ORM** — D1 のスキーマ・クエリ
- **TypeScript / Bun** — 実装・実行
- **vitest（workers-pool）** — unit / integration テスト

`nixos-rebuild` から直接叩ける標準の binary cache プロトコルに準拠。

<!--
技術スタックは Cloudflare をフルに使いつつ、上の層は型安全に寄せています。
Workers が HTTP gateway、R2 / KV / D1 がそれぞれストレージ・キャッシュ・DB。
Hono + zod-openapi で、Hono のルートと zod のスキーマから OpenAPI を自動生成。
Drizzle ORM で D1 のスキーマとクエリを TypeScript 側の型と合わせる。
実装は TypeScript / Bun、テストは vitest の workers-pool で実際の workerd 上で integration テストを回す。

ポイントは、これ全部が Nix 標準の binary cache プロトコルの上に乗っていて、
クライアント側に専用ツールを一切インストールさせないこと。
extra-substituters に URL を 1 行足すだけで使えます。
-->

---

<div class="section-label">09 — Summary</div>

# まとめ

- NixOS は「設定の再現性」は高いが「すぐ使えるまで」が遅い
- cf-edgeNix は CI で事前ビルド → Cloudflare で配布してそれを解決
- R2 / KV / D1 / Workers を役割分担させた edge-native な構成

> 再現可能な NixOS 環境を、すぐ使える NixOS 環境に。

<!--
まとめると、NixOS は「設定の再現性」は高いけれど「すぐ使えるまでが遅い」。
cf-edgeNix はそこを、CI で事前にビルドして Cloudflare で配布することで解決します。
役割分担は R2 が正本、KV が速度層、D1 が control plane、Workers が窓口で、
read path に D1 を挟まないという edge-native な設計に振り切っている。
狙いは一貫していて、「再現可能な NixOS 環境を、すぐ使える NixOS 環境にする」ことです。
-->

---
layout: center
---

# <span class="gradient-text">ありがとうございました</span>

<div class="divider" style="max-width: 8rem; margin: 1.5rem auto 2rem auto"></div>

<div style="color: var(--c-text-dim); font-size: 0.9rem; line-height: 2">
github.com/t4ko0522/cf-edgeNix<br/>
<span style="color: var(--c-text-mute)">@T4ko0522 ／ t4ko.vercel.app</span>
</div>

<!--
以上で発表を終わります。ありがとうございました。
-->

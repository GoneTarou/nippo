# HTML + Tailwind → PlaywrightでOGP画像生成

## 概要

HTMLでOGP画像用のページを作成し、そのページをPlaywrightでスクリーンショットしてPNG画像を生成する方法です。

最近のRailsアプリでよく採用されている方法で、CSSやTailwindをそのまま利用できるのが最大のメリットです。

---

## 全体の流れ

```text
ユーザーが日記を公開
        │
        ▼
RailsでOGP用HTMLを表示
(/diaries/:id/ogp)
        │
        ▼
Playwrightがページを開く
        │
        ▼
スクリーンショット撮影
        │
        ▼
PNG画像生成
        │
        ▼
<meta property="og:image">に設定
        │
        ▼
X(Twitter)などでURL共有
        │
        ▼
画像付きカードが表示される
```

---

## ① OGP専用ページを作る

通常のRailsビューを作るだけです。

```
app/views/diaries/ogp.html.erb
```

例

```erb
<div class="w-[1200px] h-[630px] bg-pink-100 flex flex-col justify-center items-center">

  <h1 class="text-6xl font-bold">
    四文字日記
  </h1>

  <p class="mt-10 text-4xl">
    <%= @diary.body %>
  </p>

  <p class="mt-6 text-xl">
    <%= @diary.diary_date %>
  </p>

</div>
```

ブラウザで開くと

```
┌──────────────────────────┐
│                          │
│       四文字日記          │
│                          │
│     今日も頑張った        │
│                          │
│      2026/07/15          │
│                          │
└──────────────────────────┘
```

のような画面になります。

---

## ② Tailwindでデザイン

普通のRails画面と同じようにTailwindを書くだけです。

例えば

```html
<div class="
bg-gradient-to-r
from-pink-300
to-orange-300
rounded-3xl
shadow-xl
">
```

これがそのまま画像になります。

つまり

```
HTML = OGP画像
```

という考え方です。

---

## ③ Playwrightで画像化

Playwrightはブラウザを自動操作するツールです。

人間なら

```
Chromeを開く
↓
ページを表示
↓
スクリーンショット
```

を行います。

Playwrightなら

```
ページを開く
↓
スクリーンショット
```

をプログラムで実行できます。

イメージ

```
OGPページ
      │
      ▼
Playwright
      │
      ▼
PNG画像完成
```

---

## ④ OGP画像として利用

生成した画像を

```html
<meta property="og:image" content="https://example.com/ogp/123.png">
```

として設定します。

すると

```
https://example.com/diaries/123
```

をXで共有すると

```
┌──────────────────────────┐
│        OGP画像            │
│                          │
│      今日も頑張った       │
│                          │
└──────────────────────────┘

https://example.com/diaries/123
```

のようなカードが表示されます。

---

# この方法のメリット

- HTMLを書くだけで画像になる
- Tailwindがそのまま使える
- Figmaのデザインを再現しやすい
- CSSだけでデザイン変更できる
- Rubyで画像を描画する必要がない
- Railsとの相性が良い

---

# Rubyだけで画像生成する方法との違い

## Rubyで画像生成

```
Ruby
│
├─ 四角を書く
├─ 文字を書く
├─ 座標を計算
├─ 色を設定
└─ PNG生成
```

デザイン変更が大変です。

---

## HTML + Playwright

```
HTML + Tailwind
        │
        ▼
ブラウザ表示
        │
        ▼
Playwright
        │
        ▼
PNG生成
```

デザイン変更はHTML・CSSを修正するだけです。

---

# この方法が向いているアプリ

- SNSシェア機能
- ブログ
- 日記アプリ
- ポートフォリオ
- ニュースサイト
- ECサイト

---

# あなたの日記アプリでの完成イメージ

```
┌────────────────────────────┐
│        🌸 四文字日記        │
│                            │
│    今日も頑張った          │
│                            │
│       2026.07.15           │
│                            │
│      4WordsDiary           │
└────────────────────────────┘
```

このデザインをTailwindで作り、PlaywrightがスクリーンショットしてOGP画像として利用します。

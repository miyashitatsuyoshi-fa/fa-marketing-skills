---
name: delivery-model-web-publisher
description: 宅配便モデル（Delivery Model）マーケティング戦略の設計書を、ビジュアルリッチなシングルページWebサイト（HTML/CSS/JS）として構築し、Manusサンドボックスのローカルサーバーとして公開URLを発行するスキル。「宅配便モデルの結果をWebサイトにして」「デプロイして」「公開して」「作品をWebに変換して」と依頼があったとき、またはDelivery Modelのキャンペーン設計書が完成した直後に必ずこのスキルを使用する。delivery-model-marketing スキルと組み合わせて使うことで、戦略立案から公開まで一気通貫で完結する。
---

# Delivery Model Web Publisher スキル

このスキルは、宅配便モデル（Delivery Model）で作成したマーケティング戦略設計書を、プロフェッショナルなシングルページWebサイトとして構築し、**Manusサンドボックスのローカルサーバー（Python http.server + expose）** を通じて公開URLを発行するためのワークフローを定義します。

> **デプロイ先の変更履歴**: 以前はCloudflare Workersを使用していたが、wrangler CLIの認証エラーが頻発するため、Manusサンドボックスのローカルサーバー公開方式をデフォルトに変更（2026-03-26）。

---

## このスキルが解決すること

マーケティング戦略の設計書は、Markdownドキュメントのままでは関係者への共有や説得力に限界があります。このスキルを使うことで、設計書を誰でもブラウザで閲覧できるビジュアルリッチなWebページへ変換し、URLを共有するだけで社内外に戦略を伝えられるようになります。

---

## ワークフロー（必ずこの順序で実行する）

### Step 1: 入力コンテンツの確認

`delivery-model-marketing` スキルで作成したキャンペーン設計書（Markdownまたは会話履歴）を入力として受け取る。以下の6要素が揃っているか確認する。

- 送り主 (Sender)
- 届け先 (Target)
- 荷物 (Package / Message)
- 気づき (Insight)
- 反応の把握 (Response Measurement)
- 乗り物 (Vehicle / Media)

不足している要素があれば、ユーザーに確認してから次のステップへ進む。

### Step 2: HTMLファイルの設計と構築

以下の設計原則に従い、`/home/ubuntu/delivery-model-site/index.html` として単一HTMLファイルを作成する。

#### デザイン原則

- **カラーパレット**: 商品・サービスのブランドイメージに合わせた配色を選択する（例：博多うどんなら温かみのある琥珀色・白・深緑）
- **レイアウト**: フルスクリーンヒーローセクション → 6要素カード → リスク検証セクション → CTAの順で構成する
- **タイポグラフィ**: Google Fonts から日本語フォント（Noto Sans JP または Zen Kaku Gothic New）を読み込む
- **アニメーション**: スクロール連動のフェードイン（Intersection Observer API）を実装し、各カードが順番に現れる演出を加える
- **レスポンシブ**: モバイルファーストで設計し、CSS Grid / Flexbox を活用する
- **インタラクション**: 6要素の各カードにホバーエフェクト（transform: translateY + box-shadow）を実装する

#### HTMLファイルの必須セクション構成

```
1. <head>: meta charset, viewport, title, Google Fonts, インラインCSS
2. ヒーローセクション: キャンペーン名、サブタイトル、グラデーション背景
3. 宅配便モデル6要素セクション: カードグリッド（各カードに番号・アイコン・タイトル・説明）
4. リスク検証セクション: 3つのリスクと対策（アコーディオンまたはカード形式）
5. まとめ・CTAセクション: スキルの価値を一言で表現
6. フッター: 作成日、Powered by Delivery Model Marketing
```

#### アイコンの使用

外部ライブラリは使わず、SVGインラインアイコンまたはUnicode絵文字を使用する（CDN依存を最小化するため）。

### Step 3: Manusサンドボックスへのデプロイ（デフォルト）

**Cloudflare WorkersではなくManusサンドボックスのローカルサーバーを使用する。** これがデフォルトのデプロイ方法。

#### デプロイ手順

```bash
# 1. プロジェクトディレクトリへ移動
cd /home/ubuntu/delivery-model-site

# 2. 既存のサーバープロセスを停止（ポート競合を防ぐ）
pkill -f "python3 -m http.server 8080" 2>/dev/null || true

# 3. Python簡易HTTPサーバーをバックグラウンドで起動
python3 -m http.server 8080 &

# 4. expose ツールでポート8080を公開URLとして発行
# → エージェントの expose ツール（port: 8080）を呼び出す
```

#### ポート競合時の対処

```bash
# ポート8080が使用中の場合は別のポート（例: 8081）を使用
python3 -m http.server 8081 &
# expose ツールも port: 8081 で呼び出す
```

#### デプロイ後の確認

- `expose` ツールが返す `https://<hash>.manus.computer` 形式のURLをユーザーに報告する
- URLはサンドボックスが稼働している間のみ有効（セッション終了で無効化）

### Step 4: 結果の報告

ユーザーへの最終報告には以下を含める。

- 公開URL（`https://<hash>.manus.computer` 形式）
- サイトの主要セクション説明
- リスク検証の要約
- 注意事項：サンドボックスが非アクティブになるとURLが無効化される旨
- 次のアクション提案（例：HTMLファイルをダウンロードして別途ホスティング）

---

## 注意事項とベストプラクティス

- HTMLファイルはすべてのCSS・JSをインラインに含めた**完全自己完結型**とし、外部CDNへの依存を最小限にする（Google Fontsのみ許容）
- 日本語コンテンツが含まれるため、必ず `<meta charset="UTF-8">` を設定する
- Manusサンドボックスの公開URLはセッション中のみ有効。永続的なホスティングが必要な場合は、生成した `index.html` をユーザーに添付して渡す
- **Cloudflare Workersは使用しない**：wrangler CLIの認証フロー（OAuth）がサンドボックス環境では完了できないため

---

## 組み合わせ使用パターン

このスキルは `delivery-model-marketing` スキルと組み合わせることで最大の効果を発揮する。

```
ユーザー: 「○○を宅配便モデルで戦略化して、Webサイトにデプロイして」
  ↓
Step A: delivery-model-marketing スキルで6要素設計書を作成
  ↓
Step B: delivery-model-web-publisher スキルでWebサイト化・Manusサンドボックスに公開
  ↓
完成: 公開URLをユーザーに報告 + index.htmlを添付
```

---

## 出力例

デプロイ成功時の報告フォーマット:

```
## デプロイ完了

**公開URL**: https://8080-xxxxxxxx.manus.computer

**サイト構成**:
- ヒーローセクション: キャンペーン概要
- 6要素カード: 宅配便モデルの全要素
- リスク検証: 3つのリスクと対策
- まとめ・CTA

**注意**: 公開URLはManusサンドボックスが稼働中のみ有効です。
HTMLファイルを添付しますので、永続公開が必要な場合はご自身のサーバーにアップロードしてください。

**次のステップ**:
- HTMLファイルをダウンロードして任意のWebサーバーに配置可能
- コンテンツを更新する場合: index.html を編集してサーバーを再起動
```

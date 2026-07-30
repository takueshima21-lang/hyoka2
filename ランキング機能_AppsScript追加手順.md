# ランキング機能を動かすために Apps Script 側へ追加するコード

修正版の tantou_final.html / tokatsu_final.html には「🏆 全職員ランキングを見る」ボタンを追加しました。
このボタンは Apps Script（GAS_URL）に `?action=ranking&period=xxx` というGETリクエストを送り、
「最終スコア確定済み（本部調整後）」の全職員データを役職混在・250点満点で受け取って順位付け表示します。

現在のApps Scriptには `action=get`（個人集計用）はあっても、全職員分をまとめて返す `action=ranking` がおそらく無いため、
以下の関数を既存の `doGet(e)` の中に追記してください。

## 追加するコード例（doGet内、if(action==='get'){...} の並びに追加）

```javascript
if (action === 'ranking') {
  const period = e.parameter.period || '';
  const ss = SpreadsheetApp.openById('18YCgpyaQb02h2fVC9uX0F0PexaJ2HQxw9wQD7L5RvrM');
  // 最終スコアを保存しているシート名に置き換えてください（例: '最終スコア' や 'finalize'）
  const sheet = ss.getSheetByName('最終スコア');
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  const rows = [];
  for (let i = 1; i < data.length; i++) {
    const row = {};
    headers.forEach((h, idx) => row[h] = data[i][idx]);
    if (!period || row.period === period) {
      rows.push({
        target: row.target,
        cls: row.cls,
        roleLabel: row.roleLabel,
        finalScore: row.finalScore,
        finalRank: row.finalRank
      });
    }
  }
  return ContentService.createTextOutput(JSON.stringify({ rows }))
    .setMimeType(ContentService.MimeType.JSON);
}
```

## 前提として直しておきたい点

現状の `confirmFinalScore()` が送る保存データ（`action:'finalize'`）に `cls`（学童保育所名）と `roleLabel`（役職）が
含まれていない可能性があります。修正版の tantou_final.html / tokatsu_final.html では `roleLabel` を追加送信するようにしましたが、
`cls` は集計画面（screen-agg-search）で氏名しか入力しない作りのため、Apps Script側の保存処理でも
`target`（氏名）と合わせて `cls` を記録できるよう、保存側のシートに列を用意しておくと、ランキング表に学童保育所名も出せます。

## 反映手順

1. https://script.google.com/ にアクセスし、対象のプロジェクトを開く
2. `doGet(e)` 関数を探し、上記のコードを追加
3. 「デプロイ」→「デプロイを管理」→ 既存のウェブアプリの編集アイコンをクリック
4. 新バージョンとして「デプロイ」（URLは変わりません）

これで tantou_final.html / tokatsu_final.html の「🏆 全職員ランキングを見る」ボタンが機能するようになります。

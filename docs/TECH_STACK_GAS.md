# YAJU & MONSTER - 技術スタック設計書（GAS版）

## 1. Google Apps Script（GAS）推奨理由

### 1.1 スマートフォン開発に最適な理由

| 特徴 | 利点 |
|-----|------|
| **インストール不要** | ブラウザだけでOK、すぐに開発開始可能 |
| **スマートフォン対応** | iOS/Android どちらでも動作 |
| **クラウド統合** | Google Sheet がデータベース、自動バックアップ |
| **無料ホスティング** | Google のサーバーで無制限実行 |
| **スマートフォンでコード編集** | テキストエディタアプリで GAS 編集可能 |
| **リアルタイム機能** | Google Realtime API で同時編集対応 |

### 1.2 完全無料

```
Google Apps Script    : 無制限実行
Google Sheet         : 無制限（約500万セル）
Google Drive         : 15GB まで無料
Firebase（オプション） : 月25万書き込みまで無料

→ 初期 1-2年完全無料運用可能
```

---

## 2. 技術スタック概要

```
┌──────────────────────────────────────────────┐
│     YAJU & MONSTER - GAS版 技術スタック     │
├──────────────────────────────────────────────┤
│ フロント    │ HTML5 + CSS + JavaScript      │
│             │ Canvas でアニメーション実装    │
│             │ 素材：テキスト/SVG/簡易画像  │
├──────────────────────────────────────────────┤
│ バック      │ Google Apps Script（GAS）    │
│             │ ゲームロジック実装           │
│             │ REST API エンドポイント      │
├──────────────────────────────────────────────┤
│ DB         │ Google Sheet                  │
│             │ プレイヤーデータ             │
│             │ モンスター図鑑               │
│             │ ガチャテーブル               │
│             │ ランキング                   │
├──────────────────────────────────────────────┤
│ リアルタイム│ Polling（定期通信）           │
│             │ WebSocket（別途検討）        │
├──────────────────────────────────────────────┤
│ ホスティング │ Google Apps Script（無料）   │
├──────────────────────────────────────────────┤
│ ストレージ  │ Google Drive / Firebase Storage |
└──────────────────────────────────────────────┘
```

---

## 3. Google Sheet データベース設計

### 3.1 Sheet 構成

```
YAJU-MONSTER（Google Sheet）
│
├── 📄 Players シート
│   ├── playerId | nickname | level | exp | hp | atk | def | spd | int | res | coins | gems | createdAt | lastLogin
│
├── 📄 Monsters シート
│   ├── monsterId | name | baseHp | baseAtk | baseDef | baseSpd | baseInt | baseRes | rarity | attribute | type
│
├── 📄 PlayerMonsters シート
│   ├── playerMonsterId | playerId | monsterId | level | exp | hp | atk | def | spd | int | res | plus（凸数） | skills | acquiredAt
│
├── 📄 Battles シート
│   ├── battleId | playerId | enemyMonsterId | winner | playerDamage | enemyDamage | turnCount | reward | timestamp
│
├── 📄 Rankings シート
│   ├── playerId | nickname | rank | wins | losses | powerScore | lastUpdate
│
├── 📄 GachaTable シート
│   ├── gachaId | type | itemId | itemName | rarity | dropRate | baseStats
│
├── 📄 Items シート
│   ├── itemId | name | type | effect | description
│
├── 📄 Messages シート
│   ├── messageId | playerId | nickname | message | room | timestamp
│
└── 📄 Config シート
    ├── 定数（最大レベル、基本HP等）
    ├── ゲームバージョン
    ├── メンテナンスモード設定
```

### 3.2 Players シートの例

| playerId | nickname | level | exp | hp | atk | def | spd | int | res | coins | gems | createdAt |
|----------|----------|-------|-----|----|----|-----|-----|-----|-----|-------|-------|-----------|
| user_001 | YAJU | 5 | 250 | 150 | 25 | 15 | 12 | 20 | 15 | 5000 | 100 | 2026-09-06 |
| user_002 | Monster | 10 | 1000 | 250 | 40 | 25 | 18 | 35 | 25 | 15000 | 500 | 2026-09-01 |

### 3.3 PlayerMonsters シートの例

| playerMonsterId | playerId | monsterId | level | exp | plus | skills | acquiredAt |
|-----------------|----------|-----------|-------|-----|------|--------|-----------|
| pm_001 | user_001 | monster_fire_001 | 10 | 1500 | 2 | [skill_1, skill_2, skill_3, skill_4] | 2026-09-06 |
| pm_002 | user_001 | monster_water_001 | 8 | 800 | 0 | [skill_5, skill_6] | 2026-09-05 |

---

## 4. Google Apps Script コード構成

### 4.1 ファイル構成

```
Google Apps Script プロジェクト（yaju-monster）
│
├── 📄 main.gs
│   └── doGet()           → HTML UI 配信
│       doPost()          → API 呼び出し処理
│
├── 📄 auth.gs
│   ├── createUser()      → ユーザー登録
│   ├── loginUser()       → ログイン認証
│   └── generateToken()   → JWT トークン生成
│
├── 📄 players.gs
│   ├── getPlayer()       → プレイヤー情報取得
│   ├── updatePlayer()    → プレイヤー情報更新
│   └── getPlayerStats()  → ステータス計算
│
├── 📄 monsters.gs
│   ├── getAllMonsters()  → 全モンスター取得
│   ├── getPlayerMonsters() → プレイヤー所持モンスター
│   └── addMonster()      → モンスター追加（ガチャ等）
│
├── 📄 battle.gs
│   ├── startBattle()     → バトル開始
│   ├── calculateDamage() → ダメージ計算
│   ├── nextTurn()        → ターン進行
│   └── endBattle()       → バトル終了・報酬
│
├── 📄 gacha.gs
│   ├── drawGacha()       → ガチャ実行
│   ├── getGachaTable()   → ガチャテーブル取得
│   └── calculateDropRate() → 確率計算
│
├── 📄 ranking.gs
│   ├── getRanking()      → ランキング取得
│   ├── updateRanking()   → ランキング更新
│   └── getPowerScore()   → パワースコア計算
│
├── 📄 database.gs
│   ├── getSheet()        → Sheet 取得
│   ├── getRow()          → 行取得
│   ├── setRow()          → 行更新
│   ├── addRow()          → 行追加
│   └── query()           → 検索
│
├── 📄 utils.gs
│   ├── generateId()      → ID 生成
│   ├── getTimestamp()    → タイムスタンプ取得
│   ├── hashPassword()    → パスワードハッシュ
│   └── log()             → ログ出力
│
└── 📄 constants.gs
    └── ゲーム定数（最大レベル、属性等）
```

### 4.2 main.gs の基本構造

```javascript
// main.gs
function doGet(e) {
  // HTML UI を返す
  return HtmlService.createHtmlOutput(getHtmlUI())
    .setWidth(window.innerWidth)
    .setHeight(window.innerHeight);
}

function doPost(e) {
  // クライアントからの API リクエスト処理
  const action = e.parameter.action;
  const data = JSON.parse(e.postData.contents);
  
  try {
    let result;
    switch(action) {
      case 'login':
        result = loginUser(data.email, data.password);
        break;
      case 'register':
        result = createUser(data.email, data.password, data.nickname);
        break;
      case 'getPlayer':
        result = getPlayer(data.playerId);
        break;
      case 'startBattle':
        result = startBattle(data.playerId, data.monsterId);
        break;
      case 'drawGacha':
        result = drawGacha(data.playerId, data.gachaType);
        break;
      case 'getRanking':
        result = getRanking(data.limit);
        break;
      default:
        result = { error: '不正なアクション' };
    }
    
    return ContentService.createTextOutput(JSON.stringify(result))
      .setMimeType(ContentService.MimeType.JSON);
  } catch(error) {
    return ContentService.createTextOutput(JSON.stringify({ error: error.toString() }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}

function getHtmlUI() {
  return `<!DOCTYPE html>
    <html>
      <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <title>YAJU & MONSTER</title>
        <link rel="stylesheet" href="https://example.com/styles.css">
      </head>
      <body>
        <div id="app"></div>
        <script src="https://example.com/game.js"></script>
      </body>
    </html>`;
}
```

### 4.3 battle.gs の例（ダメージ計算）

```javascript
// battle.gs
function calculateDamage(attackerStats, defenderStats, skillData) {
  // ダメージ計算式（詳細仕様書の計算式を実装）
  
  let baseDamage;
  
  if(skillData.type === 'physical') {
    // 物理ダメージ
    baseDamage = (attackerStats.atk * 1.2 - defenderStats.def * 0.5) 
               * skillData.power / 100;
  } else if(skillData.type === 'magic') {
    // 魔法ダメージ
    baseDamage = (attackerStats.int * 1.2 - defenderStats.res * 0.5) 
               * skillData.power / 100;
  }
  
  // 属性相性による補正
  let attributeMultiplier = 1.0;
  if(isEffectiveAttribute(skillData.attribute, defenderStats.attribute)) {
    attributeMultiplier = 1.5;
  } else if(isWeakAttribute(skillData.attribute, defenderStats.attribute)) {
    attributeMultiplier = 0.75;
  }
  
  // クリティカル判定
  let criticalMultiplier = 1.0;
  const critRate = skillData.criticalRate + (attackerStats.spd / 100);
  if(Math.random() * 100 < critRate) {
    criticalMultiplier = 1.5;
  }
  
  // 最終ダメージ
  let finalDamage = Math.max(1, Math.floor(
    baseDamage * attributeMultiplier * criticalMultiplier
  ));
  
  return {
    damage: finalDamage,
    isCritical: criticalMultiplier > 1.0
  };
}

function startBattle(playerId, enemyMonsterId) {
  // バトル開始処理
  const player = getPlayer(playerId);
  const playerMonsters = getPlayerMonsters(playerId);
  const enemyMonster = getMonster(enemyMonsterId);
  
  const battleId = generateId('battle');
  const battle = {
    id: battleId,
    playerId: playerId,
    enemyId: enemyMonsterId,
    turn: 1,
    playerTeam: playerMonsters.slice(0, 2), // 最大2体
    enemy: enemyMonster,
    log: []
  };
  
  // GAS の ScriptProperties に バトル状態を保存
  PropertiesService.getUserProperties().setProperty(
    `battle_${battleId}`, 
    JSON.stringify(battle)
  );
  
  return {
    battleId: battleId,
    message: 'バトル開始'
  };
}
```

### 4.4 gacha.gs の例（ガチャシステム）

```javascript
// gacha.gs
function drawGacha(playerId, gachaType) {
  // ガチャ実行
  const player = getPlayer(playerId);
  
  // コスト確認
  const gachaTable = getGachaTable(gachaType);
  if(player.gems < gachaTable.cost) {
    return { error: 'ジェムが足りません' };
  }
  
  // ガチャテーブルから確率抽選
  const result = drawFromTable(gachaTable.items);
  
  // プレイヤーにアイテム付与
  if(gachaType.includes('monster')) {
    addMonsterToPlayer(playerId, result.itemId);
  } else {
    addItemToPlayer(playerId, result.itemId, result.quantity);
  }
  
  // ジェム消費
  updatePlayer(playerId, {
    gems: player.gems - gachaTable.cost
  });
  
  return {
    success: true,
    item: result,
    remainingGems: player.gems - gachaTable.cost
  };
}

function drawFromTable(items) {
  // 確率抽選
  const random = Math.random() * 100;
  let cumulative = 0;
  
  for(let item of items) {
    cumulative += item.dropRate;
    if(random < cumulative) {
      return item;
    }
  }
  
  // フォールバック（通常は発生しない）
  return items[items.length - 1];
}

function getGachaTable(gachaType) {
  const sheet = getSheet('GachaTable');
  const data = sheet.getDataRange().getValues();
  
  const gachaData = data.filter(row => row[0] === gachaType);
  
  return {
    type: gachaType,
    cost: 300, // ジェム
    items: gachaData.map(row => ({
      itemId: row[1],
      itemName: row[2],
      rarity: row[3],
      dropRate: parseFloat(row[4])
    }))
  };
}
```

---

## 5. HTML5 + Canvas フロントエンド

### 5.1 HTML 構成

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>YAJU & MONSTER</title>
    <style>
      * {
        margin: 0;
        padding: 0;
        box-sizing: border-box;
      }
      body {
        font-family: Arial, sans-serif;
        background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
        min-height: 100vh;
        display: flex;
        justify-content: center;
        align-items: center;
      }
      #game-container {
        width: 100%;
        max-width: 500px;
        height: 100vh;
        background: white;
        display: flex;
        flex-direction: column;
        box-shadow: 0 0 20px rgba(0,0,0,0.3);
      }
      #game-canvas {
        flex: 1;
        background: #f0f0f0;
      }
      .ui-buttons {
        display: flex;
        gap: 10px;
        padding: 10px;
        flex-wrap: wrap;
      }
      button {
        flex: 1;
        padding: 10px;
        background: #667eea;
        color: white;
        border: none;
        border-radius: 5px;
        cursor: pointer;
        font-weight: bold;
      }
      button:hover {
        background: #764ba2;
      }
    </style>
  </head>
  <body>
    <div id="game-container">
      <canvas id="game-canvas"></canvas>
      <div class="ui-buttons">
        <button id="btn-battle">バトル</button>
        <button id="btn-gacha">ガチャ</button>
        <button id="btn-inventory">所持品</button>
        <button id="btn-ranking">ランキング</button>
      </div>
    </div>
    <script src="game.js"></script>
  </body>
</html>
```

### 5.2 JavaScript ゲームロジック（game.js）

```javascript
// game.js
const canvas = document.getElementById('game-canvas');
const ctx = canvas.getContext('2d');

// キャンバスサイズ調整
function resizeCanvas() {
  canvas.width = canvas.offsetWidth;
  canvas.height = canvas.offsetHeight;
}
resizeCanvas();
window.addEventListener('resize', resizeCanvas);

// ゲーム状態
let gameState = {
  screen: 'title', // title, battle, gacha, inventory, ranking
  playerId: null,
  currentBattle: null
};

// API 呼び出し関数
async function callGAS(action, data) {
  try {
    const response = await google.script.run
      .withSuccessHandler(result => result)
      .withFailureHandler(error => {
        console.error('エラー:', error);
        return null;
      })
      .doPost({
        action: action,
        data: JSON.stringify(data)
      });
    return response;
  } catch(error) {
    console.error('API呼び出しエラー:', error);
  }
}

// バトル画面描画
function drawBattle() {
  ctx.clearRect(0, 0, canvas.width, canvas.height);
  
  // 背景
  ctx.fillStyle = '#87CEEB';
  ctx.fillRect(0, 0, canvas.width, canvas.height);
  
  // 敵モンスター描画
  ctx.fillStyle = '#FF6B6B';
  ctx.fillRect(canvas.width / 2, 50, 80, 80);
  ctx.fillStyle = 'white';
  ctx.fillText('敵モンスター', canvas.width / 2 + 5, 105);
  
  // プレイヤーモンスター描画
  ctx.fillStyle = '#4ECDC4';
  ctx.fillRect(50, canvas.height - 150, 80, 80);
  ctx.fillStyle = 'white';
  ctx.fillText('味方モンスター', 50, canvas.height - 65);
  
  // ターン情報
  ctx.fillStyle = 'black';
  ctx.font = '16px Arial';
  ctx.fillText(`ターン: ${gameState.currentBattle?.turn || 1}`, 10, 30);
}

// ボタンイベント
document.getElementById('btn-battle').addEventListener('click', async () => {
  if(!gameState.playerId) {
    alert('ログインしてください');
    return;
  }
  
  const result = await callGAS('startBattle', {
    playerId: gameState.playerId,
    monsterId: 'monster_001'
  });
  
  if(result.success) {
    gameState.currentBattle = result.battle;
    gameState.screen = 'battle';
    drawBattle();
  }
});

document.getElementById('btn-gacha').addEventListener('click', async () => {
  const result = await callGAS('drawGacha', {
    playerId: gameState.playerId,
    gachaType: 'monster_gacha'
  });
  
  if(result.success) {
    alert(`ガチャ結果: ${result.item.itemName}`);
  }
});

// ゲームループ
function gameLoop() {
  switch(gameState.screen) {
    case 'battle':
      drawBattle();
      break;
    // 他の画面処理
  }
  
  requestAnimationFrame(gameLoop);
}

// ゲーム開始
gameLoop();
```

---

## 6. GAS データベース操作関数

### 6.1 database.gs（汎用データベース関数）

```javascript
// database.gs
function getSheet(sheetName) {
  const spreadsheet = SpreadsheetApp.getActiveSpreadsheet();
  return spreadsheet.getSheetByName(sheetName);
}

function getRow(sheetName, condition) {
  // 例：condition = { playerId: 'user_001' }
  const sheet = getSheet(sheetName);
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  
  for(let i = 1; i < data.length; i++) {
    let match = true;
    for(let key in condition) {
      const colIndex = headers.indexOf(key);
      if(data[i][colIndex] !== condition[key]) {
        match = false;
        break;
      }
    }
    if(match) {
      return rowToObject(data[i], headers);
    }
  }
  return null;
}

function setRow(sheetName, condition, updates) {
  // 既存行を更新
  const sheet = getSheet(sheetName);
  const data = sheet.getDataRange().getValues();
  const headers = data[0];
  
  for(let i = 1; i < data.length; i++) {
    let match = true;
    for(let key in condition) {
      const colIndex = headers.indexOf(key);
      if(data[i][colIndex] !== condition[key]) {
        match = false;
        break;
      }
    }
    
    if(match) {
      for(let key in updates) {
        const colIndex = headers.indexOf(key);
        sheet.getRange(i + 1, colIndex + 1).setValue(updates[key]);
      }
      return true;
    }
  }
  return false;
}

function addRow(sheetName, values) {
  // 新規行を追加
  const sheet = getSheet(sheetName);
  sheet.appendRow(values);
  return true;
}

function rowToObject(row, headers) {
  const obj = {};
  for(let i = 0; i < headers.length; i++) {
    obj[headers[i]] = row[i];
  }
  return obj;
}
```

---

## 7. 開発フロー（スマートフォン環境）

### 7.1 開発環境セットアップ

```bash
# 1. Google アカウント作成（既に持っている場合はスキップ）
# https://accounts.google.com/signup

# 2. Google Apps Script で新規プロジェクト作成
# https://script.google.com

# 3. 新規 Google Sheet を作成してデータベース構築
# https://sheets.google.com
```

### 7.2 スマートフォンでのコード編集方法

**方法1：Google Apps Script エディタ（推奨）**
- https://script.google.com にアクセス
- スマートフォンのブラウザで GAS コードを直接編集
- ボタンを押すと自動保存

**方法2：テキストエディタアプリ使用**
- QuickEdit（Android）や TextEdit（iOS）などのテキストエディタ
- コードを作成 → Google Drive に保存
- Google Apps Script でインポート

**方法3：Termux（Android のみ）**
- Termux アプリをインストール
- Node.js と git をインストール
- GitHub から クローン＆開発

### 7.3 開発スケジュール（速度重視）

```
Week 1  : Google Sheet データベース構築
         Google Apps Script プロジェクト作成
         ログイン・ユーザー登録機能
         
Week 2  : モンスター図鑑・表示機能
         HTML5 + Canvas UI 実装
         
Week 3  : バトルシステム（ダメージ計算含む）
         
Week 4  : ガチャシステム実装
         
Week 5-6: ランキング・PvP
         チャット機能（Polling方式）
         
Week 6-8: バグ修正・UI改善
         パフォーマンス最適化
         
→ **ウェブアプリとしてリリース** 🎉
```

---

## 8. スマートフォンでのデバッグ方法

### 8.1 GAS Execution Log

```javascript
// ログを出力してデバッグ
Logger.log('プレイヤーID: ' + playerId);
Logger.log('ダメージ: ' + damage);
Logger.log('エラー: ' + error.toString());

// Google Apps Script エディタの
// 「実行ログ」タブで確認
```

### 8.2 ウェブアプリのデバッグ

```javascript
// console.log をウェブアプリ内に表示
const debugLog = [];
console.log = function(msg) {
  debugLog.push(msg);
  document.getElementById('debug').innerText = debugLog.join('\n');
};
```

### 8.3 リアルタイム テスト

```
1. Google Apps Script 左上の「デプロイ」をクリック
2. 「新しいデプロイ」を選択
3. 種類：「ウェブアプリ」
4. 実行者：「自分」
5. アクセス：「全員」
6. デプロイ
7. 表示された URL をスマートフォンで開く
```

---

## 9. セキュリティ対策

### 9.1 認証・認可

```javascript
// JWT トークン管理
function generateToken(userId) {
  const header = btoa(JSON.stringify({
    alg: 'HS256',
    typ: 'JWT'
  }));
  
  const payload = btoa(JSON.stringify({
    userId: userId,
    iat: Date.now(),
    exp: Date.now() + (24 * 60 * 60 * 1000) // 24時間有効
  }));
  
  // 署名（簡易版、本番は JWT ライブラリ使用）
  const signature = Utilities.computeDigest(
    Utilities.DigestAlgorithm.SHA_256,
    header + '.' + payload
  );
  
  return header + '.' + payload + '.' + btoa(signature);
}

// リクエストごとにトークン検証
function verifyToken(token) {
  const parts = token.split('.');
  if(parts.length !== 3) return null;
  
  const payload = JSON.parse(atob(parts[1]));
  
  // 有効期限確認
  if(payload.exp < Date.now()) {
    return null;
  }
  
  return payload;
}
```

### 9.2 入力値検証

```javascript
function validateInput(data) {
  // メールアドレス検証
  if(!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email)) {
    throw new Error('無効なメールアドレス');
  }
  
  // ニックネーム検証（20文字以下）
  if(data.nickname.length > 20) {
    throw new Error('ニックネームは20文字以下');
  }
  
  // パスワード検証（8文字以上）
  if(data.password.length < 8) {
    throw new Error('パスワードは8文字以上');
  }
}
```

---

## 10. コスト概算

### 10.1 無料範囲内

| サービス | 無料枠 |
|--------|-------|
| Google Apps Script | 無制限 |
| Google Sheet | 500万セル（十分） |
| Google Drive | 15GB |
| **月額費用** | **¥0** |

### 10.2 有料化タイミング（想定）

```
用途            推定費用/月
Google Drive追加容量   ¥2.5K（100GB）
Firebase（オプション）  ¥0～5K
CDN（画像配信）      ¥0～2K
ドメイン取得        ¥100～1K/年

→ 初期1年間完全無料で運用可能
→ ユーザー数増加時に検討
```

---

## 11. メリット・デメリット再評価

### ✅ メリット

- **スマートフォンだけで開発・運用可能**
- **完全無料（スケール時も低コスト）**
- **セットアップが最短（即開始可能）**
- **Google のサービスで信頼性◎**
- **スケール時も機能追加が容易**
- **バックアップ自動化**

### ⚠️ デメリット

- **グラフィックスは限定的（Canvas か SVG）**
- **リアルタイム処理は工夫が必要（WebSocket 別途実装）**
- **大規模マルチプレイは GAS のみでは困難**
- **学習リソース少ない（ゲーム開発向けではない）**
- **実行時間に制限あり（1実行あたり最大6分）**

### 対策案

- **グラフィック：** Phaser.js（Canvas ゲームフレームワーク）導入で改善
- **リアルタイム：** Firebase Realtime DB 組み合わせで実現
- **大規模化時：** Node.js に移行（GAS コードはほぼ流用可能）

---

## 12. 今後の拡張パス

### Phase 1（GAS）：初期リリース
```
GAS + Google Sheet + HTML5
↓
ウェブアプリとしてリリース
```

### Phase 2（ハイブリッド）：成長フェーズ
```
GAS + Firebase（リアルタイム機能）
↓
マルチプレイ対応
```

### Phase 3（スケール）：大規模化時
```
Node.js + PostgreSQL へ段階的移行
↓
ネイティブアプリ化（React Native 等）
```

**GAS から Node.js への移行は容易（コードロジックはほぼ流用可能）**

---

## 13. 次のアクション

- [ ] Google Apps Script プロジェクト作成
- [ ] Google Sheet でデータベース設計
- [ ] ユーザー認証機能実装
- [ ] HTML5 UI 作成
- [ ] バトルシステム実装
- [ ] ガチャシステム実装
- [ ] テスト＆デバッグ
- [ ] ウェブアプリ公開

---

**最終更新日：** 2026年9月6日  
**ステータス：** GAS版技術スタック決定版 v1.0  
**推奨：** スマートフォン開発に最適化した設計

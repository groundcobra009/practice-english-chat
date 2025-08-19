# API設計書

## 概要

英語学習チャットシステムのAPI設計書です。Next.js 14 (App Router) を基盤とし、Supabase認証・データベース、AI SDK、ElevenLabs TTS APIを統合した設計となっています。

## 基本情報

- **ベースURL**: `https://your-domain.com/api`
- **認証方式**: JWT (Supabase Auth)
- **データ形式**: JSON
- **文字エンコーディング**: UTF-8

## 認証

### JWT Token

Supabaseの認証システムを使用し、すべての保護されたエンドポイントにはAuthorizationヘッダーが必要です。

```http
Authorization: Bearer <jwt_token>
```

### 認証関連エンドポイント

#### 1. ユーザー登録
```http
POST /auth/signup
```

**リクエスト**
```json
{
  "email": "user@example.com",
  "password": "password123",
  "display_name": "ユーザー名"
}
```

**レスポンス (201 Created)**
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "display_name": "ユーザー名",
    "created_at": "2024-01-01T00:00:00Z"
  },
  "session": {
    "access_token": "jwt_token",
    "refresh_token": "refresh_token",
    "expires_at": 1640995200
  }
}
```

#### 2. ログイン
```http
POST /auth/login
```

**リクエスト**
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**レスポンス (200 OK)**
```json
{
  "user": {
    "id": "uuid",
    "email": "user@example.com",
    "display_name": "ユーザー名"
  },
  "session": {
    "access_token": "jwt_token",
    "refresh_token": "refresh_token",
    "expires_at": 1640995200
  }
}
```

#### 3. ログアウト
```http
POST /auth/logout
```

**リクエスト**
```json
{
  "refresh_token": "refresh_token"
}
```

**レスポンス (200 OK)**
```json
{
  "message": "Successfully logged out"
}
```

#### 4. トークン更新
```http
POST /auth/refresh
```

**リクエスト**
```json
{
  "refresh_token": "refresh_token"
}
```

**レスポンス (200 OK)**
```json
{
  "session": {
    "access_token": "new_jwt_token",
    "refresh_token": "new_refresh_token",
    "expires_at": 1640995200
  }
}
```

## チャット機能 API

### 1. チャットセッション作成
```http
POST /chat/sessions
```

**リクエスト**
```json
{
  "title": "英語での挨拶について"
}
```

**レスポンス (201 Created)**
```json
{
  "session": {
    "id": "session_uuid",
    "title": "英語での挨拶について",
    "user_id": "user_uuid",
    "created_at": "2024-01-01T00:00:00Z"
  }
}
```

### 2. 学習メッセージ生成
```http
POST /chat/generate
```

**リクエスト**
```json
{
  "session_id": "session_uuid",
  "user_message": "ビジネスでの挨拶を学びたい",
  "level": "intermediate"
}
```

**レスポンス (200 OK)**
```json
{
  "messages": [
    {
      "id": "message_uuid_1",
      "role": "assistant",
      "content": {
        "text_en": "Good morning, how are you doing today?",
        "text_ja": "おはようございます、今日の調子はいかがですか？",
        "explanation": "ビジネスシーンでの丁寧な朝の挨拶です。"
      },
      "metadata": {
        "level": "intermediate",
        "category": "business_greeting"
      },
      "created_at": "2024-01-01T00:00:00Z"
    },
    {
      "id": "message_uuid_2",
      "role": "assistant",
      "content": {
        "text_en": "It's a pleasure to meet you.",
        "text_ja": "お会いできて光栄です。",
        "explanation": "初対面のビジネス相手への挨拶です。"
      },
      "metadata": {
        "level": "intermediate",
        "category": "business_greeting"
      },
      "created_at": "2024-01-01T00:00:00Z"
    },
    {
      "id": "message_uuid_3",
      "role": "assistant",
      "content": {
        "text_en": "I look forward to working with you.",
        "text_ja": "一緒にお仕事させていただくのを楽しみにしています。",
        "explanation": "今後の協力関係を表現する丁寧な挨拶です。"
      },
      "metadata": {
        "level": "intermediate",
        "category": "business_greeting"
      },
      "created_at": "2024-01-01T00:00:00Z"
    }
  ],
  "session_id": "session_uuid"
}
```

### 3. チャット履歴取得
```http
GET /chat/sessions/{session_id}/messages
```

**パラメータ**
- `limit` (optional): 取得件数 (デフォルト: 50)
- `offset` (optional): オフセット (デフォルト: 0)

**レスポンス (200 OK)**
```json
{
  "messages": [
    {
      "id": "message_uuid",
      "role": "user",
      "content": "ビジネスでの挨拶を学びたい",
      "created_at": "2024-01-01T00:00:00Z"
    },
    {
      "id": "message_uuid_1",
      "role": "assistant",
      "content": {
        "text_en": "Good morning, how are you doing today?",
        "text_ja": "おはようございます、今日の調子はいかがですか？",
        "explanation": "ビジネスシーンでの丁寧な朝の挨拶です。"
      },
      "metadata": {
        "level": "intermediate",
        "category": "business_greeting"
      },
      "created_at": "2024-01-01T00:00:00Z"
    }
  ],
  "total_count": 4,
  "has_more": false
}
```

### 4. セッション一覧取得
```http
GET /chat/sessions
```

**パラメータ**
- `limit` (optional): 取得件数 (デフォルト: 20)
- `offset` (optional): オフセット (デフォルト: 0)

**レスポンス (200 OK)**
```json
{
  "sessions": [
    {
      "id": "session_uuid",
      "title": "英語での挨拶について",
      "created_at": "2024-01-01T00:00:00Z",
      "updated_at": "2024-01-01T00:00:00Z",
      "message_count": 4
    }
  ],
  "total_count": 1,
  "has_more": false
}
```

## 音声合成 (TTS) API

### 1. 音声生成
```http
POST /tts/generate
```

**リクエスト**
```json
{
  "text": "Good morning, how are you doing today?",
  "voice_id": "us_female_1",
  "speed": 1.0
}
```

**レスポンス (200 OK)**
```json
{
  "audio_url": "https://blob-storage-url/audio_file_uuid.mp3",
  "duration": 3.2,
  "voice_id": "us_female_1",
  "expires_at": "2024-01-02T00:00:00Z"
}
```

### 2. 音声設定取得
```http
GET /tts/voices
```

**レスポンス (200 OK)**
```json
{
  "voices": [
    {
      "id": "us_female_1",
      "name": "Sarah (US Female)",
      "language": "en-US",
      "gender": "female",
      "sample_url": "https://sample-audio-url/sarah.mp3"
    },
    {
      "id": "us_male_1",
      "name": "John (US Male)",
      "language": "en-US",
      "gender": "male",
      "sample_url": "https://sample-audio-url/john.mp3"
    }
  ]
}
```

## ブックマーク API

### 1. ブックマーク作成
```http
POST /bookmarks
```

**リクエスト**
```json
{
  "message_id": "message_uuid",
  "text_en": "Good morning, how are you doing today?",
  "text_ja": "おはようございます、今日の調子はいかがですか？",
  "notes": "ビジネスシーンでの挨拶"
}
```

**レスポンス (201 Created)**
```json
{
  "bookmark": {
    "id": "bookmark_uuid",
    "user_id": "user_uuid",
    "message_id": "message_uuid",
    "text_en": "Good morning, how are you doing today?",
    "text_ja": "おはようございます、今日の調子はいかがですか？",
    "notes": "ビジネスシーンでの挨拶",
    "audio_url": null,
    "play_count": 0,
    "created_at": "2024-01-01T00:00:00Z",
    "last_played_at": null
  }
}
```

### 2. ブックマーク一覧取得
```http
GET /bookmarks
```

**パラメータ**
- `search` (optional): 英語文での検索
- `sort` (optional): `created_at` | `last_played_at` | `play_count` (デフォルト: `created_at`)
- `order` (optional): `asc` | `desc` (デフォルト: `desc`)
- `limit` (optional): 取得件数 (デフォルト: 20)
- `offset` (optional): オフセット (デフォルト: 0)

**レスポンス (200 OK)**
```json
{
  "bookmarks": [
    {
      "id": "bookmark_uuid",
      "text_en": "Good morning, how are you doing today?",
      "text_ja": "おはようございます、今日の調子はいかがですか？",
      "notes": "ビジネスシーンでの挨拶",
      "audio_url": "https://blob-storage-url/audio_file_uuid.mp3",
      "play_count": 5,
      "created_at": "2024-01-01T00:00:00Z",
      "last_played_at": "2024-01-01T12:00:00Z"
    }
  ],
  "total_count": 1,
  "has_more": false
}
```

### 3. ブックマーク更新
```http
PUT /bookmarks/{bookmark_id}
```

**リクエスト**
```json
{
  "notes": "更新されたメモ",
  "audio_url": "https://new-audio-url.mp3"
}
```

**レスポンス (200 OK)**
```json
{
  "bookmark": {
    "id": "bookmark_uuid",
    "user_id": "user_uuid",
    "text_en": "Good morning, how are you doing today?",
    "text_ja": "おはようございます、今日の調子はいかがですか？",
    "notes": "更新されたメモ",
    "audio_url": "https://new-audio-url.mp3",
    "play_count": 5,
    "created_at": "2024-01-01T00:00:00Z",
    "last_played_at": "2024-01-01T12:00:00Z",
    "updated_at": "2024-01-01T15:00:00Z"
  }
}
```

### 4. ブックマーク削除
```http
DELETE /bookmarks/{bookmark_id}
```

**レスポンス (204 No Content)**

### 5. 再生回数更新
```http
POST /bookmarks/{bookmark_id}/play
```

**レスポンス (200 OK)**
```json
{
  "bookmark": {
    "id": "bookmark_uuid",
    "play_count": 6,
    "last_played_at": "2024-01-01T16:00:00Z"
  }
}
```

## ユーザー設定 API

### 1. ユーザー設定取得
```http
GET /users/preferences
```

**レスポンス (200 OK)**
```json
{
  "preferences": {
    "tts_voice_id": "us_female_1",
    "tts_speed": 1.0,
    "language": "ja",
    "learning_level": "intermediate"
  }
}
```

### 2. ユーザー設定更新
```http
PUT /users/preferences
```

**リクエスト**
```json
{
  "tts_voice_id": "us_male_1",
  "tts_speed": 1.2,
  "language": "ja",
  "learning_level": "advanced"
}
```

**レスポンス (200 OK)**
```json
{
  "preferences": {
    "tts_voice_id": "us_male_1",
    "tts_speed": 1.2,
    "language": "ja",
    "learning_level": "advanced"
  }
}
```

## フィードバック API

### 1. フィードバック送信
```http
POST /feedback
```

**リクエスト**
```json
{
  "message_id": "message_uuid",
  "type": "helpful",
  "value": true,
  "comment": "とても役に立ちました"
}
```

**レスポンス (201 Created)**
```json
{
  "feedback": {
    "id": "feedback_uuid",
    "message_id": "message_uuid",
    "user_id": "user_uuid",
    "type": "helpful",
    "value": true,
    "comment": "とても役に立ちました",
    "created_at": "2024-01-01T00:00:00Z"
  }
}
```

## エラーレスポンス

### エラー形式

すべてのエラーは以下の形式で返されます：

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human readable error message",
    "details": "Additional error details (optional)"
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "path": "/api/endpoint"
}
```

### HTTPステータスコード

| コード | 説明 | 使用場面 |
|--------|------|----------|
| 200 | OK | 正常なリクエスト |
| 201 | Created | リソースの作成成功 |
| 204 | No Content | 削除成功 |
| 400 | Bad Request | リクエストパラメータエラー |
| 401 | Unauthorized | 認証エラー |
| 403 | Forbidden | 権限エラー |
| 404 | Not Found | リソースが見つからない |
| 409 | Conflict | データの重複エラー |
| 422 | Unprocessable Entity | バリデーションエラー |
| 429 | Too Many Requests | レート制限エラー |
| 500 | Internal Server Error | サーバー内部エラー |
| 502 | Bad Gateway | 外部API連携エラー |
| 503 | Service Unavailable | サービス利用不可 |

### エラーコード一覧

#### 認証関連
- `AUTH_INVALID_CREDENTIALS`: 認証情報が無効
- `AUTH_TOKEN_EXPIRED`: トークンの有効期限切れ
- `AUTH_TOKEN_INVALID`: 無効なトークン
- `AUTH_USER_NOT_FOUND`: ユーザーが見つからない
- `AUTH_EMAIL_ALREADY_EXISTS`: メールアドレスが既に使用されている

#### リクエスト関連
- `VALIDATION_ERROR`: バリデーションエラー
- `MISSING_REQUIRED_FIELD`: 必須フィールドが不足
- `INVALID_FIELD_FORMAT`: フィールド形式が無効
- `RESOURCE_NOT_FOUND`: リソースが見つからない

#### AI/TTS関連
- `AI_SERVICE_UNAVAILABLE`: AI推論サービスが利用不可
- `AI_RATE_LIMIT_EXCEEDED`: AI推論のレート制限に達した
- `TTS_SERVICE_UNAVAILABLE`: TTS音声合成サービスが利用不可
- `TTS_GENERATION_FAILED`: 音声生成に失敗

#### データベース関連
- `DATABASE_CONNECTION_ERROR`: データベース接続エラー
- `DATABASE_QUERY_ERROR`: データベースクエリエラー
- `CONSTRAINT_VIOLATION`: データベース制約違反

### エラー例

#### 認証エラー
```json
{
  "error": {
    "code": "AUTH_TOKEN_EXPIRED",
    "message": "Authentication token has expired",
    "details": "Please refresh your token or log in again"
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "path": "/api/chat/generate"
}
```

#### バリデーションエラー
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Request validation failed",
    "details": {
      "user_message": "Message cannot be empty",
      "level": "Level must be one of: beginner, intermediate, advanced"
    }
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "path": "/api/chat/generate"
}
```

#### AIサービスエラー
```json
{
  "error": {
    "code": "AI_SERVICE_UNAVAILABLE",
    "message": "AI service is temporarily unavailable",
    "details": "Please try again later"
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "path": "/api/chat/generate"
}
```

## レート制限

### 制限値

| エンドポイント | 制限 | 期間 |
|----------------|------|------|
| `/auth/*` | 10 requests | 1分 |
| `/chat/generate` | 30 requests | 1時間 |
| `/tts/generate` | 100 requests | 1時間 |
| `/bookmarks/*` | 1000 requests | 1時間 |
| その他 | 300 requests | 1時間 |

### レート制限ヘッダー

```http
X-RateLimit-Limit: 30
X-RateLimit-Remaining: 25
X-RateLimit-Reset: 1640995200
```

### レート制限エラー

```json
{
  "error": {
    "code": "RATE_LIMIT_EXCEEDED",
    "message": "Rate limit exceeded",
    "details": "Try again in 3600 seconds"
  },
  "timestamp": "2024-01-01T00:00:00Z",
  "path": "/api/chat/generate"
}
```

## セキュリティ

### HTTPS/TLS
- すべての通信はHTTPS/TLS暗号化必須
- TLS 1.2以上をサポート

### 認証・認可
- JWT トークンベース認証
- Row Level Security (RLS) によるデータベースレベル認可
- トークン自動更新機能

### データ保護
- 入力値サニタイゼーション (XSS防止)
- SQLインジェクション対策
- AI プロンプトインジェクション対策

### プライバシー
- ユーザーデータの地域内保管
- データ削除要請への対応
- 最小権限の原則

## パフォーマンス

### 応答時間目標
- AI メッセージ生成: < 5秒
- TTS 音声生成: < 1.5秒
- データベースクエリ: < 500ms
- 認証処理: < 200ms

### キャッシュ戦略
- AI 生成結果: クライアント側キャッシュ (1時間)
- TTS 音声: CDN キャッシュ (24時間)
- ユーザー設定: メモリキャッシュ (30分)

### 最適化
- データベース接続プーリング
- Edge Functions による地理的分散
- バンドルサイズ最適化
- 遅延読み込み実装

## 監視・ログ

### メトリクス
- API レスポンス時間
- エラー率
- レート制限ヒット率
- AI/TTS サービス可用性

### ログ形式
```json
{
  "timestamp": "2024-01-01T00:00:00Z",
  "level": "info",
  "message": "API request completed",
  "method": "POST",
  "path": "/api/chat/generate",
  "status": 200,
  "duration": 1.23,
  "user_id": "user_uuid",
  "request_id": "req_uuid"
}
```

### エラー追跡
- Sentry による例外監視
- スタックトレース収集
- ユーザーコンテキスト保持

## バージョニング

### API バージョン管理
- URL パスでのバージョン指定: `/api/v1/endpoint`
- 後方互換性の維持 (最低6ヶ月)
- 非推奨機能の段階的廃止

### 変更管理
- 破壊的変更は新バージョンでのみ実装
- 非破壊的変更は既存バージョンに追加
- 変更ログの詳細記録

## テスト

### API テスト
- 単体テスト: Jest/Vitest
- 統合テスト: Cypress E2E
- API テスト: REST Client / Postman

### テストデータ
- 開発環境用のシードデータ
- テスト用のモックデータ
- 本番データの匿名化

### 自動テスト
- CI/CD パイプラインでの自動実行
- プルリクエスト毎のテスト実行
- デプロイ前のスモークテスト

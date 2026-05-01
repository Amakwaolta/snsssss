# ひとこと SNS — セットアップガイド

## 必要なもの（すべて無料・ブラウザのみ）
- GitHubアカウント
- Supabaseアカウント

---

## ステップ1：Supabaseでデータベースを作る

1. https://supabase.com にアクセスしてGitHubでサインアップ
2. 「New project」をクリック
3. プロジェクト名（例：`hitokoto-sns`）とパスワードを設定して作成
4. プロジェクトが作成されたら左メニューの「SQL Editor」を開く
5. 以下のSQLを貼り付けて「Run」をクリック：

```sql
-- 投稿テーブル
create table posts (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users not null,
  username text not null,
  content text not null check (char_length(content) <= 280),
  liked_by uuid[] default '{}',
  created_at timestamptz default now()
);

-- 誰でも投稿を読める
alter table posts enable row level security;

create policy "誰でも読める" on posts
  for select using (true);

create policy "自分の投稿のみ作成" on posts
  for insert with check (auth.uid() = user_id);

create policy "自分の投稿のみ削除" on posts
  for delete using (auth.uid() = user_id);

create policy "いいね更新は全員OK" on posts
  for update using (true);

-- リアルタイム有効化
alter publication supabase_realtime add table posts;
```

6. 左メニューの「Project Settings」→「API」を開く
7. 以下の2つをメモする：
   - **Project URL**（`https://xxxxx.supabase.co`）
   - **anon public** キー

---

## ステップ2：index.htmlにキーを設定する

`index.html` の上部の以下の部分を書き換える：

```js
const SUPABASE_URL  = 'YOUR_SUPABASE_URL';   // ← Project URLに変更
const SUPABASE_ANON = 'YOUR_SUPABASE_ANON_KEY'; // ← anon keyに変更
```

---

## ステップ3：GitHubで公開する

1. https://github.com にアクセス
2. 「New repository」で新しいリポジトリを作成（名前例：`hitokoto-sns`）
3. 「uploading an existing file」から `index.html` をアップロード
4. リポジトリの「Settings」→「Pages」→「Source: main branch」を選択して保存
5. しばらく待つと `https://あなたのID.github.io/hitokoto-sns/` で公開される！

---

## 友達を招待する方法

公開されたURLを友達に共有するだけ。
友達はそのURLにアクセスして「新規登録」でアカウントを作れます。

---

## 機能一覧

- ✅ ユーザー登録・ログイン
- ✅ 投稿（280文字）
- ✅ タイムライン（リアルタイム更新）
- ✅ いいね
- ✅ 自分の投稿を削除
- ✅ iPad・iPhone対応

# basic_design.md  

## 1. Project Overview
| Item | Description |
|------|-------------|
| **Project Name** | 学生 ↔ 企業 マッチング Web アプリ |
| **Purpose** | 学校の卒業生リスト（CSV）をインポートし、得意分野に合わせた入社先企業を自動マッチングして管理者が簡単に閲覧・編集できるようにする。 |
| **Target Users** | ① 学校運営側（管理者）<br>② 学生本人（閲覧のみ） |
| **Hosting Platform** | Vercel (GitHub 連携自動デプロイ) |
| **Primary Tech Stack** | Next.js 13 (App Router) + React (Hooks) <br> TypeScript <br> Prisma ORM <br> PlanetScale MySQL (managed) <br> TailwindCSS (UI) <br> Jest + React‑Testing‑Library (テスト) |
| **Timeline (概算)** | 6 週間（要件定義 1 wk → 設計 1 wk → 開発 3 wk → テスト & デプロイ 1 wk） |
| **Budget (概算)** | 別途見積もり（工数 120 h × 時間単価） |

---

## 2. Scope of Work (SOW)

| Phase | Deliverable | Acceptance Criteria |
|-------|-------------|----------------------|
| **A. 要件定義・設計** | - 完全版機能要件・非機能要件書 <br>- データモデル定義書 <br>- 画面ワイヤーフレーム（Markdown/PlantUML） <br>- API 仕様書 | すべての項目がステークホルダーの署名（承認）を得ること |
| **B. 環境構築** | - GitHub リポジトリ（ブランチ戦略、CI/CD） <br>- Vercel プロジェクト作成 <br>- PlanetScale データベース作成・接続設定 | `npm run dev` でローカルが 200 OK、Vercel preview デプロイが成功すること |
| **C. 実装** | 1. CSV インポート機能 <br>2. 学生・企業 CRUD API（管理者） <br>3. マッチングロジック (得意分野 ↔ 必要スキル) <br>4. UI：管理画面、学生一覧、企業一覧、マッチ結果一覧 <br>5. 認証（NextAuth.js + Email+Password） | - すべての主要機能が UI で操作でき、ユニットテストカバレッジ ≥ 80 % <br>- 主要フローが 2 秒以内に完了 |
| **D. テスト・品質保証** | - ユニットテスト、統合テスト、E2E テスト（Playwright） <br>- パフォーマンス測定（Lighthouse） <br>- セキュリティ診断（OWASP ZAP） | 全テストが Pass、Lighthouse performance ≥ 90、セキュリティレポートで中程度以上の脆弱性なし |
| **E. デプロイ & 移行** | - Vercel 本番環境へのデプロイ <br>- DB マイグレーション（PlanetScale） <br>- 操作マニュアル・ユーザーマニュアル | 本番 URL で全機能が正常に動作し、ステークホルダーが「受領」サインを行うこと |
| **F. 保守 (Optional)** | - 3 か月間のバグフィックス SLA  (48 h以内) <br>- 月次レポート（利用統計、パフォーマンス） | 契約書に定義された保守範囲内で対応すること |

---

## 3. Assumptions & Constraints
| # | 内容 |
|---|------|
| 1 | 学校側が提供する学生情報は上記 CSV 形式で一定期間更新されること。 |
| 2 | 企業情報は管理者が手動で登録／更新する。最低 3 つの「業種カテゴリ」(例: Web Frontend, Backend, Data Science, etc.) があらかじめ設定されている。 |
| 3 | Vercel の Serverless Functions の実行時間は 10 秒以内。長時間処理はバックグラウンド (PlanetScale のスケジュールドジョブ) に委譲。 |
| 4 | 認証はシンプルな Email/Password のみ（外部 SSO は現段階で必要なし）。 |
| 5 | データベースは PlanetScale の MySQL クラスタを使用し、Vercel 環境変数で接続情報を管理。 |
| 6 | UI は PC とタブレット（横幅 768 px 以上）に最適化、スマートフォンはシンプルリストビューで閲覧可能。 |
| 7 | デザインは TailwindCSS のデフォルトコンポーネントで実装し、別途カスタムブランディングは後工程で追加可能。 |

---

## 4. Functional Requirements (機能要件)

| FR‑ID | 機能概要 | 詳細 |
|-------|----------|------|
| **FR‑001** | **CSV 学生データインポート** | - 管理者は CSV ファイル (UTF‑8, カンマ区切り) をアップロード <br>- バリデーション：必須項目 (氏名, フリガナ, 生年月日, 性別, 得意分野) が全行存在 <br>- 重複チェック（氏名＋生年月日で一意） <br>- 成功時は DB に **Student** テーブルへ保存、インポート結果サマリ表示 |
| **FR‑002** | **学生 CRUD** | - 一覧表示（ページング: 20 件/ページ） <br>- 詳細画面で閲覧・編集 (氏名, フリガナ, 生年月日, 性別, 得意分野) <br>- 削除 (論理削除: `isDeleted` フラグ) |
| **FR‑003** | **企業 CRUD** | - 企業属性：会社名, 業種カテゴリ (Enum), 必要スキル (複数選択), 会社概要, 勤務地, 採用人数 <br>- 管理者は UI で CRUD 可能 |
| **FR‑004** | **マッチングロジック** | - 学生の「得意分野」 ↔ 企業の「必要スキル」 が完全一致または部分一致した場合を **マッチ** と判定 <br>- 部分一致は以下の優先度でスコア付け：<br> 1. 完全一致 (スコア 100) <br> 2. キーワードが1語一致 (80) <br> 3. カテゴリが同一 (70) <br>- 同一学生に対して複数企業がヒットした場合、スコア降順で上位 3 件まで表示 |
| **FR‑005** | **マッチング結果閲覧** | - 学生一覧 → 「マッチング結果」ボタンでその学生のおすすめ企業リストを表示 <br>- 企業一覧 → 「マッチング対象学生」タブで該当学生一覧を表示 |
| **FR‑006** | **検索・フィルタ** | - 学生一覧/企業一覧でキーワード検索（氏名/会社名） <br>- フィルタ: 性別、得意分野/業種カテゴリ、スコア範囲 |
| **FR‑007** | **認証・権限** | - NextAuth.js の Email/Password プロバイダー <br>- ロール: `admin`（全権限） と `viewer`（閲覧のみ） <br>- 認証が必要なページはサーバーサイドで保護 |
| **FR‑008** | **レポート機能 (Optional)** | - CSV エクスポート (学生リスト、マッチング結果) <br>- 簡易統計: カテゴリ別学生人数、マッチング成功率 |
| **FR‑009** | **通知 (Optional)** | - マッチング結果が更新されたら管理者へ Email 通知（SendGrid API） |
| **FR‑010** | **多言語対応 (Future)** | - UI のテキストは i18n (next‑i18next) にて日本語・英語切替可能 (初期は日本語のみ) |

---

## 5. Non‑Functional Requirements (非機能要件)

| NFR‑ID | 項目 | 目標 / 基準 |
|--------|------|--------------|
| **NFR‑001** | **パフォーマンス** | - 初回ページロード ≤ 2 秒 (Lighthouse `Performance` ≥ 90) <br>- API 応答時間 ≤ 500 ms（平均） |
| **NFR‑002** | **可用性** | - Vercel の SLA (99.9 %) に準拠 <br>- DB (PlanetScale) のフェイルオーバー構成 |
| **NFR‑003** | **拡張性** | - 新しい業種カテゴリやスキルを *Enum* に追加しやすい設計 <br>- Prisma のスキーママイグレーションでスムーズに拡張 |
| **NFR‑04** | **セキュリティ** | - OWASP Top‑10 に準拠 <br>- パスワードは bcrypt (cost=12) でハッシュ化 <br>- CSRF トークン／SameSite Cookie 設定 <br>- データベース接続は TLS 暗号化 |
| **NFR‑05** | **アクセシビリティ** | - WCAG 2.1 AA 準拠 <br>- キーボードナビゲーションと ARIA ラベルの実装 |
| **NFR‑06** | **テスト性** | - ユニットテストカバレッジ ≥ 80 % <br>- E2E テスト (Playwright) で主要フロー 100 % カバー |
| **NFR‑07** | **デプロイ・運用** | - GitHub Actions による CI (lint, test, build) <br>- Vercel のプレビュー環境で PR ごとに自動デプロイ |
| **NFR‑08** | **ログ・監視** | - Vercel の Edge Logs + Sentry エラートラッキング <br>- データベースクエリのタイムアウト監視 |
| **NFR‑09** | **データ保持** | - 学生情報は GDPR / 日本の個人情報保護法に準拠し、退会時は完全削除 |
| **NFR‑10** | **国際化（将来）** | - 文字コードは UTF‑8、日付は ISO‑8601 (UTC) で保存 |

---

## 6. Data Model (ER図)

```
+----------------+       +-----------------+       +----------------+
|   Student      | 1   n |   Match         | n   1 |   Company      |
+----------------+-------+-----------------+-------+----------------+
| id (PK)        |       | id (PK)         |       | id (PK)        |
| name           |       | student_id (FK) |       | name           |
| kana           |       | company_id (FK) |       | industry (enum)|
| birthdate      |       | score (int)     |       | requiredSkills |
| gender (enum)  |       | matchedAt (datetime) | location |
| skill (enum)   |       | createdAt       |       | description    |
| createdAt      |       | updatedAt       |       | createdAt      |
| updatedAt      |       +-----------------+       | updatedAt      |
+----------------+                                 +----------------+
```

- **Student**: CSV からインポート → `skill` は事前定義した Enum（例: `WebFrontEnd`, `DataScience`, `Backend`, `UIUX`, `ML`, `Mobile`, `CloudInfra`, `Database`, `Security`, `GameDev`, `DevOps`, `AI` 等）。  
- **Company**: 管理者が手入力。`requiredSkills` はカンマ区切りの文字列／JSON 配列として保存し、検索時は `LIKE` または Prisma の `contains` で比較。  
- **Match**: 学生と企業のマッチング結果。`score` はロジックで算出し、上位 3 件の表示に利用。

---

## 7. Architecture Overview

```
┌─────────────────────┐
│   GitHub Repository │
│ (Next.js + Prisma)  │
└───────┬─────────────┘
        │   CI (GitHub Actions)
        ▼
┌─────────────────────┐
│    Vercel (Edge)    │
│  - Next.js Frontend │
│  - Serverless API   │
│    (pages/api/**)   │
└───────┬─────────────┘
        │
        ▼
┌─────────────────────┐
│   PlanetScale MySQL  │
│   (Production DB)   │
│   - Student table   │
│   - Company table   │
│   - Match table     │
└─────────────────────┘
```

**Key Points**
- **Static Generation** (`getStaticProps`) for public pages (e.g., landing) for SEO.
- **Server‑Side Rendering** (`getServerSideProps`) for admin dashboard to guarantee latest data.
- **API Routes** (`/api/students`, `/api/companies`, `/api/match`) handle CRUD & matching logic, protected by NextAuth session.
- **Prisma** generates type‑safe DB client. Migrations managed via `prisma migrate`.
- **Vercel Environment Variables**: `DATABASE_URL`, `NEXTAUTH_SECRET`, `NEXTAUTH_URL`, `EMAIL_SERVER`, `EMAIL_FROM`.
- **CI** runs `eslint`, `prettier`, `typecheck`, `unit test`, `build`. Deploys on merge to `main`.
- **Monitoring**: Sentry (error), Vercel Analytics (performance), PlanetScale Insight (slow queries).

---

## 8. Wireframes (画面設計)

### 8.1. 共通レイアウト
```
+-------------------------------------------------------+
| Header (Logo | Nav: Dashboard | Logout)              |
+-------------------------------------------------------+
| Sidebar (only admin)                                 |
|  - Students   | Companies   | Matches   | Settings |
+-------------------------------------------------------+
| Main Content (dynamic)                               |
+-------------------------------------------------------+
| Footer (© 2025)                                      |
+-------------------------------------------------------+
```

### 8.2. Dashboard (Admin Home)
```
+-------------------------------------------------------+
| ① 総学生数: 60 名   ② 企業数: 8 社   ③ 本日マッチング数: 23 |
+-------------------------------------------------------+
|  [マッチング結果（上位 5 件）]                        |
| ---------------------------------------------------- |
| | 学生 | 企業 | スコア | マッチ日時 | アクション | |
| |------|------|-------|------------|------------| |
| | 山田太郎 | 株式会社A | 100 | 2025‑08‑07 10:12 | 詳細 |
| | ...    | ...  | ...   | ...        | ...      |
+-------------------------------------------------------+
|  [最近のインポート履歴]                              |
| ---------------------------------------------------- |
| | ファイル名 | 件数 | インポート日時 | 結果 | |
| ---------------------------------------------------- |
+-------------------------------------------------------+
```

### 8.3. Student List
```
+-------------------------------------------------------+
|  【検索】 ＿＿＿＿＿＿＿   【フィルタ】 性別 ▼   得意分野 ▼ |
+-------------------------------------------------------+
|  表示件数: 1‑20 / 60                                 |
+-------------------------------------------------------+
|  ID | 氏名 | フリガナ | 生年月日 | 性別 | 得意分野 | マッチング | |
| ----+------+----------+-----------+------+-----------+----------- |
|  1 | 山田太郎 | ヤマダタロウ | 2005‑04‑15 | 男 | Webフロントエンド | 5件 |
|  2 | 佐藤花子 | サトウハナコ | 2005‑06‑22 | 女 | データサイエンス | 3件 |
| ...                                                |
+-------------------------------------------------------+
|  [ページング] << 1 2 3 4 >>                        |
+-------------------------------------------------------+
```

### 8.4. Student Detail / マッチ結果
```
+-------------------------------------------------------+
|   学生詳細   (山田太郎)                               |
+-------------------------------------------------------+
| 氏名: 山田太郎      | 生年月日: 2005‑04‑15          |
| フリガナ: ヤマダタロウ | 性別: 男                     |
| 得意分野: Webフロントエンド                        |
+-------------------------------------------------------+
| 【マッチング結果】                                   |
| ---------------------------------------------------- |
| | 企業名 | 必要スキル | スコア | 採用状況 | アクション | |
| | 株式会社A | WebFrontEnd, UIUX | 100 | 未応募 | 詳細 |
| | 株式会社B | WebFrontEnd | 80 | 面談済 | 詳細 |
+-------------------------------------------------------+
| 【マッチング再計算】 (ボタン)                         |
+-------------------------------------------------------+
```

### 8.5. Company List
```
+-------------------------------------------------------+
|  【検索】 ＿＿＿＿＿＿＿   【フィルタ】 業種 ▼   必要スキル ▼ |
+-------------------------------------------------------+
|  ID | 会社名 | 業種 | 必要スキル | マッチ学生数 | アクション |
| ----+--------+------+-----------+-------------+-----------|
| 1 | 株式会社A | Web | WebFrontEnd, UIUX | 12 | 編集・削除 |
| 2 | 株式会社B | Data | DataScience, AI  | 8  | 編集・削除 |
| ...                                                |
+-------------------------------------------------------+
```

### 8.6. Company Detail / 学生マッチ一覧
```
+-------------------------------------------------------+
|   企業詳細   (株式会社A)                               |
+-------------------------------------------------------+
| 会社名: 株式会社A    | 業種: Web                    |
| 必要スキル: WebFrontEnd, UIUX                      |
| 勤務地: 東京都渋谷区                                 |
+-------------------------------------------------------+
| 【マッチング学生】                                    |
| ---------------------------------------------------- |
| | 学生名 | スコア | 生年月日 | 性別 | アクション |
| | 山田太郎 | 100 | 2005‑04‑15 | 男 | 詳細 |
| | 中村優子 | 80  | 2005‑03‑18 | 女 | 詳細 |
+-------------------------------------------------------+
```

### 8.7. CSV インポート画面
```
+-------------------------------------------------------+
|  【CSV ファイル選択】  (ファイル選択ボタン)            |
|  例: students.csv                                     |
+-------------------------------------------------------+
|  [インポート開始]  (ボタン)                          |
+-------------------------------------------------------+
|  進捗バー: 0%/100% (インポート中)                     |
|  成功: 60件、失敗: 0件                               |
+-------------------------------------------------------+
```

### 8.8. 認証画面 (Login)
```
+-------------------------------------------------------+
|  ログイン                                              |
|  Email: ___________________________                  |
|  Password: _______________________                  |
|  [ログイン]  (ボタン)                                 |
|  Forgot password? (リンク)                           |
+-------------------------------------------------------+
```

---

## 9. User Stories & Acceptance Criteria

| # | User Story | Acceptance Criteria |
|---|------------|---------------------|
| **US‑1** | As an **admin**, I want to upload a CSV of students so that I can bulk‑register them. | - CSV format validation error displayed if missing columns.<br>- After upload, UI shows total rows, successful inserts, duplicates skipped.<br>- Data appears in Student List instantly. |
| **US‑2** | As an **admin**, I want to add/edit/delete a company so that the matching database stays up‑to‑date. | - CRUD UI with validation (name required, at least one skill).<br>- Delete shows confirmation modal.<br>- Changes reflected in Match results without manual refresh. |
| **US‑3** | As an **admin**, I want to view a student's matched companies sorted by relevance. | - Student Detail page shows a table with company name, required skills, match score, and status.<br>- Score is computed as per FR‑004. |
| **US‑4** | As a **viewer**, I want to search for a student by name and filter by skill. | - Search bar returns matches instantly (client‑side debounced).<br>- Filter dropdown adjusts the list without full page reload. |
| **US‑5** | As an **admin**, I want to re‑run the matching algorithm after editing a company’s required skills. | - “Re‑calculate matching” button triggers API call, shows progress spinner, updates Match table.<br>- Updated scores appear in both student and company view pages. |
| **US‑6** | As a **user**, I want the app to be secure, so my data isn’t exposed. | - All API endpoints validate session token.<br>- Passwords stored with bcrypt.<br>- No XSS / CSRF vulnerabilities detected by OWASP scan. |
| **US‑7** | As a **project manager**, I want the system to be deployed automatically on each PR. | - GitHub Action runs lint, tests, builds, then pushes to Vercel preview URL.<br>- Deploy status appears in PR comments. |
| **US‑8** | As an **admin**, I want to export the match results to CSV for reporting. | - “Export CSV” button downloads a file with columns: Student, Company, Score, MatchedAt. |

---

## 10. Technical Implementation Details

### 10.1. Directory Structure (Next.js App Router)
```
/src
 ├─ /app
 │   ├─ layout.tsx            // 共通レイアウト (Header + Sidebar)
 │   ├─ page.tsx              // Dashboard
 │   ├─ /students
 │   │    ├─ page.tsx         // List
 │   │    ├─ [id]/page.tsx    // Detail + Match list
 │   │    └─ import/page.tsx // CSV upload
 │   ├─ /companies
 │   │    ├─ page.tsx
 │   │    └─ [id]/page.tsx
 │   ├─ /matches
 │   │    └─ page.tsx          // 全マッチング一覧 (admin)
 │   └─ /auth
 │        └─ login/page.tsx
 ├─ /components
 │   ├─ Table.tsx
 │   ├─ SearchBar.tsx
 │   ├─ FilterSelect.tsx
 │   └─ ... (UI)
 ├─ /lib
 │   ├─ prisma.ts            // Prisma client singleton
 │   └─ matchEngine.ts       // Matching algorithm
 ├─ /pages
 │   └─ api
 │        ├─ auth/[...nextauth].ts
 │        ├─ students.ts
 │        ├─ students/[id].ts
 │        ├─ companies.ts
 │        ├─ companies/[id].ts
 │        └─ matches/recalculate.ts
 └─ prisma
      └─ schema.prisma
```

### 10.2. Prisma Schema (excerpt)
```prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

enum Gender {
  MALE
  FEMALE
  OTHER
}

enum Skill {
  WebFrontEnd
  DataScience
  Backend
  UIUX
  MachineLearning
  MobileApp
  CloudInfra
  DatabaseDesign
  Security
  GameDevelopment
  DevOps
  ArtificialIntelligence
}

model Student {
  id          Int      @id @default(autoincrement())
  name        String
  kana        String
  birthdate   DateTime
  gender      Gender
  skill       Skill
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  isDeleted   Boolean  @default(false)

  matches     Match[]
}

model Company {
  id            Int      @id @default(autoincrement())
  name          String
  industry      String
  requiredSkills String   // comma‑separated Skill values
  location      String?
  description   String?
  createdAt     DateTime @default(now())
  updatedAt     DateTime @updatedAt

  matches       Match[]
}

model Match {
  id          Int      @id @default(autoincrement())
  student     Student  @relation(fields: [studentId], references: [id])
  studentId   Int
  company     Company  @relation(fields: [companyId], references: [id])
  companyId   Int
  score       Int
  matchedAt   DateTime @default(now())
}
```

### 10.3. Matching Engine (`lib/matchEngine.ts`)
```ts
import { PrismaClient, Skill } from '@prisma/client';
const prisma = new PrismaClient();

type MatchResult = {
  studentId: number;
  companyId: number;
  score: number;
};

/**
 * Convert comma‑separated skill string to Set<Skill>
 */
function parseSkills(skills: string): Set<Skill> {
  return new Set(
    skills
      .split(',')
      .map(s => s.trim())
      .filter(Boolean) as Skill[]
  );
}

/**
 * Compute score based on rules:
 *  - 100: 完全一致 (student.skill === requiredSkill)
 *  - 80 : 1つ以上の部分一致 (学生のスキルが企業の requiredSkills に含まれる)
 *  - 70 : カテゴリ(大分類) が同一 (例: `MachineLearning` と `ArtificialIntelligence` は同一カテゴリ "AI")
 *  - 0  : 該当なし
 */
function computeScore(studentSkill: Skill, companySkills: Set<Skill>): number {
  // 直接一致
  if (companySkills.has(studentSkill)) return 100;

  // カテゴリマッピング (簡易実装)
  const categoryMap: Record<Skill, string> = {
    WebFrontEnd: 'Web',
    Backend: 'Web',
    UIUX: 'Web',
    GameDevelopment: 'Game',
    MachineLearning: 'AI',
    ArtificialIntelligence: 'AI',
    DataScience: 'AI',
    CloudInfra: 'Cloud',
    DatabaseDesign: 'Database',
    Security: 'Security',
    DevOps: 'Ops',
    MobileApp: 'Mobile',
  };

  const studentCat = categoryMap[studentSkill];
  for (const cs of companySkills) {
    if (categoryMap[cs] === studentCat) {
      // 部分一致 (カテゴリレベル)
      return 70;
    }
  }

  return 0;
}

/**
 * Run full matching for all students + companies.
 * Stores top 3 matches per student into DB.
 */
export async function runMatching(): Promise<void> {
  const students = await prisma.student.findMany({
    where: { isDeleted: false },
  });
  const companies = await prisma.company.findMany();

  // Delete existing matches (optional: keep history)
  await prisma.match.deleteMany({});

  const matches: MatchResult[] = [];

  for (const student of students) {
    const stuSkill = student.skill as Skill;
    const candidateScores: MatchResult[] = [];

    for (const comp of companies) {
      const compSkills = parseSkills(comp.requiredSkills) as Set<Skill>;
      const score = computeScore(stuSkill, compSkills);
      if (score > 0) {
        candidateScores.push({
          studentId: student.id,
          companyId: comp.id,
          score,
        });
      }
    }

    // Sort descending & keep top 3
    candidateScores.sort((a, b) => b.score - a.score);
    const top = candidateScores.slice(0, 3);
    matches.push(...top);
  }

  // Bulk insert
  await prisma.match.createMany({
    data: matches.map(m => ({
      studentId: m.studentId,
      companyId: m.companyId,
      score: m.score,
    })),
    skipDuplicates: true,
  });
}
```

*The engine can be triggered via an API route (`/api/matches/recalculate`) or a Next.js serverless `cron` using Vercel’s **Scheduled Functions** (e.g., nightly).*

### 10.4. API Example (`/pages/api/matches/recalculate.ts`)
```ts
import type { NextApiRequest, NextApiResponse } from 'next';
import { getSession } from 'next-auth/react';
import { runMatching } from '@/lib/matchEngine';

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  // Only POST + admin
  const session = await getSession({ req });
  if (!session?.user?.role?.includes('admin')) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method Not Allowed' });
  }

  try {
    await runMatching();
    return res.status(200).json({ message: 'Matching recalculated successfully' });
  } catch (e) {
    console.error('Matching error:', e);
    return res.status(500).json({ error: 'Internal Server Error' });
  }
}
```

### 10.5. CSV Import (`/pages/api/students/import.ts`)
```ts
import type { NextApiRequest, NextApiResponse } from 'next';
import formidable from 'formidable';
import csvParser from 'csv-parse';
import { PrismaClient, Gender, Skill } from '@prisma/client';
import { getSession } from 'next-auth/react';
import fs from 'fs';
import path from 'path';

export const config = { api: { bodyParser: false } };

const prisma = new PrismaClient();

export default async function handler(req: NextApiRequest, res: NextApiResponse) {
  const session = await getSession({ req });
  if (!session?.user?.role?.includes('admin')) {
    return res.status(403).json({ error: 'Forbidden' });
  }

  if (req.method !== 'POST') {
    return res.status(405).json({ error: 'Method Not Allowed' });
  }

  const form = new formidable.IncomingForm({ multiples: false, maxFileSize: 5 * 1024 * 1024 });

  form.parse(req, async (err, fields, files) => {
    if (err) {
      console.error(err);
      return res.status(400).json({ error: 'File parse error' });
    }

    const file = files.file as formidable.File;
    if (!file) {
      return res.status(400).json({ error: 'No file uploaded' });
    }

    const raw = await fs.promises.readFile(file.filepath, 'utf-8');
    csvParser(
      raw,
      { columns: true, trim: true, skip_empty_lines: true },
      async (parseErr, records) => {
        if (parseErr) {
          console.error(parseErr);
          return res.status(400).json({ error: 'CSV parse error' });
        }

        let success = 0;
        let duplicate = 0;
        let failed = 0;
        const errors: string[] = [];

        for (const rec of records) {
          // Validate required columns
          const { 氏名, フリガナ, 生年月日, 性別, 得意分野 } = rec;
          if (!氏名 || !フリガナ || !生年月日 || !性別 || !得意分野) {
            failed++;
            errors.push(`Missing required fields in row ${JSON.stringify(rec)}`);
            continue;
          }

          // Convert types
          const genderMap: { [k: string]: Gender } = {
            男: 'MALE',
            女: 'FEMALE',
            その他: 'OTHER',
          };
          const skillMap: { [k: string]: Skill } = {
            'Webフロントエンド': 'WebFrontEnd',
            'データサイエンス': 'DataScience',
            'バックエンド開発': 'Backend',
            'UI/UXデザイン': 'UIUX',
            '機械学習': 'MachineLearning',
            'モバイルアプリ開発': 'MobileApp',
            'クラウドインフラ': 'CloudInfra',
            'データベース設計': 'DatabaseDesign',
            'セキュリティ': 'Security',
            'ゲーム開発': 'GameDevelopment',
            'DevOps': 'DevOps',
            '人工知能': 'ArtificialIntelligence',
          };

          const gender = genderMap[性別];
          const skill = skillMap[得意分野];
          if (!gender || !skill) {
            failed++;
            errors.push(`Invalid gender or skill in row ${JSON.stringify(rec)}`);
            continue;
          }

          // Duplicate check (name + birthdate)
          const existing = await prisma.student.findFirst({
            where: {
              name: 氏名,
              birthdate: new Date(生年月日),
              isDeleted: false,
            },
          });

          if (existing) {
            duplicate++;
            continue;
          }

          try {
            await prisma.student.create({
              data: {
                name: 氏名,
                kana: フリガナ,
                birthdate: new Date(生年月日),
                gender,
                skill,
              },
            });
            success++;
          } catch (e) {
            failed++;
            errors.push(`DB error for row ${JSON.stringify(rec)}: ${e}`);
          }
        }

        return res.status(200).json({
          message: 'Import completed',
          result: { success, duplicate, failed, errors },
        });
      }
    );
  });
}
```

> **Note**: The import route uses `formidable` to handle file uploads, parses CSV, validates data, and writes to the `Student` table. After import, a UI notification displays the summary.

### 10.6. Environment Variables (Vercel)
| Variable | Example | Description |
|----------|---------|-------------|
| `DATABASE_URL` | `mysql://user:pass@aws.connect.psdb.cloud/dbname` | PlanetScale MySQL connection string |
| `NEXTAUTH_URL` | `https://your-app.vercel.app` | NextAuth base URL |
| `NEXTAUTH_SECRET` | (auto‑generated) | Cookie/Token encryption |
| `EMAIL_SERVER` | `smtp://user:pass@smtp.mailer.com:587` | SendGrid/SES 等（メール通知オプション） |
| `EMAIL_FROM` | `no-reply@yourdomain.com` | 送信元アドレス |

---

## 11. Test Strategy

| Test Type | Tools | Coverage Target |
|-----------|------|-----------------|
| Unit Tests (business logic) | Jest (ts-jest) | 80 % of functions (`matchEngine`, CSV parser, API utils) |
| Component Tests (React) | React Testing Library + Jest | 80 % of UI components |
| Integration Tests (API) | Supertest + Jest | All API routes (CRUD, import, matching) |
| End‑to‑End Tests | Playwright (Chromium) | Critical user flows: login → import CSV → view matches |
| Lint / Format | ESLint + Prettier (CI) | 0 warnings |
| Security Scan | OWASP ZAP (CI) | No high/critical issues |
| Performance Audit | Lighthouse CI (GitHub Actions) | `Performance >= 90` |

---

## 12. Deployment & CI/CD Flow

1. **Push** → GitHub Action runs:
   - `npm ci`
   - `eslint --fix`
   - `npm run typecheck`
   - `npm test`
   - `npm run build`
   - **If `main` branch** → Vercel auto‑deploy (Production)
   - **If PR** → Vercel preview URL posted as comment

2. **Secrets** in Vercel:
   - Add all environment variables from **10.6**.

3. **Database Migration**:
   - `npx prisma migrate deploy` executed via Vercel “Post‑Deploy” command.

4. **Scheduled Matching** (optional):
   - Vercel “Cron Jobs” → `0 2 * * *` → invoke `/api/matches/recalculate` (POST) with a server‑to‑server token.

5. **Rollback**:
   - Revert commit → Vercel automatically rolls back to previous build.

---

## 13. Milestones & Timeline (示例)

| Milestone | Deliverable | Duration | Owner |
|-----------|-------------|----------|-------|
| **M1** | 要件定義・設計承認 (basic_design.md 完成) | 1 wk | PM, Architect |
| **M2** | 環境構築・CIセットアップ | 1 wk | DevOps |
| **M3** | データモデル実装 & Prisma 設定 | 3 days | Backend |
| **M4** | CSV インポート機能実装 | 4 days | Backend |
| **M5** | 学生・企業 CRUD UI 実装 | 5 days | Frontend |
| **M6** | マッチングエンジン実装 & API | 5 days | Full‑stack |
| **M7** | 認証・権限 (NextAuth) | 3 days | Full‑stack |
| **M8** | UI/UX 完成・ワイヤーフレーム実装 (Tailwind) | 4 days | Frontend |
| **M9** | テスト作成 (ユニット/統合/E2E) | 5 days | QA |
| **M10** | パフォーマンス・セキュリティチューニング | 3 days | DevOps/QA |
| **M11** | 本番デプロイ & ドキュメント納品 | 2 days | All |
| **M12** | 保守フェーズ開始 (3 か月) | - | Support |

*総工数 ≈ 120 h（約 6 週間）*

---

## 14. Risk & Mitigation

| # | Risk | Impact | Probability | Mitigation |
|---|------|--------|-------------|------------|
| R1 | CSV フォーマット不一致（項目名が変わる） | データインポート失敗 | 中 | 事前サンプルテンプレート提供、インポート時バリデーションでエラーメッセージを明示 |
| R2 | マッチングロジックのスコアが期待通りに出ない | ユーザー不満 | 低 | 初期段階でステークホルダーとスコアリング基準を合意、パラメータ化して調整可能に |
| R3 | Vercel のサーバーレス関数がタイムアウト（大量データ） | バッチ処理失敗 | 中 | 用量が増えたらバックグラウンドジョブ（PlanetScale の外部ワーカー）へ切り替え、または Cron 実行を分割 |
| R4 | 認証情報漏洩 | セキュリティ重大 | 低 | 環境変数の暗号化、NextAuth の CSRF / SameSite 設定、アクセスログ監視 |
| R5 | データベーススキーマ変更によるマイグレーション失敗 | サービス停止 | 低 | Prisma のマイグレーションテスト、ステージング環境で事前検証 |
| R6 | Vercel の無料プランリソース制限 (ビルド時間・帯域) | パフォーマンス低下 | 中 | 初期は有料プランで確保、負荷が上がったらオートスケールを検討 |

---

## 15. Acceptance & Sign‑off

| 項目 | 判定基準 | 署名 |
|------|----------|------|
| 要件定義書 (この basic_design.md) | 全ステークホルダーが承認 (`✔`) |  |
| デザイン・ワイヤーフレーム | UI がモックに一致し、アクセシビリティテスト合格 |  |
| コーディング完了 | CI が全て成功、コードカバレッジ ≥ 80 % |  |
| E2E テストパス | 主要 5 つシナリオが Pass |  |
| パフォーマンス | Lighthouse `Performance ≥ 90` |  |
| セキュリティ | OWASP ZAP **High/Critical** なし |  |
| 本番デプロイ | Vercel Production URL 正常稼働 |  |
| ドキュメント | - システム構成図 <br>- API 仕様書 <br>- 運用マニュアル |  |

*全項目が合格した時点で、プロジェクトは「完了」状態とみなす。*

---

## 16. Appendices

### A. Sample CSV Header (UTF‑8)
```csv
氏名,フリガナ,生年月日,性別,得意分野
山田太郎,ヤマダタロウ,2005-04-15,男,Webフロントエンド
...
```

### B. Skill Mapping Table (used in `matchEngine.ts`)
| 日本語（得意分野） | Enum (`Skill`) |
|-------------------|----------------|
| Webフロントエンド | `WebFrontEnd` |
| データサイエンス | `DataScience` |
| バックエンド開発 | `Backend` |
| UI/UXデザイン | `UIUX` |
| 機械学習 | `MachineLearning` |
| モバイルアプリ開発 | `MobileApp` |
| クラウドインフラ | `CloudInfra` |
| データベース設計 | `DatabaseDesign` |
| セキュリティ | `Security` |
| ゲーム開発 | `GameDevelopment` |
| DevOps | `DevOps` |
| 人工知能 | `ArtificialIntelligence` |

### C. Company RequiredSkills Format
```
WebFrontEnd,UIUX
DataScience,ArtificialIntelligence
Backend,DatabaseDesign,DevOps
```

---  

**Prepared by:**  
*Solution Architect – Next.js / Vercel Specialist*  

**Date:** 2025‑08‑07  

---  

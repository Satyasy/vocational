# Vocational Monorepo

Repository ini menggunakan arsitektur **Monorepo** berbasis **pnpm Workspaces** yang mengintegrasikan frontend (**Astro**), backend (**NestJS**), dan shared contract/types library (**@repo/shared**) dalam satu repositori terpadu.

---

## ❖ Struktur Direktori

```text
vocational/
├── apps/
│   ├── web/               # Frontend (Astro v7 + Content Collections / SSR / Static)
│   │   └── .env.example
│   └── api/               # Backend (NestJS v12 + Express / Vector Search)
│       └── .env.example
├── packages/
│   └── shared/            # Single Source of Truth: DTO, Interfaces, & Types (@repo/shared)
│       ├── package.json
│       └── index.ts
├── package.json           # Root workspace configuration & global scripts
├── pnpm-workspace.yaml    # Konfigurasi workspace pnpm
├── AGENTS.md              # Standar instruksi & konteks kerja bagi AI Coding Agents
└── README.md
```

---

## ⚙ Prasyarat Sistem

* **Node.js**: Versi `20.x` atau `22.x LTS` (disarankan Node >= 22.x)
* **pnpm**: Versi `10.x` atau `12.x` (`devEngines` dikunci pada `pnpm@12.15.0`)

> ⚠ **Penting:** Semua instalasi dependensi dan eksekusi skrip wajib menggunakan `pnpm`. Hindari penggunaan `npm` atau `yarn`.

---

## ➔ Setup & Instalasi Pertama Kali

Jalankan seluruh perintah ini dari **root direktori**:

```bash
# 1. Pasang dependensi untuk seluruh apps & packages
pnpm install

# 2. Jika muncul verifikasi script build bawaan pnpm (lifecycle scripts):
pnpm approve-builds

# 3. Setup Environment Variables
cp apps/api/.env.example apps/api/.env
cp apps/web/.env.example apps/web/.env
```

> ✖ **Aturan Monorepo:** Jangan pernah menjalankan `npm install` atau `pnpm install` di dalam subfolder `apps/web` atau `apps/api`. Selalu kelola dependensi dari root.

---

## 💻 Panduan Menjalankan Project (Development)

Tersedia skrip serentak maupun terpisah langsung dari root direktori:

### Menjalankan Bersamaan (Recommended)
```bash
pnpm dev
```
> Perintah ini menggunakan `concurrently` untuk menyalakan API (`localhost:3000`) dan Web (`localhost:4321`) dalam 1 terminal.

### Menjalankan Per Service

| Target | Root Script | Atau via Filter | URL Default |
| :--- | :--- | :--- | :--- |
| **Fullstack (API + Web)** | `pnpm dev` | - | - |
| **Backend API (NestJS)** | `pnpm dev:api` | `pnpm --filter api start:dev` | `http://localhost:3000` |
| **Frontend Web (Astro)** | `pnpm dev:web` | `pnpm --filter web dev` | `http://localhost:4321` |

---

## ◆ Panduan Shared Package (`@repo/shared`)

Package `@repo/shared` merupakan jembatan kontrak antara Frontend dan Backend agar tidak terjadi redundansi definisi interface/tipe data.

### 1. Menambahkan Kontrak Baru (`packages/shared/index.ts`)
```typescript
// packages/shared/index.ts
export interface VectorSearchRequest {
  query: string;
  topK: number;
}

export interface SearchResultItem {
  id: string;
  score: number;
  content: string;
}
```

### 2. Panduan untuk Backend Developer (NestJS)
Gunakan interface dari `@repo/shared` untuk memastikan DTO atau return type service mematuhi kontrak:
```typescript
// apps/api/src/search/search.dto.ts
import { VectorSearchRequest } from '@repo/shared';

export class VectorSearchDto implements VectorSearchRequest {
  query: string;
  topK: number;
}
```

**Konfigurasi CORS (Penting):**
Pastikan NestJS mengizinkan request dari frontend (`http://localhost:4321`):
```typescript
// apps/api/src/main.ts
app.enableCors({
  origin: process.env.FRONTEND_URL || 'http://localhost:4321',
  credentials: true,
});
```

### 3. Panduan untuk Frontend Developer (Astro)
Konsumsi tipe data yang sama saat melakukan fetch ke backend:
```astro
---
// apps/web/src/pages/search.astro
import type { SearchResultItem } from '@repo/shared';

const apiUrl = import.meta.env.PUBLIC_API_URL || 'http://localhost:3000';
const response = await fetch(`${apiUrl}/search`);
const data: SearchResultItem[] = await response.json();
---
```

---

## ✦ Panduan Menambah Dependensi

Gunakan opsi `--filter` dari root direktori:

```bash
# Menambah library ke Frontend (Astro)
pnpm --filter web add lucide-react

# Menambah library ke Backend (NestJS)
pnpm --filter api add onnxruntime-node

# Menambah dependensi dev ke Backend
pnpm --filter api add -D @types/node

# Menambah dependensi ke shared package
pnpm --filter @repo/shared add zod
```

---

## 🛠 Build & Verifikasi (Production)

```bash
# Build seluruh workspace (API & Web)
pnpm build

# Atau build masing-masing service
pnpm build:api
pnpm build:web

# Menjalankan linter
pnpm lint
```


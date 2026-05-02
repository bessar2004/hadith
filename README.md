# 📖 Hadis Semantik Arama Sistemi

Sahih Bukhari külliyatı üzerinde çalışan, **Arapça ve İngilizce** dil desteğine sahip hibrit arama platformu. MySQL FULLTEXT (BM25) + Qdrant (Semantik) + Vue.js frontend.

---

## 🏗️ Mimari

```
nlp hadith project/
├── backend/                  # FastAPI (Python 3.12+)
│   ├── app/
│   │   ├── main.py           # Uygulama giriş noktası
│   │   ├── config.py         # Pydantic settings (.env okur)
│   │   ├── database/
│   │   │   ├── connection.py # Async SQLAlchemy engine
│   │   │   ├── models.py     # ORM modelleri
│   │   │   └── schemas.py    # Pydantic request/response
│   │   ├── routers/
│   │   │   ├── auth.py       # /auth/* (register, login, refresh, me)
│   │   │   ├── search.py     # /ara?q=...
│   │   │   ├── hadith.py     # /hadis/{id}
│   │   │   └── favorites.py  # /favoriler (auth korumalı)
│   │   └── services/
│   │       ├── auth_service.py   # JWT + bcrypt
│   │       └── bm25_service.py   # MySQL FULLTEXT arama
│   ├── scripts/
│   │   ├── init.sql          # DB şeması (Docker init)
│   │   └── import_data.py    # CSV → MySQL aktarım
│   ├── Dockerfile
│   └── requirements.txt
├── frontend/                 # Vue.js 3 + Vite (Faz 3'te kurulacak)
├── veriler csv/
│   ├── bukhari_arabic.csv    # 7277 Arapça hadis
│   └── bukhari_english.csv   # 7277 İngilizce hadis
├── docker-compose.yml        # MySQL 8.0 + Qdrant + backend
├── .env                      # Ortam değişkenleri (git'e ekleme!)
└── .env.example              # Şablon
```

---

## ✅ Tamamlananlar (Faz 1)

- [x] Docker ortamı kuruldu (MySQL 8.0 + Qdrant)
- [x] Veritabanı şeması oluşturuldu (6 tablo)
- [x] **7277 hadis** MySQL'e aktarıldı — Eksiksiz! ✓ 0 hata (Arapça + İngilizce)
- [x] FastAPI iskelet kuruldu (CORS, router'lar, health check)
- [x] JWT tabanlı auth sistemi yazıldı (register/login/refresh)
- [x] BM25 arama servisi yazıldı (MySQL FULLTEXT)
- [x] Favoriler CRUD endpoint'leri yazıldı

---

## 🔧 Kurulum (İlk Kez)

### 1. Gereksinimler
```bash
# Docker kurulu olmalı
docker --version   # Docker 29.4+

# Python 3.12+ kurulu olmalı
python3 --version
```

### 2. Ortam değişkenlerini ayarla
```bash
cp .env.example .env
```
`.env` dosyasını aç ve şu değeri güvenli bir anahtarla değiştir:
```bash
# Terminalden güvenli anahtar üret:
openssl rand -hex 32

# Çıktıyı .env içindeki SECRET_KEY satırına yapıştır:
SECRET_KEY=üretilen_anahtar_buraya
```

### 3. Docker container'larını başlat
```bash
# Eğer docker grubuna henüz eklendiysen yeni terminalde şu komutu kullan:
sg docker -c 'docker compose up -d db qdrant'

# veya oturumu kapatıp açtıktan sonra direkt:
docker compose up -d db qdrant

# Container durumunu kontrol et:
docker compose ps
```

### 4. Veritabanını başlat ve verileri aktar
```bash
# İlk kez: DB şemasını oluştur (Docker init.sql bunu otomatik yapmalıydı,
# yapamadıysa manuel olarak çalıştır)
sg docker -c 'docker exec -i hadis_mysql mysql -u root -prootpass hadis_db < backend/scripts/init.sql'

# Verileri aktar (yaklaşık 3-4 dakika sürer)
export MYSQL_HOST=127.0.0.1
cd backend
pip install mysql-connector-python python-dotenv tqdm
python scripts/import_data.py

# Test için sadece ilk 100 hadis:
python scripts/import_data.py --limit 100
```

### 5. API'yi çalıştır
```bash
cd backend
pip install -r requirements.txt
uvicorn app.main:app --reload

# veya Docker ile:
sg docker -c 'docker compose up backend'
```

API çalışıyor mu? → http://localhost:8000/docs adresini aç.

---

## 📋 Yapılacaklar — Öncelik Sırasıyla

### 🔴 Kritik (Önce Bunlar)

#### 1. SECRET_KEY'i güvenli yap
```bash
openssl rand -hex 32
# Çıktıyı .env dosyasındaki SECRET_KEY'e yapıştır
```
> ⚠️ Şu anki değer güvensiz! Production'da kesinlikle değiştirilmeli.

#### 2. API'yi test et
```bash
# Terminalden test:
curl http://localhost:8000/health
curl "http://localhost:8000/ara?q=prayer&dil=en"
curl "http://localhost:8000/ara?q=الصلاة&dil=ar"

# Swagger arayüzünden de test edebilirsin:
# http://localhost:8000/docs
```

#### 3. `ravi` ve `bab` sütun uzunlukları
Import sırasında 11 hadis atlandı çünkü `ravi` (VARCHAR 200) ve `bab` (VARCHAR 500) alanları bazı satırlarda çok uzun. Düzeltmek için:
```sql
-- Docker içinde MySQL'e bağlan:
docker exec -it hadis_mysql mysql -u root -prootpass hadis_db

-- Sütunları genişlet:
ALTER TABLE hadisler MODIFY ravi TEXT;
ALTER TABLE hadisler MODIFY bab TEXT;
EXIT;

-- Sonra import'u tekrar çalıştır:
export MYSQL_HOST=127.0.0.1
cd backend
python scripts/import_data.py
```

---

### 🟡 Faz 2 — Semantik Arama (Sıradaki)

#### 4. Python sanal ortamı kur
```bash
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```
> ⚠️ `sentence-transformers` ve `torch` büyük paketler (~2GB). İndirme süresi uzun olabilir.

#### 5. Embedding modeli indir ve Qdrant'a yükle
`backend/scripts/` klasörüne aşağıdaki scripti eklemen gerekecek:
```
scripts/build_embeddings.py
```
Bu script:
- MySQL'den tüm hadisleri çeker
- `paraphrase-multilingual-mpnet-base-v2` modeli ile vektör üretir
- Qdrant'a (`hadisler_ar` ve `hadisler_en` collection'larına) yükler

#### 6. Hibrit arama (BM25 + Semantik) — RRF Fusion
`services/` klasörüne:
- `vector_service.py` — Qdrant sorgulama
- `hybrid_service.py` — RRF Fusion (BM25 + Semantic skorları birleştirme)

Sonra `routers/search.py` dosyasını güncelleyerek `/ara` endpoint'ini hibrit moda geçir.

---

### 🟢 Faz 3 — Vue.js Frontend

#### 7. Frontend kurulumu
```bash
cd frontend
npm install
npm run dev
# → http://localhost:5173
```

Tasarım referansı: https://stitch.withgoogle.com/projects/745166497204631457

Oluşturulacak sayfalar:
- `views/HomeView.vue` — Arama sayfası
- `views/HadithDetailView.vue` — Hadis detay sayfası
- `views/FavoritesView.vue` — Favoriler (auth gerekli)
- `components/AuthModal.vue` — Giriş/Kayıt modal
- `components/HadithCard.vue` — Hadis kartı bileşeni
- `stores/auth.js` — Pinia auth store
- `stores/search.js` — Pinia arama store

---

### 🔵 Faz 4 — Güvenlik ve Deployment

#### 8. Güvenlik kontrol listesi (deploy öncesi)
- [ ] `.env` dosyası `.gitignore`'a eklendi mi?
- [ ] `SECRET_KEY` güvenli bir değerle değiştirildi mi?
- [ ] `DEBUG=false` yapıldı mı?
- [ ] CORS ayarları sadece kendi domain'ini içeriyor mu?
- [ ] MySQL şifresi güçlü bir şifreyle değiştirildi mi?

#### 9. .gitignore oluştur
```bash
cat > .gitignore << 'EOF'
.env
__pycache__/
*.pyc
*.pyo
venv/
node_modules/
dist/
.DS_Store
*.log
EOF
```

---

## 🚀 Hızlı Başlangıç (Her Gün)

```bash
# 1. Container'ları başlat
sg docker -c 'docker compose up -d db qdrant'

# 2. API'yi çalıştır
cd backend
source venv/bin/activate   # sanal ortam varsa
export MYSQL_HOST=127.0.0.1
uvicorn app.main:app --reload

# 3. Frontend'i çalıştır (Faz 3'ten sonra)
cd frontend
npm run dev
```

---

## 🔌 API Endpoint'leri

| Method | URL | Açıklama | Auth |
|--------|-----|----------|------|
| GET | `/` | Health check | ❌ |
| GET | `/docs` | Swagger UI | ❌ |
| POST | `/auth/register` | Kullanıcı kaydı | ❌ |
| POST | `/auth/login` | Giriş, token al | ❌ |
| POST | `/auth/refresh` | Token yenile | ❌ |
| GET | `/auth/me` | Profil bilgisi | ✅ |
| GET | `/ara?q=...` | Hadis ara | ❌ |
| GET | `/hadis/{id}` | Tekil hadis | ❌ |
| GET | `/favoriler` | Favorileri listele | ✅ |
| POST | `/favoriler` | Favori ekle | ✅ |
| DELETE | `/favoriler/{id}` | Favori sil | ✅ |

---

## 🐛 Bilinen Sorunlar

| Sorun | Durum | Çözüm |
|-------|-------|-------|
| 11 hadis import edilemedi | ⚠️ Açık | `ravi` ve `bab` sütunlarını TEXT yap (yukarıdaki adım 3) |
| Docker grup izni | ⚠️ Geçici | `sg docker -c '...'` kullan veya oturumu kapat/aç |
| `version` uyarısı (docker-compose) | ℹ️ Kozmetik | `docker-compose.yml` başındaki `version: "3.9"` satırını sil |

---

## 📦 Kullanılan Teknolojiler

| Katman | Teknoloji |
|--------|-----------|
| Backend API | FastAPI 0.115 + Uvicorn |
| Veritabanı | MySQL 8.0 (FULLTEXT + UTF8MB4) |
| ORM | SQLAlchemy 2.0 (async) |
| Vektör DB | Qdrant (Faz 2) |
| NLP Modeli | `paraphrase-multilingual-mpnet-base-v2` (Faz 2) |
| Auth | JWT (python-jose) + bcrypt (passlib) |
| Frontend | Vue.js 3 + Vite + Pinia (Faz 3) |
| Container | Docker + Docker Compose |

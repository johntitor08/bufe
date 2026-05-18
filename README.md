# Kılıçbüfe

Cloudflare Workers üzerinde çalışan, Firebase Realtime Database destekli büfe sipariş ve yönetim sistemi.

## Dosya Yapısı

```
kilicbufe-main/
├── index.html          # Müşteri arayüzü (ürünler, sepet, sipariş)
├── wrangler.jsonc      # Cloudflare Workers yapılandırması
└── admin/
    ├── login.html      # Admin girişi (Firebase Auth)
    └── dashboard.html  # Admin paneli (siparişler, ürünler, QR)
```

## Kurulum

### 1. Firebase

1. [Firebase Console](https://console.firebase.google.com)'dan yeni proje oluştur
2. **Realtime Database** → Europe-West bölgesi → Test modunda başlat
3. **Authentication** → Email/Password provider'ı etkinleştir → bir admin kullanıcısı oluştur
4. Proje ayarlarından config değerlerini kopyala

### 2. Firebase Config

`index.html` ve `admin/dashboard.html` içindeki `firebaseConfig` bloğunu kendi değerlerinle doldur:

```js
const firebaseConfig = {
  apiKey:            "...",
  authDomain:        "PROJE_ID.firebaseapp.com",
  databaseURL:       "https://PROJE_ID-default-rtdb.europe-west1.firebasedatabase.app",
  projectId:         "PROJE_ID",
  storageBucket:     "PROJE_ID.firebasestorage.app",
  messagingSenderId: "...",
  appId:             "..."
};
```

> ⚠️ Bu dosyaları `.gitignore`'a ekle veya config'i ayrı bir `firebase-config.js`'e taşıyıp git dışında tut.

### 3. Cloudflare Deploy

```bash
npm install -g wrangler
wrangler login
wrangler deploy
```

Deploy sonrası aldığın URL'yi `admin/dashboard.html` → QR Kod sekmesindeki Site URL alanına gir.

## Admin Paneline Erişim

`https://your-domain.workers.dev/admin/dashboard.html` adresine Firebase Auth ile oluşturduğun e-posta/şifre ile giriş yapılır.

**Güvenlik için önerilen:** Cloudflare Zero Trust → Access üzerinden `/admin/*` path'ini sadece belirli e-postalara kısıtla (ücretsiz planda çalışır, kod değişikliği gerekmez).

## Özellikler

**Müşteri (index.html)**
- Firebase'den canlı ürün listesi
- Kategori bazlı menü görünümü
- Sepet (localStorage ile kalıcı)
- Masa numarası seçimi ile sipariş gönderme

**Admin (dashboard.html)**
- Gerçek zamanlı sipariş takibi (masa numarası gösterimi dahil)
- Sipariş teslim et → Firebase'den sil
- Ürün ekle / fiyat düzenle / fotoğraf URL güncelle / sil
- QR kod üretici (özelleştirilebilir renk ve boyut)

## Firebase Realtime Database Kuralları

Ürünler herkese okunabilir, siparişler herkese yazılabilir; okuma ve silme sadece auth kullanıcısına açık olacak şekilde önerilen kural seti:

```json
{
  "rules": {
    "products": {
      ".read": true,
      ".write": "auth != null"
    },
    "orders": {
      ".read": "auth != null",
      ".write": true
    }
  }
}
```

# Yangin_Tespit

# 🔥 Yangın & Duman Tespit Sistemi

Gerçek zamanlı **YOLO** tabanlı yapay zeka sistemi ile IP kameraların video akışında yangın ve duman tespiti yapan Python uygulaması.

---

## 📋 Proje Nedir?

Sistem **RTSP kameralardan** gelen video akışını analiz ederek:
- ✅ Yangın ve duman otomatik olarak tespit eder
- ✅ Tespit anında **email** ile uyarı gönderir
- ✅ **Snapshot** (görüntü) kaydeder
- ✅ **Web arayüzü** üzerinden canlı izleme sağlar
- ✅ **Windows servisi** olarak arka planda çalışabilir

---

## 🏗️ Sistem Mimarisi

```
┌─────────────────────────────────────────────────────────────┐
│                     PROJE YAPISI                            │
└─────────────────────────────────────────────────────────────┘

IP KAMERALAR (RTSP)
      ↓
┌──────────────────────────────────────────────────────────────┐
│  detector.py (Tespit Motoru)                                 │
│  ├─ Kamera stream akışını al                                │
│  ├─ YOLO modelini çalıştır (her 3 frame'de 1)              │
│  ├─ Yangın/Duman tespiti yap                                │
│  ├─ Email gönder (tespit durumunda)                         │
│  └─ Frame'i JPEG formatına çevir                            │
└──────────────────────────────────────────────────────────────┘
      ↓
┌──────────────────────────────────────────────────────────────┐
│  app.py (Flask Web Uygulaması)                              │
│  ├─ API endpoints (video stream, status, kontrol)           │
│  ├─ Canlı video stream'i tarayıcıya gönder                  │
│  └─ Snapshot ve istatistik yönetimi                         │
└──────────────────────────────────────────────────────────────┘
      ↓
┌──────────────────────────────────────────────────────────────┐
│  index.html (Web Arayüzü)                                    │
│  ├─ Canlı video izleme                                      │
│  ├─ Kamera kontrolleri (başlat/durdur)                      │
│  ├─ Confidence threshold ayarı                              │
│  └─ Son tespit görüntüleri                                  │
└──────────────────────────────────────────────────────────────┘
```

---

## 📁 Dosya & Klasör Yapısı

```
fire-smoke-detection/
│
├── train.py                  # YOLO modelini eğitmek için
│
├── detector.py              # Tespit altyapısı (kamera işleme, AI modeli, email)
│
├── app.py                   # Flask API (web sunucusu)
│
├── index.html               # Web arayüzü (HTML/CSS/JS)
│
├── service_fire_smoke.py    # Windows NSSM servisi başlatıcısı
│
├── .env                     # Konfigürasyon (RTSP adresi, email, thresholds)
│
├── models/
│   └── fire_smoke_yolov8.pt # Eğitilmiş YOLO modeli
│
├── dataset/                 # Eğitim veri seti (eğitim sırasında oluşturulur)
│   ├── images/
│   │   ├── train/
│   │   └── val/
│   └── labels/
│       ├── train/
│       └── val/
│
└── snapshots/               # Tespit edilen görüntüler (cam1, cam2 klasörlerine bölünür)
    ├── cam1/
    └── cam2/
```

---

## 🚀 Hızlı Başlangıç

### 1. **Kurulum**

```bash
# Gerekli kütüphaneleri yükle
pip install flask opencv-python ultralytics torch numpy python-dotenv

# .env dosyası oluştur (RTSP adresi ve email ayarı)
cp .env.example .env
```

### 2. **Sistemi Çalıştır**

**Seçenek A: Doğrudan Python ile**
```bash
python app.py
# Tarayıcı: http://localhost:5050
```

**Seçenek B: Windows Servisi ile (NSSM kullanarak)**
```bash
python service_fire_smoke.py
# Tarayıcı: http://yapayzeka:8505
```

---

## 🔧 Ana Dosyaların Görevleri

### **train.py** - Model Eğitim
- Veri seti yapısını hazırlar
- YOLO modelini eğitir
- Modeli `models/fire_smoke_yolov8.pt` konumuna kaydeder
- Modelinin performansını test eder

### **detector.py** - Tespit Motoru ⚙️
- **Kamera bağlantısı**: RTSP akışından frame alır
- **AI işlemi**: Her 3. frame'i YOLO modelinden geçirir
- **Tespit**: Yangın/Duman kutularını bulur ve çizer
- **Email**: Tespit anında email gönderir
- **Snapshot**: Tespit edilen frame'i kaydeder
- **Thread yönetimi**: Her kamera için ayrı thread

### **app.py** - Web Uygulaması
- `/video_feed/<cam_id>` → Canlı video stream
- `/api/status/<cam_id>` → Kamera durumu (JSON)
- `/api/start/<cam_id>` → Kamerayı başlat
- `/api/stop/<cam_id>` → Kamerayı durdur
- `/api/conf/<cam_id>` → Confidence eşiğini değiştir
- `/api/snapshot/<cam_id>` → Manual snapshot al
- `/api/snapshots/<cam_id>` → Son görüntüleri listele

### **index.html** - Web Arayüzü
- Kameralardan gelen canlı video
- Kamera kontrolü (başlat/durdur)
- Confidence slider (0.10 - 0.95)
- Son tespit görüntülerinin galerisi
- Canlı istatistik (FPS, tespit sayısı)

### **service_fire_smoke.py** - Windows Servisi
- Flask uygulamasını başlatır
- Port 8505 dinler
- Log kaydı tutar
- NSSM tarafından çalıştırılmak üzere tasarlandı

---

## 📊 Veri Akışı

```
Frame Geldi (RTSP)
      ↓
   [Her 3. frame'i işle]
      ↓
YOLO Tahmini Yap
      ↓
Yangın/Duman Bulundu mu?
      ├─ EVET → Email Gönder + Snapshot Kaydet + 30sn Bekleme
      └─ HAYIR → İleri Git
      ↓
JPEG Encode Et
      ↓
Web'e Stream Et
      ↓
İstatistik Güncelle
```

---

## 🔐 Konfigürasyon Detayları

| Parametre | Varsayılan | Açıklama |
|-----------|-----------|----------|
| `CONF_THRESHOLD` | 0.45 | YOLO güven eşiği (düşük = daha duyarlı) |
| `IOU_THRESHOLD` | 0.45 | NMS kutu çakışma eşiği |
| `ALARM_COOLDOWN` | 30 | Email gönderim aralığı (saniye) |
| `FRAME_SKIP` | 3 | Her kaçıncı frame'i işle (hız için) |
| `DISPLAY_WIDTH` | 1280 | İşlenmiş frame genişliği |
| `DISPLAY_HEIGHT` | 720 | İşlenmiş frame yüksekliği |
| `SAVE_SNAPSHOTS` | true | Tespit snapshot'ı kaydet |

---

## 📧 Email Konfigürasyonu

**Gmail Uygulama Şifresi Nasıl Oluşturulur:**

1. Google Hesabında 2FA aktif et
2. https://myaccount.google.com/apppasswords adresine git
3. "Mail" ve "Windows Bilgisayar" seç
4. Oluşturulan 16 karakterlik şifreyi `.env` dosyasına yapıştır

---

## 💻 Teknik Gereksinimler

- **Python 3.8+**
- **GPU (İsteğe bağlı)**: CUDA desteğine sahip NVIDIA kartı
- **RAM**: Minimum 4GB (önerilen 8GB)
- **Disk**: Model + veritabanı için 5GB

---

## 🐛 Troubleshooting

**Kamera bağlantısı başarısız:**
- RTSP adresini ve kullanıcı/şifreyi kontrol et
- `rtsp://` protokolünü kullan, `http://` değil

**Email gönderilmiyor:**
- Gmail uygulama şifresini `.env`'de kontrol et
- Internet bağlantısını doğrula

**Yavaş performans:**
- `FRAME_SKIP` değerini artır (örn: 5)
- `DISPLAY_WIDTH` ve `DISPLAY_HEIGHT`'ı küçült

---

---

## 👨‍💼 Maintainer

- Aleyna Kapusuz
- Hande Bandırmalı

---

# 🔥 Yangın & Duman Tespit Sistemi
**Real-time Fire & Smoke Detection System | YOLOv8 + Flask**

Bu sistem, YOLOv8 derin öğrenme modeli kullanarak RTSP kamera akışlarında **yangın ve duman tespiti** yapan, web arayüzüyle kontrol edilebilen bir sistemdir. Tehdit algılanırsa otomatik olarak e-mail bildirimi gönderir ve snapshot kaydeder.

---

## 🛠️ Teknolojiler

| Teknoloji | Kullanım |
|-----------|---------|
| **Flask** | Web backend ve API sunucusu |
| **OpenCV (cv2)** | Video işleme ve frame manipülasyonu |
| **YOLO (ultralytics)** | Yangın/duman tespit modeli |
| **PyTorch** | Model çıkarımı |
| **Threading** | Çok kameralı paralel işleme |
| **SMTP/Gmail** | Uyarı bildirimleri |
| **HTML/CSS/JS** | Etkileşimli web arayüzü |

---

## 🌐 Web Arayüzü

```
http://yapayzeka:8505
```

> **Lokal test**: `http://localhost:5050` (Flask debug modu)

---

## 📁 Proje Yapısı

```
fire-smoke-detection/
│
├── app.py                      ← Flask sunucusu ve API
├── detector.py                 ← YOLO detektörü çekirdeği
├── train.py                    ← Model eğitim scripti
├── service_fire_smoke.py       ← NSSM Windows servisi
│
├── templates/
│   └── index.html              ← Web arayüzü (tab tabanlı, gerçek zamanlı polling)
│
├── models/
│   └── fire_smoke_yolov8.pt    ← Eğitilmiş YOLOv8 modeli
│
├── snapshots/
│   ├── cam1/                   ← Kamera 1 tespit snapshot'ları
│   └── cam2/                   ← Kamera 2 tespit snapshot'ları
│
├── .env                        ← Ayarlar (kamera URL'leri, Gmail, eşikler)
├── requirements.txt            ← Python bağımlılıkları
└── detection.log               ← Dedektör log dosyası
```

---

## 📋 Dosyalar ve Görevleri

### **1. `app.py`** — Flask Web Server
- REST API endpoints sağlar (`/api/status`, `/api/start`, `/api/stop` vb.)
- Video stream'i MJPEG formatında tarayıcıya yayınlar (`/video_feed/<cam_id>`)
- Snapshot yönetimi ve indirme işlemleri
- Uygulama açılırken otomatik olarak yapılandırılmış kameraları başlatır

**Ana Endpoints:**
- `GET /video_feed/<cam_id>` — Canlı video akışı
- `POST /api/start/<cam_id>` — Dedektörü başlat
- `POST /api/stop/<cam_id>` — Dedektörü durdur
- `POST /api/conf/<cam_id>` — Güven eşiğini ayarla
- `POST /api/snapshot/<cam_id>` — Manuel snapshot al
- `GET /api/snapshots/<cam_id>` — Son snapshot'ları listele

---

### **2. `detector.py`** — Dedektör Motoru
Projenin **merkezi dosyası**. Tüm tespit mantığını içerir:

| Bileşen | Açıklama |
|---------|----------|
| **CameraState** | Her kamera için durum ve istatistik tutma |
| **FireSmokeDetector** | YOLO modeliyle tespit yapan ana sınıf |
| **get_model()** | Modeli bir kez yükle, tüm kameralar paylaşsın |
| **send_gmail()** | Tespit edilirse e-mail gönder |
| **alarm_worker()** | Mail gönderme işlemini arka planda çalıştır |
| **save_snapshot()** | Tespit frame'ini kaydet |
| **draw_detection()** | Tespit kutularını frame'e çiz |

**Çalışma Akışı:**
1. Kamera kaynağı açılır (RTSP, USB, dosya)
2. Her frame işlenir (boyut, YOLO tahmini)
3. FRAME_SKIP parametresiyle performans optimize edilir
4. Yangın ve duman tespit edilirse:
   - Snapshot kaydedilir
   - E-mail gönderilir (cooldown ile)
   - Durum güncellenir
5. HUD bilgileri frame'e eklenir (FPS, tespit sayısı, durum)

---

### **3. `train.py`** — Model Eğitim Scripti
YOLOv8 modelini eğitmek için:

```bash
# Dataset hazırlama (klasörler oluştur)
python train.py setup

# Modeli eğit (50 epoch)
python train.py train

# Validsyon (test metrikleri)
python train.py validate
```

**Beklediği yapı:**
```
dataset/
├── images/
│   ├── train/      ← Eğitim görselleri
│   └── val/        ← Doğrulama görselleri
└── labels/
    ├── train/      ← YOLO format (.txt)
    └── val/
```

Eğitim sonrası model → `models/fire_smoke_yolov8.pt`

---

### **4. `service_fire_smoke.py`** — Windows NSSM Servisi
Sistem başlangıcında otomatik çalıştırılmak için NSSM ile kaydedilir:

```bash
nssm install FireSmoke "C:\path\to\python.exe" "C:\path\to\service_fire_smoke.py"
nssm start FireSmoke
```

- Flask'ı port 8505'te başlatır
- Log'ları `service_fire_smoke.log`'a yazar
- Hataları kaydeder

---

### **5. `index.html`** — Web Arayüzü
Modern, responsive arayüz:
- **Tab yapısı:** Kamera 1 | Kamera 2 | Snapshot Detayı
- **Canlı video:** MJPEG stream
- **Kontrol paneli:** Başlat/Durdur, kaynak seçme, güven eşiği slider
- **İstatistikler:** FPS, tespit sayısı, son tespitler
- **Snapshot galerisi:** Otomatik yenilenir
- **Global durum:** Başlıkta genel uyarı göstergesi
- **Gerçek zamanlı polling:** API'den her 500ms güncelleme

---

## ⚙️ Çevre Değişkenleri (`.env`)

```env
# Kamera kaynakları
CAMERA1_SOURCE=rtsp://admin:admin@192.168.2.29:554/stream1
CAMERA1_LABEL=Ofis
CAMERA2_SOURCE=rtsp://admin:Gedik.64@10.10.10.4:554/stream1
CAMERA2_LABEL=Depo

# Model ayarları
CONF_THRESHOLD=0.45          # Güven eşiği (0.0-1.0)
IOU_THRESHOLD=0.45           # IoU eşiği
FRAME_SKIP=3                 # Her 3. frame işle (performans)
DISPLAY_WIDTH=1280           # Frame genişliği
DISPLAY_HEIGHT=720           # Frame yüksekliği
RECONNECT_DELAY=3            # Bağlantı kesintisi sonrası yeniden deneme (sn)

# Alarm ayarları
ALARM_COOLDOWN=30            # Aynı kamera için en az bekleme (sn)
SAVE_SNAPSHOTS=true          # Tespit snapshot'larını kaydet

# Gmail bildirimleri
GMAIL_ENABLED=true
GMAIL_GONDEREN=your-email@gmail.com
GMAIL_SIFRE=your-app-password
GMAIL_ALICI=alert@company.com
```

---

## 🚀 Kurulum & Çalıştırma

### 1. Bağımlılıkları Yükle
```bash
pip install -r requirements.txt
```

### 2. `.env` Dosyası Oluştur
Kamera URL'leri ve Gmail bilgilerini gir.

### 3. Modeli İndir/Eğit
```bash
# Eğitilmiş model varsa: models/fire_smoke_yolov8.pt
# Yoksa eğitim datası topla ve çalıştır:
python train.py setup
# ... verileri ekle ...
python train.py train
```

### 4. Flask'ı Başlat
```bash
python app.py
```

Tarayıcı: `http://localhost:5050`

### 5. (Opsiyonel) NSSM Servisi Kur
```bash
python service_fire_smoke.py
```

---

## 🔍 Sistem Mantığı

```
┌─────────────────┐
│ RTSP Kameralar  │
└────────┬────────┘
         │ (cv2.VideoCapture)
         ↓
┌─────────────────────────────┐
│  FireSmokeDetector (Thread) │
├─────────────────────────────┤
│ 1. Frame oku                │
│ 2. Her 3. frame'i YOLO'ya   │
│ 3. Tespit: Yangın/Duman?    │
│ 4. Coodown kontrol          │
└────────┬────────────────────┘
         │
         ├→ Tespit VARSA:
         │  ├─ Snapshot kaydet
         │  ├─ Mail gönder (queue'ye koy)
         │  └─ Frame'e uyarı çiz
         │
         └→ Frame JPEG'e çevir
            ↓
        ┌─────────────┐
        │  CameraState│ (thread-safe)
        │  - frame_jpg│
        │  - fps      │
        │  - detections
        └─────┬───────┘
              │
              ↓
        ┌─────────────────┐
        │ Flask API       │
        ├─────────────────┤
        │ /video_feed     │ ← MJPEG Stream
        │ /api/status     │ ← JSON stat
        │ /api/start/stop │ ← Kontrol
        └────────┬────────┘
                 │
                 ↓
        ┌──────────────────┐
        │ Web Tarayıcısı   │
        │ (index.html)     │
        │ - Video oynat    │
        │ - Polling (500ms)│
        │ - UI güncelle    │
        └──────────────────┘
```

---

## 💡 Önemli Detaylar

### Thread Safety
- Her kamera kendi thread'de çalışır
- `CameraState.lock` ile veri tutarlılığı sağlanır
- Model tüm kameralar tarafından paylaşılır

### Performans Optimizasyonları
- **FRAME_SKIP:** Her 3. frame işlenerek CPU kullanımı düşürülür
- **Buffer Size:** `cv2.CAP_PROP_BUFFERSIZE = 1` ile lag minimize edilir
- **Shared Model:** Model memory'ye bir kez yüklenir
- **Queue Kullanımı:** Mail gönderme, main thread'i bloke etmez

### Duman Bastırması
YOLO'dan gelen bazı false positive'ler filtrelenir:
- Yangın kutusunun içinde duman tespit edilirse → duman görmezden gelinir
- Mantık: Yangın varsa zaten duman var, ayrı uyarı gerekli değil

### Error Handling
- Kamera bağlantısı koptuğunda otomatik yeniden deneme
- Model loading hataları için fallback mekanizması
- API'deki tüm istisnalar yakalanır ve loglanır

---

## 📊 API Yanıtları

### `GET /api/status/<cam_id>`
```json
{
  "cam_id": "cam1",
  "running": true,
  "fps": 24.5,
  "detection_active": false,
  "total_detections": 3,
  "conf_threshold": 0.45,
  "last_detections": [
    {"label": "fire", "conf": 0.87},
    {"label": "smoke", "conf": 0.92}
  ],
  "snapshot_count": 15,
  "source_label": "Ofis",
  "error_msg": ""
}
```

---

## 👨‍💼 Maintainer

- Aleyna Kapusuz
- Hande Bandırmalı

---

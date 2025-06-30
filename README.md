ESP32 ile Akıllı Işık Sistemi
Ne Yapar?
Bu proje ESP32 kartı ile WiFi üzerinden kontrol edilebilen bir LED ışık sistemi yapıyor. Telefondan veya bilgisayardan web sayfası açıp LED'leri kontrol edebiliyorsun.
Neler Var?

WiFi ile bağlan ve kontrol et
Farklı renkler seç (kırmızı, mavi, yeşil, beyaz)
Parlaklığı ayarla
Party modu (renkler otomatik değişir)
Otomatik mod (karanlık olunca açılır)

Malzemeler

ESP32 geliştirme kartı
RGB LED (4 pin)
LDR ışık sensörü
Bağlantı kabloları
Breadboard

Nasıl Çalışır?

ESP32'ye kodu yükle
"Akilli_Isik" WiFi ağına bağlan (şifre: 12345678)
Tarayıcıda 192.168.4.1 adresini aç
LED'leri kontrol et

Bağlantılar

Kırmızı LED -> D25
Yeşil LED -> D26
Mavi LED -> D27
LDR sensör -> D2

Özellikler

6 farklı mod var (kapalı, beyaz, renkli modlar, party, otomatik)
Parlaklık %1-100 arasında ayarlanıyor
LDR sensörü karanlığı algılayıp otomatik açıyor
Sayfa kendiliğinden güncelleniyor

Neden WiFi?
IR kumanda yerine WiFi tercih ettim çünkü:
telefondan veya bilgisayardan kontrol etmek kullanıcı kolaylığı sağlamak için

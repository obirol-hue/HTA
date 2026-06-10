# Web Süreç Analizci (HTA)

`SurecAnalizci.hta` — Girilen bir web adresine bağlanıp sayfanın **tüm süreçlerini** analiz eden ve bir **süreç dokümanı (rapor)** çıkaran, tamamen test/analiz amaçlı bir HTML Application (HTA) aracıdır. Herhangi bir site üzerinde çalışır.

## Ne yapar?

Bir URL girersiniz, "Analizi Başlat" dersiniz. Araç şunları çıkarır:

- **Genel bilgiler:** HTTP durum kodu, yanıt süresi, sayfa boyutu, Content-Type, sunucu bilgisi
- **Teknoloji tespiti:** Server/X-Powered-By başlıkları, WordPress, React, Angular, Vue, jQuery, Bootstrap, ASP.NET, Cloudflare, Google Analytics vb.
- **Sayfa yapısı:** başlık, dil, meta açıklama, H1–H6 başlık hiyerarşisi
- **Süreçler (formlar):** her formun yöntemi (GET/POST), hedefi (action) ve tüm alanları (input/select/textarea, zorunlu alanlar `*`) — bunlar sayfadaki etkileşimli süreçler olarak raporlanır
- **Bağlantılar:** iç/dış bağlantı sayıları ve listeleri
- **Kaynaklar:** script, stil, görsel, iframe sayıları
- **HTTP yanıt başlıkları:** tam liste

### Süreç Akış Haritası (tarama modu)

"İç bağlantıları tara" seçeneğini işaretlerseniz, araç iç linkleri belirlediğiniz sayfa limitine kadar (maks. 30) izleyerek her sayfanın süreçlerini çıkarır ve bir **süreç akış haritası** tablosu oluşturur.

### Rapor kaydetme

"Raporu Kaydet (HTML)" butonu, raporu HTA'nın bulunduğu klasöre `surec_raporu_<zaman>.html` adıyla bağımsız bir HTML dosyası olarak yazar.

## Çalıştırma

- Yalnızca **Windows** üzerinde çalışır (HTA = `mshta.exe`).
- Dosyaya çift tıklayın **veya** komut satırından:
  ```
  mshta.exe SurecAnalizci.hta
  ```

## Güvenlik / kapsam notu

- Araç hedef sisteme yalnızca **GET** isteği gönderir ve **pasif DOM analizi** yapar; hiçbir form göndermez, hiçbir veri değiştirmez.
- Sadece yetkili test, eğitim ve dokümantasyon amaçlı kullanın. Analiz ettiğiniz site için gerekli izinlere sahip olduğunuzdan emin olun.

## Teknik notlar

- HTTP istekleri için `Msxml2.ServerXMLHTTP` (cross-domain + tam başlık desteği) kullanılır.
- HTML, ana pencerede script çalıştırmadan güvenle ayrıştırılması için `htmlfile` ActiveX nesnesiyle işlenir.
- Rapor diske `Scripting.FileSystemObject` ile yazılır.

# Web Süreç Analizci (HTA)

`SurecAnalizci.hta` — Girilen bir web adresine bağlanıp sayfanın **tüm süreçlerini** analiz eden ve bir **süreç dokümanı (rapor)** çıkaran, tamamen test/analiz amaçlı bir HTML Application (HTA) aracıdır. Herhangi bir site üzerinde çalışır.

## Ne yapar?

Bir URL girersiniz, "Analizi Başlat" dersiniz. Araç şunları çıkarır:

- **Genel bilgiler:** HTTP durum kodu, yanıt süresi, sayfa boyutu, Content-Type, sunucu bilgisi
- **Teknoloji tespiti:** Server/X-Powered-By başlıkları, WordPress, React, Angular, Vue, jQuery, Bootstrap, ASP.NET, Cloudflare, Google Analytics vb.
- **Sayfa yapısı:** başlık, dil, meta açıklama, H1–H6 başlık hiyerarşisi
- **Süreçler (formlar):** her formun yöntemi (GET/POST), hedefi (action) ve tüm alanları — form içindeki **ve form dışındaki** (SPA/React) input/select/textarea. Her alan etiket + tür + ne topladığı (e-posta, şifre, telefon, adres, kart vb.) + zorunluluk bilgisiyle listelenir
- **Butonlar / tıklanabilir öğeler:** her butonun ne yaptığının yorumu (gönder, giriş, sepet, sil, sonraki adım vb.)
- **Bağlantılar:** iç/dış bağlantılar, metinleriyle "ne işe yarar" yorumu
- **Kaynaklar:** script, stil, görsel, iframe sayıları
- **HTTP yanıt başlıkları:** tam liste

### Anlamlı test verisi ve otomatik senaryolar ("Test verisi" seçeneği)

Varsayılan olarak açıktır. Her giriş alanının **formatına göre anlamlı test verisi** üretir (geçerli e-posta, güçlü şifre, telefon formatı, isim, adres, test kart numarası, tarih vb.) ve her form için **otomatik test senaryoları** çıkarır:

- Geçerli veri (happy path)
- Zorunlu alan(lar) boş
- Geçersiz format
- Sınır değer (uzun/maks.)
- Güvenlik: XSS denemesi
- Güvenlik: SQL injection denemesi

Ayrıca "Gönderilecek istek (önizleme)" bölümünde formun **ne göndereceğini** (URL + gövde) gösterir ama **göndermez**.

### Otomatik giriş + oturumla gezinme (opsiyonel — varsayılan KAPALI)

"Otomatik giriş" seçeneği açılırsa araç:
1. Giriş sayfasındaki **giriş formunu otomatik bulur** (şifre alanı olan form), gizli/**CSRF token** alanlarını korur
2. Girdiğiniz kullanıcı adı/e-posta ve şifreyi yerleştirip formu gönderir (`WinHttp` ile çerez ve yönlendirmeler otomatik taşınır)
3. Girişin başarılı olup olmadığını sezgisel doğrular (çıkış/hesap bağlantısı, şifre formunun kaybolması, oturum çerezi)
4. **Aynı oturumla** iç bağlantıları gezip her kimlik-doğrulamalı sayfayı analiz eder ("İç bağlantıları tara" da açık olmalı)

Kullanıcı/şifre alan adları otomatik tespit edilir; gerekirse elle de girebilirsiniz. Şifre ekranda ve raporda **maskelenir**, asla açık yazılmaz. Açmadan önce yetki onayı penceresi çıkar.

> ⚠️ Otomatik giriş, kimlik bilgilerinizle gerçek erişim yapar. **Yalnızca giriş yapma yetkiniz olan (kendi/test) sitelerde** kullanın.

### Canlı gönderim (opsiyonel — varsayılan KAPALI)

"Formları üretilen test verisiyle GERÇEKTEN gönder" seçeneği açılırsa, araç senaryoları gerçek GET/POST istekleriyle gönderir ve yanıtları değerlendirir (ör. XSS yükünün escape edilmeden yansıyıp yansımadığı, DB hata izi, boş zorunlu alanın yakalanıp yakalanmadığı). Açmadan önce bir onay penceresi çıkar.

> ⚠️ **Canlı gönderim hedef sistemi etkileyebilir** (kayıt oluşturma, mesaj gönderme vb.). Yalnızca test etme **yetkiniz olan** sitelerde kullanın.

### Süreç Akış Haritası ve Sayfalar Arası Veri Tutarlılığı (tarama modu)

"İç bağlantıları tara" seçeneğiyle araç iç linkleri (maks. 30 sayfa) izleyip her sayfanın amacını/süreçlerini çıkarır ve bir **süreç akış haritası** oluşturur. Ayrıca birden fazla sayfada görülen aynı adlı alanları karşılaştırarak **verinin sayfalar arası aynı kalıp kalmadığını** (oturum/CSRF token, önceden dolu değerlerin taşınması) raporlar.

### Rapor kaydetme

"Raporu Kaydet (HTML)" butonu, raporu HTA'nın bulunduğu klasöre `surec_raporu_<zaman>.html` adıyla bağımsız bir HTML dosyası olarak yazar.

## Çalıştırma

- Yalnızca **Windows** üzerinde çalışır (HTA = `mshta.exe`).
- Dosyaya çift tıklayın **veya** komut satırından:
  ```
  mshta.exe SurecAnalizci.hta
  ```

## Güvenlik / kapsam notu

- **Varsayılan mod (önizleme):** Araç yalnızca sayfaları çeker (GET), pasif DOM analizi yapar ve test verisi/senaryoları **üretir** — hiçbir form göndermez, hiçbir veri değiştirmez.
- **Canlı gönderim modu (opsiyonel):** Yalnızca siz açtığınızda ve onayladığınızda formları gerçek GET/POST istekleriyle gönderir. Bu, hedef sistemde değişiklik yaratabilir.
- Sadece yetkili test, eğitim ve dokümantasyon amaçlı kullanın. Test ettiğiniz site için gerekli izinlere sahip olduğunuzdan emin olun. Üçüncü taraf sitelerde canlı gönderim açmayın.

## Teknik notlar

- HTTP istekleri için `Msxml2.ServerXMLHTTP` (cross-domain + tam başlık desteği) kullanılır.
- HTML, ana pencerede script çalıştırmadan güvenle ayrıştırılması için `htmlfile` ActiveX nesnesiyle işlenir.
- Rapor diske `Scripting.FileSystemObject` ile yazılır.

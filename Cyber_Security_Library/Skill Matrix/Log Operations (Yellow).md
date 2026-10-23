# TryHackMe: Log Operations Notu

**Etiketler:** `#cybersecurity` `#soc-level-1` `#log-operations` `#detection-engineering` `#incident-response` `#compliance` `#threat-hunting`

  

## 1. Mimari Yapı ve Temel Mantık

Log Operasyonları (Log Operations), ham log verisini pasif bir veri yığını olmaktan çıkarıp tespit mühendisliği (detection engineering), olay müdahale (incident response) ve yasal denetimler için aksiyon alınabilir bir operasyonel mekanizmaya dönüştürme disiplinidir. "Samandağında iğne aramak" yerine, sistemlerin hangi amaçla ve ne derinlikte izleneceğini baştan tasarlamayı hedefler.

  

```
+---------------------------------------------------------------------------------------------------+
|                                  LOG OPERASYONLARI MİMARİSİ                                       |
+---------------------------------------------------------------------------------------------------+
|  [ PLANLAMA & KAPSAM (PLANNING & SCOPING) ]                                                      |
|  Amacın Belirlenmesi: Güvenlik, Operasyonel, Yasal Uyum, Hata Ayıklama (Debug)                    |
|  Konfigürasyon İkilemi: İhtiyaçlar (Gereksinimler) vs. Kaynaklar (Bütçe/İş Gücü) vs. Hedefler     |
+---------------------------------------------------------------------------------------------------+
                                            │
                                            ▼
|  [ TOPLAMA & STANDARTLAŞTIRMA (COLLECTION & NORMALISATION) ]                                      |
|  - Filtreleme: Gürültüyü (noise) engelleme, yalnızca analiz edilecek veriyi toplama              |
|  - Format Standardı: Zaman senkronizasyonu (NTP), ortak şema (CEE, Syslog, JSON)                  |
|  - Hassas Veri Maskeleme: PII, parola, kart numaralarının loga yazılmasını engelleme             |
+---------------------------------------------------------------------------------------------------+
                                            │
                                            ▼
|  [ SAKLAMA, ERİŞİM VE İZLEME (STORAGE, RETENTION & MONITORING) ]                                  |
|  - Yaşam Döngüsü & Arşivleme: Retention Politikaları (Örn: PCI DSS: 12 ay sakla, son 3 ay aktif) |
|  - Erişim Kontrolü ve Güvenlik: RBAC, şifreleme, bütünlük doğrulaması (SHA-256)                   |
|  - Aksiyon Alınabilir Uyarılar (Actionable Alerts): Gürültüsüz, triyajı mümkün alarmlar          |
+---------------------------------------------------------------------------------------------------+
                                            │
                                            ▼
|  [ SÜREKLİ DEĞERLENDİRME & GELİŞİM (CONTINUOUS EVOLUTION) ]                                       |
|  - "Çalışıyorsa dokunma!" mantığının reddi; tehdit dinamiklerine göre konfigürasyon güncelleme   |
|  - Proaktif Tehdit Avcılığı (Threat Hunting) ve dayanıklılık (resilience) testleri                |
+---------------------------------------------------------------------------------------------------+
```

## 2. Log Konfigürasyon Amaçları ve Kapsam Matrisi

Log konfigürasyonu tek tip bir süreç değildir. Sistemin hangi operasyonel veya idari ihtiyaca hizmet ettiğine göre dört temel sınıfa ayrılır:

  

|**Konfigürasyon Amacı**|**Odak Noktaları ve Kapsamı**|**Tipik Kullanım Senaryosu / Örnek**|
|---|---|---|
|**Security (Güvenlik)**|• Anomali ve tehdit tespiti<br><br>  <br>  <br><br>• Kullanıcı kimlik doğrulama kayıtları<br><br>  <br>  <br><br>• Sistem bütünlüğü ve veri gizliliğinin doğrulanması|Yetkisiz erişimlerin zamanında tespiti, brute-force veya yetki yükseltme aktivitelerinin analizi.|
|**Operational (Operasyonel)**|• Sistem hatalarını tespit etme ve giderme<br><br>  <br>  <br><br>• Performans, süreklilik ve güvenilirliği artırma<br><br>  <br>  <br><br>• Proaktif raporlama ve bildirimler<br><br>  <br>  <br><br>• **Hizmet faturalandırması (Service billing)** ve kapasite planlaması|Bir servisin kullanım maliyetini (cost of using a service) ölçmek, sunucu kaynak tüketimini izlemek.|
|**Legal (Yasal / Uyumluluk)**|• Standartlara, yasalara ve regülasyonlara uyum (ISO 27001, COBIT, GDPR, PCI DSS, HIPAA, FISMA)<br><br>  <br>  <br><br>• Yasal denetim (audit) gereksinimlerinin karşılanması|**PCI DSS Örneği:** Merkezi log yönetimi, 12 ay saklama (son 3 ayı anında aranabilir/hot olmalı), periyodik sistem kontrolleri.|
|**Debug (Hata Ayıklama)**|• Yazılımdaki hataların (bugs) tespiti<br><br>  <br>  <br><br>• Uygulama görünürlüğünü artırma<br><br>  <br>  <br><br>• Geliştirme sürecini hızlandırma<br><br>  <br>  <br><br>_(Genellikle geliştirme/test ortamlarında uygulanır)_|Uygulama kararlılığını ve performansını artırmak için uygulama çalışma zamanı loglarını inceleme.|

## 3. Planlama Süreci ve "Konfigürasyon İkilemi"

Log konfigürasyon sürecine başlarken sistem yöneticileri, güvenlik ekipleri, hukuk danışmanları ve bütçe yöneticileri bir araya gelerek planlama toplantıları yürütmelidir.

  

### Planlama Oturumlarında Sorulması Zorunlu Sorular

- **Neyi, ne için loglayacaksınız?** (Varlık kapsamı ve loglama amacı)
    
      
    
- **Bu amaca ulaşmak için ek taahhüt veya efor gerekiyor mu?**
    
      
    
- **Ne kadar ayrıntılı loglayacaksınız?** (Detay kapsamı)
    
      
    
- **Ne kadar loglamanız gerekiyor?** (Yasal ve regülatif zorunluluklar — _Örn: "PCI DSS gereksinimlerinde belirtildiği kadar"_)
    
      
    
- **Nasıl loglayacaksınız?** (Toplama ve taşıma yöntemleri)
    
      
    
- **Toplanan logları nasıl saklayacaksınız?** (Uyulması gereken bir saklama standardı veya mevzuat var mı?)
    
      
    
- **Logları nasıl koruyacaksınız?** (Erişim yönetimi, şifreleme, bütünlük)
    
      
    
- **Toplanan logları nasıl analiz edeceksiniz?**
    
      
    
- **Yeterli kaynak ve iş gücünüz var mı?**
    
      
    
- **Planlama, uygulama ve bakım için yeterli bütçeniz var mı?**
    
      
    

### Konfigürasyon İkilemi: Gereksinimler vs. Aspirasyonlar

```
+---------------------------------------------------------------------------+
|                          KONFİGÜRASYON İKİLEMİ                            |
+---------------------------------------------------------------------------+
|  GEREKSİNİMLER (REQUIREMENTS)          ASPİRASYONLAR (ASPIRATIONS)        |
|  • Reaktif & Olay Tespiti              • Proaktif & Tehdit Avcılığı       |
|  • Bilinen tehditlere karşı temel      • İleri düzey & sofistike tehditler|
|  • TARTIŞMASIZDIR (NON-NEGOTIABLE)     • Yüksek bütçe ve iş gücü ister   |
+---------------------------------------------------------------------------+
```

|**Boyut**|**Soru Seti ve Yaklaşım**|**Zihniyet & Karakteristik**|
|---|---|---|
|Temel Gereksinimler<br><br>  <br>  <br><br>(Base Requirements)|• Ne oldu? (What happened?)<br><br>  <br>  <br><br>• Ne zaman oldu? (When did it happen? — Zaman damgası)<br><br>  <br>  <br><br>• Nerede oldu? (Where did it happen? — Ağ, sistem, dizin, arayüz)<br><br>  <br>  <br><br>• Kim/Ne sebep oldu? (Who/What caused it?)<br><br>  <br>  <br><br>• Hangi log kaynağından geldi?|**Reaktif Olay Tespiti (Incident Detection):** Bilinen tehditleri yakalamak için asgari temel çerçevedir. **Operasyonel ve güvenlik gereksinimleri pazarlık konusu yapılamaz (non-negotiable).**|
|Aspirasyonlar<br><br>  <br>  <br><br>(Aspirations for Better Insights)|• Daha fazla veri almak mümkün mü? (Daha fazla detay)<br><br>  <br>  <br><br>• Bunun doğru olduğundan ne kadar emin olabilirim?<br><br>  <br>  <br><br>• Neler etkilendi? (What is affected?)<br><br>  <br>  <br><br>• Bir sonraki adımda ne olacak? (What will happen next?)<br><br>  <br>  <br><br>• Dikkat gerektiren başka bir şey var mı?<br><br>  <br>  <br><br>• Olay hakkında ne yapmalıyım?|**Proaktif Tehdit Avcılığı (Threat Hunting):** İleri düzey ve karmaşık saldırıları tespit etmeyi amaçlar. Daha fazla donanım kaynağı, analist yetkinliği ve bütçe gerektirir.|

> [!WARNING]
> 
> **Pazarlık Masası Kuralı (Mülakat/Danışmanlık İpucu):**
> 
> Bütçe ve kaynak kısıtlamaları tartışılırken maliyet düşürmek için operasyonel süreklilik ve asgari güvenlik loglama seviyesinden ödün verilemez. **Operasyonel ve güvenlik gereksinimleri (operational and security requirements) kesinlikle tartışılamazdır (non-negotiable).** Kısıntı yapılacaksa proaktif aspirasyon seviyelerinden yapılmalıdır.
> 
>   

## 4. Temel Loglama Prensipleri ve Karşılaşılan Zorluklar

### Temel Loglama Prensipleri (Logging Principles)

```
[ Collection ] ──> [ Format ] ──> [ Archiving/Access ] ──> [ Monitoring ] ──> [ Security ] ──> [ Continuous Change ]
```

- 📥 **Toplama (Collection):** Loglama amacını net tanımlayın. Yalnızca ihtiyaç duyulan ve analiz edilecek veriyi toplayın. Gereksiz verileri toplamaktan kaçının ve log gürültüsünü (noise) engelleyin.
    
      
    
- 📏 **Format:** Doğru detay ve seviyede loglayın. Tüm sistemlerde tutarlı bir format standardı uygulayın. Zaman damgalarının (timestamps) kesin ve senkronize (NTP) olduğundan emin olun.
    
      
    
- 📦 **Arşivleme ve Erişilebilirlik (Archiving and Accessibility):** Log saklama (retention) politikalarını belirleyin ve uygulayın. Kritik analiz verilerinin anında sorgulanabilir olmasını sağlayın; yedekleme mekanizması kurun.
    
      
    
- 🚨 **İzleme ve Uyarı (Monitoring and Alerting):** Yalnızca kayda değer ve aksiyon alınabilir (actionable) durumlar için uyarı kuralı yazın. Analist körlüğüne yol açan yanlış alarm gürültüsünü eleyin.
    
      
    
- 🔒 **Güvenlik (Security):** Loglara erişimi sıkı rol tabanlı kontrollerle (RBAC) sınırlayın. Gerekli durumlarda şifreleme uygulayın ve özel log yönetim çözümleri (SIEM) kullanın.
    
      
    
- 🔄 **Sürekli Değişim (Continuous Change):** Log kaynakları, tipleri ve mesajları sürekli güncellenir. Değişime açık olun, kuralları güncelleyin ve analist kadrosunu düzenli eğitin.
    
      
    

### Karşılaşılan Zorluklar (Logging Challenges)

|**Zorluk Alanı**|**Operasyonel Engeller ve Detaylar**|
|---|---|
|Veri Hacmi ve Gürültü<br><br>  <br>  <br><br>(Data Volume & Noise)|• Çok sayıda heterojen veri kaynağı.<br><br>  <br>  <br><br>• Uygulamalar arası hacim dengesizliği (bazı sistemler yetersiz üretirken büyük ölçekliler petabaytlarca log basar).<br><br>  <br>  <br><br>• Önemsiz verilerin asıl tehdit izlerini gölgelemesi.|
|Sistem Performansı ve Toplama<br><br>  <br>  <br><br>(System Performance & Collection)|• Ajanların (agents) CPU/RAM tüketerek sistemleri yavaşlatması.<br><br>  <br>  <br><br>• Dokunulamayan hassas veya eski/legacy sistemlerin varlığı.<br><br>  <br>  <br><br>• Binlerce uç noktada ajan güncellemelerinin yönetimi.|
|İşleme ve Arşivleme<br><br>  <br>  <br><br>(Process and Archive)|• **Farklı formatlardaki logların ayrıştırılması (parsing) sırasında ortaya çıkan aktarım ve şema hataları.**<br><br>  <br>  <br><br>• Çok sayıda regülasyon varken saklama (retention) maliyet dengesini kuramamak.|
|**Güvenlik (Security)**|• Log depolarının yetkisiz erişimlere veya tahrifata (tampering) karşı güvenliğini sağlamak.|
|**Analiz (Analysis)**|• Çoklu kaynakları gerçek zamanlı birleştirmek ve korelasyon kurmak için devasa hesaplama gücü gerekmesi.<br><br>  <br>  <br><br>• Yanlış pozitif (False Positive) ve yanlış negatiflerin (False Negative) analistleri tüketmesi.|
|**Çeşitli (Misc)**|• Yol haritası ve bütçe eksikliği.<br><br>  <br>  <br><br>• Senaryo, playbook ve tatbikat eksikliği.<br><br>  <br>  <br><br>• Analiz aşaması yerine yalnızca toplamaya odaklanma yanılgısı.<br><br>  <br>  <br><br>• İnsan hatası faktörü.|

## 5. Yaşanmış Olay İncelemesi: Windows 7 & EternalBlue

Konfigürasyonun güncellenmemesi ve varsayılan ayarlara güvenilmesinin felaketle sonuçlanabileceğini kanıtlayan en somut vaka:

  

```
[ EternalBlue (CVE-2017-0144) ] ──> Hedef: MS Windows 7 (SMBv1) ──> Tam Sistem Erişimi (RCE)
                                                │
                                                ▼
           [ Varsayılan Log Konfigürasyonu: Yetersiz / Sıfır Kayıt ]
           System, Security ve Application günlüklerinde anlamlı alarm yok!
```

> [!NOTE]
> 
> **Vaka Analizi:**
> 
>   
> 
> - **Olay:** Varsayılan log konfigürasyonu nedeniyle sistem tamamen ele geçirilmesine rağmen adli iz bırakmadı.
>     
>       
>     
> - **Zafiyet:** MS Windows 7 SMBv1 istismarı olan **EternalBlue (CVE-2017-0144)**.
>     
>       
>     
> - **Etki / Skor:** Tam sistem kontrolü (Full Compromise) — CVSS 3 Skoru: **8.1 (High)**.
>     
>       
>     
> - **Kritik Ders:** 2017'de istismar edilen ve 2020'de desteği biten Windows 7 üzerinde varsayılan `System`, `Security` ve `Application` logları hiçbir anlamlı olay kaydı üretmemiştir. "Sistem çalışıyor, dokunmayalım" yaklaşımı sistemin kör noktada kalmasına yol açmıştır.
>     
>       
>     

## 6. Yaygın Hatalar (Don'ts) vs. En İyi Uygulamalar (Dos)

|**❌ Yaygın Hatalar (Don'ts)**|**En İyi Uygulamalar (Dos)**|
|---|---|
|**Hassas bilgileri (parola, API key, PII) loglamak.**|Hassas bilgileri loglama kapsamı dışında bırakmak / maskelemek.|
|**Standart dışı, kendi yazdığın özel betiklerle eksik log üretmek.**|Sistemlere ve endüstri standartlarına uygun yapılandırılmış log modelleri kurmak.|
|Sistemlerde toplanmayan, yerelde unutulan loglar bırakmak.|Ölçeklenebilirlik, işlevsellik ve kararlılık testlerini düzenli uygulamak.|
|Her şeyi toplayıp hiçbir analiz yapmamak (Data Hoarding).|Toplanan logların güvenliğini ve bütünlüğünü (kriptografik hash) sağlamak.|
|Plansız ve rastgele log toplamak.|Yalnızca aksiyon alınabilir ve anlamlı alarmlar / bildirimler üretmek.|
|Sistemleri gereken log konfigürasyonundan yoksun bırakmak.|Analistleri sürekli eğitmek ve tespit yeteneklerini geliştirmek.|
|Yalnızca uç sınırları (perimeter) izleyip iç ağ sistemlerini atlamak.|Operasyon planlarını ve varlık konfigürasyonlarını değişen tehditlere göre güncellemek.|
|_"Görmek istediğini aramak, gördüğünü araştırmamak."_|Tehdit istihbaratı odaklı proaktif analiz yapmak.|

> [!TIP]
> 
> **Özel Betik (Custom Script) Hatası Analizi:**
> 
> Eski veya uyumsuz bir sistemden log alabilmek için geliştirme ekibinin kendi yazdığı ve bazı aşamalarda logları atlayan (omit) özel bir betik kullanması tipik bir **Mistake (Hata)** örneğidir. Eksik veri üreten ve bakım garantisi olmayan betikler olay anında delil zincirini koparır.
> 
>   

## 7. Pratik ve Laboratuvar Soru-Cevap Anahtarı

|**#**|**Soru Metni**|**Doğru Cevap**|**Teknik Açıklama / Dayanak**|
|---|---|---|---|
|**1**|Which of the given log purposes would be suitable to measure the cost of using a service?|`Operational`|Hizmet faturalandırması (service billing) ve maliyet ölçümü operasyonel amaçlı loglamanın kapsamındadır.|
|**2**|Which of the given log purposes would be suitable for investigating application logs for enhancement and stability?|`Debug`|Uygulama kararlılığını artırma ve hata ayıklama süreçleri Debug amaçlı log konfigürasyonuna girer.|
|**3**|Which question's answer can be "as much as mentioned in the PCI DSS requirements."?|`How much do you need to log?`|"Ne kadar loglamanız gerekiyor?" sorusunun yanıtı doğrudan tabi olunan yasal standart (PCI DSS) sınırlarıdır.|
|**4**|Which requirements are non-negotiable?|`operational and security requirements`|Bütçe ve kaynak tartışmalarında operasyonel ve güvenlik gereksinimleri kesinlikle pazarlık konusu yapılamaz.|
|**5**|Your team is working on policies to decide which logs will be stored and which portion will be available for analysis. Which of the given logging principles would be implemented and improved?|`Archiving and Accessibility`|Logların saklanma süresi ve analiz için erişilebilir olması ilkesi Arşivleme ve Erişilebilirlik prensibidir.|
|**6**|Your team implemented a brand new API logging product. One of the team members reported continuous errors when transferring the logs to the review platform. In this case, which of the given difficulties occurs?|`Process and Archive`|Verilerin analiz platformuna aktarılırken format uyuşmazlığı nedeniyle hata vermesi İşleme ve Arşivleme zorluğudur.|
|**7**|As a consultant, you noticed that one of the development teams implemented a custom script to generate logs for an old system, which omits loggings at some phases. What you would call this?|`Mistake`|Kritik loglama aşamalarını atlayan kontrolsüz özel betikler kullanmak en yaygın operasyonel hatalardan biridir.|

## 8. Tekrar Kategorisi ve 3 Dakikalık Pazar Stratejisi

- **Sınıf:** 🟡 **Sarı (Kavramsal / 3 Dakika Soru-Cevap)**
    
      
    
- **Gerekçe:** Oda komut ezberinden ziyade; konfigürasyon amaçları (Security, Operational, Legal, Debug), PCI DSS uyumluluk kuralları, loglama ilkeleri (Archiving & Accessibility vs. Process & Archive) ve yönetimsel karar matrislerine dayanmaktadır.
    
      
    

### ⏱️ Pazar Günü 3 Dakikalık Hızlı Tekrar Stratejisi

1. **Dakika 00:00 - 01:00 (Konfigürasyon Amaçları & İkilem):**
    
      
    - Zihninde 4 amacı ayır: _Cost of service / Billing_ $\rightarrow$ **Operational**, _Bug fix / Stability_ $\rightarrow$ **Debug**, _Compliance / PCI DSS_ $\rightarrow$ **Legal**.
        
          
        
    - Pazarlık masasında neyin tartışılamaz olduğunu hatırla: **Operational & Security Requirements (Non-negotiable)**.
        
          
        
2. **Dakika 01:00 - 02:00 (Zorluklar ve Prensipler):**
    
      
    - _Veri aktarımı ve şema hatası_ $\rightarrow$ **Process and Archive** zorluğu.
        
          
        
    - _Hot (anında erişilebilir) vs. Cold (arşiv) ayrımı_ $\rightarrow$ **Archiving and Accessibility** prensibi.
        
          
        
3. **Dakika 02:00 - 03:00 (Örnek Olay ve Hatalar):**
    
      
    - **EternalBlue (CVE-2017-0144)** Windows 7 varsayılan log körlüğü tablosunu anımsa.
        
          
        
    - Özel log üretici betiklerin eksik log atlamasının neden **Mistake** olduğunu doğrula.
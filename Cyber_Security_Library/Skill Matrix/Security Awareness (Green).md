# 📓 Security Awareness (Güvenlik Farkındalığı ve Tehdit Aktörleri)

**Tags:** #BlueTeam #SecurityAwareness #CyberHygiene #ThreatActors #DataBreaches #GDPR #SOC_L1 #TryHackMe #Walkthrough

## 🏗️ Mimari Yapı

**Security Awareness (Güvenlik Farkındalığı)**, bir organizasyonun teknik savunma duvarlarının (Firewall, SIEM, Antivirüs) ötesinde yer alan **insan faktörünü (Human Factor)** güçlendirmeyi amaçlayan bütünsel siber hijyen mimarisidir.

Teknik sistemler ne kadar gelişmiş olursa olsun, insan faktörü güvenlik zincirinin en zayıf halkası kabul edilir. Yapılan araştırmalara ve **IBM Siber Güvenlik Raporu'na göre, başarılı siber saldırıların %95'inin temel nedeni insan hatalarıdır (human errors)**.

```
                      [ Kurumsal Güvenlik Mimarisi ]
                                    │
         ┌──────────────────────────┴──────────────────────────┐
         ▼                                                     ▼
[ Teknik Katman (Hardware/Software) ]               [ İnsan Katmanı (Human Layer) ]
- Firewall, SIEM, EDR, E-posta Filtreleri            - Güvenlik Farkındalığı (Awareness)
- Otomatik Kural Setleri                             - %95 İnsan Hatası Riskini Azaltma
```

Güvenlik farkındalığı eğitimi; şirket çalışanlarının veri ihlallerini önlemesini, oltalama (phishing) ve sosyal mühendislik (social engineering) vektörlerini anında fark etmesini sağlayarak kurumun genel siber savunma kapasitesini artırır.

## 💻 Tüm Komutlar ve Parametreler

Bu döküman insan faktörü, veri güvenliği ve tehdit aktörlerinin motivasyonlarını inceleyen teorik/farkındalık odaklı bir eğitim içeriği olduğundan doğrudan terminal komutu veya kod parçacığı içermemektedir. Kurumsal istatistikler ve tehdit aktörlerinin sınıflandırması **Teknik Veriler ve Sınıflandırmalar** bölümünde tablolaştırılmıştır.

## 📊 Teknik Veriler ve Sınıflandırmalar

### A. Eğitimin Kurumsal Etkisi ve İstatistiksel Veriler (Proofpoint Study)

Siber güvenlik farkındalık eğitimlerinin kurumsal kurumlardaki saldırı engelleme başarı oranları:

|**Sektör / Kurum Türü**|**Engellenen Saldırı / Risk Türü**|**Risk Azalma Oranı (%)**|
|---|---|---|
|**Finans Kuruluşları (Financial Institutions)**|Zararlı Yazılım (Malware) ve Virüs Saldırıları|**%95 Azalma**|
|**Eğitim Kurumları (Educational Institutions)**|Oltalama (Phishing) Saldırı Girişimleri|**%90 Azalma**|
|**Kamu / Devlet Kurumları (Government Employees)**|Dolandırıcılık (Fraud) Saldırıları|**%80 Azalma**|

### B. Siber Tehdit Aktörleri (Cyber Threat Actors) ve Motivasyon Tablosu

Süreçlerde sistemleri ve kurumları hedef alan tehdit aktörlerinin motivasyonel sınıflandırması:

|**Tehdit Aktörü Grubu (Threat Actor)**|**Temel Saldırı Motivasyonu**|**Hedef Seçimi ve Davranış Tipi**|
|---|---|---|
|**Cybercriminals** _(Siber Suçlular)_|**Finansal Kazanç** (Financial Motivation)|Şirketleri, bankaları ve bireyleri fidye yazılımları (Ransomware) veya veri sızıntılarıyla soyarak para kazanmayı hedefler.|
|**Nation-State Actors** _(Devlet Destekli Aktörler)_|**Geopolitik** (Geopolitical Motivation)|Kritik altyapılara, askeri sistemlere veya rakip devletlerin kurumlarına siber casusluk (APT) düzenler.|
|**Hacktivists** _(Haktivistler)_|**İdeolojik** (Ideological Motivation)|Belirli bir siyasi, sosyal veya çevresel mesajı topluma duyurmak için web sitelerini tahrif eder (deface) veya çökertir (DDoS).|
|**Insider Threat Actors** _(İç Tehdit Aktörleri)_|**Memnuniyetsizlik / İntikam** (Discontent)|Şirket içinden erişim yetkisi olan huzursuz veya işten çıkarılmış çalışanların veri çalması veya sisteme zarar vermesi.|
|**Thrill-Seekers** _(Eğlence / Macera Arayanlar)_|**Kişisel Tatmin** (Satisfaction / Fun)|Tamamen eğlence, hava atma veya yeteneğini kanıtlama amacıyla kişisel bilgisayarları veya sistemleri hackleyenler.|
|**Terrorist Groups** _(Terörist Gruplar)_|**İdeolojik Şiddet** (Ideological Violence)|Toplumda korku, kaos ve kamu düzeninde kargaşa yaratmak amacıyla kritik sistemleri hedef alırlar.|

## 🧠 Uzman Notları ve Kritik Uyarılar

> ⚠️ **MÜLAKAT HAPI - VERİ İHLALİNİN (DATA BREACH) ŞİRKETLERE BÜTÜNSEL MALİYETİ:**
> 
> Kurumsal siber güvenlik ve SOC mülakatlarında _"Bir veri ihlalinin kuruma zararları sadece finansal mıdır?"_ sorusu geldiğinde verilecek analist yanıtı:
> 
> Veri ihlalleri ortalama **3.86 milyon dolar** doğrudan finansal zarara yol açar. Ancak finansal kaybın yanında 5 kritik kurumsal darbe vurur:
> 
> 1. **Yasal Cezalar:** GDPR, KVKK ihlalleri ve tazminat davaları.
>     
> 2. **Reputational Damage:** Müşteri ve ortakların güveninin kaybolması, marka değerinin çökmesi.
>     
> 3. **Ticari Aksamalar (Disruption to Trading):** Sistemlerin durması sonucu iş sürekliliğinin (Business Continuity) kesintiye uğraması.
>     
> 4. **Hassas Veri Kaybı:** Müşteri kredi kartları, çalışan bilgileri veya ticari sırların ele geçirilmesi.
>     
> 5. **Oltalama Hedefine Dönüşme:** Çalınan verilerin gelecekte hedef odaklı sosyal mühendislik (Spear Phishing) saldırılarında kullanılması.
>     

> 💡 **SEKTÖREL İPUCU - HAVE I BEEN PWNED (HIBP) VE VERİ İHLALİ TAKİBİ:**
> 
> Siber güvenlik analistleri ve SOC çalışanları, kurum çalışanlarının veya şahısların daha önceki veri ihlallerinde (Data Breaches) e-posta veya telefon numaralarının sızıp sızmadığını kontrol etmek için **Have I Been Pwned (`haveibeenpwned.com`)** platformunu kullanır. İhlale karışmış bir e-posta tespit edildiğinde ilk yapılan aksiyon, parola karmaşıklığını artırmak ve **Password Manager** (Bitwarden, KeePass vb.) kullanımına geçmektir.

## 🗃️ Soru-Cevap Anahtarı

| **Pratik / Laboratuvar Sorusu**                                                                               | **Doğru Cevap**                         |
| ------------------------------------------------------------------------------------------------------------- | --------------------------------------- |
| **[HISTORICAL DATA]** eBay siber saldırıya uğradığında kaç milyon insan bu ihlalden etkilenmiştir?            | `145 million`                           |
| **[HISTORICAL DATA]** PlayStation sızmasında hacklenen ve dışarı sızdırılan hassas veri türleri nelerdir?     | `names, addresses, e-mail, birth dates` |
| Bir kurumu veya ticari işletmeyi hacklemekle en çok ilgilenen (finansal motivasyonlu) tehdit aktörü kimdir?   | `Cybercriminals`                        |
| Eğlence ve kişisel tatmin amacıyla kişisel bir bilgisayarı hacklemeyle en çok ilgilenen tehdit aktörü kimdir? | `Thrill-seekers`                        |
| Topluma veya kurumlara bir mesaj iletmek amacıyla bir web sitesini hackleyen ideolojik tehdit aktörü kimdir?  | `Hacktivists`                           |
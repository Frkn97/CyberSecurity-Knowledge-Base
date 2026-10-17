# Diamond Model of Intrusion Analysis (Saldırı Analizi Elmas Modeli)

> **Etiketler:** `#cybersecurity` `#threat-intelligence` `#soc-analyst` `#diamond-model` `#intrusion-analysis` `#incident-response` `#tryhackme`
> 
>   

## 1. Mimari Yapı ve Temel Mantık

**Diamond Model of Intrusion Analysis**, 2013 yılında siber güvenlik uzmanları **Sergio Caltagirone, Andrew Pendergast ve Christopher Betz** tarafından geliştirilen, her türlü izinsiz giriş ve saldırı eyleminin **temel atomik bileşenini** modelleyen bilimsel bir çerçevedir.

  

### Temel Çalışma Mekanizması

- Model, bir siber saldırıyı dört çekirdek özelliğin (**Adversary**, **Capability**, **Infrastructure**, **Victim**) birbirine kenarlarla bağlandığı bir **elmas (diamond)** geometrisi şeklinde temsil eder.
    
      
    
- Dört temel bileşen arasındaki kenar bağlantıları, saldırı sırasındaki nedensellik ve ilişki bağlarını gösterir.
    
      
    
- Çekirdek özelliklerin yanı sıra modeli zenginleştiren **iki temel eksen / bileşen** bulunur:
    
      
    - **Social-Political (Sosyo-Politik Eksen):** Saldırganın niyeti, güdüsü ve kurban ile arasındaki çıkar ilişkisini tanımlar.
        
          
        
    - **Technology (Teknoloji Ekseni):** Saldırganın yetenekleri (Capability) ile altyapısı (Infrastructure) arasındaki teknik iletişim ve uygulama bağını gösterir.
        
          
        

```
                     [ Adversary ]
                     /     |     \
                    /      |      \
         (Social-Political)|       \
                  /        |        \
                 /         |         \
   [ Capability ]----------+----------[ Infrastructure ]
                 \         |         /
                  \        |        /
                   \       |       /
                    \      |      /
                     [  Victim  ]
```

### SOC ve CTI Ekipleri İçin Değeri

- **Gerçek Zamanlı İstihbarat Entegrasyonu:** Ağ savunmasında tespit edilen olayların gerçek zamanlı ilişkilendirilmesini sağlar.
    
      
    
- **Kampanya Sınıflandırması:** Olayları belirli güvenilirlik dereceleriyle tehdit aktörlerinin yürüttüğü kampanyalarla eşleştirir.
    
      
    
- **Operasyonel Öngörü (Forecasting):** Bir saldırı adımından sonra gelebilecek sonraki adımları tahmin ederek proaktif mitigasyon ve simülasyon stratejileri kurmaya imkân tanır.
    
      
    
- **Teknik Olmayan Paydaş İletişimi:** Teknik analizi; C-Level yöneticilere, müşterilere ve idari birimlere sade, yapılandırılmış bir hikaye akışıyla aktarmayı kolaylaştırır.
    
      
    

## 2. Dört Çekirdek Bileşen (Core Features)

```
+------------------------------------------------------------------------------------------------------+
|                                    DIAMOND MODEL ÇEKİRDEK ELEMANLARI                                  |
+------------------------------------------------------------------------------------------------------+
| 1. ADVERSARY (Saldırgan)       : Saldırının arkasındaki aktör/organizasyon.                          |
| 2. CAPABILITY (Yetenek)        : Saldırganın kullandığı araçlar, teknikler ve zararlı yazılımlar.     |
| 3. INFRASTRUCTURE (Altyapı)    : Yeteneğin iletildiği ve kontrol edildiği donanımsal/mantıksal hatlar.|
| 4. VICTIM (Kurban)             : Saldırganın hedef aldığı şahıs, kurum, IP, alan adı ve varlıklar.   |
+------------------------------------------------------------------------------------------------------+
```

### 2.1 Adversary (Saldırgan / Tehdit Aktörü)

Siber saldırının arkasında duran şahıs, grup veya devlete bağlı organizasyondur. Saldırgan bilgisi ilk tespit anında çoğunlukla boştur (unknown) veya gizemlidir; adli kanıtlar ve ilişkilendirmeler toplandıkça aydınlatılır.

  

- **Adversary Operator:** Saldırı eylemini, klavye başındaki operasyonu yürüten kişi ya da "hacker" grubudur.
    
      
    
- **Adversary Customer:** Gerçekleştirilen saldırıdan doğrudan fayda, kâr veya stratejik istihbarat sağlayacak nihai kişi, kurum ya da devlettir.
    
      
    

> [!NOTE]
> 
> **Operatör ve Müşteri Ayrımı:** Bir _Adversary Customer_, birbirinden bağımsız çalışan birden fazla _Adversary Operator_'ü finanse edebilir veya yönetebilir. Bu operatörlerin her biri kendi altyapılarına ve zararlı yazılım cephanelerine sahip olabilir. Bu ayrım, tehdit atıfı (attribution), adaptasyon yeteneği ve kalıcılık (persistence) analizinde kritik rol oynar.
> 
>   

### 2.2 Victim (Kurban / Hedef)

Saldırganın yöneldiği, sistemlerine sızmaya çalıştığı hedef noktadır. Kurban bir kurum, birey, IP adresi, e-posta adresi veya alan adı olabilir. Saldırganın organizasyonda tutunma noktası (foothold) elde etmesi için kurban bir basamaktır.

  

- **Victim Personae:** Hedef alınan kurumlar ve şahıslardır.
    
      
    - Örnekler: Şirket adları, personel isimleri, sektörler, iş unvanları, kişisel ilgi alanları.
        
          
        
- **Victim Assets:** Saldırganın yeteneklerini yönelttiği saldırı yüzeyini (attack surface) oluşturan teknik varlıklardır.
    
      
    - Örnekler: Ağ sistemleri, sunucular, ana makineler (hosts), e-posta adresleri, IP adresleri, sosyal medya hesapları.
        
          
        

### 2.3 Capability (Yetenek / TTP)

Saldırganın saldırı olayında kullandığı beceriler, araçlar ve tekniklerdir. Saldırganın Taktik, Teknik ve Prosedürlerini (TTP) yansıtır. Basit parola deneme/tahmin işlemlerinden, özel sıfır-gün (zero-day) açıklarından faydalanan gelişmiş zararlı yazılımlara kadar uzanır.

  

- **Capability Capacity (Yetenek Kapasitesi):** Tek bir zararlı yeteneğin ya da exploit'in sömürebileceği zafiyetlerin ve açıklıkların tamamıdır.
    
      
    
- **Adversary Arsenal (Saldırgan Cephaneliği):** Bir saldırgana/gruba ait tüm yeteneklerin ve araç setlerinin toplamıdır.
    
      
    

> [!INFO]
> 
> Saldırganın kendi yazılımını geliştirecek teknik kapasitesi yoksa bile "Ransomware-as-a-Service (RaaS)" veya "Malware-as-a-Service" modelleriyle bu cephaneliğe dışarıdan erişim sağlayabilir.
> 
>   

### 2.4 Infrastructure (Altyapı)

Saldırganın yeteneklerini hedefe ulaştırmak (deliver), saldırıyı yönetmek veya komuta-kontrol (C2) ile veri sızdırma (data exfiltration) operasyonlarını sürdürmek için kullandığı mantıksal ve fiziksel ara bağlantılardır.

  

- **Type 1 Infrastructure (1. Tip Altyapı):** Doğrudan saldırganın mülkiyetinde olan veya tamamen saldırgan tarafından kontrol edilen sistemlerdir (ör. doğrudan kiralanan C2 sunucusu).
    
      
    
- **Type 2 Infrastructure (2. Tip Altyapı):** Bir aracı (intermediary) tarafından kontrol edilen ve saldırgan tarafından ele geçirilip kullanılan sistemlerdir. Aracı taraf durumun farkında olabilir veya olmayabilir. Saldırının asıl kaynağını gizlemek/bulanıklaştırmak (obfuscation) amacıyla kullanılır (ör. ele geçirilmiş web siteleri, zombi makineler, ele geçirilmiş e-posta hesapları, kötü niyetli alan adları).
    
      
    
- **Service Providers (Hizmet Sağlayıcılar):** Type 1 ve Type 2 altyapıların ayakta kalması ve sürekliliği için kritik hizmetler sunan internet servis sağlayıcıları (ISP), alan adı tescil kuruluşları (registrars) ve webmail platformlarıdır.
    
      
    

## 3. Altı Meta Özellik (Meta-Features)

Meta özellikler zorunlu olmamakla birlikte, Elmas Model analizine yüksek değerli tehdit istihbaratı ve bağlam kazandırır:

  

|**Meta Özellik**|**Tanım ve Kapsam**|**Örnek / Değer Kümesi**|**Analitik Değeri**|
|---|---|---|---|
|**Timestamp**|Olayın gerçekleştiği tarih ve saat (başlangıç ve bitiş).|`2021-09-12 02:10:12.136`|Olay örüntülerini gruplar. Yerel saat farkları saldırganın hangi saat diliminden (ülkeden) çalıştığına dair ipucu verir.|
|**Phase**|Bir saldırı veya sızıntının ardışık adımlarıdır.|Cyber Kill Chain adımları (Recon, Weaponization, Delivery, Exploitation, Installation, C2, Actions on Objectives).|**Axiom 4:** "Her zararlı aktivite, istenen sonuca ulaşmak için art arda başarıyla yürütülmesi gereken iki veya daha fazla aşama içerir."|
|**Result**|Saldırganın operasyonunun post-kondisyonları ve çıktısı.|`Success`, `Failure`, `Unknown` veya CIA üçlüsü: `Confidentiality Compromised`, vb.|Operasyonun başarı derecesini ve zafiyete uğrayan güvenlik sacayağını belgeler.|
|**Direction**|Saldırının host veya ağ bazlı yönünü ifade eder.|• `Victim-to-Infrastructure`<br><br>  <br>  <br><br>• `Infrastructure-to-Victim`<br><br>  <br>  <br><br>• `Infrastructure-to-Infrastructure`<br><br>  <br>  <br><br>• `Adversary-to-Infrastructure`<br><br>  <br>  <br><br>• `Infrastructure-to-Adversary`<br><br>  <br>  <br><br>• `Bidirectional`<br><br>  <br>  <br><br>• `Unknown`|İletişimin nereden nereye başlatıldığını belirler (örneğin kurbanın zararlı bir C2 IP'sine istek yapması: Victim-to-Infrastructure).|
|**Methodology**|Saldırının genel sınıflandırması ve icra biçimidir.|Phishing, DDoS, Port Scan, Data Breach, Drive-by Download.|Olayın hangi saldırı vektörü üzerinden gerçekleştiğini hızlıca kategorize eder.|
|**Resources**|Başarıya ulaşmak için ihtiyaç duyulan dış kaynaklardır.|Yazılım (OS, Metasploit), Bilgi (şifreler), Donanım, Fon/Para (domain alımı), Tesisler (elektrik, barınma), Ağ Erişimi (ISP).|Tehdit aktörünün teknik ve lojistik imkanlarını anlamayı sağlar.|

## 4. Sosyo-Politik ve Teknoloji Eksenleri

### 4.1 Social-Political Component (Sosyo-Politik Bileşen)

- **Mantık:** Saldırganın **ihtiyacını, motivasyonunu ve niyetini** açıklar.
    
      
    
- **Güdüler:** Finansal kazanç, yeraltı hacker topluluğunda itibar/kabul görme, hacktivizm, devlet destekli casusluk (espionage) veya sabotaj.
    
      
    
- **Tüketim İlişkisi Örneği:** Kurban, saldırgana farkında olmadan bir "ürün" (örneğin botnet'e katılmış zombi bir bilgisayarın işlemci gücü ve bant genişliği) sunar; saldırgan ise bu ürünü kripto madenciliği veya DDoS için tüketerek finansal kazanç sağlar.
    
      
    

### 4.2 Technology Component (Teknoloji Bileşeni)

- **Mantık:** İki çekirdek özellik olan **Capability (Yetenek)** ile **Infrastructure (Altyapı)** arasındaki ilişkiyi inceler. Saldırganın operasyonunu nasıl yürüttüğünü ve nasıl iletişim kurduğunu teknik olarak aydınlatır.
    
      
    
- **Senaryo Örneği (Watering-Hole Saldırısı):** Saldırgan, hedef aldığı kurbanların düzenli ziyaret ettiğini bildiği meşru bir web sitesini (altyapı) hackleyerek buraya bir exploit kiti/zararlı kod (yetenek) enjekte eder.
    
      
    

## 5. Kritik Analiz Notları ve Uzman İpuçları

> [!WARNING]
> 
> **Adversary Bilgisinin Eksikliği Tuzağı:** Bir ihlal analizi yaparken elmas modelin "Adversary" düğümünün boş olması sürecin tıkandığı anlamına gelmez. Gerçek dünyadaki SOC vakalarında bu alan ilk müdahale anında genellikle boştur. Altyapı (Type 2 logları), yetenek (kullanılan malware hash'i) ve kurban analiz edildikçe saldırgan düğümü doldurulur.
> 
>   

> [!TIP]
> 
> **Axiom 4 ve Tehdit Avcılığı (Hunting):** Diamond Model Axiom 4 gereği bir saldırı tekil bir olay değildir; bir zincirdir. Eğer elinizde sadece "Installation" (Zararlı Kurulumu) aşamasına ait bir bulgu varsa, geriye dönük korelasyonla "Delivery" (Teslimat/Phishing) ve ileriye dönük olarak "C2" iletişimini aramanız gerekir.
> 
>   

> [!EXAMPLE]
> 
> **Direction Belirleme Mantığı:**
> 
> Bir kullanıcı oltalama bağlantısına tıklayıp sahte bir oturum açma sayfasına gittiğinde trafik akışı **Victim-to-Infrastructure** olarak etiketlenir. Eğer saldırgan ele geçirdiği bir Type 1 sunucu üzerinden kurbanın dışa açık portuna exploit paketi yolluyorsa yön **Infrastructure-to-Victim** olur.
> 
>   

## 6. Soru-Cevap Anahtarı (Lab & Teori)

|**Soru No**|**İlgili Konu / Alan**|**Soru Metni**|**Doğru Cevap**|
|---|---|---|---|
|**Q1**|Giriş|Read the above.|_No answer needed_|
|**Q2**|Adversary|What is the term for a person/group that has the intention to perform malicious actions against cyber resources?|**Adversary Operator**|
|**Q3**|Adversary|What is the term of the person or a group that will receive the benefits from the cyberattacks?|**Adversary Customer**|
|**Q4**|Victim|What is the term that applies to the Diamond Model for organizations or people that are being targeted?|**Victim Personae**|
|**Q5**|Capability|Provide the term for the set of tools or capabilities that belong to an adversary.|**Adversary Arsenal**|
|**Q6**|Infrastructure|To which type of infrastructure do malicious domains and compromised email accounts belong?|**Type 2 Infrastructure**|
|**Q7**|Infrastructure|What type of infrastructure is most likely owned by an adversary?|**Type 1 Infrastructure**|
|**Q8**|Meta-Features|What meta-feature does the axiom "Every malicious activity contains two or more phases which must be successfully executed in succession to achieve the desired result" belong to?|**Phase**|
|**Q9**|Meta-Features|You can label the event results as "success", "failure", and "unknown". What meta-feature is this related to?|**Result**|
|**Q10**|Meta-Features|To what meta-feature is this phrase applicable "Every intrusion event requires one or more external resources to be satisfied prior to success"?|**Resources**|
|**Q11**|Social-Political|Read the above.|_No answer needed_|
|**Q12**|Technology|Read the above.|_No answer needed_|
|**Q13**|Pratik Uygulama|Ensure you have deployed the static site attached to this task... Complete all eight areas of the diamond. What is the flag that is displayed to you?|**THM{DIAMOND_MODEL_ATTACK_CHAIN}**|
|**Q14**|Kapanış|Read the above.|_No answer needed_|

## 7. Tekrar Kategorisi ve 3 Dakikalık Pazar Stratejisi

- **Sınıf:** 🟡 **Sarı Sınıf (Kavramsal / 3 Dk Soru-Cevap Odaklı)**
    
      
    
- **Gerekçe:** Oda herhangi bir CLI/terminal komutu veya exploit payload'u içermemektedir; doğrudan analitik zihniyeti, CTI terminolojisini ve kavramsal ayrımları test eden teorik bir çerçeve sunar.
    
      
    

### 🗓️ Pazar Günü 3 Dakikalık Tekrar Stratejisi

1. **1. Dakika (Çekirdek Düğümler ve Ayrım Testi):** Gözlerini kapat ve şu 3 farkı zihninden açıkla:
    
      
    - _Adversary Operator_ (eylemi yapan) vs. _Adversary Customer_ (faydayı sağlayan).
        
          
        
    - _Victim Personae_ (hedef şahıs/kurum) vs. _Victim Assets_ (hedef sistem/IP/e-posta).
        
          
        
    - _Type 1 Infrastructure_ (saldırganın kendi sistemi) vs. _Type 2 Infrastructure_ (aracı / hacklenmiş sistem).
        
          
        
2. **2. Dakika (Meta Özellikler & Aksiyom):**
    
      
    - "Axiom 4" nedir? (_Saldırı tekil değil, ardışık en az iki evreden oluşur -> Phase meta özelliği_).
        
          
        
    - 7 adet "Direction" değerini hızlıca gözden geçir (_Victim-to-Infra, Infra-to-Victim vb._).
        
          
        
3. **3. Dakika (Eksenler):**
    
      
    - Dikey ve yatay köprüleri hatırla: _Social-Political_ (Adversary-Victim niyet bağı) ve _Technology_ (Capability-Infrastructure teknik iletişim bağı).
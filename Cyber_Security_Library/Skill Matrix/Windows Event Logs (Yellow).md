# 📑 Windows Event Logs (Windows Olay Günlükleri) 

**Etiketler:** `#SOC` `#BlueTeam` `#DFIR` `#WindowsEventLogs` `#EventViewer` `#Wevtutil` `#GetWinEvent` `#XPath` `#PowerShell` `#TryHackMe`

## 1. Mimari Yapı ve Windows Günlükleme Çerçevesi

Windows Olay Günlükleri (Windows Event Logs), işletim sistemi bileşenlerinin, donanım sürücülerinin, güvenlik denetimlerinin ve yüklü servis/uygulamaların aktivitelerini kronolojik bir denetim izi (**audit trail**) halinde saklayan tescilli ikili (**proprietary binary**) bir veri tabanı yapısıdır.

  

- **Fiziksel Konum:** Windows Vista ve sonrasında olay günlükleri varsayılan olarak `C:\Windows\System32\winevt\Logs` dizininde `.evtx` (eski sürümlerde `.evt`) uzantısıyla tutulur.
    
      
    
- **Biçim ve Erişim:** Düz metin (TXT) değildir; doğrudan Notepad ile okunamaz. Windows API aracılığıyla çalışma anında XML formatına dönüştürülerek kullanıcıya ve analiz araçlarına sunulur.
    
      
    
- **Kurumsal Mimarideki Yeri:** Büyük ölçekli kurumsal ağlarda analistlerin binlerce uç noktaya tek tek RDP/PowerShell ile bağlanması operasyonel olarak mümkün değildir. Bu noktada uç noktalara kurulan **Log Toplayıcılar / Ajanlar (Splunk Universal Forwarder, Elastic Winlogbeat, WEF - Windows Event Forwarding)** devreye girerek bu ikili `.evtx` akışını merkezi SIEM platformlarına iletir.
    
      
    

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        WINDOWS GÜNLÜKLEME VE ERİŞİM MİMARİSİ                          │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 1. KLASİK WINDOWS LOG KANALLARI (Standard Channels):                                   │
│    • System (Sistem): Donanım, çekirdek, sürücü ve işletim sistemi çökmeleri/durumları │
│    • Security (Güvenlik): Kimlik doğrulama, oturum açma (Logon/Logoff), yetki değişimleri│
│    • Application (Uygulama): Yüklü yazılımların çalışma hataları, uyarıları ve logları │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 2. UYGULAMA VE SERVİS LOGLARI (Applications and Services Logs):                        │
│    • Microsoft-Windows-PowerShell/Operational (Betik çalıştırma, pipeline, scriptblock)│
│    • Microsoft-Windows-Sysmon/Operational (Gelişmiş uç nokta süreç & ağ telemetrisi)   │
│    • Directory Service, DNS Server, File Replication Service vb.                       │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 3. ANALİSTİN ERİŞİM VE SORGULAMA ARAÇLARI:                                             │
│    • GUI Analizi: Event Viewer (eventvwr.msc)                                          │
│    • Komut Satırı: wevtutil.exe (Sorgulama, dışa aktarma, temizleme)                   │
│    • PowerShell Otomasyonu: Get-WinEvent (-FilterHashtable, -FilterXPath)              │
│    • Yüksek Hızlı Adli Ayrıştırma (EZTools): EvtxECmd.exe ──► Timeline Explorer       │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

## 2. Günlük Sınıflandırmaları, Olay Seviyeleri ve Triaj Metodolojisi

### A. Olay Seviyeleri (Event Levels)

Windows sisteminde kaydedilen her olay önem derecesine göre bir seviye (Level) değeri alır:

  

|**Seviye (Level) Adı**|**Sayısal Değer (ID)**|**Açıklama ve Analist Önemi**|
|---|---|---|
|**Critical (Kritik)**|`1`|Sistemin hemen kurtarılması gereken, kritik bir bileşenin çökmesine yol açan ciddi hata.|
|**Error (Hata)**|`2`|Bir işlevin veya servisin çalışmasını engelleyen önemli aksaklık.|
|**Warning (Uyarı)**|`3`|Hemen bir çöküş yaratmayan ancak gelecekte sorun oluşturabilecek riskli durum.|
|**Information (Bilgi)**|`4`|Bir görevin, uygulamanın veya servisin başarıyla çalıştığını belirten rutin operasyonel kayıt.|
|**Verbose (Ayrıntılı)**|`5`|Hata ayıklama (debugging) amacıyla üretilen en ince detay telemetrisi.|

### B. Analist Triaj Metodolojisi ve Tuzaklar

> [!WARNING]
> 
> **Event ID Benzersiz (Unique) Değildir!**
> 
>   
> 
> En büyük analist tuzaklarından biri bir Event ID'nin tüm sistemde aynı anlama geldiğini varsaymaktır.
> 
>   
> 
> - Örnek: `Event ID 100`, **WLMS** sağlayıcısında rutin bir lisans/sistem bildirimi iken; başka bir uygulamada kritik bir çökme anlamına gelebilir.
>     
>       
>     
> - Bir olay kodu her zaman **ProviderName (Sağlayıcı)** ve **LogName (Kanal)** bağlamı ile birlikte değerlendirilmelidir.
>     
>       
>     

> [!TIP]
> 
> **Günlük Temizleme (Log Clearing) ve Saldırgan Ayak İzi:**
> 
>   
> 
> Saldırganlar bir sistemi ele geçirdikten sonra arkalarında iz bırakmamak için `wevtutil cl <log>` komutuyla veya Event Viewer arayüzündeki _Clear Log_ butonuyla günlükleri silmeye çalışırlar. Ancak Windows bu eylemin kendisini de güvenlik günlüğüne kaydeder:
> 
>   
> 
> - **Event ID 1102:** Security (Güvenlik) günlüğü temizlendiğinde tetiklenir (Audit Log Cleared).
>     
>       
>     
> - **Event ID 104:** System günlüğü temizlendiğinde tetiklenir.
>     
>       
>     
>     Bu iki olay kodunun SIEM'de tetiklenmesi doğrudan bir **Kritik İhlal Göstergesi (IOC)** olarak ele alınmalıdır.
>     
>       
>     

## 3. İzlenmesi Gereken Kritik Olay Kodları (Monitoring & Threat Hunting)

|**Kategori**|**Olay Kodu (Event ID)**|**Günlük Kanalı**|**Anlamı ve Tehdit Avı (Threat Hunting) Değeri**|
|---|---|---|---|
|**Kimlik Doğrulama**|`4624`|Security|Başarılı Oturum Açma (Logon). `LogonType: 10` RDP'yi, `LogonType: 3` ağ oturumunu gösterir.|
|**Kimlik Doğrulama**|`4625`|Security|Başarısız Oturum Açma (Failed Logon). Yüksek sayıda gelmesi kaba kuvvet (Brute-Force) atağıdır.|
|**Süreç Takibi**|`4688`|Security|Yeni Süreç Oluşturuldu (Process Creation). _Audit Process Creation_ politikasıyla komut satırı açılmalıdır.|
|**Hesap Yönetimi**|`4720`|Security|Yeni Kullanıcı Oluşturuldu (Arka kapı / Backdoor hesap tespiti).|
|**Grup Yönetimi**|`4732`|Security|Yerel Güvenlik Grubuna Üye Eklendi (Örn: `Administrators` grubuna saldırgan ekleme).|
|**Grup Sayımı**|`4799`|Security|Güvenlik Etkin Yerel Grup Üyeleri Numaralandırıldı/Sorgulandı (`net localgroup administrators` vb.).|
|**Parola Sıfırlama**|`4724`|Security|Bir hesabın parolasını sıfırlama girişimi yapıldı.|
|**Servis Kurulumu**|`7045`|System|Sisteme yeni bir servis yüklendi (Kalıcılık / Persistence veya PsExec kullanımı).|
|**Günlük Silme**|`1102`|Security|Denetim günlüğü kasten temizlendi (Anti-forensics).|
|**PowerShell**|`400`|PowerShell|PowerShell motor durumu başladı. Eski sürüme çekme (**Downgrade Attack**) tespitinde kullanılır.|
|**PowerShell**|`800`|PowerShell|Pipeline Execution Details (Boru hattı yürütme detayları).|
|**PowerShell**|`4103`|PowerShell/Oper.|Module Logging: Yürütülen modül ve boru hattı detayları.|
|**PowerShell**|`4104`|PowerShell/Oper.|Script Block Logging: Çalıştırılan betiklerin **açık metin** tam kod blokları (Emotet, zararlı payload tespiti).|

## 4. Tüm Komutlar, Parametreler ve Sorgulama Sentaksı

### A. wevtutil.exe (Windows Events Command Line Utility)

Windows'un yerleşik komut satırı aracıdır. Otomasyonlarda ve uzaktan yönetimde kullanılır.

  

DOS

```
:: 1. Sistemde kayıtlı olan tüm log adlarını listeleme
wevtutil.exe enum-logs
:: Kısayol: wevtutil el

:: 2. Application günlüğünden en son oluşan 3 olayı düz metin olarak okuma
wevtutil.exe qe Application /c:3 /rd:true /f:text

:: 3. Application günlüğünde Event ID 100 olan olayları XPath ile sorgulama (1 adet getir)
wevtutil.exe qe Application /q:*/System[EventID=100] /f:text /c:1

:: 4. Belirli bir günlüğü tamamen temizleme (Saldırganların iz silme tekniği)
wevtutil.exe cl Application
```

#### wevtutil Parametre Detayları:

- `qe` (`query-events`): Belirtilen log dosyasından veya sağlayıcıdan olayları sorgular.
    
      
    
- `/c:<Sayı>`: Okunacak maksimum olay adedini belirler (`/c:3`).
    
      
    
- `/rd:<true|false>`: Okuma yönünü belirler. `true` en yeni olaydan geriye doğru okur (`read direction`).
    
      
    
- `/f:<format>`: Çıktı biçimi (`text`, `xml`).
    
      
    
- `/q:<sorgu>`: Olayları filtrelemek için kullanılan XPath sorgusudur.
    
      
    
- `/lf:<true|false>`: Sorgulanacak hedefin canlı bir kanal değil, diskteki harici bir log dosyası (`log file`) olduğunu belirtir.
    
      
    

### B. Get-WinEvent ve FilterHashtable

Büyük log kümelerinde `Where-Object` kullanmak belleği tüketir ve yavaştır. PowerShell'in önerdiği en hızlı yöntem **FilterHashtable** kullanmaktır.

  

PowerShell

```
# 1. Bilgisayardaki tüm log kanallarını ve kayıt sayılarını listeleme
Get-WinEvent -ListLog *

# 2. Belirli bir sağlayıcıya (Provider) ait log sağlayıcı bilgilerini getirme
Get-WinEvent -ListProvider *PowerShell*

# 3. FilterHashtable ile hızlı filtreleme (LogName, ProviderName ve Seviye)
Get-WinEvent -FilterHashtable @{
    LogName = 'Application'
    ProviderName = 'WLMS'
    Level = 4   # 4 = Informational
}

# 4. PowerShell Script Block (EID 4104) içinde parola/anahtar avı
Get-WinEvent -FilterHashtable @{LogName='Microsoft-Windows-PowerShell/Operational'; ID=4104} | 
    Select-Object -Property Message | 
    Select-String -Pattern 'SecureString'
```

#### FilterHashtable Sözdizimi Kuralları:

- `@` işaretiyle başlar ve süslü parantez `{}` ile kapanır.
    
      
    
- Anahtar-değer çiftleri eşittir `=` ile ayrılır (`LogName='Security'`).
    
      
    
- Birden çok anahtar ya noktalı virgül `;` ile ya da her biri ayrı satıra yazılarak tanımlanır.
    
      
    

### C. XPath (XML Path Language) ile Gelişmiş Filtreleme

Windows olay günlüğü XML şeması iki ana bloktan oluşur: `<System>` ve `<EventData>`.

  

PowerShell

```
# 1. Canlı logda Provider adı ve tam oluşturulma zamanına (SystemTime) göre filtreleme
Get-WinEvent -LogName Application -FilterXPath '*/System/Provider[@Name="WLMS"] and */System/TimeCreated[@SystemTime="2020-12-15T01:09:08.940277500Z"]'

# 2. Harici bir .evtx dosyasında (-Path) EventData içindeki TargetUserName ve Event ID filtreleme
Get-WinEvent -Path 'C:\Users\Administrator\Desktop\merged.evtx' -FilterXPath '*/EventData/Data[@Name="TargetUserName"]="Sam" and */System/EventID=4720'

# 3. İki farklı koşulu birleştirerek sorgulama
Get-WinEvent -LogName Application -FilterXPath '*/System/EventID=101 and */System/Provider[@Name="WLMS"]'
```

#### XPath Şema Gezintisi Kuralları:

- Kök dizin `*` veya `Event` ile başlar.
    
      
    
- `<System>` altındaki doğrudan etiketler slash ile bağlanır: `*/System/EventID=100`.
    
      
    
- Etiket içi özellikler (attributes) köşeli parantez içinde `@` ile belirtilir: `Provider[@Name="WLMS"]`.
    
      
    
- `<EventData>` içindeki değişken veriler `Data[@Name="DegiskenAdi"]="Deger"` biçiminde filtrelenir.
    
      
    

## 5. Pratik Laboratuvar ve Görev Soru-Cevap Anahtarı

|**Görev #**|**Soru**|**Doğru Cevap**|**Analiz Metodolojisi ve Kullanılan Komut**|
|---|---|---|---|
|**T2.Q1**|What is the Event ID for the earliest recorded event? (Exclude the warning event)|`4103`|`merged.evtx` açılıp tarihe göre en eskiye sıralandığında görülen ilk bilgi seviyesindeki olaydır.|
|**T2.Q2**|Filter on Event ID 4104. What was the 2nd command executed in the PowerShell session?|`whoami`|EID 4104 filtrelenip zaman sırasına dizildiğinde yürütülen 2. komuttur.|
|**T2.Q3**|What is the Task Category for Event ID 4104?|`Execute a Remote Command`|Event Viewer arayüzünde EID 4104 kaydının Görev Kategorisi sütunudur.|
|**T2.Q4**|Within the opened merged.evtx saved log, what is the Task Category for Event ID 800?|`Pipeline Execution Details`|Olay detaylarında EID 800 için belirtilen Görev Kategorisi değeridir.|
|**T3.Q1**|How many log names are in the machine?|`1071`|`(wevtutil el).count` komutuyla sistemde kayıtlı toplam log kanalı adedidir.|
|**T3.Q2**|What event files would be read when using the query-events command?|`event log, log file, structured query`|`wevtutil qe /?` yardım çıktısında belirtilen desteklenen okuma kaynaklarıdır.|
|**T3.Q3**|What option would you use to provide a path to a log file?|`/lf:true`|wevtutil sorgusunda kaynak olarak harici dosya belirtme anahtarıdır.|
|**T3.Q4**|What is the VALUE for /q?|`Xpath query`|wevtutil yardım menüsünde `/q` seçeneğinin aldığı değer türüdür.|
|**T3.Q5**|What is the log name? (wevtutil qe Application /c:3 /rd:true /f:text)|`Application`|Sorgu komutunda hedeflenen günlüğün adıdır.|
|**T3.Q6**|What is the /rd option for?|`Event read direction`|Olayların hangi yönde (en yeni/en eski) okunacağını belirleyen parametredir.|
|**T3.Q7**|What is the /c option for?|`Maximum number of events to read`|Çıktılanacak azami kayıt adedini belirleyen bayraktır.|
|**T4.Q1**|Execute the command from Example 1 (as is). What are the names of the logs related to OpenSSH?|`OpenSSH/Admin,OpenSSH/Operational`|`Get-WinEvent -ListLog *OpenSSH*` komutu ile listelenen günlüklerdir.|
|**T4.Q2**|Execute Example 8 with _PowerShell_. What is the name of the 3rd log provider?|`Microsoft-Windows-PowerShell-DesiredStateConfiguration-FileDownloadManager`|`Get-WinEvent -ListProvider *PowerShell*` çıktısındaki 3. sağlayıcı adıdır.|
|**T4.Q3**|Use Microsoft-Windows-PowerShell as the log provider. How many event ids are displayed?|`192`|İlgili sağlayıcıya ait olay kodlarının toplam adedidir.|
|**T4.Q4**|How do you specify the number of events to display?|`-MaxEvents`|`Get-WinEvent` komutunda getirilecek kayıt sayısını kısıtlama parametresidir.|
|**T4.Q5**|When using the FilterHashtable parameter and filtering by level, what is the value for Informational?|`4`|Microsoft dokümantasyonunda `Level` anahtarında "Bilgi" seviyesinin sayısal değeridir.|
|**T5.Q1**|Query to find WLMS events with System Time of 2020-12-15T01:09:08.940277500Z?|`Get-WinEvent -LogName Application -FilterXPath '*/System/Provider[@Name="WLMS"] and */System/TimeCreated[@SystemTime="2020-12-15T01:09:08.940277500Z"]'`|Provider ve SystemTime etiketlerini birleştiren tam XPath sorgusudur.|
|**T5.Q2**|Query against merged.evtx to find user Sam with Event ID 4720?|`Get-WinEvent -Path 'C:\Users\Administrator\Desktop\merged.evtx' -FilterXPath '*/EventData/Data[@Name="TargetUserName"]="Sam" and */System/EventID=4720'`|Harici dosyadan `-Path` ile `TargetUserName` ve `EventID` süzen XPath komutudur.|
|**T5.Q3**|Based on the previous query, how many results are returned?|`2`|Önceki sorgunun sonucunda dönen toplam nesne sayısıdır.|
|**T5.Q4**|Based on the output from question #2, what is Message?|`A user account was created`|EID 4720 olayının varsayılan Windows açıklama mesajıdır.|
|**T5.Q5**|Still working with Sam as user, what time was Event ID 4724 recorded?|`12/17/2020 1:57:14 PM`|Parola sıfırlama (4724) olayının zaman damgasıdır.|
|**T5.Q6**|What is the Provider Name?|`Microsoft-Windows-Security-Auditing`|Windows Güvenlik günlüğünün standart sağlayıcı (provider) adıdır.|
|**T7.Q1**|What event ID is to detect a PowerShell downgrade attack?|`400`|PowerShell v2 gibi eski bir motora geçildiğinde üretilen `Engine state is changed to Available` olayıdır.|
|**T7.Q2**|What is the Date and Time this attack took place?|`12/18/2020 7:50:33 AM`|EID 400 kaydının zaman damgasıdır.|
|**T7.Q3**|A Log clear event was recorded. What is the 'Event Record ID'?|`27736`|Günlük silme olayının (EID 1102 / 104) `Record ID` değeridir.|
|**T7.Q4**|What is the name of the computer?|`PC01.example.corp`|Günlük temizleme olayının gerçekleştiği bilgisayar adıdır.|
|**T7.Q5**|What is the name of the first variable within the PowerShell command? (Emotet)|`$Va5w3n8`|EID 4104 ScriptBlock metninde gizlenmiş (obfuscated) betiğin ilk değişken adıdır.|
|**T7.Q6**|What is the Date and Time this attack took place?|`8/25/2020 10:09:28 PM`|Emotet betiğinin çalıştırıldığı EID 4104 kaydının zaman damgasıdır.|
|**T7.Q7**|What is the Execution Process ID?|`6620`|Bu zararlı betiği yürüten sürecin kimlik numarasıdır (PID).|
|**T7.Q8**|What is the Group Security ID of the group she enumerated?|`S-1-5-32-544`|Yerel Administrators grubunun iyi bilinen SID (Well-Known SID) değeridir.|
|**T7.Q9**|What is the event ID? (Group enumeration)|`4799`|Güvenlik etkin yerel grup üyeliğinin numaralandırıldığını gösteren olay kodudur.|

## 6. Tekrar Kategorisi ve Pazar Stratejisi

- **Kategori Sınıfı:** 🟡 **Sarı (Kavramsal Triaj & Kritik Event ID Eşleştirmesi / 3 Dakika)**
    
      
    
- **Gerekçe:** Konsoldaki XPath ve FilterHashtable yazım hamallığı günlük işte nadiren elle yazılsa da; mülakatlarda `4624`, `4688`, `4720`, `4799`, `1102`, `4104` olay kodlarının anlamları ve log temizleme göstergeleri doğrudan sorulur.
    
      
    
- **Pazar Günü 3 Dakikalık Tekrar Stratejisi:**
    
      
    1. **1. Dakika:** Olay Seviyelerini ve Log Yollarını hatırla:
        
          
        
        _Loglar `C:\Windows\System32\winevt\Logs` dizininde ikili `.evtx` formatında saklanır. Seviyeler: 1=Critical, 2=Error, 3=Warning, 4=Information._
        
          
        
    2. **2. Dakika:** Kritik Olay Kodlarını hızla say:
        
          
        - `4720`: Yeni kullanıcı açıldı.
            
              
            
        - `4732`: Gruba eklendi.
            
              
            
        - `4799`: Grup üyeleri numaralandırıldı (`net localgroup`).
            
              
            
        - `1102`: Güvenlik günlüğü silindi (**Kritik İhlal**).
            
              
            
        - `4104`: PowerShell Script Block metni (Zararlı kod çözme).
            
              
            
    3. **3. Dakika:** PowerShell Downgrade saldırısını açıkla:
        
          
        
        _Saldırgan modern denetimleri ve Script Block Logging'i atlatmak için PowerShell sürüm 2'yi başlatır; bu durum sistemde **Event ID 400** olarak iz bırakır._
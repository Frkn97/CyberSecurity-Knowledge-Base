# 🐧 Linux Logging for SOC Dökümantasyonu

**Etiketler:** `#SOC` `#Linux` `#Logging` `#Syslog` `#Auditd` `#DetectionEngineering` `#TryHackMe` `#DFIR`

## 1. Mimari Yapı ve Linux Loglama Felsefesi

Linux tabanlı sistemler siber saldırılara karşı bağışık değildir; aksine bulut konteynerleri ve kurumsal sunucularda saldırganların birincil hedefidir. Windows ortamındaki hiyerarşik ve ikili (binary) formattaki EVTX yapısının aksine, Linux loglama mekanizması temel Unix felsefesine dayanır: **"Her şey bir dosyadır ve loglar düz metindir (plain text)."**

  

```
┌────────────────────────────────────────────────────────┐
│                      KULLANICI ALANI (User Space)      │
│  Uygulamalar / Servisler / Saldırgan Scriptleri        │
└───────────────────────────┬────────────────────────────┘
                            │ Sistem Çağrısı (Syscall: execve, openat)
                            ▼
┌────────────────────────────────────────────────────────┐
│                     ÇEKİRDEK ALANI (Kernel Space)      │
│  LSM (Yama, AppArmor) ──► Syscall Interceptor          │
└─────────────┬─────────────────────────────┬────────────┘
              │                             │
              ▼ (/dev/log)                  ▼ (Netlink Socket)
      ┌───────────────┐             ┌───────────────┐
      │ rsyslogd /    │             │    auditd     │
      │ systemd-      │             │ (Audit Daemon)│
      │ journald      │             └───────┬───────┘
      └───────┬───────┘                     │
              ▼                             ▼
     /var/log/syslog              /var/log/audit/audit.log
     /var/log/auth.log            (ausearch ile taranır)
```

### Temel Linux Dağıtım Farklılıkları

- **Debian / Ubuntu:** Kimlik doğrulama logları `/var/log/auth.log` altında, genel sistem logları `/var/log/syslog` altında toplanır. Paket yönetimi logları `/var/log/dpkg.log` veya `/var/log/apt/` dizinindedir.
    
      
    
- **RHEL / CentOS / Fedora:** Kimlik doğrulama logları `/var/log/secure` altında, genel sistem logları `/var/log/messages` altında tutulur. Paket yönetimi `/var/log/dnf.log` veya `/var/log/yum.log` dosyasına yazılır.
    
      
    

## 2. Temel Log Kaynakları ve Triage Metodolojisi

### A. Genel Sistem Logları (`/var/log/syslog` veya `/var/log/messages`)

Sistem servislerinin başlangıç, hata ve zaman senkronizasyonu kayıtlarını içerir.

  

> [!NOTE]
> 
> **Zaman Senkronizasyonu (NTP) Neden Kritiktir?**
> 
>   
> 
> Çoklu sistemleri kapsayan olay müdahalesi (Incident Response) senaryolarında, makineler arasındaki saat farkı zaman çizelgesi (timeline) korelasyonunu tamamen bozar. Sistem loglarında NTP sunucusu denetlenerek saldırganın saati manipüle edip etmediği veya sistemin güvenilir bir zaman kaynağıyla konuşup konuşmadığı (`ntp.ubuntu.com`) doğrulanmalıdır.
> 
>   

> [!INFO]
> 
> **Yama LSM (Linux Security Module):**
> 
>   
> 
> Çekirdek seviyesinde çalışan bir güvenlik modülüdür. Süreçlerin bellek alanlarının başka süreçler tarafından okunmasını veya manipüle edilmesini (`ptrace` kısıtlamaları) engeller. Çekirdek logunda görülen `Yama: becoming mindful.` ibaresi, çekirdeğin belleği koruma kalkanını devreye aldığını gösterir.
> 
>   

### B. Kimlik Doğrulama Logları (`/var/log/auth.log`)

SOC analistinin en sık ziyaret ettiği dosyadır. Şu üç temel aktiviteyi içerir:

  

1. **Oturum Yönetimi:** `pam_unix` aracılığıyla açılan/kapatılan yerel (`login:session`), uzaktan (`sshd:session`), SMB (`samba:session`) ve zamanlanmış görev (`cron:session`) oturumları.
    
      
    
2. **Kaba Kuvvet (Brute-Force) Belirtileri:** `sshd` servisi üzerinden gelen ardışık `Failed password` kayıtları.
    
      
    
3. **Kalıcılık (Persistence) ve Yetki Yükseltme:** `useradd`, `userdel`, `usermod` ve `sudo` ile yürütülen komutlar.
    
      
    

### C. Uygulama Logları (Örn: Nginx `/var/log/nginx/access.log`)

Her satır sunucuya yapılan bir web isteğini temsil eder. İstemci IP adresi, zaman damgası, HTTP metodu, istenen URL, HTTP durum kodu (200, 302, 403 vb.) ve aktarılan bayt miktarını içerir.

  

### D. Shell Geçmişi (`~/.bash_history`)

Kullanıcıların terminalde çalıştırdığı komutları kaydeder. Bellekte tutulur ve oturum kapandığında ilgili kullanıcının ev dizinindeki `.bash_history` dosyasına yazılır.

  

> [!WARNING]
> 
> **Bash History'nin SOC Analizindeki Kör Noktaları:**
> 
>   
> 
> Bir saldırgan Bash geçmişini kolayca atlatabilir:
> 
>   
> 
> 1. Komutun başına **boşluk karakteri** koyarak (`HISTCONTROL=ignorespace` aktifse kaydedilmez).
>     
>       
>     
> 2. Komutları bir betik dosyasına yazıp çalıştırarak (`./malicious.sh`).
>     
>       
>     
> 3. Geçmiş tutmayan başka bir kabuğa geçerek (`/bin/sh`).
>     
>       
>     
> 4. Servisler, arka plan görevleri ve web kabukları (web shell) etkileşimli olmadığı için Bash geçmişine hiçbir kayıt bırakmaz.
>     
>       
>     

## 3. Çalışma Zamanı İzleme (Runtime Monitoring) ve Sistem Çağrıları (Syscalls)

Linux varsayılan olarak süreç oluşturma, dosya değişiklikleri veya ağ soketi açma gibi olayları loglamaz. Bir dosyanın kim tarafından silindiğini veya hangi ikili dosyanın çalıştırıldığını görmek için **Sistem Çağrısı (Syscall)** seviyesinde izleme yapılması zorunludur.

  

> [!IMPORTANT]
> 
> **Sistem Çağrıları Atlana Bilir mi?**
> 
>   
> 
> **HAYIR (Nay).** Bir kullanıcı alanı (user-space) programının çekirdekten bağımsız olarak donanıma erişmesi, dosya açması veya süreç başlatması mümkün değildir. Standart bir işletim sisteminde tüm işlemler `execve` (program çalıştırma), `openat` (dosya açma), `connect` (ağ bağlantısı) gibi sistem çağrıları üzerinden çekirdeğe (kernel) iletilmek zorundadır. EDR ve audit mekanizmaları bu çağrıları izler.
> 
>   

### Auditd (Linux Audit Daemon)

Auditd, çekirdek seviyesindeki sistem çağrılarını izleyen ve yapılandırılan kurallara göre `/var/log/audit/audit.log` dosyasına ham telemetri basan denetim mekanizmasıdır. Kurallar `/etc/audit/rules.d/` dizini altında tanımlanır.

  

#### Auditd Log Kayıt Formatı Alanları:

- `type=PROCTITLE`: Çalıştırılan sürecin tam komut satırı argümanları.
    
      
    
- `type=CWD`: Sürecin tetiklendiği çalışma dizini (Current Working Directory).
    
      
    
- `type=EXECVE`: Sürece aktarılan argüman dizilimi (`argc`, `a0`, `a1`).
    
      
    
- `type=SYSCALL`: Sistem çağrısının çekirdek düzeyindeki teknik detayları:
    
      
    - `pid` / `ppid`: Süreç ID'si ve Ebeveyn Süreç ID'si (Süreç ağacı oluşturmak için kritiktir).
        
          
        
    - `auid` (Audit UID): Sisteme ilk giriş yapılan asıl kullanıcı kimliği (Örn: `ubuntu`).
        
          
        
    - `uid`: Komutu o an fiilen çalıştıran kimlik (Örn: `sudo` çekildiyse `root`).
        
          
        
    - `tty`: Oturum tanımlayıcısı (`pts/0`, `pts/1`).
        
          
        
    - `exe`: Yürütülen ikili dosyanın mutlak disk yolu (Örn: `/usr/bin/wget`).
        
          
        
    - `key`: Kural yazarı tarafından olayı filtrelemek için atanan özel etiket (Örn: `proc_wget`).
        
          
        

## 4. Kullanılan Araçlar ve Alternatif Teknolojiler

|**Araç / Teknoloji**|**Açıklama ve SOC Kullanım Amacı**|**Dokümantasyon / Bağlantı**|
|---|---|---|
|**Auditd**|Linux yerleşik çekirdek denetim mekanizması. Sistem çağrılarını ve dosya erişimlerini izler.|[Auditd Man Page](https://man7.org/linux/man-pages/man8/auditd.8.html)|
|**Sysmon for Linux**|Microsoft Sysinternals aracının Linux sürümü. XML tabanlı kural setiyle olayları toplar.|[Sysmon for Linux GitHub](https://github.com/Sysinternals/SysmonForLinux)|
|**Falco**|Cloud-native ve konteyner (Kubernetes) ortamları için eBPF tabanlı çalışma zamanı güvenlik aracı.|[Falco Official Site](https://falco.org/)|
|**Osquery**|İşletim sistemini ilişkisel bir veritabanı gibi sorgulamayı sağlayan framework.|[Osquery Official Site](https://osquery.io/)|
|**EDR Çözümleri**|Çekirdek olaylarını toplayan ticari uç nokta tespit ve müdahale yazılımları.|—|

## 5. Tüm Terminal Komutları ve Parametre Referansı

### A. Düz Metin Log Analizi Komutları

Bash

```
# Syslog dosyasının ilk satırlarını okuma
cat /var/log/syslog | head

# Syslog içinden belirli bir anahtar kelimeyi (CRON) süzme
cat /var/log/syslog | grep CRON

# Belirli bir kelimeyi hariç tutarak filtreleme (-v parametresi)
cat /var/log/syslog | grep -v CRON

# /var/log altındaki tüm log dosyalarını detaylı listeleme
ls -l /var/log

# /var/log dizinindeki tüm dosyalarda rekürsif ve regex araması yapma
# -R: Alt dizinlere doğru özyinelemeli (recursive) arama
# -E: Genişletilmiş düzenli ifadeleri (Extended Regex) destekleme
grep -R -E "auth|login|session" /var/log

# Sistem zaman senkronizasyonu için NTP kayıtlarını büyük/küçük harf duyarsız arama (-i)
grep -i ntp /var/log/syslog

# Çekirdek loglarında Yama Güvenlik Modülü mesajını arama
grep -i Yama /var/log/syslog
```

### B. Kimlik Doğrulama Log Analiz Komutları

Bash

```
# Başarılı oturum açma ve kapatma olaylarını filtreleme
cat /var/log/auth.log | grep -E 'session opened|session closed'

# SSH üzerinden gerçekleşen başarılı ve başarısız giriş denemelerini listeleme
cat /var/log/auth.log | grep "sshd" | grep -E 'Accepted|Failed'

# Kullanıcı yönetimi (ekleme, silme, parola değiştirme, grup değişikliği) hareketlerini süzme
cat /var/log/auth.log | grep -E '(passwd|useradd|usermod|userdel)\['

# sudo yetkisiyle çalıştırılan komutları tespit etme
cat /var/log/auth.log | grep -E 'COMMAND='
```

### C. Sistem ve Shell Geçmişi Analiz Komutları

Bash

```
# Debian tabanlı paket yöneticisi loglarında belirli bir paketin (unzip) kurulumunu inceleme
cat /var/log/dpkg.log | grep unzip

# Normal kullanıcının Bash geçmişini inceleme
cat /home/ubuntu/.bash_history

# Aktif oturumun komut geçmişini numaralandırılmış olarak görme
history

# Root kullanıcısının bash geçmişinde bayrak / saldırgan izi arama
sudo grep -i "THM{" /root/.bash_history
```

### D. Auditd Analiz Komutları (`ausearch`)

Bash

```
# Belirli bir anahtar kuralına (key) göre olayları aranabilir ve insan tarafından okunabilir (-i) formatta listeleme
# -i: UID'leri kullanıcı adına, zaman damgalarını okunabilir tarihe çevirir (interpret)
# -k: Denetim kuralına atanmış etikete (key) göre filtreler
ausearch -i -k proc_wget

# Belirli bir dosya izleme kuralını (file_sshconf) arama
ausearch -i -k file_sshconf

# secret.thm dosyasına yapılan ilk erişimi arama
ausearch -i | grep "secret.thm"

# Audit loglarında GitHub üzerinden indirilen dosyayı ve parametrelerini tespit etme
ausearch -i | grep "github"

# Ağ tarama aracı Naabu'nun çalışma parametrelerini tespit etme
ausearch -i | grep naabu
```

## 6. Pratik Laboratuvar ve Görev Soru-Cevap Anahtarı

|**#**|**Soru**|**Cevap**|**Analiz Metodolojisi & İlgili Komut**|
|---|---|---|---|
|**T2.1**|Which time server domain did the VM contact to sync its time?|`ntp.ubuntu.com`|`grep -i ntp /var/log/syslog` komutuyla zaman eşitleme isteği yakalanır.|
|**T2.2**|What is the kernel message from Yama in /var/log/syslog?|`Becoming mindful.`|`grep -i Yama /var/log/syslog` ile çekirdek başlatma mesajı bulunur.|
|**T3.1**|Which IP address failed to log in on multiple users via SSH?|`10.14.94.82`|`cat /var/log/auth.log \| grep "sshd" \| grep -E "Failed"` ile kaba kuvvet kaynağı tespit edilir.|
|**T3.2**|Which user was created and added to the "sudo" group?|`xerxes`|`cat /var/log/auth.log \| grep -E "(useradd\|usermod)"` komutuyla kalıcılık sağlanan hesap bulunur.|
|**T4.1**|According to the VM's package manager logs, which version of unzip was installed on the system?|`6.0-28ubuntu4.1`|`cat /var/log/dpkg.log \| grep unzip` çıktısındaki sürüm dizesinden elde edilir.|
|**T4.2**|What is the flag you see in one of the users' bash history?|`THM{note_to_remember}`|`sudo grep -i "THM{" /root/.bash_history` (Root dizinindeki geçmiş taranmalıdır).|
|**T5.1**|Which Linux system call is commonly used to execute a program?|`execve`|Bir sürecin başka bir programı çalıştırmak için çağırdığı temel sistem çağrısıdır.|
|**T5.2**|Can a typical program open a file or create a process bypassing system calls? (Yea/Nay)|`Nay`|Kullanıcı alanındaki yazılımlar çekirdek seviyesindeki sistem çağrılarını atlayamaz.|
|**T6.1**|When was the secret.thm file opened for the first time? (MM/DD/YY HH:MM:SS)|`08/13/25 18:36:54`|`ausearch -i -k file_thmsecret` veya `ausearch -i \| grep "secret.thm"` çıktısındaki zaman damgasıdır.|
|**T6.2**|What is the original file name downloaded from GitHub via wget?|`naabu_2.3.5_linux_amd64.zip`|`ausearch -i -k proc_wget` komutuyla `wget` komut satırı argümanlarından çekilir.|
|**T6.3**|Which network range was scanned using the downloaded tool?|`192.168.50.0/24`|`ausearch -i \| grep naabu` çıktısındaki `-host` parametresinin aldığı CIDR bloğudur.|

## 7. Tekrar Kategorisi ve Pazar Stratejisi

- **Kategori Sınıfı:** 🟡 **Sarı (Kavramsal ve Hızlı Triage / 3 Dakika)**
    
      
    
- **Gerekçe:** Linux dosya yolları (`/var/log/auth.log`, `/var/log/syslog`), temel filtreleme boru hatları (`grep -E`, `ausearch -i -k`) ve sistem çağrısı mekanizması (`execve`, `auid` vs `uid` farkı) mülakatlarda doğrudan senaryo sorusu olarak gelir.
    
      
    
- **Pazar Günü 3 Dakikalık Tekrar Stratejisi:**
    
      
    1. **1. Dakika:** Gözlerini kapat ve şu farkı hatırla: `auid` (sisteme ilk giren orijinal kullanıcı kimliği) ile `uid` (komutu o an çalıştıran, örn. sudo ile root olan kimlik) arasındaki ilişki neden EDR ve Auditd için vazgeçilmezdir?
        
          
        
    2. **2. Dakika:** Bash history'nin 4 kör noktasını say (boşluk bırakma, script çalıştırma, `/bin/sh` kullanımı, servis/cron çağrıları).
        
          
        
    3. **3. Dakika:** Şu üç dosyanın amacını eşleştir: `/var/log/auth.log` (Giriş/sudo/kullanıcı yönetimi), `/var/log/syslog` (Genel sistem/NTP/servisler), `/var/log/audit/audit.log` (Syscall seviyesinde derin çalışma zamanı takibi).
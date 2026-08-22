# 📓 THM - Sudo Buffer Overflow (CVE-2019-18634)

**Tags:** #BlueTeam #BufferOverflow #SudoVulns #CVE201918634 #pwfeedback #LinuxSecurity #PrivilegeEscalation #SOC_L1 #TryHackMe #Walkthrough

## 🏗️ Mimari Yapı

**CVE-2019-18634**, Sudo programının `1.8.26` öncesi sürümlerinde (ve bazı dağıtımların `1.8.30` öncesi derlemelerinde) ortaya çıkan kritik bir **Yığın Arabelleği Taşması (Heap-based Buffer Overflow)** zafiyetidir.

Bu zafiyet, Sudo'nun parola girdisi sırasındaki estetik bir konfigürasyon seçeneği olan **`pwfeedback`** modülünden kaynaklanır.

```
                      [ Linux Sudo Parola Girdisi ve Bellek Mimarisi ]
                                              │
         ┌────────────────────────────────────┴────────────────────────────────────┐
         ▼                                                                         ▼
[ Varsayılan Yapılandırma ]                                            [ `pwfeedback` Etkin Yapılandırma ]
- Parola girilirken ekrana çıktı verilmez.                            - Parola girilirken ekrana `*` karakteri basılır.
- Girdi arabelleği (Buffer) standart sınırlar içindedir.               - Sudo, girdileri işlerken bellek taşması denetiminde hata yapar.
- Taşma riski bulunmaz.                                                - Yığına (Heap) aşırı boyutta girdi gönderilerek bellek taşırılır.
                                                                       - Sonuç: Kök (root) yetkisinde kod yürütme!
```

### Zafiyetin Çalışma Mantığı

1. **`pwfeedback` Seçeneği:** Linux sistemlerde varsayılan olarak kullanıcı parola yazarken ekranda hiçbir şey görünmez. `/etc/sudoers` dosyasına `Defaults pwfeedback` satırı eklendiğinde, kullanıcı her tuşa bastığında ekrana bir yıldız (`*`) karakteri basılır.
    
2. **Buffer Overflow Tespiti:** Parola alanına taşıyıcı miktarda veri gönderildiğinde, Sudo için ayrılan bellek alanı (buffer) aşılır ve komşu bellek hücreleri ezilir (Segmentation Fault).
    
3. **Privilege Escalation:** Sudo programı $SUSD$ / $SetUID$ bitiyle (`root` yetkisinde) çalıştığı için, bellek alanı uygun şekilde manipüle edildiğinde herhangi bir Sudo yetkisi olmayan yerel kullanıcı bile sistemde doğrudan `root` yetkisinde kabuk (shell) elde edebilir.
    

## 💻 Tüm Komutlar, Parametreler ve Kod Parçacıkları

### A. Uzak Sunucuya Bağlantı (SSH)

- **SSH Bağlantı Komutu:**
    
    Bash
    
    ```
    ssh -p 4444 tryhackme@MACHINE_IP
    ```
    
    - `ssh`: Secure Shell protokolü üzerinden uzak sunucuya erişim başlatır.
        
    - `-p 4444`: Standart 22 portu yerine `4444` portundan bağlantı kurulacağını belirtir.
        
    - `tryhackme@MACHINE_IP`: Target sisteme `tryhackme` kullanıcısı ile oturum açar (Parola: `tryhackme`).
        

### B. Zafiyet Tespit ve Konfigürasyon Kontrol Komutları

- **Sudo Sürümünü Kontrol Etme:**
    
    Bash
    
    ```
    sudo --version
    ```
    
    - `--version`: Sistemde yüklü olan Sudo paketinin sürüm numarasını görüntüler. (`< 1.8.26` veya `< 1.8.30` sürümleri zafiyet barındırır).
        
- **Sudoers Konfigürasyonunda `pwfeedback` Tespiti:**
    
    Bash
    
    ```
    cat /etc/sudoers
    ```
    
    - `cat`: `/etc/sudoers` dosyasını okur. Dosya içinde `Defaults pwfeedback` parametresinin etkin olup olmadığı taranır.
        
- **Alternatif `pwfeedback` Görsel Tespiti:**
    
    - Eğer `/etc/sudoers` okuma yetkisi yoksa, herhangi bir `sudo` komutu çalıştırılıp parola yazılırken ekranda `*` (yıldız) simgelerinin çıkıp çıkmadığı kontrol edilir. Ekranda yıldızlar beliriyorsa `pwfeedback` aktiftir.
        

### C. PoC (Proof of Concept) ve Derleme Adımları

- **Buffer Overflow Varlığını Kanıtlama (Perl PoC):**
    
    Bash
    
    ```
    perl -e 'print "A" x 100000' | sudo -S id
    ```
    
    - `perl -e`: Tek satırlık Perl kodu çalıştırır.
        
    - `'print "A" x 100000'`: 100.000 adet "A" karakterinden oluşan devasa bir veri dizesi üretir.
        
    - `|` (Pipe): Üretilen bu çöp veriyi `sudo` komutuna parola girdisi olarak aktarır.
        
    - `-S`: Sudo'nun parolayı standart girdiden (stdin) okumasını sağlar.
        
    - **Çıktı:** `Segmentation fault` hatası alınması, belleğin taştığını ve sistemde Bof zafiyeti olduğunu doğrular.
        
- **C Kodunu Derleme ve Çalıştırma:**
    
    Bash
    
    ```
    gcc -o exploit exploit.c
    ./exploit
    ```
    
    - `gcc`: GNU C Derleyicisidir.
        
    - `-o exploit`: Derlenen çıktı dosyasının adını `exploit` olarak belirler.
        
    - `exploit.c`: C dilinde yazılmış kaynak kod dosyasıdır.
        
    - `./exploit`: Derlenmiş ikili dosyayı (binary) çalıştırarak `root` yetkisinde kabuk başlatır.
        

## 📊 Teknik Veriler ve Sınıflandırmalar

### A. CVE-2019-14287 ve CVE-2019-18634 Karşılaştırması

|**Metrik / Özellik**|**CVE-2019-14287 (Sudo Security Bypass)**|**CVE-2019-18634 (Sudo Buffer Overflow)**|
|---|---|---|
|**Zafiyet Türü**|Mantıksal / Tamsayı Dönüşüm Hatası (Integer Underflow)|Yığın Tabakası Arabelleği Taşması (Heap Buffer Overflow)|
|**Ön Koşul (Prerequisites)**|Kullanıcının `/etc/sudoers` içinde `!root` kısıtlamasıyla tanımlı Sudo yetkisinin olması.|Kullanıcının **hiçbir Sudo yetkisinin olmasına gerek yoktur**. Sadece `pwfeedback` etkin olmalıdır.|
|**Tetikleyici Anahtar**|`sudo -u#-1` veya `sudo -u#4294967295`|`Defaults pwfeedback` konfigürasyon seçeneği.|
|**Etkilenen Sürümler**|Sudo `< 1.8.28`|Sudo `< 1.8.26` (ve bazı dağıtımlarda `< 1.8.30`)|
|**Etki Seviyesi**|Yüksek (Yetki Yükseltme - PrivEsc)|**Kritik (Sıfır Yetkiden Root Yetkisine Geçiş)**|

### B. Zafiyet Teknik Özeti

|**Parametre**|**Değer**|
|---|---|
|**CVE Kodu**|CVE-2019-18634|
|**Keşfeden Araştırmacı**|Joe Vennix (Apple)|
|**Exploit Yazarı**|Saleem Rashid (`@saleemrash1d`)|
|**Çözüm / Yama**|`pwfeedback` seçeneğini kapatmak veya Sudo sürümünü güncellemek.|

## 🧠 Uzman Notları ve Kritik Uyarılar

> ⚠️ **MÜLAKAT HAPI - "PWFEEDBACK" NEDEN RİSKLİDİR?**
> 
> SOC ve Mavi Takım mülakatlarında _"Bir sistemde kullanıcı Sudo yetkisine sahip olmasa dahi Sudo üzerinden nasıl root yetkisi alabilir?"_ sorusu geldiğinde verilecek yanıt:
> 
> Sudoers dosyasında `Defaults pwfeedback` ayarı etkinse ve Sudo sürümü güncellenmemişse (`CVE-2019-18634`), Sudo'nun parola girdi arabelleğinde yığın taşması (Heap Buffer Overflow) meydana gelir. Saldırganın `/etc/sudoers` dosyasında hiçbir özel izne (NOPASSWD vb.) ihtiyacı yoktur. Sadece sisteme standart bir kullanıcı olarak oturum açabilmesi yeterlidir.

> 💡 **SEKTÖREL İPUCU - Sıkılaştırma (Hardening) Ve İyileştirme:**
> 
> Sistem yöneticileri bu zafiyetten korunmak için:
> 
> 1. Sudo paketlerini acilen güvenli sürüme yükseltmelidir.
>     
> 2. `/etc/sudoers` dosyası içindeki `Defaults pwfeedback` satırını kaldırmalı veya `Defaults !pwfeedback` şeklinde pasifize etmelidir.
>     

> 🚨 **SOC ANALİSTİ UYARISI - SEGMENTATION FAULT LOGLARI:**
> 
> Bir kullanıcı arka arkaya Sudo komutları çalıştırırken sistem loglarında (`/var/log/syslog` veya `dmesg`) `segmentation fault` veya `segfault` kayıtları üretiyorsa, bu durum bir saldırganın bellek taşması (Buffer Overflow) zafiyetini tetiklemeye veya offset değerlerini bulmaya çalıştığının belirgin bir göstergesidir (Indicator of Attack - IoA).

## 🗃️ Soru-Cevap Anahtarı

| **Pratik / Laboratuvar Sorusu**                                                                                                  | **Doğru Cevap**              |
| -------------------------------------------------------------------------------------------------------------------------------- | ---------------------------- |
| **[LAB]** Pre-compiled exploit dosyası çalıştırılıp root kabuğu alındıktan sonra `/root/root.txt` dosyasından okunan flag nedir? | `THM{buff3r_0v3rfl0w_rul3s}` |
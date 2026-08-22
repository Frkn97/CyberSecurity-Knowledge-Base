# 📓THM - Linux Fundamentals Part 3 (Metin Editörleri, Süreç Yönetimi, Sistem Otomasyonu ve Adli Log Analizi)

**Tags:** #Linux #LinuxFundamentals #Processes #CronJobs #PackageManagement #LogAnalysis #BlueTeamFundamentals

## 📝 1. Terminal Metin Editörleri (Terminal Text Editors)

Dosyalara sadece `echo` ve `>` kullanarak veri yazmak büyük dosyalarda veya kod düzenlemede verimsizdir. Linux kabuğunda terminal üzerinden metin yönetimi için iki ana araç kullanılır:

### A) Nano Editörü (Kullanıcı Dostu)

Yeni başlayanlar için basit ve pratiktir. Alt kısımda yer alan kısayollardaki `^` sembolü `Ctrl` tuşunu temsil eder.

- **Dosya Açma / Oluşturma:** `nano dosya_adi`
    
- **Temel Kısayollar:** `Ctrl + O` (Dosyayı Kaydet/Write Out), `Ctrl + W` (Metin İçi Arama/Where Is), `Ctrl + X` (Çıkış).
    
- _Lab Bulgusu:_ `tryhackme` ev dizinindeki "task3" dosyası Nano ile açıldığında elde edilen bayrak: **`THM{TEXT_EDITORS}`**
    

### B) VIM Editörü (Gelişmiş & Profesyonel)

Tamamen klavye odaklı, tüm Unix/Linux sistemlerinde istisnasız yerleşik gelen, kodlar için sözdizimi vurgulama (_Syntax Highlighting_) desteği sunan güçlü ve siber güvenlik operasyonlarında kas hafızası olması gereken editördür.

## 🌐 2. Dosya Transferi ve Sunumu (File Transfer & Serving)

### A) `wget` ile Dosya İndirmek

Web üzerinden (HTTP/HTTPS protokolü kullanarak) ağdaki veya internetteki dosyaları yerel makineye indirmeyi sağlar.

- **Kullanımı:** `wget <dosya_url_adresi>`
    

### B) Python3 ile Pratik Geçici Web Sunucusu Kurmak

Mavi veya Kırmızı takım operasyonlarında, bulunulan dizindeki dosyaları ağdaki diğer makinelerin hızlıca indirebilmesi için anlık bir HTTP sunucusu başlatılabilir:

- **Komut:** `python3 -m http.server` _(Varsayılan olarak 8000 portunda çalışır)._
    
- _Lab Operasyonu:_ Hedef makinede sunucu başlatıldıktan sonra AttackBox üzerinden koşturulan `wget http://machine_ip:8000/.flag.txt` komutuyla elde edilen bayrak: **`THM{WGET_WEBSERVER}`**
    

### C) `scp` (Secure Copy) ile Güvenli Dosya Transferi

SSH protokolünün şifreleme altyapısını kullanarak iki makine arasında güvenli dosya kopyalamayı sağlar:

- **Lokalden Uzak Makineye:** `scp lokal_dosya.txt kullanıcı@uzak_ip:/home/kullanıcı/`
    
- **Uzak Makineden Lokale:** `scp kullanıcı@uzak_ip:/uzak_yol/dosya.txt /lokal_hedef_klasor/`
    

## ⚙️ 3. Süreç Yönetimi Mimarisi (Processes 101)

Sistemde çalışan her programa **Process (Süreç)** denir. Her sürece Linux çekirdeği (Kernel) tarafından benzersiz ve ardışık olarak artan bir **PID (Process ID)** numarası atanır. Eğer mevcut süreç ID'si X ise, bir sonraki süreç **X+1** (Örn: 300'den sonra **301**) değerini alır.

### A) Süreçleri İzleme Komutları

- **`ps`:** Sadece mevcut kullanıcının o anki terminal oturumuna ait süreçleri gösterir.
    
- **`ps aux`:** Sistemdeki tüm kullanıcıların ve arka plan servislerinin detaylı listesini döker.
    
    - _Lab Bulgusu:_ `ps aux | grep thm` kombinasyonuyla arka planda çalışan gizli süreçten yakalanan bayrak: **`THM{PROCESSES}`**
        
- **`top`:** Sistem kaynaklarını (CPU, RAM) en çok tüketen süreçleri anlık (_real-time_) olarak listeleyen canlı paneldir.
    

### B) Süreç Sonlandırma ve Kritik Sinyal Anatomisi (Mülakat Sorusu)

Bir süreci kapatmak için `kill <PID>` komutu eşliğinde belirli sinyaller gönderilir:

1. **`SIGTERM` (Sinyal 15):** Süreci **temiz bir şekilde (cleanly)** kapatır. Sürecin kapanmadan önce geçici dosyalarını temizlemesine ve log yazmasına izin verir. (Varsayılan güvenli kapatma sinyalidir).
    
2. **`SIGKILL` (Sinyal 9):** Süreci anında ve acımasızca sonlandırır (`kill -9 <PID>`). Sürecin temizlik yapmasına izin vermez; kilitlenen süreçleri vurmak için kullanılır.
    
3. **`SIGSTOP`:** Süreci kapatmaz, arka planda dondurur/askıya alır.
    

### C) systemd ve systemctl Servis Yönetimi

Linux önyükleme yaparken (boot) çalışan ilk süreç `systemd` (PID 1) mimarisidir. Arka plan servislerini yönetmek için `systemctl [opsiyon] [servis_adi]` yapısı kullanılır:

- `systemctl stop myservice` ➡️ Çalışan servisi durdurur.
    
- `systemctl enable myservice` ➡️ Servisin sistem her açıldığında (**boot-up**) otomatik başlamasını sağlar.
    
- `systemctl disable myservice` ➡️ Otomatik başlangıcı iptal eder.
    

### D) Süreçleri Arka Planda (Background) ve Ön Planda (Foreground) Yönetme

- Bir komutun sonuna **`&`** operatörü eklenirse süreç arka planda çalışır, terminal kilitlenmez.
    
- Ön planda çalışan bir süreç `Ctrl + Z` ile askıya alınabilir.
    
- Arka planda duraklatılmış veya çalışan bir süreci tekrar terminal ekranının odağına getirmek için **`fg`** (_foreground_) komutu koşturulur.
    

## ⏰ 4. Sistem Otomasyonu: Görev Zamanlama (Cron Jobs)

Belirli scriptlerin veya yedekleme görevlerinin sistem tarafından otomatik olarak belirli zaman aralıklarında çalıştırılmasını sağlar. Bu işlemi `cron` servisi yürütür.

- **Crontab Düzenleme:** `crontab -e` komutuyla zamanlanmış görevler dosyası açılır.
    
- **Zamanlama Formatı:** `DAKİKA SAAT AYIN_GÜNÜ AY HAFTANIN_GÜNÜ KOMUT` _(Örn: `0 */12 * * * /backup.sh`)_
    
- 🚨 **`@reboot` Parametresi:** Zaman parametrelerinden bağımsız olarak, cron işinin sistem her yeniden başlatıldığında (**reboot**) otomatik olarak koşturulmasını sağlayan kritik bir otomasyon/persistance tetikleyicisidir.
    

## 📦 5. Paket Yönetimi ve Üçüncü Parti Depolar (Package Management)

Ubuntu tabanlı sistemlerde yazılım yüklemek, güncellemek ve kaldırmak için `apt` (_Advanced Package Tool_) aracı kullanılır. Yazılımlar _Repository_ (Depo) adı verilen uzak sunuculardan çekilir.

### Üçüncü Parti Güvenli Depo Ekleme Adımları

Varsayılan depolarda olmayan bir yazılımı güvenli şekilde sisteme dahil etme standardı:

1. **GPG Anahtarı Ekleme:** İndirilecek paketlerin bütünlüğünü ve orijinal olduğunu doğrulamak için geliştiricinin şifreleme anahtarı sisteme tanıtılır: `wget -qO - [https://download.sublimetext.com/sublimehq-pub.gpg](https://download.sublimetext.com/sublimehq-pub.gpg) | sudo apt-key add -`
    
2. **Depoyu Listeye Ekleme:** `/etc/apt/sources.list.d/` dizini altına yeni bir `.list` dosyası oluşturulup deponun adresi yazılır.
    
3. **Depoları Güncelleme & Yükleme:** Sistemin yeni depoyu tanıması sağlanır ve paket kurulur: `sudo apt update && sudo apt install sublime-text`
    

## 📊 6. Adli Bilişim ve Sistem Günlükleri (Log Analysis)

Linux'ta tüm servislerin, kimlik doğrulama adımlarının ve işletim sisteminin hareket izleri `/var/log` dizini altında tutulur.

- **Log Rotasyonu (Log Rotation):** Log dosyalarının diski doldurmaması için sistem bunları otomatik olarak arşivler (`.log.1`, `.log.2.gz` şeklinde) ve belirli aralıklarla temizler.
    
- **Apache2 Web Sunucu Log Analizi Durum Çalışması (Case Study):**
    
    - Standart `access.log` dosyasına düşük yetkili kullanıcıların erişim izni yokken, rotasyona uğramış **`access.log.1`** dosyası `tryhackme` kullanıcısı tarafından okunabilmektedir (`ls -la` analizi).
        
    - _Adli Bulgular:_ Log dosyası incelendiğinde, web sitesini ziyaret eden şüpheli saldırgan IP adresinin **`10.9.232.111`** olduğu ve bu IP'nin sunucu üzerinde doğrudan **`/catsanddogs.jpg`** dosyasına erişim sağladığı tespit edilmiştir.
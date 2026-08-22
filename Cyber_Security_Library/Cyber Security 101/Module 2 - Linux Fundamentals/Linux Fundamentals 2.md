# 📓THM - Linux Fundamentals Part 2 (SSH Bağlantısı, İzinler Mimarisi, Dosya Operasyonları ve Sistem Dizinleri)

**Tags:** #Linux #LinuxFundamentals #SSH #Permissions #FileManagement #FileCommand #BlueTeamFundamentals

## 🌐 1. Uzak Bağlantı Güvenliği: SSH (Secure Shell)

Siber güvenlik operasyonlarında veya sunucu yönetiminde, uzak bir Linux makinesinin komut satırına (Terminal) güvenli, şifreli ve uzaktan erişim sağlamak için **SSH** protokolü kullanılır. Tüm ağ trafiği kriptolanarak dinlenmesi (sniffing) engellenir.

- **Bağlantı Sentaksı:** `ssh kullanıcı_adı@IP_ADRESI`
    
- 🚨 **Kritik Güvenlik Refleksi:** SSH üzerinden terminalde şifre yazarken ekran üzerinde hiçbir karakter, imleç hareketi veya yıldız (`*`) işareti görünmez. Bu durum Linux kabuğunun yerleşik bir güvenlik özelliğidir; panik yapmadan şifreyi yazıp `Enter` tuşuna basılmalıdır.
    

## 🛠️ 2. Komutların Gücü: Flaglar, Parametreler ve Man Sayfaları

Linux komutları tek başlarına kısıtlı çalışır. Onların işlevlerini genişletmek ve özelleştirmek için arkalarına **Flag (Bayrak)** veya **Switch (Anahtar)** dediğimiz parametreleri ekleriz.

- **Yardım Almak (`--help`):** Komutun en temel ve hızlı kullanım kılavuzunu ekrana basar.
    
- **Kılavuz Sayfaları (`man`):** Komutun dünyadaki tüm parametrelerini, ne işe yaradığını gösteren devasa kılavuz dökümanını açar. (Örn: `man ls`). Sayfadan çıkmak için klavyeden `q` tuşuna basılır.
    
- **İnsan Gözü Standardı (`-h` / `--human-readable`):** Dosya boyutlarını bayt yığını yerine `K` (Kilobayt), `M` (Megabayt), `G` (Gigabayt) cinsinden okunabilir hale getirir. (Örn: `ls -lh`).
    

## 📂 3. Gelişmiş Dosya Sistemi Operasyonları ve Dosya Tipi Analizi

Linux terminallerinde dosyaları yönetmek ve adli analiz (forensics) yapmak için kullanılan çekirdek komut setleri:

|Komut|Teknik İşlevi|Operasyonel Kullanım Formatı|
|---|---|---|
|**`touch`**|Belirtilen isimde boş bir dosya yaratır.|`touch newnote`|
|**`mkdir`**|Sistemde yeni bir klasör/dizin oluşturur.|`mkdir myfolder`|
|**`cp`**|Dosya veya klasörleri kopyalar (çoğaltır).|`cp kaynak_dosya hedef_konum`|
|**`mv`**|Dosyayı taşır veya dosyanın adını değiştirir.|`mv myfile myfolder`|
|**`rm`**|Dosya siler. Klasör silmek için `-R` (_recursive_) şarttır.|`rm -R silinecek_klasor`|
|**`file`**|**Adli Bilişim Silahı.** Uzantısı olmasa bile dosyanın gerçek imzasını okur.|`file unknown1` ➡️ Çıktı: `ASCII text`|

## 🔐 4. İzinler 101 ve Kullanıcı Yönetimi (Permissions Mimarisi)

Linux çok kullanıcılı bir sistemdir. `ls -lh` komutu koşturulduğunda dosyaların başında `rwxr-xr-x` gibi izin maskeleri görünür. Bu yapı 3 ana grupta incelenir: **Owner (Sahip)**, **Group (Grup)** ve **Others (Diğerleri)**.

### Sayısal (Numeric) İzin Hesaplama Mantığı

İzin atamaları ve kontrolleri bit düzeyinde matematiksel değerlere sahiptir:

- **Read (r) = 4** (Okuma Yetkisi)
    
- **Write (w) = 2** (Yazma/Düzenleme Yetkisi)
    
- **Execute (x) = 1** (Çalıştırma Yetkisi)
    

> 💡 **Kritik İzin Kalıpları Analizi:**
> 
> - `755` (`rwxr-xr-x`): Dosya sahibi her şeyi yapar; grup ve dışarıdakiler sadece okur ve çalıştırır.
>     
> - `644` (`rw-r--r--`): Dosya sahibi okur ve yazar; geri kalan herkes sadece okuyabilir.
>     
> - ⚠️ `777` (`rwxrwxrwx`): **Büyük Güvenlik Açığı!** Dünyadaki herkesin o dosyayı silme, değiştirme ve çalıştırma yetkisi vardır. Kurumsal sistemlerde sıkı takip edilmelidir.
>     

### Kullanıcı Değiştirme Operasyonu (`su`)

- **`su [kullanıcı]`:** Mevcut terminal oturumundan çıkmadan, belirtilen kullanıcının yetki düzeyine geçiş yapar ve ilgili kullanıcının parolasını talep eder.
    
- **`su -l [kullanıcı]`:** Hedef kullanıcının sadece yetkilerini değil, tüm çevre ortam değişkenlerini (Environment Variables) ve ana ev (`/home`) dizinini de yükleyerek temiz bir geçiş sağlar.
    

## 📁 5. Kritik Sistem Dizinleri (Nerede Ne Var?)

Siber güvenlik analistlerinin sızma sonrası (Post-Exploitation) veya log incelemede ezbere bilmesi gereken kök dizin haritası:

|Dizin|Sistemdeki Görevi|Siber Güvenlik ve Tehdit Avcılığı Açısından Önemi|
|---|---|---|
|**`/etc`**|Sistem genelindeki tüm konfigürasyon ve yapılandırma dosyalarını barındırır.|⭐⭐⭐ `passwd`, `shadow` ve `sudoers` gibi kritik kimlik denetim dosyaları buradadır.|
|**`/var`**|Sistem çalıştıkça boyutu değişen dinamik ve değişken verileri tutar.|⭐⭐⭐ **Tehdit Avcılığı Kalbi.** `/var/log` dizini altındaki tüm sistem ve servis logları adli analiz için buradadır.|
|**`/root`**|En yetkili kullanıcı olan "root" profilinin kişisel ev (home) dizinidir.|⭐⭐ Standart kullanıcıların bu dizine erişim ve okuma yetkisi kesinlikle bulunmaz.|
|**`/tmp`**|Geçici dosyaların tutulduğu dizindir. Bilgisayarlardaki **RAM** mantığı gibi çalışır; sistem yeniden başladığında tamamen temizlenir.|⭐⭐ **Sızma Sonrası İstasyon.** Sistemdeki herkesin yazma/çalıştırma hakkı olduğu için exploit scriptlerini çalıştırmak için ideal yerdir.|

## 🛠️ 6. Laboratuvar Pratik Çıktıları ve Nokta Atışı Cevaplar

Odadaki etkileşimli Linux görevlerinde komut satırında doğrulatılan resmi çözümler:

- **Soru 1:** "newnote" isminde yeni bir boş dosya oluşturmayı sağlayan komut nedir?
    
    - _Yanıt:_ **`touch newnote`**
        
- **Soru 2:** `tryhackme` kullanıcısının ev dizininde yer alan "unknown1" dosyasının `file` komutuyla analiz edilen gerçek dosya türü nedir?
    
    - _Yanıt:_ **`ASCII text`**
        
- **Soru 3:** "myfile" isimli dosyayı "myfolder" klasörünün içerisine taşımayı sağlayan komut nedir?
    
    - _Yanıt:_ **`mv myfile myfolder`**
        
- **Soru 4:** Taşınan bu dosyanın `cat` ile okunan içeriğindeki ilk bayrak (flag) nedir?
    
    - _Yanıt:_ **`THM{FILESYSTEM}`**
        
- **Soru 5:** Sistemde yer alan "important" isimli dosyanın asıl sahibi (owner) hangi kullanıcıdır?
    
    - _Yanıt:_ **`user2`**
        
- **Soru 6:** Terminal üzerinden "user2" isimli kullanıcının oturumuna geçiş yapmayı sağlayan komut nedir?
    
    - _Yanıt:_ **`su user2`**
        
- **Soru 7:** `su user2` geçişi yapıldıktan sonra, "important" dosyasının okunmasıyla elde edilen ikinci bayrak (flag) nedir?
    
    - _Yanıt:_ **`THM{SU_USER2}`**
        
- **Soru 8:** Linux sistemlerde log dosyalarının depolanmasını beklediğimiz standart dizin yolu nedir?
    
    - _Yanıt:_ **`/var/log`**
        
- **Soru 9:** Bilgisayarlardaki RAM mantığı gibi çalışan, geçici verileri tutan ve sistem kapanınca sıfırlanan kök dizin hangisidir?
    
    - _Yanıt:_ **`/tmp`**
        
- **Soru 10:** En yetkili sistem yöneticisi olan "root" kullanıcısının ev (home) dizininin adı nedir?
    
    - _Yanıt:_ **`/root`**
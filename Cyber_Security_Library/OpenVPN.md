# TryHackMe OpenVPN Bağlantı Rehberi

TryHackMe platformundaki hackleme ve öğrenme odaklı sanal makineleri (VM) kontrol edebilmek için öncelikle THM ağına bağlanmanız gerekir. Bu bağlantı, kendi siber güvenlik laboratuvarınız (veya ana makineniz) ile THM ağ arasında güvenli bir tünel (VPN) oluşturur.

> ⚠️ **Önemli Kural:** Eğer bir Sanal Makine (Virtual Machine - Kali Linux vb.) kullanıyorsanız, VPN bağlantısını ana bilgisayarınızda (Host) değil, **sanal makinenizin içinde** kurmalısınız.

## 0. Ön Hazırlık

Hangi işletim sistemini kullanırsanız kullanın, ilk olarak THM platformunun **Access (Erişim)** sayfasına gidip size özel olarak üretilen `.ovpn` uzantılı **VPN konfigürasyon dosyasını** bilgisayarınıza indirmelisiniz.

---

## 1. Windows İşletim Sistemi için Bağlantı Adımları

1. **İndirin:** [OpenVPN GUI](https://openvpn.net/community-downloads/) resmi açık kaynaklı uygulamasını bilgisayarınıza indirin.
    
2. **Kurun:** İndirdiğiniz kurulum (`.exe`) dosyasını açarak kurulum sihirbazını takip edin.
    
3. **Yönetici Olarak Çalıştırın:** Masaüstündeki veya aramadaki OpenVPN GUI uygulamasına sağ tıklayıp **"Yönetici Olarak Çalıştır" (Run as Administrator)** seçeneğini seçin.
    
4. **Dosyayı İçe Aktarın:** Uygulama ekranın sağ alt köşesinde, saat/tarih yanındaki sistem tepsisinde (system tray) çalışmaya başlayacaktır. Buradaki OpenVPN simgesine sağ tıklayın ve **"File İçe Aktar" (Import File)** seçeneğine tıklayın.
    
5. **Konfigürasyonu Seçin:** THM'den indirdiğiniz `.ovpn` uzantılı dosyayı seçerek sisteme yükleyin.
    
6. **Bağlanın:** Sistem tepsisindeki simgeye tekrar sağ tıklayın, yüklediğiniz dosyayı seçin ve **"Bağlan" (Connect)** butonuna basın. _(Bağlantıyı kesmek istediğinizde yine aynı yolu izleyip "Bağlantıyı Kes" / "Disconnect" diyebilirsiniz)._
    

---

## 2. MacOS İşletim Sistemi için Bağlantı Adımları

1. **İndirin:** macOS uyumlu OpenVPN istemcisini indirin.
    
2. **Kurun:** İndirdiğiniz `.dmg` uzantılı dosyayı açarak yönergeleri takip edin ve kurulumu tamamlayın.
    
3. **Çalıştırın:** OpenVPN GUI uygulamasını başlatın.
    
4. **Dosyayı İçe Aktarın:** Uygulama üst menü barında belirecektir. Simgeye sağ tıklayıp **Import File -> Local file** (Dosya İçe Aktar -> Yerel Dosya) yolunu izleyin.
    
5. **Konfigürasyonu Seçin:** Önceden indirdiğiniz `.ovpn` uzantılı THM dosyasını seçin.
    
6. **Bağlanın:** Üst bardaki simgeye tekrar sağ tıklayın, profilinizi seçin ve **"Connect" (Bağlan)** seçeneğine tıklayın. _(Bağlantıyı kesmek için aynı menüden "Disconnect" seçeneğini kullanabilirsiniz)._
    

---

## 3. Linux İşletim Sistemi için Bağlantı Adımları

Linux (Debian/Kali/Ubuntu vb.) tabanlı sistemlerde işlemler tamamen terminal üzerinden yürütülür:

1. **Uygulamayı Kurun:** Terminali açın ve OpenVPN'i yüklemek için şu komutu çalıştırın:
    
    Bash
    
    ```
    sudo apt install openvpn
    ```
    
2. **Dosya Yolunu Belirleyin:** İndirdiğiniz `.ovpn` dosyasının bilgisayardaki tam konumunu bulun (Genellikle `/home/kullanici_adi/Downloads/` veya Türkçe sistemlerde `~/İndirilenler/` klasöründedir).
    
3. **Bağlantıyı Başlatın:** Terminalde şu komut şablonunu kendi dosya adınıza göre düzenleyerek çalıştırın:
    

Bash

```
   sudo openvpn /dosya-yolu/dosya-adi.ovpn
```

_(Terminal açık kaldığı sürece bağlantınız aktif olacaktır. Bağlantıyı koparmak için terminal ekranında `CTRL + C` tuş kombinasyonuna basmanız yeterlidir)._

---

## 4. Alternatif Yöntem: TryHackMe AttackBox (Tarayıcı Tabanlı)

Eğer kendi bilgisayarınızda VPN bağlantısı kurarken teknik bir sorun yaşıyorsanız veya VPN kullanmak istemiyorsanız THM size hazır bir Linux makinesi sunar:

- **Nedir?:** THM platformu üzerinde, tamamen tarayıcınızın içinden kontrol edebileceğiniz Kali Linux tabanlı hazır bir siber güvenlik makinesidir (**AttackBox**).
    
- **Nasıl Açılır?:** Bulunduğunuz odanın (room) sağ üst köşesinde yer alan **"Start AttackBox"** butonuna basarak başlatabilirsiniz. Makinenin yüklenmesi birkaç dakika sürer.
    
- **Kullanım Hakları:** Ücretsiz kullanıcılar AttackBox'ı günde **1 saat** boyunca kullanabilirken, premium aboneler için herhangi bir süre sınırı yoktur.
    
- **Avantajı:** AttackBox zaten THM ağının içinde hazır başladığı için **ekstra bir OpenVPN kurulumu veya bağlantısı yapmanıza gerek kalmaz.**
    

---

## 5. Bağlantı Kontrolü ve Doğrulama (Flag Alımı)

Bağlantınızın başarıyla gerçekleşip gerçekleşmediğini test etmek için şu adımları izleyin:

1. **Erişim Sayfası Kontrolü:** THM'deki _Access_ sayfasına gidin. Buradaki "Network Information" (Ağ Bilgisi) tablosunda "Connected" yazısının yanında **yeşil bir onay işareti (tick)** görüyorsanız bağlantınız başarılı demektir.
    
2. **Web Sitesi Testi:** İlgili görevde yer alan sanal makineyi başlatın (**Deploy** butonuna basın). Makinenin tamamen açılması 1-2 dakika sürebilir.
    
3. **Adrese Gitme:** Makine açıldığında size bir IP adresi verilecektir (`MACHINE_IP`). Tarayıcınızın adres çubuğuna `http://MACHINE_IP` yazarak giriş yapın.
    
4. **Bayrağı Yakalama (Flag):** Eğer bağlantınız sağlıklıysa makinenin içindeki web sitesi açılacaktır. Sayfa üzerinde yer alan ve sizden soru kutucuğuna girmeniz istenen özel **Flag (Bayrak)** metnini bularak görevi tamamlayabilirsiniz.
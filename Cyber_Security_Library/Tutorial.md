# TryHackMe: Temel Laboratuvar Kullanımı ve İlk Flag

## 📌 Genel Bakış

TryHackMe (THM) platformunda eğitimler ve siber güvenlik görevleri iki temel bileşenden oluşur:

- **Saldırı Makinesi (AttackBox):** Sizin siber güvenlik araçlarını çalıştırdığınız, saldırıyı gerçekleştirdiğiniz kendi sisteminizdir.
    
- **Hedef Makine (Target Machine):** Zafiyet barındıran, sızılması veya incelenmesi gereken karşı sistemdir.
    

---

## 🛠️ Erişim Yöntemleri: AttackBox vs. OpenVPN

Platformdaki hedef makinelere erişmek ve onları analiz etmek için iki farklı yol vardır:

### 1. Web Tabanlı AttackBox (Önerilen / Kolay)

- **Nedir?** Tarayıcı üzerinden tek tıkla çalışan, içinde ihtiyacınız olan tüm siber güvenlik araçları hazır kurulu gelen bir Linux (Ubuntu) makinesidir.
    
- **Nasıl Kullanılır?** Odanın en üstündeki mavi **"Start AttackBox"** butonuna basılarak başlatılır. Ekran ikiye bölünür; sağ tarafta kendi saldırı makinenizin masaüstü açılır.
    

### 2. Kendi Bilgisayarınız + OpenVPN (Gelişmiş / CLI Severler İçin)

- **Nedir?** Araçları kendi yerel Linux sisteminizde veya terminalinizde çalıştırmak istediğinizde, THM ağına güvenli bir tünel açarak laboratuvara bağlanma yöntemidir. Grafik arayüz yerine komut satırı hızı ve esnekliği sağlar.
    
- **Nasıl Kullanılır?** TryHackMe Access sayfasından kişisel `.ovpn` konfigürasyon dosyası indirilir. Terminalden `sudo openvpn dosya_adi.ovpn` komutuyla çalıştırılarak ağa dahil olunur.
    

---

## 🚀 Adım Adım Laboratuvar Uygulaması ve Flag Alma

Bu görevde ilk laboratuvar deneyimi ve sisteme erişim şu adımlarla gerçekleştirilir:

1. **Hedef Makineyi Başlatın:** Görev kartındaki yeşil veya mavi **"Start Machine"** butonuna basın. Makinenin arka planda tamamen kurulması ve size özel bir IP adresi (örn: `10.10.x.x`) üretmesi için yaklaşık **1 dakika** bekleyin.
    
2. **IP Adresini Kopyalayın:** Sistem tarafından üretilen bu hedef IP adresini panoya alın.
    
3. **Hedef Web Sitesine Bağlanın:**
    
    - Eğer **AttackBox** kullanıyorsanız: Sağ tarafta açılan AttackBox içindeki **Firefox** tarayıcısını açın, kopyaladığınız IP adresini URL çubuğuna yapıştırıp `Enter` tuşuna basın.
        
    - Eğer **OpenVPN** kullanıyorsanız: Kendi bilgisayarınızın yerel tarayıcısını açıp bu IP'ye gidin.
        
4. **Flag'i Alın:** Karşınıza çıkan web sitesinde doğrudan odanın doğrulanmasını sağlayacak olan özel kod yer alacaktır.
    

---

## 🔑 Temel Kavramlar

- **Flag (Bayrak):** Bir siber güvenlik görevini veya sızma işlemini başarıyla tamamladığınızı kanıtlamak için sistem derinliklerinden ya da hedef sayfalardan elde ettiğiniz, genellikle `THM{...}` formatında olan benzersiz metin dizisidir.
    
- **Terminate (Sonlandırma):** Laboratuvardaki işiniz bittiğinde, platform kaynaklarını boşa çıkarmak ve günlük makine haklarınızın boş yere harcanmasını önlemek için kırmızı **"Terminate"** butonuna basarak makineleri her zaman kapatmalısınız.
    

---

**Doğru Cevap (İlk Flag):** Bu odadaki adımları tamamladığınızda tarayıcı ekranınızda belirecek olan ilk tanıtım bayrağı: **`THM{HACKING_YOUR_FIRST_MACHINE}`**

📅 **Not Tarihi:** 2026-05-16

🏷️ **Etiketler:** #cybersecurity #tryhackme #ctf #attackbox #openvpn #notes
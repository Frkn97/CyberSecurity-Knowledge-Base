# 🛡️ Ağ Güvenliği Temelleri ve Siber Saldırı Metodolojisi (Network Security)

**Etiketler:** `#AğGüvenliği` `#NetworkSecurity` `#CyberKillChain` `#Nmap` `#FTP` `#SSH` `#LinuxTemelleri` `#TryHackMe`

## 1. Mimari Yapı ve Ağ Savunma Çerçevesi

Bilgisayar ağı; veri, kaynak ve hizmet paylaşımı amacıyla birbirine bağlanan cihazlar topluluğudur. Ağ güvenliği ise bu cihazların ve aralarındaki bağlantı hatlarının **Gizlilik (Confidentiality)**, **Bütünlük (Integrity)** ve **Erişilebilirlik (Availability)** ilkelerini (CIA Triad) korumayı hedefler.

  

Güvenlik mekanizmaları kurumsal yapılarda iki ana grupta ele alınır:

  

- **Donanım Çözümleri (Ağ Katmanı / Perimeter):** Ağ trafiğinin merkezinde fiziksel olarak konumlandırılan, ayrılmış donanım bileşenleridir.
    
      
    
- **Yazılım Çözümleri (Uç Nokta / Host Katmanı):** Doğrudan işletim sistemine entegre veya son kullanıcı cihazlarına kurulan güvenlik yazılımlarıdır.
    
      
    

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        AĞ GÜVENLİK MİMARİSİ VE SAVUNMA KATMANLARI                      │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 1. DONANIM CİHAZLARI (Hardware Appliances - Ağ Geçidi / Perimeter):                    │
│    • Güvenlik Duvarı (Firewall): Tanımlı kurallara göre ağ trafiğini filtreler.        │
│    • IDS (Intrusion Detection): Yetkisiz giriş ve saldırı girişimlerini tespit eder.   │
│    • IPS (Intrusion Prevention): Tespit edilen saldırıyı gerçek zamanlı engeller.      │
│    • VPN Konsantratörü: Şifreli tünellerle gizlilik ve veri bütünlüğü sağlar.          │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 2. YAZILIM ÇÖZÜMLERİ (Software Solutions - Uç Nokta / Host):                           │
│    • Host Firewall: Doğrudan işletim sistemini korur (Örn: Windows Defender Firewall). │
│    • Antivirüs (AV) / EDR: Cihaz üzerindeki zararlı yazılımları ve süreçleri engeller. │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 3. SALDIRGAN YAKLAŞIMI: LOCKHEED MARTIN CYBER KILL CHAIN                               │
│    Keşif ──► Silahlanma ──► Teslimat ──► İstismar ──► Yükleme ──► C2 ──► Hedef Eylemleri│
│   (Recon)  (Weaponize)   (Delivery)   (Exploit)    (Install)   (C2)    (Actions)   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

> [!INFO]
> 
> **Veri İhlali Maliyetleri (IBM Cost of a Data Breach):**
> 
>   
> 
> Veri ihlallerinin küresel ortalama maliyeti milyonlarca dolar seviyesindedir. Özellikle kritik ve hassas kayıtların tutulduğu sağlık sektörü en yüksek maliyet riskine sahipken, eğitim ve benzeri sektörlerde maliyetler nispeten daha düşük gerçekleşmektedir.
> 
>   

## 2. Savunma Mimarisi ve Siber Saldırı Yaşam Döngüsü

### A. Ağ ve Uç Nokta Güvenlik Cihazları Sınıflandırması

|**Çözüm Türü**|**Cihaz / Yazılım**|**Temel Görev ve Çalışma Prensibi**|
|---|---|---|
|**Donanım (Hardware)**|**Firewall Appliance**|Önceden belirlenmiş kurallar dizisine göre bağlantılara izin verir veya engeller; ağa giriş-çıkışı sınırlar.|
|**Donanım (Hardware)**|**IDS (Intrusion Detection System)**|Ağ ve sistem üzerindeki yetkisiz girişleri ve ihlal teşebbüslerini algılar ve alarm üretir (Pasif).|
|**Donanım (Hardware)**|**IPS (Intrusion Prevention System)**|Tehditleri algıladığı anda bağlantıyı keserek saldırıyı aktif olarak engeller (Aktif).|
|**Donanım (Hardware)**|**VPN Concentrator**|Veri trafiğini şifreleyerek üçüncü taraflarca okunmasını ve değiştirilmesini önler; gizlilik ve bütünlüğü korur.|
|**Yazılım (Software)**|**Host Firewall**|Doğrudan uç nokta işletim sisteminde çalışarak o makineye gelen/giden trafiği denetler (Örn: Windows Defender Firewall, macOS Application Firewall).|
|**Yazılım (Software)**|**Antivirus (AV)**|Cihaza indirilen zararlı dosyaları tespit eder ve yürütülmesini durdurur.|

### B. Lockheed Martin Cyber Kill Chain (7 Adım)

Saldırganlar kurumsal bir ağa sızarken rastgele hareket etmez; aşamalı bir metodoloji izler:

  

1. **Keşif (Reconnaissance):** Hedef hakkında işletim sistemi, IP adresleri, çalışan servisler, kullanıcı adları ve e-postalar gibi olabildiğince çok bilgi toplama aşaması.
    
      
    
2. **Silahlanma (Weaponization):** Hedefe uzaktan erişim sağlamak amacıyla zararlı bir bileşen (exploit/payload) barındıran dosyanın hazırlanması.
    
      
    
3. **Teslimat (Delivery):** Silah haline getirilmiş dosyanın e-posta, web veya USB gibi yöntemlerle hedefe iletilmesi.
    
      
    
4. **İstismar (Exploitation):** Kullanıcı zararlı dosyayı açtığında zafiyetin tetiklenmesi ve zararlı kodun çalıştırılması.
    
      
    
5. **Yükleme (Installation):** Zararlı yazılımın (arka kapı/malware) kurban sisteme yerleşmesi ve kalıcı hale gelmesi.
    
      
    
6. **Komuta ve Kontrol (Command & Control - C2):** Zararlı yazılımın dışarıdaki saldırgan sunucusuyla iletişim kanalı kurarak uzaktan emir alabilir hale gelmesi.
    
      
    
7. **Hedef Üzerindeki Eylemler (Actions on Objectives):** Saldırganın asıl amacını gerçekleştirmesi (örneğin veri sızdırma / Data Exfiltration, şifreleme veya sistemleri yok etme).
    
      
    

> [!WARNING]
> 
> **Host Firewall ile Hardware Appliance Arasındaki Ayrım:**
> 
>   
> 
> Mülakatlarda sıkça sorulan klasik bir ayrımdır: Ağ seviyesindeki donanım güvenlik duvarları tüm alt ağı (subnet) dış dünyadan izole ederken; **Host Firewall (Windows Defender vb.)** doğrudan yerel işletim sisteminin çekirdeğinde veya servis katmanında çalışır ve aynı yerel ağ içindeki diğer komşu makinelerden (Lateral Movement) gelen trafiği de engelleyebilir.
> 
>   

## 3. Pratik Laboratuvar Saldırı Senaryosu ve Analist Refleksleri

Laboratuvarda hedef Linux sunucusuna yönelik Kill Chain adımları şu mantıkla yürütülmüştür:

  

```
┌────────────────────────────────────────────────────────────────────────────────────────┐
│                        LABORATUVAR İSTİSMAR VE ERİŞİM AKIŞI                            │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 1. RECON (Aktif Port Taraması):                                                        │
│    nmap MACHINE_IP ──► 21/tcp (FTP), 22/tcp (SSH), 80/tcp (HTTP) açık tespit edildi.   │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 2. ANONYMOUS FTP ERİŞİMİ & BİLGİ İFŞASI (Information Disclosure):                     │
│    ftp MACHINE_IP ──► Kullanıcı: anonymous ──► get secret.txt                          │
│    cat secret.txt ──► Parola elde edildi: ABC789xyz123                                 │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 3. SSH İLE ROOT OTURUMU & YETKİLİ ERİŞİM (Privileged Access):                          │
│    ssh root@MACHINE_IP ──► Tam yönetici (root) erişimi sağlandı.                      │
├────────────────────────────────────────────────────────────────────────────────────────┤
│ 4. HEDEF ÜZERİNDEKİ EYLEMLER (Actions on Objectives):                                  │
│    • /root/flag.txt ──► THM{FTP_SERVER_OWNED}                                          │
│    • /home/librarian/flag.txt ──► THM{LIBRARIAN_ACCOUNT_COMPROMISED}                   │
└────────────────────────────────────────────────────────────────────────────────────────┘
```

> [!TIP]
> 
> **Anonim FTP (Anonymous Login) Güvenlik Riski:**
> 
>   
> 
> FTP servislerinde `anonymous` kullanıcısıyla parolasız girişin açık bırakılması, en kritik yanlış yapılandırmalardan (Misconfiguration) biridir. Sistem yöneticilerinin yedekleme betiklerini (`backup.sh`), hassas şifreleri (`secret.txt`) veya sistem yapılandırma dosyalarını bu dizinde unutması, saldırganlara sıfır eforla ilk erişim ve kimlik bilgisi (Credential Leakage) sağlar.
> 
>   

## 4. Kullanılan Komutlar, Parametreler ve Çalışma Mantıkları

### A. Keşif (Reconnaissance) Komutları

Bash

```
# Hedef IP üzerindeki en yaygın 1000 TCP portunu tarar ve açık servisleri listeler
nmap MACHINE_IP
```

- `nmap`: Ağ haritalama ve port tarama aracı.
    
      
    
- `MACHINE_IP`: Taranacak kurban makinenin IP adresi. Çıktıda 21 (FTP), 22 (SSH) ve 80 (HTTP) portlarının açık olduğu belirlenmiştir.
    
      
    

### B. FTP İstemcisi Komutları

Bash

```
# 1. Hedef makinenin FTP servisine bağlantı başlatır
ftp MACHINE_IP

# 2. Giriş kullanıcı adı sorulduğunda girilen anonim kimlik
anonymous

# 3. FTP sunucusundaki mevcut dosya ve dizinleri listeler
ls

# 4. Sunucudaki belirtilen dosyayı yerel saldırgan makinesine indirir
get secret.txt

# 5. FTP oturumunu sonlandırır ve terminale döner
exit
# (Alternatif: bye)
```

### C. Linux Terminal ve Dosya İnceleme Komutları

Bash

```
# İndirilen metin dosyasının içeriğini ekrana basar (Kök kullanıcı parolasını açığa çıkarır)
cat secret.txt

# Bulunan parola ile hedef sunucuda root yetkisinde güvenli kabuk (SSH) oturumu açar
ssh root@MACHINE_IP

# Bulunulan aktif çalışma dizininin tam yolunu ekrana yazdırır (Print Working Directory)
pwd

# Dizin içeriğindeki dosya ve klasörleri listeler
ls

# Belirtilen hedef dizine geçiş yapar (Change Directory)
cd /home
cd librarian
```

## 5. Pratik Laboratuvar Soru-Cevap Anahtarı

|**Görev #**|**Soru**|**Doğru Cevap**|**Analiz Metodolojisi & İlgili Komut**|
|---|---|---|---|
|**T1.Q1**|What type of firewall is Windows Defender Firewall?|`Host Firewall`|İşletim sistemi içine gömülü yazılımsal güvenlik duvarı kategorisindedir.|
|**T2.Q1**|During which step of the Cyber Kill Chain does the attacker gather information about the target?|`Recon`|Saldırganın hedef hakkında bilgi topladığı ilk aşamadır (Reconnaissance).|
|**T3.Q1**|What is the password in the `secret.txt` file?|`ABC789xyz123`|FTP'den indirilen dosya `cat secret.txt` komutuyla okunarak elde edilmiştir.|
|**T3.Q2**|What is the content of the `flag.txt` in the `/root` directory?|`THM{FTP_SERVER_OWNED}`|SSH ile root girişi yapıldıktan sonra `/root` altında `cat flag.txt` ile okunmuştur.|
|**T3.Q3**|What is the content of the `flag.txt` in the `/home/librarian` directory?|`THM{LIBRARIAN_ACCOUNT_COMPROMISED}`|`cd /home/librarian` dizinine geçilerek `cat flag.txt` komutuyla elde edilmiştir.|

## 6. Tekrar Kategorisi ve Pazar Stratejisi

- **Kategori Sınıfı:** 🟢 **Yeşil (Genel Kültür / Temel Tekrar / Tekrar Etme)**
    
      
    
- **Gerekçe:** Cyber Kill Chain aşamaları ve donanım/yazılım güvenlik duvarı ayrımları temel konulardır; lab uygulaması (anonim FTP'den şifre alıp SSH ile bağlanma) standart bir giriş seviyesi senaryodur.
    
      
    
- **Pazar Günü 3 Dakikalık Tekrar Stratejisi:**
    
      
    1. **1. Dakika:** Lockheed Martin Kill Chain'in 7 basamağını sırasıyla hatırla: _Recon $\rightarrow$ Weaponization $\rightarrow$ Delivery $\rightarrow$ Exploitation $\rightarrow$ Installation $\rightarrow$ C2 $\rightarrow$ Actions on Objectives._
        
          
        
    2. **2. Dakika:** Cihaz ayrımlarını zihninde canlandır: Ağ geçidindeki kutu = _Firewall Appliance / IDS / IPS_; işletim sistemindeki yerleşik koruma = _Host Firewall (Windows Defender)_.
        
          
        
    3. **3. Dakika:** Zafiyet mantığını özetle: _Ağdaki servislerde anonim girişlerin (FTP anonymous vb.) açık bırakılması ve hassas kimlik bilgilerinin düz metin olarak paylaşılması bir "Information Disclosure" zafiyetidir ve doğrudan sistemin ele geçirilmesine yol açar._
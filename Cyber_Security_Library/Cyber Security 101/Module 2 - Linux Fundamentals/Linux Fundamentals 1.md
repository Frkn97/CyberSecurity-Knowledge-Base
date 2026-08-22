# 📓THM - Linux Fundamentals Part 1 (Temel Dosya Sistemi, Terminal Komutları ve Kabuk Operatörleri)

**Tags:** #Linux #LinuxFundamentals #Terminal #CommandLine #FileStructure #Grep #BlueTeamFundamentals

## 🎛️ 1. Giriş: Linux Nedir ve Neden Önemlidir?

Linux; akıllı arabalardan süper bilgisayarlara, kurumsal web sunucularından IoT cihazlarına ve siber güvenlik araçlarına (Kali Linux) kadar dijital dünyayı yöneten açık kaynaklı, güvenli, hafif (lightweight) ve kararlı bir işletim sistemidir. Windows'un aksine, grafik arayüzü (GUI) olmadan sadece **Terminal (Komut Satırı)** üzerinden çok daha yüksek performansla yönetilebilir.

- **İlk Çıkış Yılı:** Linux çekirdeği (Kernel), Linus Torvalds tarafından ilk kez **1991** yılında piyasaya sürülmüştür.
    
- **Dağıtımlar (Distros / Flavours):** Linux tabanlı Kali Linux, Ubuntu, Fedora, Debian ve Arch gibi farklı ihtiyaçlara odaklanmış sürümler mevcuttur. TryHackMe laboratuvar altyapısı varsayılan olarak **Ubuntu** dağıtımını kullanır.
    

## 🛠️ 2. İlk Temel Komutlar ve Kullanıcı Keşfi

Terminal ile etkileşime girerken koşturulan en temel iki komut:

- **`echo`:** Verilen bir metni veya değişkeni doğrudan terminal ekranına yazdırır (Farklı dillerdeki `print` karşılığı). Birden fazla kelime içeren dizgilerde çift tırnak (`"..."`) kullanılması sentaks standardıdır. (Örn: `echo TryHackMe`).
    
- **`whoami`:** Terminalde o esnada aktif olarak oturum açmış, komut koşturan güncel kullanıcı adını döndürür. (TryHackMe makinelerindeki varsayılan kullanıcı: `tryhackme`).
    

## 📂 3. Dosya Sistemi ve Navigasyon (Interacting With Filesystem)

Linux'ta klasör yapısı ağaç şeması gibidir ve her şey kök dizinden (`/`) başlar. Sistemde gezinmek için şu komutlar kas hafızası haline getirilmelidir:

- **`pwd` (Print Working Directory):** Tam olarak hangi klasörün içinde bulunduğunuzun açık ve net tam yolunu (**Absolute Path**) ekrana basar.
    
- **`ls` (Listing):** Aktif bulunulan dizinin içerisindeki tüm dosya ve klasörleri listeler.
    
- **`cd` (Change Directory):** Klasörler arasında geçiş yapmayı, dizin değiştirmeyi sağlar (Örn: `cd folder4`).
    
- **`cat` (Concatenate):** Belirtilen dosyanın içeriğini terminal ekranına doğrudan döker. Konfigürasyon, kaynak kod veya parola dosyalarını hızlıca okumak için kritiktir.
    

## 🔍 4. Dosya ve İçerik Arama Komutları ( find & grep )

Dehasal dosya sistemleri içinde aranan veriyi saniyeler içinde bulmayı sağlayan iki büyük operasyonel komut:

### A) `find` Komutu (Dosya/Klasör Avcısı)

Belirtilen kriterlere göre dosya sisteminde arama yapar. İki yaygın parametresi mevcuttur: `-type` (tür) ve `-name` (isim).

- `d` parameteresi klasörleri (**directory**), `f` parametresi ise normal dosyaları (**file**) hedefler:
    
    Bash
    
    ```
    # Sadece note.txt isimli dosyayı arar:
    find -type f -name note.txt
    
    # Wildcard (*) kullanarak tüm .txt uzantılı dosyaları bulur:
    find -name *.txt
    ```
    

```

### B) `grep` Komutu (İçerik Süzgeci)
Dosyaların içerisindeki metinleri, log satırlarını veya belirli örüntüleri (pattern) tarayıp ayıklamak için kullanılır.
* **`wc` (Word Count):** `grep` ile filtrelenen veya doğrudan hedef alınan bir dosyadaki toplam satır, kelime ve karakter sayısını ölçen hayati yardımcı komuttur.

---

## ⚡ 5. Kabuk Operatörleri (Shell Operators)
Komutları birbirine bağlayan, arka plana atan veya çıktıları dosyalara yönlendiren hız çarpanlarıdır:

| Operator | Teknik İşlevi | Operasyonel Örnek |
| :--- | :--- | :--- |
| **`&`** | Komutu terminali kilitlemeden **arka planda (background)** çalıştırır. | `cp buyuk_log_dosyasi.log /backup &` |
| **`&&`** | İki komutu bağlar. **2. komut sadece 1. komut başarıyla biterse** çalışır. | `cd folder4 && cat note.txt` |
| **`>`** | Çıktı yönlendiricidir. Komut çıktısını dosyaya **üzerine yazarak (overwrite)** kaydeder. | `echo password123 > passwords` |
| **`>>`** | Çıktı yönlendiricidir. Veriyi silmeden dosyanın **en sonuna ekleme (append)** yapar. | `echo tryhackme >> passwords` |

---

## 🛠️ 6. Laboratuvar Pratik Çıktıları ve Nokta Atışı Cevaplar

Odadaki interaktif Linux makinesinde uygulanan görevler ve resmi çözümler:

* **Soru 1:** Linux işletim sisteminin ilk sürümü hangi yılda yayınlanmıştır?
    * *Yanıt:* **`1991`**
* **Soru 2:** Terminale "TryHackMe" metnini yazdırmak için hangi komut kullanılır?
    * *Yanıt:* **`echo TryHackMe`**
* **Soru 3:** İlk deploy edilen THM laboratuvar makinesindeki aktif kullanıcı adı nedir?
    * *Yanıt:* **`tryhackme`**
* **Soru 4:** Kullanıcının ev (home) dizininde toplam kaç adet klasör/folder bulunmaktadır?
    * *Yanıt:* **`4`**
* **Soru 5:** Bu 4 klasörden hangisinin içerisinde okunabilir bir dosya yer almaktadır?
    * *Yanıt:* **`folder4`**
* **Soru 6:** `folder4` içerisindeki gizli dosyanın `cat` ile okunan içeriği nedir?
    * *Yanıt:* **`Hello World!`**
* **Soru 7:** Bu dosyanın bulunduğu dizinin tam sistem yolu (path) nedir?
    * *Yanıt:* **`/home/tryhackme/folder4`**
* **Soru 8:** `access.log` dosyası içinde "THM" ön ekine (prefix) sahip bayrağı bulan `grep` sorgusunun getirdiği flag nedir?
    * *Yanıt:* **`THM{ACCESS}`**
* **Soru 9:** Bir komutu arka planda çalıştırmak için hangi operatör kullanılır?
    * *Yanıt:* **`&`**
* **Soru 10:** "passwords" isimli dosyanın içeriğini tamamen silip yerine "password123" yazan komut nedir?
    * *Yanıt:* **`echo password123 > passwords`**
* **Soru 11:** "passwords" dosyasındaki eski veriyi koruyarak altına "tryhackme" kelimesini ekleyen komut nedir?
    * *Yanıt:* **`echo tryhackme >> passwords`**

***

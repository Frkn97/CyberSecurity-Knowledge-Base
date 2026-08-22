# 📓 THM - The Game (Game Hacking & Binary Static Analysis)

**Tags:** #BlueTeam #MalwareAnalysis #ReverseEngineering #StaticAnalysis #LinuxMint #GHex #Strings #Grep #SOC_L1 #TryHackMe #CTF #Walkthrough

## 🏗️ Mimari Yapı

**The Game** (Hackfinity Battle 2025 CTF etkinliği oda konsepti), oyuncudan klasik bir Tetris oyunu (`Tetrix.exe`) içerisine gizlenmiş ve şifrelendiği iddia edilen kritik verileri/bayrağı (flag) açığa çıkarmasını bekleyen bir **Game Hacking / Binary Analiz** senaryosudur.

Bir ikili dosya (binary / executable) derlenirken (compile), geliştirici tarafından kod içine gömülen metin sabitleri (hardcoded strings), düz metin (plain-text) karakter dizileri halinde dosyanın bellek sektörlerine yazılır. Eğer bu veriler özel bir obfuskasyon (kod karıştırma), şifreleme veya paketleyici (Packer / UPX vb.) mekanizmasıyla gizlenmemişse, uygulama çalıştırılmadan statik analiz araçlarıyla doğrudan okunabilir.

```
                  [ Statik İkili Dosya Analizi (Static Binary Analysis) ]
                                            │
         ┌──────────────────────────────────┴──────────────────────────────────┐
         ▼                                                                     ▼
[ GUI Tabanlı Hex Editörler ]                                       [ CLI Tabanlı Dizecik Araçları ]
- GHex (Linux Mint) / Bless Hex Editor / HxD                        - `strings` + `grep` Kombinasyonu
- Verinin ASCII / Hex karşılığını aratır.                            - İkili verideki okunabilir karakterleri süzer.
```

## 💻 Tüm Komutlar ve Parametre Kırılımları

### A. Komut Satırı (CLI) Tabanlı Statik Analiz Komutları

- **İkili Dosya İçinden Okunabilir Metinleri Süzme:**
    
    Bash
    
    ```
    strings Tetrix.exe | grep "THM"
    ```
    
    - **`strings`:** İkili (binary) veya çalıştırılabilir (executable) bir dosya içerisindeki yazdırılabilir (printable) ASCII ve UNICODE metin karakter dizilerini (en az 4 karakter uzunluğundaki dizecikleri) tek tek ayıklayıp standart çıktıya (stdout) basan temel Linux/Unix komutudur.
        
    - **Pipe Operatörü (`|`):** `strings` komutunun ürettiği tüm metin dökümünü (çıktısını), sağ tarafındaki arama motoruna (`grep`) girdi (input) olarak aktarır.
        
    - **`grep "THM"`:** Sol taraftan gelen binlerce satırlık metin bloğu arasından yalnızca tırnak içinde belirtilen **"THM"** desenini (pattern) içeren satırları filtreleyip ekrana basar.
        

> ⚠️ **UYARI - `grep` KULLANIMINDA YAPILAN YAYGIN MANTIKSAL HATA:**
> 
> Doğrudan `grep Tetrix.exe "THM"` veya `grep "THM" Tetrix.exe` komutunu çalıştırmak ikili (binary) dosyalarda **başarısız olur** veya terminale `Binary file Tetrix.exe matches` uyarısı basar. Çünkü `grep` bir metin işleyicidir; ham ikili koddaki (0 ve 1'ler) ASCII karakterleri tek başına ayıklayamaz. İkili dosyada arama yapabilmek için veri önce **`strings`** aracıyla metne dönüştürülmeli, ardından `grep`'e aktarılmalıdır!

## 📊 Teknik Veriler ve Sınıflandırmalar

### A. Statik Binary Analiz Araçları Karşılaştırma Matrisi

|**Araç Adı**|**Platform / Arayüz**|**Çalışma Mantığı ve Kullanım Amacı**|**Sektörel Analist Avantajı**|
|---|---|---|---|
|**GHex**|Linux (GUI)|GNOME tabanlı grafiksel Hex editörüdür. Dosyaları bayt bayt incelemeyi sağlar.|**Linux Mint** ortamında görsel arama paneli (`Ctrl + F`) ile ASCII/Hex dizeciklerini anlık yakalar.|
|**Bless Hex Editor**|Linux / Windows (GUI)|GHex benzeri açık kaynaklı grafiksel hex analiz editörüdür.|Ham bayt bloklarını görselleştirmek ve adres (offset) takibi yapmak için kullanılır.|
|**`strings` + `grep`**|Linux Terminal (CLI)|Komut satırı tabanlı, boru hattı (pipeline) ile çalışan minimalist analiz ikilisi.|Devasa dosyaları saniyeler içinde tarar; GUI açmaya gerek kalmadan otomasyon script'lerine entegre edilir.|
|**HxD**|Windows (GUI)|Windows ortamındaki en popüler hafif hex ve bellek editörüdür.|Windows tabanlı statik analizlerde dosya başlığı (magic bytes) doğrulamak için kullanılır.|

## 🧠 Uzman Notları ve Kritik Uyarılar

> 🛡️ **ANALİST REFLEKSİ - DUAL-BOOT VE LINUX MINT AVANTAJI:**
> 
> Windows ortamında bu challenge çalıştırıldığında bir Tetris oyunu (`Tetrix.exe`) olarak açılır. Bir siber güvenlik analisti, kaynağı belirsiz veya CTF ortamındaki bir yürütülebilir dosyayı kendi ana sisteminde **asla doğrudan çalıştırmamalıdır**! **Linux Mint** üzerinde çalışırken Windows çalıştırılabilir dosyaları (PE / `.exe`) yerel olarak yürütülmediği için izole bir analiz avantajı sağlar. Dosyayı çalıştırmadan yerel Linux araçlarıyla (GHex veya `strings`) incelemek en güvenli statik analiz yaklaşımıdır.

> 💡 **SEKTÖREL İPUCU - C2 VE HARDCODED STRING TESPİTİ:**
> 
> Gerçek dünya zararlı yazılımlarında (Malware), saldırganlar bazen Komuta Kontrol (C2) IP adreslerini, API anahtarlarını veya şifreleme parolalarını kodun içine gömerler (Hardcoded Data). Bir SOC analisti olarak şüpheli bir binary geldiğinde koşturulacak ilk statik adım `strings <file> | grep -iE "http|https|ip|key"` olmalıdır.

## 🗃️ Soru-Cevap Anahtarı

|**Pratik / Laboratuvar Sorusu**|**Doğru Cevap**|
|---|---|
|**[LAB]** Tetris oyunu (`Tetrix.exe`) binary dosyası statik olarak analiz edildiğinde ortaya çıkarılan gizli flag nedir?|`THM{I_CAN_READ_IT_ALL}`|
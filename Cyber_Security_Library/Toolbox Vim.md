# 📓THM - Toolbox: Vim (Metin Düzenleme Mimarisi, Mod Yönetimi ve Gelişmiş Komut Setleri)

**Tags:** #Linux #Vim #TextEditors #CommandLine #Automation #SysAdmin #BlueTeamFundamentals

## 🎛️ 1. Giriş: Siber Güvenlikte Neden VIM?

Siber güvenlik operasyonlarında, sızılan bir sunucuda (Post-Exploitation), adli bilişim analizi yapılan sıkılaştırılmış bir Linux sisteminde veya kısıtlı bir ters kabuk (**Reverse Shell**) bağlantısında, grafik arayüzü (GUI) veya kullanıcı dostu hafif editörler (Nano, gedit vb.) sistemde bulunmayabilir.

**Vim (Vi Improved);** fareye kesinlikle ihtiyaç duymadan, tamamen klavye üzerindeki kısayollar ve modlar vasıtasıyla devasa log dosyalarını taramanızı, konfigürasyon dosyalarını milimetrik düzenlemenizi sağlayan, her Unix/Linux sisteminde yerleşik olarak gelen en güçlü terminal metin editörüdür.

### 📦 Farklı Linux Dağıtımlarında Yükleme Komutları

Eğer minimal bir imajda Vim kurulu değilse, yetkili kullanıcı yetkisiyle şu komutlar koşturulur:

- **Debian / Ubuntu Tabanlı:** `sudo apt install vim`
    
- **Arch Linux Tabanlı:** `sudo pacman -S vim`
    
- **Fedora / RHEL Tabanlı:** `sudo dnf install vim-enhanced`
    

## 🏗️ 2. VIM'in 3 Çekirdek Mod Mimarisi (Vim Modes)

Vim, standart metin editörlerinden farklı olarak modlu bir yapıya sahiptir. Klavyedeki harflerin işlevi bulunulan moda göre tamamen değişir:

1. **Command Mode (Komut Modu):** Vim ilk açıldığında devreye giren **varsayılan (default)** moddur. Bu modda klavyedeki harfler yazı yazmak için değil; yön bulmak, metin kopyalamak, silmek veya arama yapmak için birer sistem komutu olarak çalışır.
    
2. **Insert Mode (Yazma Modu):** Belge içerisine gerçek anlamda metin, kod veya kural yazılan moddur. Ekranda `-- INSERT --` ibaresi görünür.
    
3. **Visual Mode (Görsel Seçim Modu):** Fareyle sürüklüyor gibi, klavye ile metinleri blok halinde seçmek ve sadece seçili bölge üzerinde toplu bir işlem/komut çalıştırmak için kullanılır.
    

## 🧭 3. Mod Geçişleri, Navigasyon ve Metin Giriş Çeşitleri

### A) Modlar Arası Kritik Geçişler

- **Komut Modundan ➡️ INSERT Moduna Geçiş:** `i` tuşuna basılarak tetiklenir (Harfi yazar yazmaz `typing` moduna geçilir).
    
- **INSERT Modundan ➡️ Komut Moduna Geri Dönüş:** **`Esc`** tuşuna basılarak sağlanır. _(Herhangi bir komut çalıştırmadan önce mutlaka Esc ile komut moduna dönülmelidir)._
    

### B) Klavyeyle Yön Bulma (Yön Tuşları Yerine Klasik Cluster)

Komut modundayken yön tuşlarına el uzatarak vakit kaybetmemek için klavyenin ana harf kümesi kullanılır:

- **`h`** ➡️ İmleci bir karakter **Sola** kaydırır.
    
- **`l`** ➡️ İmleci bir karakter **Sağa** kaydırır.
    
- **`k`** ➡️ İmleci bir satır **Yukarı** kaydırır.
    
- **`j`** ➡️ İmleci bir satır **Aşağı** kaydırır.
    
- **`w`** ➡️ İmleci bir sonraki **kelimenin başlangıcına** zıplatır (jump to start of a word).
    
- **`e`** ➡️ İmleci mevcut veya bir sonraki **kelimenin son karakterine** zıplatır (jump to end of a word).
    

### C) Metin Giriş Varyasyonları (Insert & Append İncellikleri)

Yazma moduna geçiş şekliniz, imlecin konumuna göre hız kazanmanızı sağlar:

- **`i`** ➡️ Yazma modunu imlecin **bulunduğu karakterin hemen önünde** başlatır.
    
- **`I`** ➡️ İmleci doğrudan **bulunulan satırın en başına** fırlatır ve yazma moduna geçer.
    
- **`a`** ➡️ Yazma modunu imlecin **bulunduğu karakterin bir arkasında (append)** başlatır.
    
- **`A`** ➡️ İmleci doğrudan **bulunulan satırın en sonuna** fırlatır ve yazma moduna geçer.
    
- **`o`** ➡️ **En Popüler Kısayol.** Mevcut satırın hemen **altına yeni, boş bir satır açar** ve yazma moduna geçer.
    

## 💾 4. Dosya Yönetimi: Güvenli Kaydetme ve Çıkış Komutları

Komut modundayken `:` (iki nokta üst üste) sembolüne basılarak çalıştırılan hayati alt satır komutları:

- **`:w`** ➡️ Değişiklikleri dosyaya yazar/kaydeder (**Write**) ama editörden çıkmaz.
    
- **🚨 `:w !sudo tee %` (Root Bypass Kuralı):** Bir dosyayı standart kullanıcıyla açıp düzenledikten sonra kaydetme aşamasında yetki hatası (`Permission Denied`) alırsanız; Vim'i kapatıp emeklerinizi kaybetmek yerine bu komutu çalıştırırsınız. Vim'den çıkmadan, arka planda root yetkisiyle (`sudo`) o anki dosyaya (`%`) verileri zorla yazar.
    
- **`:wq`** ➡️ Değişiklikleri kaydeder ve editörden temiz bir şekilde çıkış yapar (**Write and Quit**).
    
- **`:q`** ➡️ Eğer dosyada hiçbir değişiklik yapılmadıysa editörü kapatır (**Quit**).
    
- **`:q!`** ➡️ Yapılan değişiklikleri kaydetmeden, her şeyi çöpe atarak editörden zorla çıkış yapar (**Force Quit**).
    
- **`:wqa`** ➡️ Vim üzerinde açık olan tüm aktif sekmeleri/sekmelerdeki dokümanları tek seferde kaydedip kapatır.
    

## ✂️ 5. Pano İşlemleri: Kopyalama, Kesme ve Yapıştırma (Yank & Delete)

Vim mimarisinde kopyalama işlemine **Yank**, silme/kesme işlemine ise **Delete** denir. Komut modunda çalışırlar:

- **`yy`** ➡️ İmlecin bulunduğu **1 satırı komple kopyalar** (Yank line).
    
- **`2yy`** ➡️ Başına sayı çarpanı ekleyerek imleçten itibaren **2 satırı birden kopyalar**.
    
- **`y$`** ➡️ İmlecin bulunduğu karakterden, o **satırın en sonuna kadar olan kısmı** kopyalar.
    
- **`p`** ➡️ Panoya (clipboard) alınmış içeriği, imlecin bulunduğu yerin **bir alt satırına / arkasına yapıştırır** (Paste after).
    
- **`P`** ➡️ Panoya alınmış içeriği, imlecin bulunduğu yerin **bir üst satırına / önüne yapıştırır** (Paste before).
    
- **`dd`** ➡️ İmlecin bulunduğu **1 satırı komple keser / siler** (Delete line).
    
- **`2dd`** ➡️ İmleç dahil alt alta olan **2 satırı birden keser / siler**.
    
- **`D`** ➡️ **Kritik Kısayol (`d$`).** İmlecin durduğu karakterden başlayarak o **satırın sonuna kadar olan tüm içeriği keser/siler**.
    
- **``x`** ➡️ İmlecin tam olarak **üzerinde durduğu tek bir karakteri siler/keser**.
    

## 🔍 6. Düzenli Kelime Arama ve Küresel Değiştirme

Büyük log dökümlerinde veya kod satırlarında hedef odaklı arama yapmak için koşturulan komutlar:

- **`/pattern`** ➡️ Belirtilen kelimeyi veya örüntüyü imleçten **ileriye doğru (forward)** arar. (Örn: `/192.168.1.1`).
    
- **`?pattern`** ➡️ Belirtilen kelimeyi veya örüntüyü imleçten **geriye doğru (backward)** arar.
    
- **`n`** ➡️ Yapılan arama yönünde bir sonraki (**next**) eşleşen kelimeye zıplar.
    
- **`N`** ➡️ Yapılan arama yönünün tam **tersi istikametinde** bir önceki eşleşen kelimeye zıplar.
    
- **`:%s/old/new/g`** ➡️ **Regex Değiştirme Gücü.** Tüm döküman içindeki (`%`) eski (`old`) kelimelerini bulup yeni (`new`) kelimesiyle küresel olarak (`g` - globally) değiştirir.
    
- **`:vimgrep /pattern/ *.txt`** ➡️ Belirtilen uzantıdaki tüm dosyalar içinde dahili `grep` motorunu çalıştırarak spesifik bir kelimeyi veya atağı aratır.
    

## 🛡️ 7. Mavi Takım / SOC Analisti Operasyonel Özet Notu

> 💡 **Mülakat ve Operasyon Manifestosu:** "Kısıtlı yetkilere sahip veya sıkılaştırılmış Unix/Linux kurumsal sunucularında grafik arayüzü (GUI) veya kullanıcı dostu Nano editörü bulunmayabilir. Bu acil durum senaryolarında sistem mimarisinde dahili olarak gelen VIM editörünü kullanırım. Operasyonel seviyede VIM mod yönetimine (`i` ile Insert modu, `Esc` ile Komut modu), hızlı kelime ve satır navigasyonuna (`w`, `e`, `o`), satır kopyalama ve silme süreçlerine (`yy`, `dd`, `D`) ve güvenli/zorla çıkış senaryolarına (`:wq`, `:q!`, `:w !sudo tee %`) tam operasyonel aşinalığım bulunmaktadır."
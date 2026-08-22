# 📓THM - Regular Expressions (RegEx Formülleri, Log Filtreleme ve Metinsel Örüntü Avcılığı)

**Tags:** #RegEx #RegularExpressions #LogAnalysis #SIEM #ThreatHunting #PatternMatching #BlueTeamFundamentals

## 🧱 1. Giriş: Siber Güvenlikte Neden RegEx?

Mavi Takım analistleri (SOC Analyst) veya adli bilişim uzmanları (Forensics) her gün milyonlarca satırdan oluşan devasa log çorbalarını incelemek zorundadır. Bu kaotik veri yığınları içinden şüpheli bir IP adresini, sızdırılmış bir e-postayı veya zararlı bir URL örüntüsünü el ile aramak imkansızdır.

**Regular Expressions (RegEx / Düzenli İfadeler);** metinler içerisinde belirli kurallara uyan karakter dizilimlerini milimetrik olarak tespit etmemizi, süzmemizi ve ayıklamamızı sağlayan evrensel bir arama ve şablon eşleştirme dilidir.

## 📐 2. Karakter Kümeleri ve Hariç Tutma Mimarisi (Charsets)

Köşeli parantezler `[ ]` Regex'in en temel süzgecidir. İçeriye yazılan karakter havuzundan **sadece tek bir karakter** yakalamak için kullanılır.

- **`[cog]`** ➡️ İçerideki harflerden herhangi birini tek başına yakalar (`c`, `o` veya `g`).
    
- **`[cfh]at`** ➡️ `cat`, `fat` veya `hat` kelimelerinin tamamını vuran süzgeçtir.
    
- **`[CcHh]at`** ➡️ Büyük/küçük harf duyarlılığını aşarak `Cat`, `cat`, `Hat` ve `hat` kelimelerini yakalar.
    
- **`[Ff]ile[1–9]`** ➡️ Tire (`-`) işareti aralık belirtir. `File1` ile `file9` arasındaki tüm dizilimleri kapsar.
    
- **`[^7]` (Hariç Tutma Kapısı):** Şapka işareti (`^`) köşeli parantezin **İÇİNDEYSE**, "bu karakter hariç her şey gelsin" anlamına gelen bir olumsuzlama filtresidir.
    
    - _Lab Örneği:_ **`[Ff]ile[^7]`** ➡️ Listeden `File7` hariç tüm dosya isimlerini yakalar.
        
    - _Lab Örneği:_ **`[^r]ats?`** ➡️ Baş harfi `r` olmayan, arkasından `at` gelen kelimeleri yakalar (`bat`, `hat` geçer; `rat` engellenir).
        

## 🔮 3. Joker Karakterler, Opsiyoneller ve Kaçış Kuralları

- **Nokta (`.`) [Açgözlü Joker]:** Satır sonu (line break) hariç dünyadaki **herhangi bir tek karakterin** (harf, sayı, sembol, boşluk) yerini tutar.
    
    - _Lab Örneği:_ **`...[^n-z]`** ➡️ İlk 3 karakteri ne olursa olsun, son 4. karakteri küçük `n` ile `z` harfleri arasında olmayan tüm 4 karakterlik dizilimleri vurur.
        
- **Soru İşareti (`?`) [Opsiyonel Harf]:** Önüne geldiği karakteri isteğe bağlı yapar. "Bu karakter burada ya 0 kez vardır ya da 1 kez" anlamına gelir.
    
    - _Lab Örneği:_ **`[Cc]ats?`** ➡️ Kelimenin sonundaki `s` takısı olsa da olur olmasa da; hem `Cat` hem `cats` yakalanır.
        
- **Ters Eğik Çizgi (`\`) [Kaçış Karakteri]:** Nokta (`.`), soru işareti (`?`), dolar (`$`) gibi Regex'in kendisine ait özel fonksiyonel sembolleri metin içerisinde "gerçek, ham haliyle" aramak istiyorsak, soluna `\` koyarak onları etkisiz hale getiririz.
    
    - _Lab Örneği:_ **`cat\.xyz`** ➡️ Buradaki `\.` ifadesi herhangi bir karakteri değil, sadece gerçek bir nokta işaretini arar.
        

## 🐧 4. Meta Karakterler (Sihirli Kısıtlayıcılar)

Uzun uzun `[0-9]` veya `[a-zA-Z]` yazarak filtre alanını şişirmemek için kullanılan global kısayollardır. Küçük harfler olumluyu, büyük harfler tam tersi olan olumsuz kümelemeyi temsil eder:

|Meta Karakter|Teknik Görevi|Gerçek Karşılığı / Regex Karşılığı|
|---|---|---|
|**`\d`** _(Digit)_|Herhangi bir tek **rakamı** yakalar.|`[0-9]` ile birebir aynıdır.|
|**`\D`**|Rakam **olmayan** her şeyi yakalar.|`[^0-9]` (Harfler, semboller, boşluklar).|
|**`\w`** _(Word)_|Herhangi bir alfanümerik karakteri yakalar.|`[a-zA-Z0-9_]` (**Alt çizgi dahildir**).|
|**`\W`**|Alfanümerik olmayanları (özel sembolleri) vurur.|`!`, `@`, `#`, `$`, `~` gibi tüm semboller.|
|**`\s`** _(Space)_|Herhangi bir **boşluk** karakterini yakalar.|Boşluk tuşu (Space), Tab veya alt satır (`\n`).|
|**`\S`**|Boşluk **olmayan** geri kalan her şeyi yakalar.|Harfler, sayılar ve tüm semboller bir aradadır.|

## 🔄 5. Repetitions (Tekrarlayıcı ve Adet Belirleyiciler)

Bir karakterin veya meta karakter grubunun yan yana kaç defa geleceğini belgelendirmek için kullanılır:

- **`*` (Asterisk / Yıldız):** Önündeki karakter veya grup **0 ya da daha fazla kez** tekrarlayabilir (Ucu açıktır).
    
- **`+` (Artı Sembolü):** Önündeki karakter veya grup **en az 1 kez** veya daha fazla kez tekrarlamak zorundadır.
    
    - _Lab Örneği:_ **`kali\s+tools`** ➡️ Araya 1 tane boşluk da girse, 10 tane boşluk da girse (`\s+`) hepsini tek seferde yutar.
        
    - _Lab Örneği:_ **`\.?\w+`** ➡️ Başında nokta olsa da olmasa da (`\.?`) tüm kelime ve dosya isimlerini (`.bash_rc`, `note1`) yakalar.
        
- **Süslü Parantez `{ }` [Milimetrik Adet Kilidi]:**
    
    - `{n}` ➡️ Tamı tamına n defa tekrarlayacak (Örn: `cats{4}` ➡️ `catssss`).
        
    - `{min,max}` ➡️ En az ve en fazla aralığını sabitler (Örn: `\d{1,2}` ➡️ En az 1, en fazla 2 basamaklı sayılar).
        

## 🧭 6. Satır Sınırları, Mantıksal Seçimler ve Gruplama (Advanced)

- **`^` (Satır Başı):** Eğer şapka işareti köşeli parantezin dışında, formülün en başındaysa **"Satır kesinlikle bu örüntüyle başlamalı"** şartı koşar.
    
    - _Lab Örneği:_ **`^username:\s`** ➡️ Satırın tam başlangıcında `username:` arar.
        
    - _Lab Örneği:_ **`^\D`** ➡️ Rakam ile **başlamayan** tüm satırları filtreler.
        
- **`$` (Satır Sonu):** **"Satır kesinlikle bu örüntüyle bitmeli"** şartı koşar.
    
    - _Lab Örneği:_ **`EOF\$$`** ➡️ Satırın tam sonu `EOF$` ile bitmek zorundadır. (İlk `$` kaçış karakteriyle korunmuştur).
        
- **Parantez `( )` ve Boru `|` (Gruplama ve VEYA):** Mantıksal seçimler ve veriyi alt gruplara ayırmak için kullanılır.
    
    - _Lab Örneği:_ **`I use (nano|vim)`** ➡️ "I use nano" veya "I use vim" cümlelerinin ikisini birden yakalar.
        

## 📊 7. Laboratuvar Siber Güvenlik Filtreleri ve Tam Çıktılar

Odadaki karmaşık eşleştirme görevlerini başarıyla tamamlayan nokta atışı formül çıktıları:

- **Görev 1 (Dosya Taraması):** `ab0001`, `abc1000`, `cba0110` gibi harfle başlayıp ikilik sayıyla bitenleri meta karakter kullanmadan yakalayan formül:
    
    - ➡️ **`[abc]{1,3}[01]{4}`**
        
- **Görev 2 (Log Sembol Avcılığı):** `notes~`, `stuff@`, `gtfob#`, `lmaoo!` gibi 5 harfli ve sonu sembolle biten kelimeleri avlayan filtre:
    
    - ➡️ **`\w{5}\W`**
        
- **Görev 3 (Tırnak İçi Karışık Trafik):** `"2f0h@f0j0%! a)K!F49h!FFOK"` tırnak içindeki boşluklu/sembollü karmaşık dizilimi yutan formül:
    
    - ➡️ **`\S*\s*\S*`**
        
- **Görev 4 (Güvenli Parola Kontrolü):** "Password:" ile başlayan ve ardından `0` rakamı içermeyen 10 karakter gelen satırları bulan filtre:
    
    - ➡️ **`Password:[^0]{10}`**
        
- **Görev 5 ($ İşaretli Log Avı):** Başında `$`, ortasında tek rakam, ardından tekrar `$`, sonunda ise boşluksuz karakterler olan satırları seçen formül:
    
    - ➡️ **`\$\d\$\S+`**
        

### 🎯 8. Mavi Takım Başyapıt Formülleri (Mülakat Sırları)

- 🔥 **Küresel IPv4 Adresi Yakalayıcı:**
    

(\d{1,3}.){3}\d{1,3}

```
    *   *Analizi:* 1 ile 3 basamaklı bir sayının ardından nokta gelme durumunu (`\d{1,3}\.`) tam 3 kez tekrarla (`{3}`) ve sonuna noktasız 1-3 basamaklı son bir sayı grubu ekle. `192.168.1.1` gibi tüm IP dünyasını tek satırda havada yakalar.
*   📦 **E-Posta Ayrıştırıcı ve Yakalama Grupları (Capturing Groups):**
    ```regex
(\w+)@(\w+)\.com
```

```
*   *Analizi:* `hello@tryhackme.com` trafiği geldiğinde parantezler devreye girer. 1. grup (`\w+`) kullanıcı adını (`hello`), 2. grup ise domain adını (`tryhackme`) hafızaya alır ve veritabanına ayrı kolonlar olarak kaydetmemizi sağlar.
```


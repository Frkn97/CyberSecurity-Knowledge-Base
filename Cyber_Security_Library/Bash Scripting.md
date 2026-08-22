# 📓THM - Bash Scripting (Unix Kabuk Programlama, Otomasyon ve Sistem Mantık Kontrolleri)

**Tags:** #Linux #BashScripting #Automation #Variables #Arrays #Conditionals #ForLoops #Functions #BlueTeamFundamentals

## 🚨 EN KRİTİK ALTIN KURAL (Windows vs Linux Tuzağı)

Kodları asla Windows ortamındaki Notepad, VS Code veya PowerShell gibi editörlerde yazıp Linux'a atmaya çalışmayın! Windows satır sonlarında `CRLF` (`\r\n`) kullanırken, Linux sadece `LF` (`\n`) kullanır. Windows'ta yazılan bir script Linux'ta çalıştırıldığında görünmeyen `\r` karakterleri yüzünden kod doğru olsa bile **"Bad Interpreter"** hatası verir ve patlar.

- **Doğru Yöntem:** Kali Linux terminalini açın, `nano script.sh` veya `vim script.sh` komutuyla doğrudan Linux içinde temiz bir dosya oluşturup kodları orada yazın.
    

## 🎛️ 1. Giriş: Bash ve Shell Kavramı

- **Shell (Kabuk):** Bilgisayar mimarisinde kullanıcının veya diğer programların işletim sisteminin çekirdeğine (**Kernel**) erişmesini, komut göndermesini sağlayan ve içerisi ile dışarısı arasındaki sınırı belirleyen bir arayüz programıdır.
    
- **Bash (Bourne-Again Shell):** GNU Projesi kapsamında Brian Fox tarafından yazılmış, Unix/Linux dünyasını domine eden ücretsiz ve açık kaynaklı bir komut dili ve kabuk yazılımıdır. Büyük miktardaki log verilerini aramak, sıralamak, filtrelemek ve operasyonları otomatize etmek için siber güvenlik dünyasının vazgeçilmezidir.
    

### 🚀 İlk Script Yapısı ve Çalıştırma Kuralları

- **Shebang (`#!/bin/bash`):** Bir script dosyasının en üst satırında yer alan ve karakter sekansı numara işareti ile ünlemden (`#!`) oluşan yapıdır. Sisteme bu dosyanın hangi yorumlayıcı (interpreter) motoru ile çalıştırılacağını bildirir.
    
- **Yorum Satırı (`#`):** Bash, satır başında kare/diyez (`#`) işareti gördüğü her şeyi görmezden gelir. Kod içine not düşmek veya bir satırı devre dışı bırakmak için kullanılır.
    
- **Yetki Verme (`chmod`):** Linux'ta yeni oluşturulan dosyalar varsayılan olarak kilitlidir (çalıştırılamaz). Dosyaya execution (çalıştırma) izni vermek için şu komut koşturulur:
    
    Bash
    
    ```
    chmod +x script.sh
    ```
    
- **Çalıştırma (`./`):** Terminalde dosyanın başına `./` konularak tetiklenir (Örn: `./script.sh`). Bu işaret "işletim sistemine başka yere bakma, bu dosyayı tam olarak şu an içinde bulunduğum aktif klasörde ara ve çalıştır" talimatı verir.
    
- **`echo` Komutu:** Belirtilen metni veya değişken içeriğini doğrudan terminal ekranına basar (Python'daki `print()` karşılığı).
    

## 📦 2. Değişken Tanımlama Kuralları

Bash variable yapıları, verileri geçici olarak hafızada tutan kutucuklardır.

- ⚠️ **Boşluk Tuşu Tuzağı:** Değişken tanımlarken eşittir (`=`) işaretinin sağına ve soluna **asla boşluk bırakılamaz!** Aksi takdirde Bash bunu bir değişken değil, sistem komutu olarak algılar.
    
    Bash
    
    ```
    # DOĞRU YAZIM:
    name="Enes"
    # YANLIŞ YAZIM (HATA VERİR):
    name = "Enes"
    ```
    
- **Değişkeni Çağırma (`$`):** Tanımlanan bir değişkenin içindeki veriyi kullanmak veya ekrana yazdırmak için adının başına mutlaka dolar (`$`) işareti konur.
    
    Bash
    
    ```
    name="Enes"
    echo $name
    ```
    
- **Dinamik Komut Atama (`$()`):** Bir Linux komutunun ürettiği çıktıyı doğrudan bir değişkenin içine haritalamak için kullanılır.
    
    Bash
    
    ```
    current_user=$(whoami)
    echo "Aktif Kullanıcı: $current_user"
    ```
    

## 🛰️ 3. Shell Parametreleri ve Kullanıcı Girişi (Parameters & Input)

Script çalıştırılırken arkasına eklenen dış mühimmatlar (IP adresleri, wordlist yolları vb.) parametre değişkenlerinde saklanır.

- **`$0`:** Çalışan script'in **kendi dosya adını** verir. Kullanıcıya yardım (usage) mesajları dönerken hayati önem taşır.
    
- **`$1`, `$2`, `$3`, `$4`:** Script çalıştırılırken sırasıyla arkasına yazılan 1., 2., 3. ve 4. argüman değerlerini tutar. (Örn: `./script.sh hello hola aloha` komutunda `$1` = `hello`, `$3` = `aloha` olur).
    
- **`$#`:** Scripte dışarıdan toplam kaç tane parametre/argüman fırlatıldığını sayan sayaçtır.
    
- **`read` Komutu:** Kodun akışını durdurarak kullanıcının klavyeden interaktif bir veri yazmasını bekler ve bu veriyi belirtilen değişkene atar. (`read test` komutu gelen girdiyi `test` değişkenine yazar).
    
- **`sleep <saniye>`:** Kodun yürütülmesini belirtilen saniye kadar geciktirir veya duraklatır.
    

## 📊 4. Diziler (Arrays)

Birden fazla veriyi tek bir değişken altında, indeks numaralarıyla saklama mimarisidir. Sayım her zaman **0'ıncı indeksten** başlar.

- **Dizi Tanımlama:** Elemanlar parantez içinde, aralarına sadece boşluk bırakılarak yazılır:
    
    Bash
    
    ```
    cars=('honda' 'audi' 'bmw' 'tesla')
    ```
    
- **Tüm Diziyi Ekrana Basma (`[@]`):** `@` joker karakteri tüm elemanları çağırır:
    
    Bash
    
    ```
    echo "${cars[@]}"
    ```
    
- **İndeks ile Eleman Çağırma:** `audi` elemanını (1. indeks) ekrana basmak için:
    
    Bash
    
    ```
    echo "${cars[1]}"
    ```
    
- **Eleman Silme (`unset`):** Dizideki `tesla` elemanını (3. indeks) tamamen uçurmak için:
    
    Bash
    
    ```
    unset cars[3]
    ```
    
- **Eleman Değiştirme / Güncelleme:** 3. indeksteki elemanı silmek yerine `toyota` ile değiştirmek için:
    
    Bash
    
    ```
    cars[3]='toyota'
    ```
    

## 📐 5. Koşullu İfadeler ve Güvenlik Kontrolleri (Conditionals - If/Else)

- ⚠️ **Sentaks Kuralı (Boşluk Hassasiyeti):** Değişkenlerin aksine, `if` bloklarında kullanılan çift parantezlerin `[[` iç sınırlarında ve karşılaştırma operatörlerinin sağında/solunda **mutlaka boşluk bırakılmalıdır!** Blok her zaman **`fi`** kelimesiyle mühürlenir.
    
    Bash
    
    ```
    if [[ $count -eq 10 ]]
    then
      echo "True"
    fi
    ```
    

### 🔢 Sayısal ve Metinsel Karşılaştırma Operatörleri

- `-eq` : Eşittir (Equal)
    
- `-ne` : Eşit Değildir (Not Equal)
    
- `-gt` : Büyüktür (Greater Than)
    
- `-lt` : Küüçüktür (Less Than)
    
- `==` : Metinsel ifadelerde birebir eşitlik kontrolü.
    
- `&&` : "VE" mantıksal kapısı (İki şartın da doğru olmasını zorunlu kılar).
    

### 🛡️ Mavi Takım Dosya ve Dizin Kontrol Flag'leri (Audit Flags)

Siber olay müdahalesinde (Incident Response) bir dosyanın durumunu denetlemek için if bloklarında şu bayraklar koşturulur:

- **`-r`** : Belirtilen dosya üzerinde **Okuma (Read)** yetkimiz var mı?
    
- **`-w`** : Belirtilen dosya üzerinde **Yazma (Write)** yetkimiz var mı?
    
- **`-x`** : Belirtilen dosya **Çalıştırılabilir (Execute)** bir betik/program mı?
    
- **`-d`** : Belirtilen yol normal bir dosya değil, bir **Klasör/Dizin (Directory)** mü?
    
- **`-e`** : Nesne türüne bakılmaksızın, bu hedef sistemde fiziksel olarak **Mevcut mu (Exists)?**
    
- **`-f`** : Bu hedefin klasör değil, normal bir **Dosya (Regular File)** olduğunu doğrular.
    

## 🔄 6. Döngüler ve Fonksiyonlar (For Loops & Functions)

### A) For Loops (Tekrarlı Döngüler)

Belirli bir sayı aralığında veya bir dizinin tüm elemanları üzerinde sırayla işlem yapmak (liste taramak) için kullanılır:

Bash

```
# 1'den 5'e kadar sayıları sırayla ekrana basar
for n in {1..5}
do
  echo $n
done
```

### B) Bash Functions (Modüler Fonksiyonlar)

Aynı kod bloklarını script içinde defalarca yazıp amelelik yapmamak ve kodu okunabilir kılmak için kullanılan adlandırılmış komut kümeleridir.

Bash

```
#!/bin/bash
# Fonksiyon Tanımlama
function test_passwd {
  if [[ -e /etc/passwd ]]
  then
    echo "Yes it exists."
  else
    echo "The file doesn't exist."
  fi
}

# Fonksiyonu Çağırma / Tetikleme
test_passwd
```

## 🛠️ 7. Laboratuvar Pratik Çıktıları ve Nokta Atışı Cevaplar

Odadaki interaktif testler ve komut sorgularından elde edilen resmi çözümler:

- **Soru 1:** Bir kod satırını yorum satırı haline getirmek (comment out) için başına hangi karakter konur?
    
    - _Yanıt:_ **`#`**
        
- **Soru 2:** `echo "BishBashBosh"` komutunun ekran çıktısı ne olur?
    
    - _Yanıt:_ **`BishBashBosh`**
        
- **Soru 3:** Scripte dışarıdan gönderilen toplam argüman/parametre sayısını hangi özel parametre verir?
    
    - _Yanıt:_ **`$#`**
        
- **Soru 4:** Çalışan script'in kendi dosya adını (ilk argümanı) hangi parametre tutar?
    
    - _Yanıt:_ **`$0`**
        
- **Soru 5:** Dışarıdan girilen 4. argüman değerini alan parametre hangisidir?
    
    - _Yanıt:_ **`$4`**
        
- **Soru 6:** Kullanıcıdan girdi alıp bunu `test` isimli değişkene aktaran read komutu nasıl yazılır?
    
    - _Yanıt:_ **`read test`**
        
- **Soru 7:** Bir script `./script.sh hello hola aloha` komutuyla tetiklenirse, `echo $1 $3` sorgusunun ekran çıktısı ne olur?
    
    - _Yanıt:_ **`hello aloha`**
        
- **Soru 8:** `cars` dizisindeki `audi` elemanını indeks kullanarak ekrana basan komut nedir?
    
    - _Yanıt:_ **`echo "${cars[1]}"`**
        
- **Soru 9:** `cars` dizisindeki 3. indekste yer alan `tesla` elemanını diziden tamamen silen komut nedir?
    
    - _Yanıt:_ **`unset cars[3]`**
        
- **Soru 10:** Dizideki 3. indekse `tesla` yerine `toyota` değerini atayan/güncelleyen sentaks nedir?
    
    - _Yanıt:_ **`cars[3]='toyota'`**
        
- **Soru 11:** Bir dosya üzerinde okuma (read) erişimimizin olup olmadığını denetleyen if flag'i hangisidir?
    
    - _Yanıt:_ **`-r`**
        
- **Soru 12:** Belirtilen yolun bir klasör (directory) olup olmadığını denetleyen if flag'i hangisidir?
    
    - _Yanıt:_ **`-d`**
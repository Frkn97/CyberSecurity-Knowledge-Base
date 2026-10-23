# Linux Live Incident Response (IR) & Network Triage

- **Kategori:** Olay Müdahale / Uç Nokta İnceleme
    
- **Rol:** SOC Level 1 / Incident Responder
    
- **Amaç:** Şüpheli dış ağ bağlantısı (C2/Exfiltration) tespit edildiğinde Linux terminalinde sürecin, ikili dosyanın (binary) ve kök nedenin hızlıca ortaya çıkarılması.
    

## 1. Dış IP İle Konuşan Süreci ve PID'yi Bulma

SIEM veya ağ izleme aracından şüpheli dış IP (`<SUSPICIOUS_IP>`) uyarısı geldiğinde ilk komut:

```
# Aktif bağlantıları, IP adreslerini ve bağlantıyı açan PID/Process bilgisini listele
ss -antp | grep <SUSPICIOUS_IP>
```

- **Bayrakların Anlamı (`-antp`):**
    
    - `-a`: Tüm soketleri (dinleyen ve kurulu) listele.
        
    - `-n`: DNS çözümü yapma, doğrudan IP/port numaralarını göster (hızlı analiz için kritik).
        
    - `-t`: Yalnızca TCP bağlantılarını filtrele.
        
    - `-p`: Bağlantıyı açan **PID** ve **Process Name** bilgisini göster (_root/sudo yetkisi gerektirebilir_).
        

_Alternatif Komut:_

```
# Belirli bir dış IP'ye giden bağlantıları ve süreçleri listele
lsof -i @<SUSPICIOUS_IP>
```

## 2. Sürecin Ayrıntılarını İnceleme (Process Triage)

`ss` çıktısından elde edilen PID (örneğin `24959`) üzerinden sürecin çalıştırılma parametrelerini dök:

```
# Sürecin tam komut satırı argümanlarını gör
ps aux | grep <PID>
```

- **Ne Aranır?:**
    
    - Gizli parametreler (`-hidden`, `base64`, `nohup`),
        
    - Tek satırlık ters kabuk (Reverse Shell) veya soket komutları (`python -c ...`, `bash -i >& /dev/tcp/...`, `nc -e /bin/bash`).
        

## 3. Diskteki Asıl Dosyayı Doğrulama (/proc Dosya Sistemi)

Saldırganlar process ismini sistem servisi gibi (`systemd`, `kworker`) gösterebilir. Hafızada çalışan sürecin diskteki gerçek konumunu bulmak için Linux'un `/proc` sanal dosya sistemi kullanılır:

```
# Sürecin diskteki gerçek ikili dosya yolunu göster (symlink çözümü)
ls -l /proc/<PID>/exe

# Sürecin hangi dizinden çalıştırıldığını gör (Current Working Directory)
ls -l /proc/<PID>/cwd

# Çalıştırılan ham komut satırını incele
cat /proc/<PID>/cmdline
```

- **Şüpheli Dizinler:** `/tmp`, `/var/tmp`, `/dev/shm` (Saldırganların genellikle yazma yetkisi bulup zararlıyı indirdiği alanlar).
    

## 4. Tehdidi Durdurma (Containment)

Zararlı ağ trafiğini derhal kesmek ve süreci sonlandırmak için:

```
# Süreci zorla sonlandır
kill -9 <PID>
```

## 5. Sızma Kök Nedenini Doğrulama (Log Analizi)

Saldırgan sisteme nasıl erişti?

### A. SSH / Kimlik Doğrulama Logları

- **Debian / Ubuntu:** `/var/log/auth.log`
    
- **RHEL / CentOS / Fedora:** `/var/log/secure`
    

```
# Başarılı oturumları ve kaynak IP'leri filtrele
grep "Accepted" /var/log/auth.log

# Kaba kuvvet (Brute-Force) saldırılarını filtrele
grep "Failed password" /var/log/auth.log
```

### B. Web Sunucusu Logları (DMZ / Web Server ise)

- **Nginx:** `/var/log/nginx/access.log`
    
- **Apache:** `/var/log/apache2/access.log` (veya `/var/log/httpd/access_log`)
    

```
# Dışarıya giden bağlantı öncesindeki şüpheli POST/GET isteklerini incele
tail -n 200 /var/log/nginx/access.log | grep -E "POST|UNION|SELECT|sh|bash"
```

## Mülakat Özeti (30 Saniyelik Cevap)

> _"Bir Linux sistemde dışarıya şüpheli bağlantı tespit ettiğimde:_
> 
> 1. _`ss -antp | grep <IP>` ile bağlantıyı kuran PID'yi ve süreci yakalarım._
>     
> 2. _`ps aux | grep <PID>` ile komut satırı argümanlarını dökerim._
>     
> 3. _`ls -l /proc/<PID>/exe` ile diskin neresinden çalıştığını doğrularım._
>     
> 4. _Süreci `kill -9 <PID>` ile sonlandırıp `/var/log/auth.log` (SSH) veya web access logları üzerinden sızma yöntemini analiz ederim."_
>
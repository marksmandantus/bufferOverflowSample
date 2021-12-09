INSTALLATION 

https://github.com/corelan/mona
https://github.com/stephenbradshaw/vulnserver
https://www.immunityinc.com/products/debugger/


REQUIREMENTS / used tools 

msfvenom
netcat 
Immunity debugger (reverse engineering)

FOLDER SHORTCUTS

/usr/share/metasploit-framework/tools/exploit  ./nasm_shell.rb -> use JMP ESP code
/usr/share/metasploit-framework/tools/exploit  ./pattern_created.rb -> use -l (in this code -l 3000)
/usr/share/spike/audits -> example spike codes


SUMMARY

Pythonla yazdığım bufferoverflow örneği;

İlk olarak https://github.com/stephenbradshaw/vulnserver ' ı indiriyoruz. Bunun sebebi zafiyetli makine üzerinden örnek göstermek için. Ardından bize en çok gerekecek olan Immunity Debugger(https://debugger.immunityinc.com/ID_register.py) aracını 
indiriyoruz. İkisini de yönetici olarak çalıştırmamız gerek. İmmunity debuggerdan vulnserver'ı takip etcez. Bunun için sol üstte bulunan attach kısmından vulnserver'ı seçin. Eğer gözükmüyorsa vulnserver'ı tekrar yönetici olarak çalıştırmayı deneyin. play tuşuna basarak takip edebiliriz. Çalıştığını sağ alttaki Ready/Running kısmından anlayabiliriz.
![immunity](https://user-images.githubusercontent.com/59103139/145373244-d63b56ef-3bfe-4287-bb9f-a5e7035673a9.PNG)

![fuzzing](https://user-images.githubusercontent.com/59103139/145376900-567fe671-05a5-4b05-b473-bff255be6ed7.PNG)

İlk olarak fuzzing kodumuzu yazalım.Kodu anlatmak gerekirse socket modülü bağlantı kurmamızı sağlar. Sleep modülü programın ne kadar süre bekletileceğini ayarlar. Bekletme amacımız server'a fazla yüklenmemek. sys modülü ise sistemde yapılabilecek işlemleri yönetir. 

TRUN yazan şeyi ise deneyerek bulabileceğimiz bir şey. (RTIME, GDOG, GTER, LTER gibi versiyonlar da var.) Daha fazlasını şuradan okuyabilirsiniz: https://fluidattacks.com/blog/reversing-vulnserver/ .: ise çalışması için gerekli bir kod. Kodu özet olarak anlatacak olursam, program çökene kadar 100 tane a yolluyor. Çökünce bize kaçta çöktüğünü belirtiyor.
Socket.connect kısmında ise kendi local ip ve 9999 portunu kullanmamız gerek. Latin1 encode kullanma sebebimiz karakterleri byte olarak yollamamız.

![crashed](https://user-images.githubusercontent.com/59103139/145376823-66f1c508-5b52-4e82-85b4-fd28509b560c.PNG) 

Görüldüğü gibi 2009 da crashlendi.O zaman offset kısmına geçelim.

![offset](https://user-images.githubusercontent.com/59103139/145377122-28d1b666-9546-4a4b-985c-885e68f8eb67.PNG)

Bu kodun diğer koddan tek farkı characters kısmı. 2009'da crash ettiğini bildiğimiz için kendimize 3000 karakterli pattern yapıyoruz.(Yorum satırında belirtilen yerden yapabilirsiniz. Sadece linuxta geçerlidir.) 3000 yapma sebebimiz ise  2009 dan daha sonra crash yiyebilme ihtimalimiz var. Sıradaki kısım ise kötü karakterler.

![badchars](https://user-images.githubusercontent.com/59103139/145377866-c9347ef4-233d-40c1-b403-7f456cc0a581.PNG)

2009'da crash yediğimizi önceden biliyorduk. Şimdi ise kodun çalışıp çalışmadığını deneyelim.
![Bx4](https://user-images.githubusercontent.com/59103139/145378638-8b61eec0-b19c-4eda-a8db-a6d6daa61eae.PNG)
4 adet B eklediğimizde görüldüğü üzere 414141 den 424242 ye geçiş yapmış, yani çalışıyor. Kötü karakterleri ekleme sebebimiz buffer overflow çalışıyor mu sorusuna cevap vermek. Bunu immunity debuggerdan takip edebilirsiniz.
Bad Characters : https://www.adamcouch.co.uk/python-generate-hex-characters/

Sıradaki kısım Bufferoverflow kısmı.

![bfoverflow](https://user-images.githubusercontent.com/59103139/145383730-ac126db0-6ebe-4d4b-9268-53769764d61b.PNG)

İlk işlemimiz https://github.com/corelan/mona ' yı indirip immunity debugger dosya konumundan PyCommands klasörüne .py uzantılı dosyayı atmak.

![ffe4](https://user-images.githubusercontent.com/59103139/145384076-50167b06-01c5-4861-b414-eaa11eeb5c19.PNG)
Sonraki işlemimiz /usr/share/metasploit-framework/tools/exploit klasöründen nasm_shell.rb yi çalıştırıp JMP ESP yazmak. Sonra bize 4 bitlik hexadecimal sayı vercek. Bunun sebebini immunity debuggerda aşağıdaki arama kısmına verilen 4 bitlik sayı ile birlikte komutumuzu yazdığımızda öğrenceksiniz. 

-s komutu ile bize verilen hexadecimal sayıyı tırnak işareti arasında yazarız.
-m komutu ile açık olan dosyayı belirtiriz. Açık olduğunu ise şu şekilde anlarız; solundaki bütün değerler false ise koruması yok demektir. Görüldüğü gibi essfunc.dll değerleri false.


![0x625](https://user-images.githubusercontent.com/59103139/145383983-16ae325c-0204-4522-989a-da87b9d55153.PNG)
!mona find -s "\xff\xe4" -m essfunc.dll yazıp entera bastığımızda sonuçlar önümüze gelecektir. Gelen sonuçları deneyerek JMP ESP'nin çalıştığı sonucu bulmamız gerek. Örneğin en üstteki olan 0x625011AF den örnek verecek olursak 

Kodumuzda "A"*2009 un yanına yazıyoruz ve sonuna \x90 ekliyoruz. Bunun sebebi x90 eklemediğimiz bazı zamanlar kodun çalışmadığı olabiliyor. 32 ile çarpma sebebimiz 32 bit olması.


En son kısmımız ise asıl amacımız olan kendimize bağlantı açmamız. Msfvenom ile c dilinde hexadecimal bir payload oluşturup netcat ile dinleyeceğiz. Ardından verilen hex payloadı python kodumuza yazalım.
![msfvenom](https://user-images.githubusercontent.com/59103139/145384444-de88d80b-2f18-4ebe-80b4-87484ddcf5a3.PNG)


![shellcode](https://user-images.githubusercontent.com/59103139/145384582-e4854bab-eeaf-4f59-bc4f-411523ffc122.PNG)

En son işlem olan netcat ile nc -nvlp 4444(msfvenomda belirtilen lport) yazarak bağlantımızı oluşturmuş olduk.


NOT: Bu şekilde hata alıyorsanız yönetici olarak çalıştırmayı deneyiniz.
![fuzzing](https://user-images.githubusercontent.com/59103139/145384780-b7cd6928-aa00-4968-b106-c5e53e04c7fb.PNG)

NOT 2: Local IP'lerin farklı olmasının sebebi biri Linux'un diğeri Windows'un IP'si. 

NOT 3: msfvenom kısmında EXITFUNC=thread yazdık fakat process de kullanabilirdik. Fakat öyle bir durum olsaydı multi/handler ile dinlememiz gerekirdi. Biz burada netcat ile dinledik.

NOT 4: Kodumuzda A, B gibi karakterler kullanarak açık bulmayı denedik. Linuxta bazı hazır spike kodları var. Spike kodları hakkında daha fazla bilgi sahibi olmak istiyorsanız https://resources.infosecinstitute.com/topic/intro-to-fuzzing/ sitesini ziyaret edebilirsiniz.
https://null-byte.wonderhowto.com/how-to/hack-like-pro-build-your-own-exploits-part-3-fuzzing-with-spike-find-overflows-0162789/

Okuduğunuz için teşekkürler. Yanlış belirttiğim yer fark ederseniz mehmetkocadag679@gmail.com adresinden iletişime geçebilirsiniz.






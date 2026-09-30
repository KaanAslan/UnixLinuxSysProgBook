==================
**exec İşlemleri**
==================

exec Fonksiyonları
===================

Bir program dosyasını yükleyip çalıştırmak için ismine *exec fonksiyonları* denilen bir grup POSIX fonksiyonu
kullanılmaktadır. Bu fonksiyonların yaptıkları işlemler birbirine benzerdir. Ancak fonksiyonların parametrik
yapıları arasında ve işlevsellikleri arasında bazı farklılıklar vardır. POSIX standartlarında bulunan 7 exec
fonksiyonunun isimleri şöyledir:

.. code-block:: text

    execl
    execle
    execlp
    execv
    execve
    execvp
    fexecve

Ayrıca POSIX standartlarında tanımlı olmasa da GNU C kütüphanesinde ``execvpe`` isimli bir exec fonksiyonu da
bulunmaktadır. (Bu fonksiyon *glibc* kütüphanesinde olduğu için bu kütüphanenin kullanıldığı BSD gibi diğer UNIX
türevi sistemlerde de bulunmaktadır.) Ayrıca Linux sistemlerine özgü bir biçimde ``sys_execveat`` isimli bir sistem
fonksiyonu da bulunmaktadır. Linux'ta bu fonksiyon ``execveat`` ismiyle kullanılabilmektedir.

Aslında UNIX/Linux sistemleri bu exec fonksiyonlarının hepsini sistem fonksiyonu biçiminde bulundurmamaktadır.
Örneğin Linux sistemlerinde ``execve`` fonksiyonu bir sistem fonksiyonu biçiminde (``sys_execve``) yazılmıştır.
Diğer exec fonksiyonları bu sistem fonksiyonunu çağıran kütüphane fonksiyonları biçiminde gerçekleştirilmiştir.
Yukarıda da belirttiğimiz gibi Linux'taki ``execveat`` fonksiyonu da bir sistem fonksiyonu biçiminde
(``sys_execveat``) gerçekleştirilmiştir. Bu durumda yukarıdaki POSIX fonksiyonları dışında Linux'a özgü olan exec
fonksiyonları şunlardır:

.. code-block:: text

    execvpe
    execveat

exec fonksiyonları prosesin yaşamına başka bir program koduyla devam etmesini sağlamaktadır. exec fonksiyonlarına
biz *çalıştırılabilen bir program dosyasını* argüman olarak veririz. exec fonksiyonları o anda çalışmakta olan
programın bellek alanını tamamen boşaltıp onun yerine bizim verdiğimiz program dosyasını belleğe yükler ve o
yüklediği programın kodunu çalıştırır. exec işlemi ile prosesin kontrol bloğundaki pek çok alan
değiştirilmemektedir. Yani prosesin ID'si, kullanıcı ve grup ID'leri, prosesin çalışma dizini vs. değişmez. exec
işlemleriyle prosesin yalnızca çalıştırdığı program dosyası değiştirilmektedir. Örneğin *sample* programının
içerisinde biz exec fonksiyonlarıyla *other* programını çalıştırmak istediğimizde *sample* programı bellekten
tamamen atılır, onun yerine *other* programının kodu ve verileri belleğe yüklenir ve *other* programının kodu
çalıştırılır. Yukarıda da belirttiğimiz gibi exec işlemi sırasında prosesin kontrol bloğundaki temel bilgiler
değişmez. Yani exec fonksiyonları uygulandığında proses yaşamına başka bir program koduyla devam etmektedir.
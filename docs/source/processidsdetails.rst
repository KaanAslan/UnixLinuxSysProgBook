==============================================================
**Proseslerin Kullanıcı ve Grup ID'lerine İlişkin Ayrıntılar**
==============================================================

set-user-id, set-group-id ve sticky Erişim Hakları
==================================================

Şimdi dosyaların henüz görmediğimiz *set-user-id*, *set-group-id* ve *sticky* denilen erişim hakları
üzerinde duracağız.

Biz şimdiye kadar dosyalar için 9 erişim bayrağı gördük: ``S_IRUSR``, ``S_IWUSR``, ``S_IXUSR``,
``S_IRGRP``, ``S_IWGRP``, ``S_IXGRP``, ``S_IROTH``, ``S_IWOTH``, ``S_IXOTH``. Bu bayraklar dosyanın
``rwx rwx rwx`` erişim haklarını belirtmektedir. Ancak aslında dosyaların erişim hakları 9 tane değil 12
tanedir. Henüz görmediğimiz üç erişim hakkına *set-user-id*, *set-group-id* ve *sticky* hakları
denilmektedir. Şimdi dikkatimizi bu üç erişim hakkına çevireceğiz. Bu üç erişim hakkı ``open``
fonksiyonunda ya da ``chmod`` fonksiyonunda sırasıyla ``S_ISUID``, ``S_ISGID`` ve ``S_ISVTX`` sembolik
sabitleriyle kullanılabilmektedir. Bu sembolik sabitler aslında en yüksek anlamlı dördüncü octal digit'e
karşılık gelmektedir:

.. code-block:: text

    UGS rwx rwx rwx rwx

Dolayısıyla POSIX 2008 ve sonrasında bu bayrakları yüksek anlamlı dördüncü octal digit'le de
belirtebiliriz.

chmod Komutuyla set-user-id / set-group-id / sticky Ayarlama
============================================================

Sembolik Gösterim (u+s, g+s, +s, +t)
------------------------------------

Komut satırında dosyanın set-user-id bayrağını set etmek için ``chmod`` komutunda ``u+s`` seçeneği
kullanılabilir. Örneğin:

.. code-block:: console

    $ ls -l sample
    -rwxrwxrwx 1 kaan study 17176 Şub 19 11:46 sample
    $ chmod u+s sample
    $ ls -l sample
    -rwsrwxrwx 1 kaan study 17176 Şub 19 11:46 sample

Görüldüğü gibi eğer dosyanın hem ``x`` bayrağı hem de set-user-id bayrağı set edilmişse ``x`` hakkının
bulunduğu yerde ``s`` harfi gözükmektedir. Ancak dosyanın yalnızca set-user-id bayrağı set edilmişse
``x`` hakkının bulunduğu yerde ``S`` harfi gözükür. Örneğin:

.. code-block:: console

    $ ls -l test.txt
    -rw-rw-rw- 1 kaan study 2087 Şub 19 10:49 test.txt
    $ chmod u+s test.txt
    $ ls -l test.txt
    -rwSrw-rw- 1 kaan study 2087 Şub 19 10:49 test.txt

Dosyanın set-group-id bayrağının set edilmesi de ``chmod`` komutunda ``g+s`` ile yapılmaktadır. İşlem
sonrasında yine grup bilgisinde ``x`` hakkı yerinde ``s`` ya da ``S`` görünür. ``chmod`` komutunda
``+s`` kullanılırsa bu durumda dosyanın hem set-user-id hem de set-group-id bayrakları set edilir.
Dosyanın sticky bayrağını set etmek için ``chmod`` komutunda ``+t`` kullanılmaktadır. Bu işlem
yapıldığında görüntü olarak grup hakkında ``x`` varsa ``x`` hakkının olduğu yerde ``t``, yoksa orada
``T`` görülmektedir.

Yukarıdaki komutlarda ``+`` yerine ``-`` karakterini getirerek reset işlemlerini yapabilirsiniz.

Octal Gösterim (chmod 4755 ...)
-------------------------------

Benzer biçimde istersek yine ``chmod`` komutunda octal digit'lerle set-user-id, set-group-id ve sticky
bitlerini set edebiliriz. Örneğin:

.. code-block:: console

    $ chmod 4755 sample

Burada en soldaki octal digit 4 olduğu için dosyanın set-user-id bayrağı da set edilmiştir. En soldaki
octal digit'in bitleri yukarıda da belirttiğimiz gibi şu sıradadır:

.. code-block:: text

    set-user-id  set-group-id  sticky

chmod Fonksiyonuyla Programatik Olarak Ayarlama
===============================================

``chmod`` fonksiyonunda yukarıda belirttiğimiz bayrakları kullanarak set-user-id, set-group-id ve sticky
bayraklarını set edebiliriz. Örneğin biz ``sample`` programını ``rwsrwxrwx`` haline şöyle getirebiliriz:

.. code-block:: c

    if (chmod("sample", S_IRWXU|S_IRWXG|S_IRWXO|S_ISUID) == -1)
        exit_sys("chmod");

Peki zaten var olan bir dosyaya mevcut erişim haklarını bozmadan bu özellikleri programlama yoluyla nasıl
ekleyebiliriz? Bunun için önce dosyanın erişim haklarının ``stat``, ``lstat`` ya da ``fstat``
fonksiyonuyla elde edilmesi gerekmektedir. Ondan sonra bu erişim haklarına biz ``S_ISUID``, ``S_ISGID``
ve ``S_ISVTX`` bayraklarını OR işlemiyle ekleyebiliriz. Ancak ``stat`` fonksiyonunun bize verdiği
``st_mode`` değeri dosyanın türünü de içermektedir. Gerçi ``chmod`` fonksiyonu bu ekstra bitleri dikkate
almamaktadır. Ancak yine de ``stat`` yapısının ``st_mode`` elemanındaki değeri ``S_IFMT`` ile maskelemek
daha uygundur. (Stevens *"Advanced Programming in the UNIX Environment"* kitabında böyle yapmamıştır.) O
halde bu işlem şöyle yapılabilir:

.. code-block:: c

    struct stat finfo;

    if (stat("sample", &finfo) == -1)
        exit_sys("stat");

    if (chmod("sample", (finfo.st_mode & ~S_IFMT) | S_ISUID) == -1)
        exit_sys("chmod");

``S_IFMT`` bayrağı erişim hakları dışındaki dosya tür bayraklarını temsil etmektedir. ``~S_IFMT`` erişim
hakları bayraklarını 1, diğer bayrakları 0 hale getirmektedir.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/stat.h>

    void exit_sys(const char *msg);

    int main(int argc, char *argv[])
    {
        struct stat finfo;

        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!..\n");
            exit(EXIT_FAILURE);
        }

        if (stat(argv[1], &finfo) == -1)
            exit_sys("stat");

        if (chmod(argv[1], (finfo.st_mode & ~S_IFMT) | S_ISUID) == -1)
            exit_sys("chmod");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

set-user-id ve set-group-id'nin Anlamı (Çalıştırılabilir Dosyalarda)
====================================================================

Peki bir dosyanın set-user-id ve set-group-id bayraklarının set edilmiş olmasının ne anlamı vardır?
Öncelikle bu bayrakların yalnızca *çalıştırılabilir dosyalar için* anlamlı olduğunu belirtelim. Yani bu
bayraklar tasarımda *çalıştırılabilir dosyalar* için düşünülmüştür. Çalıştırılabilir dosyanın set-user-id
bayrağı set edilmişse bu dosya exec yapıldığında prosesin etkin kullanıcı ID'si işletim sistemi
tarafından dosyanın kullanıcı ID'si olacak biçimde değiştirilmektedir. Benzer biçimde çalıştırılabilir
dosyanın set-group-id bayrağı set edilmişse bu dosya exec yapıldığında prosesin etkin grup ID'si
dosyanın grup ID'si olarak değiştirilmektedir.

passwd Örneği
-------------

Set-user-id bayrağının kullanım gerekçesini basit bir örnekle açıklayabiliriz. Bilindiği gibi kullanıcı
parolaları komut satırında ``passwd`` komutuyla değiştirilmektedir. Default durumda ``passwd`` komutu
kullanıcının kendi parolasını değiştirmektedir. Örneğin:

.. code-block:: console

    $ passwd

Burada ``passwd`` önce mevcut parolayı sonra da yeni parolayı bize sormaktadır. ``passwd`` programı
parolayı ``/etc/shadow`` dosyasına yazmaktadır. Bu dosyaya yalnızca ``root`` prosesler
erişebilmektedir:

.. code-block:: console

    $ ls -l /etc/passwd
    -rw-r--r-- 1 root root 3008 Haz 11 13:33 /etc/passwd

Peki biz ``passwd`` komutunu çalıştırdığımızda kabuk önce ``fork`` sonra exec yaptığına göre bizim
çalıştırdığımız ``passwd`` programı ``/etc/shadow`` dosyasına nasıl erişmektedir? İşte aslında
``/bin/passwd`` programının set-user-id bayrağı set edilmiştir:

.. code-block:: console

    $ ls -l /bin/passwd
    -rwsr-xr-x 1 root root 64152 May 30  2024 /bin/passwd

Böylece bu dosya exec yapıldığında prosesin etkin kullanıcı id'si de set-user-id bayrağının etkisiyle
``root`` olmaktadır. Şimdi siz bunun bir güvenlik açığı oluşturabileceğini düşünebilirsiniz. Ancak
aslında bir güvenlik açığı söz konusu değildir. Bu programın sahibi ``root`` kullanıcısıdır ve o
isteyerek bu programın ``root`` etkin kullanıcı ID'siyle çalıştırılmasına olanak sağlamıştır. Herhangi
bir kullanıcı zaten başkalarına ait dosyaların erişim haklarını değiştirememektedir.

Burada bir noktaya dikkat ediniz. set-user-id ya da set-group-id bayrağı set edilmiş dosyalar exec
yapılırken prosesin yalnızca etkin kullanıcı ID'si ve etkin grup ID'si değiştirilmektedir. Gerçek
kullanıcı ID'si ve gerçek grup ID'si değiştirilmemektedir. Biz genellikle gerçek ve etkin ID'lerin aynı
değerde olduğunu, ancak test işlemlerine etkin ID'lerin girdiğini belirtmiştik. İşte set-user-id ve
set-group-id bayrakları set edilmiş bir program dosyası exec yapıldığında artık gerçek kullanıcı ve
grup ID'leriyle etkin kullanıcı ID'leri farklılaşabilmektedir.

Gerçek ve Etkin ID Farkı Örneği (sample.c / other.c)
----------------------------------------------------

Aşağıdaki örnekte ``sample`` programı ``other`` programını exec yaparak çalıştırmıştır. Bu ``other``
programı prosesin gerçek ve etkin kullanıcı ve grup ID'lerini ekrana isim olarak yazdırmaktadır. Biz bu
örnekte ``other`` programını derledikten sonra ``chown`` komutuyla ``sudo`` ile birlikte kullanıcı
ID'sini ve grup ID'sini ``root`` olarak değiştirdik. Bu deneyi önce ``other`` programının set-user-id
bayrağı set edilmeden ve set edildikten sonra yineleyiniz. Aşağıda yapılanlar özetlenmiştir:

.. code-block:: console

    $ gcc -Wall -o sample sample.c
    $ gcc -Wall -o other other.c
    $ ls -l other
    -rwxr-xr-x 1 kaan study 17088 Şub 19 13:42 other
    $ sudo chown root:root mample
    $ ls -l other
    -rwxr-xr-x 1 root root 17088 Şub 19 13:42 mample
    $ ./sample
    Real user ID: kaan
    Effective user ID: kaan
    Real group ID: study
    Effective group ID: study
    $ sudo chmod u+s other
    $ ls -l other
    -rwsr-xr-x 1 root root 17088 Şub 19 13:42 other
    $ ./sample
    Real user ID: kaan
    Effective user ID: root
    Real group ID: study
    Effective group ID: study
    $ sudo chmod g+s other
    $ ls -l other
    -rwsr-sr-x 1 root root 17088 Şub 19 13:42 other
    $ ./sample
    Real user ID: kaan
    Effective user ID: root
    Real group ID: study
    Effective group ID: root

Biz bir programı ``sudo`` ile root önceliğinde (etkin proses ID'si 0 olacak biçimde) çalıştırıyor
olalım. Dosyanın set-user-id bayrağı da set edilmiş olsun. Bu durumda program çalışırken prosesin etkin
kullanıcı ID'si ``root`` değil, program dosyasının kullanıcı ID'si olacaktır. Yani set-user-id bayrağı
set edilmiş olan programların ``sudo`` ile root önceliğinde çalıştırılmasının bir anlamı kalmamaktadır.
Örneğin:

.. code-block:: console

    $ sudo chown ali: other
    $ ls -l other
    -rwxr-xr-x 1 ali study 16344 Eki  1 11:48 other
    $ sudo chmod u+s other
    $ ls -l other
    -rwsr-xr-x 1 ali study 16344 Eki  1 11:48 other
    $ sudo other
    $ sudo ./other
    Real user ID: root
    Effective user ID: ali
    Real group ID: root
    Effective group ID: root

.. code-block:: c

    /* sample.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/wait.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        pid_t pid;

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid == 0 && execl("mample", "mample", (char *)0) == -1)
            exit_sys("execl");

        if (wait(NULL) == -1)
            exit_sys("wait");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

.. code-block:: c

    /* other.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <errno.h>
    #include <unistd.h>
    #include <pwd.h>
    #include <grp.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        struct passwd *pass;
        struct group *gr;

        errno = 0;
        if ((pass = getpwuid(getuid())) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid user ID!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getpwuid");
        }

        printf("Real user ID: %s\n", pass->pw_name);

        errno = 0;
        if ((pass = getpwuid(geteuid())) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid user ID!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getpwuid");
        }
        printf("Effective user ID: %s\n", pass->pw_name);

        errno = 0;
        if ((gr = getgrgid(getgid())) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid group ID!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getgrgid");
        }
        printf("Real group ID: %s\n", gr->gr_name);

        errno = 0;
        if ((gr = getgrgid(getegid())) == NULL) {        if (errno == 0) {
                fprintf(stderr, "invalid group ID!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getgrgid");
        }
        printf("Effective group ID: %s\n", gr->gr_name);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Dizinlerde set-group-id Bayrağı
===============================

Normal olarak set-user-id ve set-group-id bayrakları çalıştırılabilen dosyalar için söz konusudur. Ancak
dizinler için de set-group-id bayrağının bir anlamı vardır. Pek çok UNIX türevi sistemde (Linux da buna
dahil) bir dizinin set-group-id bayrağı set edilirse Linux o dizin içerisinde ``open`` fonksiyonuyla
(zaten başka yolu yoktur) ya da ``mkdir`` fonksiyonuyla bir dosya ya da dizin yaratıldığında dosyanın ya
da dizinin grup ID'sini prosesin etkin grup ID'si olarak değil, o dizinin grup ID'si olarak set
etmektedir. BSD sistemlerinde zaten default olarak bir dosya ya da dizin yaratıldığında dosyanın ya da
dizinin grup ID'si o dosyanın ya da dizinin içinde bulunduğu dizinin grup ID'si olarak set edilmektedir.
O halde Linux'ta ``open`` fonksiyonu ile bir dosya ya da dizin yaratılırken dosya ya da dizinin grup
ID'si, eğer o dosya ya da dizinin içinde bulunduğu dizinin set-group-id bayrağı set edilmemişse prosesin
etkin grup ID'si olarak, eğer set edilmişse dizinin grup ID'si olarak set edilmektedir. Bu durumu basit
bir biçimde şöyle test edebilirsiniz:

Önce ``xxx`` gibi bir isimle bir dizin yaratınız:

.. code-block:: console

    $ mkdir xxx
    $ ls -ld xxx
    drwxr-xr-x 2 kaan study 4096 Eki  1 12:37 xxx

Görüldüğü gibi dizinin kullanıcı ve grup ID'si prosesin etkin kullanıcı ID'si ve grup ID'si biçimindedir.
Şimdi biz bu dizin içerisinde bir dosya yaratalım:

.. code-block:: console

    $ cd xxx
    $ touch x.txt
    $ ls -l x.txt
    -rw-r--r-- 1 kaan study 0 Eki  1 12:38 x.txt

Görüldüğü gibi ``x.txt`` dosyasının kullanıcı ve grup ID'leri prosesin etkin kullanıcı ve grup ID'si
biçimindedir. Şimdi biz ``xxx`` dizininin grup ID'sini ``root`` olarak değiştirelim ve dizinin
set-group-id bayrağını set edelim:

.. code-block:: console

    $ cd ..
    $ sudo chown :root xxx
    $ sudo chmod g+s xxx
    $ ls -ld xxx
    drwxr-sr-x 2 kaan root 4096 Eki  1 12:38 xxx

Şimdi yeniden dizine geçip dosya yaratalım:

.. code-block:: console

    $ cd xxx
    $ touch y.txt
    $ ls -l x.txt y.txt
    -rw-r--r-- 1 kaan study 0 Eki  1 12:38 x.txt
    -rw-r--r-- 1 kaan root  0 Eki  1 12:43 y.txt

Görüldüğü gibi artık dosyanın grup ID'si prosesin etkin grup ID'si olarak değil, içinde bulunduğu
dizinin grup ID'si olarak set edilmiştir.

Dizinlerin set-user-id bayraklarının set edilmesi benzer bir etkiye yol açmamaktadır. (Bu durum bazı eski
UNIX sistemlerinde denenmiştir, ancak modern sistemlerde böyle bir semantik yoktur.)

Betik Dosyalarında set-user-id / set-group-id Güvenlik Açığı
============================================================

Peki betik dosyaları (shebang içeren text dosyalar) için set-user-id ve set-group-id bayrakları set
edilebilir mi? Bu işlem ilk zamanlar uygulanmıştır. Ancak güvenlik açığı nedeniyle sonra uygulamadan
kaldırılmıştır. Bugünkü modern UNIX/Linux sistemleri betik dosyalarının set-user-id ve set-group-id
bayrakları set edilmiş olsa bile onları dikkate almamaktadır. Buradaki güvenlik açığı ilginç bir
biçimde aşağıdaki gibi oluşmaktadır:

Betik dosyasının ismi ``x.txt`` olsun. Eğer bu dosyanın set-user-id ve set-group-id bayrakları dikkate
alınsaydı bu durumda exec fonksiyonları bu dosyayı açıp shebang satırında bulunan programı çalıştırırken
prosesin etkin kullanıcı ve/veya grup ID'sini ``x.txt`` dosyasının kullanıcı ve/veya grup ID'si olarak
set ederdi. Bu durumda da eğer birisi örneğin ``y.txt`` sembolik bağlantı dosyası oluşturup bu dosyanın
``x.txt`` dosyasını göstermesini sağlarsa ve bu ``y.txt`` ile exec yaparsa bu durumda aslında exec
fonksiyonu sembolik bağlantıyı izleyecek ve ``x.txt`` dosyasını çalıştıracaktır. Ancak exec fonksiyonları
bu ``x.txt`` dosyasının shebang satırındaki programı çalıştırırken yine komut satırı argümanı olarak
``y.txt`` dosyasını kullanacaktır. İşte tam bu sırada birisi bu ``y.txt`` dosyasının sembolik
bağlantısını değiştirirse maalesef prosesin etkin kullanıcı ID'si ``x.txt``'nin kullanıcı ID'si olacak
biçimde aslında başka dosyayı çalıştırır.

sticky Bayrağı
==============

Peki dosyaların sticky bayraklarının ne işlevi vardır? Aslında sticky bayrağı tasarımda başka bir amaçla
düşünülmüştür. Eski sistemlerde bu bayrak çalıştırılabilen dosyaların çalıştırılması sonrasında programın
bellekten atılmaması gibi bir ipucu oluşturmaktadır. Ancak modern sistemlerde böyle bir etkinin bir
anlamı kalmadığı için sticky bayrağı da ilk tasarlandığı zamanki işlevinden tamamen kopmuştur. Bugün
sticky bayrağı değişik sistemlerde değişik amaçlarla kullanılabilmektedir. POSIX standartları eskiden
sticky bayrağı üzerinde açıklama yapmıyordu. Ancak belli zamandan sonra sticky için şöyle bir
işlevsellik tanımlanmıştır: Bir dizinin sticky bayrağı set edilirse ve dizinin sahibi başkaları ise,
dizine prosesin yazma hakkı olsa bile dizin içerisindeki başkalarına ait (yani kullanıcı ID'si başka)
olan dosyalar silinememekte ve ismi değiştirilememektedir. Bugünkü sistemlerde dizin dışında diğer
dosyaların sticky bayraklarının set edilmiş olup olmamasının işlevsel bir anlamı yoktur. Örneğin Linux
sistemlerinde ``/tmp`` dizininin sticky bayrağı set edilmiştir ve bu dizine yazma hakkı verilmiştir. Bu
durumda biz bu dizinde dosya yaratabiliriz, kendi dosyamızı silebiliriz. Ancak başkalarının dosyalarını
silemeyiz. ``/tmp`` dizininin erişim hakları şöyledir:

.. code-block:: text

    drwxrwxrwt 19 root root 65536 Şub 25 10:05 /tmp

/tmp Dizini Örneği
------------------

Bu durumu Linux sistemlerinde şöyle test edebiliriz:

Önce ``yyy`` isimli bir dizin yaratalım, bu dizinin içine geçip orada sahibi ``root`` olan bir dosya
yaratalım:

.. code-block:: console

    $ mkdir yyy
    $ cd yyy
    $ sudo touch x.txt
    $ ls -l x.txt
    -rw-r--r-- 1 root root 0 Eki  1 12:53 x.txt

Görüldüğü gibi dosyanın kullanıcı ve grup ID'si ``sudo`` uygulanmasından dolayı ``root`` olmuştur. Şimdi
burada ``sudo`` uygulamadan ikinci bir dosya da yaratalım:

.. code-block:: console

    $ touch y.txt
    $ ls -l x.txt y.txt
    -rw-r--r-- 1 root root  0 Eki  1 12:53 x.txt
    -rw-r--r-- 1 kaan study 0 Eki  1 12:55 y.txt

Biz her iki dosyayı da silebiliriz. Çünkü bir dosyayı silebilmek için dosyanın sahibi olmaya, dosyaya
``w`` hakkına sahip olmaya gerek yoktur. Tek gereken şey dizin için ``w`` hakkına sahip olmaktadır. Şimdi
dizinin sahipliğini değiştirelim ve sticky bayrağını set edip, herkese ``w`` hakkı verelim:

.. code-block:: console

    $ cd ..
    $ sudo chown root:root yyy
    $ sudo chmod 777 yyy
    $ sudo chmod +t yyy
    $ ls -ld yyy
    drwxrwxrwt 2 root root 4096 Eki  1 13:05 yyy

Şimdi dizine girip ``x.txt`` dosyasını silmeye çalışalım:

.. code-block:: console

    $ rm x.txt
    rm: yazma korumalı normal boş dosya 'x.txt' kaldırılsın mı? y
    rm: 'x.txt' silinemedi: İşleme izin verilmedi

Görüldüğü gibi dizine yazma hakkımız olsa da dizindeki başkalarına ait dosyaları silemiyoruz. Şimdi
``y.txt`` dosyasını silmeye çalışalım:

.. code-block:: console

    $ rm y.txt

Görüldüğü gibi kendimize ait olan bu dosya silinebilmiştir.

Gerçek, Etkin ve Saklı Kullanıcı/Grup ID'leri
=============================================

Daha önceden de belirttiğimiz gibi bir prosesin *gerçek kullanıcı ID'si (real user ID)* ile *etkin
kullanıcı ID'si (effective user ID)*, *gerçek grup ID'si (real group ID)* ile de *etkin grup ID'si
(effective group ID)* genellikle aynı olmaktadır. Ancak set-user-id ve set-group-id bayrakları set
edilmiş çalıştırılabilir programlar çalıştırıldığında bu ID'ler farklı hale gelebilmektedir. Örneğin
prosesimizin gerçek kullanıcı ID'si ve etkin kullanıcı ID'si ``kaan`` olsun. Biz set-user-id bayrağı set
edilmiş ``/bin/passwd`` programını exec yaptığımızda prosesimizin gerçek kullanıcı ID'si ``kaan`` olmaya
devam eder, ancak etkin kullanıcı ID'si ``root`` olur. Dosya işlemlerinde teste her zaman etkin ID'ler
sokulmaktadır.

Gerçek kullanıcı ID'si ve gerçek grup ID'si, etkin kullanıcı ID'si ve etkin grup ID'si dışında prosesin
bir de *saklı kullanıcı ID'si (saved set user ID)* ve *saklı grup ID'si (saved set group ID)* denilen
iki ID daha vardır. Bir proses exec uyguladığında programın set-user-id ve set-group-id bayrakları set
edilmiş olsun ya da olmasın, her zaman çekirdek yeni etkin kullanıcı ID'sini ve yeni etkin grup ID'sini
saklı kullanıcı ID'si ve saklı grup ID'si olarak set etmektedir. Örneğin prosesimizin gerçek kullanıcı
ID'si ``kaan`` ve etkin kullanıcı ID'si ``kaan``, gerçek grup ID'si ``study`` ve etkin grup ID'si
``study`` olsun. Şimdi biz set-user-id bayrağı set edilmiş olan ``/bin/passwd`` programını exec ile
çalıştıralım. Artık prosesimizin gerçek kullanıcı ID'si ``kaan``, etkin kullanıcı ID'si ``root``
olacaktır. Gerçek grup ID'si ``study`` ve etkin grup ID'si de ``study`` olarak kalacaktır. İşte çekirdek
aynı zamanda bu yeni etkin kullanıcı ID'sini (örneğimizdeki ``root`` ID'sini kastediyoruz) ve grup ID'sini
prosesin *saklı kullanıcı ID'si (saved set user ID)* ve *saklı grup ID'si* olarak da set etmektedir. O
halde prosesimizin ID'leri artık şöyle olacaktır:

.. code-block:: text

    gerçek kullanıcı ID'si: kaan
    etkin kullanıcı ID'si:  root
    saklı kullanıcı ID'si:  root
    gerçek grup ID'si:      study
    etkin grup ID'si:       study
    saklı grup ID'si:       study

Bu işlem set-user-id ya da set-group-id bayrağı set edilmemiş programlar çalıştırılırken de
yürütülmektedir. Örneğin prosesimizin gerçek kullanıcı ID'si ``kaan``, etkin kullanıcı ID'si ``kaan``,
gerçek grup ID'si ``study`` ve etkin grup ID'si ``study`` olsun. Biz de set-user-id bayrağı set edilmemiş
olan bir programı exec yapmış olalım. Yeni ID'ler şöyle olacaktır:

.. code-block:: text

    gerçek kullanıcı ID'si: kaan
    etkin kullanıcı ID'si:  kaan
    saklı kullanıcı ID'si:  kaan
    gerçek grup ID'si:      study
    etkin grup ID'si:       study
    saklı grup ID'si:       study

Tabii saklı kullanıcı ID'si ve saklı grup ID'si yine proses kontrol bloğu içerisinde (Linux'taki
``task_struct`` yapısı içerisinde) saklanmaktadır.

getuid, geteuid, getgid, getegid Fonksiyonları
==============================================

O anda çalışmakta olan prosesin (yani kendi prosesimizin) gerçek kullanıcı ID'si ``getuid`` isimli
POSIX fonksiyonuyla, etkin kullanıcı ID'si de ``geteuid`` isimli POSIX fonksiyonuyla elde
edilebilmektedir. Fonksiyonların prototipleri şöyledir:

.. code-block:: c

    #include <unistd.h>

    uid_t getuid(void);
    uid_t geteuid(void);

Daha önce belirttiğimiz gibi ``uid_t`` türü ``<unistd.h>`` ve ``<sys/types.h>`` dosyaları içerisinde bir
tamsayı türü olacak biçimde typedef edilmiştir. Bu fonksiyonlar başarısız olamamaktadır.

O anda çalışmakta olan prosesin gerçek grup ID'si ``getgid`` POSIX fonksiyonu ile, etkin grup ID'si ise
``getegid`` POSIX fonksiyonu ile elde edilebilmektedir. Fonksiyonların prototipleri şöyledir:

.. code-block:: c

    #include <unistd.h>

    gid_t getgid(void);
    gid_t getegid(void);

Daha önce de belirttiğimiz gibi ``gid_t`` türü ``<unistd.h>`` ve ``<sys/types.h>`` dosyaları içerisinde
bir tamsayı türü olacak biçimde typedef edilmiştir.

Bu fonksiyonlar da başarısız olamamaktadır.

POSIX standartlarında saklı ID'leri alan fonksiyonlar yoktur. Ancak Linux sistemlerinde bu işlemi
yapacak fonksiyon bulunmaktadır.

Örnek Program
-------------

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <stdint.h>
    #include <errno.h>
    #include <unistd.h>
    #include <pwd.h>
    #include <grp.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        uid_t ruid, euid;
        gid_t rgid, egid;
        struct passwd *pw;
        struct group *gr;

        ruid = getuid();
        euid = geteuid();

        errno = 0;
        if ((pw = getpwuid(ruid)) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid user name!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getpwuid");
        }

        printf("Real user ID: %jd (%s)\n", (intmax_t)ruid, pw->pw_name);

        errno = 0;
        if ((pw = getpwuid(euid)) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid user ID!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getpwuid");
        }
        printf("Effective user ID: %jd (%s)\n", (intmax_t)ruid, pw->pw_name);

        errno = 0;
        if ((gr = getgrgid(ruid)) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid group ID!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getgrgid");
        }

        rgid = getgid();
        egid = getegid();

        printf("Real group ID: %jd (%s)\n", (intmax_t)rgid, gr->gr_name);

        errno = 0;
        if ((gr = getgrgid(egid)) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid group ID!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getgrgid");
        }

        printf("Effective group ID: %jd (%s)\n", (intmax_t)ruid, pw->pw_name);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }


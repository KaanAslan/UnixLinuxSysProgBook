==================================================
**Dosya ve Dizin Erişimlerine İlişkin Ayrıntılar**
==================================================

Bu bölümde dosya ve dizin erişimlerinde etkili olan çeşitli kavramlar üzerinde duracağız. Bölüm içerisinde önce 
çalıştırılabilen dosyalara ilişkin *set-user-ID*, *set-group-ID* ve *sticky* erişim bayrakları duracağız. Sonra proseslerin
kullanıcı, grup ve saklı ID'lerinin nasıl elde edilip değiştirileceğini açıklayacağız.

Dosyaların set-user-ID, set-group-ID ve sticky Erişim Bayrakları
================================================================

Biz şimdiye kadar dosyalar için ``9`` erişim bayrağı gördük: ``S_IRUSR``, ``S_IWUSR``, ``S_IXUSR``,
``S_IRGRP``, ``S_IWGRP``, ``S_IXGRP``, ``S_IROTH``, ``S_IWOTH``, ``S_IXOTH``. Bu bayraklar dosyanın
``rwx rwx rwx`` erişim haklarını belirtmektedir. Ancak aslında dosyaların erişim bilgileri ``9`` tane değil ``12``
tanedir. Henüz görmediğimiz üç erişim bayrağına *set-user-ID*, *set-group-ID* ve *sticky* bayrakları
denilmektedir. Şimdi dikkatimizi bu üç erişim bayrağına çevireceğiz. Bu üç erişim bayrağı ``open``
fonksiyonunda ya da ``chmod`` fonksiyonunda sırasıyla ``S_ISUID``, ``S_ISGID`` ve ``S_ISVTX`` sembolik
sabitleriyle kullanılabilmektedir. Bu sembolik sabitler aslında en yüksek anlamlı dördüncü octal digit'e
karşılık gelmektedir:

.. code-block:: text

    UGS rwx rwx rwx rwx

Dolayısıyla POSIX 2008 ve sonrasında bu bayrakları yüksek anlamlı dördüncü octal digit'le de
belirtebiliriz.

Burada bir anımsatmada bulunmak istiyoruz. Daha önce de belirttiğimiz gibi POSIX standartlarında dosyanın sahip olduğu 
"UGS rwx rwx rwx rwx" bilgilerine dosyanın "mod (mode)" bilgileri denilmektedir. Biz kitabımızda bunlara "erişim bayrağı" 
diyeceğiz.

Bir dosyanın *set-user-ID* bayrağı  ``chmod`` komutunda ``u+s`` seçeneği kullanılarak set edilebilir. Örneğin:

.. code-block:: console

    $ ls -l sample
    -rwxrwxrwx 1 kaan study 17176 Şub 19 11:46 sample
    $ chmod u+s sample
    $ ls -l sample
    -rwsrwxrwx 1 kaan study 17176 Şub 19 11:46 sample

Görüldüğü gibi eğer dosyanın hem ``'x'`` hakkı hem de *set-user-ID* bayrağı set edilmişse ``x`` harfinin
bulunduğu yerde ``s`` harfi, yalnızca *set-user-ID* bayrağı set edilmişse ``x`` harfinin bulunduğu yerde ``S`` 
harfi görüntülenmektedir. Örneğin:

.. code-block:: console

    $ ls -l test.txt
    -rw-rw-rw- 1 kaan study 2087 Şub 19 10:49 test.txt
    $ chmod u+s test.txt
    $ ls -l test.txt
    -rwSrw-rw- 1 kaan study 2087 Şub 19 10:49 test.txt

Dosyanın *set-group-ID* bayrağının set edilmesi de ``chmod`` komutunda ``g+s`` ile yapılmaktadır. İşlem
sonrasında yine grup bilgisinde ``x`` harfi yerinde ``s`` ya da ``S`` harfi görüntülenir. ``chmod`` komutunda
``+s`` kullanılırsa bu durumda dosyanın hem *set-user-ID* hem de *set-group-ID* bayrakları set edilir.
Dosyanın *sticky* bayrağını set etmek için ``+t`` kullanılmaktadır. Bu işlem yapıldığında grupta ``'x'`` hakkı 
varsa ``x`` harfinin olduğu yerde ``t`` harfi, grupta ``'x'`` hakkı yoksa ``x`` harfinin olduğu yerde ``T`` harfi 
görüntülenmektedir.

Yukarıdaki komutlarda ``+`` yerine ``-`` karakterini kullanarak reset işlemlerini yapabilirsiniz.

Benzer biçimde istersek yine ``chmod`` komutunda octal digit'lerle *set-user-ID*, *set-group-ID* ve *sticky*
bitlerini set edebiliriz. Örneğin:

.. code-block:: console

    $ chmod 4755 sample

Burada en soldaki octal digit ``4`` olduğu için dosyanın *set-user-ID* bayrağı da set edilmiştir. En soldaki
octal digit'in bitleri yukarıda da belirttiğimiz gibi şu sıradadır:

.. code-block:: text

    *set-user-ID*  *set-group-ID*  sticky

``chmod`` fonksiyonunda yukarıda belirttiğimiz bayrakları kullanarak *set-user-ID*, *set-group-ID* ve *sticky*
bayraklarını set edebiliriz. Örneğin biz *sample* programını ``rwsrwxrwx`` haline şöyle getirebiliriz:

.. code-block:: c

    if (chmod("sample", S_IRWXU|S_IRWXG|S_IRWXO|S_ISUID) == -1)
        exit_sys("chmod");

Peki zaten var olan bir dosyaya mevcut erişim haklarını bozmadan bu özellikleri programlama yoluyla nasıl
ekleyebiliriz? Bunun için önce dosyanın erişim haklarının ``stat``, ``lstat`` ya da ``fstat``
fonksiyonuyla elde edilmesi gerekir. Ondan sonra bu erişim haklarına biz ``S_ISUID``, ``S_ISGID``
ve ``S_ISVTX`` bayraklarını ``|`` operatörüyle ekleyebiliriz. Ancak ``stat`` fonksiyonlarının bize verdiği
``st_mode`` değeri dosyanın türünü de içermektedir. Gerçi Linux'un ``chmod`` fonksiyonu bu ekstra bitleri dikkate
almamaktadır, fakat POSIX uyumu için ``stat`` yapısının ``st_mode`` elemanındaki değerin ``S_IFMT`` ile maskelenmesi
gerekir. (POSIX standratlarında ``chmod`` fonksiyonunun erişim bayraklarının dışında bayraklarla karşılaştığında başarısız 
olabileceği belirtilmiştir.) O halde bu işlem şöyle yapılabilir:

.. code-block:: c

    struct stat finfo;

    if (stat("sample", &finfo) == -1)
        exit_sys("stat");

    if (chmod("sample", (finfo.st_mode & ~S_IFMT) | S_ISUID) == -1)
        exit_sys("chmod");

``S_IFMT`` bayrağı erişim hakları dışındaki dosya tür bayraklarını temsil etmektedir. ``~S_IFMT`` erişim
hakları bayraklarını ``1``, diğer bayrakları ``0`` hale getirmektedir.

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

set-user-ID ve set-group-ID Erişim Bayraklarının Anlamı
-------------------------------------------------------

Peki bir dosyanın *set-user-ID* ve *set-group-ID* erişim haklarının set edilmiş olmasının ne anlamı vardır?
Öncelikle bu hakların yalnızca *çalıştırılabilir (executable) dosyalar için* anlamlı olduğunu belirtelim. Yani bu
bayraklar tasarımda *çalıştırılabilir dosyalar* için düşünülmüştür. Çalıştırılabilir dosyanın *set-user-ID*
erişim bayrağı set edilmişse bu dosya ``exec`` yapıldığında prosesin etkin kullanıcı ID'si işletim sistemi
tarafından dosyanın kullanıcı ID'si olacak biçimde değiştirilmektedir. Benzer biçimde çalıştırılabilir
dosyanın *set-group-ID* bayrağı set edilmişse bu dosya ``exec`` yapıldığında prosesin etkin grup ID'si
dosyanın grup ID'si olarak değiştirilmektedir.

*Set-user-id* bayrağının kullanım gerekçesini basit bir örnekle açıklayabiliriz. Bilindiği gibi kullanıcı
parolaları komut satırında ``passwd`` komutuyla değiştirilmektedir. Default durumda ``passwd`` programı
kullanıcının kendi parolasını değiştirmektedir. Örneğin:

.. code-block:: console

    $ passwd

Burada ``passwd`` programı önce mevcut parolayı sonra da yeni parolayı bize soracaktır. ``passwd`` programı
parolayı ``/etc/shadow`` dosyasına yazmaktadır. Bu dosyaya yalnızca ``root`` prosesler
erişebilmektedir:

.. code-block:: console

    $ ls -l /etc/passwd
    -rw-r--r-- 1 root root 3008 Haz 11 13:33 /etc/passwd

Peki biz ``passwd`` komutunu çalıştırdığımızda kabuk önce ``fork`` sonra exec yaptığına göre bizim
çalıştırdığımız ``passwd`` programı ``/etc/shadow`` dosyasına nasıl erişmektedir? İşte aslında
``/bin/passwd`` programının *set-user-ID* bayrağı set edilmiştir:

.. code-block:: console

    $ ls -l /bin/passwd
    -rwsr-xr-x 1 root root 64152 May 30  2024 /bin/passwd

Böylece bu dosya exec yapıldığında prosesin etkin kullanıcı id'si de *set-user-ID* bayrağının etkisiyle
``root`` olmaktadır. Şimdi siz bunun bir güvenlik açığı oluşturabileceğini düşünebilirsiniz. Ancak
aslında bir güvenlik açığı söz konusu değildir. Bu programın sahibi ``root`` kullanıcısıdır ve o
isteyerek bu programın ``root`` etkin kullanıcı ID'siyle çalıştırılmasına olanak sağlamıştır. Herhangi
bir kullanıcı zaten başkalarına ait dosyaların erişim haklarını değiştirememektedir.

Burada bir noktaya dikkat ediniz. *set-user-ID* ya da *set-group-ID* erişim hakları set edilmiş dosyalar ``exec``
yapılırken prosesin yalnızca etkin kullanıcı ID'si ve etkin grup ID'si değiştirilmektedir. Gerçek
kullanıcı ID'si ve gerçek grup ID'si değiştirilmemektedir. Biz genellikle gerçek ve etkin ID'lerin aynı
değerde olduğunu, ancak test işlemlerine etkin ID'lerin girdiğini belirtmiştik. İşte *set-user-ID* ve
*set-group-ID* bayrakları set edilmiş bir program dosyası exec yapıldığında artık gerçek kullanıcı ve
grup ID'leriyle etkin kullanıcı ID'leri farklılaşabilmektedir.

Aşağıdaki örnekte *sample* programı *other* programını ``exec`` yaparak çalıştırmıştır. *other*
programı prosesin gerçek ve etkin kullanıcı ve grup ID'lerini ekrana isim olarak yazdırmaktadır. Biz bu
örnekte *other* programını derledikten sonra ``chown`` komutuyla ``sudo`` ile birlikte kullanıcı
ID'sini ve grup ID'sini ``root`` olarak değiştirdik. Bu deneyi önce *other* programının *set-user-ID*
bayrağını set etmeden sonra da set ederek yapınız. Aşağıda yapılanlar özetlenmiştir:

.. code-block:: console

    $ gcc -Wall -o sample sample.c
    $ gcc -Wall -o other other.c
    $ ls -l other
    -rwxr-xr-x 1 kaan study 17088 Şub 19 13:42 other
    $ sudo chown root other
    $ ls -l other
    -rwxr-xr-x 1 root study 17088 Şub 19 13:42 other
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

Biz bir programı ``sudo`` ile ``root`` önceliğinde (etkin proses ID'si 0 olacak biçimde) çalıştırıyor
olalım. Dosyanın *set-user-ID* bayrağı da set edilmiş olsun. Bu durumda program çalışırken prosesin etkin
kullanıcı ID'si ``root`` değil, program dosyasının kullanıcı ID'si olacaktır. Yani *set-user-ID* bayrağı
set edilmiş olan programların ``sudo`` ile ``root`` önceliğinde çalıştırılmasının bir anlamı kalmamaktadır.
Örneğin:

.. code-block:: console

    $ sudo chown ali other
    $ ls -l other
    -rwxr-xr-x 1 ali study 16344 Eki  1 11:48 other
    $ sudo chmod u+s other
    $ ls -l other
    -rwsr-xr-x 1 ali study 16344 Eki  1 11:48 other
    $ sudo ./other
    Real user ID: root
    Effective user ID: ali
    Real group ID: root
    Effective group ID: root

Burada tek farklılık dosyanın grup ID'sinin de ``root`` haline gelmiş olmasıdır.

``sample.c``

.. code-block:: c

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

        if (pid == 0 && execl("other", "other", (char *)0) == -1)
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

``other.c`` 

.. code-block:: c

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

Dizinlerde set-group-ID Erişim Bayrağının İşlevi
------------------------------------------------

Normal olarak *set-user-ID* ve *set-group-ID* bayrakları çalıştırılabilir dosyalar için söz konusudur. Ancak
dizinler için de *set-group-ID* bayrağının bir anlamı vardır. Pek çok UNIX türevi sistemde (Linux da buna
dahil) bir dizinin *set-group-ID* bayrağı set edilirse Linux o dizin içerisinde ``open`` fonksiyonuyla
(zaten başka yolu yoktur) ya da ``mkdir`` fonksiyonuyla bir dosya ya da dizin yaratıldığında dosyanın ya
da dizinin grup ID'sini prosesin etkin grup ID'si olarak değil, o dizinin grup ID'si olarak set
etmektedir. BSD sistemlerinde zaten varsayılan durumda bir dosya ya da dizin yaratıldığında dosyanın ya da
dizinin grup ID'si o dosyanın ya da dizinin içinde bulunduğu dizinin grup ID'si olarak set edilmektedir.
O halde Linux'ta ``open`` fonksiyonu ile bir dosya ya da dizin yaratılırken dosya ya da dizinin grup
ID'si, eğer o dosya ya da dizinin içinde bulunduğu dizinin *set-group-ID* bayrağı set edilmemişse prosesin
etkin grup ID'si olarak, eğer set edilmişse dizinin grup ID'si olarak set edilmektedir. Bu durumu basit
bir biçimde şöyle test edebilirsiniz:

Önce ``xxx`` gibi bir isimle bir dizin yaratalım:

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
*set-group-ID* erişim bayrağını set edelim:

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

Dizinlerin *set-user-ID* bayraklarının set edilmesi benzer bir etkiye yol açmamaktadır. (Bu durum bazı eski
UNIX sistemlerinde denenmiştir, ancak modern sistemlerde böyle bir semantik yoktur.)

Peki betik dosyaları (shebang içeren text dosyalar) için *set-user-ID* ve *set-group-ID* bayrakları set
edilebilir mi? Bu işlem ilk zamanlar uygulanmıştır. Ancak güvenlik açığı nedeniyle sonra uygulamadan
kaldırılmıştır. Bugünkü modern UNIX/Linux sistemleri betik dosyalarının *set-user-ID* ve *set-group-ID*
hakları set edilmiş olsa bile onları dikkate almamaktadır. Buradaki güvenlik açığı ilginç bir
biçimde şöyle oluşmaktadır:

Betik dosyasının ismi ``x.txt`` olsun. Bu dosyanın *set-user-ID* ve *set-group-ID* haklarının dikkate alınmış
olduğunu varsayalım. Bu durumda ``exec`` fonksiyonları bu dosyayı açıp shebang satırında bulunan programı çalıştırırken
prosesin etkin kullanıcı ve/veya grup ID'sini ``x.txt`` dosyasının kullanıcı ve/veya grup ID'si olarak
set edecektir. Bu durumda da eğer birisi örneğin ``y.txt`` gibi bir sembolik bağlantı dosyası oluşturup bu dosyanın
``x.txt`` dosyasını göstermesini sağlarsa ve bu ``y.txt`` ile ``exec`` yaparsa bu durumda aslında ``exec``
fonksiyonu sembolik bağlantıyı izleyecek ve ``x.txt`` dosyasını çalıştıracaktır. Ancak ``exec`` fonksiyonu
bu ``x.txt`` dosyasının shebang satırındaki programı çalıştırırken komut satırı argümanı olarak
``y.txt`` dosyasını kullanacaktır. İşte tam bu sırada birisi bu ``y.txt`` dosyasının sembolik
bağlantısını değiştirirse maalesef prosesin etkin kullanıcı ID'si ``x.txt``'nin kullanıcı ID'si olacak
biçimde başka dosyayı çalıştıracaktır.

sticky Erişim Bayrağı
---------------------

Peki dosyaların *sticky* bayraklarır (modu'nun) ne işlevi vardır? Aslında *sticky* tasarımda başka bir amaçla
düşünülmüştür. Eski sistemlerde bu ``S_ISVTX`` bayrağı çalıştırılabilir dosyaların çalıştırılması sonrasında programın
bellekten atılmaması gibi bir ipucu oluşturmaktaydı. Ancak modern sistemlerde böyle bir etkinin bir
anlamı kalmadığı için *sticky* bayrağı da ilk tasarlandığı zamanki işlevinden tamamen kopmuştur. Bugün
sticky bayrağı değişik sistemlerde değişik amaçlarla kullanılabilmektedir. POSIX standartları eskiden
sticky bayrağı üzerinde açıklama yapmıyordu. Ancak belli zamandan sonra *sticky* için şöyle bir
işlevsellik tanımlanmıştır: Bir dizinin *sticky* bayrağı set edilirse ve dizinin sahibi başkaları ise,
dizine prosesin yazma hakkı olsa bile dizin içerisindeki başkalarına ait olan (yani kullanıcı ID'si işlemi uygulayan 
prosesin kullanıcı ID'sinden farklı olan) dosyalar silinememekte ve ismi değiştirilememektedir". Bugünkü 
sistemlerde dizin dışında diğer dosyaların *sticky* bayraklarının set edilmiş olup olmamasının işlevsel bir anlamı yoktur. 
Örneğin Linux sistemlerinde ``/tmp`` dizininin *sticky* bayrağı set edilmiştir ve bu dizine yazma hakkı verilmiştir. Bu
durumda biz bu dizinde dosya yaratabiliriz, dizindeki kendi dosyamızı silebiliriz. Ancak başkalarının dosyalarını
silemeyiz. ``/tmp`` dizininin erişim hakları şöyledir:

.. code-block:: text

    drwxrwxrwt 19 root root 65536 Şub 25 10:05 /tmp

Linux sistemlerinde *sticky* bayrağının dizin üzerindeki etkisini şöyle test edebiliriz:

Önce ``yyy`` isimli bir dizin yaratalım, sonra bu dizinin içine geçip orada sahibi ``root`` olan bir dosya
yaratalım:

.. code-block:: console

    $ mkdir yyy
    $ cd yyy
    $ sudo touch x.txt
    $ ls -l x.txt
    -rw-r--r-- 1 root root 0 Eki  1 12:53 x.txt

Görüldüğü gibi dosyanın kullanıcı ve grup ID'si ``sudo`` işleminden dolayı ``root`` olmuştur. Şimdi
burada ``sudo`` uygulamadan ikinci bir dosya da yaratalım:

.. code-block:: console

    $ touch y.txt
    $ ls -l x.txt y.txt
    -rw-r--r-- 1 root root  0 Eki  1 12:53 x.txt
    -rw-r--r-- 1 kaan study 0 Eki  1 12:55 y.txt

Biz her iki dosyayı da silebiliriz. Çünkü bir dosyayı silebilmek için dosyanın sahibi olmaya, dosyaya
``w`` hakkına sahip olmaya gerek yoktur. Tek gereken şey dizin için ``w`` hakkına sahip olmaktadır. Şimdi
dizinin sahipliğini değiştirelim ve *sticky* bayrağını set edip, herkese ``w`` hakkı verelim:

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

Görüldüğü gibi kendimize ait olan dosyaları silebilmekteyiz. 

Proseslerin Saklı Kullanıcı ve Grup ID'leri
===========================================

Daha önceden de belirttiğimiz gibi bir prosesin *gerçek kullanıcı ID'si (real user ID)* ile *etkin
kullanıcı ID'si (effective user ID)*, *gerçek grup ID'si (real group ID)* ile de *etkin grup ID'si
(effective group ID)* genellikle aynı olmaktadır. Ancak *set-user-ID* ve *set-group-ID* bayrakları set
edilmiş çalıştırılabilir programlar çalıştırıldığında bu ID'ler farklı hale gelebilmektedir. Örneğin
prosesimizin gerçek kullanıcı ID'si ve etkin kullanıcı ID'si ``kaan`` olsun. Biz *set-user-ID* bayrağı set
edilmiş ``/bin/passwd`` programını exec yaptığımızda prosesimizin gerçek kullanıcı ID'si ``kaan`` olmaya
devam eder, ancak etkin kullanıcı ID'si ``root`` olur. Dosya işlemlerinde teste her zaman etkin ID'ler
sokulmaktadır.

Gerçek kullanıcı ID'si ve gerçek grup ID'si, etkin kullanıcı ID'si ve etkin grup ID'si dışında prosesin
bir de *saklı kullanıcı ID'si (saved set user ID)* ve *saklı grup ID'si (saved set group ID)* denilen
iki ID'si daha vardır. Bir proses exec uyguladığında programın *set-user-ID* ve *set-group-ID* hakları set
edilmiş olsun ya da olmasın, her zaman çekirdek yeni etkin kullanıcı ID'sini ve yeni etkin grup ID'sini
saklı kullanıcı ID'si ve saklı grup ID'si olarak set etmektedir. Örneğin prosesimizin gerçek kullanıcı
ID'si ``kaan`` ve etkin kullanıcı ID'si ``kaan``, gerçek grup ID'si ``study`` ve etkin grup ID'si
``study`` olsun. Şimdi biz *set-user-ID* bayrağı set edilmiş olan ``/bin/passwd`` programını ``exec`` ile
çalıştıralım. Artık prosesimizin gerçek kullanıcı ID'si ``kaan``, etkin kullanıcı ID'si ``root``
olacaktır, gerçek grup ID'si ``study`` ve etkin grup ID'si ``study`` olarak kalacaktır. İşte çekirdek
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

Bu işlem *set-user-ID* ya da *set-group-ID* hakları set edilmemiş programlar çalıştırılırken de
yürütülmektedir. Örneğin prosesimizin gerçek kullanıcı ID'si ``kaan``, etkin kullanıcı ID'si ``kaan``,
gerçek grup ID'si ``study`` ve etkin grup ID'si ``study`` olsun. Biz de *set-user-ID* bayrağı set edilmemiş
olan bir programı ``exec`` yapmış olalım. Yeni ID'ler şöyle olacaktır:

.. code-block:: text

    gerçek kullanıcı ID'si: kaan
    etkin kullanıcı ID'si:  kaan
    saklı kullanıcı ID'si:  kaan
    gerçek grup ID'si:      study
    etkin grup ID'si:       study
    saklı grup ID'si:       study

Tabii saklı kullanıcı ID'si ve saklı grup ID'si yine proses kontrol bloğu içerisinde (Linux'taki
``task_struct`` yapısı içerisinde) saklanmaktadır.

Proseslerin ID'lerini Elde Eden ve Onları Değiştiren Fonksiyonlar
=================================================================

UNIX/Linux sistemlerinde proseslerin ID'lerini elde eden ve değiştren çeşitli fonksiyonlar bulunmaktadır. Bu 
fonksiyonların bazıları eskiden POSIX standartlarında yoktu. Ancak zaman içerisinde Linux ve/veya BSD sistemlerinde 
olup da POSIX standartlarında olmayan fonksiyonalar da POSIX standartlarına sokuldu. Biz bu bölümde tek tek 
alt başlıklar halinde bunları ele alıp açıklayacağız.

getuid, geteuid, getgid, getegid Fonksiyonları
----------------------------------------------

O anda çalışmakta olan prosesin (yani kendi prosesimizin) gerçek kullanıcı ID'si ``getuid`` isimli
POSIX fonksiyonuyla, etkin kullanıcı ID'si de ``geteuid`` isimli POSIX fonksiyonuyla elde edilebilmektedir. 
Fonksiyonların prototipleri şöyledir:

.. code-block:: c

    #include <unistd.h>

    uid_t getuid(void);
    uid_t geteuid(void);

Daha önce belirttiğimiz gibi ``uid_t`` türü ``<unistd.h>`` ve ``<sys/types.h>`` dosyaları içerisinde bir
tamsayı türünü belirtecek biçimde typedef edilmiştir. Bu fonksiyonlar başarısız olamamaktadır.

O anda çalışmakta olan prosesin gerçek grup ID'si ``getgid`` POSIX fonksiyonu ile, etkin grup ID'si ise
``getegid`` POSIX fonksiyonu ile elde edilebilmektedir. Fonksiyonların prototipleri şöyledir:

.. code-block:: c

    #include <unistd.h>

    gid_t getgid(void);
    gid_t getegid(void);

Daha önce de belirttiğimiz gibi ``gid_t`` türü ``<unistd.h>`` ve ``<sys/types.h>`` dosyaları içerisinde
bir tamsayı türünü beliretecek biçimde typedef edilmiştir. Bu fonksiyonlar da başarısız olamamaktadır. 

Aşağıda bu fonksiyonların kullanımına ilişkin bir örnek veriyoruz:

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

.. _setuid-ailesi-ve-ek-gruplar:

getresuid ve getresgid Fonksiyonları
------------------------------------

Eskiden POSIX standartlarında saklı kullanıcı ve grup ID'lerini almanın bir yolu yoktu. Fakat Linux
sistemleri buna izin veriyordu. POSIX'in 2024 versiyonu ile birlikte ``getresuid`` ve ``getresgid``
fonksiyonları bu ID'lerin hepsini tek hamlede elde etmeye izin vermektedir.

.. code-block:: c

    #define _GNU_SOURCE         /* See feature_test_macros(7) */
    #include <unistd.h>

    int getresuid(uid_t *ruid, uid_t *euid, uid_t *suid);
    int getresgid(gid_t *rgid, gid_t *egid, gid_t *sgid);

Fonksiyonlar ilgili nesnelerin adreslerini alıp değerleri oraya yerleştirmektedir. Başarı durumunda ``0``,
başarısızlık durumunda ``-1`` değerine geri dönmektedir. (Fonksiyonlarn ancak geçersiz bir adres girildiğinde 
başarız olabileceğine dikkat ediniz.)

Eskiden bu *glibc* fonksiyonlarını kullanabilmek için ``_GNU_SOURCE`` makrosunun programın başında define
edilmesi gerekiyordu. Ancak güncel durumda bu zorunluluk ortadan kalkmıştır. Ancak eski *glibc*
versiyonlarıyla çalışıyorsanız bu makroyu define etmelisiniz.

Aşağıdaki örnekte prosesin gerçek, etkin ve saklı kullanıcı ID'leri ``getresuid`` fonksiyonuyla elde
edilip ekrana yazdırılmıştır.

.. code-block:: c

    #define _GNU_SOURCE

    #include <stdio.h>
    #include <stdlib.h>
    #include <stdint.h>
    #include <errno.h>
    #include <unistd.h>
    #include <pwd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        uid_t ruid, euid, ssuid;
        struct passwd *pass;

        if (getresuid(&ruid, &euid, &ssuid) == -1)
            exit_sys("getresuid");

        errno = 0;
        if ((pass = getpwuid(ruid)) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid user id!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getpwuid");
        }

        printf("Real User ID: %s (%ju)\n", pass->pw_name, (uintmax_t)ruid);

        errno = 0;
        if ((pass = getpwuid(euid)) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid user id!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getpwuid");
        }

        printf("Effective User ID: %s (%ju)\n", pass->pw_name, (uintmax_t)euid);

        errno = 0;
        if ((pass = getpwuid(ssuid)) == NULL) {
            if (errno == 0) {
                fprintf(stderr, "invalid user id!..\n");
                exit(EXIT_FAILURE);
            }
            exit_sys("getpwuid");
        }

        printf("Saved Set User ID: %s (%ju)\n", pass->pw_name, (uintmax_t)ssuid);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

setuid, seteuid, setgid ve setegid Fonksiyonları
------------------------------------------------

Prosesin kullanıcı ve grup ID'lerini set etmek için ``setuid``, ``setgid``, ``seteuid`` ve ``setegid``
POSIX fonksiyonları bulundurulmuştur. Ancak bu fonksiyonların semantiği kişilere biraz karmaşık
gelmektedir. Biz burada bu fonksiyonları tek tek açıklayacağız.

``setuid`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int setuid(uid_t uid);

Her ne kadar fonksiyonun ismi sanki yalnızca prosesin yalnızca gerçek kullanıcı ID'sini değiştiriyor gibi
görünüyorsa da aslında fonksiyon şöyle davranmaktadır:

| **1.** Prosesin önceliği (appropriate privileges) uygunsa yani prosesin etkin kullanıcı ID'si ``0`` ise ya da
   Linux sistemlerinde proses bu işi yapacak yeteneğe (*capability*) sahipse (``CAP_SETUID``) fonksiyon
   prosesin hem gerçek kullanıcı ID'sini, hem etkin kullanıcı ID'sini hem de saklı kullanıcı ID'sini
   parametresi ile belirtilen kullanıcı ID'si yapar. Yani eğer uygun önceliğe sahipsek biz bu fonksiyonla
   prosesin gerçek kullanıcı ID'sini, etkin kullanıcı ID'sini ve saklı kullanıcı ID'sini tek hamlede
   değiştirebiliriz.

| **2.** Eğer prosesin önceliği uygun değilse (yani proses ``root`` değilse ya da Linux'ta ``CAP_SETUID`` yeteneğine
   sahip değilse) bu durumda ``setuid`` fonksiyonu eğer parametresi ile belirtilen uid değeri prosesin gerçek ya
   da saklı kullanıcı ID'si ile aynı ise prosesin yalnızca etkin kullanıcı ID'sini parametresi ile
   belirtilen ID olarak değiştirir. Bu koşullar sağlanmıyorsa fonksiyon başarısız olmaktadır. Peki bunun
   anlamı nedir? Bu işlem set-user-ID programların daha sonra dönüp yeniden önceliği geri almasını
   sağlamak için düşünülmüştür. Saklı ID'lerin kullanılmasının tek nedeni de budur. Şöyle ki:

Biz ``kaan`` prosesi olarak set-user-ID bayrağı set edilmiş ``root`` programını çalıştırdığımızı
düşünelim. Şimdi bizim ID'lerimiz şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  root
    Saklı kullanıcı ID'si:  root

Şimdi biz sonraki paragrafta göreceğimiz ``seteuid`` fonksiyonu ile geri dönüp prosesin yeteneğini
azaltarak onun etkin kullanıcı ID'sinin ``kaan`` olmasını sağlayalım. Bu durumdaki ID'ler şöyle
olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  root

İşte şimdi bu noktada biz ``setuid`` fonksiyonu ile yeniden ``root`` önceliğine dönebiliriz. Çünkü bizim
şu anda uygun önceliğimiz olmasa da saklı kullanıcı ID'miz hala ``root`` durumundadır. 

``setuid`` fonksiyonu başarı durumunda ``0`` değerine, başarısızlık durumunda ``-1`` değerine geri dönmekte ve
başarısızlık durumunda ``errno`` değişkeni uygun biçimde set edilmektedir.

Örneğin ``login`` programı eğer parola doğrulamasını yaparsa ``setuid`` fonksiyonuyla tek hamlede
prosesin hem gerçek kullanıcı ID'sini, hem etkin kullanıcı ID'sini hem de saklı kullanıcı ID'sini
``/etc/passwd`` dosyasında belirtildiği gibi değiştirmektedir.

``seteuid`` fonksiyonu ``setuid`` fonksiyonunun yalnızca etkin kullanıcı ID'sini değiştiren biçimidir.
Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int seteuid(uid_t uid);

Fonksiyonun çalışması şöyledir:

| **1.** Prosesin önceliği uygunsa yani prosesin etkin kullanıcı ID'si 0 ise ya da proses Linux sistemlerinde 
   bu işi yapacak yeteneğe (capability) sahipse (``CAP_SETUID``) fonksiyon prosesin yalnızca etkin kullanıcı ID'sini 
   parametresiyle belirtilen ID olarak değiştirir. Bu fonksiyon prosesin gerçek kullanıcı ID'sini ve saklı kullanıcı 
   ID'sini değiştirmez.

| **2.** Eğer prosesin önceliği uygun değilse (yani proses ``root`` değilse ya da Linux'ta ``CAP_SETUID``
   yeteneğine sahip değilse) bu durumda ``seteuid`` fonksiyonu eğer parametresi ile belirtilen uid
   değeri prosesin gerçek ya da saklı kullanıcı ID'si ile aynı ise prosesin yalnızca etkin kullanıcı
   ID'sini parametresi ile belirtilen ID olarak değiştirir. Bu maddedeki davranış ``setuid``
   fonksiyonuyla aynıdır.

Görüldüğü gibi eğer prosesin önceliği uygun değilse zaten ``setuid`` fonksiyonu ile ``seteuid``
fonksiyonu arasında bir fark kalmamaktadır.

Bu durumda geri dönüş senaryosu tam olarak şöyle gerçekleştirilir. Yine prosesimizin ID'leri şöyle olsun:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  kaan

Şimdi biz set-user-ID bayrağı set edilmiş kullanıcı ID'si ``root`` olan bir programı ``exec`` yapalım.
ID'lerimiz şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si: root
    Saklı kullanıcı ID'si: root

Şimdi çalıştırdığımız program ``kaan`` olarak bazı şeyleri yapmak istesin, o zaman şu çağrıyı yapacaktır:

.. code-block:: c

    seteuid(getuid());

Şimdi prosesin ID'leri şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  root

Burada saklı kullanıcı ID'sinin değişmediğine dikkat ediniz. Şimdi proses ``kaan`` olarak bazı şeyleri
yaptıktan sonra yeniden ``root`` olmak istesin:

.. code-block:: c

    seteuid(0);

Tabii aynı işlem ``setuid(0)`` ile de yapılabilirdi. Şimdi prosesin ID'leri şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  root
    Saklı kullanıcı ID'si:  root

Eğer saklı kullanıcı ID'si diye bir kavram olmasaydı bu biçimde bir geri dönüş yapılamazdı.

``seteuid`` fonksiyonu başarı durumunda ``0`` değerine, başarısızlık durumunda ``-1`` değerine geri dönmektedir.
``errno`` değişkeni uygun biçimde set edilmektedir.

``setgid`` fonksiyonunun temel semantiği ``setuid`` fonksiyonunda olduğu gibidir. Fonksiyonun prototipi
şöyledir:

.. code-block:: c

    #include <unistd.h>

    int setgid(gid_t gid);

Fonksiyon şöyle çalışmaktadır:

| **1.** Prosesin önceliği uygunsa (appropriate privileges) yani prosesin etkin kullanıcı ID'si ``0`` ise ya da
   Linux sistemlerinde proses bu işi yapacak yeteneğe (capability) sahipse (``CAP_SETGID``) fonksiyon
   prosesin hem gerçek grup ID'sini, hem etkin grup ID'sini hem de saklı grup ID'sini parametresi ile
   belirtilen grup ID'si yapar.

| **2.** Eğer prosesin önceliği uygun değilse (yani ``root`` değilse ya da Linux'ta ``CAP_SETGID`` yeteneği
   yoksa) bu durumda ``setgid`` fonksiyonu eğer parametresi ile belirtilen gid değeri prosesin gerçek ya
   da saklı grup ID'si ile aynı ise prosesin yalnızca etkin grup ID'sini parametresi ile belirtilen grup
   ID olarak değiştirir.

Buradaki amaç tamamen set-group-ID bayrağı set edilmiş programların grup ID'lerini geri döndürüp yeniden
eski değere set edebilmesidir.

``setegid`` fonksiyonunun temel semantiği de ``seteuid`` fonksiyonunda olduğu gibidir. Yani fonksiyon
yalnızca etkin grup ID'lerini değiştirmektedir. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int setegid(gid_t gid);

Fonksiyon şöyle çalışmaktadır:

| **1.** Prosesin önceliği uygunsa yani prosesin etkin kullanıcı ID'si ``0`` ise ya da Linux sistemlerinde 
   proses bu işi yapacak yeteneğe (capability) sahipse (``CAP_SETGID``) fonksiyon prosesin yalnızca etkin 
   grup ID'sini parametresiyle belirtilen grup ID'si olarak değiştirmektedir.

| **2.** Eğer prosesin önceliği uygun değilse (yani ``root`` değilse ya da Linux'ta ``CAP_SETGID`` yeteneği
   yoksa) bu durumda ``setegid`` fonksiyonu eğer parametresi ile belirtilen gid değeri prosesin gerçek ya
   da saklı grup ID'si ile aynı ise prosesin yalnızca etkin grup ID'sini parametresi ile belirtilen grup
   ID olarak değiştirir.

Yani eğer prosesin önceliği uygun değilse bu durumda ``setgid`` ile ``setegid`` fonksiyonları arasında
bir fark kalmamaktadır.

``setgid`` ve ``setegid`` fonksiyonları yine grup ID bakımından geriye dönüşü sağlamak için
düşünülmüştür. Şöyle ki: Prosesimizin işin başındaki ID'leri şöyle olsun:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  kaan
    Gerçek grup ID'si:      study
    Etkin grup ID'si:       study
    Saklı grup ID'si:       study

Şimdi biz set-group-ID bayrağı set edilmiş bir programı çalıştıralım. Program dosyasının kullanıcı ID'si
``ali``, grup ID'si ise ``test`` olsun. Biz exec yaptığımızda ID'lerimiz şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  kaan
    Gerçek grup ID'si:      study
    Etkin grup ID'si:       test
    Saklı grup ID'si:       test

Şimdi biz grup olarak geçmişe dönüp bazı şeyleri yapmak isteyelim:

.. code-block:: c

    setegid(getgid());

Tabii burada ``setgid`` fonksiyonunu da kullanabilirdik. Şimdi prosesimizin ID'leri şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  kaan
    Gerçek grup ID'si:      study
    Etkin grup ID'si:       study
    Saklı grup ID'si:       test

Şimdi eski grup ID'ye yeniden geri dönmek isteyelim:

.. code-block:: c

    setegid(test_gid);

Tabii burada yine ``setgid`` fonksiyonu da kullanılabilirdi. Görüldüğü gibi grup temelinde geri dönüp
yeniden aynı etkin grup ID'ye sahip olabilmek için saklı grup ID'den faydalanılmıştır.

Geri Dönüş Senaryosu: setuid ve seteuid Farkı
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Bizim prosesimizin kullanıcı ID'leri şöyle olsun:

.. code-block:: text

    Gerçek kullanıcı ID'si: root
    Etkin kullanıcı ID'si:  root
    Saklı kullanıcı ID'si:  root

Şimdi biz aşağıdaki gibi bir çağrı yaparsak artık geri dönüş olanağımız kalmaz:

.. code-block:: c

    setuid(uid_kaan);

Çünkü uygun önceliğe sahip olan program ``setuid`` fonksiyonunu uyguladığında yalnızca kullanıcı etkin
ID'si değil, gerçek kullanıcı ID'si ve saklı kullanıcı ID'si de değişmektedir. Yani bu işlem sonucunda
kullanıcı ID'leri şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  kaan

Ancak biz aşağıdaki çağrıyı yapmış olsaydık geri dönebilirdik:

.. code-block:: c

    seteuid(uid_kaan);

Şimdi kullanıcı ID'leri şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: root
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  root

Şimdi artık ``setuid`` ya da ``seteuid`` fonksiyonu ile geri dönüş mümkündür.

setreuid ve setregid Fonksiyonları
----------------------------------

Yukarıdaki fonksiyonlara ek olarak ``setreuid`` ve ``setregid`` isimli iki yardımcı fonksiyon da
bulunmaktadır. (Bu fonksiyonların isimlerindeki *re* harfleri *real* ve *effective* sözcüklerinden
gelmektedir.) Aslında bu fonksiyonlara mutlak anlamda gerek yoktur. Ancak bu fonksiyonlar bazı işlemleri
kolaylaştırmaktadır. Bu fonksiyonlar ilk kez BSD UNIX sistemlerinde kullanılmış daha sonra POSIX
standartlarına dahil edilmiştir. Bu fonksiyonların ana amacı tek hamlede prosesin gerçek ve etkin
kullanıcı ID'lerini, gerçek ve etkin grup ID'lerini değiştirebilmektedir. Genellikle gerçek
ID'lerle etkin ID'leri yer değiştirmek amacıyla bu fonksiyon kullanılmaktadır.

``setreuid`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int setreuid(uid_t ruid, uid_t euid);

Fonksiyon parametre olarak değiştirilmek istenen gerçek kullanıcı ID'sini ve etkin kullanıcı ID'sini
almaktadır. Eğer bunlardan herhangi biri değiştirilmek istenmiyorsa bu durumda o parametre için ``-1``
girilir.

Fonksiyon şöyle çalışmaktadır:

| **1.** Eğer proses uygun önceliğe sahipse (yani ``root`` ise ya da Linux sistemlerinde ``CAP_SETUID``
   yeteneğine sahipse) her iki ID'yi de değiştirir. Yani bu durumda herhangi bir kontrol yapılmaz.

| **2.** Eğer proses uygun önceliğe sahip değilse fonksiyon prosesin etkin kullanıcı ID'sini yalnızca argüman 
   olarak girilen ``euid`` değeri prosesin gerçek kullanıcı ID'si ya da, etkin kullanıcı ID'si ya da saklı 
   kullanıcı ID'si ile aynı ise değiştirebilmektedir.

| **3.** POSIX standartlarında eğer proses uygun önceliğe sahip değilse bu fonksiyon ile gerçek kullanıcı
   ID'sinin etkin kullanıcı ID'si ya da saklı kullanıcı ID'si biçiminde değiştirilip
   değiştirilemeyeceği *belirsiz (unspecified)* bırakılmıştır. Linux sistemlerinde ``setreuid``
   fonksiyonu *prosesin gerçek kullanıcı ID'sini etkin kullanıcı ID'si olarak* değiştirebilmektedir.
   Ancak saklı kullanıcı ID'si olarak değiştirememektedir.

| **4.** Bu fonksiyon eğer set edilmek istenen etkin kullanıcı ID'si (yani hedef etkin kullanıcı ID'si)
   prosesin o andaki gerçek kullanıcı ID'sine eşit değilse bu durumda saklı kullanıcı ID'sini de etkin
   kullanıcı ID'si olarak set etmektedir. Aynı zamanda fonksiyon yine eğer gerçek kullanıcı ID'si set
   ediliyorsa saklı kullanıcı ID'sini hedef etkin kullanıcı ID'si olarak set etmektedir.

Fonksiyonun parametrelerinden biri bile uygunsuzsa fonksiyon başarısız olmaktadır. Fonksiyon başarı
durumunda ``0`` değerine, başarısızlık durumunda ``-1`` değerine geri döner ve ``errno`` değişkeni uygun
biçimde set edilir. Örneğin:

.. code-block:: c

    setreuid(getuid(), getuid());

Burada çağrı sonucunda prosesin gerçek kullanıcı ID'si değişmeyecektir. Ancak prosesin etkin kullanıcı
ID'si ve saklı kullanıcı ID'si gerçek kullanıcı ID'si olarak set edilecektir. Artık geri dönüş mümkün
değildir. Örneğin:

.. code-block:: c

    setreuid(geteuid(), getuid());

Burada prosesin gerçek kullanıcı ID'si ile etkin kullanıcı ID'si yer değiştirilmiştir. Örneğin prosesin
çağrı öncesindeki kullanıcı ID'leri şöyle olsun:

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  root
    Saklı kullanıcı ID'si:  root

Şimdi şu çağrıyı yapalım:

.. code-block:: c

    setreuid(geteuid(), getuid());

Şimdi ID'ler şöyle olacaktır:

.. code-block:: text

    Gerçek kullanıcı ID'si: root
    Etkin kullanıcı ID'si:  kaan
    Saklı kullanıcı ID'si:  root

Buradan eski duruma şöyle dönebiliriz:

.. code-block:: c

    setreuid(geteuid(), getuid());

.. code-block:: text

    Gerçek kullanıcı ID'si: kaan
    Etkin kullanıcı ID'si:  root
    Saklı kullanıcı ID'si:  root

Tabii yukarıda da belirttiğimiz gibi bu fonksiyonda proses uygun önceliğe sahip değilse gerçek kullanıcı ID'sinin etkin
kullanıcı ID'si olarak değiştirilip değiştirilemeyeceği POSIX sistemlerinde belirsiz bırakılmıştır.
Ancak Linux ve BSD sistemleri bunu yapabilmektedir.

``setregid`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int setregid(gid_t rgid, gid_t egid);

``setregid`` fonksiyonu da ``setreuid`` fonksiyonu ile grup temelinde benzer semantiğe sahiptir:

| **1.** Eğer proses uygun önceliğe sahipse (yani ``root`` ise ya da Linux sistemlerinde ``CAP_SETGID``
   yeteneğine sahipse) fonksiyon her iki ID'yi de değiştirir. Herhangi bir kontrol yapılmaz.

| **2.** Eğer proses uygun önceliğe sahip değilse fonksiyon gerçek grup ID'sini saklı grup ID'si olarak, etkin grup ID'sini 
   ise gerçek ya da saklı grup ID'si olarak değiştirebilmektedir. 

| **3.** Bu fonksiyon eğer set edilmek istenen etkin grup ID'si (yani hedef etkin grup ID'si) prosesin o
   andaki gerçek grup ID'sine eşit değilse bu durumda saklı grup ID'sini de etkin grup ID'si olarak set
   etmektedir. Aynı zamanda fonksiyon yine eğer gerçek grup ID'si set ediliyorsa saklı grup ID'sini
   hedef etkin grup ID'si olarak set etmektedir.

setresuid ve setresgid Fonksiyonları
------------------------------------

Linux sistemlerinde uzun zamandır ``getresuid`` ve ``getresgid`` fonksiyonlarının bir de set'li
versiyonları bulunmaktadır. Ancak bu fonksiyonlar POSIX tarafından desteklenmiyordu. POSIX'in 2024
versiyonuyla birlikte bu fonksiyonlar POSIX standartlarına da sokulmuştur:

.. code-block:: c

    #define _GNU_SOURCE            /* See feature_test_macros(7) */
    #include <unistd.h>

    int setresuid(uid_t ruid, uid_t euid, uid_t suid);
    int setresgid(gid_t rgid, gid_t egid, gid_t sgid);

Fonksiyonlar sırasıyla gerçek, etkin ve saklı ID'leri alarak proses için set işlemi yapmaktadır. Set
işlemi için şu koşullar bulunmaktadır: Uygun önceliğe sahip olmayan prosesler gerçek, etkin ve saklı
ID'lerini ancak o andaki gerçek, etkin ve saklı ID'lerinden biri olarak değiştirebilirler. Uygun
önceliğe sahip olan prosesler bunları herhangi bir biçimde değiştirebilirler. Yine bu fonksiyonlar
herhangi bir parametrede uygunsuz bir durumla karşılaşırsa tüm işlem başarısız olurlar ve -1 değeri ile
geri dönerler. Başarı durumunda fonksiyonlar 0 ile geri dönmektedir.

Eskiden bu *glibc* fonksiyonlarını kullanabilmek için ``_GNU_SOURCE`` makrosunun programın başında define
edilmesi gerekiyordu. Ancak güncel durumda bu zorunluluk ortadan kalkmıştır. Ancak eski *glibc*
versiyonlarıyla çalışıyorsanız bu makroyu define etmelisiniz.

Proseslerin Ek Grup ID'leri 
===========================

Biz daha önce *ek grup ID'leri (supplementary group IDs)* bahsetmiştik. Bu başlık altında konunun ayrıntılarına gireceğiz. 

İlk zamanlar UNIX sistemlerinde prosesler tek bir grup ile ilişkilendirilmişti. Sonra bir prosesin birden
fazla grup ile ilişkili olması gerektiği anlaşıldı ve sorun *ek gruplar (supplementary groups)*
kavramı ile çözüldü. Ek gruplar konusu uzun süredir POSIX standartlarının içerisinde var olan 
ve neredeyse tüm UNIX türevi sistemlerin desteklediği bir özelliktir.

Bir kullanıcının gerçek bir grubu vardır. Dolayısıyla işin başında gerçek grup ID'si ve etkin grup ID'si
bu gruptur. Anımsanacağı gibi dosya erişimlerinde test işlemlerine etkin kullanıcı ID'si ve etkin grup
ID'si girmektedir. Yine anımsanacağı gibi bir dosyanın ``rwx`` biçimindeki erişim haklarının hangi üçlük 
kısmının dikkate alınacağı şöyle belirleniyordu:

.. code-block:: text

    if (prosesin etkin kullanıcı ID'si == 0 ya da uygun yeteneğe sahipse)
        <proses her şeyi yapabilir>
    else if (prosesin etkin kullanıcı ID'si == dosyanın kullanıcı ID'si)
        <dosyanın rwx sahiplik hakları dikkate alınır>
    else if (prosesin etkin grup ID'si ya da ek gruplarından birinin ID'si == dosyanın grup ID'si)
        <dosyanın rwx grup hakları dikkate alınır>
    else
        <dosyanın rwx diğer (other) hakları dikkate alınır>

Buradan görüldüğü gibi grup kontrolü yapılırken yalnızca prosesin etkin grup ID'si değil aynı zamanda ek
grup ID'leri de eşdeğer düzeyde etkili olmaktadır. Yani örneğin prosesimizin etkin grup ID'si ``study``
olsun. Ancak ek gruplarının ID'leri de ``work`` ve ``test`` olsun. Erişmeye çalıştığımız dosyanın grup
ID'si ``test`` ise her ne kadar bizim etkin grup ID'miz ``test`` değilse de ek gruplarımızdan biri
``test`` olduğu için biz bu dosya ile aynı grupta olan bir proses olarak kabul ediliriz.

Peki ek gruplara neden gereksinim duyulmuştur? Bunun gerekçesini şöyle bir örnekle açıklayabiliriz:
``ali``, ``veli`` ve ``selami`` ortak bir proje üzerinde çalışıyor olsunlar. Bu kişilerin proje
dosyalarına ortak biçimde erişebilmesi için aynı grupta olmaları gerekir. Ancak ``ali`` kullanıcısının
``ayse`` ve ``fatma`` ile başka bir proje üzerinde de çalıştığını varsayalım. Eğer bu ikinci proje
grubundaki üyeler de aynı grup ID'ye sahip olurlarsa bu durumda bu ikinci proje grubuna dahil olmayanlar
da ikinci proje grubunun dosyalarına erişebilecektir. Yani bir kullanıcı birden fazla proje üzerinde
çalışacaksa birden fazla gruba üye olabilmelidir.

Daha önceden de belirttiğimiz gibi bir prosesin gerçek kullanıcı ve grup ID'leri login prosesi
tarafından ``/etc/passwd`` dosyasına başvurularak belirlenmektedir. İşte ``login`` programı prosesin ek
gruplarını da ``/etc/group`` dosyasına bakarak belirlemektedir. Anımsanacağı gibi ``/etc/group``
dosyasındaki her satır bir gruba ilişkin bilgileri barındırıyordu. Satırın sonundaki son eleman ise o
gruba ek grup olarak dahil olan kullanıcıları belirtmekteydi. Örneğin:

.. code-block:: text

    ...
    sys:x:3:
    adm:x:4:syslog,kaan
    tty:x:5:syslog
    disk:x:6:
    ...

Burada ``syslog`` kullanıcısı hem ``adm`` grubuna hem de ``tty`` grubuna ek grup olarak üyedir. ``kaan``
kullanıcısı da ek grup olarak ``adm`` grubuna üyedir. Tabii login prosesinin bir kullanıcının ek
gruplarını belirleyebilmesi için ``/etc/group`` dosyasındaki tüm satırları gözden geçirmesi gerekir.

Bu durumda ``login`` programı şöyle yazılmıştır:

1. Kullanıcı adı ve parola iste (bunu terminal programları da yapabilmektedir).
2. ``/etc/passwd`` ve ``/etc/shadow`` dosyalarına başvurarak parola doğrulamasını yap.
3. Kullanıcının ek gruplarını tespit etmek için ``/etc/group`` dosyasını dolaş ve kullanıcının ek
   gruplarını elde et.
4. ``setuid`` fonksiyonuyla prosesin gerçek, etkin ve saklı kullanıcı ID'lerini ``/etc/passwd`` dosyasında
   belirtilen kullanıcı ID'si olarak set et.
5. ``setgid`` fonksiyonuyla prosesin gerçek, etkin ve saklı grup ID'sini ``/etc/passwd`` dosyasında
   belirtilen grup ID'si olarak set et ve kullanıcının ek gruplarını da aşağıda açıklayacağımız gibi set et.
6. ``chdir`` fonksiyonuyla prosesin çalışma dizinini ``/etc/passwd`` dosyasında belirtildiği gibi set et
7. exec ile ``/etc/passwd`` dosyasında belirtilen programı (tipik olarak kabuk programı) çalıştır.

Genel olarak ``login`` programı kabuk programını ``fork``/``exec`` ile değil yalnızca ``exec`` ile çalıştırmaktadır.
Biz kabuktan çıkınca normalde yeniden ``login`` programı çalıştırılmaktadır.

Tabii prosesin ek grup ID'leri de diğer ID'leri gibi olduğu gibi prosesin kontrol bloğunda saklanmaktadır.

getgroups Fonksiyonu
--------------------

O anda çalışmakta olan prosesin ek grup ID'leri ``getgroups`` isimli POSIX fonksiyonu ile elde
edilebilmektedir. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int getgroups(int gidsetsize, gid_t grouplist[]);

Fonksiyonun ikinci parametresi ek grup ID'lerinin yerleştirileceği ``gid_t`` türünden dizinin başlangıç
adresini, birinci parametresi ise bu dizinin uzunluğunu belirtmektedir. Fonksiyon başarı durumunda diziye
yerleştirdiği eleman sayısına geri döner. Eğer dizi yeterince büyük değilse fonksiyon başarısız olur ve
``-1`` değerine geri döner. getgroups fonksiyonu ``/etc/group`` dosyasına başvurmamaktadır, prosesin ek grup 
ID'lerini doğrudan proses kontrol bloğundan elde etmektedir. 

Peki biz bu fonksiyona geçireceğimiz dizinin uzunluğunu nasıl belirleyebiliriz? Daha ileride göreceğimiz
gibi ``<limits.h>`` dosyası içerisinde ``NGROUPS_MAX`` isimli bir sembolik sabit vardır. Bu sembolik sabit ilgili
sistemdeki proseslerin sahip olabileceği maksimum grup sayısını belirtmektedir. Ancak maalesef bu
sembolik sabit *Runtime Increasable Values* grubundadır. Yani bu değer sistem açıldıktan sonra
artırılmış olabilir. Gerçek değer ise ``sysconf`` fonksiyonuyla elde edilmektedir. Ancak ``sysconf``
fonksiyonunun çağrılması zahmetlidir. ``getgroups`` fonksiyonunun özel bir durum olarak birinci
parametresi ``0`` geçilirse zaten fonksiyon bize o kullanıcının ek gruplarının sayısını vermektedir. Biz de
bu sayı kadar alanı ``malloc`` ile tahsis edebiliriz. Fonksiyonun birinci parametresine ``0`` girildiğinde
ikinci parametresi zaten dikkate alınmaz, ``NULL`` adres geçilebilir.

``getgroups`` fonksiyonunun bize verdiği grup listesi içerisinde prosesin etkin grup ID'sinin de olup
olmayacağı sistemden sisteme değişebilmektedir. Linux sistemleri her zaman prosesin etkin grup ID'sini
de bu diziye yerleştirmektedir. ``NGROUPS_MAX`` sembolik sabitine bu değer dahil değildir. Yani buradan
hareketle dizi uzunluğu belirlenecekse bu değerden bir fazla değer kadar alan ``malloc`` fonksiyonuyla
tahsis edilmelidir. Ancak birinci parametre ``0`` geçilirse zaten geri döndürülen değere bu değer dahildir.
Özetle fonksiyonun çağrılmasında şu yöntemler izlenebilir:

1. Dizi ``NGROUPS_MAX + 1`` uzunlukta açılıp çağrı yapılır. Fonksiyon başarısız olursa ``sysconf`` fonksiyonu ile
   gerçek değer elde edilebilir. Ya da doğrudan ``sysconf`` fonksiyonu kullanılabilir.

2. Dizi uzunluğu büyük bir değer olarak girilebilir. Tabii fonksiyonun başarısı kontrol edilmelidir.

3. Fonksiyonun birinci parametresine ``0`` geçilerek fonksiyon çağrılabilir. Elde edilen değerden hareketle
   ``malloc`` fonksiyonu ile tam istenen uzunlukta alan tahsis edilebilir.

Aşağıdaki örnekte prosesin ek grup ID'leri alınıp ekrana (``stdout`` dosyasına) yazdırılmıştır. Linux
sistemlerinde bu listeye prosesin etkin grup ID'sinin de dahil edildiğine dikkat ediniz.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <stdint.h>
    #include <errno.h>
    #include <unistd.h>
    #include <grp.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        gid_t *gids;
        struct group *grp;
        int ngroups;

        if ((ngroups = getgroups(0, NULL)) == -1)
            exit_sys("getgroups");

        if ((gids = (gid_t *)malloc(sizeof(gid_t) * ngroups)) == NULL) {
            fprintf(stderr, "cannot allocate memory!..\n");
            exit(EXIT_FAILURE);
        }

        if ((getgroups(ngroups, gids)) == -1)
            exit_sys("getgroups");

        for (int i = 0; i < ngroups; ++i) {
            if (i != 0)
                printf(", ");
            errno = 0;
            if ((grp = getgrgid(gids[i])) == NULL) {
                if (errno == 0)
                    printf("?(%jd)", (intmax_t)gids[i]);
                else
                    exit_sys("getgrgid");
            }
            else
                printf("%s(%jd)", grp->gr_name, (intmax_t)gids[i]);
        }
        putchar('\n');

        free(gids);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

setgroups Fonksiyonu
--------------------

POSIX standartlarında prosesin ek gruplarını set eden bir fonksiyon bulundurulmamıştır. Ancak işletim
sistemlerinde bunu yapan bir sistem fonksiyonu olmak zorundadır. Linux sistemlerinde ve diğer pek çok 
UNIX türevi sistemde ``setgroups`` isimli fonksiyon ilgili sistem fonksiyonunu çağırarak bu işi yapmaktadır. 
Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <grp.h>

    int setgroups(size_t size, const gid_t *list);

Fonksiyonun birinci parametresi grup ID'lerinin bulunduğu dizinin uzunluğunu, ikinci parametresi bu
dizinin başlangıç adresini almaktadır. Bu listeye prosesin etkin grup ID'si dahil edilmemelidir.
Fonksiyon başarı durumunda ``0`` değerine, başarısızlık durumunda ``-1`` değerine geri döner. Tabii fonksiyonu
herkes çağıramaz. Fonksiyonun başarılı olması için prosesin uygun önceliğe (appropriate privileges)
sahip olması gerekmektedir. (Yani prosesin etkin kullanıcı ID'si ``0`` (``root``) olması ya da Linux
sistemlerinde ``CAP_SETGID`` yeteneğine sahip olması gerekir.)

Yukarıda da belirttiğimiz gibi aslında bu fonksiyon sisteme giriş sırasında ``/etc/group`` dosyası
taranarak ``login`` programı tarafından çağrılmaktadır.

Aşağıdaki programda önce prosesin ek grupları elde edilmiş sonra ek gruplara bir grup dahil edilip
``setgroups`` fonksiyonuyla prosesin ek grupları set edilmiştir. ``setgroups`` işleminde prosesin etkin
kullanıcı ID'si ``0`` olduğu için fonksiyon başarılı olacaktır. En sonunda programda ``setuid`` fonksiyonu
ile prosesin etkin kullanıcı ve saklı kullanıcı ID'si gerçek kullanıcı ID'si haline getirilmiştir. Örneğimizde derleme 
işleminden sonra çalıştırılabilir dosyanın set-user-ID bayrağı set edilmiş ve sahibi ``root`` yapılmıştır. 
Böylece programı çalıştırırken ``sudo`` yapmaya gerek kalmayacaktır. Bunun için program önce aşağıdaki gibi 
derlenmelidir:

.. code-block:: console

    $ gcc -Wall -o sample sample.c

Sonra program dosyasının sahibi ``root`` olarak değiştirilmelidir:

.. code-block:: console

    $ sudo chown root sample
    $ ls -l sample
    -rwxr-xr-x 1 root study 17400 Şub 26 12:18 sample

Sonra da program dosyasının set-user-ID bayrağı set edilmelidir:

.. code-block:: console

    $ sudo chmod u+s sample
    $ ls -l sample
    -rwsr-xr-x 1 root study 17400 Şub 26 12:18 sample

Burada önce dosyanın sahibinin değiştirildiğine, daha sonra set-user-ID bayrağının set edildiğine dikkat
ediniz. Çünkü ``chown`` POSIX fonksiyonu dosyanın set-user-ID ve set-group-ID bayraklarını reset
etmektedir. ``chown`` fonksiyonunun dokümantasyonunu dikkatlice bir kez daha okuyunuz.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <stdint.h>
    #include <errno.h>
    #include <unistd.h>
    #include <grp.h>

    void disp_groups(void);
    void exit_sys(const char *msg);

    int main(void)
    {
        gid_t *gids;
        int ngroups;

        disp_groups();

        if ((ngroups = getgroups(0, NULL)) == -1)
            exit_sys("getgroups");

        if ((gids = (gid_t *)malloc(sizeof(gid_t) * (ngroups + 1))) == NULL) {
            fprintf(stderr, "cannot allocate memory!..\n");
            exit(EXIT_FAILURE);
        }

        if ((getgroups(ngroups, gids)) == -1)
            exit_sys("getgroups");

        gids[ngroups] = 0;

        if (setgroups(ngroups + 1, gids) == -1)
            exit_sys("setgroups");

        free(gids);

        disp_groups();

        return 0;
    }

    void disp_groups(void)
    {
        gid_t *gids;
        struct group *grp;
        int ngroups;

        if ((ngroups = getgroups(0, NULL)) == -1)
            exit_sys("getgroups");

        if ((gids = (gid_t *)malloc(sizeof(gid_t) * ngroups)) == NULL) {
            fprintf(stderr, "cannot allocate memory!..\n");
            exit(EXIT_FAILURE);
        }

        if ((getgroups(ngroups, gids)) == -1)
            exit_sys("getgroups");

        for (int i = 0; i < ngroups; ++i) {
            if (i != 0)
                printf(", ");
            errno = 0;
            if ((grp = getgrgid(gids[i])) == NULL) {
                if (errno == 0)
                    printf("?(%jd)", (intmax_t)gids[i]);
                else
                    exit_sys("getgrgid");
            }
            else
                printf("%s(%jd)", grp->gr_name, (intmax_t)gids[i]);
        }
        putchar('\n');

        free(gids);
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

initgroups Fonksiyonu
---------------------

Aslında bir POSIX fonksiyonu olmasa da Linux, BSD ve pek çok UNIX türevi sistemlerde ``initgroups`` isimli
bir fonksiyon da bulunmaktadır. Bu fonksiyon ``/etc/group`` dosyasını dolaşarak belli bir kullanıcının
bütün ek grup bilgilerini elde eder ve ``setgroups`` fonksiyonu ile bunları set eder. Yani aslında 
``login`` programı bu ``initgroups`` fonksiyonunu çağırmaktadır. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <grp.h>

    int initgroups(const char *user, gid_t group);

Fonksiyon birinci parametresiyle kullanıcının ismini alır. ``/etc/group`` dosyasına başvurarak
kullanıcının ek grup ID'lerini elde eder. Sonra da ``setgroups`` fonksiyonunu uygulayarak prosesin ek
gruplarını set eder. Tabii fonksiyonun bu işlemi yapabilmesi için yine uygun önceliğe sahip olması
gerekmektedir. Fonksiyon başarı durumunda 0 değerine, başarısızlık durumunda -1 değerine döner ve
``errno`` değişkeni uygun biçimde set edilir. Fonksiyonun ikinci parametresi ``/etc/group`` dosyasından
elde edilen ek gruplara dahil edilecek ekstra bir grubu belirtmektedir. Tipik olarak ``login`` prosesi
prosesin etkin grup ID'sini bu listeye eklemek için fonksiyonun ikinci parametresini prosesin etkin grup
ID'si ile çağırmaktadır.


getgrouplist Fonksiyonu
-----------------------

*glibc* kütüphanesindeki ``getgrouplist`` fonksiyonu da açıklanacak.

id, groups, adduser, usermod ve deluser Kabuk Komutları
-------------------------------------------------------

Bir kullanıcının (yani kullanıcı ID'sinin) ilişkin olduğu tüm grupları kabuk üzerinde ``id`` komutu ya da ``groups`` 
komutu ile görüntülenebilir. Örneğin:

.. code-block:: console

    $ id
    uid=1000(kaan) gid=1000(study) gruplar=1000(study),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev),115(lpadmin),136(sambashare)
    $ groups
    study adm cdrom sudo dip plugdev lpadmin sambashare
    $ id ali
    uid=1001(ali) gid=1000(study) gruplar=1000(study)
    $ groups ali
    ali : study

Var olan bir kullanıcıya ``/etc/group`` dosyasında bir ek grup atamak için ``adduser`` komutu
kullanılabilir. Aslında bu ``adduser`` komutu daha önce de gördüğümüz gibi yeni bir kullanıcı yaratmak
için kullanılmaktadır. Ancak bu komuta biz var olan bir kullanıcı ve grup ismini verirsek bu komut
kullanıcıyı ek grup olarak da ilgili gruba eklemektedir. Örneğin:

.. code-block:: console

    $ sudo adduser kaan work
    [sudo] kaan için parola:
    "kaan" kullanıcısı "work" grubuna ekleniyor ...
    kaan kullanıcısı work grubuna ekleniyor
    Tamamlandı.

Biz ``adduser`` komutunu kabuk üzerinden uyguladığımızda bu komut o andaki kabuk prosesinin proses kontrol
bloğundaki ek gruplarda bir değişiklik yapmaz. Yalnızca ``/etc/group`` dosyasında güncelleme
yapmaktadır. ``adduser`` komutunda girilen grup var olmasa bile komut tarafından ``/etc/group``
dosyasında oluşturulmaktadır. Burada girilen yeni ek grubun işleme sokulabilmesi için ``login``
prosesinin çalıştırılması gerekir. Biz masaüstünden terminal açtığımızda ``login`` programı devreye
girmemektedir. (Buna *interactive non-login shell* dendiğini anımsayınız.)

Aynı işlemler ``usermod`` kabuk komutuyla da yapılabilmektedir:

.. code-block:: console

    $ sudo usermod -G work kaan

Bir kullanıcının ``/etc/group`` dosyasındaki ek gruptan silinmesi de ``deluser`` komutuyla
yapılabilmektedir. Örneğin:

.. code-block:: console

    $ sudo deluser kaan work
    `kaan' kullanıcısı `work' grubundan siliniyor ...
    Tamamlandı.

Burada ``kaan`` kullanıcısı ``work`` isimli gruptan çıkarılmıştır. Tabii bu komut da yalnızca
``/etc/group`` dosyası üzerinde güncelleme yapmaktadır.

.. _zaman-paylasimli-calisma-ve-bloke-olma:

Ek Grupların fork İşlemi Sırasnda Alt Prosese Aktarılması
---------------------------------------------------------

Prosesin ek grupları proses kontrol bloğu içerisinde (Linux'taki ``task_struct``) saklanmaktadır.
``fork`` işlemi sırasında üst prosesin proses kontrol bloğu alt prosese kopyalanırken üst prosesin tüm
ek grupları da alt prosese aktarılmaktadır. Yani örneğin biz kabuk üzerinden bir program
çalıştırdığımızda kabuk prosesinin ek grupları bizim prosesimize kabuğun uyguladığı ``fork`` sonucunda
aktarılmış olacaktır. Tabii kabuk prosesinin de ek grupları aslında ``login`` programı tarafından
``/etc/group`` dosyasına başvurularak ``setgroups`` fonksiyonuyla set edilmektedir.

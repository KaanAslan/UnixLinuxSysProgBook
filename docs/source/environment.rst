==============================
Proseslerin Çevre Değişkenleri
==============================

Modern işletim sistemlerinin büyük çoğunluğunda, prosese özgü, ismine *çevre değişkenleri (environment
variables)* denilen bir veri yapısı bulundurulmaktadır. Çevre değişkenleri anahtar-değer çiftlerini
tutan, anahtar verildiğinde onun değerini bize veren *sözlük (dictionary)* tarzı bir veri yapısı
organizasyonudur. Tabii sözlük tarzı veri yapıları pek çok nesne yönelimli programlama dilinin standart
kütüphanesinde *map*, *set*, *dictionary*, *hashtable* gibi isimlerle de bulunmaktadır. Ancak çevre
değişkenleri, bir sözlük veri yapısının basit bir biçimde işletim sistemi tarafından aşağı seviyeli
gerçekleştirimidir.

Çevre değişkenleri bağlamında anahtar-değer çiftlerinin anahtarlarına *çevre değişkeni (environment
variable)* denilmektedir. O anahtara karşı gelen değere de *o çevre değişkeninin değeri* denir. Çevre
değişkenlerinin anahtarları da değerleri de birer yazı biçimindedir. Örneğin anahtar *ankara* yazısı
olabilir, onun değeri de *06* yazısı olabilir. Anahtar *eskisehir* yazısı olabilir, onun değeri de *26*
yazısı olabilir.

Çevre değişkenleri ve değerleri pek çok işletim sisteminde prosesin bellek alanı içerisinde
tutulmaktadır. Örneğin Windows sistemleri, UNIX/Linux sistemleri tipik olarak çevre değişkenlerini proses
bellek alanı içerisinde özel bir alanda tutmaktadır.

UNIX/Linux sistemlerinde prosesin çevre değişkenlerinin (yani anahtarların) büyük harf-küçük harf
duyarlılığı vardır. Ancak Windows sistemlerinde çevre değişkenlerinin büyük harf-küçük harf duyarlılığı
yoktur. Genel olarak çevre değişkenleri (yani anahtarlar) boşluk karakterleri içermemektedir.

Çevre değişkenleri yukarıda da belirttiğimiz gibi prosese özgüdür. ``fork`` işlemi sırasında alt prosese
aktarılmaktadır. Örneğin biz kabuk üzerinden bir program çalıştırdığımızda kabuk prosesinin çevre
değişkenleri bizim çalıştırdığımız programa ilişkin prosese aktarılmaktadır. Bir proses başka bir
prosesin çevre değişkenlerine herhangi bir biçimde müdahale edememektedir.

Peki çevre değişkenlerine neden gereksinim duyulmaktadır? İşte bazı POSIX fonksiyonları ve sistem
fonksiyonları prosesin belli çevre değişkenlerine başvurabilmektedir. Çevre değişkenleri programcılar
tarafından da çeşitli amaçlarla kullanılabilmektedir.

Çevre değişkenleri ile ilgili programcının şu işlemleri yapabilmesi gerekmektedir:

- Bir çevre değişkeni (yani anahtar) verildiğinde onun değerini elde etmek.
- Prosesin çevre değişken listesine yeni bir anahtar-değer çifti eklemek.
- Prosesin tüm çevre değişken listesini elde etmek.

getenv Fonksiyonu
=================

Bir çevre değişkeni (yani anahtar) verildiğinde onun değerini elde etmek için ``getenv`` isimli standart
C fonksiyonu kullanılmaktadır. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <stdlib.h>

    char *getenv(const char *name);

Fonksiyon parametre olarak çevre değişkeninin ismini (yani anahtarı) alır, geri dönüş değeri olarak onun
değerinin bulunduğu bellek adresini verir. Fonksiyonun geri döndürdüğü adres prosesin adres alanı
içerisindeki statik düzeyde tahsis edilmiş bir alanın adresidir. Tabii bu adresteki yazının sonunda null
karakter bulunmaktadır. Fonksiyon eğer ilgili çevre değişkeni yoksa ``NULL`` adrese geri dönmektedir.
Fonksiyonun geri dönüş değeri ``const`` olmayan bir gösterici olsa da programcı geri döndürülen bu
adresteki yazıyı değiştirmeye çalışmamalıdır. C standartlarında bu değiştirme durumu işletim sisteminin
isteğine bırakılmış olsa da UNIX/Linux sistemlerinde bu durum tanımsız davranışa yol açmaktadır.
``getenv`` fonksiyonu başarısızlık durumunda ``errno`` değişkenini herhangi bir değerle set etmemektedir.
Örneğin:

.. code-block:: c

    char *value;
    /* ... */

    if ((value = getenv("PATH")) == NULL) {
        fprintf(stderr, "cannot find environment variable!..\n");
        exit(EXIT_FAILURE);
    }
    puts(value);

Burada ``PATH`` isimli çevre değişkeninin değeri elde edilip yazdırılmıştır.

Aşağıdaki örnekte komut satırından alınan çevre değişkeninin değeri ``stdout`` dosyasına yazdırılmıştır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>

    int main(int argc, char *argv[])
    {
        char *value;

        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }

        if ((value = getenv(argv[1])) == NULL) {
            fprintf(stderr, "environment variable not found: %s\n", argv[1]);
            exit(EXIT_FAILURE);
        }

        puts(value);

        return 0;
    }

Çevre Değişkenlerinin fork ile Aktarımı
=======================================

Yukarıda da belirttiğimiz gibi prosesin çevre değişkenleri, ``fork`` işlemi sırasında üst prosesten alt
prosese aktarılmaktadır. Zaten çevre değişkenleri prosesin bellek alanında saklandığından ve ``fork``
işlemi de prosesin bellek alanının kopyasını oluşturduğundan bu aktarım ``fork`` işleminde doğal olarak
yapılmaktadır. Örneğin biz kabuk üzerinden bir program çalıştırdığımızda kabuğun çevre değişkenleri bizim
programımıza aktarılacaktır.

Kabuktaki Çevre Değişkenleri (env, cd, PWD)
===========================================

Kabuğun çevre değişken listesi ``env`` kabuk komutuyla her satırda *anahtar=değer* biçiminde
görüntülenebilmektedir. Peki kabuk programındaki çevre değişkenleri nasıl oluşturulmuştur? İşte
prosesler birbirlerini yaratırken kabuk prosesine gelene kadar bazı prosesler çevre değişkenlerine
eklemeler yapmaktadır. Örneğin kabuk programını çalıştıran login programı ``HOME``, ``USER``, ``SHELL``
gibi çevre değişkenlerini prosesin çevre değişken listesine eklemektedir. Benzer biçimde kabuk da pek çok
çevre değişkenini çevre değişken listesine eklemiş durumdadır. Yani biz programımızı kabuk üzerinden
çalıştırırken kümülatif olarak çeşitli prosesler çevre değişken listesine çeşitli çevre değişkenlerini
zaten eklemiş durumdadır. Örneğin biz kabuk üzerinde ``cd`` komutunu kullandığımızda kabuk ``PWD`` isimli
çevre değişkeninin değerini o anda geçilen dizinin yol ifadesini belirtecek biçimde değiştirmektedir.
(Bunu ``chdir`` POSIX fonksiyonu yapmaz, zaten yapamaz. Kabuktaki ``cd`` komutu bunu yapmaktadır.)

Kabukta echo ve Çevre Değişkeni Genişletmesi ($VAR)
===================================================

Bilindiği gibi ``echo`` isimli kabuk komutu yanındaki yazıyı ``stdout`` dosyasına yazdırmaktadır. Örneğin:

.. code-block:: console

    $ echo merhaba nasılsın?
    merhaba nasılsın?

``echo`` komutunda yazdırılacak yazı iki tırnak ya da tek tırnağa da alınabilir. Örneğin:

.. code-block:: console

    $ echo "merhaba nasılsın?"
    merhaba nasılsın?

Tabii ``echo`` komutu aslında ``/bin`` dizininde bir program dosyası biçiminde bulunmaktadır. Dolayısıyla
bu program ``argv`` ile komut satırı argümanlarını alıp birleştirerek aralara SPACE karakteri ekleyip
yazdırmaktadır. Örneğin:

.. code-block:: console

    $ echo merhaba    nasılsın?
    merhaba nasılsın?

Burada *merhaba* ile *nasılsın?* arasındaki birden fazla boşluk karakterinin kaybolduğuna dikkat ediniz.
Kabuk üzerinden çift tırnak ve tek tırnak (aralarında küçük farklılıklar vardır) tek bir komut satırı
argümanı oluşturmaktadır. Örneğin:

.. code-block:: console

    $ echo "merhaba    nasılsın?"
    merhaba    nasılsın?

Kabuk üzerinde bir çevre değişkenini başına ``$`` getirerek yazarsak kabuk sanki o yazı yerine onun
değerine ilişkin yazıyı oraya yazmışız gibi davranmaktadır. Örneğin biz kabuk üzerinde ``$PATH`` yazarsak
kabuk bu ``$PATH`` yazısını kaldırıp onun yerine onun değerini oraya yerleştirecektir. (Aynı işlem Windows
sistemlerinde ``%NAME%`` ile yapılmaktadır.) O halde biz kabuğun bir çevre değişkeninin değerini şöyle
yazdırabiliriz:

.. code-block:: console

    $ echo $PATH
    /home/kaan/.local/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin

Eğer ``$`` karakterinin yanındaki çevre değişkeni mevcut değilse bu durumda kabuk onun yerine boş bir
yazı (boş string) yerleştirmektedir. Örneğin:

.. code-block:: console

    $ echo -$XXX-
    --


=================================================================
Çevre Değişkenlerini Değiştirme: setenv, putenv, environ ve Kabuk
=================================================================

setenv Fonksiyonu
=================

Prosesin çevre değişken listesine yeni bir anahtar-değer çifti eklemek için ``setenv`` ve ``putenv``
isimli POSIX fonksiyonları kullanılmaktadır. Bu fonksiyonlar standart C fonksiyonları değildir. C'de
prosesin çevre değişken listesine ekleme yapan standart bir fonksiyon yoktur.

``setenv`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <stdlib.h>

    int setenv(const char *name, const char *value, int overwrite);

Fonksiyonun birinci parametresi çevre değişkeninin ismini, ikinci parametresi onun değerini alır. Üçüncü
parametre eğer o çevre değişkeni zaten varsa onun değerinin değiştirilip değiştirilmeyeceğini belirtir. Bu
parametre sıfır dışı bir değer olarak geçilirse çevre değişkeninin değeri değiştirilir. Sıfır geçilirse
değiştirilmez ve fonksiyon yine başarıyla geri döner. Fonksiyon başarı durumunda 0 değerine, başarısızlık
durumunda -1 değerine geri dönmektedir. Başarısızlık durumunda ``errno`` değeri uygun biçimde set
edilmektedir.

setenv Kullanım Örneği
----------------------

Aşağıdaki örnekte komut satırı argümanı ile verilen çevre değişkenleri ``setenv`` fonksiyonu ile prosesin
çevre değişken listesine eklenmiş ve sonra ``getenv`` fonksiyonu ile onların değerleri elde edilmiştir.
Girişin aşağıdaki gibi yapılması gerekir:

.. code-block:: console

    $ ./setenv ali=100 veli=200 selami=300

Program ``=`` karakterini ``strchr`` fonksiyonu ile aramış, eğer onu bulursa ``=`` karakteri yerine
``\0`` karakterini yerleştirmiştir.

.. code-block:: c

    /* setenv.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    int main(int argc, char *argv[])
    {
        char *str;
        char *value;

        if (argc == 1) {
            fprintf(stderr, "too few arguments!..\n");
            exit(EXIT_FAILURE);
        }

        for (int i = 1; i < argc; ++i) {
            if ((str = strchr(argv[i], '=')) == NULL) {
                fprintf(stderr, "invalid argument: %s\n", argv[i]);
                exit(EXIT_FAILURE);
            }
            *str = '\0';
            if (setenv(argv[i], str + 1, 1) == -1)
                perror("setenv");
        }

        for (int i = 1; i < argc; ++i) {
            if ((value = getenv(argv[i])) == NULL) {
                fprintf(stderr, "environment variable not found: %s\n", argv[i]);
                continue;
            }
            printf("%s ===> %s\n", argv[i], value);
        }

        return 0;
    }

putenv Fonksiyonu
=================

``putenv`` fonksiyonu da yine prosesin çevre değişken listesine ekleme yapmak için kullanılmaktadır.
Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <stdlib.h>

    int putenv(char *str);

Fonksiyon parametre olarak ``anahtar=değer`` biçiminde bir yazı almaktadır. Fonksiyon ilgili çevre
değişkeni zaten varsa her zaman onun değerini değiştirmektedir. Eğer yazıda ``=`` karakteri
kullanılmazsa, değeri boş olan (yani elde edildiğinde yalnızca null karakter veren) bir çevre değişkeni
oluşturulmaktadır. Fonksiyon yine başarı durumunda 0 değerine, başarısızlık durumunda -1 değerine geri
döner ve ``errno`` değişkeni uygun biçimde set edilir. ``putenv`` fonksiyonunda verilen adres doğrudan
prosesin çevre değişken listesinde kullanmaktadır. Verilen adresteki bilginin program çalıştığı sürece
kalıcı olmasına dikkat ediniz. Örneğin:

.. code-block:: c

    if (putenv("city=istanbul") == -1)
        exit_sys("putenv");

UNIX türevi sistemlerde prosesin çevre değişken listesi, izleyen paragraflarda da görüleceği gibi,
prosesin sanal bellek alanında ``environ`` isimli bir gösterici dizisinde tutulmaktadır. ``environ``
gösterici dizisini daha önce görmüştük. İşte ``putenv`` fonksiyonu bu gösterici dizisindeki yeni bir
elemana parametresiyle verilen ``anahtar=değer`` yazısının adresini yerleştirmektedir.

putenv Kullanım Örneği (Komut Satırı Argümanları)
-------------------------------------------------

Aşağıdaki örnekte yine program aşağıdakine benzer çalıştırılmalıdır:

.. code-block:: console

    $ ./putenv ali=100 veli=200

.. code-block:: c

    /* putenv.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    int main(int argc, char *argv[])
    {
        char *value;
        char *str;
        char env[4096];
        ptrdiff_t len;

        if (argc == 1) {
            fprintf(stderr, "too few arguments!..\n");
            exit(EXIT_FAILURE);
        }

        for (int i = 1; i < argc; ++i)
            if (putenv(argv[i]) == -1)
                perror("putenv");

        for (int i = 1; i < argc; ++i) {
            if ((str = strchr(argv[i], '=')) != NULL) {
                len = str - argv[i];
                memcpy(env, argv[i], len);
                env[len] = '\0';
            }

            if ((value = getenv(env)) == NULL) {
                fprintf(stderr, "environment variable not found: %s\n", argv[i]);
                continue;
            }
            printf("%s ===> %s\n", env, value);
        }

        return 0;
    }

putenv ile Program İçi Değişiklik Örneği
----------------------------------------

Aşağıdaki örnekte ``putenv`` fonksiyonu ile prosesin çevre değişken listesine bir ekleme yapılmıştır.
Sonra buradaki anahtar-değer çifti program içerisinde değiştirilmiştir. Prosesin çevre değişken
listesinin nasıl organize edildiği izleyen paragrafta ele alınmaktadır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        char *value;
        char env[1024] = "city=istanbul";

        if (putenv(env) == -1)
            exit_sys("setenv");

        if ((value = getenv("city")) == NULL) {
            fprintf(stderr, "cannot find environment variable \"city\"!...\n");
            exit(EXIT_FAILURE);
        }

        puts(value);        /* istanbul */

        strcpy(env, "village=urla");

        if ((value = getenv("village")) == NULL) {
            fprintf(stderr, "cannot find environment variable \"village\"!...\n");
            exit(EXIT_FAILURE);
        }

        puts(value);        /* urla */

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Çevre Değişkenlerinin Bellekte Organizasyonu (environ Dizisi)
=============================================================

UNIX/Linux sistemlerinde genel olarak çevre değişkenleri bir gösterici dizisi yoluyla tutulmaktadır. Her
çevre değişkeni aslında ``anahtar=değer\0`` biçiminde bir yazı olarak oluşturulmakta ve bu yazıların
başlangıç adresleri de bir gösterici dizisinde saklanmaktadır. Bu gösterici dizisinin sonunda da ``NULL``
adres bulunmaktadır. Bu gösterici dizisinin başlangıç adresi ``environ`` isimli bir global göstericiyi
gösteren göstericiyle tutulmaktadır. Yani prosesin çevre değişken listesi aşağıdaki gibi bir veri
yapısıyla oluşturulmuştur:

.. code-block:: text

    environ ---->  adres  ---> ali=100\0
                    adres  ---> veli=200\0
                    adres  ---> selami=300\0
                    ...
                    NULL

``environ`` göstericisinin gösterdiği yerdeki gösterici dizisinin sonunda ``NULL`` adres bulundurulduğuna
dikkat ediniz.

Aslında ``putenv`` fonksiyonu bizim ``anahtar=değer`` biçiminde verdiğimiz yazının adresini eğer anahtar
yoksa doğrudan bu gösterici dizisine eklemektedir. Bu gösterici dizisinin hemen aşağısında boş yer olmak
zorunda değildir. Genellikle gerçekleştirimleri yapanlar yeni bir ``environ`` dizisini tahsis edip
eklemeyi oraya yapmaktadır. Örneğin:

.. code-block:: c

    char s[] = "ayse=500";

    putenv(s);

.. code-block:: text

    environ (yeri değişmiş olabilir) ----> adres              ---> ali=100\0
                                            adres              ---> veli=200\0
                                            adres              ---> selami=300\0
                                            ...
                                            s dizisinin adresi ---> ayse=500\0
                                            NULL

Maalesef bu ``environ`` global değişkeninin ``extern`` bildirimi herhangi bir başlık dosyasında
bulundurulmamıştır. Prosesin çevre değişken listesine erişmek isteyen programcıların bu ``extern``
bildirimini kendilerinin yapması gerekir. Örneğin:

.. code-block:: c

    extern char **environ;

O halde prosesin bütün çevre değişkenlerinin listesini almak oldukça kolaydır:

.. code-block:: c

    for (int i = 0; environ[i] != NULL; ++i)
        puts(environ[i]);

Tüm Çevre Değişkenlerini Listeleme (environ ile)
------------------------------------------------

Aşağıdaki örnekte prosesin tüm çevre değişkenlerinin listesi elde edilerek ekrana (``stdout`` dosyasına)
yazdırılmıştır.

.. code-block:: c

    #include <stdio.h>

    extern char **environ;

    int main(void)
    {
        for (int i = 0; environ[i] != NULL; ++i)
            puts(environ[i]);

        return 0;
    }

env Kabuk Komutu
----------------

Kabuk üzerinde ``env`` komutu yukarıdaki programda olduğu gibi kabuğun tüm çevre değişken listesini
ekrana (``stdout`` dosyasına) yazdırmaktadır. Örneğin:

.. code-block:: console

    $ env
    SHELL=/bin/bash
    SESSION_MANAGER=local/kaan-virtual-machine:@/tmp/.ICE-unix/1361,unix/kaan-virtual-machine:/tmp/.ICE-unix/1361
    QT_ACCESSIBILITY=1
    COLORTERM=truecolor
    XDG_CONFIG_DIRS=/etc/xdg/xdg-cinnamon:/etc/xdg
    XDG_SESSION_PATH=/org/freedesktop/DisplayManager/Session0
    GNOME_DESKTOP_SESSION_ID=this-is-deprecated
    GTK_IM_MODULE=ibus
    QT_IM_MODULES=wayland;ibus
    ...

İstediğimiz bir ismin bulunduğu satırları elde etmek için ``grep`` komutu ile boru işlemi yapabiliriz.
Örneğin:

.. code-block:: console

    $ env | grep PWD
    PWD=/home/kaan/Study/UnixLinux-SysProg/09-EnvironmentVariables
    OLDPWD=/home/kaan/Study/UnixLinux-SysProg

getenv Fonksiyonunun Basit Bir Gerçekleştirimi
----------------------------------------------

``getenv`` fonksiyonu aşağıdaki gibi basit bir biçimde gerçekleştirilebilir.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <stddef.h>
    #include <string.h>

    extern char **environ;

    char *mygetenv(const char *env)
    {
        char *str;
        size_t len1, len2;

        if (*env == '\0' || environ == NULL)
            return NULL;

        len1 = strlen(env);
        for (int i = 0; environ[i] != NULL; ++i) {
            if ((str = strchr(environ[i], '=')) != NULL) {
                len2 = (size_t)(str - environ[i]);               /* size_t dönüştürmesi gerekmiyor */
                if (len1 == len2 && strncmp(environ[i], env, len1) == 0)
                    return str + 1;
            }
        }

        return NULL;
    }

    int main(int argc, char *argv[])
    {
        char *value;

        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!..\n");
            exit(EXIT_FAILURE);
        }

        if ((value = mygetenv(argv[1])) == NULL) {
            fprintf(stderr, "environment not found!..\n");
            exit(EXIT_FAILURE);
        }
        puts(value);

        return 0;
    }

fork ile Çevre Değişkenlerinin Aktarımı
---------------------------------------

Prosesin çevre değişkenlerine ilişkin gösterici dizisi ve onların gösterdikleri yerler prosesin bellek
alanı içerisindedir. ``fork`` işlemi sırasında üst prosesin tüm bellek alanının bir kopyası
oluşturulduğuna göre alt prosesin çevre değişken listesi üst prosesinkinin aynısı olacaktır. Ancak
``fork`` işleminden sonra üst proses kendi çevre değişken listesinde bir değişiklik yaparsa artık
yalnızca o değişiklik o prosese özgü hale gelecektir. Çünkü ``fork`` işlemi sırasında bellek alanları
kopyalandıktan sonra artık üst prosesle alt prosesin bellek alanları birbirinden tamamen ayrılmış olur.

unsetenv Fonksiyonu
===================

Bir çevre değişkeni ``unsetenv`` isimli POSIX fonksiyonuyla prosesin çevre değişken listesinden
silinebilmektedir. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <stdlib.h>

    int unsetenv(const char *name);

Fonksiyon çevre değişkenin ismini almaktadır. Başarı durumunda 0 değerine, başarısızlık durumunda -1
değerine geri döner ve ``errno`` uygun biçimde set edilir. Eğer ilgili çevre değişkeni zaten yoksa
fonksiyon bir şey yapmaz, ancak başarılı bir biçimde geri dönmektedir. Örneğin:

.. code-block:: c

    if (unsetenv("city") == -1)
        exit_sys("unsetenv");

``unsetenv`` işlemi sırasında çevre değişken bloğu için (yani ``environ`` gösterici dizisi için)
küçültme amaçlı yeniden tahsisat da yapılabilir.

Aşağıdaki örnekte komut satırından alınan bir çevre değişkeninin önce değeri yazdırılmış, sonra o çevre
değişkeni silinmiş, sonra da o çevre değişkeninin yeniden değeri elde edilmek istenmiştir.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>

    void exit_sys(const char *msg);

    int main(int argc, char *argv[])
    {
        char *value;

        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }

        if ((value = getenv(argv[1])) == NULL) {
            fprintf(stderr, "environment variable not found: %s\n", argv[1]);
            exit(EXIT_FAILURE);
        }

        puts(value);

        if (unsetenv(argv[1]) == -1)
            exit_sys("ensetenv");

        if ((value = getenv(argv[1])) == NULL) {
            fprintf(stderr, "environment variable not found: %s\n", argv[1]);
            exit(EXIT_FAILURE);
        }

        puts(value);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Kabuk Üzerinde Çevre Değişkeni Tanımlama (export, unset)
========================================================

Peki biz programımızı çalıştırdığımızda belli bir çevre değişkeninin zaten var olmasını nasıl
sağlayabiliriz? Çevre değişkenleri ``fork`` işlemi sırasında alt prosese aktarıldığına göre biz eğer
kabuk programının (bash) çevre değişken listesine bir ekleme yaparsak kabuk bizim programımızı
çalıştırırken ``fork`` yapacak ve onun çevre değişkenleri bizim programımıza aktarılacaktır. Peki kabuk
programının çevre değişken listesine nasıl ekleme yapabiliriz? İşte bu işlem şöyle yapılabilmektedir:

.. code-block:: console

    $ city=eskisehir
    $ export city

Kabuk Değişkeni ile Çevre Değişkeni Farkı ve export Komutu
----------------------------------------------------------

Komut satırında ``anahtar=değer`` biçiminde bir yazı yazıp ENTER tuşuna basarsak biz kabuk dili için bir
kabuk değişkeni yaratmış oluruz. Bu kabuk değişkeninin aynı zamanda kabuğun çevre değişkeni yapılması için
``export`` komutu kullanılmaktadır. Tabii bu iki komut tek hamlede de verilebilmektedir:

.. code-block:: console

    $ export city=eskisehir

Çevre değişkenini kabuktan silmek için de ``unset`` komutu kullanılmaktadır. Örneğin:

.. code-block:: console

    $ unset city

Çevre değişkeni bir kere ``export`` edildikten sonra artık onun değerini değiştirirken yeniden ``export``
işlemi yapılmasına gerek olmaz. Örneğin:

.. code-block:: console

    $ export city=eskisehir
    $ echo $city
    eskisehir
    $ city=izmir
    $ env | grep city

$ ile Değişken Genişletme ve Küme Parantezleri
----------------------------------------------

Anımsanacağı gibi bir çevre değişkeninin (aslında genel olarak kabuk değişkeninin) değerini elde etmek
için kabuk üzerinde ismin önüne ``$`` karakteri getirilmektedir. Örneğin:

.. code-block:: console

    $ echo $city

Burada biz ``city`` çevre değişkeninin değerini ekrana yazdırmış olduk. ``$`` karakterinden sonra çevre
değişkeni küme parantezlerine de alınabilir. Örneğin:

.. code-block:: console

    $ echo ${city}

Buradaki küme parantezine bazı durumlarda gereksinim duyulabilmektedir. Örneğin kabuk üzerinde ``city``
çevre değişkenini yukarıdaki gibi yaratmış olalım. Şimdi de bu çevre değişkeninden hareketle bu çevre
değişkeninin değerine yapışık olarak *center* sözcüğünü de eklemek isteyelim:

.. code-block:: console

    $ export OTHER=$citycenter

Bu durumda kabuk sanki bizim ``citycenter`` isimli bir çevre değişkeninin değerini elde etmek
istediğimizi sanacaktır. Bu durumda mecburen küme parantezleri kullanılmalıdır. Örneğin:

.. code-block:: console

    $ export OTHER=${city}center

Yalnızca Çalıştırılan Programa Özgü Çevre Değişkenleri
------------------------------------------------------

Aslında pek çok kabuk programında hiç kabuk programının çevre değişkenlerini set etmeden, doğrudan
çalıştırılacak program için çevre değişkenleri belirlenebilmektedir. Bunun için önce ``değişken=değer``
çiftleri aralarına boşluk karakterleri olacak biçimde yazılarak program dosyası belirtilir. Örneğin:

.. code-block:: console

    $ XX=10 YY=20 ./sample

Burada ``XX`` ve ``YY`` kabuğun çevre değişken listesine eklenmemektedir. Doğrudan ``sample`` prosesinin
çevre değişkeni yapılmaktadır. Kabuk bu durumda ``fork`` işleminden sonra alt proseste bu çevre
değişkenlerini ekleyip ``exec`` yapmaktadır.

Kabukların Startup (Başlangıç) Dosyaları
========================================

Kabuk üzerinde yukarıdaki gibi çevre değişkeni oluşturduğumuzda bunun kalıcılığı olmaz. Yani bu çevre
değişkeni o kabuk programının (o kabuk prosesinin) çevre değişkeni olur. Biz başka terminal açtığımızda
orada başka bir kabuk prosesi çalışacağı için bu çevre değişkeni orada bulunmayacaktır. Peki kabuk
üzerindeki çevre değişkenlerinin kalıcılığını nasıl sağlayabiliriz? İşte bunu sağlamak için kabukların
*startup* dosyaları kullanılmaktadır.

Kabuk Çalıştırma Biçimleri (login / non-login / non-interactive)
----------------------------------------------------------------

Kabukların startup dosyaları kabuğun nasıl çalıştırıldığına bağlı olarak değişmektedir. Kabuk programları
üç biçimde çalıştırılabilmektedir:

1. Interactive login shell
2. Interactive non-login shell
3. Non-interactive shell

*Interactive shell* demek *komut satırına düşen kullanıcının komut vererek çalıştırdığı shell* demektir.
*login shell* demek bize *user name* ve *password* soran shell demektir. *Non-interactive shell* demek
ise tek bir komutu çalıştırıp işlemini sonlandıran shell demektir. Değişik kabuk programlarının startup
dosyaları farklıdır. Biz burada ``bash`` kabuğu üzerinde duracağız. ``bash`` kabuğunun *user manual*
dokümanındaki ilgili bölüm aşağıdaki bağlantıdan incelenebilir:

``https://www.gnu.org/software/bash/manual/html_node/Bash-Startup-Files.html``

bash Startup Dosyaları
----------------------

Eğer ``bash`` *interactive login shell* biçiminde çalıştırılmışsa shell önce ``/etc/profile`` dosyasını
çalıştırır, sonra sırasıyla aşağıdaki dosyalardan hangisini ilk bulursa yalnız onun içerisindeki
komutları çalıştırır:

.. code-block:: text

    ~/.bash_profile
    ~/.bash_login
    ~/.profile

Eğer ``bash`` *interactive non-login shell* olarak çalıştırılırsa (örneğin masaüstünden) bu durumda
``bash`` ``~/.bashrc`` dosyasındaki komutları çalıştırmaktadır. Yani örneğin biz ``~/.bashrc`` dosyasına
``export`` ile çevre değişkeni eklersek masaüstünden terminali açtığımızda o çevre değişkeni kabuk
üzerinde ekli olarak görünecektir. Tabii programcı hem *interactive login shell* hem de *interactive
non-login shell* için aynı komutların çalıştırılmasını isteyebilir. Bunu sağlamanın pratik bir yolu
komutları ``~/.bashrc`` dosyasına yazıp ``~/.bash_profile`` içerisinden bu dosyanın çalıştırılmasını
sağlamaktır. Bu işlem şöyle yapılabilir:

.. code-block:: bash

    if [ -f ~/.bashrc ]; then . ~/.bashrc; fi

Eğer ``bash`` interactive olmayan bir biçimde (``-c`` seçeneği ile) çalıştırılırsa bu durumda
``BASH_ENV`` isimli bir çevre değişkenini araştırır. Eğer bulursa onun değerinin belirttiği script
dosyasını çalıştırır.

Çevre Değişkenlerinin Kullanım Amaçları
=======================================

Peki çevre değişkenlerine neden gereksinim duyulmaktadır? Çevre değişkenleri birtakım aşağı seviyeli
işlemlerin parametrik hale getirilmesi için kullanılabilmektedir. Yani çevre değişkenleri aşağı seviyeli
bazı işlemlerin basit bir biçimde dışarıdan değiştirilmesine olanak sağlamaktadır. Bazı çevre değişkenleri
bazı POSIX fonksiyonları tarafından kullanılmaktadır. Örneğin ``exec`` fonksiyonlarının p'li biçimleri
prosesin ``PATH`` çevre değişkenine başvurmaktadır. Ya da örneğin dinamik bir kütüphane yüklenirken
dinamik yükleyici prosesin ``LD_LIBRARY_PATH`` çevre değişkenine başvurmaktadır. Bazen çevre değişkenleri
uygulama programcıları tarafından da kullanılmaktadır.

Programlarda Çevre Değişkeni ile Parametrik Ayar Örneği
-------------------------------------------------------

Örneğin biz programımız içerisinde bir dosyanın yerini belirlemek isteyelim. Ancak kullanıcı bu dosyayı
farklı bir yere yerleştirebiliyor olsun. Bunu bir çevre değişkeni ile ayarlanabilir hale getirebiliriz:

.. code-block:: c

    char *data_path = "datafile.dat";
    char *value;
    FILE *s;

    if ((value = getenv("DATA_LOCATION")) != NULL)
        data_path = value;

    if (f = fopen(data_path, "r")) == NULL) {
        fprintf(stderr, "cannot open file!...\n");
        exit(EXIT_FAILURE);
    }
    ...

Derleyicilerde Çevre Değişkeni Kullanımı (C_INCLUDE_PATH)
---------------------------------------------------------

Örneğin ``gcc`` derleyicisi ``<...>`` biçiminde include edilmiş dosyaların yerlerini aynı zamanda
``C_INCLUDE_PATH`` isimli bir çevre değişkeninde de aramaktadır. Yani derleyici standart include
dosyalarının bulunduğu yerin dışında bu çevre değişkeni ile belirtilen dizinlere de bakmaktadır. Tabii
birden fazla dizin belirtilebilir, bu durumda ``:`` ile onları ayırmak gerekir. Örneğin:

.. code-block:: console

    $ export C_INCLUDE_PATH=/home/kaan:/home/kaan/Study
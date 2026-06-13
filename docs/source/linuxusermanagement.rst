==============================
Unix/Linux Kullanıcı Yönetimi
==============================


Kullanıcı Yönetimi: ``/etc/passwd`` Dosyası
============================================

Linux sistemlerinde kullanıcılara ilişkin bilgiler ``/etc/passwd`` dosyası içerisinde tutulmaktadır. ``/etc/passwd``
dosyası satırlardan oluşmaktadır. Her satır ``:`` karakterleriyle ayrılmış 7 alandan oluşmaktadır. Örneğin:

.. code-block:: text

    ali:x:1002:1001::/home/ali:/bin/myshell

Bu alanlardaki bilgiler şunlardır:

.. code-block:: text

    username:password:UID:GID:GECOS:home_dir:shell

Birinci alanda *username* kullanıcının oturum açarken kullandığı ismi belirtmektedir. İkinci alanda şifrelenmiş parola
bilgisi bulunmaktadır. Eğer bu ikinci alanda yalnızca ``x`` harfi varsa bu durum şifre bilgisinin burada değil
``/etc/shadow`` dosyasında bulundurulduğu anlamına gelmektedir. ``/etc/shadow`` dosyası yalnızca *root* kullanıcısı
tarafından okunabilmektedir. Oysa ``/etc/passwd`` dosyası herkes tarafından okunabilmektedir. Her kullanıcının ileride
de ele alacağımız gibi bir kullanıcı adının yanı sıra çekirdek tarafından kullanılan bir kullanıcı id'si (user id)
vardır. Üçüncü alanda bu kullanıcı id'si bulundurulmaktadır. Sıradan kullanıcılara yönelik kullanıcı id'leri tipik
olarak 1000'den itibaren atanmaktadır. UNIX/Linux sistemlerinde aynı zamanda her kullanıcının bir grup bilgisi de
vardır. Gruplar da isimlere ve id'lere sahiptir. İşte dördüncü alanda kullanıcının grubunun id'si bulunmaktadır.
Kullanıcı id'leri ile isimler ``/etc/group`` dosyasında ilişkilendirilmiştir. Beşinci alanda kullanıcıya ilişkin
bilgiler bulundurulur; çekirdek bu alanla ilgilenmez. Altıncı alanda oturum açıldıktan sonra çalıştırılacak
programın çalışma dizini (yani oturum açıldıktan sonra hangi dizine geçileceği) belirtilmektedir. Geleneksel
olarak her kullanıcı için ``/home`` dizininin altında bir dizin yaratılır. Yedinci alanda oturum açıldığında
çalıştırılacak program bulundurulmaktadır. Oturum açıldığında tipik olarak ``/bin/bash`` gibi bir kabuk programı
çalıştırılmaktadır.

Linux sistemlerinde yeni kullanıcı oluşturmak aslında ``/etc/passwd`` dosyasına yeni bir satır eklemekten ibarettir.
``/etc/passwd`` dosyası herkes tarafından okunabilmesine karşın yalnızca *root* kullanıcısı tarafından
yazılabilmektedir. Bu nedenle bu dosyada değişiklik yapacaksanız editörünüzü ``sudo`` komutu ile çalıştırmalısınız.
Örneğin:

.. code-block:: bash

    $ sudo vim /etc/passwd

Örneğin bu dosyanın sonuna aşağıdakini ekleyerek yeni bir kullanıcı oluşturabiliriz:

.. code-block:: text

    ali::1001:1000:Ali Serçe,,,:/home/ali:/bin/bash

Burada parola alanının ``::`` biçiminde boş bırakıldığına dikkat ediniz. Eğer parola alanı boş bırakılırsa oturum
açma sırasında kullanıcıdan parola istenmeyecektir. Parola atamak için ``passwd`` programı kullanılmaktadır.
Örneğin:

.. code-block:: bash

    $ sudo passwd ali

Aynı zamanda bu yeni kullanıcı için ``/home`` dizininin altında bir dizin de yaratmamız gerekir:

.. code-block:: bash

    $ sudo mkdir /home/ali

Ancak yaratılan bu dizinin kullanıcısı ve grubu *root* olmaktadır. Onu şöyle değiştirebiliriz:

.. code-block:: bash

    $ sudo chown ali:study /home/ali

Aslında Linux sistemlerinde ``useradd`` ve ``adduser`` isimli komutlar zaten kendi içlerinde yukarıdaki adımlardan
geçerek kullanıcı yaratımını yapmaktadır. Örneğin:

.. code-block:: bash

    $ sudo useradd -m -s /bin/bash -c "Veli Can" veli
    $ sudo passwd veli

``-g`` seçeneği ile kullanıcının grubu da belirtilebilmektedir. (Aksi takdirde kullanıcı ismiyle aynı isimli yeni
bir grup yaratılmaktadır.) Örneğin:

.. code-block:: bash

    $ sudo useradd -m -g study -s /bin/bash -c "Veli Can" veli

Yukarıdaki işlemlerin tersi yapılarak kullanıcı silinebilir. Kullanıcı silmek için ``userdel`` isimli bir komut da
bulunmaktadır. Örneğin:

.. code-block:: bash

    $ sudo userdel -r veli

----

Basit Kabuk Programı
=====================

Şimdi kabuk görevi görecek basit bir program yazalım. Sonra da ``/etc/passwd`` dosyasının 7. alanını değiştirerek
oturum açtığımızda kendi kabuk programımızın çalıştırılmasını sağlayalım.

.. code-block:: c

    /* myshell.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    #define MAX_CMD_LINE        4096
    #define MAX_CMD_PARAMS      128

    struct cmd {
        const char *name;
        void (*proc)(void);
    };

    void parse_cmd_line(char *cmdline);
    void rm_proc(void);
    void cp_proc(void);
    void mv_proc(void);

    struct cmd g_cmds[] = {
        {"rm", rm_proc},
        {"cp", cp_proc},
        {"mv", mv_proc},
        {NULL, NULL}
    };

    char *g_params[MAX_CMD_PARAMS];
    int g_nparams;

    int main(void)
    {
        char cmdline[MAX_CMD_LINE];
        char *str;
        int i;

        for (;;) {
            printf("CSD>");
            fflush(stdout);

            if (fgets(cmdline, MAX_CMD_LINE, stdin) == NULL)
                continue;
            if ((str = strchr(cmdline, '\n')) != NULL)
                *str = '\0';

            parse_cmd_line(cmdline);
            if (!strcmp(g_params[0], "exit"))
                break;

            for (i = 0; g_cmds[i].name != NULL; ++i)
                if (!strcmp(g_cmds[i].name, g_params[0])) {
                    g_cmds[i].proc();
                    break;
                }
            if (g_cmds[i].name == NULL)
                printf("command not found: %s\n", g_params[0]);
        }

        return 0;
    }

    void parse_cmd_line(char *cmdline)
    {
        char *arg;

        g_nparams = 0;
        for ((arg = strtok(cmdline, " \t")); arg != NULL; arg = strtok(NULL, " \t"))
            g_params[g_nparams++] = arg;
        g_params[g_nparams] = NULL;
    }

    void rm_proc(void)
    {
        if (g_nparams == 1) {
            printf("too few command parameters!...\n");
            return;
        }
        printf("rm command...\n");
    }

    void cp_proc(void)
    {
        if (g_nparams != 3) {
            printf("wrong number of command parameters!...\n");
            return;
        }
        printf("cp command...\n");
    }

    void mv_proc(void)
    {
        if (g_nparams != 3) {
            printf("wrong number of command parameters!...\n");
            return;
        }
        printf("mv command...\n");
    }

----

POSIX Fonksiyonlarında Hata Yönetimi
=====================================

Bu bölümde POSIX fonksiyonlarının başarılarının nasıl kontrol edileceği ve başarısızlık durumunda hata mesajlarının
nasıl rapor edilebileceği üzerinde duracağız.

Başarı Kontrolü
----------------

POSIX fonksiyonlarının önemli bir bölümü ``int`` türden geri dönüş değerine sahiptir. Bu geri dönüş değeri onların
başarı durumunu belirtmektedir. Geri dönüş değeri ``int`` türünden olan POSIX fonksiyonlarının çoğu başarı durumunda
``0`` değerine, başarısızlık durumunda ``-1`` değerine geri dönmektedir. (``0`` değeri C'de mantıksal olarak *false*
biçiminde ele alınmasına karşın burada ``0`` değerinin *başarı* belirttiğine dikkat ediniz.) Bu durumda ``int`` geri
dönüş değeri türüne sahip bir POSIX fonksiyonunun başarısını tipik olarak şöyle test ederiz:

.. code-block:: c

    if (some_posix_func(...) == -1) {
        /* ... */
    }

Bazı programcılar kontrolü ``== -1`` ile değil ``< 0`` ile de yapabilmektedir. Örneğin:

.. code-block:: c

    if (some_posix_func(...) < 0) {
        /* ... */
    }

Böyle bir POSIX başarı kontrolü görürseniz *fonksiyonun başarısızlık durumunda herhangi bir negatif değere geri
dönebileceğini* düşünmemelisiniz. ``== -1`` kontrolü yerine ``< 0`` kontrolü bazı işlemcilerde mikro düzeyde daha
etkin bir kod üretimine yol açabilmektedir. Ancak biz kursumuzda başarı kontrolünü açıkça ``== -1`` ile yapacağız.

Bazı POSIX fonksiyonlarının geri dönüş değerleri bir göstericidir. Bu tür POSIX fonksiyonlar başarısızlık durumunda
NULL adres ile geri dönmektedir. Kontrolü şöyle yapabilirsiniz:

.. code-block:: c

    if ((ptr = some_other_posix_func(...)) == NULL) {
        /* ... */
    }

``errno`` Değişkeni
--------------------

POSIX fonksiyonları başarısızlık durumunda başarısızlığın nedenine ilişkin hata kodunu ``int`` türden global ``errno``
isimli bir değişkenin içerisine yazmaktadır. Yani bir POSIX fonksiyonu başarısız olmuşsa global ``errno`` değişkeni
başarısızlığın nedenini anlatan ``int`` türden bir değerle set edilmektedir. Her thread'in ``errno`` değişkeni
farklıdır. Bu nedenle ``errno`` uzunca bir süredir artık bir değişken olarak değil bir makro biçiminde
tanımlanmaktadır. Ancak biz onu bir değişken gibi kullanabiliriz. ``errno`` değişkenini doğrudan kullanabilmemiz
için ``<errno.h>`` dosyasını include etmemiz gerekir.

POSIX standartlarında ``errno`` değerleri için sayısal açıklamalar yapılmamıştır. Her ``errno`` değeri için
``<errno.h>`` içerisinde ``EXXX`` biçiminde bir sembolik sabit tanımlanmıştır. POSIX standartlarına göre bu sembolik
sabitlerin isimleri her sistemde aynı olmak zorundadır. Ancak sayısal değerleri sistemden sisteme farklı olabilir.
``0`` numaralı ``errno`` değeri özel bir değerdir ve hata belirtmek için kullanılmamaktadır. POSIX standartlarındaki
``errno`` değerleri şunlardır:

.. list-table:: POSIX ``errno`` Değerleri
   :header-rows: 1
   :widths: 25 75

   * - Hata Kodu
     - Standart Mesaj (İngilizce)
   * - ``E2BIG``
     - Argument list too long
   * - ``EACCES``
     - Permission denied
   * - ``EADDRINUSE``
     - Address already in use
   * - ``EADDRNOTAVAIL``
     - Cannot assign requested address
   * - ``EAFNOSUPPORT``
     - Address family not supported by protocol
   * - ``EAGAIN``
     - Resource temporarily unavailable
   * - ``EALREADY``
     - Connection already in progress
   * - ``EBADF``
     - Bad file descriptor
   * - ``EBADMSG``
     - Bad message
   * - ``EBUSY``
     - Device or resource busy
   * - ``ECANCELED``
     - Operation canceled
   * - ``ECHILD``
     - No child processes
   * - ``ECONNABORTED``
     - Connection aborted
   * - ``ECONNREFUSED``
     - Connection refused
   * - ``ECONNRESET``
     - Connection reset by peer
   * - ``EDEADLK``
     - Resource deadlock avoided
   * - ``EDESTADDRREQ``
     - Destination address required
   * - ``EDOM``
     - Numerical argument out of domain
   * - ``EDQUOT``
     - Disk quota exceeded
   * - ``EEXIST``
     - File exists
   * - ``EFAULT``
     - Bad address
   * - ``EFBIG``
     - File too large
   * - ``EHOSTUNREACH``
     - No route to host
   * - ``EIDRM``
     - Identifier removed
   * - ``EILSEQ``
     - Invalid or incomplete multibyte or wide character
   * - ``EINPROGRESS``
     - Operation now in progress
   * - ``EINTR``
     - Interrupted system call
   * - ``EINVAL``
     - Invalid argument
   * - ``EIO``
     - Input/output error
   * - ``EISCONN``
     - Transport endpoint is already connected
   * - ``EISDIR``
     - Is a directory
   * - ``ELOOP``
     - Too many levels of symbolic links
   * - ``EMFILE``
     - Too many open files
   * - ``EMLINK``
     - Too many links
   * - ``EMSGSIZE``
     - Message too long
   * - ``EMULTIHOP``
     - Multihop attempted
   * - ``ENAMETOOLONG``
     - File name too long
   * - ``ENETDOWN``
     - Network is down
   * - ``ENETUNREACH``
     - Network is unreachable
   * - ``ENFILE``
     - Too many open files in system
   * - ``ENOBUFS``
     - No buffer space available
   * - ``ENODATA``
     - No message is available on the STREAM head read queue
   * - ``ENODEV``
     - No such device
   * - ``ENOENT``
     - No such file or directory
   * - ``ENOEXEC``
     - Exec format error
   * - ``ENOLCK``
     - No locks available
   * - ``ENOLINK``
     - Link has been severed
   * - ``ENOMEM``
     - Cannot allocate memory
   * - ``ENOMSG``
     - No message of desired type
   * - ``ENOPROTOOPT``
     - Protocol not available
   * - ``ENOSPC``
     - No space left on device
   * - ``ENOSR``
     - Out of streams resources
   * - ``ENOSTR``
     - Device not a stream
   * - ``ENOSYS``
     - Function not implemented
   * - ``ENOTCONN``
     - Transport endpoint is not connected
   * - ``ENOTDIR``
     - Not a directory
   * - ``ENOTEMPTY``
     - Directory not empty
   * - ``ENOTRECOVERABLE``
     - State not recoverable
   * - ``ENOTSOCK``
     - Socket operation on non-socket
   * - ``ENOTSUP``
     - Operation not supported
   * - ``ENOTTY``
     - Inappropriate ioctl for device
   * - ``ENXIO``
     - No such device or address
   * - ``EOPNOTSUPP``
     - Operation not supported on socket
   * - ``EOVERFLOW``
     - Value too large for defined data type
   * - ``EOWNERDEAD``
     - Owner died
   * - ``EPERM``
     - Operation not permitted
   * - ``EPIPE``
     - Broken pipe
   * - ``EPROTO``
     - Protocol error
   * - ``EPROTONOSUPPORT``
     - Protocol not supported
   * - ``EPROTOTYPE``
     - Protocol wrong type for socket
   * - ``ERANGE``
     - Numerical result out of range
   * - ``EROFS``
     - Read-only file system
   * - ``ESPIPE``
     - Illegal seek
   * - ``ESRCH``
     - No such process
   * - ``ESTALE``
     - Stale file handle
   * - ``ETIME``
     - Timer expired
   * - ``ETIMEDOUT``
     - Connection timed out
   * - ``ETXTBSY``
     - Text file busy
   * - ``EWOULDBLOCK``
     - Operation would block
   * - ``EXDEV``
     - Invalid cross-device link

POSIX standartlarında fonksiyonların başarısızlık durumunda ``errno`` değişkenini hangi değerlerle set edeceği her
fonksiyonda ayrıca belirtilmiştir. POSIX fonksiyonları ``errno`` değişkenini başarısızlık durumunda set etmektedir.
Ancak standartlarda başarı durumunda ``errno`` değişkeninin değerinin değiştirilmeyeceği söylenmemiştir. Yani bir
POSIX fonksiyonu başarılı olsa bile ``errno`` değişkeninin değerini değiştirebilir. Programcının *fonksiyon başarısız
olmuşsa ``errno`` değişkenine başvurması* gerekir. POSIX fonksiyonları başarı durumunda ``errno`` değişkenine ``0``
gibi özel bir değer yerleştirmemektedir.

Ayrıca ``errno`` değişkeni Linux'ta çekirdek tarafından set edilen bir değişken değildir; tamamen kullanıcı
modundaki POSIX kütüphanesi tarafından set edilmektedir. Tipik olarak Linux çekirdeğinde bir sistem fonksiyonu
başarısız olduğunda sistem fonksiyonu negatif ``errno`` değeriyle geri döner. Sistem fonksiyonunu çağıran POSIX
fonksiyonu da bu negatif ``errno`` değerini pozitife dönüştürerek ``errno`` değişkenini set etmektedir.

``strerror`` Fonksiyonu
------------------------

Peki bir POSIX fonksiyonu başarısız olduğunda biz hataya ilişkin bir yazıyı nasıl ``stderr`` dosyasına
yazdırabiliriz? ``errno`` değişkenini ``switch`` içerisine sokarak yazdırma yapmak oldukça zahmetlidir. Örneğin:

.. code-block:: c

    if (some_posix_func() == -1) {
        switch (errno) {
            case EINVAL:
                /* ... */
                break;
            case EPERM:
                /* ... */
            /* ... */
        }
    }

Bir ``errno`` değerini alıp hata yazısını veren ``strerror`` isimli bir POSIX fonksiyonu bulunmaktadır. Fonksiyonun
prototipi şöyledir:

.. code-block:: c

    #include <string.h>

    char *strerror(int errnum);

Fonksiyon hata yazısının adresiyle geri dönmektedir. Hata yazılarına ilişkin diziler statik düzeyde tahsis
edilmiştir; geri döndürülen adresi ``free`` etmeye çalışmayınız. Örneğin ``rmdir`` POSIX fonksiyonuyla bir dizini
silmek isteyelim. Fonksiyon başarısız olduğunda ``-1`` değerine geri dönecektir. Biz de ``strerror`` fonksiyonu ile
hata mesajını aşağıdaki gibi yazdırabiliriz:

.. code-block:: c

    if (rmdir("mydir") == -1) {
        fprintf(stderr, "rmdir: %s\n", strerror(errno));
        exit(EXIT_FAILURE);
    }

``strerror`` fonksiyonu ile alınan hata yazısı varsayılan durumda İngilizce'dir. POSIX standartlarına göre bu
yazının içeriği locale'in ``LC_MESSAGES`` kategorisine göre ayarlanmaktadır. Dolayısıyla eğer mesajları Türkçe
yazdırmak istiyorsanız ``LC_MESSAGES`` kategorisine ilişkin locale'i ``setlocale`` fonksiyonu ile
değiştirmelisiniz. Tabii genel olarak tüm kategorilerin değiştirilmesi yoluna gidilmektedir. Türkçe Unicode UTF-8
locale'i ``"tr_TR.UTF-8"`` ile temsil edilmektedir. Dolayısıyla bu işlemi şöyle yapabilirsiniz:

.. code-block:: c

    if (setlocale(LC_ALL, "tr_TR.UTF-8") == NULL) {
        fprintf(stderr, "cannot set locale!...\n");
        exit(EXIT_FAILURE);
    }

``setlocale`` fonksiyonu aslında standart bir C fonksiyonudur. (Her standart C fonksiyonu aynı zamanda bir POSIX
fonksiyonu durumundadır.) Prototipi şöyledir:

.. code-block:: c

    #include <locale.h>

    char *setlocale(int category, const char *locale);

``perror`` Fonksiyonu
----------------------

POSIX fonksiyonlarında oluşan hatayı rapor etmek için ``perror`` isimli, daha pratik kullanımı olan bir POSIX
fonksiyonu (aynı zamanda standart C fonksiyonudur) bulundurulmuştur. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <stdio.h>

    void perror(const char *str);

Fonksiyon önce argüman olarak girilen yazıyı, sonra bu yazının hemen yanına ``:`` ve bir boşluk karakterini ve
sonra da o andaki ``errno`` değerinin yazısını yazdırır. En sonunda imleci aşağı satırın başına geçirir.
Fonksiyonun aşağıdaki gibi yazıldığını varsayabilirsiniz:

.. code-block:: c

    void perror(const char *str)
    {
        fprintf(stderr, "%s: %s\n", str, strerror(errno));
    }

Örneğin:

.. code-block:: c

    if (rmdir("mydir") == -1) {
        perror("rmdir");        /* rmdir: No such file or directory */
        exit(EXIT_FAILURE);
    }

``exit_sys`` Sarma Fonksiyonu
------------------------------

Kursumuzda bir POSIX fonksiyonu başarısız olduğunda genellikle (ancak her zaman değil) programımızı
sonlandıracağız. Bu durumda yazımı kolaylaştırmak için ``exit_sys`` isimli *sarma (wrapper)* bir fonksiyondan
faydalanacağız. Bu fonksiyon önce ``perror`` fonksiyonu ile hatayı ``stderr`` dosyasına yazdıracak, sonra da
``exit`` fonksiyonu ile programı sonlandıracaktır:

.. code-block:: c

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Örneğin:

.. code-block:: c

    if (rmdir("mydir") == -1)
        exit_sys("rmdir");

Örnek kullanım:

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        if (rmdir("mydir") == -1)
            exit_sys("rmdir");

        printf("Ok\n");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Bazı programcılar ``exit_sys`` fonksiyonunu ``printf`` fonksiyonuna benzetmektedir. Örneğin Stevens,
*Advanced Programming in the UNIX Environment* kitabında böyle bir sarma fonksiyon kullanmıştır. Böyle bir sarma
fonksiyona örnek şu olabilir:

.. code-block:: c

    #include <string.h>
    #include <errno.h>
    #include <stdarg.h>

    void exit_sys(const char *format, ...)
    {
        va_list ap;

        va_start(ap, format);
        vfprintf(stderr, format, ap);
        fprintf(stderr, ": %s\n", strerror(errno));
        va_end(ap);

        exit(EXIT_FAILURE);
    }

Bu versiyonda ``exit_sys`` fonksiyonu dosya adı gibi ek bağlam bilgisini de mesaja ekleyebilmektedir. Örneğin:

.. code-block:: c

    #include <stdio.h>
    #include <string.h>
    #include <stdlib.h>
    #include <stdarg.h>
    #include <errno.h>
    #include <fcntl.h>

    void exit_sys(const char *format, ...);

    int main(void)
    {
        int fd;
        char path[] = "xxx.txt";

        if ((fd = open(path, O_RDONLY)) == -1)
            exit_sys("open (%s)", path);

        printf("success\n");

        return 0;
    }

    void exit_sys(const char *format, ...)
    {
        va_list ap;

        va_start(ap, format);
        vfprintf(stderr, format, ap);
        fprintf(stderr, ": %s\n", strerror(errno));
        va_end(ap);

        exit(EXIT_FAILURE);
    }

``errno`` ve Standart C Fonksiyonları
--------------------------------------

Aslında ``errno`` kavramı C standartlarında da bulunmaktadır. Ancak C standartlarında ``errno`` değeri çok kısıtlı
bir biçimde kullanılmıştır. Yani C standartlarında pek az fonksiyon ``errno`` değişkenini set etmektedir. C
standartları, standart C fonksiyonlarının ``errno`` değişkenini derleyiciye bağlı olarak da set edebileceğini
belirtmektedir. Daha önce de belirttiğimiz gibi POSIX standartlarına göre her standart C fonksiyonu aynı zamanda
bir POSIX fonksiyonu olarak ele alınmaktadır. POSIX'teki standart C fonksiyonları aynı zamanda ``errno``
değişkenini de set etmektedir. Örneğin biz ``fopen`` fonksiyonu ile bir dosyayı açmak istesek, ``fopen`` başarısız
olduğunda UNIX/Linux sistemleri ``errno`` değerini uygun biçimde set edebilmektedir. Böylece biz standart C
fonksiyonlarındaki hata mesajlarını da aşağıdaki gibi yazdırabilmekteyiz:

.. code-block:: c

    if ((f = fopen("test.dat", "r")) == NULL)
        exit_sys("fopen");

Ya da örneğin:

.. code-block:: c

    if ((p = malloc(SIZE)) == NULL)
        exit_sys("malloc");

Her ne kadar standart C fonksiyonları UNIX/Linux sistemlerinde ``errno`` değişkenini set ediyorsa da biz standart
C uyumunu korumak için kursumuzda standart C fonksiyonlarında ``errno`` değişkeninden faydalanmayacağız. Örneğin:

.. code-block:: c

    if ((f = fopen("test.dat", "r")) == NULL) {
        fprintf(stderr, "cannot open file!...\n");
        exit(EXIT_FAILURE);
    }

----

Kullanıcı ve Grup Kimlik Bilgileri
===================================

UNIX/Linux sistemlerinde her kullanıcının bir *kullanıcı ismi (user name)* ve bir kullanıcı id'si (user id)
bulunmaktadır. Kullanıcıların isimleri ile onların id'leri daha önce de belirttiğimiz gibi ``/etc/passwd``
dosyasında eşleştirilmiştir. Örneğin ``/etc/passwd`` dosyasında *kaan* kullanıcısının bulunduğu satır şöyledir:

.. code-block:: text

    kaan:x:1000:1000:Kaan Aslan,,,:/home/kaan:/bin/bash

Burada birinci alandaki ``kaan`` kullanıcının ismini, üçüncü alandaki ``1000`` ise onun id'sini belirtmektedir.
Çekirdek isimlerle değil id'lerle işlem yapmaktadır. İsimler kullanıcılar için okunabilirlik sağlamak amacıyla
bulundurulmaktadır.

UNIX/Linux sistemlerinde her kullanıcı aynı zamanda bir grup içerisindedir. Buna kullanıcının grubu denilmektedir.
Grupların da isimleri ve id'leri vardır. Grup isimleri grup id'leriyle ``/etc/group`` dosyasında
eşleştirilmiştir. Yukarıdaki ``/etc/passwd`` dosyasındaki *kaan* kullanıcısına ilişkin satırın dördüncü alanında
grubun id'si yer almaktadır. (Kullanıcı id'leri ile grup id'leri farklı isim alanlarındadır. Yani bunların
numaralarının çakışması bir sorun oluşturmamaktadır.) ``/etc/group`` dosyasındaki satırlar aşağıdaki gibidir:

.. code-block:: text

    study:x:1000:

Burada ``study`` grup ismine karşı gelen id'nin ``1000`` olduğu görülmektedir.

Peki *grup (group)* ne anlama gelmektedir? İşte *bir grup kullanıcı ortak dosyalar üzerinde işlem yapabilsin ancak
diğerleri yapmasın* durumunu oluşturmak için grup kavramı kullanılmıştır. Eskiden bir kullanıcının tek bir grubu
olabiliyordu. Zaman içerisinde bunun yetersizliği görüldü ve bir kullanıcının birden fazla grup içerisinde
bulunabilmesi sağlandı. Güncel UNIX/Linux sistemlerinde her kullanıcının asli bir grubu vardır. Buna *gerçek grup
(real group)* da denilmektedir. Ancak ayrıca bir kullanıcı birden fazla gruba da üye olabilmektedir. Bunlara da
kullanıcının *ek grupları (supplementary groups)* denilmektedir. Çekirdek gerçek gruplarla ek gruplar arasında bir
ayrım yapmamaktadır. Bir kullanıcının bilgileri kabuk üzerinde ``id`` komutuyla görüntülenebilmektedir. Örneğin:

.. code-block:: bash

    $ id
    uid=1000(kaan) gid=1000(study) gruplar=1000(study),4(adm),24(cdrom),27(sudo),30(dip),
    46(plugdev),100(users),105(lpadmin),125(sambashare)
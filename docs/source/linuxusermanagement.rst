==============================
Unix/Linux Kullanıcı Yönetimi
==============================


Kullanıcı Yönetimi: /etc/passwd Dosyası
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

errno Değişkeni
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

strerror Fonksiyonu
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

perror Fonksiyonu
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

exit_sys Sarma Fonksiyonu
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

errno ve Standart C Fonksiyonları
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



Proses Kavramı
===============

İşletim sistemlerinde çalışmakta olan programlara *proses (process)* denilmektedir. (*Process* sözcüğü Türkçede
*süreç* biçiminde ifade edilmektedir. *Süreç* dilimizde oldukça yaygınlaşmış bir sözcüktür. Ancak biz kursumuzda
*süreç* yerine bu kavramı *proses* biçiminde ifade edeceğiz.) *Program* terimi *bir programın kaynak kodları*
anlamında ya da *derlenmiş çalıştırılabilir dosya* anlamında kullanılmaktadır. Biz bir programı çalıştırdığımızda
artık işletim sistemi bakış açısıyla bir proses oluşturulmuş olur. İşletim sistemleri bir program çalıştığında
çekirdek alanı içerisinde çalıştırılan o program için birtakım veri yapıları oluşturmaktadır. Proses canlı bir
kavramı ifade etmektedir. Aynı programı biz birden fazla kez çalıştırabiliriz. Bu durumda işletim sistemleri
çalıştırılabilen dosya aynı olsa da birbirinden bağımsız farklı prosesler oluşturmaktadır.

----

Proses Hiyerarşisi: Üst Proses ve Alt Proses
=============================================

İşletim sistemlerinde her proses (yani her çalışan program) başka bir proses tarafından (yani çalışmakta olan
başka bir program tarafından) yaratılmaktadır. Örneğin Linux'ta komut satırına geçip ``ls`` komutunu
uyguladığımızda aslında kabuk ``/bin`` içerisindeki ``ls`` programını çalıştırmaktadır. Yani ``ls`` programı
proses haline gelmektedir. Kabuk programının kendisi de bir prosestir. UNIX/Linux sistemlerinde proseslerin
yaratılması ileride ayrıntılarıyla görecek olduğumuz ``fork`` isimli POSIX fonksiyonuyla yapılmaktadır.

Her proses diğerini yarattığına göre prosesler arasında bir *altlık-üstlük (parent)* ilişkisinin olması gerekir.
İşte UNIX/Linux sistemlerinde bir prosesi yaratan prosese *üst proses (parent process)*, yaratılan prosese de
*alt proses (child process)* denilmektedir. Örneğin biz ``bash`` komut satırında kendi ``sample`` programımızı
çalıştırmış olalım. Bu durumda ``sample`` programının üst prosesi ``bash`` olacaktır. ``sample`` prosesi de
``bash`` prosesinin alt prosesi olacaktır.

Her proses başka bir proses tarafından yaratıldığına göre peki ``bash`` kabuk prosesi kimin tarafından
yaratılmaktadır? İşte bize *user name* ve *password* soran aslında ``login`` prosestir. ``login`` prosesi
``bash`` programını çalıştırmaktadır. Peki ``login`` programını kim çalıştırmaktadır? İşte proses hiyerarşisinin
en tepesinde ``init`` denilen bir proses bulunmaktadır. UNIX/Linux sistemleri boot edildiğinde ilk çalıştırılan
proses bu ``init`` prosesidir. ``init`` prosesi tüm proseslerin atası durumundadır. ``init`` prosesi sonlanmaz,
sürekli bir biçimde *bir servis gibi* (UNIX/Linux sistemlerinde *servis* terimi yerine *daemon* terimi
kullanılmaktadır) çalışmaya devam etmektedir. Örneğin Linux sistemlerinde ``bash`` kabuğu çalıştırılana kadar
oluşturulan prosesler şöyledir:

.. code-block:: text

    PID 0 ── idle/swapper (çekirdek, kullanıcı alanı değil)
                │
                ▼
    PID 1 ── init / systemd
            (ilk kullanıcı alanı prosesi, çekirdek başlatır)
                │
        ┌───────┴────────┐
        │                │
        ▼                ▼
    getty / agetty    diğer servisler
    (TTY bekler)      (sshd, cron …)
        │                │
        │ (lokal giriş)  │ (SSH bağlantısı)
        │                │
        ▼                ▼
    login          sshd (child)
    (kimlik doğr.)  (oturumu yönetir)
        │                │
        └───────┬────────┘
                │
                ▼
            bash
            (kullanıcı kabuğu)

----

Proses ID (PID)
================

UNIX/Linux sistemlerinde (ileride de ayrıntılarıyla ele alacağız) her prosesin sistem genelinde *tek olan
(unique)* bir proses id'si vardır. Proses id değeri tamsayısal bir değerdir. POSIX standartlarında proseslerin
id değerleri ``pid_t`` türüyle temsil edilmektedir. POSIX standartlarında ``pid_t`` türünün *işaretli bir
tamsayı türü olacak biçimde* typedef edilmesi zorunlu tutulmuştur. Linux sistemlerinde ``pid_t`` türü ``int``
olarak typedef edilmiştir. Proseslerin id değerleri belli bir anda sistemde tektir. Bir proses sonlandıktan
belli bir zaman sonra o prosesin id değeri yeni prosesler için kullanılabilir. Prosesler hakkında bilgiler ``ps``
kabuk komutuyla elde edilmektedir. Örneğin:

.. code-block:: bash

    $ ps
      PID TTY          TIME CMD
    49784 pts/0    00:00:00 bash
    51143 pts/0    00:00:00 ps

Buradaki ``PID`` sütununda ilgili prosesin id değeri belirtilmektedir. Eğer prosesler arasındaki altlık-üstlük
ilişkisi de görülmek isteniyorsa ``ps`` komutu ``-l`` seçeneğiyle kullanılmalıdır. Örneğin:

.. code-block:: bash

    $ ps -l
    F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    0 S  1000   49784   49773  0  80   0 -  3568 do_wai pts/0    00:00:00 bash
    4 R  1000   51157   49784 99  80   0 -  4142 -      pts/0    00:00:00 ps

Buradaki ``PPID`` sütunu üst prosesin proses id değerini belirtmektedir. Burada ``ps`` prosesinin üst prosesinin
``bash`` olduğu görülmektedir.

``ps`` komutunun pek çok ayrıntısı vardır. Biz bu komut hakkında ileride daha fazla açıklamalar yapacağız.

Bir programı her yeniden çalıştırdığımızda yeni bir proses yaratıldığı için prosese yeni *proses id*
atanmaktadır. Prosese atanan *proses id* değerinin geçerliliği proses sonlandığında yok olmaktadır.

----

Gerçek ve Etkin Kullanıcı/Grup ID'leri
=======================================

UNIX/Linux sistemlerinde her prosesin bir *gerçek kullanıcı id'si (real user id)*, *gerçek grup id'si (real
group id)*, *etkin kullanıcı id'si (effective user id)* ve *etkin grup id'si (effective group id)* vardır.
*Prosesin kullanıcı id'si* denildiğinde varsayılan olarak *gerçek kullanıcı id'si*, *prosesin grup id'si*
denildiğinde de varsayılan olarak *gerçek grup id'si* anlaşılmaktadır. Peki gerçek id'lerle etkin id'ler
arasında ne farklılık vardır? Aslında prosesin *gerçek kullanıcı id'si* ile *etkin kullanıcı id'si*, *gerçek
grup id'si* ile de *etkin grup id'si* çoğu kez aynı değerdedir. Bunlar özel durumlarda farklılaşmaktadır. Biz bu
konuyu ileride ele alacağız. Ancak dosyalarda erişim işlemlerinde gerçek id'ler değil etkin id'ler test işlemine
sokulmaktadır. Her ne kadar çekirdek aslında sayısal id değerleriyle çalışıyorsa da programcılar tarafından
konuşmalarda kullanıcı ve grup id'leri Linux sistemlerindeki ``/etc/passwd`` ve ``/etc/group`` dosyalarında
belirlenmiş olan isimlerle ifade edilmektedir.

Peki bir proses yaratıldığında onun gerçek ve etkin kullanıcı ve grup id'leri nasıl oluşturulmaktadır? İşte bir
prosesin gerçek ve etkin kullanıcı ve grup id'leri alt proses yaratılırken (``fork`` işlemi sırasında) üst
prosesten alınmaktadır. Örneğin ``bash`` prosesinin gerçek ve etkin kullanıcı id'si ``kaan``, gerçek ve etkin
grup id'si de ``study`` olsun. Biz kabuk üzerinden ``sample`` programını çalıştırmış olalım. Bu durumda
``sample`` prosesinin gerçek ve etkin kullanıcı id'si ``kaan``, gerçek ve etkin grup id'si ise ``study``
olacaktır.

Peki kabuk prosesinin gerçek ve etkin kullanıcı ve grup id'leri nasıl oluşturulmaktadır? İşte bize *user name* ve
*password* soran ``login`` prosesi doğrulamayı yaparsa çalıştırdığı ``bash`` prosesinin gerçek ve etkin kullanıcı
ve grup id'lerini ``/etc/passwd`` dosyasındaki ilgili satırda belirtilen *kullanıcı ve grup id'si* ile set
etmektedir. ``/etc/passwd`` dosyasının satırlarının aşağıdaki gibi olduğunu anımsayınız:

.. code-block:: text

    kaan:x:1000:1000:Kaan Aslan,,,:/home/kaan:/bin/bash

Buradaki ilk ``1000`` değeri kullanıcı id'sini, ikinci ``1000`` değeri grup id'sini belirtmektedir. İşte biz
login olduğumuzda ``/bin/bash`` çalıştırılarak kabuk prosesi oluşturulacak; o prosesin gerçek ve etkin kullanıcı
id'si ``1000 (kaan)`` olarak, gerçek ve etkin grup id'si de ``1000 (study)`` olarak set edilecektir.

----

Dosyaların Kullanıcı ve Grup ID'leri
=====================================

UNIX/Linux sistemlerinde aynı zamanda her dosyanın da bir kullanıcı id'si (user id) ve grup id'si (group id)
bulunmaktadır. (Dosyaların gerçek ve etkin biçiminde id'leri yoktur, yalnızca id'leri vardır.) Dosyanın kullanıcı
ve grup id'lerini ``ls -l`` komutu ile görebilirsiniz. Örneğin:

.. code-block:: bash

    $ ls -l
    toplam 52
    drwxrwxr-x 2 kaan study  4096 May 21 13:44 01-CommandLineParsing
    drwxrwxr-x 2 kaan study  4096 Haz  4 10:27 02-BasicCustomShell
    drwxrwxr-x 2 kaan study  4096 Haz  4 12:18 03-POSIXErr
    -rwxrwxr-x 1 kaan study 15960 May  7 12:33 a.out
    -rwxrwxr-x 1 kaan study 15960 Haz  9 11:51 sample
    -rw-rw-r-- 1 kaan study   110 Haz  9 11:51 sample.c
    -rw-rw-r-- 1 kaan study  1512 May  7 12:39 sample.o

Burada üçüncü sütunda dosyaların kullanıcı id'leri, dördüncü sütunda ise grup id'leri bulunmaktadır. Peki
dosyaların kullanıcı ve grup id'leri nasıl oluşturulmaktadır?

UNIX/Linux sistemlerinde tüm dosyalar ``open`` isimli bir POSIX fonksiyonu tarafından yaratılmaktadır. Bir
dosyanın kullanıcı id'si onu yaratan prosesin etkin kullanıcı id'sinden alınmaktadır. Örneğin çalışan
prosesimizin etkin kullanıcı id'si ``kaan (1000)`` olsun. Biz bu programda ``open`` fonksiyonu ile
``sample.txt`` dosyasını yaratmış olalım. Bu durumda ``sample.txt`` dosyasının kullanıcı id'si de ``kaan (1000)``
olacaktır.

Dosyaların grup id'lerinin set edilmesi konusunda tarihsel olarak UNIX/Linux sistemler arasında bir anlaşmazlık
oluşmuştur. Bazı sistemler ``open`` fonksiyonuyla yaratılan dosyaların grup id'lerini dosyayı yaratan prosesin
etkin grup id'si olarak, bazı sistemler ise yaratılan dosyanın içinde bulunduğu dizinin grup id'si olarak set
etmektedir. POSIX standartları her iki biçimi de geçerli kabul etmektedir. Linux sistemlerinde yaratılan
dosyanın grup id'si onu yaratan prosesin etkin grup id'si biçiminde set edilmektedir. BSD sistemlerinde ise
yaratılan dosyanın grup id'si dosyanın yaratıldığı dizinin grup id'si biçiminde set edilmektedir. Linux
sistemlerinde *mount parametreleriyle* BSD tarzı davranış istenirse oluşturulabilmektedir. Aynı zamanda Linux
sistemlerinde *dosyanın içinde bulunduğu dizinin set group id* bayrağı set edilerek de aynı etki
oluşturulabilmektedir.

----

Dosya ve Dizinlerin Erişim Hakları
===================================

UNIX/Linux sistemlerinde her dosyanın ve dizinin erişim hakları vardır. Teknik olarak dosyanın erişim haklarına
*dosyanın mode bilgisi* de denilmektedir. Dosyaların ve dizinlerin erişim hakları ``ls -l`` komutunda en soldaki
sütunda belirtilmektedir. Örneğin:

.. code-block:: bash

    $ ls -l
    toplam 52
    drwxrwxr-x 2 kaan study  4096 May 21 13:44 01-CommandLineParsing
    drwxrwxr-x 2 kaan study  4096 Haz  4 10:27 02-BasicCustomShell
    drwxrwxr-x 2 kaan study  4096 Haz  4 12:18 03-POSIXErr
    -rwxrwxr-x 1 kaan study 15960 May  7 12:33 a.out
    -rwxrwxr-x 1 kaan study 15960 Haz  9 11:51 sample
    -rw-rw-r-- 1 kaan study   110 Haz  9 11:51 sample.c
    -rw-rw-r-- 1 kaan study  1512 May  7 12:39 sample.o

Dosyaların erişim haklarına ilişkin en soldaki sütunun 10 karakterden oluştuğuna dikkat ediniz. Bu 10 karakterin
en solundaki karakter dosyanın türünü belirtmektedir. Burada ``d`` harfi varsa dosya bir dizin (directory)
belirtmektedir. Burada ``-`` karakteri varsa dosya sıradan bir dosyadır. Normal sıradan dosyalara UNIX/Linux
dünyasında İngilizce *regular file* denilmektedir. Erişim haklarının en başındaki karakter şunlardan biri
olabilir:

.. list-table:: Dosya Türü Karakterleri
   :header-rows: 1
   :widths: 20 80

   * - Karakter
     - Dosya Türü
   * - ``-``
     - Normal dosya
   * - ``d``
     - Dizin
   * - ``l``
     - Sembolik bağ dosyası
   * - ``c``
     - Karakter aygıt dosyası
   * - ``b``
     - Blok aygıt dosyası
   * - ``p``
     - İsimli boru (FIFO)
   * - ``s``
     - Unix domain socket

``ls -l`` komutunda erişim hakları sütununda dosyanın türünden sonra ``rwx`` biçiminde üç tane alan
bulunmaktadır:

.. code-block:: text

    rwx rwx rwx

İlk üçlük kısma *sahiplik (owner)* hakları, ikinci üçlük kısma *grup (group)* hakları ve son üçlük kısma da
*diğer (other)* hakları denilmektedir:

.. list-table:: Erişim Hakları Yapısı
   :header-rows: 1
   :widths: 25 25 25 25

   * - Dosya Türü
     - Owner (rwx)
     - Group (rwx)
     - Other (rwx)
   * - ``- d l c b p s``
     - ``r`` ``w`` ``x``
     - ``r`` ``w`` ``x``
     - ``r`` ``w`` ``x``

Sahiplik, grup ve diğer kısımlarda ``rwx`` hakları varsa ilgili karakter, yoksa ``-`` karakteri
kullanılmaktadır. Örneğin:

.. code-block:: text

    -rw-r--r--

Burada dosya normal bir dosyadır. Dosyanın sahiplik haklarının ``rw-`` olduğunu görüyorsunuz. Bu ``r`` hakkının
ve ``w`` hakkının olduğunu, ancak ``x`` hakkının olmadığını belirtmektedir. Grup haklarının ve diğer hakların
``r--`` biçiminde olduğuna dikkat ediniz. Bu durum da ``r`` hakkının olduğu ancak ``w`` ve ``x`` haklarının
olmadığı anlamına gelmektedir.


open Fonksiyonunda Erişim Kontrolü
=======================================

UNIX/Linux sistemlerinde normal dosyalar ``open`` POSIX fonksiyonuyla, dizin dosyaları ise ``opendir`` POSIX
fonksiyonuyla açılmaktadır. ``open`` POSIX fonksiyonunda (ileride ayrıntılarıyla açıklayacağız) dosyayı açarken
hangi niyetle açtığımızı fonksiyonun bir parametresiyle belirtiriz. Buna dosyanın açış modu da denilmektedir.
``open`` fonksiyonunda dosyalar üç biçimde açılabilmektedir:

.. list-table::
   :header-rows: 1
   :widths: 25 75

   * - Mod
     - Anlamı
   * - ``O_RDONLY``
     - Yalnızca okuma niyetiyle
   * - ``O_WRONLY``
     - Yalnızca yazma niyetiyle
   * - ``O_RDWR``
     - Hem okuma hem yazma niyetiyle

Örneğin:

.. code-block:: c

    fd = open("test.txt", O_RDWR);

Burada dosya hem okuma hem de yazma niyetiyle açılmak istenmiştir. İşte ``open`` fonksiyonunda çekirdek,
dosyanın niyet edilen açış modu ile dosyanın erişim haklarını kontrol eder. Eğer bu kontrol başarısız olursa
``open`` fonksiyonu başarısızlıkla geri döner.

İşletim sistemi ``open`` fonksiyonunda (ve diğer bazı fonksiyonlarda) sırasıyla şu kontrolleri uygulamaktadır
(bu kontroller ``else-if`` biçiminde sıralanmıştır):

1. Eğer işlem yapmak isteyen prosesin etkin kullanıcı id'si (etkin grup id'sinin burada önemi yoktur) ``0`` ise
   işlem yapmak isteyen proses sistemdeki yetkili kullanıcının bir prosesidir. Bu tür proseslere *root
   prosesler* ya da *super user prosesler* ya da *öncelikli (privileged) prosesler* denilmektedir. Bu durumda
   işletim sistemi, yapılmak istenen işlem ne olursa olsun bu işleme onay verir. Yani etkin kullanıcısı ``0``
   olan root prosesler her türlü dosyaya her türlü biçimde erişebilmektedir.

2. Eğer işlem yapmak isteyen prosesin etkin kullanıcı id'si (effective user id) dosyanın kullanıcı id'si ile
   aynıysa bu durumda *dosyanın sahibinin dosya üzerinde işlem yaptığı* gibi mantıksal bir çıkarım
   yapılmaktadır. Yapılmak istenen işlem ile dosyanın sahiplik (owner) erişim bilgileri karşılaştırılır. Eğer
   bu erişim bilgileri işlemi destekliyorsa işleme onay verilir, değilse işlem başarısızlıkla sonuçlanır.

3. Eğer işlem yapmak isteyen prosesin etkin grup id'si (effective group id) ya da *ek grup (supplementary
   groups)* id'lerinden biri dosyanın grup id'si ile aynıysa bu durumda *dosya ile aynı grupta bulunan bir
   kullanıcının dosya üzerinde işlem yaptığı* gibi mantıksal bir çıkarım yapılmaktadır. Yapılmak istenen işlem
   ile dosyanın grup (group) erişim bilgileri karşılaştırılır. Eğer bu erişim bilgileri işlemi destekliyorsa
   işleme onay verilir, değilse işlem başarısızlıkla sonuçlanır.

4. İşlem yapmak isteyen proses yukarıdaki üç durumdan herhangi birine girmiyorsa bu durumda yapılmak istenen
   işlem ile dosyanın *diğer (other)* erişim bilgileri karşılaştırılır. Eğer bu erişim bilgileri işlemi
   destekliyorsa işleme onay verilir, değilse işlem başarısızlıkla sonuçlanır.

Örnek
------

Örneğin aşağıdaki gibi bir dosya söz konusu olsun:

.. code-block:: text

    -rw-r--r-- 1 kaan study    20 Kas 13 13:54 test.txt

Dosyaya erişim yapmak isteyen proses, *okuma ve yazma amaçlı* erişim yapmak istesin. Yani örneğin ``open``
fonksiyonuyla dosyayı ``O_RDWR`` moduyla açmak istesin. Eğer prosesin etkin kullanıcı id'si ``0`` ise bu işlem
onaylanacaktır. Eğer prosesin etkin kullanıcı id'si ``kaan`` ise bu işlem yine onaylanacaktır, çünkü dosyanın
sahiplik erişim hakları ``rw-`` biçimdedir. Ancak prosesin etkin grup id'si ya da ek grup id'lerinden biri
``study`` ise işlem onaylanmayacaktır. Çünkü erişim hakları gruptaki üyelere yalnızca okuma izni vermektedir.
Benzer biçimde prosesin etkin kullanıcı id'si ya da etkin grup id'si (ve ek grup id'leri) burada
belirtilenlerin dışında ise yine prosese bu işlem için onay verilmeyecektir.

Yukarıdaki maddeler ``else-if`` biçiminde düşünülmelidir. Örneğin dosyanın erişim hakları aşağıdaki gibi olsun:

.. code-block:: text

    -r--rw-r-- 1 kaan study    20 Kas 13 13:54 test.txt

Burada dosyanın sahibi (yani etkin kullanıcı id'si dosyanın kullanıcı id'si ile aynı olan proses) dosya
üzerinde yazma yapamayacaktır. Ancak aynı grupta olan prosesler bunu yapabilecektir. Tabii bu biçimdeki erişim
hakları mantıksal olarak tuhaf ve anlamsızdır. Yani dosyanın sahibine verilmeyen bir hakkın gruba ya da
diğerlerine verilmesi normal bir durum değildir.

Çalıştırma Hakkı (x)
-------------------------

Çalıştırılabilir bir dosya ``x`` hakkı ile temsil edilmiştir. Bir dosyanın ``x`` hakkına sahip olabilmesi için
ya onun *derlenmiş ve bağlanmış çalıştırılabilir (executable) bir dosya* olması ya da bir *betik (script)
dosyası* olması gerekir. Çalıştırılamayan bir dosyanın ``x`` hakkına sahip olması zaten anlamsızdır. Ancak
çalıştırılabilir bir dosyanın da başkaları tarafından çalıştırılması engellenebilir. Örneğin:

.. code-block:: text

    -rwxr--r-- 1 kaan study 16816 Kas 13 13:49 sample

Burada muhtemelen dosya bir program dosyasıdır. Dosyanın sahibi (ve tabii root kullanıcısı) bu dosyayı
çalıştırabilir. Ancak diğer kullanıcılar bu dosyayı çalıştıramazlar. Örneğin:

.. code-block:: text

    -rw-r--r-- 1 kaan study 16816 Kas 13 13:49 sample

Burada artık root kullanıcısı da dosyayı çalıştıramaz. root kullanıcısının dosyayı çalıştırabilmesi için
sahiplik, grup ya da diğer erişim bilgilerinin en az birinde ``x`` hakkının belirtilmiş olması gerekmektedir.

Dosyanın ``x`` hakkı kontrolü, dosyayı yükleyip çalıştıran ``exec`` fonksiyonları tarafından yapılmaktadır.
``exec`` fonksiyonları ileride ayrı bir bölümde ele alınacaktır.

----

Erişim Hatası ve Dosya Erişim Haklarının Değiştirilmesi
========================================================

``open`` fonksiyonu ile dosya açım işlemi sırasında eğer ``open`` fonksiyonu yukarıda açıkladığımız erişim
hakları testinde başarısız olursa bu durumda ``errno`` değişkeni ``EACCES`` değeriyle set edilmektedir.
``EACCES`` ``errno`` değişkeninin İngilizce mesaj yazısı *Permission denied* biçimindedir.

Peki yeni yaratılan bir dosyanın erişim hakları nasıl belirlenmektedir? İşte dosyalar aslında ``open`` POSIX
fonksiyonuyla yaratılmaktadır. ``open`` fonksiyonuna yaratım sırasında erişim hakları da verilmektedir. Yani bu
durumda dosyanın erişim hakları, dosya yaratılırken dosyayı yaratan kişi tarafından belirlenmektedir. Dosyanın
erişim hakları daha sonra ``chmod`` isimli POSIX fonksiyonuyla ya da komut satırından ``chmod`` komutuyla
(``chmod`` programı da zaten ``chmod`` POSIX fonksiyonu kullanılarak yazılmıştır) değiştirilebilmektedir. Tabii
dosyanın erişim haklarını herkes değiştiremez. Erişim hakları ancak etkin kullanıcı id'si dosyanın kullanıcı
id'si ile aynı olan (yani dosyanın sahibi olan) prosesler tarafından ve etkin kullanıcı id'si ``0`` olan root
prosesleri tarafından değiştirilebilmektedir. ``chmod`` POSIX fonksiyonu ve ``chmod`` kabuk komutu ileride ele
alınacaktır.

sudo ve su Komutları
=============================

Modern UNIX/Linux sistemlerinde bir programın *etkin kullanıcı id'si 0 olacak biçimde çalıştırılmasını sağlayan*
``sudo`` isimli bir komut vardır. ``sudo`` komutu uygulandığında sistem root kullanıcısına ilişkin parolayı
sormaktadır. Yani ``sudo`` komutu ancak sistem yöneticileri tarafından kullanılabilmektedir. Tabii kendi Linux
makinemizde ana kullanıcının parolası aynı zamanda root parolası biçimindedir. Linux sistemlerinde her kullanıcı
``sudo`` komutunu kullanamamaktadır. ``sudo`` komutunu kullanabilmesi için kullanıcının *sudoer* olması
gerekmektedir. Biz bu kavramı ileride başka bir bölümde ele alacağız. Örneğin:

.. code-block:: bash

    $ sudo ./sample

Burada artık ``sample`` programı etkin kullanıcı id'si ``0`` olacak biçimde çalıştırılacaktır. Program her
türlü dosyaya erişebilecektir. Biz yukarıda ``chmod`` gibi kabuk komutuyla yalnızca kendi dosyalarımızın erişim
haklarını değiştirebileceğimizi söylemiştik. Tabii bu komutu ``sudo`` ile çalıştırırsak her türlü dosyanın
erişim haklarını değiştirebiliriz. Örneğin:

.. code-block:: bash

    $ sudo chmod 666 test.txt

Burada ``test.txt`` dosyasının erişim hakları, dosyanın sahibi kim olursa olsun, ``rw-rw-rw-`` biçiminde
değiştirilmektedir.

``sudo`` ismi *switch user and do* sözcüklerinden kısaltılmıştır. Aslında UNIX türevi sistemlerin çoğunda kabuk
üzerinde başka bir kullanıcı ile işlem yapmak için ``su`` komutu da bulunmaktadır. Örneğin:

.. code-block:: bash

    $ su ali

Burada adeta ``ali`` kullanıcısıyla oturum açılmış etkisi yaratılmaktadır. Geri dönmek için ``exit`` komutu
uygulanmaktadır.

``su`` komutu ile *root* için komut satırına da geçebilirsiniz. Bunun için sudoer olmak gerekir. Örneğin:

.. code-block:: bash

    $ sudo su root

----

appropriate privileges Terimi
==================================

POSIX standartlarında erişim mekanizması üzerinde açıklamalar yapılırken *root önceliği* ya da *prosesin etkin
kullanıcı id'sinin 0 olması* gibi bir anlatım tercih edilmemiştir. Onun yerine POSIX standartlarında
*appropriate privileges* terimi kullanılmıştır. Çünkü bir POSIX sistemi *ya hep ya hiç* biçiminde tasarlanmak
zorunda değildir. Gerçekten de örneğin Linux sistemlerinde *capability* denilen bir özellik bulunmaktadır. Bu
*capability* sayesinde bir prosesin etkin kullanıcı id'si ``0`` olmamasına karşın o proses, belirlenen bazı
şeyleri yapabilir duruma getirilebilmektedir. İşte POSIX standartlarındaki *appropriate privileges* terimi bunu
anlatmaktadır. Yani buradaki *appropriate privileges* terimi *prosesin etkin kullanıcı id'sinin 0 olduğunu ya da
0 olmasa da prosesin bu işlemi yapabilme yeteneğine sahip olduğunu* belirtmektedir. Biz *appropriate
privileges* terimi yerine Türkçe *uygun önceliğe sahip prosesler* terimini kullanacağız.

Klasik UNIX tasarımında *ya hep ya hiç* sistemi kullanılmıştır. Yani ya bir kullanıcı root olarak her şeyi
yapabilir ya da yalnızca kendine ilişkin şeyleri yapabilir. Ancak bu *ya hep ya hiç* sistemi bazı UNIX türevi
sistemler tarafından zaman içerisinde gevşetilmiştir. Örneğin Linux sistemlerinde yukarıda da belirttiğimiz
*capability* denilen özellik sayesinde prosesler *her şeyi değil bazı şeyleri yapabilir* hale
getirilebilmektedir. İşte bu nedenle POSIX standartları *root* terimini ya da *proses id'si 0 olan prosesler*
terimi yerine *appropriate privileges* terimini kullanmaktadır.

----

Yol İfadeleri (Pathnames)
==========================

Şimdi de UNIX/Linux sistemlerinde *yol ifadeleri* üzerinde duralım. Bir dosyanın ya da dizinin dosya sisteminde
nerede olduğunu belirten yazısal ifadelere *yol ifadeleri* denilmektedir. Yol ifadesi için İngilizce *path* ya
da *pathname* terimleri kullanılmaktadır. *Path* sözcüğü günlük yaşamda *pathname* sözcüğüyle eşdeğer biçimde
kullanılmaktadır. POSIX standartları dosyanın yerini belirten yazısal ifadeler için *pathname* sözcüğünü tercih
etmiştir. Windows sistemlerinde *pathname* genellikle hedefi dizin değil normal bir dosya olan yol ifadeleri
için kullanılmaktadır. Biz kursumuzda *path* ya da *pathname* terimi yerine Türkçe *yol ifadesi* terimini
kullanacağız. İngilizce *path* sözcüğü patika yol ya da mantıksal yol anlamına gelmektedir.

Örneğin:

.. code-block:: text

    "/home/kaan/Study/test.txt"
    "notes/list.txt"
    "sample.txt"

birer yol ifadesidir.

Mutlak ve Göreli Yol İfadeleri
-------------------------------

Yol ifadeleri iki gruba ayrılmaktadır: *Mutlak yol ifadeleri (absolute pathnames)* ve *göreli yol ifadeleri
(relative pathnames)*. Eğer yol ifadesinin ilk karakteri ``/`` ise böyle yol ifadelerine mutlak yol ifadeleri
denilmektedir. Mutlak yol ifadeleri her zaman kök dizinden itibaren çözülmektedir. Örneğin:

.. code-block:: text

    "/home/kaan/test.txt"

Bu yol ifadesinde kökün altında ``home`` dizini, ``home`` dizininin altında ``kaan`` dizini olmalıdır ve
``kaan`` dizininin altındaki ``test.txt`` dosyasına referans edilmektedir.

Göreli yol ifadelerinin ilk karakteri ``/`` değildir. Örneğin:

.. code-block:: text

    "notes/readme.txt"

Bu bir göreli yol ifadesidir. Peki göreli yol ifadeleri nereden itibaren bir yol belirtmektedir? İşte göreli yol
ifadeleri prosesin *çalışma dizini (current working directory)* denilen bir dizinden itibaren yol
belirtmektedir. Proseslerin çalışma dizinleri *proses kontrol bloğu* içerisinde saklanmaktadır ve göreli yol
ifadeleri için orijin belirtmektedir. Örneğin prosesimizin çalışma dizini ``/home/student`` olsun. Bu durumda
``notes/readme.txt`` yol ifadesi aslında ``/home/student/notes/readme.txt`` mutlak yol ifadesiyle aynı anlama
gelmektedir. Örneğin:

.. code-block:: text

    "test.txt"

Bu yol ifadesi de görelidir. Prosesin çalışma dizini hangi dizinse bu yol ifadesi de o dizindeki ``test.txt``
dosyasını belirtmektedir.

Proseslerin çalışma dizinleri işin başında üst prosesten alınmaktadır; ancak program çalışırken
değiştirilebilmektedir. Kabuk programları genellikle kendi çalışma dizinlerini prompt'un bir parçası olarak
komut satırına yazdırmaktadır. Örneğin:

.. code-block:: text

    kaan@kaan-virtual-machine:~/Study$

Burada ``:`` karakterinden sonraki ``~`` karakteri *home dizinini* belirtmektedir. Biz bu komut satırında bir
programı çalıştıracak olsak, yaratılacak prosesin çalışma dizini şu andaki prompt'ta belirtilen dizin olacaktır.
Buradaki ``~`` karakteri programlamada kullanılan bir karakter değildir; kabuk programının kullandığı bir
semboldür.

Yol Ayıracı ve Yol Bileşenleri
-------------------------------

Bir yol ifadesindeki dizin geçişleri UNIX/Linux sistemlerinde ``/`` karakteriyle belirtilmektedir. Windows
sistemleri ``/`` karakteri yerine ``\`` karakterini kullanmaktadır. Bu karakterlere *yol ayıracı (path
separator)* denilmektedir. Yol ifadelerindeki dizin geçişlerinde birden fazla ``/`` karakteri yan yana
kullanılabilmektedir. Örneğin aşağıdaki yol ifadesi geçerlidir:

.. code-block:: text

    "/home///////kaan////test.txt"

Yol ifadelerindeki her bir bileşene *yol bileşeni (path component)* denilmektedir. Örneğin:

.. code-block:: text

    "/home/kaan/temp/test.txt"

Buradaki yol bileşenleri şunlardır:

.. code-block:: text

    home
    kaan
    temp
    test.txt

UNIX/Linux sistemlerinde dosya ve dizin isimleri *büyük harf küçük harf duyarlılığına (case sensitivity)*
sahiptir. (Ancak Windows sistemlerinde yol bileşenlerinde büyük harf küçük harf duyarlılığı yoktur. Windows
dosya ve dizin isimlerini kullanıcının girdiği biçimde saklamaktadır ancak işleme sokarken büyük harf küçük harf
ayrımı yapmamaktadır.)

. ve .. Özel Yol Bileşenleri
-------------------------------------

Yol ifadelerinde ``.`` ve ``..`` karakterleri özel yol bileşenleridir. ``.`` yol bileşeni *yol ifadesindeki son
dizini*, ``..`` ise *önceki dizini* belirtmektedir. Örneğin:

.. code-block:: text

    "/home/student/notes/./llm/../../test.txt"

Yol ifadesindeki ``.`` yol bileşeninin bir etkisi yoktur. Yani ``/home/student/notes/.`` yol ifadesi ile
``/home/student/notes`` eşdeğerdir. ``..`` yol bileşeni son dizinin öncesini belirtmektedir. Bu yol ifadesini
aşama aşama çözümleyelim:

- ``/home/student/notes/.`` yol ifadesi yukarıda da belirttiğimiz gibi ``/home/student/notes`` ile eşdeğerdir.
- ``/home/student/notes/./llm`` yol ifadesi ``/home/student/notes/llm`` anlamına gelmektedir.
- ``/home/student/notes/./llm/..`` yol ifadesi ``/home/student/notes/`` anlamına gelmektedir.
- ``/home/student/notes/./llm/../..`` yol ifadesi ``/home/student`` anlamına gelmektedir.
- ``/home/student/notes/./llm/../../test.txt`` yol ifadesi de ``/home/student/test.txt`` anlamına gelmektedir.

Şimdi *mademki ``.`` yol bileşeni zaten bulunulan dizini belirtiyor, o zaman bunun kullanılmasına ne gerek var?*
sorusu aklınıza gelebilir. İşte bazı durumlarda bu belirlemenin açıkça yapılması gerekebilmektedir. Kabuk
üzerinde kullanılan ``~`` sembolü *home* dizini anlamına gelmektedir. Ancak bu sembol kabuğa ilişkindir.
Çekirdekte böyle bir yol bileşeni yoktur.

Yol İfadesinin Çözümlenmesi (Pathname Resolution)
--------------------------------------------------

İşletim sistemlerinde bir yol ifadesi verildiğinde, işletim sisteminin o yol ifadesine ilişkin hedeflenen
dosyayı ya da dizini elde etmesi sürecine *yol ifadesinin çözümlenmesi (pathname resolution)* denilmektedir.
Yol ifadelerinin çözümlenmesi, dizin geçişleriyle yapılan yavaş bir işlemdir. Bu nedenle işletim sistemleri bu
işlemi hızlandırmak için bir önbellek mekanizması kullanmaktadır.

``/etc/passwd`` dosyasındaki kullanıcılara ilişkin satırları anımsayınız. Örneğin:

.. code-block:: text

    kaan:x:1000:1000:Kaan Aslan,,,:/home/kaan:/bin/bash

Burada 6. sütundaki yol ifadesi, 7. sütundaki program çalıştırıldığında onun varsayılan çalışma dizinini
belirtmektedir. Yani ``login`` prosesi, 7. sütundaki programı, onun çalışma dizini *6. sütundaki dizin* olacak
biçimde çalıştırmaktadır.

Proseslerin çalışma dizinleri proses kontrol bloğunda göreli değil mutlak bir biçimde tutulmaktadır.

Aslında bir prosesin kök dizini de değiştirilebilmektedir. Prosesin kök dizininin değiştirilmesine İngilizce
*change root* işlemi de denilmektedir. Ancak *change root* işlemi tehlikeli bir işlemdir. Çünkü prosesin kök
dizini değiştiğinde artık tüm mutlak yol ifadeleri o dizin referans alınarak çözülmeye çalışılır. *Change root*
işlemini yapmadan önce pek çok hazırlığın yapılması gerekebilmektedir. Prosesin kök dizini de proses kontrol
bloğunda tutulmaktadır.

----

getcwd Fonksiyonu
======================

Prosesin çalışma dizini ``getcwd`` isimli POSIX fonksiyonuyla elde edilebilmektedir. Fonksiyonun prototipi
şöyledir:

.. code-block:: c

    #include <unistd.h>

    char *getcwd(char *buf, size_t size);

Fonksiyonun birinci parametresi, yol ifadesinin yerleştirileceği dizinin adresini; ikinci parametresi ise bu
dizinin uzunluğunu almaktadır. Fonksiyon başarı durumunda birinci parametresiyle belirtilen adresin aynısına,
başarısızlık durumunda NULL adrese geri dönmektedir. Fonksiyonun ikinci parametresinde belirtilen uzunluk eğer
yol ifadesini ve null karakteri içerecek büyüklükte değilse fonksiyon başarısız olur. Bu durumda ``errno``
değişkeni ``ERANGE`` değeriyle set edilmektedir. Örneğin:

.. code-block:: c

    char cwd[4096];

    if (getcwd(cwd, 4096) == NULL)
        exit_sys("getcwd");

UNIX/Linux sistemlerinde bir yol ifadesinin maksimum karakter sayısı (null karakter dahil olmak üzere)
``<limits.h>`` içerisindeki ``PATH_MAX`` sembolik sabitiyle belirtilmiştir. Ancak bu konuda bazı ayrıntılar
vardır. Bazı sistemlerde bu ``PATH_MAX`` sembolik sabiti tanımlı değildir. Dolayısıyla bazı sistemlerde
maksimum yol ifadesi uzunluğu ``pathconf`` denilen özel bir fonksiyon ile elde edilebilmektedir. Linux
sistemlerinde ``<limits.h>`` dosyası içerisinde ``PATH_MAX`` sembolik sabiti ``4096`` olarak tanımlanmıştır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <limits.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        char buf[PATH_MAX];

        if (getcwd(buf, PATH_MAX) == NULL)
            exit_sys("getcwd");

        puts(buf);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }


chdir Fonksiyonu
=====================

Prosesin çalışma dizini ``chdir`` isimli POSIX fonksiyonuyla değiştirilmektedir. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int chdir(const char *path);

Fonksiyon yeni çalışma dizininin yol ifadesini parametre olarak alır. Başarı durumunda ``0`` değerine,
başarısızlık durumunda ``-1`` değerine geri döner. Örneğin:

.. code-block:: c

    if (chdir("/home/student/notes") == -1)
        exit_sys("chdir");

Tabii ``chdir`` fonksiyonunun argümanı göreli bir yol ifadesi biçiminde de girilebilir. Ancak sistem her zaman
çalışma dizinini mutlak yol ifadesiyle tutmaktadır. Örneğin prosesimizin çalışma dizini ``/home/student`` olsun.
Biz de fonksiyonu aşağıdaki gibi çağırmış olalım:

.. code-block:: c

    if (chdir("notes") == -1)
        exit_sys("chdir");

Burada prosesin çalışma dizini ``/home/student/notes`` biçiminde olacaktır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <limits.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        char buf[PATH_MAX];

        if (getcwd(buf, PATH_MAX) == NULL)
            exit_sys("getcwd");

        puts(buf);

        if (chdir("/usr/bin") == -1)
            exit_sys("chdir");

        if (getcwd(buf, PATH_MAX) == NULL)
            exit_sys("getcwd");

        puts(buf);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

Kabuk Programına cd Komutunun Eklenmesi
============================================

Şimdi daha önce yazmış olduğumuz kabuk programımıza ``cd`` komutunu ekleyelim. Örneğimizde kabuğumuzun çalışma
dizinini ``g_cwd`` global dizisinde tuttuk. Her ``cd`` komutu girildiğinde ``getcwd`` uygulayarak yeni çalışma
dizinini bu diziye yerleştirdik:

.. code-block:: c

    void cd_proc(void)
    {
        if (g_nparams != 2) {
            printf("wrong number of command parameters!..\n");
            return;
        }

        if (chdir(g_params[1]) == -1) {
            printf("%s: \"%s\"\n", strerror(errno), g_params[1]);
            return;
        }

        if (getcwd(g_cwd, PATH_SIZE) == NULL)
            exit_sys("fatal error");
    }

Programın tamamı şöyledir:

.. code-block:: c

    /* myshell.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <errno.h>
    #include <unistd.h>

    #define MAX_CMD_LINE            4096
    #define MAX_CMD_PARAMS          1024
    #define PATH_SIZE               4096

    struct cmd {
        const char *name;
        void (*proc)(void);
    };

    void parse_cmd_line(char *cmdline);
    void rm_proc(void);
    void cp_proc(void);
    void mv_proc(void);
    void cd_proc(void);
    void exit_sys(const char *msg);

    struct cmd g_cmds[] = {
        {"rm", rm_proc},
        {"cp", cp_proc},
        {"mv", mv_proc},
        {"cd", cd_proc},
        {NULL, NULL}
    };

    char *g_params[MAX_CMD_PARAMS];
    int g_nparams;
    char g_cwd[PATH_SIZE];

    int main(void)
    {
        char cmdline[MAX_CMD_LINE];
        char *str;
        int i;

        if (getcwd(g_cwd, PATH_SIZE) == NULL)
            exit_sys("fatal error");

        for (;;) {

            printf("CSD:%s$ ", g_cwd);
            fflush(stdout);

            if (fgets(cmdline, MAX_CMD_LINE, stdin) == NULL)
                continue;
            if ((str = strchr(cmdline, '\n')) != NULL)
                *str = '\0';

            parse_cmd_line(cmdline);
            if (g_nparams == 0)
                continue;
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

    void cd_proc(void)
    {
        if (g_nparams != 2) {
            printf("wrong number of command parameters!..\n");
            return;
        }

        if (chdir(g_params[1]) == -1) {
            printf("%s: \"%s\"\n", strerror(errno), g_params[1]);
            return;
        }

        if (getcwd(g_cwd, PATH_SIZE) == NULL)
            exit_sys("fatal error");
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

Dizinlerin Yapısı ve Erişim Hakları
====================================

Dizinler de işletim sistemi tarafından birer dosyaymış gibi ele alınmaktadır. Gerçekten de dizinleri sanki
*içerisinde dosya bilgilerini tutan dosyalar* gibi düşünebiliriz. Yani dizinler *dizin giriş bilgilerinden*
oluşmaktadır. Her dizin girişi bir dosya ya da dizin hakkında bilgi tutmaktadır. Bir dizini temsili olarak şöyle
bir yapı gibi düşünebilirsiniz:

.. code-block:: text

    <dizin_girişi>
    <dizin_girişi>
    <dizin_girişi>
    ...

Dizinler ileride göreceğimiz gibi ``opendir`` POSIX fonksiyonuyla açılıp içindeki girişler ``readdir`` POSIX
fonksiyonuyla okunmaktadır. Örneğin ``ls`` komutu da bu fonksiyonları kullanmaktadır.

r, w Hakları
---------------------

Bir dizine ``r`` hakkının olması, o dizinin içeriğinin ``ls`` gibi bir komutla görüntülenebileceği anlamına
gelmektedir. (Aslında bu kontrol ``opendir`` POSIX fonksiyonunda yapılmaktadır.) Bir dizin içerisinde bir
dosyanın ya da dizinin yaratılması için dizine ``w`` hakkının olması gerekir. Çünkü dizin içerisinde dosya ya da
dizin yaratmak aslında dizin dosyasına yeni bir giriş eklemek (bunun bir yazma işlemi olduğuna dikkat ediniz)
anlamına gelmektedir.

Bir dizin içerisindeki bir dosyayı ya da dizini silmek için tek gereken şey, o dosya ya da dizinin içinde
bulunduğu dizine ``w`` hakkının olmasıdır. Silinecek dosya ya da dizine ``w`` hakkının olup olmadığının hiçbir
önemi yoktur. Ancak bazı kabuk programları, dizine ``w`` hakkı varsa ancak silinmek istenen dosya ya da dizine
``w`` hakkı yoksa bir uyarı mesajı da verebilmektedir.

x Hakkı ve Yol İfadesinin Çözümlenmesi
-------------------------------------------

Dizinlerde ``x`` hakkı farklı bir anlama gelmektedir. İşletim sistemi, bir yol ifadesi verildiğinde yol
ifadesinde hedeflenen dizin girişi için bilgileri elde etmek ister. Buna *yol ifadesinin çözümlenmesi (pathname
resolution)* denilmektedir. Örneğin:

.. code-block:: text

    "/home/kaan/Study/C/sample.c"

Burada hedeflenen dosya ``sample.c`` dosyasıdır. İşletim sistemi bu dosyanın yerini bulabilmek için yol
ifadesindeki bileşenlerin üzerinden geçmek ister. İşte *yol ifadesinin çözümlenmesi* işleminde dizin geçişleriyle
hedefe ulaşılabilmesi için prosesin, yol ifadesine ilişkin tüm dizinler için ``x`` hakkına sahip olması gerekir.
Yani dizinlerdeki ``x`` hakkı *içinden geçilebilirlik* gibi bir anlama gelmektedir. Biz bir dizindeki ``x``
hakkını kaldırırsak, işletim sistemi *yol ifadesinin çözümlenmesi* işleminde başarısız olur. Yukarıda da
belirttiğimiz gibi yol ifadesinin başarılı bir biçimde çözümlenmesi için, yol ifadesindeki dizin belirten tüm
yol bileşenleri için erişim yapan prosesin ``x`` hakkına sahip olması gerekir. Yukarıdaki örnekte *yol
ifadesinin çözümlenmesi* işleminin başarıyla bitirilebilmesi için prosesin ``home`` dizinine, ``kaan`` dizinine,
``Study`` dizinine ve ``C`` dizinine ``x`` hakkına sahip olması gerekir.

``x`` hakkı göreli yol ifadelerinde de aynı biçimde uygulanmaktadır. Örneğin biz ``test.txt`` dosyasını ``open``
fonksiyonu ile ``test.txt`` yol ifadesini vererek açmak isteyelim. Eğer içinde bulunduğumuz dizin için ``x``
hakkına sahip değilsek yine yol ifadesi başarılı bir biçimde çözümlenemeyecektir. Başka bir deyişle ``test.txt``
yol ifadesi sanki ``./test.txt`` gibi ele alınmaktadır. Örneğin ``a/b/c/test.txt`` gibi bir yol ifadesinin
başarılı bir biçimde çözülmesi için prosesin çalışma dizini de dahil olmak üzere ``a``, ``b`` ve ``c``
dizinlerine ``x`` hakkının olması gerekir.

``x`` hakkı dizin ağacında bir noktaya duvar örmek için kullanılabilmektedir. ``mkdir`` gibi kabuk komutları
dizin yaratırken zaten ``x`` hakkını varsayılan durumda vermektedir. Proses id'si ``0`` olan *root prosesler*
her zaman yol ifadesinin çözümlenmesi sırasında dizinlerin içerisinden geçebilirler.

Yol ifadesinin çözümlenmesi sırasında prosesin dizinlere ``r`` hakkının bulunması gerekmemektedir. Örneğin
``a/b/c/test.txt`` gibi bir yol ifadesinde, prosesin ``a`` dizinine, ``b`` dizinine ve ``c`` dizinine ``r``
hakkı olmasa bile ``test.txt`` dosyasına gerekli erişim izni varsa bu dosya açılabilir. Yani bir dizinin
içeriğini görüntüleyemediğimiz halde, eğer bir dosyanın o dizinin içerisinde bulunduğunu biliyorsak, o dosyayı
yine de kullanabiliriz.

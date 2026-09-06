
=============================================================
Standart C'deki Dosya Fonksiyonlarının Tamponlama Mekanizması
=============================================================

Standart C Dosya Fonksiyonlarında Tamponlama
============================================

Bu bölümde standart C'deki dosya fonksiyonlarının tamponlama mekanizması üzerinde duracağız.

C'nin prototipleri ``<stdio.h>`` içerisinde bulunan ve başı "f" ile başlayan dosya fonksiyonları aslında
birer *sarma fonksiyon (wrapper function)* gibidir. Biz bu fonksiyonları kullandığımızda arka planda bu
fonksiyonlar UNIX/Linux ve macOS sistemlerinde POSIX fonksiyonlarını, Windows sistemlerinde ise Windows
API fonksiyonlarını çağırmaktadır. Tabii bu fonksiyonlar da aslında ilgili sistemdeki sistem fonksiyonlarını
çağırarak işlemlerini yapmaktadır. Örneğin biz Linux sistemlerinde ``fopen`` fonksiyonunu kullanmış olalım:

.. code-block:: text

    fopen (kullanıcı modu)  --->  open (kullanıcı modu)  --->  sys_open (çekirdek modu)

``fopen`` fonksiyonu bize ``FILE *`` türünden bir *dosya bilgi göstericisi (stream)* vermektedir. Aslında
``FILE`` bir typedef ismidir ve bir yapıyı belirtmektedir:

.. code-block:: c

    typedef struct {

        /* ... */

    } FILE;

Peki bu yapının içerisinde hangi bilgiler vardır? Bir kere ``fopen`` dosyayı gerçekte UNIX/Linux
sistemlerinde ``open`` POSIX fonksiyonunu kullanarak açtığına göre bir biçimde onun içerisinde ``open``
fonksiyonundan elde edilen dosya betimleyicisi bulunacaktır:

.. code-block:: c

    typedef struct {
        /*... */
        int fd;
        /* ... */
    } FILE;

Standart dosya fonksiyonlarının en önemli özellikleri bir *cache sistemi* oluşturmalarıdır. Burada *cache*
terimi daha uygun olmasına karşın daha çok *tampon (buffer)* terimi kullanılmaktadır. Bu nedenle C'nin
dosya fonksiyonlarına *tamponlu (buffered) IO fonksiyonları* denilmektedir.

read1.c ve read2.c: Sistem Çağrılarının Maliyeti
------------------------------------------------

Aşağıda iki program verilmiştir. Bu iki program da bir dosyanın bütün karakterlerini ekrana yazdırmaktadır.
``read1.c`` programı bu işlemi her defasında ``read`` fonksiyonunu çağırarak yaparken ``read2.c`` programı
bir defasında 512 byte okuma yaparak okunanları bir tampona yerleştirip oradan alıp yazdırmaktadır.
Dolayısıyla ``read1.c`` programının daha hızlı çalışması beklenir. Çünkü bu program sistem
fonksiyonlarını daha az çağırmaktadır. Aşağıda kursun yapıldığı sanal makinede ``read1.c`` ve
``read2.c`` programlarının ``/usr/include/math.h`` gibi bir dosyanın yazdırılması işlemindeki çalışma
zamanları verilmiştir:

.. code-block:: console

    $ time ./read1 /usr/include/math.h
    ...
    real    0m0,032s
    user    0m0,006s
    sys     0m0,025s

    $ time ./read2 /usr/include/math.h
    ...
    real    0m0,007s
    user    0m0,001s
    sys     0m0,006s

Görüldüğü gibi küçük bir dosyada bile çalışma zamanı arasında önemli farklılıklar gözlemlenmektedir.

.. code-block:: c

    /* read1.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(int argc, char *argv[])
    {
        int fd;
        char ch;
        ssize_t result;

        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }

        if ((fd = open(argv[1], O_RDONLY)) == -1)
            exit_sys("open");

        while ((result = read(fd, &ch, 1)) > 0)
            putchar(ch);

        if (result == -1)
            exit_sys("read");

        putchar('\n');

        close(fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

.. code-block:: c

    /* b.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    #define BUFSIZE        512

    int main(int argc, char *argv[])
    {
        int fd;
        char buf[BUFSIZE];
        ssize_t result;

        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }

        if ((fd = open(argv[1], O_RDONLY)) == -1)
            exit_sys("open");

        while ((result = read(fd, buf, BUFSIZE)) > 0) {
            for (int i = 0; i < result; ++i)
                putchar(buf[i]);
        }

        if (result == -1)
            exit_sys("read");

        putchar('\n');

        close(fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Tamponlama Mekanizmasının Çalışma Mantığı
-----------------------------------------

İşte standart C fonksiyonları da yukarıdaki örnekte olduğu gibi sistem fonksiyonlarını daha az çağırmak
için bir tampon kullanmaktadır. Biz örneğin ``fgetc`` fonksiyonu ile bir byte bile okumak istesek ``fgetc``
bir tamponluk bilgiyi okur ve bize onun içerisinden bir byte'ı verir. Biz daha sonra yeniden ``fgetc``
fonksiyonunu çağırdığımızda ``fgetc`` zaten tamponda daha önce okunmuş olan bilgi yığını olduğu için
``read`` fonksiyonu ile okuma yapmaz, bize doğrudan byte'ı tampondan verir. Tabii tampondaki her byte
okunduktan sonra (yani tamponun sonuna gelindiğinde) ``fgetc`` yeniden ``read`` fonksiyonunu çağıracak ve
tamponu yeniden dolduracaktır.

``fopen`` fonksiyonunun geri döndürdüğü ``FILE`` türünden yapının içerisinde aslında bu tamponu yönetmek
için gerekli olan bilgiler de bulunmaktadır. Örneğin tamponun adresi, büyüklüğü, tamponda nerede kalındığı
gibi bilgiler bu ``FILE`` yapısının içerisinde tutulmaktadır.

Dosya işlemlerinde tamponlama C'ye özgü bir durum değildir. C++'taki iostream sınıfları, dosya işlemlerini
yapan Java ve C# sınıfları, Rust'taki yapılar benzer tamponlamayı yapmaktadır.

Aşağıda daha önce yapmış olduğumuz okuma zaman ölçümü standart C fonksiyonları kullanılarak
yapılmıştır.

.. code-block:: console

    $ time ./read3 /usr/include/math.h
    ...
    real    0m0,006s
    user    0m0,001s
    sys     0m0,005s

Görüldüğü gibi standart C fonksiyonları da bizim manuel yaptığımız tamponlamayı kendi içlerinde
yapmaktadır.

.. code-block:: c

    /* a.c */

    #include <stdio.h>
    #include <stdlib.h>

    int main(int argc, char *argv[])
    {
        FILE *f;
        int ch;

        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }

        if ((f = fopen(argv[1], "r")) == NULL) {
            fprintf(stderr, "cannot open file!..\n");
            exit(EXIT_FAILURE);
        }

        while ((ch = fgetc(f)) != EOF)
            putchar(ch);

        if (ferror(f)) {
            fprintf(stderr, "cannot read file!..\n");
            exit(EXIT_FAILURE);
        }

        putchar('\n');

        fclose(f);

        return 0;
    }

BUFSIZ ve Varsayılan Tampon Büyüklüğü
-------------------------------------

C'de dosya tamponları dosya açıldığında o dosyaya ilişkin olacak biçimde oluşturulmaktadır. Yani her
dosyanın tamponu birbirinden ayrıdır. Standart C fonksiyonlarının kullandıkları default tampon büyüklüğü
bilgisi ``<stdio.h>`` içerisinde ``BUFSIZ`` sembolik sabitiyle dışarıya verilmektedir. (Tabii bu ``BUFSIZ``
değerini değiştirmenin bir anlamı yoktur. Kod çoktan derlenmiştir. Bu sembolik sabit sadece dış dünyaya
default durum hakkında bilgi vermek için bulundurulmuştur.)

Aşağıdaki programda ``BUFSIZ`` değeri ekrana (``stdout`` dosyasına) yazdırılmıştır. Kursun yapıldığı
makinede bu değer 8192'dir. Tabii bu 8192 değeri *glibc* kütüphanesi tarafından belirlenmiş değerdir.

.. code-block:: c

    #include <stdio.h>

    int main(void)
    {
        printf("%d\n", BUFSIZ);        /* 8192 */

        return 0;
    }

Okuma/Yazma Tamponları ve flush İşlemi
--------------------------------------

Standart C fonksiyonlarının kullandığı bu tamponlar read/write tamponlardır. Yani yalnızca okuma sırasında
değil yazma sırasında da kullanılmaktadır. Örneğin biz ``fputc`` fonksiyonu ile bir byte'ı dosyaya yazmak
istesek bu bir byte aslında bu tampona yazılır. Tamponun içerisindekiler tampon dolduğunda, açıkça
``fflush`` fonksiyonu çağrıldığında ya da en kötü olasılıkla ``fclose`` işlemi sırasında ``write``
fonksiyonu çağrılarak diske yazılmaktadır. Biz tampondaki bilginin aktarılmasını garanti etmek için
``fflush`` fonksiyonu kullanabiliriz. Tampondaki bilginin diske yazılması işlemine dosya terminolojisinde
*flush işlemi* denilmektedir. ``fflush`` fonksiyonunun kullanılabilmesi için dosyanın yazma modunda
açılmış olması (yani *"w"*, *"r+"* gibi modlarda) gerekmektedir.

FILE Yapısının İçeriği
----------------------

Bildiğiniz gibi C'nin dosya açmakta kullanılan ``fopen`` fonksiyonu bize ``FILE`` türünden bir yapı
nesnesinin adresini vermektedir. Bu ``FILE`` nesnesine *stream* de denilmektedir. Biz kursumuzda buna
genel olarak *dosya bilgi göstericisi* diyoruz. İşte yukarıda da belirttiğimiz gibi bu ``FILE`` yapısının
içerisinde tamponu yönetmek için de bilgiler bulunmaktadır. ``FILE`` yapısının içerisinde tipik olarak şu
bilgiler bulunur:

- İşletim sistemi düzeyinde okuma/yazma işlemleri için gereken dosya betimleyicisi
- Tamponun başlangıç adresini tutan bir gösterici
- Tampondaki aktif noktayı tutan bir gösterici
- Tamponun uzunluğunu tutan bir eleman ya da tamponun sonunu tutan bir gösterici
- Diğer bilgiler

Örneğin kursun yapıldığı makinedeki *glibc* kütüphanesinde ``FILE`` yapısı önce ``struct _IO_FILE``
biçiminde tanımlanıp sonra ``FILE`` olarak typedef edilmiştir:

.. code-block:: c

    struct _IO_FILE
    {
        int _flags;             /* High-order word is _IO_MAGIC; rest is flags. */

        /* The following pointers correspond to the C++ streambuf protocol. */
        char *_IO_read_ptr;     /* Current read pointer */
        char *_IO_read_end;     /* End of get area. */
        char *_IO_read_base;    /* Start of putback+get area. */
        char *_IO_write_base;   /* Start of put area. */
        char *_IO_write_ptr;    /* Current put pointer. */
        char *_IO_write_end;    /* End of put area. */
        char *_IO_buf_base;     /* Start of reserve area. */
        char *_IO_buf_end;      /* End of reserve area. */

        /* The following fields are used to support backing up and undo. */
        char *_IO_save_base;    /* Pointer to start of non-current get area. */
        char *_IO_backup_base;  /* Pointer to first valid character of backup area */
        char *_IO_save_end;     /* Pointer to end of non-current get area. */

        struct _IO_marker *_markers;

        struct _IO_FILE *_chain;

        int _fileno;
        int _flags2;
        __off_t _old_offset;    /* This used to be _offset but it's too small.  */

        /* 1+column number of pbase(); 0 is unknown. */
        unsigned short _cur_column;
        signed char _vtable_offset;
        char _shortbuf[1];

        _IO_lock_t *_lock;
        #ifdef _IO_USE_OLD_IO_FILE
    };

    typedef struct _IO_FILE FILE;

*musl* isimli POSIX kütüphanesinde de benzer tanımlamalar kullanılmıştır:

.. code-block:: c

    struct _IO_FILE {
        unsigned flags;
        unsigned char *rpos, *rend;
        int (*close)(FILE *);
        unsigned char *wend, *wpos;
        unsigned char *mustbezero_1;
        unsigned char *wbase;
        size_t (*read)(FILE *, unsigned char *, size_t);
        size_t (*write)(FILE *, const unsigned char *, size_t);
        off_t (*seek)(FILE *, off_t, int);
        unsigned char *buf;
        size_t buf_size;
        FILE *prev, *next;
        int fd;
        int pipe_pid;
        long lockcount;
        int mode;
        volatile int lock;
        int lbf;
        void *cookie;
        off_t off;
        char *getln_buf;
        void *mustbezero_2;
        unsigned char *shend;
        off_t shlim, shcnt;
        FILE *prev_locked, *next_locked;
        struct __locale_struct *locale;
    };

    typedef struct _IO_FILE FILE;

Peki ``fopen`` tarafından bu ``FILE`` yapısı nasıl tahsis edilmektedir? Standart C kütüphanelerini
yazanlar birkaç teknik kullanabilmektedir. Birincisi doğrudan tahsisatın ``malloc`` fonksiyonu ile
yapılmasıdır. Tabii bu durumda ``free`` işlemi ``fclose`` fonksiyonu tarafından yapılacaktır. İkincisi bu
``FILE`` yapısı zaten işin başında static düzeyde tahsis edilmiş bir ``FILE`` dizisinin içerisinde
alınabilir. Örneğin:

.. code-block:: c

    ...
    static FILE g_files[FILE_MAX];
    ...

Örneğin *musl* kütüphanesinde ``FILE`` nesnesi ve onun kullandığı tampon tek hamlede ``malloc`` fonksiyonu
ile tahsis edilmiştir:

.. code-block:: c

    if (!(f=malloc(sizeof *f + UNGET + BUFSIZ))) return 0;

*uclibc (mikro C kütüphanesinde)* de tahsisat ``malloc`` fonksiyonuyla yapılmıştır:

.. code-block:: c

    if ((stream = malloc(sizeof(FILE))) == NULL) {
        return stream;
    }

C standartlarında ``FILE`` yapısının içeriği hakkında bir bilgi verilmemiştir. Bu durumda bu ``FILE``
yapısının içeriği kütüphaneyi yazanlar tarafından istenildiği gibi oluşturulabilir.


fileno ve fdopen Fonksiyonları
==============================

fileno Fonksiyonu
-----------------

Biz bir dosyayı ``fopen`` fonksiyonuyla açıp o dosyanın dosya betimleyicisini elde edebiliriz. ``fileno``
isimli POSIX fonksiyonu ``FILE`` yapısının içerisindeki dosya betimleyicisini bize vermektedir. ``fileno``
fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <stdio.h>

    int fileno(FILE *stream);

Fonksiyonun geri dönüş değeri dosya betimleyicisidir. Peki bu fonksiyon başarısız olabilir mi ya da
başarısızlığı tespit edebilir mi? POSIX standartlarına göre fonksiyon başarısız olabilir. Bu durumda -1
değerine geri döner. Ancak fonksiyonun başarısızlığı tespit etmesi yeterli bir biçimde yapılamayabilir.
Fonksiyon ``FILE`` yapısının içerisindeki dosya betimleyicisini tutan elemana başlangıçta geçersiz bir
değer atayıp bu değere bakmaktadır. Örneğin:

.. code-block:: c

    FILE *f;
    int fd;
    /* ... */

    if ((f = fopen("test.txt", "r")) == NULL) {
        fprintf(stderr, "cannot open file!...\n");
        exit(EXIT_FAILURE);
    }

    if ((fd = fileno(f)) == -1)
        exit_sys("fileno");

Tabii ``fileno`` fonksiyonuyla ``FILE`` yapısı içerisindeki dosya betimleyicisini alıp onunla dosya işlem
yaptığımızda dosya göstericisinin değeri de değişmiş olacaktır.

``fileno`` bir standart C fonksiyonu değildir, bir POSIX fonksiyonudur. Microsoft Windows sistemlerinde de
bu fonksiyonu ``_fileno`` ismiyle bulundurmaktadır.

Aşağıdaki örnekte dosya önce ``fopen`` fonksiyonuyla açılıp ``fileno`` fonksiyonuyla dosya betimleyicisi
elde edilmiş ve sonra o betimleyici ile okuma yapılmıştır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        FILE *f;
        int fd;
        char buf[10 + 1];
        ssize_t result;

        if ((f = fopen("test.txt", "r")) == NULL) {
            fprintf(stderr, "cannot open file!...\n");
            exit(EXIT_FAILURE);
        }

        if ((fd = fileno(f)) == -1)
            exit_sys("fileno");

        if ((result = read(fd, buf, 10)) == -1)
            exit_sys("read");

        buf[result] = '\0';
        puts(buf);

        fclose(f);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

fdopen Fonksiyonu
-----------------

``fileno`` POSIX fonksiyonunun mantıksal olarak tersini yapan ``fdopen`` isimli bir POSIX fonksiyonu da
vardır. (``fdopen`` da bir standart C fonksiyonu değildir.) Bu fonksiyon ``open`` POSIX fonksiyonuyla açıp
betimleyicisini elde ettiğimiz dosyaya ilişkin dosya bilgi göstericisini (``FILE *``) bize verir. Yani
``fdopen`` sanki o dosyayı ``fopen`` ile açmışız gibi bir durum oluşturmaktadır. ``fdopen`` fonksiyonunun
prototipi şöyledir:

.. code-block:: c

    #include <stdio.h>

    FILE *fdopen(int fd, const char *mode);

Fonksiyonun birinci parametresi ``open`` fonksiyonu ile elde edilen dosya betimleyicidir. İkinci parametre
dosyanın ``fopen`` fonksiyonundaki açış modudur. Tabii buradaki açış modunun ``open`` fonksiyonuyla dosya
açılırkenki mod ile uyuşması gerekir. Fonksiyon başarı durumunda dosya bilgi göstericisine, başarısızlık
durumunda ``NULL`` adrese geri döner. ``errno`` değeri uygun biçimde set edilir. Örneğin:

.. code-block:: c

    int fd;
    FILE *f;
    /* ... */

    if ((fd = open("test.txt", O_RDONLY)) == -1)
        exit_sys("open");

    if ((f = fdopen(fd, "r")) == NULL)
        exit_sys("fdopen");

``fdopen`` fonksiyonundaki açış modunun betimleyici oluşturulurken belirtilen açış moduyla uyumlu olması
gerektiğine bir kez daha dikkatinizi çekmek istiyoruz. Yukarıdaki örnekte biz ``fdopen`` fonksiyonunda açış
modunu *r+* biçiminde verseydik ``fdopen`` başarısız olurdu.

Aşağıdaki örnekte önce ``open`` POSIX fonksiyonu ile dosya açılmış sonra dosya betimleyicisi kullanılarak
``fdopen`` fonksiyonu ile dosya bilgi göstericisi elde edilmiştir. İşlemlere standart C fonksiyonlarıyla
devam edilmiştir. ``fclose`` işlemi zaten bu betimleyiciyi kapatacağı için ayrıca ``close`` fonksiyonu
çağrılmamıştır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;
        FILE *f;
        int ch;

        if ((fd = open("test.txt", O_RDONLY)) == -1)
            exit_sys("open");

        if ((f = fdopen(fd, "r")) == NULL)
            exit_sys("fdopen");

        while ((ch = fgetc(f)) != EOF)
            putchar(ch);

        if (ferror(f)) {
            fprintf(stderr, "cannot read file!..\n");
            exit(EXIT_FAILURE);
        }

        fclose(f);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Standart C Dosya Fonksiyonlarında Tamponlama Modları
====================================================

Şimdi de C'nin standart dosya fonksiyonlarının uyguladığı tamponlama (buffering) hakkında bilgiler
verelim. Standart C'nin ``<stdio.h>`` fonksiyonları tamponlamayı üç moda (ya da stratejiye) göre farklı
biçimlerde yapmaktadır.

Üç Tamponlama Modu
------------------

**Tam Tamponlamalı (Full Buffered) Mod:** Burada okuma sırasında tampon tamamen doldurulur. Tamponun
sonuna gelindiğinde tampon yeniden doldurulur. Yazma sırasında da tampona yazılır. Tamponun sonuna
gelindiğinde ya da ``fseek`` işlemi yapıldığında tampona yazılmış olanlar flush edilir.

**Satır Tamponlamalı (Line Buffered) Mod:** Bu modda tampon tamamen doldurulmaz. Yalnızca tek satırlık
bilgi ('\\n' karakteri dahil olmak üzere) tampona çekilmektedir. Okuma sırasında bu tampondan byte'lar
verilir. Dosyaya yazılmak istenen byte'lar yine tampona yazılır. flush işlemi '\\n' karakteri tampona
yazılınca (ya da ``fflush`` ve ``fclose`` fonksiyonları çağrılınca) yapılmaktadır. Satır tamponlamalı mod
tipik olarak text dosyalar için kullanılmaktadır. Binary dosyalar için bu mod kullanılabilse de
anlamsızdır.

**Sıfır Tamponlamalı (Unbuffered) Mod:** Burada tampon hiç kullanılmaz. Doğrudan ilgili aşağı seviyeli
fonksiyonlarla (yani UNIX/Linux sistemlerinde ``read`` ve ``write`` POSIX fonksiyonlarıyla) aktarım
yapılır.

.. code-block:: text

    +------------------------+  +-------------------------+  +---------------------------+
    |  Tam Tamponlamalı      |  |  Satır Tamponlamalı     |  |  Sıfır Tamponlamalı       |
    |  (Full Buffered)       |  |  (Line Buffered)        |  |  (Unbuffered)             |
    |------------------------|  |-------------------------|  |---------------------------|
    |  Tampon tamamen        |  |  Yalnızca bir satır     |  |  Tampon kullanılmaz,      |
    |  doldurulunca flush    |  |  ('\n' dahil) tutulur,  |  |  her çağrı doğrudan       |
    |  edilir.               |  |  '\n' görülünce flush   |  |  read/write ile yapılır.  |
    |                        |  |  edilir.                |  |                           |
    +------------------------+  +-------------------------+  +---------------------------+

Satır tamponlaması kişilere biraz tuhaf gelebilmektedir. Çünkü satır tamponlaması yapabilmek için standart
C kütüphanesinin okuma sırasında '\\n' karakterini görmesi gerekir ki bazı durumlarda bunun etkin bir
biçimde yapılabilme olanağı yoktur. Ancak bazı durumlarda zaten aygıt sürücüler bize satırsal bilgi
vermektedir. Standart C kütüphaneleri disk dosyaları için satır tamponlaması yaparken aslında çoğu kez
'\\n' karakterine kadar değil tüm tampon kadar okuma yapmaktadır. Satır tamponlamasının en önemli etkisi
yazma işleminde kendini göstermektedir. Satır tamponlamalı modda '\\n' karakteri tampona yazıldığında flush
işlemi yapılmaktadır. C standartları bu üç tamponlama biçimini kabaca betimlemiş olsa da ayrıntılar
konusunda bir açıklama yapmamıştır. Dolayısıyla kütüphaneleri gerçekleştirenler satır tamponlaması ile
okuma yapılırken '\\n' karakterine kadar değil tüm tamponu da doldurabilmektedir. C standartlarında
tamponlama stratejisi için yalnızca kabaca *niyet* belirtilmiştir. Yukarıda da belirttiğimiz gibi ayrıntılı
bir açıklama yapılmamıştır. Standartlar ayrıntıların *derleyicileri yazanların isteğine bırakıldığını
(implementation-defined)* belirtmektedir.

C Standardının Tamponlama Tanımı
--------------------------------

C standartlarında tamponlamayla ilgili kısım şöyledir:

    When a stream is unbuffered, characters are intended to appear from the source or at the
    destination as soon as possible. Otherwise characters may be accumulated and
    transmitted to or from the host environment as a block. When a stream is fully buffered,
    characters are intended to be transmitted to or from the host environment as a block when
    a buffer is filled. When a stream is line buffered, characters are intended to be
    transmitted to or from the host environment as a block when a new-line character is
    encountered. Furthermore, characters are intended to be transmitted as a block to the host
    environment when a buffer is filled, when input is requested on an unbuffered stream, or
    when input is requested on a line buffered stream that requires the transmission of
    characters from the host environment. Support for these characteristics is
    implementation-defined, and may be affected via the setbuf and setvbuf functions.

    -- ISO/IEC C Standardı

Tamponlama modu ile ilgili iki önemli soru gündeme gelmektedir:

1. ``fopen`` fonksiyonu ile bir dosya açıldığında dosyanın default tamponlama modu nedir?
2. Dosyanın tamponlama modu nasıl değiştirilmektedir?

``fopen`` fonksiyonu ile dosya açıldığında dosyanın default tamponlama modu hakkında C standartlarında bir
şey söylenmemiştir. Bu durum *bunun herhangi bir biçimde olabileceği* anlamına gelmektedir. Fakat mevcut
standart C kütüphanelerinin hepsi disk dosyalarında default durumda *tam tamponlamalı (full buffered)*
modu esas almaktadır. Ancak C standartlarında ``stdin``, ``stdout`` ve ``stderr`` dosyalarının default
tamponlama modu için bazı şeyler söylenmiştir. Bir dosyanın tamponlama modu, dosya ``fopen`` fonksiyonuyla
açıldıktan sonra ancak henüz hiçbir işlem yapmadan ``setbuf`` ve ``setvbuf`` standart C fonksiyonlarıyla
değiştirilebilmektedir. Dosya üzerinde herhangi bir işlem yaptıktan sonra bu fonksiyonların çağrılması
*tanımsız davranışa (undefined behavior)* yol açmaktadır. ``setvbuf`` fonksiyonu işlevsel olarak
``setbuf`` fonksiyonunu zaten kapsamaktadır.

Tamponlama Modunun Değiştirilmesi: setbuf
-----------------------------------------

``setbuf`` fonksiyonu aslında kullanılan tamponun yerini değiştirmek için tasarlanmıştır. Fonksiyonun
prototipi şöyledir:

.. code-block:: c

    #include <stdio.h>

    void setbuf(FILE *stream, char *buf);

Fonksiyonun birinci parametresi dosya bilgi göstericisini, ikinci parametresi yeni tamponun yerini
belirtmektedir. Bu tamponun ``BUFSIZ`` uzunluğunda olması gerekir. Eğer ikinci parametre ``NULL`` adres
olarak girilirse bu durumda dosya sıfır tamponlamalı moda sokulmaktadır. Fonksiyon başarıyı kontrol
edememektedir. Örneğin:

.. code-block:: c

    FILE *f;
    char mybuf[BUFSIZ];
    /* ... */

    if ((f = fopen("test.txt", "r+")) == NULL) {
        fprintf(stderr, "cannot open file!...\n");
        exit(EXIT_FAILURE);
    }
    setbuf(f, mybuf);

Burada artık açılan dosya için ``mybuf`` ile belirtilen tampon kullanılmaktadır. Biz dosyayı sıfır
tamponlamalı moda şöyle geçirebiliriz:

.. code-block:: c

    setbuf(f, NULL);

Aslında tamponun yerini değiştirmenin gerektiği durumlar oldukça seyrektir. Tampona doğrudan erişilmek
istendiğinde, ya da tamponun heap'te değil de statik bir alanda oluşturulması istendiğinde bu değişiklik
yapılabilmektedir. Bazı standart C kütüphaneleri (örneğin *musl* ve *uclibc*) dosya tamponunu ``fopen``
işlemi sırasında tahsis etmektedir. *glibc* gibi bazı kütüphaneler ise tamponu *ilk kez kullanıldığında*
tahsis etmektedir.

Aşağıdaki örnekte ``setbuf`` fonksiyonu ile dosya için kullanılacak tamponun yeri değiştirilmiştir.
``fgetc`` işlemi sonrasında bu tamponun doldurulduğunu gözlemleyebilirsiniz.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>

    int main(void)
    {
        FILE *f;
        char mybuf[BUFSIZ];
        long size;
        int n;
        int ch;

        if ((f = fopen("test.txt", "r+")) == NULL) {
            fprintf(stderr, "cannot open file!...\n");
            exit(EXIT_FAILURE);
        }

        fseek(f, 0, SEEK_END);
        size = ftell(f);
        fseek(f, 0, SEEK_SET);
        n = size < 512 ? size : 512;

        setbuf(f, mybuf);

        ch = fgetc(f);
        putchar(ch);

        for (int i = 0; i < n; ++i)
            putchar(mybuf[i]);
        putchar('\n');

        fclose(f);

        return 0;
    }

Tamponlama Modunun Değiştirilmesi: setvbuf
------------------------------------------

``setvbuf`` fonksiyonu ile hem tamponun yeri, hem büyüklüğü hem de tamponlama modu değiştirilebilmektedir.
Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <stdio.h>

    int setvbuf(FILE *stream, char *buf, int mode, size_t size);

Fonksiyonun birinci parametresi dosya bilgi göstericisini (stream) belirtir. Üçüncü parametre
değiştirilecek tamponlama modunu belirtmektedir. Bu parametre şu değerlerden birini alabilmektedir:

- ``_IONBF`` (unbuffered)
- ``_IOLBF`` (line buffered)
- ``_IOFBF`` (fully buffered)

İkinci parametre tamponun yerini değiştirmek için kullanılmaktadır. Bu parametre ``NULL`` adres geçilirse
tamponun yeri değiştirilmez. Son parametre ise tamponun yeni uzunluğunu belirtmektedir. Programcı ikinci
parametreye ``NULL`` adres geçip son parametre yoluyla tamponun büyüklüğünü de değiştirebilir. Bu durumda
tamponu ``setvbuf`` kendisi tahsis edecektir. Eğer tamponlama modu ikinci parametreye ``_IONBF`` geçilerek
sıfır tamponlamalı mod olarak ayarlanırsa artık ikinci ve dördüncü parametrenin bir önemi kalmamaktadır.
Fonksiyon başarı durumunda 0 değerine, başarısızlık durumunda sıfır dışı bir değere geri dönmektedir.
POSIX sistemlerinde ``errno`` değeri yine uygun biçimde set edilmektedir. Örneğin:

.. code-block:: c

    FILE *f;
    /* ... */

    if ((f = fopen("test.txt", "r+")) == NULL) {
        fprintf(stderr, "cannot open file!...\n");
        exit(EXIT_FAILURE);
    }

    if (setvbuf(f, NULL, _IONBF, 0) == -1) {
        fprintf(stderr, "setvbuf failed!..\n");
        exit(EXIT_FAILURE);
    }

Burada dosya sıfır tamponlamalı moda sokulmuştur. Örneğin:

.. code-block:: c

    FILE *f;
    char mybuf[1024];
    /* ... */

    if ((f = fopen("test.txt", "r+")) == NULL) {
        fprintf(stderr, "cannot open file!...\n");
        exit(EXIT_FAILURE);
    }

    if (setvbuf(f, mybuf, _IONBF, 1024) == -1) {
        fprintf(stderr, "setvbuf failed!..\n");
        exit(EXIT_FAILURE);
    }

Burada tamponun yeri ve uzunluğu değiştirilmiştir.

glibc'ye Özgü setbuffer ve setlinebuf Fonksiyonları
---------------------------------------------------

*glibc* kütüphanesinde ``setbuffer`` ve ``setlinebuf`` isimli iki fonksiyon da bulunmaktadır. Bu
fonksiyonlar C standartlarında ve POSIX standartlarında bulunmamaktadır. *glibc* kütüphanesine özgüdür.
Bu fonksiyonların prototipleri şöyledir:

.. code-block:: c

    #include <stdio.h>

    void setbuffer(FILE *stream, char *buf, size_t size);
    void setlinebuf(FILE *stream);

``setbuffer`` fonksiyonu ``setbuf`` fonksiyonunun tampon büyüklüğünün de belirlenebildiği biçimidir.
``setlinebuf(stream)`` çağrısı ise aşağıdakiyle eşdeğerdir:

.. code-block:: c

    setvbuf(stream, NULL, _IOLBF, 0);

Aşağıdaki örnekte bir dosya ``fopen`` fonksiyonuyla açılmış ve *satır tamponlamalı moda* geçirilmiştir.
Yukarıda da belirttiğimiz gibi C standartları tamponlama modları için mutlak uyulması gereken kuralları
açıkça belirtmemiştir. Örneğin *glibc* kütüphanesi normal dosyalarda satır tamponlaması sırasında satır
sonuna kadar değil tamponun tamamını doldurmaktadır. Ancak '\\n' karakteri dosyaya yazıldığında flush
işlemini yapmaktadır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>

    int main(void)
    {
        FILE *f;
        char mybuf[512];
        long size;
        int n;
        int ch;

        if ((f = fopen("test.txt", "r+")) == NULL) {
            fprintf(stderr, "cannot open file!...\n");
            exit(EXIT_FAILURE);
        }

        fseek(f, 0, SEEK_END);
        size = ftell(f);
        fseek(f, 0, SEEK_SET);
        n = size < 512 ? size : 512;

        if (setvbuf(f, mybuf, _IOLBF, 512) == -1) {
            fprintf(stderr, "setvbuf failed!..\n");
            exit(EXIT_FAILURE);
        }

        ch = fgetc(f);
        putchar(ch);

        for (int i = 0; i < n; ++i)
            putchar(mybuf[i]);
        putchar('\n');

        fclose(f);

        return 0;
    }

Standart C Kütüphanesi Gerçekleştirimleri İçin Kaynaklar
--------------------------------------------------------

*glibc* dışındaki çeşitli standart C kütüphanelerinin özellikle stdio fonksiyonlarının gerçekleştirimini
inceleyebilirsiniz. İncelemek için alternatifler şunlar olabilir:

- uclibc (Mikro C kütüphanesi): ``https://elixir.bootlin.com/uclibc-ng/latest/source``
- musl libc kütüphanesi: ``http://www.musl-libc.org/``
- diet libc kütüphanesi: ``http://www.fefe.de/dietlibc/``
- Plauger'in *The C Standard Library* kitabında gerçekleştirimini yaptığı kütüphane:
  ``https://github.com/topics/c-standard-library``

stdin, stdout, stderr ve Varsayılan Tamponlama
==============================================

stdin/stdout/stderr Akışlarının Genel Özellikleri
-------------------------------------------------

Daha önce de belirttiğimiz gibi C'nin ``<stdio.h>`` dosyası içerisinde ``FILE *`` türünden yani *stream*
belirten üç değişken ismi bulunmaktadır: ``stdin``, ``stdout`` ve ``stderr``. Bu değişkenler ``fopen``
fonksiyonunun geri döndürdüğü ``FILE`` nesnesi türünden adres belirtmektedir. Dolayısıyla C'nin standart
dosya fonksiyonlarında bunları kullanabiliriz. Örneğin aslında:

.. code-block:: c

    printf(...);

çağrısı ile aşağıdaki ``fprintf`` çağrısının bir farkı yoktur:

.. code-block:: c

    fprintf(stdout, ...);

Zaten örneğin C standartlarında ``printf`` için ayrıntılı açıklama yapılmamış, bu fonksiyonun *fprintf
fonksiyonunu stdout dosyasına yazan biçimi olduğu* söylenmiştir. Asıl açıklama ``fprintf`` fonksiyonunda
yapılmıştır.

``stdin``, ``stdout`` ve ``stderr`` dosya bilgi göstericileri (streams) programcı tarafından açılmamıştır
ve programcı tarafından kapatılmamalıdır. Programcı bunları doğrudan kullanabilir. Şüphesiz UNIX/Linux
sistemlerinde ``stdin`` dosya bilgi göstericisinin gösterdiği ``FILE`` nesnesinin içerisinde 0 numaralı
betimleyici, ``stdout`` ``FILE`` nesnesinin içerisinde 1 numaralı betimleyici ve ``stderr`` ``FILE``
nesnesinin içerisinde 2 numaralı betimleyici vardır.

Varsayılan Tamponlama Kuralları
-------------------------------

``stdin``, ``stdout`` ve ``stderr`` dosyaları için de ``FILE`` nesneleri, dolayısıyla tampon
oluşturulmaktadır. Yani bu dosyalar da tamponlu bir biçimde işleme sokulmaktadır. C standartları herhangi
bir dosyanın default tamponlaması hakkında bir şey söylememiş olsa da ``stdin``, ``stdout`` ve ``stderr``
dosyalarının default tamponlaması hakkında şunları söylemiştir:

- ``stdin`` ve ``stdout`` dosyaları default durumda *eğer interaktif olmayan bir aygıta yönlendirilmişse
  işin başında tam tamponlamalı* moddadırlar. Ancak bu dosyalar *interaktif olan bir aygıta
  yönlendirilmişse işin başında tam tamponlamalı olamazlar, satır tamponlamalı ya da sıfır tamponlamalı*
  olabilirler. Klavye ve ekran yani terminal *interaktif aygıt* kabul edilmektedir. Ancak disk dosyaları
  interaktif aygıt kabul edilmemektedir.

- ``stderr`` dosyası ister interaktif olmayan aygıta yönlendirilmiş olsun isterse interaktif aygıta
  yönlendirilmiş olsun işin başında tam tamponlamalı olamaz. Ancak satır tamponlamalı ya da sıfır
  tamponlamalı olabilir.

Windows ve UNIX/Linux Arasındaki Farklar, flush Garantisi
---------------------------------------------------------

Örneğin Windows sistemlerindeki C derleyicilerinde default durumda dosyaya yönlendirme yapılmamışsa
``stdout`` sıfır tamponlamalı, ``stdin`` satır tamponlamalıdır. Ancak UNIX/Linux sistemlerinde ``stdout``
ve ``stdin`` dosyaları default durumda satır tamponlamalıdır. Aşağıdaki örnekte bu durum anlaşılabilir.

.. code-block:: c

    #include <stdio.h>

    int main(void)
    {
        printf("ankara");        /* Windows sistemlerinde yazı gözükecek, UNIX/Linux'ta gözükmeyecek */

        for (;;)
            ;

        return 0;
    }

Bu örnekte eğer ``stdout`` default durumda satır tamponlamalı ise "ankara" yazısı önce tampona
aktarılacak, '\\n' basılana kadar tamponda kalacaktır. Tabii program sonlanırken ``stdin``, ``stdout`` ve
``stderr`` dosyaları zaten derleyiciler tarafından kapatılacağı için her durumda bu flush işlemi
yapılacaktır. Örneğin aşağıdaki programda programın çalışması bitince her sistemde yazı görünecektir:

.. code-block:: c

    #include <stdio.h>

    int main(void)
    {
        printf("ankara");

        return 0;
    }

Peki ``stdout`` dosyası terminale yönlendirilmişken satır tamponlamalı ya da sıfır tamponlamalı modda
olabiliyorsa bir yazının ekrana çıkmasını nasıl garanti edebiliriz? Mademki ``stdout`` terminale
yönlendirildiğinde en kötü olasılıkla satır tamponlamalı olabilir. O zaman yazının sonuna '\\n' karakteri
koyarız. Örneğin:

.. code-block:: c

    #include <stdio.h>

    int main(void)
    {
        printf("ankara\n");        /* Hem Windows'ta hem de Linux sistemlerinde yazı görülecek */

        for (;;)
            ;

        return 0;
    }

Ancak burada imleç aynı zamanda aşağı satıra da geçirilmektedir. Peki imleç aşağı satıra geçirilmeden
yazının ekrana çıkması nasıl garanti edilebilir? Bunun iki yolu vardır. Birincisi ``stdout`` dosyasını
``fflush(stdout)`` çağrısıyla flush etmektir:

.. code-block:: c

    #include <stdio.h>

    int main(void)
    {
        printf("ankara");
        fflush(stdout);

        for (;;)
            ;

        return 0;
    }

İkincisi ise ``stdout`` dosyasını her ihtimale karşı açıkça sıfır tamponlamalı moda çekmektir:

.. code-block:: c

    #include <stdio.h>

    int main(void)
    {
        setvbuf(stdout, NULL, _IONBF, 0);        /* setbuf(stdout, NULL) */
        printf("ankara");

        for (;;)
            ;

        return 0;
    }

C derleyicilerinin çoğunda ``stdin`` dosyasından okuma yapıldığında okuma yapan fonksiyonlar önce
``stdout`` dosyasını flush etmektedir. Standartlarda bu durum garanti edilmemiştir. Örneğin:

.. code-block:: c

    #include <stdio.h>

    int main(void)
    {
        printf("ankara");

        getchar();        /* Hem Windows hem de UNIX/Linux sistemlerindeki derleyicilerde stdout flush edilecek */

        return 0;
    }

Mademki bu davranış standartlarda garanti edilmemiş, o halde yine en doğru uygulama yazının sonunda '\\n'
yoksa açıkça ``fflush(stdout)`` çağrısını yapmaktır.

stdin Tamponunun Temizlenmesi (clear_stdin)
-------------------------------------------

``stdin`` dosyası hem Windows hem de UNIX/Linux sistemlerinde dosyaya yönlendirilmemişse satır
tamponlamalı moddadır. Dolayısıyla biz klavyeden bir karakter bile okumak istesek UNIX/Linux
sistemlerinde ``read`` fonksiyonu 0 numaralı betimleyici ile çağrılıp bir satırlık bilgi okunacak ve bu
bir satırlık bilgi sonunda '\\n' olacak biçimde tampona yerleştirilecektir. Artık tamponda bilgi olduğu
sürece okuma fonksiyonları tampondakileri okuyacaktır. Tamponda bir karakter kalmadığında yeniden
klavyeden bir satırlık okuma yapılıp tampona yerleştirilecektir. Örneğin üst üste iki ``getchar`` çağrısı
ile iki karakteri ``stdin`` dosyasından okumak isteyelim:

.. code-block:: c

    ch1 = getchar();
    ch2 = getchar();
    ch3 = getchar();

Birinci ``getchar`` çağrısı bizden bir satır alarak onu ``stdin`` dosyasının tamponuna yerleştirir. Tabii
tamponun sonunda '\\n' karakteri de bulunacaktır. İkinci ``getchar`` çağrısı tampon boş olmadığı için
klavyeden giriş istemeyip tampondan girişi karşılayacaktır. Yukarıdaki örnekte biz ilk ``getchar``
fonksiyonunda klavyeden *a* karakterine basıp ENTER tuşuna basmış olalım. Bu durumda tamponda şu karakter
olacaktır:

.. code-block:: text

    a\n

İlk ``getchar`` bu 'a' karakterini, ikinci ``getchar`` ise '\\n' karakterini alacaktır. Üçüncü ``getchar``
çağrısında artık tampon boş olduğu için yeni bir satır istenecektir. Yani ``stdin`` dosyasından okuma
yapan fonksiyonlar tampon boşsa ``read`` fonksiyonunu çağırarak bizden bir satırlık bilgi istemektedir.
Aşağıdaki programla test işlemini yapabilirsiniz:

.. code-block:: c

    #include <stdio.h>

    int main(void)
    {
        int ch;

        ch = getchar();
        printf("%c (%d)\n", ch, ch);

        ch = getchar();
        printf("%c (%d)\n", ch, ch);

        ch = getchar();
        printf("%c (%d)\n", ch, ch);

        return 0;
    }

Tabii ``scanf``, ``getchar``, ``gets`` gibi fonksiyonların hepsi ortak tampondan çalışmaktadır. Yani bu
fonksiyonların hepsi ``stdin`` dosyasından okuma yapar. ``stdin`` dosyasının da toplamda bir tane tamponu
vardır.

Peki biz gerçekten ikinci ``getchar`` fonksiyonu ile yeni bir klavye girişi yapmak istiyorsak bunu nasıl
sağlayabiliriz? ``stdin`` dosyasının flush edilmesi geçersiz bir işlemdir. Zira C'de salt okunur (read-only)
dosyalar flush edilemezler. Bunun için özel bir fonksiyon da bulundurulmamıştır. O zaman tek yapılacak şey
'\\n' karakterini görene kadar ``stdin`` dosyasından karakter karakter okuma yapmaktır. Bu işlem şöyle bir
döngü ile yapılabilir:

.. code-block:: c

    while (getchar() != '\n')
        ;

Tabii sonraki paragraflarda görüleceği üzere ``EOF`` durumunun da kontrol edilmesi daha uygun olur. Bu
nedenle aşağıdaki gibi bir fonksiyon bu iş için kullanılabilir:

.. code-block:: c

    void clear_stdin(void)
    {
        int ch;

        while ((ch = getchar()) != '\n' && ch != EOF)
            ;
    }

Maalesef bu işlemin C'de daha pratik bir yolu yoktur. Örneğin:

.. code-block:: c

    #include <stdio.h>

    void clear_stdin(void)
    {
        int ch;

        while ((ch = getchar()) != '\n' && ch != EOF)
            ;
    }

    int main(void)
    {
        int ch;

        ch = getchar();
        printf("%c (%d)\n", ch, ch);

        clear_stdin();

        ch = getchar();
        printf("%c (%d)\n", ch, ch);

        return 0;
    }


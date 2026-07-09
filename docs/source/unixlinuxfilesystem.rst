
=========================
UNIX/Linux Dosya Sistemi
=========================

Dosya işlemleri UNIX/Linux sistemlerinde en önemli konulardan biridir. Çünkü bu sistemlerde pek çok olgu bir
dosya gibi ele alınmaktadır. UNIX/Linux sistemlerinde sistem programlama için ilk öğrenilecek temel konu dosya
işlemleri olmalıdır. Bu bölümde UNIX/Linux sistemlerindeki dosya işlemleri üzerinde duracağız. UNIX/Linux
sistemlerinde dosya sistemi işletim sisteminin kalbi gibidir.

İşletim sistemlerinin dosya işlemleriyle ilgili alt sistemlerine *dosya sistemi (file system)* denilmektedir.
Dosya sisteminin iki yönü vardır: Disk ve Bellek. İşletim sistemi, dosyaların kalıcı olarak diskte saklanması
için diski bölümlere ayırır ve belli biçimlerde organize eder. Ancak bir dosya açıldığında işletim sistemi
çekirdek alanı içerisinde o dosyayı yönetebilmek için bazı veri yapıları da oluşturmaktadır.

Pek çok POSIX uyumlu işletim sistemi dosya işlemleri için 5 sistem fonksiyonu bulundurmaktadır:

- Dosyayı açmak için gereken sistem fonksiyonu (Linux'ta ``sys_open``)
- Dosyayı kapatmak için gereken sistem fonksiyonu (Linux'ta ``sys_close``)
- Dosyadan okuma yapmak için gereken sistem fonksiyonu (Linux'ta ``sys_read``)
- Dosyaya yazma yapmak için gereken sistem fonksiyonu (Linux'ta ``sys_write``)
- Dosya göstericisini konumlandırmak için gereken sistem fonksiyonu (Linux'ta ``sys_lseek``)

Bu 5 sistem fonksiyonunu çağıran 5 POSIX fonksiyonu bulunmaktadır: ``open``, ``close``, ``read``, ``write`` ve
``lseek``. Dosya işlemleri temelde bu 5 POSIX fonksiyonuyla yapılmaktadır.

Biz bir UNIX/Linux sisteminde hangi düzeyde çalışıyor olursak olalım, eninde sonunda dosya işlemleri bu 5 POSIX
fonksiyonu çağrılarak gerçekleştirilmektedir. Bu POSIX fonksiyonları da yukarıda belirttiğimiz gibi işletim
sisteminin ilgili sistem fonksiyonlarını çağırmaktadır. Programlama dili ne olursa olsun, bu sistemlerde tüm
dosya işlemleri eninde sonunda bu temel POSIX fonksiyonları çağrılarak yapılmaktadır.

----

Dosya Nesnesi (File Object)
============================

Bir dosya açıldığında işletim sistemi, açılacak dosyanın bilgilerini yol ifadesini çözümleyerek diskten elde
eder. Bu dosya bilgilerini çekirdek alanı içerisine çeker. Diskteki dosya bilgilerinin çekirdek alanında
yerleştirildiği yere *dosya nesnesi (file object)* denilmektedir. Buradaki *nesne (object)* terimi tahsis
edilmiş yapı alanları için kullanılmaktadır; nesne yönelimli programlama tekniğindeki *nesne* terimi ile
doğrudan bir ilgisi yoktur. Dosya nesnesi Linux'un kaynak kodlarında ``file`` isimli bir yapıyla temsil
edilmektedir. Güncel çekirdeklerde ``file`` yapısı Linux kaynak kodlarında ``include/linux/fs.h`` dosyasında
şöyle bildirilmiştir:

.. code-block:: c

    struct file {
        spinlock_t			f_lock;
        fmode_t				f_mode;
        const struct file_operations	*f_op;
        struct address_space		    *f_mapping;
        void				    *private_data;
        struct inode			*f_inode;
        unsigned int			f_flags;
        unsigned int			f_iocb_flags;
        const struct cred		*f_cred;
        struct fown_struct		*f_owner;
        /* --- cacheline 1 boundary (64 bytes) --- */
        union {
            const struct path	f_path;
            struct path		    __f_path;
        };
        union {
            /* regular files (with FMODE_ATOMIC_POS) and directories */
            struct mutex		f_pos_lock;
            /* pipes */
            u64			        f_pipe;
        };
        loff_t				    f_pos;
    #ifdef CONFIG_SECURITY
        void				    *f_security;
    #endif
        /* --- cacheline 2 boundary (128 bytes) --- */
        errseq_t			    f_wb_err;
        errseq_t			    f_sb_err;
    #ifdef CONFIG_EPOLL
        struct hlist_head	    *f_ep;
    #endif
        union {
            struct callback_head	f_task_work;
            struct llist_node	    f_llist;
            struct file_ra_state	f_ra;
            freeptr_t		        f_freeptr;
        };
        file_ref_t		f_ref;
        /* --- cacheline 3 boundary (192 bytes) --- */
    } __randomize_layout
    __attribute__((aligned(4)));	/* lest something weird decides that 2 is OK */

Yapının elemanlarını anlamlandıramayabilirsiniz. Bu yapının pek çok elemanını anlamlandırmak için başka
bilgilere sahip olmak gerekir.

İşletim sistemi, bir proses bir dosyayı açtığında açılan dosyayı o proses ile ilişkilendirmektedir. Yani dosya
nesnelerine proses kontrol blokları yoluyla erişilmektedir. Güncel Linux çekirdeklerinde bu durum biraz
karmaşıktır:

.. code-block:: text

    task_struct (files) ---> files_struct (fdt) ---> fdtable (fd) ---> file * türünden bir dizi ---> file nesnesi

Linux'ta proses kontrol bloğundan dosya nesnesine erişim birkaç yapıdan geçilerek yapılmaktadır. Ancak biz bu
durumu şöyle basitleştirerek ifade edebiliriz: proses kontrol bloğunda bir eleman bir diziyi göstermektedir. Bu
diziye *dosya betimleyici tablosu (file descriptor table)* denilmektedir. Dosya betimleyici tablosunun her
elemanı bir dosya nesnesini göstermektedir. Yani biz yukarıdaki yapıyı aşağıdaki gibi sadeleştirerek
kavramsallaştırıyoruz:

.. code-block:: text

    proses kontrol bloğu ---> betimleyici tablosu --> dosya nesneleri

----

Dosya Betimleyici Tablosu (File Descriptor Table)
-----------------------------------------------------

Dosya betimleyici tablosu (file descriptor table), dosya nesnelerinin adreslerini tutan bir gösterici dizisidir:

.. code-block:: text

                  Dosya Betimleyici Tablosu
    --------->      ┌──────────────────┐
                0   │      adres       │──────────►  dosya nesnesi
                    ├──────────────────┤
                1   │      adres       │──────────►  dosya nesnesi
                    ├──────────────────┤
                2   │      adres       │──────────►  dosya nesnesi
                    ├──────────────────┤
                3   │      adres       │──────────►  dosya nesnesi
                    ├──────────────────┤
                4   │      NULL        │
                    ├──────────────────┤
                5   │      NULL        │
                    ├──────────────────┤
                ... │      ...         │
                    ├──────────────────┤
               1023 │      NULL        │
                    └──────────────────┘

Dosya betimleyici tablosu prosese özgüdür ve ona o prosesin proses kontrol bloğu (Linux'ta ``task_struct``
yapısı) yoluyla erişilmektedir. Görüldüğü gibi dosya betimleyici tablosu aslında dosya nesnelerinin
(``struct file`` türünden nesnelerin) adreslerini tutmaktadır. Bir dosya açıldığında işletim sistemi dosyanın
bilgilerini diskten elde eder, bir dosya nesnesi tahsis edip o dosyanın bilgilerini dosya nesnesinin içerisine
yerleştirir ve dosya betimleyici tablosunun bir slotuna (dizinin elemanına) o adresi yazar.

Yukarıda da belirttiğimiz gibi dosya betimleyici tablosu prosese özgüdür, thread'e özgü değildir. Prosesin tüm
thread'leri aynı dosya betimleyici tablosunu kullanmaktadır. İşletim sistemi, o anda çalışan thread'in ilişkin
olduğu prosesin dosya betimleyici tablosunu kullanmaktadır.

Dosya nesnelerinin içerisinde açış bayrakları gibi, dosya göstericisinin konumu gibi pek çok bilgi doğrudan ve
pek çok bilgi de dolaylı bir biçimde saklanmaktadır. Yani dosya nesnesi, diskteki dosya üzerinde işlem yapmak
için gereken tüm bilgileri doğrudan ya da dolaylı biçimde bulundurmaktadır.


open Fonksiyonu
====================

UNIX/Linux sistemlerinde dosyayı açmak için ``open`` isimli POSIX fonksiyonu kullanılmaktadır. (Örneğin ``fopen``
standart C fonksiyonu da UNIX/Linux sistemlerinde aslında ``open`` fonksiyonunu çağırmaktadır.) Fonksiyonun
prototipi şöyledir:

.. code-block:: c

    #include <fcntl.h>

    int open(const char *path, int flags, ...);

``open`` fonksiyonu duruma göre üçüncü bir argüman da alabilmektedir. Eğer fonksiyon üç argümanla çağrılacaksa
üçüncü argüman ``mode_t`` türünden olmalıdır. Her ne kadar prototipteki ``...`` atomu *istenildiği kadar argüman
girilebilir* anlamına geliyorsa da ``open`` ya iki argümanla ya da üç argümanla çağrılmalıdır. ``open``
fonksiyonunu daha fazla argümanla çağırmak *tanımsız davranışa (undefined behavior)* yol açmaktadır. Biz daha
açıklayıcı olacak biçimde bu prototipi şöyle de yazabiliriz:

.. code-block:: c

    int open(const char *path, int flags, ... /* mode_t modu */ );

Başka bir gösterim de şöyle olabilir:

.. code-block:: c

    int open(const char *path, int flags);
    int open(const char *path, int flags, mode_t);

Tabii C'de aynı isimli birden fazla fonksiyon bulunamaz. Yukarıdaki gösterim yalnızca kullanımın nasıl
olabileceğini açıklamaktadır.

Açış Modları
-------------

``open`` fonksiyonunun birinci parametresi açılacak dosyanın yol ifadesini belirtir. İkinci parametre açış
bayraklarını (modlarını) belirtmektedir. Bu parametre ``O_XXX`` biçiminde isimlendirilmiş sembolik sabitlerin
*bit OR* işlemine sokulmasıyla oluşturulur. Açış sırasında aşağıdaki sembolik sabitlerden yalnızca birinin
belirtilmesi zorunludur:

.. list-table::
   :header-rows: 1
   :widths: 20 80

   * - Bayrak
     - Anlamı
   * - ``O_RDONLY``
     - Yalnızca okuma niyetiyle
   * - ``O_WRONLY``
     - Yalnızca yazma niyetiyle
   * - ``O_RDWR``
     - Hem okuma hem yazma niyetiyle
   * - ``O_SEARCH``
     - *at*'li fonksiyonlar için
   * - ``O_EXEC``
     - ``fexecve`` fonksiyonu için

Bu bayraklara başka bayraklar da eşlik edebilir; ancak yukarıdaki bayrakların yalnızca bir tanesi kullanılmak
zorundadır.

Buradaki ``O_RDONLY`` *yalnızca okuma yapma amacıyla*, ``O_WRONLY`` *yalnızca yazma yapma amacıyla* ve ``O_RDWR``
*hem okuma hem de yazma yapma amacıyla* dosyanın açılmak istendiği anlamına gelmektedir. İşletim sistemi,
prosesin etkin kullanıcı id'sine ve etkin grup id'sine ve dosyanın kullanıcı ve grup id'sine bakarak prosesin
dosyaya ``r``, ``w`` hakkının olup olmadığını kontrol eder. Eğer proses bu hakka sahip değilse ``open``
fonksiyonu başarısız olur. Yani erişim kontrolleri dosyadan okurken ya da dosyaya yazarken değil ``open`` ile
açış sırasında yapılmaktadır. Örneğin biz dosyayı şöyle açmak isteyelim:

.. code-block:: c

    fd = open("test.txt", O_RDONLY);

Burada işletim sistemi prosesin dosyaya ``r`` hakkı olup olmadığını kontrol edecektir. Örneğin:

.. code-block:: c

    fd = open("test.txt", O_WRONLY);

Burada işletim sistemi prosesin dosyaya ``w`` hakkı olup olmadığını kontrol edecektir. Örneğin:

.. code-block:: c

    fd = open("test.txt", O_RDWR);

Burada işletim sistemi prosesin dosyaya hem ``r`` hem de ``w`` hakkı olup olmadığını kontrol edecektir.

Zorunlu açış bayraklarından ``O_SEARCH`` bayrağı bazı POSIX fonksiyonlarının *at*'li versiyonları için,
``O_EXEC`` bayrağı ise ``fexecve`` fonksiyonu için bulundurulmuştur. Bu bayraklar ileride ele alınacaktır.

----

O_CREAT ve Erişim Hakları
==============================

``open`` fonksiyonu yalnızca olan bir dosyayı açmak için değil aynı zamanda yeni bir dosya yaratmak için de
kullanılmaktadır. ``O_CREAT`` bayrağı, dosya varsa etkili olmaz; dosya yoksa dosyanın yaratılmasını sağlar. Yani
``O_CREAT`` bayrağı *dosya varsa olanı aç, yoksa yarat ve aç* anlamına gelmektedir. Bir dosya yaratılırken
dosyanın erişim haklarını, dosyayı yaratan kişi ``open`` fonksiyonunun üçüncü parametresinde vermek zorundadır.
Yani dosyanın erişim haklarını dosyayı yaratan kişi belirlemektedir. Biz ``O_CREAT`` bayrağını açış moduna
eklemişsek bu durumda *dosya yaratılabilir* fikri ile erişim haklarını ``open`` fonksiyonunun üçüncü
parametresinde belirtmemiz gerekir.

Erişim hakları ``<sys/stat.h>`` dosyası içerisinde, tüm bitleri sıfır tek biti ``1`` olan sembolik sabitlerin
*bit OR* işlemine sokulmasıyla oluşturulmaktadır. Bu sembolik sabitlerin hepsi ``S_I`` öneki ile başlar. Bunu
``R``, ``W`` ya da ``X`` harfi izler. Bunu da ``USR``, ``GRP`` ya da ``OTH`` harfleri izlemektedir. Yani bu
sembolik sabitlerin oluşturulma biçimi şöyledir:

.. code-block:: text

    S_I[RWX][USR GRP OTH]

Böylece 9 tane erişim hakkı şöyle isimlendirilmiştir:

.. code-block:: c

    S_IRUSR
    S_IWUSR
    S_IXUSR
    S_IRGRP
    S_IWGRP
    S_IXGRP
    S_IROTH
    S_IWOTH
    S_IXOTH

Örneğin ``S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH`` erişim hakları ``rw-r--r--`` anlamına gelmektedir. Örneğin
``S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH`` erişim hakları ``rw-rw-r--`` anlamına gelmektedir. Burada *owner*
sözcüğü yerine *user* sözcüğünün kullanıldığına dikkat ediniz.

Ayrıca ``<sys/stat.h>`` içerisinde aşağıdaki sembolik sabitler de bildirilmiştir:

.. code-block:: c

    S_IRWXU
    S_IRWXG
    S_IRWXO

Bu sembolik sabitler şöyle oluşturulmuştur:

.. code-block:: c

    #define S_IRWXU (S_IRUSR|S_IWUSR|S_IXUSR)
    #define S_IRWXG (S_IRGRP|S_IWGRP|S_IXGRP)
    #define S_IRWXO (S_IROTH|S_IWOTH|S_IXOTH)

Bu durumda örneğin ``S_IRWXU|S_IRWXG|S_IRWXO`` işlemi ``rwxrwxrwx`` anlamına gelmektedir.

Sayısal Değerler
-----------------

Yukarıdaki ``S_IXXX`` biçimindeki sembolik sabitlerin değerlerinin eskiden sistemden sisteme değişebileceği
varsayılmıştır. Bu nedenle POSIX standartları başlarda bu sembolik sabitlerin sayısal değerlerini işletim
sistemlerini oluşturanların belirlemesini istemiştir. Ancak daha sonraları (2008 ve sonrasında, SUS 4 ve
sonrasında) bu sembolik sabitlerin değerleri POSIX standartlarında açıkça belirtilmiştir. Dolayısıyla
programcılar artık bu sembolik sabitleri kullanmak yerine bunların sayısal karşılıklarını da kullanabilir
duruma gelmiştir. Ancak eski sistemler dikkate alındığında bunların sayısal karşılıkları yerine yukarıdaki
sembolik sabitlerin kullanılması tavsiye edilmektedir. Bu sembolik sabitler aynı zamanda okunabilirliği de
artırmaktadır. POSIX standartları 2008 ve sonrasında bu sembolik sabitlerin sayısal değerlerini aşağıdaki gibi
belirlemiştir:

.. list-table::
   :header-rows: 1
   :widths: 30 30

   * - Sembolik Sabit
     - Sayısal Değer (octal)
   * - ``S_IRWXU``
     - ``0700``
   * - ``S_IRUSR``
     - ``0400``
   * - ``S_IWUSR``
     - ``0200``
   * - ``S_IXUSR``
     - ``0100``
   * - ``S_IRWXG``
     - ``070``
   * - ``S_IRGRP``
     - ``040``
   * - ``S_IWGRP``
     - ``020``
   * - ``S_IXGRP``
     - ``010``
   * - ``S_IRWXO``
     - ``07``
   * - ``S_IROTH``
     - ``04``
   * - ``S_IWOTH``
     - ``02``
   * - ``S_IXOTH``
     - ``01``
   * - ``S_ISUID``
     - ``04000``
   * - ``S_ISGID``
     - ``02000``
   * - ``S_ISVTX``
     - ``01000``

Yani 2008 ve sonrasında artık ``rwxrwxrwx`` biçiminde owner, group ve other bilgilerine ilişkin ``S_IXXX``
biçimindeki sembolik sabitler gerçekten yukarıdaki sıraya göre bitleri temsil eder hale gelmiştir. Örneğin
``S_IWGRP`` sembolik sabiti ``000010000`` bitlerinden oluşmaktadır. Bu durumda 2008 ve sonrasında örneğin
``S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH`` bir erişim hakkını biz doğrudan ``0644`` octal değeri ile de verebiliriz. Bu
sembolik sabitlerin binary karşılıklarını da vermek istiyoruz:

.. code-block:: text

    S_IRUSR        100 000 000
    S_IWUSR        010 000 000
    S_IXUSR        001 000 000

    S_IRGRP        000 100 000
    S_IWGRP        000 010 000
    S_IXGRP        000 001 000

    S_IROTH        000 000 100
    S_IWOTH        000 000 010
    S_IXOTH        000 000 001

Örneğin:

.. code-block:: c

    fd = open("test.txt", O_RDWR|O_CREAT, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH);

çağrısı aşağıdakiyle eşdeğerdir:

.. code-block:: c

    fd = open("test.txt", O_RDWR|O_CREAT, 0644);

``open`` fonksiyonunda ``O_CREAT`` bayrağı belirtilmemişse erişim haklarının girilmesinin hiçbir anlamı yoktur.
Kaldı ki ``O_CREAT`` bayrağı girildiğinde de dosya varsa erişim hakları yine dikkate alınmamaktadır. Ancak biz
``open`` fonksiyonunun ikinci parametresinde ``O_CREAT`` bayrağını girmişsek *dosya yaratılabilir* düşüncesiyle
dosyanın erişim haklarını da ``open`` fonksiyonunun üçüncü parametresi için girmeliyiz.

Yeni yaratılacak dosya için en çok kullanılan erişim hakları ``rw-r--r--`` biçimindedir. Bu haklar
``S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH`` ya da doğrudan ``0644`` ile verilebilir.

POSIX sistemlerinde yukarıdaki ``S_IXXX`` biçimindeki sembolik sabitler ``mode_t`` türüyle temsil edilmiştir.
``mode_t`` türü ``<sys/types.h>`` ve bazı başlık dosyalarında (örneğin ``<sys/stat.h>``) sistemi oluşturanların
belirlediği bir *tamsayı türü olarak* typedef edilmiştir. Linux'ta ``<sys/types.h>`` dosyası içerisinde
``mode_t`` türü ``unsigned int`` biçiminde typedef edilmiştir.

----

Diğer Önemli Açış Bayrakları
=============================

``O_TRUNC`` açış bayrağı *eğer dosya varsa onu sıfırlayarak aç* anlamına gelmektedir. Ancak ``O_TRUNC`` yazma
modunda açılan dosyalarda kullanılabilmektedir. Yani ``O_TRUNC`` bayrağını kullanabilmek için ``O_WRONLY`` ya da
``O_RDWR`` bayraklarından birinin de belirtilmiş olması gerekmektedir. Örneğin ``O_WRONLY|O_CREAT|O_TRUNC``
açış modu *dosya yoksa yarat ancak dosya varsa içini sıfırlayarak aç* anlamına gelmektedir. ``O_TRUNC`` bayrağı
için dosyanın yaratılıyor olması gerekmez (zaten dosya yaratılırken içinde bir şey olmayacaktır).
``O_WRONLY|O_TRUNC`` geçerli bir açış modudur. Bu durumda dosya yoksa ``open`` başarısız olur. Ancak dosya
varsa sıfırlanarak açılır.

``O_APPEND`` bayrağı yazma işlemlerinin dosyanın sonuna yapılacağı anlamına gelmektedir. Yani bu bayrak
kullanılırsa tüm yazma işlemlerinde işletim sistemi dosya göstericisini dosyanın sonuna çekip sonra yazmayı
yapmaktadır. Bu açış modu da ``O_WRONLY`` ya da ``O_RDWR`` için anlamlıdır. Örneğin ``O_RDWR|O_APPEND``
kullanıldığında dosyaya her yazılan sona eklenecektir. Ancak dosyanın herhangi bir yerinden okuma
yapılabilecektir.

O halde standart C'nin ``fopen`` fonksiyonundaki açış modlarının POSIX karşılıkları şöyle oluşturulabilir:

.. list-table::
   :header-rows: 1
   :widths: 20 50

   * - Standart C
     - POSIX
   * - ``"w"``
     - ``O_WRONLY|O_CREAT|O_TRUNC``
   * - ``"w+"``
     - ``O_RDWR|O_CREAT|O_TRUNC``
   * - ``"r"``
     - ``O_RDONLY``
   * - ``"r+"``
     - ``O_RDWR``
   * - ``"a"``
     - ``O_WRONLY|O_CREAT|O_APPEND``
   * - ``"a+"``
     - ``O_RDWR|O_CREAT|O_APPEND``

``O_EXCL`` bayrağı *exclusive* açım için kullanılmaktadır. Bu bayrak tek başına değil ``O_CREAT`` ile birlikte
kullanılmak zorundadır. ``O_CREAT|O_EXCL`` biçimindeki açış modu *dosya yoksa yarat, varsa yaratma başarısız ol*
anlamına gelmektedir. Yani bu modu kullanan programcı *mutlaka dosyanın sıfırdan yaratılmasını* istemektedir.
``O_EXCL`` bayrağının ``O_CREAT`` olmadan kullanılması *tanımsız davranışa* yol açmaktadır.

``O_DIRECTORY`` bayrağı, açılmak istenen dosya bir dizin dosyası değilse açımın başarısız olmasını
sağlamaktadır. Dizin dosyaları da ``open`` fonksiyonuyla bu bayrak kullanılarak açılabilmektedir.

``open`` fonksiyonunun diğer açış modları ileride başka konular içerisinde ele alınacaktır. Bazı modları
görmemiş olsak da açış modlarının hepsini aşağıda bir tablo halinde veriyoruz:

.. list-table:: ``open`` Açış Bayrakları
   :header-rows: 1
   :widths: 18 62 20

   * - Bayrak
     - İşlev
     - Standart
   * - ``O_RDONLY``
     - Dosyayı yalnızca okuma modunda aç.
     - POSIX
   * - ``O_WRONLY``
     - Dosyayı yalnızca yazma modunda aç.
     - POSIX
   * - ``O_RDWR``
     - Dosyayı hem okuma hem yazma modunda aç.
     - POSIX
   * - ``O_CREAT``
     - Dosya yoksa oluştur. ``mode`` argümanıyla izin bitleri verilir.
     - POSIX
   * - ``O_EXCL``
     - ``O_CREAT`` ile birlikte: dosya zaten varsa hata döndür (``EEXIST``).
     - POSIX
   * - ``O_TRUNC``
     - Dosya varsa ve yazma modundaysa içeriğini sıfırla.
     - POSIX
   * - ``O_APPEND``
     - Her ``write()`` çağrısından önce dosya göstericisini sona taşır; atomik ekleme garantisi verir.
     - POSIX
   * - ``O_NONBLOCK``
     - G/Ç işlemlerini bloke etmeyen (non-blocking) modda gerçekleştirir.
     - POSIX
   * - ``O_NOCTTY``
     - Dosya bir terminal aygıtıysa, sürecin kontrol terminali (controlling terminal) olarak atanmasını
       engeller.
     - POSIX
   * - ``O_CLOEXEC``
     - Dosya tanımlayıcısına ``FD_CLOEXEC`` bayrağı atar; ``exec()`` sonrası fd otomatik kapanır.
       ``fork()`` + ``exec()`` yarış koşulunu önler.
     - POSIX
   * - ``O_DSYNC``
     - Her ``write()``, yalnızca veriyi diske flush edene kadar bloke eder; meta veri (erişim zamanı vb.)
       beklenmez.
     - POSIX (2008'den)
   * - ``O_SYNC``
     - Her ``write()``, veri ve meta veriyi diske flush edene kadar bloke eder. ``O_DSYNC`` + meta veri
       garantisi; ``__O_SYNC | O_DSYNC`` olarak tanımlıdır.
     - POSIX (2008'den)
   * - ``O_RSYNC``
     - ``read()`` çağrıları, bekleyen ``write()`` G/Ç'lerinin tamamlanmasını bekler. Linux'ta ``O_SYNC`` ile
       eş anlamlı davranır.
     - POSIX (2008'den)
   * - ``O_NDELAY``
     - ``O_NONBLOCK`` ile eş anlamlı; eski BSD uyumluluğu için korunur.
     - Linux'a Özgü
   * - ``O_DIRECTORY``
     - Yol bir dizin dosyası değilse ``ENOTDIR`` döndürür; yalnızca dizin dosyası açmak için kullanılır.
     - Linux'a Özgü
   * - ``O_NOFOLLOW``
     - Yolun son bileşeni sembolik bağsa (symlink) ``ELOOP`` döndürür; symlink takibini engeller.
     - Linux'a Özgü
   * - ``O_NOATIME``
     - ``read()`` çağrılarında inode'un ``atime`` (erişim zamanı) alanını güncellemez. Yedekleme ve dizin
       tarama araçlarında kullanılır.
     - Linux'a Özgü
   * - ``O_PATH``
     - Dosya içeriğine değil yalnızca dosya sistemi konumuna referans için fd açar. Okuma/yazma yapılamaz;
       ``fstatat()``, ``openat()`` vb. için kullanılır.
     - Linux'a Özgü
   * - ``O_TMPFILE``
     - İsimsiz geçici bir dosya oluşturur; fd kapanınca dosya otomatik silinir. ``linkat()`` ile kalıcı hale
       getirilebilir.
     - Linux'a Özgü
   * - ``O_LARGEFILE``
     - 32-bit sistemlerde 2 GB'ı aşan dosyaların açılmasına izin verir. 64-bit sistemlerde örtük olarak
       etkindir.
     - Linux'a Özgü
   * - ``O_ASYNC``
     - Dosya üzerinde G/Ç hazır olduğunda ``SIGIO`` sinyali gönderir (sinyal güdümlü G/Ç). ``fcntl()`` ile de
       ayarlanabilir.
     - Linux'a Özgü

Yukarıda da belirttiğimiz gibi erişim hakları ``open`` fonksiyonu tarafından (yani ``open`` fonksiyonunun
çağırdığı sistem fonksiyonu tarafından) kontrol edilmektedir. Örneğin biz dosyayı ``O_RDWR`` modunda açmak
isteyelim; bu durumda prosesimizin dosyaya ``r`` ve ``w`` haklarına sahip olması gerekir. Eğer prosesimiz dosya
için bu haklara sahip değilse ``open`` başarısız olur ve ``errno`` ``EACCES`` (*Permission denied*) değeri ile
set edilir. Burada önemli olan nokta, kontrolün en başta ``open`` fonksiyonu tarafından yapılmasıdır.

``open`` fonksiyonu başarı durumunda ``int`` türden *dosya betimleyicisi (file descriptor)* denilen bir değerle
geri dönmektedir. Dosya betimleyicisi bir handle olarak diğer fonksiyonlar tarafından istenmektedir. ``open``
başarısızlık durumunda ``-1`` ile geri döner ve ``errno`` uygun biçimde set edilir. ``open`` fonksiyonunun
başarısız olması için pek çok neden söz konusudur. Bundan dolayı açma işleminin başarısı kesinlikle test
edilmelidir. Örneğin:

.. code-block:: c

    int fd;

    if ((fd = open("test.txt", O_RDONLY)) == -1)
        exit_sys("open");

----

open Sisteminin İşleyişi ve Dosya Betimleyici Tablosu
==========================================================

``open`` fonksiyonu işletim sisteminin dosya açan sistem fonksiyonunu (Linux'ta ``sys_open``) çağırmaktadır. Bu
sistem fonksiyonu açılacak dosyaya ilişkin bilgileri diskten bulur ve o bilgileri daha önceden de belirttiğimiz
gibi *dosya nesnesi (file object)* denilen bir yapının içerisine yerleştirir. Dosya nesnesinin Linux'un kaynak
kodlarında ``struct file`` türü ile temsil edildiğini söylemiştik. İşletim sistemi dosya nesnesinin içini
doldurduktan sonra dosya betimleyici tablosunda boş bir slot bulur ve o slota dosya nesnesinin adresini yazar.
Anımsanacağı gibi dosya betimleyici tablosu, dosya nesnelerinin adreslerini tutan bir gösterici dizisi
biçiminde organize edilmiştir. Dosya betimleyici tablosunun yeri prosesin kontrol bloğundan hareketle elde
edilmektedir. İşte ``open`` fonksiyonunun bize geri döndürdüğü dosya betimleyicisi aslında dosya betimleyici
tablosunda (yani gösterici dizisinde) bir indeks belirtmektedir. Örneğin ``open`` fonksiyonu dosya nesnesinin
adresini dosya betimleyici tablosunun 3. indeksli slotuna yerleştirmiş olsun:

.. code-block:: text

                 Dosya Betimleyici Tablosu
    --------->      ┌──────────────────┐
                0   │      adres       │──────────►  dosya nesnesi
                    ├──────────────────┤
                1   │      adres       │──────────►  dosya nesnesi
                    ├──────────────────┤
                2   │      adres       │──────────►  dosya nesnesi
                    ├──────────────────┤
                3   │      adres       │──────────► Açtığımız dosyaya ilişkin dosya nesnesi
                    ├──────────────────┤
                4   │      NULL        │
                    ├──────────────────┤
                5   │      NULL        │
                    ├──────────────────┤
                ... │      ...         │
                    ├──────────────────┤
               1023 │      NULL        │
                    └──────────────────┘

Bu durumda ``open`` fonksiyonu ``3`` değeri ile geri dönecektir.

Bir program çalıştığında genellikle dosya betimleyici tablosunun ilk üç betimleyicisi dolu, diğerleri boştur.
Dosya betimleyici tablosunun 0. slotu (yani ``0`` numaralı betimleyici) terminal aygıt sürücüsü için
oluşturulmuş dosya nesnesini göstermektedir. Buna *stdin dosya betimleyicisi* de denilmektedir. ``1`` ve ``2``
numaralı betimleyiciler yine terminal aygıt sürücüsü için oluşturulmuş dosya nesnesini gösterir (``1`` ve ``2``
numaralı betimleyiciler aynı nesneyi göstermektedir). Bu betimleyicilere de sırasıyla *stdout ve stderr dosya
betimleyicileri* denilmektedir. Böylece ilk boş betimleyici genellikle ``3`` numaralı betimleyici olmaktadır.
Yukarıda da belirttiğimiz gibi ``open`` fonksiyonunun dosya betimleyici tablosunda ilk boş betimleyiciyi vermesi
POSIX standartlarında garanti edilmiştir.

Her prosesin proses kontrol bloğu ve dolayısıyla dosya betimleyici tablosu birbirinden farklıdır. O halde
dosya betimleyicileri kendi prosesinin dosya betimleyici tablosunda bir indeks belirtmektedir. Yani dosya
betimleyicileri prosese özgü bir anlama sahiptir. Biz bir prosesteki dosya betimleyicisini prosesler arası
haberleşme yöntemleriyle başka bir prosese göndersek, o betimleyici o proseste farklı bir dosyaya referans
edebilir ya da geçersiz olabilir.

Bu durumda tipik olarak işletim sisteminin dosya açan sistem fonksiyonu sırasıyla şu işlemleri yapmaktadır:

1. Dosya betimleyici tablosunda ilk boş betimleyiciyi bulmaya çalışır. Boş bir betimleyici bulamazsa
   başarısız olur ve ``errno`` değerini ``EMFILE`` ile set eder.
2. Dosya nesnesini tahsis eder ve bunun içini diskten elde ettiği bilgilerle doldurur. Bunun adresini de dosya
   betimleyici tablosunda ilk boş betimleyiciye ilişkin slota yerleştirir.
3. Dosya betimleyici tablosunda indeks belirten betimleyici değeri ile geri döner.

C'nin ``fopen`` fonksiyonunda dosya açımı sırasında *text mode*, *binary mode* gibi bir kavram vardır. Halbuki
işletim sisteminde böyle bir kavram yoktur. İşletim sistemine göre dosya byte'lardan oluşmaktadır. *Text mode*,
*binary mode* C ve diğer diller tarafından uydurulmuş olan yapay bir kavramdır.

Bir proses her ``open`` işlemi yaptığında kesinlikle yeni bir dosya nesnesi oluşturulmaktadır. Bu durumda bir
proses aynı dosyayı aynı biçimde ikinci kez açmış olsa bile aynı dosya nesnesi kullanılmaz. Her iki ``open``
çağrısı iki farklı dosya nesnesinin ve dosya betimleyicisinin oluşmasına yol açmaktadır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <sys/stat.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;

        if ((fd = open("test.txt", O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH)) == -1)
            exit_sys("open");

        printf("file opened: %d\n", fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

Dosya Betimleyici Tablosunun Sınırı: EMFILE
----------------------------------------------------

Linux sistemlerinde varsayılan olarak proseslerin dosya betimleyici tabloları 1024 slottan oluşmaktadır. Yani
varsayılan durumda bu sistemlerde bir proses, kapatmadan en fazla 1024 dosyayı açık olarak tutabilmektedir.
Yukarıda da belirttiğimiz gibi eğer dosya betimleyici tablosunda boş yer yoksa ``open`` fonksiyonu başarısız
olur ve ``EMFILE`` ``errno`` değeri set edilir. Aşağıdaki örnekte döngü içerisinde dosya hiç kapatılmadan
sürekli açılmıştır. Açım işlemi başarısız olduğunda döngüden çıkılmış ve ``errno`` değerini gözlemlemek için
dosya bir kez daha açılmak istenmiştir. En sonunda şöyle bir hata ile karşılaşılacaktır:

.. code-block:: text

    open: Too many open files

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;

        for (int i = 0;; ++i) {
            if ((fd = open("test.txt", O_RDONLY)) == -1)
                break;
            printf("%d\n", fd);
        }

        if ((fd = open("test.txt", O_RDONLY)) == -1)
            exit_sys("open");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

close Fonksiyonu
------------------------

Açılan her dosyanın kapatılması gerekir. Bir dosyanın kapatılması sırasında işletim sistemi, dosyanın açılması
sırasında yapılan işlemleri geri almaktadır. Tipik olarak UNIX/Linux sistemlerinde dosya kapatıldığında şunlar
yapılmaktadır:

1. Dosya nesnesi, eğer onu gösteren tek bir betimleyici varsa, yok edilir.
2. Dosya betimleyici tablosundaki betimleyiciye ilişkin slot boşaltılır.

İleride de göreceğimiz gibi dosya betimleyici tablosunda birden fazla betimleyici aynı dosya nesnesini
gösteriyor durumda olabilmektedir. Bu durumda işletim sistemi dosya nesnesi içerisinde bir sayaç tutup bu
sayacı artırıp eksiltmektedir. Sayaç ``0``'a düştüğünde nesneyi silmektedir. (Linux'un kaynak kodlarında bu
sayaç eski çekirdeklerde ``struct file`` yapısının ``f_count`` elemanında, yeni çekirdeklerde ise ``f_ref``
elemanında tutulmaktadır.)

Bir dosya artık kullanılmayacaksa onu kapatmak iyi bir tekniktir. Çünkü bu sayede:

1. Dosya betimleyici tablosundaki slot serbest bırakılır.
2. Dosya nesnesi gereksiz bir biçimde çekirdek alanı içerisinde yer kaplamaz.

Tabii işletim sistemi, proses dosyayı kapatmasa bile proses sonlandırılırken prosesin dosya betimleyici
tablosunu inceler ve açık dosyaları ``close`` işlemi ile kapatır. Yani biz bir dosyayı kapatmasak bile proses
bittiğinde dosyalar zaten kapatılmaktadır. Ancak dosyaların kullanımı bittikten sonra erken bir biçimde
programcı tarafından kapatılması iyi bir tekniktir.

Dosyanın kapatılması için ``close`` isimli POSIX fonksiyonu bulundurulmuştur. Bu POSIX fonksiyonu doğrudan
işletim sisteminin dosyayı kapatan sistem fonksiyonunu (Linux'ta ``sys_close``) çağırmaktadır. ``close``
fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int close(int fd);

Fonksiyon parametre olarak dosya betimleyicisini alır. ``close`` fonksiyonu başarı durumunda ``0``,
başarısızlık durumunda ``-1`` değerine geri dönmektedir. Fonksiyonun geri dönüş değeri genellikle kontrol
edilmez. Eğer programcı fonksiyona geçerli bir dosya betimleyicisini argüman olarak geçmişse fonksiyonun
başarısız olması mümkün değildir.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <sys/stat.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;

        if ((fd = open("test.txt", O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH)) == -1)
            exit_sys("open");

        printf("file opened: %d\n", fd);

        close(fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

creat Fonksiyonu
------------------------

İlk UNIX sistemlerinden beri ``creat`` isimli bir fonksiyon da ``open`` fonksiyonunun bir sarma fonksiyonu
biçiminde bulundurulmaktadır. ``creat`` fonksiyonu POSIX standartlarında var olan bir fonksiyondur. Fonksiyonun
prototipi şöyledir:

.. code-block:: c

    #include <fcntl.h>

    int creat(const char *path, mode_t mode);

Fonksiyonun birinci parametresi dosyanın yol ifadesini belirtmektedir. İkinci parametre erişim bilgisini
belirtir. Görüldüğü gibi fonksiyonda açış modu belirten ``flags`` parametresi yoktur. Çünkü bu parametre
``O_WRONLY|O_CREAT|O_TRUNC`` biçiminde alınmaktadır. ``creat`` fonksiyonu aşağıdaki gibi yazılmıştır:

.. code-block:: c

    int creat(const char *path, mode_t mode)
    {
        return open(path, O_WRONLY|O_CREAT|O_TRUNC, mode);
    }

Ancak biz kursumuzda bu ``creat`` fonksiyonu yerine asıl fonksiyon olan ``open`` fonksiyonunu kullanacağız.

----

Dosya Göstericisi (File Pointer) ve Offset Kavramı
------------------------------------------------------

Dosyadaki her bir byte'a bir offset numarası karşı getirilmiştir. Buna ilgili byte'ın offset'i denilmektedir.
Dosya göstericisi (file pointer), okuma ve yazma işlemlerinin hangi offset'ten itibaren yapılacağını gösteren
bir offset belirtmektedir. Okuma ya da yazma miktarı kadar dosya göstericisi otomatik olarak ilerletilmektedir.
Dosya ilk açıldığında dosya göstericisi ``0`` durumundadır. Örneğin dosyanın içerisinde ``ankara`` byte'ları
olsun:

.. code-block:: text

    0123456
    ankara
    ^

Bu dosya açıldığında dosya göstericisi ilk byte'ın offset'i olan ``0``'ı göstermektedir. Biz bu pozisyondan iki
byte okursak ``an`` byte'larını okuruz ve dosya göstericisi de 2 byte ilerletilir:

.. code-block:: text

    0123456
    ankara
      ^

Şimdi 2 byte daha okursak artık ``ka`` byte'larını okuruz:

.. code-block:: text

    0123456
    ankara
        ^

Dosya göstericisinin dosyanın son byte'ından sonraki byte'ı göstermesi durumuna *EOF (End of File) durumu*
denir. EOF durumunda dosyadan okuma yapılamaz, çünkü okunacak bir şey yoktur. Ancak EOF durumunda dosyaya yazma
yapılabilir. Bu durumda yazılanlar dosyaya eklenmiş olur. Dosyada araya bir şey eklemek (insert) diye bir
kavram yoktur. Dosya boyutunu değiştirmek için dosya göstericisini EOF'a çekip yazma yapmak gerekir. Örneğin:

.. code-block:: text

    0123456
    ankara
          ^

Burada dosya göstericisi EOF konumundadır. Şimdi biz bu dosyaya ``istanbul`` yazısının byte'larını yazacak
olsak bunlar artık dosyaya eklenecektir:

.. code-block:: text

    012345678901234
    ankaraistanbul
                  ^

Bir dosya yeni yaratıldığında dosyanın içi boştur, dolayısıyla dosya göstericisi de zaten EOF durumundadır.
Örneğin:

.. code-block:: text

    0
    ^

Biz yeni yaratılmış bir dosyaya yazma yaparsak ona ekleme yapmış oluruz. Örneğin:

.. code-block:: text

    012345678
    istanbul
            ^

Dosya göstericisinin konumu dosya nesnesi içerisinde saklanmaktadır. (Linux'un kaynak kodlarında ``file``
yapısının ``f_pos`` elemanı dosya göstericisinin konumunu tutmaktadır.) Biz aynı dosyayı ikinci kez açmış olsak
bile yeni bir dosya nesnesi, dolayısıyla yeni bir dosya göstericisi elde etmiş oluruz.


----

read Fonksiyonu
====================

Dosyadan okuma yapmak için ``read`` POSIX fonksiyonu kullanılmaktadır. Pek çok sistemde bu POSIX fonksiyonu
doğrudan işletim sisteminin okuma yapan sistem fonksiyonunu (Linux'ta ``sys_read``) çağırmaktadır. ``read``
fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    ssize_t read(int fd, void *buf, size_t nbyte);

Fonksiyonun birinci parametresi okuma işleminin yapılacağı dosya betimleyicisini belirtmektedir. İşletim sistemi
bu betimleyiciden hareketle dosya nesnesine erişir. İkinci parametre dosyadan okunan byte'ların
yerleştirileceği bellek transfer adresini, üçüncü parametre okunacak byte sayısını belirtmektedir.

Fonksiyon başarı durumunda okuyabildiği byte sayısıyla geri döner. ``read`` fonksiyonu ile eğer dosya
göstericisinin gösterdiği yerden itibaren dosya sonuna kadar mevcut olan byte miktarından daha fazla byte
okunmak istenirse, ``read`` fonksiyonu okuyabildiği kadar byte'ı okur ve okuyabildiği byte sayısına geri döner.
Dosya göstericisi EOF durumunda ise ``read`` hiç okuma yapamayacağı için ``0`` ile geri dönmektedir. Ancak
argümanların yanlış girilmesinde ya da G/Ç hatalarında ``read`` başarısız olur ve ``-1`` değerine geri döner;
``errno`` uygun bir biçimde set edilmektedir. ``ssize_t`` türü ``<unistd.h>`` ve ``<sys/types.h>`` dosyaları
içerisinde *işaretli bir tamsayı türü olacak biçiminde* typedef edilmek zorunda olan POSIX'e özgü bir tür
ismidir. ``ssize_t`` türünü ``size_t`` türünün işaretli biçimi olarak düşünebilirsiniz. ``size_t`` türü C
standartlarında olduğu halde ``ssize_t`` türü C standartlarında bulunmamaktadır. ``read`` fonksiyonunu tipik
olarak şöyle kullanmalısınız:

.. code-block:: c

    ssize_t result;
    char buf[BUFFER_SIZE];

    if ((result = read(fd, buf, BUFFER_SIZE)) == -1)
        exit_sys("read");

Talep ettiğiniz kadar bilginin okunup okunmadığını anlamak için ayrıca kontrol yapabilirsiniz:

.. code-block:: c

    if ((result = read(fd, buf, BUFFER_SIZE)) != BUFFER_SIZE) {
        if (result == -1)
            exit_sys("read");
        fprintf(stderr, "cannot read enough!..\n");
        exit(EXIT_FAILURE);
    }

Eğer bir metin dosyasından okuma yapıp okunanı yazdırmak istiyorsanız null karakteri eklemeyi unutmayınız.
Örneğin:

.. code-block:: c

    char buf[BUFFER_SIZE + 1];
    ssize_t result;

    if ((result = read(fd, buf, BUFFER_SIZE)) == -1)
        exit_sys("read");

    buf[result] = '\0';
    puts(buf);

``read`` fonksiyonu ile dosyadan ``0`` byte okunmak istendiğinde ``read`` fonksiyonu temel bazı kontrolleri
yapar (örneğin dosyanın okuma modunda açılmış olup olmadığı kontrol edilir). Eğer bu kontrollerde bir sorun
çıkarsa fonksiyon başarısız olur ve ``-1`` değerine geri döner. Eğer bu kontrollerde bir sorun çıkmazsa ``0``
değerine geri döner ve herhangi bir okuma işlemi yapmaz.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;
        char buf[10 + 1];
        ssize_t result;

        if ((fd = open("test.txt", O_RDONLY)) == -1)
            exit_sys("open");

        if ((result = read(fd, buf, 10)) == -1)
            exit_sys("read");

        buf[result] = '\0';
        printf(":%s:\n", buf);

        close(fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Şimdi bir dosyayı dosya sonuna kadar ``read`` fonksiyonu ile bir döngü içerisinde okuyalım. Bu tür durumlarda
klasik yöntem aşağıdaki gibi bir döngü oluşturmaktır:

.. code-block:: c

    while ((result = read(fd, buf, BUFSIZE)) > 0) {
        /* ... */
    }
    if (result == -1)
        exit_sys("read");

Bu döngüden G/Ç hatası oluşunca ya da dosya göstericisi dosyanın sonuna geldiğinde çıkılacaktır. Döngüden
çıkıldığında neden çıkıldığı da ayrıca sorgulanmıştır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <unistd.h>

    #define BUFSIZE        4096

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;
        char buf[BUFSIZE + 1];
        ssize_t result;

        if ((fd = open("sample.c", O_RDONLY)) == -1)
            exit_sys("open");

        while ((result = read(fd, buf, BUFSIZE)) > 0) {
            buf[result] = '\0';
            printf("%s", buf);
        }

        if (result == -1)
            exit_sys("read");

        close(fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

write Fonksiyonu
=====================

Dosyaya yazma yapmak için ``write`` isimli POSIX fonksiyonu kullanılmaktadır. Bu fonksiyon da pek çok sistemde
doğrudan işletim sisteminin dosyaya yazma yapan sistem fonksiyonunu (Linux'ta ``sys_write``) çağırmaktadır.
Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    ssize_t write(int fd, const void *buf, size_t nbyte);

Fonksiyonun birinci parametresi yazma yapılacak dosyaya ilişkin dosya betimleyicisini, ikinci parametre
yazılacak bilgilerin bulunduğu bellek transfer adresini, üçüncü parametre ise yazılacak byte sayısını
belirtmektedir. ``write`` fonksiyonu başarılı olarak yazılan byte sayısıyla geri dönmektedir. Normal olarak bu
değer üçüncü parametrede belirtilen yazılmak istenen byte sayısıdır. Ancak çok seyrek bazı durumlarda ``write``
fonksiyonu talep edilenden daha az byte'ı da dosyaya yazabilir. Bu durumda ``write`` fonksiyonu yazabildiği
byte sayısıyla geri dönmektedir. ``write`` fonksiyonu başarısız olursa ``-1`` değerine geri döner ve ``errno``
uygun biçimde set edilir. Fonksiyon tipik olarak şöyle kullanılmaktadır:

.. code-block:: c

    if (write(fd, buf, size) == -1)
        exit_sys("write");

Disk doluysa ya da yazılmak istenen dosya işletim sisteminin kullandığı dosya sisteminin izin verilen uzunluğunu
aşıyorsa ``write`` fonksiyonu yazabildiği kadar byte'ı yazar, yazabildiği byte sayısına geri döner (*partial
write*); ancak bu nedenlerle ``write`` hiç byte yazamazsa başarısız olup ``-1`` değeriyle geri dönmektedir. Bu
durumda ``errno`` ``EFBIG`` değeri ile set edilmektedir. ``write`` fonksiyonunun boru gibi özel dosyalardaki
davranışı farklıdır. Bu konu ileride ele alınacaktır.

``write`` fonksiyonu yazmayı EOF ötesine yapabilir; bu durumda yazılanlar dosyaya eklenmiş olmaktadır. Örneğin
dosya yeni yaratılmış olsun. Bu durumda ``write`` fonksiyonuyla her yazılan dosyaya eklenmiş olacaktır.

``write`` fonksiyonu ile dosyaya ``0`` byte yazılmak istendiğinde gerçek bir yazma yapılmaz. ``write``
fonksiyonu bu durumda yazma için gerekli kontrolleri yapar (örneğin dosyanın yazma modunda açılıp açılmadığı
gibi). Eğer bu kontrollerde başarısızlık oluşursa ``write`` başarısız olur ve ``-1`` değeriyle geri döner.
Eğer bu kontrollerde başarısızlık oluşmazsa ``write`` ``0`` ile geri döner. Ancak yukarıda da belirttiğimiz
gibi bu durumda gerçek bir yazma yapılmamaktadır. POSIX standartları normal dosyaların dışında (yani *regular*
olmayan dosyaların dışında) 0 byte yazma işlemini *unspecified* olarak belirtmiştir. Dolayısıyla ileride
göreceğimiz boru gibi dosyalara 0 byte yazıldığında ne olacağı o sisteme bağlı bir durumdur.

Aşağıdaki programda klavyeden (``stdin`` dosyasından) yazılar alınıp ``write`` fonksiyonu ile dosyaya
yazdırılmıştır. ``fgets`` fonksiyonunun ``'\n'`` karakterini de diziye yerleştirdiğini anımsayınız.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <fcntl.h>
    #include <unistd.h>

    #define BUFFER_SIZE         4096

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;
        char buf[BUFFER_SIZE];

        if ((fd = open("test.txt", O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH)) == -1)
            exit_sys("open");

        for (;;) {
            printf("Enter text:");
            fflush(stdout);
            if (fgets(buf, BUFFER_SIZE, stdin) == NULL)
                continue;
            if (!strcmp(buf, "quit\n"))
                break;
            if (write(fd, buf, strlen(buf)) == -1)
                exit_sys("write");
        }

        close(fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

Dosya Kopyalama
================

UNIX/Linux sistemlerinde dosya kopyalama tipik olarak bir döngü içerisinde kaynak dosyadan hedef dosyaya blok
blok okuma yazma işlemi ile yapılmaktadır. Ancak bazı UNIX türevi işletim sistemleri dosya kopyalama işlemi
için sistem fonksiyonları da bulundurabilmektedir. Örneğin Linux sistemlerinde ``sys_copy_file_range`` isimli
sistem fonksiyonu doğrudan disk üzerinde blok kopyalaması yoluyla, hiç kullanıcı modunda işlem yapmadan dosya
kopyalamasını gerçekleştirebilmektedir. Ancak dosya kopyalama işleminin taşınabilir yolu yukarıda belirttiğimiz
gibi kaynaktan hedefe aktarım yapmaktır.

Peki bu kopyalama işleminde hangi büyüklükte bir tampon kullanılmalıdır? Tipik olarak bunun için dosya
sistemindeki blok uzunluğu tercih edilir. ``stat``, ``fstat``, ``lstat`` gibi fonksiyonlar bu bilgiyi bize
verirler. Bu fonksiyonları izleyen paragraflarda ele alacağız. Kopyalama sırasında kaynak dosyanın ``O_RDONLY``
bayrağı ile, hedef dosyanın ise ``O_WRONLY|O_CREAT|O_TRUNC`` bayrağı ile açılması uygun olur. Ancak hedef
dosya varsa onu ezmemek için hedef dosya ``O_WRONLY|O_CREAT|O_EXCL`` bayraklarıyla da açılabilir. Kopyalama
tipik olarak aşağıdaki gibi bir döngüyle yapılmaktadır:

.. code-block:: c

    if ((fds = open(argv[1], O_RDONLY)) == -1)
        exit_sys(argv[1]);

    if ((fdd = open(argv[2], O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH)) == -1)
        exit_sys(argv[2]);

    while ((result = read(fds, buf, BUFFER_SIZE)) > 0)
        if (write(fdd, buf, result) == -1)
            exit_sys(argv[2]);

    if (result == -1)
        exit_sys(argv[1]);

    close(fds);
    close(fdd);

Kopyalama yapılırken aslında hedef dosyanın erişim hakları kaynak dosyanın aynısı olacak biçimde ayarlanmalıdır.
Ancak biz henüz o konuları görmedik. Bu nedenle aşağıdaki programla çalıştırılabilir bir dosyayı kopyalamaya
çalışırsak ``x`` hakkı hedef dosyada olmadığı için kopyasını çalıştıramayız.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <fcntl.h>
    #include <unistd.h>

    #define BUFFER_SIZE         8192

    void exit_sys(const char *msg);

    int main(int argc, char *argv[])
    {
        int fds, fdd;
        char buf[BUFFER_SIZE];
        ssize_t result;

        if (argc != 3) {
            fprintf(stderr, "wrong number of arguments!..\n");
            exit(EXIT_FAILURE);
        }

        if ((fds = open(argv[1], O_RDONLY)) == -1)
            exit_sys(argv[1]);

        if ((fdd = open(argv[2], O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH)) == -1)
            exit_sys(argv[2]);

        while ((result = read(fds, buf, BUFFER_SIZE)) > 0)
            if (write(fdd, buf, result) == -1)
                exit_sys(argv[2]);

        if (result == -1)
            exit_sys(argv[1]);

        close(fds);
        close(fdd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

``write`` çok seyrek de olsa başarılı olduğu halde talep edilen miktar kadar hedef dosyaya yazamayabilir.
Örneğin diskin dolu olması durumunda ya da bir sinyal oluşması durumunda ``write`` talep edilen miktar kadar
yazma yapamayabilir. Bu tür durumları diğer durumlardan ayırmak için ayrı bir kontrol yapmak gerekebilir:

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <unistd.h>

    #define BUFFER_SIZE        4096

    void exit_sys(const char *msg);

    int main(int argc, char *argv[])
    {
        char buf[BUFFER_SIZE];
        int fds, fdd;
        ssize_t size, result;

        if (argc != 3) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }

        if ((fds = open(argv[1], O_RDONLY)) == -1)
            exit_sys(argv[1]);

        if ((fdd = open(argv[2], O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH)) == -1)
            exit_sys(argv[2]);

        while ((result = read(fds, buf, BUFFER_SIZE)) > 0) {
            if ((size = write(fdd, buf, result)) == -1)
                exit_sys("write");
            if (size != result) {
                fprintf(stderr, "cannot write file!...\n");
                exit(EXIT_FAILURE);
            }
        }

        if (result == -1)
            exit_sys("read");

        close(fds);
        close(fdd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

pread ve pwrite Fonksiyonları
======================================

``read`` ve ``write`` POSIX fonksiyonları yukarıda da belirttiğimiz gibi dosya göstericisinin gösterdiği yerden
itibaren okuma ve yazma işlemlerini yapmaktadır. Bu fonksiyonlar dosya göstericisinin konumunu okunan ya da
yazılan miktar kadar ilerletmektedir. İşte ``read`` ve ``write`` fonksiyonlarının ``pread`` ve ``pwrite``
biçiminde bir versiyonu daha bulunmaktadır. ``pread`` ve ``pwrite`` fonksiyonları işlemlerini dosya
göstericisinin gösterdiği yerden itibaren değil, parametreleriyle belirtilen offset'ten itibaren yapmaktadır.
Bu fonksiyonlar dosya göstericisinin konumunu değiştirmezler. Uygulamada ``pread`` ve ``pwrite`` fonksiyonları
seyrek kullanılmaktadır. Örneğin dosyanın farklı yerlerinden sürekli okuma/yazma yapıldığı durumlarda bu
fonksiyonlar kullanım kolaylığı sağlayabilmektedir. Fonksiyonların prototipleri şöyledir:

.. code-block:: c

    #include <unistd.h>

    ssize_t pread(int fildes, void *buf, size_t nbyte, off_t offset);
    ssize_t pwrite(int fildes, const void *buf, size_t nbyte, off_t offset);

``pread`` ve ``pwrite`` fonksiyonlarının ``read`` ve ``write`` fonksiyonlarından tek farkı ``offset``
parametresidir. Bu fonksiyonlar dosya göstericisinin gösterdiği yerden itibaren değil, son parametreleriyle
belirtilen yerden itibaren okuma ve yazma işlemini yaparlar. Fonksiyonların dosya göstericisinin konumunu
değiştirmediğine dikkat ediniz. Örneğin:

.. code-block:: c

    if ((result = pread(fd, buf, 10, 5)) == -1)
        exit_sys("pread");

Burada dosyanın 5. offset'inden itibaren 10 byte okunmuştur.

Dosyalara okuma yazma işlemi genellikle ardışıl bir biçimde yapıldığı için bu fonksiyonlar seyrek
kullanılmaktadır. Ancak örneğin veritabanı işlemlerinde dosyanın farklı offset'lerinden sıkça okuma ve yazmanın
yapıldığı durumlarda bu fonksiyonlar tercih edilebilmektedir.

``pread`` ve ``pwrite`` fonksiyonları da doğrudan ilgili sistem fonksiyonlarını çağırmaktadır (Linux
sistemlerinde ``sys_pread`` ve ``sys_pwrite``). Tabii bu işlemler önce dosya göstericisini saklayıp, sonra
konumlandırıp, sonra ``read``/``write`` işlemlerini yapıp, sonra da yeniden dosya göstericisini eski konumuna
yerleştirmekle de yapılabilir. Ancak ``pread`` ve ``pwrite`` işlemlerini yapan sistem fonksiyonları bu biçimde
değil, daha doğrudan aynı işlemi yapmaktadır.

----

lseek Fonksiyonu
=====================

Daha önceden de belirttiğimiz gibi dosya göstericisi dosya açıldığında 0. offset'tedir. Ancak okuma ve yazma
yapıldığında okunan ya da yazılan miktar kadar otomatik ilerletilmektedir. Dosya göstericisini belli bir konuma
yerleştirmek için ``lseek`` isimli POSIX fonksiyonu kullanılmaktadır. Bu fonksiyon da pek çok işletim
sisteminde doğrudan dosyayı konumlandıran sistem fonksiyonunu (Linux'ta ``sys_lseek``) çağırmaktadır. ``lseek``
fonksiyonunun genel kullanımı ``fseek`` standart C fonksiyonuna çok benzemektedir. Fonksiyonun prototipi
şöyledir:

.. code-block:: c

    #include <unistd.h>

    off_t lseek(int fd, off_t offset, int whence);

Fonksiyonun birinci parametresi dosya göstericisi konumlandırılacak dosyaya ilişkin dosya betimleyicisini
belirtir. Dosya göstericisi dosya nesnesinin (Linux'ta ``struct file`` yapısının) içerisinde tutulmaktadır.
İkinci parametre konumlandırma offset'ini belirtir. ``off_t`` türü ``<unistd.h>`` ve ``<sys/types.h>``
içerisinde işaretli bir tamsayı türü biçiminde typedef edilmiş olmak zorundadır. Fonksiyonun üçüncü parametresi
konumlandırma orijinini belirtmektedir. Bu üçüncü parametre ``0``, ``1`` ya da ``2`` olarak girilebilir. Tabii
sayısal değer girmek yerine ``SEEK_SET`` (0), ``SEEK_CUR`` (1) ve ``SEEK_END`` (2) sembolik sabitlerini
kullanabiliriz. Bu sembolik sabitler ``<unistd.h>`` ve ``<stdio.h>`` dosyaları içerisinde tanımlanmıştır.
Fonksiyon başarı durumunda dosyanın başından itibaren konumlandırılan offset'e, başarısızlık durumunda ``-1``
değerine geri dönmektedir.

``SEEK_SET`` konumlandırmanın dosyanın başından itibaren yapılacağını, ``SEEK_CUR`` o anda dosya göstericisinin
gösterdiği yerden itibaren yapılacağını, ``SEEK_END`` ise EOF durumundan itibaren yapılacağını belirtmektedir.
En normal durum ``SEEK_SET`` orijininde ikinci parametrenin ``>= 0``, ``SEEK_END`` orijininde ``<= 0`` biçiminde
girilmesidir. ``SEEK_CUR`` orijininde ikinci parametre pozitif ya da negatif girilebilir. Pozitif, bulunulan
yerden ileriye doğru; negatif ise bulunulan yerden geriye doğru anlamına gelmektedir. Örneğin dosya göstericisini
EOF durumuna şöyle konumlandırabiliriz:

.. code-block:: c

    lseek(fd, 0, SEEK_END);

``lseek`` fonksiyonunun başarısı bariz durumlarda genellikle programcı tarafından kontrol edilmemektedir.
Örneğin yukarıdaki çağrıda zaten disk dosyalarında (*regular files*) dosya göstericisinin EOF durumuna
konumlandırılamaması mümkün değildir. Yukarıdaki çağrının başarısız olmasının tek nedeni geçersiz bir
betimleyicinin kullanılmış olmasıdır.

Dosya sistemine de bağlı olarak UNIX/Linux sistemleri dosya göstericisini EOF'un ötesine konumlandırmaya izin
verebilmektedir. Bu özel bir durumdur. Bu tür durumlarda dosyaya yazma yapıldığında *dosya delikleri (file
holes)* oluşmaktadır. Dosya delikleri konusu ileride ele alınacaktır.

Aslında dosya açarken ``O_APPEND`` modu atomik bir biçimde her ``write`` işleminden önce dosya göstericisini
EOF durumuna çekmektedir. Bu nedenle her yazılan dosyanın sonuna eklenmektedir.

Aşağıdaki örnekte ``test.txt`` dosyası ``O_WRONLY`` modunda açılmış ve dosya göstericisi EOF durumuna çekilerek
dosyaya ekleme yapılmıştır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;
        char buf[] = "\nthis is a test";

        if ((fd = open("test.txt", O_WRONLY)) == -1)
            exit_sys("open");

        lseek(fd, 0, SEEK_END);

        if (write(fd, buf, strlen(buf)) == -1)
            exit_sys("write");

        close(fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

Dosya İşlemleri İçin Hangi Fonksiyonlar Kullanılmalı?
======================================================

Bir C/C++ programcısı olarak UNIX/Linux sistemlerinde dosya işlemleri yapmak için üç seçenek söz konusu
olabilir:

1. C'nin ya da C++'ın standart dosya fonksiyonlarını kullanmak.
2. POSIX dosya fonksiyonlarını kullanmak.
3. Sistem fonksiyonlarını kullanmak.

Burada en taşınabilir olan standart C/C++ fonksiyonlarıdır. Dolayısıyla ilk tercih bunlar olmalıdır. Ancak C
ve C++'ın standart dosya fonksiyonları spesifik bir sistemin gereksinimini karşılayacak biçimde tasarlanmamıştır.
Bu nedenle bazen doğrudan POSIX fonksiyonlarının kullanılması gerekebilmektedir. Genellikle dosya işlemleri
yapan sistem fonksiyonlarının kullanılması hiç gerekmez. Çünkü Linux'ta olduğu gibi pek çok UNIX türevi
sistemde yukarıda da belirttiğimiz gibi POSIX fonksiyonları zaten doğrudan sistem fonksiyonlarını
çağırmaktadır. Biz kursumuzda dosya işlemlerini daha çok POSIX fonksiyonlarını kullanarak gerçekleştireceğiz.

----

read ve write İşlemlerinde Atomiklik
=====================================

POSIX standartlarına göre disk dosyalarına yapılan ``read`` ve ``write`` işlemleri sistem genelinde atomiktir.
Yani örneğin iki program aynı anda aynı dosyanın aynı yerine yazma yapsa bile iç içe geçme oluşmaz. Önce
birisi yazar daha sonra diğeri yazar. Tabii hangi prosesin önce yazacağını bilemeyiz. Ancak burada önemli olan
nokta iç içe geçmenin olmamasıdır. Benzer biçimde bir ``read`` ile bir disk dosyasının bir yerinden ``n`` byte
okumak istediğimizde, başka bir proses aynı dosyanın aynı yerine yazma yaptığında biz ya o prosesin
yazdıklarını okuruz ya da onun yazmadan önceki dosya içeriğini okuruz. Yarısı eski yarısı yeni bir bilgi
okumayız.

Ancak işletim sistemi farklı ``read`` ve ``write`` çağrılarını bu anlamda senkronize etmemektedir. Yani örneğin
biz bir dosyanın belli bir yerine iki farklı ``write`` fonksiyonu ile ardışık şeyler yazdığımızı düşünelim.
Birinci ``write`` işleminden sonra başka bir proses artık orayı değiştirebilir. Dolayısıyla bu anlamda bir iç
içe girme durumu oluşabilir. Veritabanı programlarında bu tür durumlarla sık karşılaşılmaktadır. Örneğin
veritabanı programı bir kaydı *data* dosyasına yazıp ona ilişkin indeksleri de *index* dosyasına yazıyor
olabilir. Bu durumda iki ``write`` işlemi söz konusudur. Data dosyasına bilgiler yazıldıktan sonra henüz indeks
dosyasına bilgi yazılmadan başka bir proses bu iki işlemi hızlı davranarak yaparsa *data* ve *indeks* bütünlüğü
bozulabilir. İşletim sisteminin burada bir sorumluluğu yoktur. Bu tarz işlemlerde senkronizasyon programcılar
tarafından sağlanmak zorundadır. Bu tür senkronizasyonlar senkronizasyon nesneleriyle (semaphore gibi, mutex
gibi) dosya bütününde yapılabilir. Ancak tüm dosyaya erişimin engellenmesi iyi bir teknik değildir. İşte bu
tür durumlar için işletim sistemleri çekirdeğe entegre edilmiş olan *dosya kilitleme (file locking)* mekanizması
bulundurmaktadır. Dosya kilitleme tüm dosyayı değil dosyanın belli offset'lerine erişimi engelleme amacındadır.

----

umask Değeri ve umask Fonksiyonu
=========================================

Biz ``open`` fonksiyonu ile bir dosya yaratırken yaratacağımız dosyaya verdiğimiz erişim hakları dosyaya tam
olarak yansıtılmayabilir. Yani örneğin biz gruba ``w`` hakkı vermek istesek bile bunu sağlayamayabiliriz. Çünkü
belirtilen erişim değerlerini maskeleyen (yani ortadan kaldıran) bir mekanizma vardır. Buna prosesin *umask
değeri* denilmektedir. Prosesin umask değeri ``mode_t`` türü ile ifade edilir; sahiplik, grupluk ve diğerlik
bilgilerini içerir. Bu bilgiler aslında maskelenecek değerleri belirtmektedir.

Örneğin prosesin umask değerinin ``S_IWGRP|S_IWOTH`` olduğunu varsayalım. Bu umask değeri *biz ``open``
fonksiyonu ile bir dosyayı yaratırken grup için ve diğerleri için ``w`` hakkı versek bile bu hak dosyaya
yansıtılmayacak* anlamına gelmektedir. Eğer prosesin umask değeri ``0`` ise bu durumda maskelenecek bir şey
yoktur; dolayısıyla verilen hakların hepsi dosyaya yansıtılır. Prosesin umask değerinin ``umask`` olduğunu,
dosyaya vermek istediğimiz erişim haklarının da ``mode`` olduğunu varsayalım. (Yani ``mode``, ``S_IXXX`` gibi
tek biti ``1`` olan değerlerin bit düzeyinde OR'lanması ile oluşturulmuş değer olsun.) Bu durumda dosyaya
yansıtılacak erişim hakları ``mode & ~umask`` olacaktır. Yani prosesin umask değerindeki ``1`` olan bitler
maskelenecek erişim haklarını belirtmektedir.

Prosesin başlangıçtaki umask değeri üst prosesten aktarılmaktadır. Örneğin biz kabuktan program çalıştırırken
çalıştırdığımız programın umask değeri, kabuğun (örneğin ``bash`` prosesinin) umask değeri olarak bizim
prosesimize geçirilecektir. Kabuğun umask değeri ``umask`` isimli komutla elde edilebilir. Kabuğun umask
değeri genellikle ``0022`` ya da ``0002`` gibi bir değerde olur. Buradaki basamaklar octal sayı (sekizlik
sistemde sayı) belirtmektedir. Bir octal digit 3 bitle açılmaktadır. Dolayısıyla bu bitler maskelenecek erişim
haklarının durumunu belirtir:

.. code-block:: text

    ? owner group other

En yüksek anlamlı octal digit (``?`` ile gösterilenin) şimdiye kadar görmediğimiz başka haklarla ilgilidir. Bu
haklara *set user id*, *set group id* ve *sticky* hakları denilmektedir. Ancak diğer üç octal digit sırasıyla
owner, group ve other maskeleme bitlerini belirtmektedir.

``umask`` komutuyla aynı zamanda kabuğun umask değeri de değiştirilebilir. Bu durumda yine değiştirme değerleri
octal digit biçiminde verilmelidir. Örneğin:

.. code-block:: bash

    $ umask 022

Burada en yüksek anlamlı octal digit verilmediğine göre o digit ``0`` kabul edilir. O halde burada belirtilen
umask değeri grup için ve diğerleri için ``w`` hakkını maskeleyecektir. (Zaten pek çok kabukta umask değerinin
varsayılan durumu böyledir.) Bazen programcı umask değerini tamamen sıfırlamak da isteyebilir. Bu işlem şöyle
yapılabilir:

.. code-block:: bash

    $ umask 0

Burada yüksek anlamlı üç octal digit de ``0`` kabul edilmektedir. Bu durumda artık çalıştırdığımız programda
``open`` fonksiyonunun tüm erişim hakları dosyalara yansıtılacaktır.

Bir proses başlangıçta umask değerini üst prosesten almaktadır. Ancak proses istediği zaman ``umask`` isimli
POSIX fonksiyonu ile kendi umask değerini değiştirebilmektedir. ``umask`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <sys/stat.h>

    mode_t umask(mode_t cmask);

Fonksiyon belirtilen değerle prosesin umask değerini set eder ve prosesin eski umask değerine geri döner.
Fonksiyon başarısız olamaz. ``umask`` fonksiyonu ile kendi prosesimizin umask değerini almak için onu
değiştirmemiz gerekir. Bunu aşağıdaki gibi bir kodla yapabiliriz:

.. code-block:: c

    mode_t mode;

    mode = umask(0);
    printf("%03jo\n", (intmax_t)mode);
    umask(mode);

Tabii programcı ``umask`` fonksiyonuna octal değerler de girebilir. Ancak daha önceden de bahsedildiği gibi
POSIX standartlarında 2008'de ``S_IXXX`` sembolik sabitlerinin değerleri octal değerlerle eşleştirilmiştir.
Örneğin:

.. code-block:: c

    umask(0022);                /* Eskiden bu biçimde belirleme taşınabilir değildi, eski sistemlerde dikkat edilmesi gerekir */
    umask(S_IWGRP|S_IWOTH);     /* Bu biçimde belirleme daha okunabilirdir. */

Aşağıdaki örnekte prosesin umask değeri önce sıfırlanmış, sonra bir dosya yaratılmıştır. ``open`` fonksiyonunda
verilen erişim hakları artık dosyaya tamamen yansıtılacaktır. Ayrıca bu programda prosesin eski umask değeri
de yazdırılmaktadır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <stdint.h>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;
        mode_t oldmask;

        oldmask = umask(0);

        if ((fd = open("test.txt", O_WRONLY|O_CREAT|O_EXCL, S_IRUSR|S_IWUSR|S_IRGRP|S_IWGRP|S_IROTH|S_IWOTH)) == -1)
            exit_sys("open");

        printf("%03jo\n", (intmax_t)oldmask);
        umask(oldmask);

        close(fd);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

----

Kabuk Programına umask Komutunun Eklenmesi
===============================================

Şimdi daha önce yazmış olduğumuz kabuk programına ``umask`` komutunu ekleyelim:

.. code-block:: c

    void umask_proc(void)
    {
        mode_t oldmask;
        long mode;

        if (g_nparams == 1) {
            oldmask = umask(0);
            printf("%04jo\n", (intmax_t)oldmask);
            umask(oldmask);
        }
        else if (g_nparams == 2) {
            if (!isoctal(g_params[1])) {
                printf("invalid mask...\n");
                return;
            }
            mode = strtol(g_params[1], NULL, 8);
            umask(mode);
        }
        else
            printf("too many arguments!..\n");
    }

Burada komut argüman almamışsa prosesin umask değeri doğrudan yazdırılmıştır. Eğer komut tek argüman almışsa
önce girilen umask değerinin geçerliliği sınanmış, sonra prosesin umask değeri set edilmiştir. Geçerlilik
sınaması aşağıdaki gibi basit bir fonksiyonla yapılmıştır:

.. code-block:: c

    int isoctal(const char *str)
    {
        if (strlen(str) > 4)
            return 0;
        while (*str != '\0') {
            if (*str < '0' || *str > '7')
                return 0;
            ++str;
        }
        return 1;
    }

Programın tamamı şöyledir:

.. code-block:: c

    /* myshell.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <stdint.h>
    #include <string.h>
    #include <errno.h>
    #include <sys/stat.h>
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
    void umask_proc(void);

    int isoctal(const char *str);
    void exit_sys(const char *msg);

    struct cmd g_cmds[] = {
        {"rm", rm_proc},
        {"cp", cp_proc},
        {"mv", mv_proc},
        {"cd", cd_proc},
        {"umask", umask_proc},
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

    void umask_proc(void)
    {
        mode_t oldmask;
        long mode;

        if (g_nparams == 1) {
            oldmask = umask(0);
            printf("%04jo\n", (intmax_t)oldmask);
            umask(oldmask);
        }
        else if (g_nparams == 2) {
            if (!isoctal(g_params[1])) {
                printf("invalid mask...\n");
                return;
            }
            mode = strtol(g_params[1], NULL, 8);
            umask(mode);
        }
        else
            printf("too many arguments!..\n");
    }

    int isoctal(const char *str)
    {
        if (strlen(str) > 4)
            return 0;
        while (*str != '\0') {
            if (*str < '0' || *str > '7')
                return 0;
            ++str;
        }
        return 1;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }



==================
**exec İşlemleri**
==================

Bu bölümde bir programın başka bir programı nasıl yükleyip çalıştırdığı üzerinde duracağız. ``fork`` işlemi yeni bir prosesin 
yaratılmasına yol açmaktadır. ``exec`` işlemleri ise yaratılmış olan prosesin başka bir program koduyla çalışmasına devam etmesini 
sağlamaktadır. Kabuk programları da ``exec`` işlemleri yoluyla programları çalıştırmaktadır. 

exec Fonksiyonları
==================

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

Ayrıca POSIX standartlarında tanımlı olmasa da GNU C kütüphanesinde ``execvpe`` isimli bir ``exec`` fonksiyonu da
bulunmaktadır. (Bu fonksiyon *glibc* kütüphanesinde olduğu için bu kütüphanenin kullanıldığı BSD gibi diğer UNIX
türevi sistemlerde de bulunmaktadır.) Ayrıca Linux sistemlerine özgü bir biçimde ``sys_execveat`` isimli bir sistem
fonksiyonu da bulunmaktadır. Linux'ta bu fonksiyon ``execveat`` ismiyle kullanılabilmektedir.

Aslında UNIX/Linux sistemleri bu ``exec`` fonksiyonlarının hepsini sistem fonksiyonu biçiminde bulundurmamaktadır.
Örneğin Linux sistemlerinde ``execve`` fonksiyonu bir sistem fonksiyonu biçiminde (``sys_execve``) yazılmıştır.
Diğer ``exec`` fonksiyonları bu sistem fonksiyonunu çağıran kütüphane fonksiyonları biçiminde gerçekleştirilmiştir.
Yukarıda da belirttiğimiz gibi Linux'taki ``execveat`` fonksiyonu da bir sistem fonksiyonu biçiminde
(``sys_execveat``) gerçekleştirilmiştir. Bu durumda yukarıdaki POSIX fonksiyonları dışında Linux'a özgü olan exec
fonksiyonları şunlardır:

.. code-block:: text

    execvpe
    execveat

``exec`` fonksiyonları prosesin yaşamına başka bir program koduyla devam etmesini sağlamaktadır. ``exec`` fonksiyonlarına
biz "çalıştırılabilen bir program dosyasını" argüman olarak veririz. ``exec`` fonksiyonları o anda çalışmakta olan
programın bellek alanını tamamen boşaltıp onun yerine bizim verdiğimiz program dosyasını belleğe yükler ve o
yüklediği programın kodunu çalıştırır. ``exec`` işlemi ile prosesin kontrol bloğundaki pek çok alan
değiştirilmemektedir. Yani prosesin ID'si, kullanıcı ve grup ID'leri, prosesin çalışma dizini vs. değişmez. exec
işlemleriyle prosesin yalnızca çalıştırdığı program dosyası değiştirilmektedir. Örneğin *"sample"* programının
içerisinde biz ``exec`` fonksiyonlarıyla *"other"* programını çalıştırmak istediğimizde *"sample"* programı bellekten
tamamen atılır, onun yerine *"other"* programının kodu ve verileri belleğe yüklenir ve *"other"* programının kodu
çalıştırılır. Yukarıda da belirttiğimiz gibi ``exec`` işlemi sırasında prosesin kontrol bloğundaki temel bilgiler
değişmez. Yani ``exec`` fonksiyonları uygulandığında proses yaşamına başka bir program koduyla devam etmektedir.

``exec`` fonksiyonlarının isimlerinin sonlarında bulunan ``l`` harfi (``execl``, ``execlp``) komut satırı argümanlarının tek
tek bir liste biçiminde, fonksiyonların isimlerinin sonundaki ``v`` harfi ise komut satırı argümanlarının bir dizi (vector)
biçiminde verileceğini belirtir. Fonksiyonların isimlerinin sonlarındaki ``p`` harfi (*path* sözcüğünden geliyor) aramanın
``PATH`` çevre değişkenine bakılarak yapılacağını, ``e`` harfi (*environment* sözcüğünden geliyor) ise prosesin çevre
değişkenlerinin ``exec`` işlemi sırasında değiştirileceği anlamına gelmektedir. Yukarıda da belirttiğimiz gibi Linux 
sistemlerinde ``execl``, ``execv``, ``execlp``, ``execvp`` ve ``execle`` fonksiyonları aslında ``execve`` fonksiyonu 
(``sys_execve`` sistem fonksiyonu) çağrılarak, ``fexecve`` fonksiyonu ise ``execveat`` fonksiyonu (``sys_execveat`` 
sistem fonksiyonu) çağrılarak gerçekleştirilmiştir. Biz burada bu fonksiyonların üzerinde tek tek duracağız.

``exec`` fonksiyonları başarı durumunda geri dönmezler. Çünkü zaten başarı durumunda bu fonksiyonlar başka bir programı
yüklemiş ve çalıştırmış durumda olurlar. Bu fonksiyonlar başarısızlık durumunda yine ``-1`` değerine geri dönerler ve
``errno`` değişkeni uygun biçimde set edilir.

execl Fonksiyonu
================

``execl`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int execl(const char *path, const char *arg0, ... /*, (char *)0 */);

Fonksiyonun birinci parametresi çalıştırılacak olan program dosyasının yol ifadesini almaktadır. Bu yol ifadesi mutlak ya
da göreli olabilir. Fonksiyonun diğer parametreleri sırasıyla çalıştırılacak programa geçirilecek komut satırı
argümanlarının listesini belirtir. Birinci komut satırı argümanının (``argv[0]``) her zaman program ismi olacak biçimde
oluşturulması genel bir beklenti ve C standartlarında öngörülen bir durumdur. Programcı ``exec`` uygularken bunu sağlamak
zorunda değildir. Ancak bunun sağlanmaması kötü bir tekniktir ve çalıştırılacak programların hatalı çalışmasına yol
açabilir. Fonksiyon değişken sayıda (``...`` parametresine dikkat ediniz) argüman aldığı için argüman listesinin sonunda
``NULL`` adresin bulunması gerekmektedir. Ancak C'de "default argüman dönüştürmesi (default argument conversion)" denilen
kurala göre eğer argümanın karşılığında bir parametre türü belirtilmemişse "int türünden küçük türler int türüne, float
türü ise double türüne dönüştürülerek" fonksiyona yollanmaktadır. Burada programcının ``NULL`` adres sabitini yalnızca
``NULL`` sembolik sabiti biçiminde ya da ``0`` biçiminde belirtmemesi gerekir. Çünkü ``NULL`` sembolik sabiti düz sıfır
olarak da define edilmiş olabilir. Bu durumda düz ``0`` sabiti int olarak fonksiyona yollanır. Uygun olan durum düz sıfır
değerinin ya da ``NULL`` sembolik sabitinin bir adres türüne (tipik olarak ``char *`` türüne) dönüştürülerek fonksiyona
aktarılmasıdır. (C23 ile C'ye de eklenen ``nullptr`` sabitini hiç dönüştürme yapmadan kullanabilirsiniz.) Yukarıda da
belirtildiği gibi ``exec`` fonksiyonları başarı durumunda zaten geri dönmezler. Başarısızlık durumunda ``-1`` değerine geri
dönerler ve ``errno`` değişkeni uygun biçimde set edilir. ``execl`` fonksiyonunun çağrılması tipik olarak şöyle
yapılmaktadır:

.. code-block:: c

    if (execl("/bin/ls", "/bin/ls", "-l", "-i", (char *)0) == -1)
        exit_sys("execl");

    /* unreachable code */

Burada ``execl`` ile ``/bin/ls`` dosyası çalıştırılmak istenmiştir. Diğer argümanlar bu programın ``main`` fonksiyonuna
``argv`` parametresi olarak geçirilecek olan komut satırı argümanlarını belirtmektedir.

``exec`` fonksiyonları çeşitli nedenlerle başarısız olabilir. Örneğin çalıştırılacak program dosyası bulunamayabilir, bulunduğu
halde proses dosya için ``'x'`` hakkına sahip olmayabilir, çalıştırılabilen dosyanın formatı bozulmuş olabilir. Başarısızlık
durumunda ``errno`` değişkeni uygun biçimde set edilmektedir.

Aşağıdaki örnekte *"sample"* programı *"other"* isimli başka bir programı çalıştırmaktadır. *"sample"* programı
çalıştırıldığında ekrana (``stdout`` dosyasına) şu yazılar basılacaktır:

.. code-block:: console

    $ ./sample
    sample running...
    other running...
    argv[0]: other
    argv[1]: ali
    argv[2]: veli
    argv[3]: selami
    other ends...

``sample.c```

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        printf("sample running...\n");

        if (execl("other", "other", "ali", "veli", "selami", (char *)0) == -1)
            exit_sys("execl");

        printf("unreachable code...\n");

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

    int main(int argc, char *argv[])
    {
        printf("other running...\n");

        for (int i = 0; i < argc; ++i)
            printf("argv[%d]: %s\n", i, argv[i]);

        printf("other ends...\n");

        return 0;
    }

Mademki ``exec`` fonksiyonları başarılı olduğunda zaten geri dönmemektedir, o halde ``exec`` işlemi aşağıdaki gibi de
yapılabilir:

.. code-block:: c

    execl(...);
    exit_sys("execl");

Burada ``exec`` fonksiyonları zaten başarılı olduğunda akış aşağıya geçmeyecektir, başarısız olduğunda akış aşağıya
geçecektir. Bu durumda başarı kontrolü yapmaya aslında gerek yoktur. Fakat biz kursumuzda genel olarak ``exec`` işlemlerini
aşağıdaki gibi uygulayacağız:

.. code-block:: c

    if (execl(...) == -1)
        exit_sys("execl");

fork ve exec İşlemlerinin Birlikte Uygulanması: fork/exec Kalıbı
================================================================

``exec`` işleminin tek başına uygulanması mevcut programı bellekten atarak başka bir programı çalıştırmaktadır. Ancak
genellikle programcı kendi programının da devam etmesini ister. İşte eğer biz hem başka bir programı çalıştırmak
istiyorsak hem de kendi programımızın devam etmesini istiyorsak bu durumda ``fork`` ve ``exec`` işlemlerini birlikte
uygulamamız gerekir.

``fork`` işlemi ile yeni bir proses yaratılıp yaratılan yeni proses üst proses ile aynı kodu çalıştırıyordu. exec
işleminde ise prosesin bellek alanı atılıp başka bir program dosyası belleğe yükleniyordu. Peki biz hem kendi programımız
devam etsin hem de başka bir programı da çalıştıralım istiyorsak bunu nasıl yapabiliriz? İşte bu durumda yalnızca
``fork`` ya da yalnızca ``exec`` işe yaramamaktadır. ``fork`` ve ``exec`` fonksiyonlarının birlikte kullanılması gerekmektedir.
Şöyle ki: Programcı önce ``fork`` yapar, sonra alt proseste ``exec`` işlemini uygular. Yani başka bir programın kodunu alt
proses çalıştırmış olur. Bu işlem tipik olarak şöyle yapılmaktadır:

.. code-block:: c

    pid_t pid;

    if ((pid = fork()) == -1)
        exit_sys("fork");

    if (pid == 0) {
        if (exec(...) == -1)            /* dikkat! exec demekle ailedeki herhangi bir fonksiyonu kastediyoruz */
            exit_sys("exec");
    }

    /* Yalnızca üst prosesin akışı buraya gelir */

Burada ``exec`` başarılı olursa zaten artık alt prosesin bellek alanı boşaltılıp yeni program yüklenecektir. ``exec`` başarısız
olduğunda da alt proses sonlandırılmıştır. Tabii yukarıdaki kalıp ``&&`` operatörüyle şöyle de oluşturulabilmektedir:

.. code-block:: c

    pid_t pid;

    if ((pid = fork()) == -1)
        exit_sys("fork");

    if (pid == 0 && exec(...) == -1)        /* dikkat! exec demekle ailedeki herhangi bir fonksiyonu kastediyoruz */
        exit_sys("exec");

    /* Yalnızca üst prosesin akışı buraya gelir */

Tabii ``exec`` yapılmış olsa da üst prosesin yine alt prosesi ``wait`` fonksiyonlarıyla beklemesi gerekmektedir.
Çalıştırılan programa ilişkin prosesin üst prosesi yine ``fork`` işlemi yapan prosestir. Örneğin:

.. code-block:: c

    pid_t pid;

    if ((pid = fork()) == -1)
        exit_sys("fork");

    if (pid == 0 && exec(...) == -1)        /* dikkat! exec demekle ailedeki herhangi bir fonksiyonu kastediyoruz */
        exit_sys("exec");

    /* Yalnızca üst prosesin akışı buraya gelir */

    if (waitpid(pid, NULL, 0) == -1)
        exit_sys("waitpid");

Bazen ``fork`` işleminden sonra programcı alt proseste bazı ayarlamalar yaptıktan sonra ``exec`` uygulamak isteyebilir.
Örneğin:

.. code-block:: c

    pid_t pid;

    if ((pid = fork()) == -1)
        exit_sys("fork");

    if (pid == 0) {
        /* alt proseste bazı işlemler */
        if (exec(...) == -1)                /* dikkat! exec demekle ailedeki herhangi bir fonksiyonu kastediyoruz */
            exit_sys("exec");
        /* unreachable code */
    }

    /* Yalnızca üst prosesin akışı buraya gelir */

    if (waitpid(pid, NULL, 0) == -1)
        exit_sys("waitpid");

Aslında ``fork`` ve ``exec`` nadiren tek başına uygulanmaktadır. Genellikle ``fork`` ve ``exec`` bir arada yukarıdaki kalıp
eşliğinde uygulanmaktadır.

Aşağıdaki örnekte üst proses ``/bin/ls`` programını çalıştırıp yoluna devam etmektedir:

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        pid_t pid;

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid == 0 && execl("/bin/ls", "/bin/ls", "-l", (char *)0) == -1)
            exit_sys("execl");

        for (int i = 0; i < 10; ++i) {
            printf("sample continues: %d\n", i);
            sleep(1);
        }

        if (waitpid(pid, NULL, 0) == -1)
            exit_sys("waitpid");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

``fork``/``exec`` işlemlerinde kişilerin kafasını karıştıran bir durum oluşmaktadır. Kişiler haklı olarak şöyle düşünmektedir:
"fork işlemi ile üst prosesin bellek alanı alt proses için kopyalandığına göre ve alt proseste de ``exec`` yapıldığında alt
prosesin bellek alanı hemen boşaltılacağına göre burada üst prosesin bellek alanı gereksiz biçimde alt prosese
kopyalanmış olmuyor mu?" Gerçekten de ilk bakışta böyle bir durum söz konusu gibi gözükmektedir. Ancak modern
işlemcilerin "sayfalama (paging)" mekanizmaları sayesinde aslında ``fork`` işlemi sırasında *copy-on-write* mekanizması
işletilmektedir. Yani aslında bugün kullandığımız işlemcilerde ``fork`` işlemi sırasında işletim sistemi üst prosesin
bellek alanını zaten alt prosese bütünsel olarak kopyalamamaktadır. Kopyalama işlemi aslında *gerektiğinde*
yapılmaktadır. Bu mekanizmaya *copy-on-write* denilmektedir. Bu konuda bilgiler ileride verilecektir. Ancak bazı eski
sistemlerde *copy-on-write* mekanizması ya yoktu ya da etkin olarak gerçekleştirilemiyordu. Yani eski sistemlerde
yukarıdaki durum gerçekten etkinlik bakımından bir problem oluşturuyordu. Bu nedenle bu eski sistemler zamanında
``fork`` fonksiyonunun bellek kopyalamasını yapmayan (ya da minimal düzeyde yapan) ``vfork`` isminde bir benzeri de
bulundurulmuştur. ``vfork`` fonksiyonu eskiden POSIX standartlarında bulunuyordu. 2008'den itibaren POSIX
standartlarından kaldırılmıştır. Fakat *glibc* kütüphanesi bu fonksiyonu bulundurmaya devam etmektedir. Zaten yukarıda
da belirttiğimiz gibi modern sistemlerde artık ``vfork`` fonksiyonuna gereksinim de kalmamıştır. ``vfork`` tamamen
``fork`` işlemi yapar. Ancak üst prosesin bellek alanını alt prosese kopyalamaz. Çünkü ``vfork`` fonksiyonu ``exec`` için
düşünülmüştür. Yani ``vfork`` işleminden sonra ``exec`` yapılmalıdır. Eğer ``vfork`` işleminden sonra ``exec`` yapılmayıp sanki
``fork`` yapılmış gibi program devam ettirilirse "tanımsız davranış (undefined behavior)" oluşmaktadır. ``vfork``
fonksiyonunun prototipi ``fork`` ile aynı biçimdedir:

.. code-block:: c

    #include <unistd.h>

    pid_t vfork(void);

Eski POSIX standartlarına göre ``vfork`` işleminden sonra yalnızca ``_exit`` fonksiyonu ya da ``exec`` fonksiyonları
çağrılabilir. Bunun dışında başka bir fonksiyon çağrılamaz. Yani ``vfork`` başarılı ise biz ya ``_exit`` fonksiyonu ile
prosesi sonlandırmalıyız ya da ``exec`` uygulamalıyız. Tabii ``exec`` de başarısız olursa ``_exit`` ile (``exit`` ile değil) alt
prosesi sonlandırmalıyız. Başka bir fonksiyonun kullanılamamasının nedeni o fonksiyonların kodlarının alt prosese
kopyalanmamış olmasıdır.
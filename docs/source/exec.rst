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

execv Fonksiyonu
================
 
``execv`` fonksiyonu ``execl`` fonksiyonu ile aynı işlevselliğe sahiptir.. Ancak bu fonksiyon çalıştırılacak
programın komut satırı argümanlarını bir gösterici dizisi biçiminde almaktadır. Fonksiyonun prototipi
şöyledir:
 
.. code-block:: c
 
    #include <unistd.h>
 
    int execv(const char *path, char * const *argv);
 
Fonksiyonun birinci parametresi çalıştırılacak program dosyasının yol ifadesini, ikinci
parametresi ise komut satırı argümanlarının bulunduğu ``char`` türden gösterici dizisinin başlangıç
adresini almaktadır. Yani bizim komut satırı argümanlarını bir gösterici dizisine yerleştirip fonksiyona 
onun adresini vermemiz gerekir. Bu gösterici dizisinin son elemanında ``NULL`` adres bulunmalıdır. Tabii bu durumda
tür dönüştürmesi yapmaya gerek yoktur. Örneğin:
 
.. code-block:: c
 
    char *argv[] = {"/bin/ls", "-l", NULL};
    /* ... */
 
    execv("/bin/ls", argv);
    exit_sys("execv");
 
``execv`` fonksiyonunun ikinci parametresindeki ``const`` niteleyicisinin yerine dikkat ediniz. Buradaki
``const`` niteleyicisi adresi geçirilen gösterici dizisinin ``const`` olduğunu belirtmektedir. Yani
fonksiyon hem o gösterici dizisinde değişiklik yapmamaktadır.
 
Aşağıda ``execv`` fonksiyonunun kullanımına bir örnek verilmiştir.
 
.. code-block:: c
 
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>
 
    void exit_sys(const char *msg);
 
    int main(int argc, char *argv[])
    {
        pid_t pid;
        char *args[] = {"/bin/ls", "-l", "-i", NULL};
 
        if ((pid = fork()) == -1)
            exit_sys("fork");
 
        if (pid == 0 && execv("/bin/ls", args) == -1)
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
 
 
Peki ``execv`` ne zaman tercih edilebilir? İşte bazen ``execl`` fonksiyonu yerine ``execv``
fonksiyonunun kullanılması daha uygun olabilmektedir. Örneğin biz ``runprog`` isimli bir program yazalım.
Bu program da komut satırı argümanlarıyla aldığı programı çalıştırsın. Yani ``runprog`` programı şöyle
çalıştırılsın:
 
.. code-block:: console
 
    $ ./runprog /bin/ls -l -i
 
Eğer böyle bir programı ``execl`` ile yazmaya çalışırsak bunu pratik bir biçimde başaramayız. Çünkü
çalıştıracağımız programın kaç komut satırı argümanı ile çalıştırılacağını baştan bilmemekteyiz. Aşağıda
böyle bir programa örnek verilmiştir. Programı şöyle çalıştırabilirsiniz:
 
.. code-block:: console
 
    $ ./runprog /bin/ls -l -i
    $ ./runprog /bin/cp sample.c x.c
    $ ./runprog other ali veli selami
 
Programda ``exec`` çağrısına dikkat ediniz:
 
.. code-block:: c
 
    if ((pid = fork()) == -1)
        exit_sys("fork");
 
    if (pid == 0 && execv(argv[1], &argv[1]) == -1)
        exit_sys("execl");
 
Burada ``execv`` fonksiyonuna ``argv`` gösterici dizisinin 1'inci indeksli elemanının adresi
geçirilmiştir. ``argv`` dizisinin sonunda zaten ``NULL`` adres bulunduğunu anımsayınız:
 
.. figure:: _static/argv-array.png
    :align: center
    :width: 65%
 
``runprog.c``

.. code-block:: c
 
 
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>
 
    void exit_sys(const char *msg);
 
    int main(int argc, char *argv[])
    {
        pid_t pid;
 
        if (argc < 2) {
            fprintf(stderr, "wrong number of arguments!..\n");
            exit(EXIT_FAILURE);
        }
 
        if ((pid = fork()) == -1)
            exit_sys("fork");
 
        if (pid == 0 && execv(argv[1], &argv[1]) == -1)
            exit_sys("execv");
 
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
 
execlp ve execvp Fonksiyonları
==============================
 
``exec`` fonksiyonlarının iki p'li versiyonu da vardır: ``execlp`` ve ``execvp``. Bu ``p``'li versiyonların
prototipleri ``p``'siz versiyonlarla aynıdır. Yalnızca ilk parametrenin semantik anlamı farklıdır. Bunların
prototipleri şöyledir:
 
.. code-block:: c
 
    #include <unistd.h>
 
    int execlp(const char *file, const char *arg0, ... /*, (char *)0 */);
    int execvp(const char *file, char *const argv[]);
 
exec fonksiyonlarının ``p``'li versiyonları şöyle çalışmaktadır:
 
- Eğer bu fonksiyonların birinci parametrelerinde belirtilen dosya isminde hiç ``'/'`` karakteri
  kullanılmamışsa bu fonksiyonlar önce ``PATH`` çevre değişkeninin değerini ``getenv`` fonksiyonuyla
  elde edip buradaki yazıyı ``':'`` karakterlerinden parçalara ayırırlar (parse ederler). Bu ``:``
  karakterlerinin arasındaki yazıların dizin belirttiğini varsayarlar. Sonra ``exec`` yapılacak dosyayı
  sırasıyla bu dizinlerde ararlar. Eğer bulurlarsa onu ``exec`` yaparlar, bulamazlarsa bu fonksiyonlar
  başarısız olur. Tabii bu fonksiyonlar ``PATH`` çevre değişkeninde belirtilen dizinlerdeki aramayı
  baştan sona doğru yapmaktadır ve ilk bulduğu dizindeki programı ``exec`` işlemine sokmaktadır. (Yani eğer
  söz konusu program dosyası birden fazla ``PATH`` dizininde varsa dosyanın ilk bulunduğu dizindeki
  program çalıştırılır.) ``PATH`` çevre değişkeninin değeri aşağıdakine benzer bir biçimdedir:
 
.. code-block:: text
 
    /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games:/usr/local/games:/snap/bin
 
- Eğer ``p``'li ``exec`` fonksiyonlarının birinci parametresiyle belirtilen dosya isminde en az bir ``'/'``
  karakteri varsa bu durumda fonksiyonlar ``PATH`` çevre değişkenine başvurmazlar. Birinci parametresiyle
  belirtilen göreli ya da mutlak yol ifadesinden hareketle dosyanın yerini belirlemeye çalışırlar. Başka
  bir deyişle bu durumda fonksiyonların ``p``'li versiyonlarının ``p``'siz versiyonlarından hiçbir farkı
  kalmamaktadır. Örneğin:
 
.. code-block:: c
 
    execlp("ls", ...);          /* PATH çevre değişkenine başvurulur */
    execlp("./sample", ...);    /* PATH çevre değişkenine başvurulmaz */
    execlp("a/sanple", ...);    /* PATH çevre değişkenine başvurulmaz */
 
``exec`` fonksiyonlarının ``p``'li versiyonları eğer dosya isminde hiç ``'/'`` karakteri yoksa ve ``PATH``
dizinlerinde de dosyayı bulamazlarsa prosesin çalışma dizinine bakmamaktadır. Yani bu durumda bu
fonksiyonlar yalnızca ``PATH`` çevre değişkenindeki dizinlere bakmaktadır. Tabii ``PATH`` çevre
değişkeninde o andaki prosesin çalışma dizini ``'.'`` karakteri ile de belirtilebilir. Örneğin:
 
.. code-block:: text
 
    /bin:/usr/bin:/:.

 
Buradaki ``.`` prosesin çalışma dizinini belirtmektedir. Biz ``PATH`` çevre değişkeninin sonuna dizinler
ekleyebiliriz. Örneğin:
 
.. code-block:: console
 
    $ PATH=$PATH:/home/kaan
 
Tabii bunun kalıcı hale getirilmesi için kabuk programının startup dosyalarına yerleştirilmesi gerekir.
Prosesin çalışma dizininin ``PATH`` çevre değişkenine eklenmesi güvenlik zafiyeti nedeniyle iyi bir
teknik kabul edilmemektedir. Örneğin:
 
.. code-block:: console
 
    $ PATH=$PATH:.
 
Peki exec fonksiyonlarının p'li versiyonları ``PATH`` çevre değişkenini bulamazsa ne olur? POSIX
standartları bu durumdaki davranışın sistemden sisteme değişebileceğini (implementation dependent)
belirtmektedir. Pek çok sistem (örneğin Linux ve BSD) bu durumda sanki ``PATH`` çevre değişkeni
``/bin:/usr/bin`` biçimindeymiş gibi davranmaktadır.
 
p'li Versiyonların shebang'siz Dosyalarda Davranışı
---------------------------------------------------
 
exec fonksiyonlarının p'li versiyonları (``execlp`` ve ``execvp``) aramayı ``PATH`` dizinlerinde
sırasıyla yapmaktadır. Ancak bu fonksiyonlar dosyayı bir dizinde bulduğunda ve onu sistem fonksiyonuyla
(``execve``) çalıştırmaya çalıştığında başarısız olup ``EINVAL`` ve ``ENOEXEC`` errno değeri oluşursa
dosyanın bir kabuk betiği (shell script) olduğundan çalıştırılamadığı sonucunu çıkartmaktadır ve bu
durumda dosyayı ``/bin/sh`` (default shell) programı ile çalıştırmaktadır. Ancak exec fonksiyonlarının
diğer versiyonları ``EINVAL`` ve ``ENOEXEC`` errno değeri oluştuğunda bunu yapmamaktadır. Tabii bu
davranışı yalnızca exec fonksiyonlarının p'li versiyonları göstermektedir. exec fonksiyonlarının p'li
versiyonları ``PATH`` dizinlerinin birinde dosyayı sistem fonksiyonuyla (Linux'taki ``sys_execve``)
çalıştırmaya çalıştığında ``EACCES`` errno değeri ile başarısız olurlarsa dosyayı sonraki ``PATH``
dizinlerinde aramaya devam ederler. Ancak bu arama sırasında bu fonksiyonlar artık dosyayı diğer ``PATH``
dizinlerinde bulamazlarsa ``EACCES`` errno değeri ile başarısız olurlar.
 
execlp Örneği
-------------
 
Aşağıda ``execlp`` fonksiyonuna bir örnek verilmiştir. Örnekte ``execlp`` fonksiyonu şöyle çağrılmıştır:
 
.. code-block:: c
 
    if ((pid = fork()) == -1)
        exit_sys("fork");
 
    if (pid == 0 && execlp("ls", "ls", "-l", (char *)0) == -1)
        exit_sys("execv");
 
Burada ``ls`` programı ``PATH`` çevre değişkeninde belirtilen ``/bin`` dizininde bulunacaktır.
 
.. code-block:: c
 
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>
 
    void exit_sys(const char *msg);
 
    int main(void)
    {
        pid_t pid;
 
        printf("sample running...\n");
 
        if ((pid = fork()) == -1)
            exit_sys("fork");
 
        if (pid == 0 && execlp("ls", "ls", "-l", (char *)0) == -1)
            exit_sys("execv");
 
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
 
execvp Örneği
-------------
 
Aşağıda ``execvp`` kullanımına örnek verilmiştir. Örnekte ``execvp`` fonksiyonu şöyle kullanılmıştır:
 
.. code-block:: c
 
    if ((pid = fork()) == -1)
        exit_sys("fork");
 
    if (pid == 0 && execvp(argv[1], &argv[1]) == -1)
        exit_sys("execvp");
 
Burada ``argv[1]`` ile girilen dosya isminde hiç ``/`` karakteri yoksa dosya ``PATH`` çevre değişkeni ile
belirtilen dizinlerde aranacaktır.
 
.. code-block:: c
 
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>
 
    void exit_sys(const char *msg);
 
    int main(int argc, char *argv[])
    {
        pid_t pid;
 
        if (argc == 1) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }
 
        printf("sample running...\n");
 
        if ((pid = fork()) == -1)
            exit_sys("fork");
 
        if (pid == 0 && execvp(argv[1], &argv[1]) == -1)
            exit_sys("execvp");
 
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
 
Kabuğun "./" Kullanımı ve PATH Güvenliği
========================================
 
Şimdi kabuk üzerinden programları neden ``./sample`` biçiminde başına ``./`` getirerek çalıştırdığımız
artık anlaşılabilir. Kabuk programları önce ``fork`` yapıp alt proseste exec fonksiyonlarının p'li
versiyonlarıyla programları çalıştırmaktadır. Dolayısıyla biz programı ``sample`` biçiminde çalıştırmak
istediğimizde bu p'li versiyonlar bu programı ``PATH`` çevre değişkeninin belirttiği dizinlerde
bulamayacaktır. Ancak biz programı ``./sample`` biçiminde çalıştırmak istediğimizde bu fonksiyonlar
artık ``PATH`` çevre değişkenine bakmayacak, bulunulan dizindeki ``sample`` programını çalıştıracaktır.
 
Peki kabuk programları neden exec fonksiyonlarının p'li versiyonlarını kullanmaktadır? Bunun birinci
sebebi kolaylık sağlamak içindir. Örneğin ``ls`` komutunu biz ``/bin/ls`` biçiminde kullanmak istemeyiz.
Bunun ikinci nedeni güvenliktir. Eskiden durum böyle değilken programın çalışma dizinine gerçek
komutlarla aynı isimli komutlar yerleştirerek hileli işlemler yapmaya yeltenenler olmuştur. İşte bu
nedenle ``PATH`` dizinlerinin içerisinde prosesin çalışma dizini yerleştirilmemektedir. Eğer durum böyle
olmasaydı bazen hatalı yazılmış komutlarla istenmeden başka programlar da çalıştırılabilirdi. Örneğin
dizinimizde ``co`` isminde bir program olsun; biz ``cp`` yerine yanlışlıkla ``co`` yazarsak bu programı
istemeden de çalıştırabiliriz.
 
myshell Programına fork/exec Ekleme
===================================
 
Şimdi de daha önce yapmış olduğumuz ``myshell`` kabuk programına fork/exec işlemini ekleyelim. Programın
bu versiyonu önce *içsel (internal)* komutlara bakacak, eğer içsel komutlarda verilen komutu bulmazsa
onu fork/exec ile program dosyası gibi çalıştıracaktır. Aslında ``bash`` gibi kabuk programları da böyle
yapmaktadır.
 
Biz ``myshell`` programımızda komut satırından aldığımız yazıyı parse edip parametrelerini zaten
``g_params`` isimli bir gösterici dizisinde saklamıştık. Örneğimizde eğer komut içsel komut listesinde
bulunamadıysa aşağıdaki gibi fork/exec uygulanmıştır:
 
.. code-block:: c
 
    if (g_cmds[i].name == NULL) {
            pid_t pid;
 
            if ((pid = fork()) == -1)
                exit_sys("fork");
            if (pid == 0 && execvp(g_params[0], &g_params[0]) == -1) {
                fprintf(stderr, "%s: %s\n", g_params[0], strerror(errno));
                continue;
            }
            if (waitpid(pid, NULL, 0) == -1)
                exit_sys("waitpid");
        }
 
.. code-block:: c
 
    /* myshell.c */
 
    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <errno.h>
    #include <unistd.h>
    #include <sys/wait.h>
 
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
            if (g_cmds[i].name == NULL) {
                pid_t pid;
 
                if ((pid = fork()) == -1)
                    exit_sys("fork");
                if (pid == 0 && execvp(g_params[0], &g_params[0]) == -1) {
                    fprintf(stderr, "%s: %s\n", g_params[0], strerror(errno));
                    continue;
                }
                if (waitpid(pid, NULL, 0) == -1)
                    exit_sys("waitpid");
            }
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
 
exec Hatalarına İlişkin errno Değerleri
=======================================
 
exec fonksiyonlarının başarısızlığının nedeni olabilecek çeşitli ``errno`` değerleri vardır. Bunların en
önemlilerinden birkaçı şunlardır:
 
- ``ENOENT`` ("No such file or directory"): Dosya bulunamamıştır.
- ``EACCES`` ("Permission denied"): Dosya bulunmuştur ancak proses dosyaya ``x`` hakkına sahip değildir.
- ``ENOEXEC`` ("Exec format error"): Dosya bulunmuştur. Prosesin dosyaya ``x`` hakkı da vardır. Ancak
  dosyanın formatı çalıştırmaya uygun değildir. Yani dosya çalıştırılabilir bir dosya değildir ya da
  dosyanın başında *shebang* yoktur.
- ``EINVAL`` ("Invalid argument"): Dosya bulunmuştur, proses dosyaya ``x`` hakkına sahiptir. Ancak dosya
  bu sistem tarafından desteklenen *çalıştırılabilir (executable)* bir formata sahip değildir.
 
Yukarıda da belirttiğimiz gibi exec fonksiyonlarının p'li versiyonları (``execlp`` ve ``execvp``)
``PATH`` dizinlerinde tek tek dosyayı aramaktadır. Ancak bu fonksiyonlar dosyayı bir dizinde bulduğunda
ve onu sistem fonksiyonuyla (Linux'ta ``sys_execve``) çalıştırmaya çalıştığında ``EINVAL`` ve ``ENOEXEC``
errno değerleri oluşursa dosyanın bir kabuk betiği (shell script) olduğundan çalıştırılamadığı sonucunu
çıkartmaktadır ve bu durumda dosyayı *"/bin/sh (default shell)"* programı ile çalıştırmaktadır. Ancak
exec fonksiyonlarının diğer versiyonları ``EINVAL`` ve ``ENOEXEC`` hatalarında bunu yapmamaktadır. Bunu
yalnızca exec fonksiyonlarının p'li versiyonları yapmaktadır. exec fonksiyonlarının p'li versiyonları
``PATH`` dizinlerinin birinde dosyayı sistem fonksiyonuyla (``sys_execve``) çalıştırmaya çalıştığında
``EACCES`` errno değeri ile başarısız olursa dosyayı sonraki ``PATH`` dizinlerinde aramaya devam
ederler. Ancak bu arama sırasında bu fonksiyonlar artık dosyayı diğer ``PATH`` dizinlerinde de
bulamazlarsa ``EACCES`` errno değeri ile başarısız olmaktadır.
 
Bu davranışın anlamı izleyen bölümlerde başka paragraflarda daha iyi anlaşılacaktır.
 
execle ve execve Fonksiyonları (e'li Versiyonlar)
=================================================
 
exec fonksiyonlarının iki tane e'li biçimleri vardır: ``execle`` ve ``execve``. Buradaki *e* harfi
*environment* yani *çevre değişkenleri* anlamında isme eklenmiştir.
 
Anımsanacağı gibi çevre değişkenleri tipik olarak prosesin bellek alanında bulunduruluyordu ve ``fork``
işlemi sırasında üst prosesin bellek alanının alt prosese kopyalanmasıyla alt prosese geçiriliyordu.
Ancak exec işlemleri prosesin bellek alanını ortadan kaldırıp yeni bir program kodunu yüklediğine göre
prosesin çevre değişkenleri ne olacaktır? İşte exec işlemi sırasında prosesin bellek alanı boşaltılıp
yeni program için prosesin bellek alanı yeniden oluşturulurken çevre değişkenleri de sıfırdan
oluşturulabilmektedir. Bunu exec fonksiyonlarının e'li versiyonları yapmaktadır. exec fonksiyonlarının
e'siz versiyonları o andaki prosesin çevre değişkenlerinin aynısını exec yapılan programın bellek
alanına taşımaktadır. Yani biz exec fonksiyonlarının e'siz versiyonlarını kullandığımızda exec yapmadan
önceki çevre değişkenleriyle exec yapıldıktan sonraki programın çevre değişkenleri aynı olacaktır.
 
``execle`` ve ``execve`` fonksiyonlarının prototipleri şöyledir:
 
.. code-block:: c
 
    #include <unistd.h>
 
    int execle(const char *path, const char *arg0, ... /*, (char *)0, char *const envp[]*/);
    int execve(const char *path, char *const argv[], char *const envp[]);
 
``execle`` fonksiyonunun birinci parametresi yine çalıştırılacak dosyanın yol ifadesini almaktadır.
Diğer parametreler programa geçirilecek komut satırı argümanlarını belirtir. Bu argüman listesinin sonu
yine ``NULL`` adresle bitirilmelidir. Bu ``NULL`` adresten sonra son parametre ``char`` türden bir
gösterici dizisi olmalıdır. Bu gösterici dizisi çevre değişkenlerini ``anahtar=değer`` biçiminde tutan
yazıların başlangıç adreslerinden oluşmalıdır (yani ``environ`` global değişkeninde olduğu gibi). Bu
fonksiyonlardaki çevre değişkenleri için oluşturulan gösterici dizilerinin sonunda ``NULL`` adres
bulunmalıdır.
 
``execve`` fonksiyonu da benzerdir. Bu fonksiyon da önce çalıştırılacak programın yol ifadesini alır.
Sonra komut satırı argümanlarını bir gösterici dizisi olarak, sonra da çevre değişkenlerini bir gösterici
dizisi olarak almaktadır.
 
execle Örneği
-------------
 
Aşağıdaki örnekte ``execve`` fonksiyonunun kullanımına bir örnek verilmiştir. Örnekte ``sample``
programı aynı dizindeki ``other`` programını çalıştırmaktadır. ``execle`` işlemi şöyle yapılmıştır:
 
.. code-block:: c
 
    pid_t pid;
    char *env[] = {"city=ankara", "furit=banana", "color=red", NULL};
 
    if ((pid = fork()) == -1)
        exit_sys("fork");
 
    if (pid == 0) {
        execle("other", "other", "ali", "veli", "selami", (char *)0, env);
        exit_sys("execle");
    }
 
    printf("parent continues...\n");
 
Burada ``execle`` fonksiyonunun argümanlarına dikkat ediniz. Artık ``other`` programı çalıştırıldığında
alt prosesin çevre değişken listesi ``env`` gösterici dizisindeki gibi olacaktır. ``sample`` programını
çalıştırmadan önce ``other`` programını da derlemelisiniz. ``sample`` programı çalıştırıldığında ekrana
şunlar basılacaktır:
 
.. code-block:: console
 
    $ ./sample
    parent continues...
    other command line arguments:
    other
    ali
    veli
    selami
    other environment variables:
    city=ankara
    furit=banana
    color=red
 
.. code-block:: c
 
    /* sample.c */
 
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>
 
    void exit_sys(const char *msg);
 
    int main(void)
    {
        pid_t pid;
        char *env[] = {"city=ankara", "furit=banana", "color=red", NULL};
 
        if ((pid = fork()) == -1)
            exit_sys("fork");
 
        if (pid == 0) {
            execle("other", "other", "ali", "veli", "selami", (char *)0, env);
            exit_sys("execle");
        }
 
        printf("parent continues...\n");
 
        if (waitpid(pid, NULL, 0) == -1)
            exit_sys("waitpid");
 
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
 
    extern char **environ;
 
    int main(int argc, char *argv[])
    {
        printf("other command line arguments:\n");
 
        for (int i = 0; i < argc; ++i)
            puts(argv[i]);
 
        printf("other environment variables:\n");
 
        for (int i = 0; environ[i] != NULL; ++i)
            puts(environ[i]);
 
        return 0;
    }
 
execve Örneği (execle'nin execve ile Yazımı)
--------------------------------------------
 
Daha önceden de belirtildiği gibi UNIX türevi sistemlerde yalnızca ``execve`` fonksiyonu sistem
fonksiyonu olarak işletim sistemi içerisinde bulunmaktadır. Aslında ``execl``, ``execlp``, ``execv``,
``execvp``, ``execle`` fonksiyonları, ``execve`` fonksiyonunu çağıran birer kütüphane fonksiyonu
biçiminde bulundurulmaktadır. Yani burada *taban (base)* fonksiyon ``execve`` fonksiyonudur.
 
Aşağıda ``execv`` fonksiyonunun ``execve`` kullanılarak basit biçimde yazımına örnek verilmiştir. Bu
örnek yukarıdaki örneğin aynısıdır. Yalnızca ``execle`` yerine ``execve`` fonksiyonu kullanılmıştır.
 
.. code-block:: c
 
    /* sample.c */
 
    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>
 
    void exit_sys(const char *msg);
 
    int main(void)
    {
        pid_t pid;
        char *args[] = {"ali", "veli", "selami", NULL};
        char *env[] = {"city=ankara", "furit=banana", "color=red", NULL};
 
        if ((pid = fork()) == -1)
            exit_sys("fork");
 
        if (pid == 0) {
            execve("other", args, env);
            exit_sys("execve");
        }
 
        printf("parent continues...\n");
 
        if (waitpid(pid, NULL, 0) == -1)
            exit_sys("waitpid");
 
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
 
    extern char **environ;
 
    int main(int argc, char *argv[])
    {
        printf("other command line arguments:\n");
 
        for (int i = 0; i < argc; ++i)
            puts(argv[i]);
 
        printf("other environment variables:\n");
 
        for (int i = 0; environ[i] != NULL; ++i)
            puts(environ[i]);
 
        return 0;
    }


execve Kullanarak Değişken Sayıda Argüman Alan execl Gerçekleştirimi
====================================================================

Aşağıdaki örnekte de ``execl`` fonksiyonunun ``execve`` kullanılarak nasıl yazıldığı hakkında bir fikir
verilmiştir. Burada komut satırı argümanlarının sayısı ``MAX_ARG`` ile sınırlandırılmıştır.

Değişken sayıda argüman alan fonksiyonların yazımını inceleyiniz.

.. code-block:: c

    /* execl.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>
    #include <stdarg.h>

    #define MAX_ARG        4096

    void exit_sys(const char *msg);

    extern char **environ;

    int myexecl(const char *path, const char *arg0, ...)
    {
        va_list vl;
        char *args[MAX_ARG + 1];
        char *arg;
        int i;

        va_start(vl, arg0);

        args[0] = (char *)arg0;
        for (i = 1; (arg = va_arg(vl, char *)) != NULL && i < MAX_ARG; ++i)
            args[i] = arg;
        args[i] = NULL;

        va_end(vl);

        return execve(path, args, environ);
    }

    int main(void)
    {
        pid_t pid;

        printf("execl running...\n");

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid == 0 && myexecl("/bin/ls", "/bin/ls", "-l", "-i", (char *)0) == -1)
            exit_sys("myexecl");

        printf("ok, parent continues...\n");

        if (waitpid(pid, NULL, 0) == -1)
            exit_sys("waitpid");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

fexecve Fonksiyonu
==================

``fexecve`` isimli POSIX fonksiyonu ``execve`` fonksiyonu gibidir. Ancak bunun tek farkı yol ifadesi
yerine dosya betimleyicisini alarak çalışmasıdır. Yani biz çalıştırmak istediğimiz program dosyasını
zaten ``open`` fonksiyonu ile açmışsak bu durumda doğrudan ``fexecve`` fonksiyonunu kullanabiliriz.
Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int fexecve(int fd, char *const argv[], char *const envp[]);

Fonksiyonun birinci parametresi çalıştırılacak dosyanın dosya betimleyicisini belirtmektedir. Diğer
parametreler ``execve`` fonksiyonu ile tamamen aynıdır. Bu fonksiyonun birinci parametresinde belirtilen
betimleyiciye ilişkin dosya hangi modda açılmış olmalıdır? POSIX standartlarında dosyanın ``O_EXEC``
bayrağı ile ya da ``O_RDONLY`` bayrağı ile açılması gerektiği belirtilmiştir. ``O_EXEC`` bayrağında zaten
açış sırasında dosyanın ``x`` hakkına sahip olup olmadığına bakılmaktadır. ``O_RDONLY`` bayrağında açış
sırasında ``x`` hakkına bakılmaz, ancak ``fexecve`` çağrısı sırasında prosesin dosyaya ``x`` hakkına
sahip olup olmadığı kontrol edilmektedir. Linux çekirdeği ``O_EXEC`` bayrağını desteklemediği için Linux
sistemlerinde dosya ``O_RDONLY`` bayrağı ile ya da ``O_PATH`` bayrağı ile açılmalıdır. Anımsanacağı gibi
``O_PATH`` bayrağı da POSIX tarafından desteklenmemektedir.

Aşağıda ``fexecve`` fonksiyonunun kullanımına bir örnek verilmiştir. Burada dosya alt proseste açılmıştır:

.. code-block:: c

    if (pid == 0) {
        if ((fd = open(argv[1], O_RDONLY)) == -1)
            exit_sys("open");
        if (fexecve(fd, &argv[1], environ) == -1)
            exit_sys("fexecve");
        /* unreachable code */
    }
    /* ... */

Açış işleminin Linux'ta ``O_RDONLY`` bayrağı ile yapıldığına dikkat ediniz. (Linux ``O_EXEC`` bayrağını
desteklememektedir.) Örneğimizde ``fexecve`` fonksiyonuna üst prosesin çevre değişken listesi
geçirilmiştir.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <unistd.h>
    #include <sys/wait.h>

    void exit_sys(const char *msg);

    extern char **environ;

    int main(int argc, char *argv[])
    {
        pid_t pid;
        int fd;

        if (argc == 1) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }

        printf("sample running...\n");

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid == 0) {
            if ((fd = open(argv[1], O_RDONLY)) == -1)
                exit_sys("open");
            if (fexecve(fd, &argv[1], environ) == -1)
                exit_sys("fexecve");
            /* unreachable code */
        }

        printf("ok, parent continues...\n");

        if (waitpid(pid, NULL, 0) == -1)
            exit_sys("waitpid");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

exec ve Açık Dosyalar: close-on-exec Bayrağı
============================================

exec işlemi yapıldığında o ana kadar açık olan dosyaların akıbeti ne olacaktır? Anımsanacağı gibi açık
dosyaların dosya nesnelerinin adresleri *dosya betimleyici tablosu* denilen bir tabloda tutuluyordu. exec
işlemi sırasında prosesin betimleyici tablosu korunmaktadır. Bu durumda örneğin bir program 100 tane
dosya açıp sonra exec işlemi uygulasa yeni çalıştırılacak program bu 100 dosyanın farkında olmayacaktır.
Ancak dosya betimleyici tablosunda bu 100 betimleyici çoğu kez gereksiz bir biçimde (seyrek olarak
böylesi bir durum kasten istenebilir) bulunmaya devam edecektir. İşte UNIX/Linux sistemlerinde her açık
dosya için *close-on-exec* isminde bir bayrak da tutulmaktadır. Eğer bu bayrak *set* edilmişse bu durumda
exec işlemi sırasında bu dosya işletim sistemi tarafından otomatik olarak kapatılır. Eğer bu bayrak
*reset* durumdaysa bu durumda exec işlemi sırasında dosya kapatılmaz, exec yapılan program kodu dosyanın
betimleyicisini bilirse onu kullanmaya devam edebilir. Bu bayrak default olarak *reset* durumdadır. Yani
exec sonrasında önceki programın açmış olduğu dosyalar açık kalmaya devam etmektedir.

O_CLOEXEC ile Açılışta Bayrağı Ayarlama
---------------------------------------

İşte ``open`` fonksiyonuyla dosya açılırken açış modunda ``O_CLOEXEC`` bayrağı belirtilirse bu bayrak set
edilmiş olur. Böylece exec işlemi sırasında dosya otomatik biçimde kapatılır. Örneğin:

.. code-block:: c

    fd = open("test.txt", O_RDONLY|O_CLOEXEC);

fcntl ile close-on-exec Bayrağını Sonradan Değiştirme
-----------------------------------------------------

Programcı isterse herhangi bir zaman ``fcntl`` fonksiyonu ile de bu bayrağı set ya da reset edebilir. Biz
bu ``fcntl`` fonksiyonunu henüz görmedik. Ancak bu bayrağın set edilmesi işlemi şöyle yapılabilmektedir:

.. code-block:: c

    if (fcntl(fd, F_SETFD, fcntl(fd, F_GETFD)|FD_CLOEXEC) == -1)
        exit_sys("fcntl");

Benzer biçimde bu bayrak şöyle de reset edilebilir:

.. code-block:: c

    if (fcntl(fd, F_SETFD, fcntl(fd, F_GETFD) & ~FD_CLOEXEC) == -1)
        exit_sys("fcntl");

Close-on-exec bayrağı dosya nesnesinin içerisinde tutulmamaktadır. Çünkü aynı dosya nesnesini gösteren
farklı betimleyiciler olabilir. Bu betimleyicilerden birinin close-on-exec bayrağı set edilmişken
diğerinin set edilmemiş olabilir. Yani close-on-exec bayrağı dosya nesnesinin içerisinde değil, proses
kontrol bloğu içerisinde başka bir yerdedir.

close-on-exec Örneği (sample.c / other.c)
-----------------------------------------

Aşağıdaki örnekte ``sample`` programı ``execl`` ile ``other`` programını çalıştırmıştır. Ancak ``other``
programı ``sample`` programının açmış olduğu dosyanın betimleyici numarasını bilmediği için ``sample``
programı komut satırı argümanıyla bu bilgiyi ``other`` programına iletmiştir.

Aşağıdaki programı daha sonra dosyanın close-on-exec bayrağını set ederek yeniden deneyiniz:

.. code-block:: c

    if ((fd = open("sample.c", O_RDONLY|O_CLOEXEC)) == -1)
        exit_sys("open");

Tabii aynı işlem şöyle de yapılabilirdi:

.. code-block:: c

    if ((fd = open("sample.c", O_RDONLY)) == -1)
        exit_sys("open");

    if (fcntl(fd, F_SETFD, fcntl(fd, F_GETFD)|FD_CLOEXEC) == -1)
        exit_sys("fcntl");

Bu durumda alt proseste dosya betimleyicisi kapalı olduğu için ``read`` fonksiyonu -1 ile geri dönecek ve
``errno`` değişkeni *EBADF ("Bad file descriptor")* ile set edilecektir.

close-on-exec bayrağı bazı işlemler sırasında işletim sistemi tarafından set ya da reset edilebilmektedir.
Örneğin ``dup`` ve ``dup2`` fonksiyonları ile dosya betimleyicisinin kopyası çıkartılırken her zaman yeni
betimleyicinin close-on-exec bayrağı reset durumda olur.

O_CLOFORK Bayrağı (POSIX 2024)
------------------------------

Anımsanacağı gibi POSIX'e 2024 versiyonu ile ``fork`` yaparken de dosyanın otomatik kapatılmasını
sağlayan ``O_CLOFORK`` bayrağı eklenmiştir. Ancak Linux'un bu bayrağı desteklemediğini belirtmiştik.
Linux'un bu bayrağı desteklememesinin nedeni exec işlemi olmadan tek başına ``fork`` işleminin artık pek
kullanılmaması ve bu desteğin mevcut çekirdek tasarımına bir yük getirmesidir.

.. code-block:: c

    /* sample.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <unistd.h>
    #include <sys/wait.h>

    void exit_sys(const char *msg);

    int main(int argc, char *argv[])
    {
        pid_t pid;
        int fd;
        char fd_str[64];

        if ((fd = open("test.txt", O_RDONLY)) == -1)
            exit_sys("open");

        if ((pid = fork()) == -1)
            exit_sys("fork");

        sprintf(fd_str, "%d", fd);
        if (pid == 0 && execl("other", "other", fd_str, (char *)0) == -1)
            exit_sys("execve");

        printf("parent process continues...\n");

        if (waitpid(pid, NULL, 0) == -1)
            exit_sys("waitpid");

        close(fd);

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
    #include <unistd.h>

    #define BUFFER_SIZE        4096

    void exit_sys(const char *msg);

    int main(int argc, char *argv[])
    {
        int fd;
        char buf[BUFFER_SIZE + 1];
        ssize_t result;

        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!...\n");
            exit(EXIT_FAILURE);
        }
        fd = atoi(argv[1]);
        lseek(fd, 0, SEEK_SET);

        while ((result = read(fd, buf, BUFFER_SIZE)) > 0) {
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

Kabukta IO Yönlendirmesini Taklit Etme (redirect_stdout Örneği)
===============================================================

Şimdi de kabuk programlarının IO yönlendirmesini nasıl yaptığına ilişkin küçük bir uygulama üzerinde
duralım. Anımsanacağı gibi kabuk üzerinde ``>`` operatörü çalıştırılan programın 1 numaralı
betimleyicisini (``STDOUT_FILENO``) ``>`` operatörünün sağındaki dosyaya yönlendirmektedir. Örneğin:

.. code-block:: console

    # ./sample > test.txt

Burada ``sample`` programının ``stdout`` dosyasına yazdıkları ekrana yazılmayacak, ``test.txt``
dosyasına yazılacaktır. Peki kabuk bunu nasıl yapmaktadır? İşlemin şu biçimde olduğunu varsayalım:

.. code-block:: console

    $ a > b

İşte tipik olarak kabuk önce *a* programı için ``fork`` yapar. Ancak henüz exec yapmadan alt prosesin 1
numaralı betimleyicisini *b* dosyasını açarak ona yönlendirir. Sonra da exec uygular.

Biz bu işlemi yapan aşağıdaki gibi bir fonksiyon yazmak isteyelim:

.. code-block:: c

    int redirect_stdout(const char *cmd);

Fonksiyon bizden tıpkı kabukta olduğu gibi ``>`` ile yapılan yönlendirme yazısını alacak olsun. Örneğin:

.. code-block:: c

    result = redirect_stdout("ls -l > test.txt");

Bizim bu fonksiyon içerisinde önce ``>`` karakterini bulup onun solunu ve sağını ayrıştırmamız gerekir.
Sonra ``fork`` uygulayıp alt proseste yönlendirmeyi yapıp exec uygulamamız gerekir. Bu işlemi aşağıdaki
örnekte şöyle yaptık:

.. code-block:: c

    if ((pid = fork()) == -1)
        return -1;

    if (pid == 0) {
        int fd;

        if ((fd = open(pi.path, O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH)) == -1)
            _exit(1);
        if (dup2(fd, 1) == -1)
            _exit(2);
        execvp(pi.args[0], &pi.args[0]);
        _exit(3);
    }

Burada ``pi``, ``>`` karakteri ile ayrıştırma sonucunda ayrıştırılmış bilgilerin yerleştirildiği
``parse_info`` isimli bir yapı nesnesidir. ``parse_info`` yapısı şöyle tanımlanmıştır:

.. code-block:: c

    struct parse_info {
        char *args[MAX_PARAM + 1];
        char *path;
    };

Aşağıda örneği bütünsel olarak veriyoruz. Programı şöyle test edebilirsiniz:

.. code-block:: console

    $ ./redirect "ls -l  > test.txt"

.. code-block:: c

    /* redirect.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>
    #include <ctype.h>
    #include <fcntl.h>
    #include <sys/stat.h>
    #include <unistd.h>
    #include <errno.h>
    #include <sys/wait.h>

    #define MAX_CMD_PARSE       4096
    #define MAX_PARAM           128

    struct parse_info {
        char *args[MAX_PARAM + 1];
        char *path;
    };

    int redirect_stdout(char *cmd);
    void exit_sys(const char *msg);

    int main(int argc, char *argv[])
    {
        if (argc != 2) {
            fprintf(stderr, "wrong number of arguments!..\n");
            exit(EXIT_FAILURE);
        }
        if (redirect_stdout(argv[1]) == -1)
            exit_sys("Error");

        return 0;
    }

    int parse_redirect(char *cmd, struct parse_info *pi)
    {
        char *str;
        char *tok;
        size_t i;

        if ((str = strchr(cmd, '>')) == NULL || strchr(str + 1, '>'))
            return -1;
        *str++ = '\0';

        for (i = 0, tok = strtok(cmd, " \t"); tok != NULL; tok = strtok(NULL, " \t"), ++i)
            pi->args[i] = tok;

        if (i == 0)
            return -1;

        pi->args[i] = NULL;

        for (i = 0; isspace(str[i]); ++i)
            ;
        if (str[i] == '\0')
            return -1;
        pi->path = str + i;
        for (; !isspace(str[i]) && str[i] != '\0'; ++i)
            ;
        str[i] = '\0';

        return 0;
    }

    int redirect_stdout(char *cmd)
    {
        pid_t pid;
        char cmd_parse[MAX_CMD_PARSE + 1];
        struct parse_info pi;

        if (strlen(cmd) > MAX_CMD_PARSE) {
            errno = EINVAL;
            return -1;
        }
        strcpy(cmd_parse, cmd);

        if (parse_redirect(cmd_parse, &pi) == -1) {
            errno = EINVAL;
            return -1;
        }

        if ((pid = fork()) == -1)
            return -1;

        if (pid == 0) {
            int fd;

            if ((fd = open(pi.path, O_WRONLY|O_CREAT|O_TRUNC, S_IRUSR|S_IWUSR|S_IRGRP|S_IROTH)) == -1)
                _exit(1);
            if (dup2(fd, 1) == -1)
                _exit(2);
            execvp(pi.args[0], &pi.args[0]);
            _exit(3);
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

exec ile Betik (Script) Dosyalarının Çalıştırılması ve shebang
==============================================================

exec fonksiyonları ile betik (script) dosyaları da (yani text dosyalar da) çalıştırılabilmektedir. Bu
özellik tamamen çekirdekte bulunan sistem fonksiyonları (Linux'ta ``execve``) tarafından sağlanmaktadır.
exec fonksiyonları (aslında Linux'ta ``execve`` sistem fonksiyonu) eğer çalıştırılmak istenen dosya
*çalıştırılabilir bir dosya değilse (örneğin Linux'ta ELF formatı ya da a.out formatı değilse)* bu
dosyanın birinci satırını okuyarak onunla özel bir işlem yapmaktadır. Çalıştırılabilir formata sahip
olmayan bir dosyanın (tipik olarak bir text dosya) birinci satırı aşağıdaki gibi ise exec fonksiyonları
burada özel bir işlem uygulamaktadır:

.. code-block:: text

    #! [optional SPACE'ler] <executable file mutlak yol ifadesi> [isteğe bağlı argüman(lar)]

shebang Satırının Biçimi
------------------------

Burada ``#!`` karakterlerine genellikle *shebang* denilmektedir. Bu karakterler hemen dosyanın başında
bulunmak zorundadır. Shebang karakterlerinden sonra isteğe bağlı bir ya da birden fazla SPACE karakteri
bulundurulabilmektedir. Bundan sonra gerçekten çalıştırılacak olan *çalıştırılabilir bir dosyanın* mutlak
yol ifadesi olmalıdır. Bunu isteğe bağlı argümanlar izleyebilir. Örneğin aşağıdaki satırlar geçerlidir:

.. code-block:: text

    #! /bin/bash
    #!/bin/bash
    #!/usr/bin/python
    #!/usr/bin/make -f

exec işlemini yapan sistem fonksiyonları, eğer exec yapılmak istenen dosya çalıştırılabilir bir dosya
değilse (burada ``x`` hakkını kastetmiyoruz, dosyanın ELF gibi bir formata sahip olmadığını
kastediyoruz), onun birinci satırını okuyarak orada belirtilen çalıştırılabilir dosyayı çalıştırmaktadır.
Ancak exec fonksiyonlarının bu işlemi yapabilmesi için exec yapılan dosyanın yine de (text dosyası
olmasına karşın) ``x`` hakkına sahip olması gerekmektedir. Aksi takdirde exec fonksiyonları başarısız
olur ve yine ``errno`` değeri ``EACCES`` biçiminde set edilir.

Yukarıdaki gibi shebang satırı içeren bir betik dosyası exec fonksiyonlarıyla çalıştırılmak istendiğinde
exec fonksiyonları betik dosyasını değil *shebang* satırında belirtilen çalıştırılacak dosyayı
çalıştırmaktadır. Ancak o dosyayı çalıştırırken betik dosyasının yol ifadesini de o programa komut
satırı argümanı olarak geçirmektedir. Örneğin ``myscript`` ismindeki aşağıdaki dosyayı exec
fonksiyonlarıyla çalıştırmak isteyelim:

.. code-block:: bash

    #!/bin/bash

    for i in {1..10}; do
        echo "$i"
    done

Burada dosyanın shebang satırında ``/bin/bash`` dosyası belirtilmektedir. İşte exec fonksiyonları aslında
bu ``/bin/bash`` dosyasını çalıştırıp ``myscript`` dosyasını da bu programa komut satırı argümanı olarak
geçirmektedir. Yani aslında aşağıdaki çalıştırmayla eşdeğer bir durum ortaya çıkmaktadır:

.. code-block:: console

    $ /bin/bash myscript

Görüldüğü gibi bu örnekte aslında betik dosyasını exec fonksiyonları değil ``/bin/bash`` programı
çalıştırmaktadır. exec fonksiyonları bu sürece yalnızca aracılık etmektedir.

shebang'te Belirtilen Programa Aktarılan Komut Satırı Argümanları
-----------------------------------------------------------------

Shebang satırında belirtilen programın çalıştırılması sırasında bu programa geçirilen komut satırı
argümanları şöyledir:

.. code-block:: text

    argv[0] ---> shebang'te belirtilen program dosyasına ilişkin yol ifadesi
    argv[1] ---> Eğer shebang'te çalıştırılabilen programın yanında isteğe bağlı argüman varsa o argüman
    argv[2] ---> exec fonksiyonunda belirtilen çalıştırılabilir olmayan dosyanın (yani betik dosyasının)
                 yol ifadesi
    argv[3] ve sonrası ---> exec fonksiyonunda belirtilen komut satırı argümanları, ancak ilk argüman
                 dahil değil

Eğer shebang'in yanındaki programın yol ifadesinin yanında isteğe bağlı argüman verilmemişse bu durumda
shebang'te belirtilen programın komut satırı argümanları şöyle olacaktır:

.. code-block:: text

    argv[0] ---> shebang'te belirtilen program dosyasına ilişkin yol ifadesi
    argv[1] ---> exec fonksiyonunda belirtilen çalıştırılabilir olmayan dosyanın (yani betik dosyasının)
                 yol ifadesi
    argv[2] ve sonrası ---> exec fonksiyonunda belirtilen komut satırı argümanları, ancak ilk argüman
                 dahil değil

Burada dikkat edilmesi gereken bir nokta şudur: exec fonksiyonunda belirtilen ``argv[0]`` için girilen
argüman shebang satırında belirtilen programa aktarılmamaktadır.

Argüman Aktarımı Denemeleri
---------------------------

Şimdi çeşitli denemelerle argüman aktarımını anlamaya çalışalım.

``sample.py`` isimli Python programını biz normalde şöyle çalıştırırız.

.. code-block:: console

    $ python3 sample.py

``python3`` isimli yorumlayıcı (ismi ``python`` da olabilir) çalıştıracağı programı komut satırı
argümanı olarak almaktadır. Şimdi biz çalıştırma işlemini kolaylaştırmak isteyelim. Bunun için
``sample.py`` dosyasının başına shebang satırı yerleştirmeliyiz:

.. code-block:: python

    #!/usr/bin/python3

    for i in range(10):
        print(i)

``sample.py`` dosyasına ``chmod`` komutu ile ``x`` hakkı verelim:

.. code-block:: console

    $ chmod +x sample.py

Artık Python programını sanki bir C programıymış gibi çalıştırabiliriz:

.. code-block:: console

    $ ./sample.py
    0
    1
    2
    3
    4
    5
    6
    7
    8
    9

Burada exec fonksiyonları aslında aşağıdaki gibi bir çalıştırma yapılmış gibi işlem exec uygulayacaktır:

.. code-block:: console

    $ python3 sample.py

``test.txt`` dosyasının shebang satırı şöyle olsun:

.. code-block:: text

    #!/home/kaan/Study/UnixLinux-SysProg/sample ankara

Burada biz denememizin ``/home/kaan/Study/UnixLinux-SysProg`` dizininde yapıldığını varsayıyoruz. Siz bu
denemeyi yaparken shebang satırındaki dizini kendi çalıştığınız dizinle değiştirmelisiniz.

Burada görüldüğü gibi shebang'te belirtilen programın yanında isteğe bağlı bir argüman (*ankara*
argümanı) bulunmaktadır. Şimdi ``sample`` programının da C'de şöyle yazıldığını varsayalım:

.. code-block:: c

    /* sample.c */

    int main(int argc, char *argv[])
    {
        printf("sample running...\n");

        for (int i = 0; i < argc; ++i)
            printf("argv[%d]: %s\n", i, argv[i]);

        return 0;
    }

Şimdi aşağıdaki gibi exec yapmış olalım:

.. code-block:: c

    execl("test.txt", "test.txt", "ali", "veli", "selami", (char *)0);

Ekranda şunları görmeliyiz:

.. code-block:: text

    sample running...
    argv[0]: /home/kaan/Study/UnixLinux-SysProg/10-Exec/sample
    argv[1]: ankara
    argv[2]: test.txt
    argv[3]: ali
    argv[4]: veli
    argv[5]: selami

exec işlemi şöyle yapılmış olsun:

.. code-block:: c

    execl("test.txt", "ali", "veli", "selami", (char *)0);

Ekrana şunlar çıkacaktır:

.. code-block:: text

    sample running...
    argv[0]: /home/kaan/Study/UnixLinux-SysProg/10-Exec/sample
    argv[1]: ankara
    argv[2]: test.txt
    argv[3]: veli
    argv[4]: selami

Şimdi de shebang satırı şöyle olsun:

.. code-block:: text

    #!/home/kaan/Study/UnixLinux-SysProg/sample

Görüldüğü gibi burada artık shebang'te belirtilen programın yanında isteğe bağlı argüman yoktur. Şimdi
exec işlemini şöyle yapmış olalım:

.. code-block:: c

    execl("test.txt", "test.txt", "ali", "veli", "selami", (char *)0);

.. code-block:: text

    sample running...
    argv[0]: /home/kaan/Study/UnixLinux-SysProg/10-Exec/sample
    argv[1]: test.txt
    argv[2]: ali
    argv[3]: veli
    argv[4]: selami

shebang Satırının Uzunluk Sınırı ve Göreli Yol Kullanımı
--------------------------------------------------------

Sistemlerde genellikle shebang satırları için maksimum bir uzunluk belirlenmiş olmaktadır. Örneğin eski
Linux sistemlerinde eğer shebang satırı uzunsa çekirdek bunun ilk 127 karakterini dikkate almaktadır.
Ancak Linux'ta 5.1 çekirdeği ile birlikte bu uzunluk 255'e yükseltilmiştir.

Shebang'te belirtilen çalıştırılabilir program genellikle *mutlak yol ifadesi* ile belirtilmektedir.
Ancak Linux'ta buradaki program *göreli yol ifadesi* ile de belirtilebilmektedir. Örneğin:

.. code-block:: text

    #!sample

Bu durumda burada belirtilen program exec işlemini yapan prosesin çalışma dizini temel alınarak
aranmaktadır.

shebang'e Birden Fazla Argüman Yazma
------------------------------------

Shebang'te belirtilen programın yanına birden fazla argüman yazabilir miyiz? Örneğin:

.. code-block:: text

    #!/home/kaan/Study/Unix-Linux-SysProg/sample ankara izmir istanbul

Maalesef bu durumda UNIX türevi sistemler arasında bazı farklılıklar söz konusu olmaktadır. Bu durum
POSIX standartlarında açık biçimde belirtilmemiş ve işletim sistemini yazanların isteğine bırakılmıştır.
Linux ve pek çok sistem bu durumda shebang'te belirtilen programın sağındaki tüm argümanları tek bir
argümanmış gibi aktarmaktadır. ``test.txt`` dosyasının başının yukarıdaki gibi olduğunu varsayalım. Bu
dosya aşağıdaki gibi exec yapılmış olsun:

.. code-block:: c

    execl("test.txt", "test.txt", "ali", "veli", "selami", (char *)0);

Linux sistemlerinde aşağıdaki gibi bir çıktı elde edilmiştir:

.. code-block:: text

    sample running...
    argv[0]: /home/kaan/Study/UnixLinux-SysProg/10-Exec/sample
    argv[1]: ankara izmir istanbul
    argv[2]: test.txt
    argv[3]: ali
    argv[4]: veli
    argv[5]: selami

Bazı UNIX türevi sistemler bu durumda yalnızca boşlukla ayrılmış ilk argümanı (örneğimizde *ankara*)
programa aktarıp diğerlerini ihmal edebilmektedir. Bu durumda programcının taşınabilirliği sağlamak için
shebang satırında tek bir argüman kullanması tavsiye edilmektedir.

Betik Dosyasının Doğrudan Kabuktan Çalıştırılması
-------------------------------------------------

Tabii biz bir script dosyasını doğrudan kabuk üzerinden de çalıştırabiliriz. Fark eden bir şey yoktur. Bu
durumda zaten exec işlemini kabuk uygulamaktadır. Örneğin ``test.txt`` dosyası şöyle olsun:

.. code-block:: text

    #!/home/kaan/Study/UnixLinux-SysProg/sample ankara

Şimdi bunu kabuk üzerinden çalıştıralım:

.. code-block:: console

    $ ./test.txt ali veli selami

    sample running...
    argv[0]: /home/kaan/Study/UnixLinux-SysProg/10-Exec/sample
    argv[1]: ankara
    argv[2]: ./test.txt
    argv[3]: ali
    argv[4]: veli
    argv[5]: selami

Görüldüğü gibi burada exec işlemini kabuk uygulamıştır. Kabuk exec uygularken dosya ismini yine exec'te
ilk komut satırı argümanı olarak kullanır. Ancak exec bunu shebang'te belirtilen programa
aktarmamaktadır.

Tam Örnek (exec-prog.c / sample.c / test.txt)
---------------------------------------------

Aşağıda shebang programına argüman aktarımının test edilmesi için bir örnek verilmiştir. Buradaki
``test.txt`` script programına ``chmod`` komutu ile ``x`` hakkı vermeyi unutmayınız. Burada biz denemeyi
kendi makinemizde ``/home/kaan/Study/UnixLinux-SysProg`` dizininde yaptık. Siz kendi dizininizde
yaparken shebang satırındaki dizini kendi çalıştığınız dizinle değiştirmelisiniz.

.. code-block:: c

    /* exec-prog.c */

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

        if (pid == 0) {
            execl("test.txt", "test.txt", "ali", "veli", "selami", (char *)0);
            exit_sys("execl");
        }

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

    /* sample.c */

    #include <stdio.h>
    #include <stdlib.h>

    int main(int argc, char *argv[])
    {
        printf("sample running...\n");

        for (int i = 0; i < argc; ++i)
            printf("argv[%d]: %s\n", i, argv[i]);

        return 0;
    }

.. code-block:: text

    /* test.txt */

    #!/home/kaan/Study/UnixLinux-SysProg/10-Exec/sample ankara



.. _shebang-mekanizmasi-ve-system-fonksiyonu:

========================================
shebang Mekanizması ve system Fonksiyonu
========================================

shebang Mekanizmasının Amacı ve Örnekler
========================================

Peki bütün bunların anlamı nedir? Yani shebang ile bir script dosyasının aslında başka bir programı
çalıştırmasının ne faydası olabilir? İşte bu mekanizma sayesinde yorumlayıcı yoluyla çalıştırılan
dosyaların doğrudan çalıştırılabilmesine olanak sağlanmaktadır.

Bash Betiği Örneği (sample.sh)
------------------------------

Örneğin aşağıdaki gibi ``sample.sh`` isimli bir bash script dosyası olsun:

.. code-block:: bash

    #!/bin/bash

    for i in {1..10}
    do
        echo $i
    done

Bu program 1'den 10'a kadar sayıları ekrana yazdırmaktadır. Normal olarak bir bash programı aşağıdaki
gibi çalıştırılır:

.. code-block:: console

    $ /bin/bash sample.sh

Burada ``sample.sh`` dosyasının ``x`` hakkına sahip olması gerekmez. Ancak biz dosyayı doğrudan aşağıdaki
gibi çalıştırmak isteyebiliriz:

.. code-block:: console

    $ ./sample.sh

Bu durumda dosyanın ``x`` hakkına sahip olması gerekir. Dosyayı böyle çalıştırmak istediğimizde kabuk
programı exec işlemi uygulayıp ``sample.sh`` programını çalıştırmak isteyecektir. Sistem fonksiyonu da
``sample.sh`` programının çalıştırılabilir bir dosya formatına sahip olmadığını anladığında shebang
satırına bakıp orada belirtilen ``/bin/bash`` programını çalıştıracaktır. Ancak bu programa script
dosyasının kendisini argüman olarak geçirecektir. Yani program adeta şöyle çalıştırılmış olacaktır:

.. code-block:: console

    $ /bin/bash sample.sh

Peki ``/bin/bash`` programı buradaki ``sample.sh`` programını çalıştırırken onun başındaki shebang
satırı bir soruna yol açmayacak mı? İşte betik dilleriyle, yorumlayıcılarla çalışılan dillerin hemen
hepsinde ``#`` özellikle bu shebang kullanımını desteklemek için yorum satırı biçiminde ele alınmaktadır.
Aynı durum Python, Perl, sed, awk gibi dillerde de böyledir.

Python Betiği Örneği (sample.py)
--------------------------------

Şimdi bir Python programını shebang ile çalıştıralım. Programın ismi ``sample.py`` olsun:

.. code-block:: python

    #!/usr/bin/python3

    for i in range(10):
        print(i)

Bu dosyaya ``x`` vererek biz artık onu komut satırından çalıştırabiliriz:

.. code-block:: console

    $ ./sample.py

.. code-block:: python

    #!/usr/bin/python3

    for i in range(10):
        print(i)

make Betiği Örneği (sample.mak)
-------------------------------

Aşağıdaki örnekte bir ``make`` dosyası shebang yoluyla çalıştırılmaktadır:

.. code-block:: makefile

    #!/bin/make -f

    sample: sample.o
        gcc -o sample sample.o
    sample.o: sample.c
        gcc -c sample.c

    clean:
        rm -f *.o
        rm -f sample

Burada dosyanın ``sample.mak`` isminde olduğunu varsayalım. Bu dosyaya ``x`` hakkını verdikten sonra onu
aşağıdaki gibi çalıştırmış olalım:

.. code-block:: console

    $ ./sample.mak

Bu çalıştırma aslında aşağıdakiyle eşdeğer olacaktır:

.. code-block:: console

    $ /bin/make -f sample.mak

shebang ve Dosya Formatı Kontrolü Sırası
========================================

Linux çekirdeklerinde exec fonksiyonları genel olarak (bazı ayrıntıları da vardır) önce shebang kontrolü
yapıp sonra ELF dosyası kontrolünü (ve diğer bazı çalıştırılabilir dosya formatlarının kontrolünü)
yapmaktadır. Ancak aslında bu sıranın da bir önemi yoktur. Çünkü ELF gibi çalıştırılabilir dosya
formatlarının ilk bayt'larında *sihirli sayılar (magic numbers)* vardır. Bu sihirli sayılarla ``#!``
shebang karakterleri zaten çakışmamaktadır.

Shebang satırında bazı şeylere de dikkat etmek gerekir. Örneğin shebang karakterlerinin hemen ilk satırın
başından başlatılması gerekir. Aksi takdirde exec fonksiyonlarının p'siz versiyonları (izleyen
paragrafta ayrıntıları göreceksiniz) dosya çalıştırılabilir bir dosya değilse ve dosyanın ilk iki
karakteri ``#!`` biçiminde de değilse ``ENOEXEC`` ile başarısız olmaktadır. Eğer exec fonksiyonları
shebang karakterlerinin yanındaki dosyayı bulamazsa bu durumda ``ENOENT`` errno değeri ile başarısız
olmaktadır.

exec'in p'li Versiyonlarında shebang'siz Betik Çalıştırma
=========================================================

exec fonksiyonlarının p'li versiyonları (yani ``execlp`` ve ``execvp``) özel bir davranışa sahiptir.
Bilindiği gibi bu fonksiyonlar ``PATH`` çevre değişkeninde belirtilen dizinlerde exec yapılan dosyayı tek
tek aramaktadır. Eğer bunlar betik dosyasını (ELF dosyasını değil) ``x`` hakkına sahip olarak bulup ancak
dosyanın başında *shebang* görmezlerse sanki dosyanın başında varmış gibi onları işleme sokmaktadır:

.. code-block:: text

    #!/bin/sh

Buradan şu sonuç çıkmaktadır: exec fonksiyonlarının p'li versiyonları ile bir shell script dosyasını biz
başında shebang satırı olmadan da çalıştırabiliriz. Ancak exec fonksiyonlarının p'siz versiyonlarında
bunu yapamayız. Öte yandan Linux sistemlerinde zaten ``execve`` dışındaki exec fonksiyonlarının sistem
fonksiyonu olmadığını anımsayınız. O halde exec fonksiyonlarının p'li versiyonları tamamen kullanıcı
modunda script dosyasını ``execve`` yaptıktan sonra ``ENOEXEC`` errno değeri ile fonksiyonun başarısız
olduğunu gördüklerinde bu kez ``/bin/sh`` dosyasını ``execve`` ile exec yapmaktadır. Dosya isminin
içerisinde ``/`` karakteri kullanılsa bile exec fonksiyonlarının p'li versiyonlarının davranışı yine bu
biçimdedir. Tabii bu durumda ``PATH`` çevre değişkenine başvurulmamaktadır. exec fonksiyonlarının p'li
versiyonlarının bu davranışı POSIX'te eskiden isteğe bağlı bırakılmıştı. Ancak sonra standartlarda bu
davranış zorunlu tutulmuştur. Ancak POSIX standartları çalıştırılacak kabuk programının ne olacağı
konusunda bir belirlemede bulunmamıştır.

Yukarıdaki açıklamalarımızdan çıkan bir sonuç şudur: Biz kabuk üzerinde kabuk betiğini aslında başında
hiç shebang satırı olmadan da çalıştırabiliriz. Çünkü kabuk exec fonksiyonlarının p'li versiyonlarını
kullanmaktadır. Aşağıdaki gibi ``x`` verilmiş ``myscript`` isminde bir Bash betik dosyası olsun:

.. code-block:: bash

    for i in {1..10}; do
        echo "$i"
    done

Biz bu dosyayı başında shebang satırı olmadığı halde kabuk üzerinden çalıştırabiliriz:

.. code-block:: console

    $ ./myscript
    1
    2
    3
    4
    5
    6
    7
    8
    9
    10

shebang'in Özyinelemeli Olması
==============================

Peki shebang satırında belirtilen dosyanın kendisi de bir betik dosyası olabilir mi? Yani bu shebang
işlemi özyinelemeli midir? Aslında POSIX standartları bu konuda bir şey söylememiştir. Bu durumda böyle
bir işlemin özyinelemeli yapılacağının bir garantisi yoktur. Linux çekirdeği bu tür durumlarda dört
kademeye kadar özyineleme yapabilmektedir.

system Fonksiyonu
=================

``system`` isimli ilginç bir standart C fonksiyonu vardır. Bu fonksiyon ilgili sistemdeki kabuk
programını (command interpreter) interaktif olmayan modda (non-interactive shell) çalıştırarak bizim
verdiğimiz bir kabuk komutunun kabuk tarafından işletilmesini sağlar. Böylece biz kabuk üzerinde
çalıştırabildiğimiz tüm komutları bir C programının içerisinde bu yolla çalıştırabiliriz. ``system``
fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <stdlib.h>

    int system(const char *command);

Fonksiyon parametre olarak kabuğa işletilecek komut yazısını almaktadır. Tabii ``system`` bir standart C
fonksiyonu olduğuna göre yalnızca UNIX/Linux sistemlerinde değil diğer tüm sistemlerde de
kullanılabilmektedir. Örneğin ``system`` fonksiyonu UNIX/Linux sistemlerinde ``/bin/sh`` programını
çalıştırırken, Windows sistemlerinde ``cmd.exe`` programını çalıştırmaktadır. Tabii bir sistemde kabuk
programı bulunuyor olmak zorunda da değildir. Örneğin pek çok gömülü sistemde bir işletim sistemi
olmadığı için kabuk programı da yoktur. İşte programcı ilgili sistemde kabuk programının olup olmadığını
fonksiyonun parametresine ``NULL`` adres geçerek test edebilir. Bu durumda ``system`` fonksiyonu eğer
ilgili sistemde kabuk programı varsa sıfır dışı bir değere, yoksa 0 değerine geri dönmektedir. Tabii
programcı Windows, UNIX/Linux ve macOS gibi sistemlerde çalışıyorsa böyle bir kontrol yapmaz.

system Fonksiyonunun Çalışma Mantığı ve Geri Dönüş Değeri
---------------------------------------------------------

Daha önceden de belirttiğimiz gibi pek çok sistemde kabuk programları *interaktif olmayan
(noninteractive)* bir modda çalıştırılabilmektedir. Örneğin UNIX/Linux kabuk programları ``-c`` seçeneği
ile çalıştırılırsa yalnızca bir komutu çalıştırıp sonlanmaktadır. Örneğin:

.. code-block:: console

    $ bash -c "ls -l; cat sample.c"

Benzer biçimde Windows sistemlerinde de ``cmd.exe`` kabuk programı ``/C`` seçeneği ile benzer biçimde
çalıştırılabilmektedir.

O halde UNIX/Linux sistemlerinde ``system`` fonksiyonu kabuk programını ``-c`` seçeneği ile fork/exec
yoluyla çalıştırmaktadır. Eğer ``system`` fonksiyonu ``fork`` ya da ``wait`` işleminde başarısız olursa
-1 değeri ile geri dönmektedir. Eğer ``fork`` yapıp exec başarısız olursa sanki ``_exit(127)`` biçiminde
oluşturulan ve ``waitpid`` fonksiyonu ile elde edilen değere (status) geri dönmektedir. (Yani başarısız
olursa geri dönüş değeri hem sonlanma bilgisini hem de çıkış kodunu içermektedir.) Diğer durumlarda
(yani ``fork`` ve exec başarılı bir biçimde yapılmışsa) ``system`` fonksiyonu çalıştırdığı kabuk
programının ``waitpid`` fonksiyonuyla elde edilen değerine (status) geri dönmektedir. (Yani başarı
durumunda ``system`` fonksiyonu kabuğun status değeriyle geri dönmektedir.) Tabii kabuk programları da
interaktif olmayan modda çalıştırılan komutun ``waitpid`` fonksiyonu ile elde edilen status değerine geri
dönerler. Bu durumda başarı durumunda aslında kabuktan çalıştırılan komutun (yani programın) status
değeri elde edilmektedir. Örneğin çağrı şöyle yapılmış olsun:

.. code-block:: c

    system("ls -l");

Burada ``system`` fonksiyonu fork/exec ile ``/bin/sh`` programını ``-c`` seçeneği ile çalıştırmaktadır.
Tabii kabuk programı da ``ls`` programını fork/exec ile çalıştıracaktır. (Bazen kabuk programları
interaktif modda eğer tek bir komut işletiliyorsa boşuna fork yapmayabilir.) Burada kabuk programı aslında
``ls`` programının ``waitpid`` fonksiyonu ile elde edilen değer (status) ile sonlanmaktadır. Dolayısıyla
biz aslında ``system`` fonksiyonunun geri dönüş değeri olarak çalıştırdığımız ``ls`` programının
``waitpid`` fonksiyonu ile elde edilen (status) değerini elde etmiş oluruz. UNIX/Linux sistemlerinde
genel olarak kabuk komutları (yani programları) başarı durumunda exit kodu olarak 0 değerini
oluşturmaktadır.

system Başarı Kontrolü
----------------------

Peki ``system`` fonksiyonunun başarısını nasıl kontrol etmeliyiz? Biz fonksiyonun geri dönüş değerini -1
ve 0'dan farklılık ile test edebiliriz. Örneğin:

.. code-block:: c

    result = system("ls -l");

    if (result == -1 || (WIFEXITED(result) && WEXITSTATUS(result) != 0) || WIFSIGNALED(result)) {
        fprintf(stderr, "command failed!..\n");
        exit(EXIT_FAILURE);
    }

Biz kabuk programını ``system`` fonksiyonu ile çalıştırırken komutlar arasına ``;`` koyarak birden fazla
komutun çalıştırılmasını sağlayabiliriz. Genel olarak kabuk bu durumda son komutun status değerini bize
vermektedir.

Aslında programcılar genellikle ``system`` fonksiyonu için yalnızca -1 kontrolünü yapmaktadır. Yani
çalıştırdıkları komutun başarısını kontrol etmemektedir. Örneğin:

.. code-block:: c

    if (system("any command") == -1)
        exit_sys("system");

``system`` fonksiyonu POSIX standartlarında ``errno`` değişkenini set etmektedir. POSIX standartlarına
göre fonksiyonun -1 değeri ile geri döndüğünde ``errno`` değişkeni ancak ``ECHILD`` değeri ile set
edilmektedir.

Aşağıda ``system`` fonksiyonunun kullanımına bir örnek verilmiştir.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <sys/wait.h>

    int main(int argc, char *argv[])
    {
        int result;

        result = system("ls -l");
        if (result == -1 || (WIFEXITED(result) && WEXITSTATUS(result) != 0) || WIFSIGNALED(result)) {
            fprintf(stderr, "command failed!..\n");
            exit(EXIT_FAILURE);
        }

        printf("Success...\n");

        return 0;
    }

system Fonksiyonunun Kendi Gerçekleştirimi (mysystem)
-----------------------------------------------------

Peki ``system`` fonksiyonunu nasıl yazabiliriz? Aşağıda buna bir örnek verilmiştir. Ancak aşağıdaki
örnekte bazı noktalar henüz kursumuzda o konu anlatılmadığı için ihmal edilmiştir. Bu noktalar şunlardır:

- ``waitpid`` fonksiyonu sinyalle kesilirse yeniden çalıştırılması (restart edilmesi) gerekir.
- Üst prosesin işlemler sırasında ``SIGCHLD`` sinyalini, ``SIGINT`` ve ``SIGQUIT`` sinyallerini bloke
  etmesi gerekmektedir.

Bu konular kursumuzda *sinyaller (signals)* konusu içerisinde ileride ele alınacaktır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>

    void exit_sys(const char *msg);

    int mysystem(const char *command)
    {
        pid_t pid;
        int status;

        if (command == NULL)
            return 1;

        if ((pid = fork()) == -1)
            return -1;

        if (pid == 0) {
            if (execl("/bin/sh", "/bin/sh", "-c", command, (char *)0) == -1)
                _exit(127);
            /* unreachable code */
        }
        if (waitpid(pid, &status, 0) == -1)
            return -1;

        return status;
    }

    int main(void)
    {
        int result;

        result = mysystem("ls");

        if (result == -1 || WIFEXITED(result) && WEXITSTATUS(result) != 0) {
            fprintf(stderr,"command failed!...\n");
            exit(EXIT_FAILURE);
        }

        printf("Ok\n");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

system mi fork/exec mi?
=======================

Peki mademki ``system`` fonksiyonu bizim için zaten fork/exec işlemlerini yapmaktadır, bu durumda
örneğin bir programı çalıştırmak için biz fork/exec kullanmak yerine bu işlemi ``system`` fonksiyonu ile
yapamaz mıyız? Evet aslında yapabiliriz. Ancak bu konudaki her türlü gereksinimimizi ``system``
fonksiyonu karşılayamaz. Örneğin ``fork`` işleminden sonra alt proseste ayarlamalar yapıp exec yapmak
isteyebiliriz. Ayrıca ``system`` fonksiyonu kendi içerisinde kabuk programını çalıştırdığı için daha
yavaş ve daha fazla kaynak kullanır durumdadır. Bizim tavsiyemiz bir programı açıkça fork/exec ile
çalıştırmanız, ancak karmaşık işlemleri (örneğin IO yönlendirmesi, boru vs. gibi) ``system`` fonksiyonuyla
yapmanızdır.

system ile Basit Bir Kabuk Sarmalayıcısı Örneği
===============================================

Aşağıdaki örnekte kabuk programı ``system`` fonksiyonu sayesinde sarmalanmıştır. Tabii komut satırından
komut alıp onu ``system`` fonksiyonu yoluyla asıl kabuk programına çalıştırmak gerçek anlamda bir kabuk
yazmak anlamına gelmemektedir. Bu bir sarmalama (wrapping) işlemidir.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <string.h>

    int main(void)
    {
        char cmd[4096];
        char *str;

        for (;;) {
            printf("CSD>");
            fflush(stdout);

            if (fgets(cmd, 4096, stdin) != NULL)
                if ((str = strchr(cmd, '\n')) != NULL)
                    *str = '\0';
                if (!strcmp(cmd, "exit"))
                    break;
            if (system(cmd) == -1)
                perror("system");
        }

        return 0;
    }


.. _set-user-id-set-group-id-sticky:

=================================================================
set-user-id, set-group-id, sticky Bayrakları ve Proses Kimlikleri
=================================================================

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


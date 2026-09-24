=======================================
Proseslerin Yaratılması ve Yok Edilmesi
=======================================

Anımsanacağı gibi işletim sistemlerinde çalışmakta olan programlara "proses" denilmektedir. Her proses başka bir proses 
tarafından yaratılmaktadır. Bu bölümde dikkatimizi prosesler üzerinde üzerine çevireceğiz. 

Proses ID (PID) Kavramı ve pid_t Türü
=====================================

UNIX/Linux sistemlerinde her prosesin o anda "sistem genelinde tek olan (*unique*)" bir *proses ID (PID)* değeri
vardır. PID değeri prosesin kontrol bloğuna erişmek için bir anahtar olarak kullanılmaktadır. Yani biz işletim 
sistemine bu PID değerini verdiğimizde işletim sistemi hızlı bir biçimde bu PID değerinden hareketle prosesin 
kontrol bloğuna erişebilmektedir.

PID değerleri ``pid_t`` türüyle temsil edilmiştir. POSIX standartlarına göre ``pid_t`` türü işaretli 
bir tamsayı türü olmak koşuluyla ``<sys/types.h>`` ve ``<unistd.h>`` dosyalarında typedef edilmiş olmak zorundadır. 
Bu türün hangi işaretli tamsayı türü olarak typedef edildiğinin programcı tarafından bilinmesine gerek ypktur. 

Sistem boot edildiğinde boot kodu ``0`` numaralı PID'ye sahip proses biçimine dönüştürülmektedir. Buna *swapper*
ya da *pager* da denilebilmektedir. Daha sonra da bu ``0`` numaralı PID bir daha sistemde kullanılmamaktadır.
(Yani ``0`` numaralı PID geçerli bir PID değildir.) Sistemde ikinci yaratılan proses ``1`` numaralı ID'ye sahip 
olan *init* ismiyle temsil edilen prosestir. ``0`` numaralı proses yok edildiği için sistemdeki bütün proseslerin 
atası bu *init* prosesidir. *init* prosesi arka planda bir *daemon* gibi çalışmaktadır. UNIX/Linux dünyasında 
terminal etkileşimi olmayan arka planda çalışan proseslere *daemon* denilmektedir. (Bu tür proseslere Windows 
dünyasında da *servis (service)* denilmektedir.)

İşletim sisteminin çekirdeği tipik olarak yeni yaratılan proses için proses PID değerini bir sayaç
kullanarak vermektedir. Her proses yaratıldığında bu sayaç değeri bir artırılır. Sayaç sona geldiğinde
yeniden başa geçilir ve bitmiş proseslerin PID'leri kullanılır. Örneğin sistem açıldığında ``1`` numaralı PID'ye
sahip *init* prosesi tarafından yaratılan proseslere artık sistem ``2``, ``3``, ``4``, ... PID'lerini vermektedir.
Proses PID değerlerinin belli bir anda sistemde tek olduğuna dikkat ediniz. Zaman içerisinde
bilgisayarınız uzun süre açık kalırsa sonlanmış olan eski PID değerleri yeniden kullanılacaktır.

PID'ler İçin Tavan Değeri
-------------------------

UNIX/Linux sistemlerinde genellikle PID değerleri için bir tavan değer de belirlenmektedir. Bu tavan
değere ulaşıldığında yukarıda da belirttiğimiz gibi yeniden başa dönülüp boş olan PID'ler kullanılmaktadır. 
Linux sistemlerinde default durumda PID tavan değeri ``32768``'dir. Bu değer sistem yöneticisi tarafından
değiştirilebilmektedir. Ancak yükseltilecek maksimum değer de önceden belirlenmiştir. Aşağıda bu değerleri
tablo halinde veriyoruz:

.. list-table::
   :header-rows: 1

   * - Sistem
     - PID_MAX_DEFAULT
     - PID_MAX_LIMIT (üst sınır)
   * - 32-bit
     - 32768
     - 32768 (0x8000)
   * - 64-bit
     - 32768
     - 4194304 (4*1024*1024)

Çalışan sistemdeki PID tavan değerini ``/proc/sys/kernel/pid_max`` dosyasından görüntüleyebilirsiniz.
Bunun için ``sysctl`` komutunu da kullanabilirsiniz:

.. list-table::
   :header-rows: 1

   * - Yöntem
     - Komut / Dosya
     - Açıklama
   * - Çalışma zamanı
     - ``/proc/sys/kernel/pid_max``
     - Sistemde o an geçerli üst sınır
   * - sysctl
     - ``sysctl kernel.pid_max``
     - Aynı değerin sysctl arayüzü
   * - Değiştirme
     - ``sysctl -w kernel.pid_max=N``
     - Root yetkisi gerekir

Her ne kadar çekirdekteki default değer ``32768`` olsa da *systemd* init sistemi açılış sırasında bu değeri
maksimum değer olan ``4194304`` değerine çekmektedir. Örneğin:

.. code-block:: console

    $ cat /proc/sys/kernel/pid_max
    4194304

Proses ID'lerine İlişkin Fonksiyonlar

getpid ve getppid Fonksiyonları
===============================

O anda çalışmakta olan programa ilişkin PID değeri ``getpid`` isimli POSIX fonksiyonu ile elde
edilebilmektedir:

.. code-block:: c

    #include <unistd.h>

    pid_t getpid(void);

Fonksiyon başarısız olamaz. Bu değeri şöyle yazdırabiliriz:

.. code-block:: c

    printf("%jd\n", (intmax_t)getpid());

``pid_t`` türünün gerçekte hangi tür olduğunu bilmediğimiz için onu sistemde olabilecek en büyük işaretli
tamsayı türünü temsil eden ``intmax_t`` türüne dönüştürdük.

.. code-block:: c

    #include <stdio.h>
    #include <stdint.h>
    #include <unistd.h>

    int main(void)
    {
        pid_t pid;

        pid = getpid();
        printf("%jd\n", (intmax_t)pid);

        return 0;
    }

Her proses başka bir proses tarafından yaratılmaktadır. Bir prosesi yaratan prosese o prosesin *üst
prosesi (parent process)*, yaratılan prosese de üst prosesin *alt prosesi (child process)* denilmektedir.
Her prosesin bir üst prosesi vardır. Bir prosesin üst prosesi ``getppid`` fonksiyonu ile elde
edilmektedir. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    pid_t getppid(void);

Bu fonksiyon da başarısız olamamaktadır.

Biz programı kabuk üzerinden çalıştırdığımızda çalıştırdığımız prosesin üst prosesi kabuk prosesi
(muhtemelen bash) olacaktır.

Bir prosesin üst prosesi sonlanırsa bu tür proseslere *öksüz (orphan) prosesler* denilmektedir. Sistem
böyle bir durumda ``1`` numaralı ID'ye sahip olan *init* ismiyle temsil ettiğimiz prosesi öksüz duruma düşmüş
prosesin üst prosesi olarak atamaktadır. Dolayısıyla her zaman prosesin bir üst prosesi bulunmaktadır.

.. code-block:: c

    #include <stdio.h>
    #include <stdint.h>
    #include <unistd.h>

    int main(void)
    {
        pid_t pid, ppid;

        pid = getpid();
        printf("pid = %jd\n", (intmax_t)pid);

        ppid = getppid();
        printf("ppid = %jd\n", (intmax_t)ppid);

        return 0;
    }

ps Komutu ile Proseslerin Görüntülenmesi
========================================

UNIX/Linux sistemlerinde o anda sistemde bulunan prosesler hakkında bilgiler ``ps`` isimli POSIX komutuyla
elde edilmektedir. Linux sistemlerinde ``ps`` komutu ``proc`` dosya sistemini kullanmaktadır. ``ps`` komutu
oldukça ayrıntılı bir komuttur ve pek çok seçeneğe sahiptir. Komutu seçeneksiz kullanırsak yalnızca çalışan
terminaldeki prosesler görüntülenmektedir. Örneğin:

.. code-block:: console

    $ ps
    PID TTY          TIME CMD
    9804 pts/0    00:00:00 bash
    10127 pts/0    00:00:00 ps

Kullanıcının bütün terminallerdeki proseslerini görmek için ``-u`` seçeneği kullanılmaktadır. ``-u``
seçeneğinin yanına kullanıcı ismi de getirilebilmektedir. Örneğin:

.. code-block:: console

    $ ps -u
    USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
    kaan        9804  0.0  0.1  14276  5844 pts/0    Ss+  11:04   0:00 bash
    kaan       10094  0.0  0.1  14144  5664 pts/1    Ss   11:27   0:00 bash
    kaan       10131  200  0.1  16532  4728 pts/1    R+   11:35   0:00 ps -u

``-l`` seçeneği prosesler hakkında daha ayrıntılı bilgiler vermektedir. Örneğin:

.. code-block:: console

    $ ps -l
    F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    0 S  1000   10094    9793  0  80   0 -  3536 do_wai pts/1    00:00:00 bash
    4 R  1000   10139   10094  0  80   0 -  4142 -      pts/1    00:00:00 ps

Görüldüğü gibi ``-l`` seçeneği ile prosesin "etkin kullanıcı ID'si", "etkin grup ID'si", "üst proses
ID'si", "bağlantılı olduğu terminal" bilgileri de verilmektedir.

Sistemdeki tüm prosesleri ``-e`` seçeneğiyle görüntüleyebiliriz. Örneğin:

.. code-block:: console

    $ ps -e
    PID TTY          TIME CMD
      1 ?        00:00:02 systemd
      2 ?        00:00:00 kthreadd
      3 ?        00:00:00 pool_workqueue_release
      4 ?        00:00:00 kworker/R-rcu_gp
      5 ?        00:00:00 kworker/R-sync_wq
      6 ?        00:00:00 kworker/R-kvfree_rcu_reclaim
      7 ?        00:00:00 kworker/R-slub_flushwq
    ....

Belli bir terminalden çalıştırılmış olan prosesleri ``-t`` seçeneği ile görüntüleyebiliriz:

.. code-block:: console

    $ ps -t pts/1
      PID TTY          TIME CMD
    10094 pts/1    00:00:00 bash
    10168 pts/1    00:00:00 ps

Biz kursumuzda yeri geldikçe ``ps`` komutunun diğer bazı seçenekleri üzerinde de açıklamalar yapacağız.

Yaratılabilecek Proses ve Thread Sayısına İlişkin Limitler 
==========================================================

Sistemlerde prosesler konusunda bazı limitler söz konusu olabilmektedir. Çünkü her proses bir kaynak
kullanmaktadır. Bu kaynakların da bir limiti vardır. Örneğin Linux sistemlerinde, sistem genelinde aynı
anda var olabilecek toplam proseslerin sayısı (bunu proses ID'lerinin alabileceği tavan değerle
karıştırmayınız) ``/proc/sys/kernel/threads-max`` dosyasında belirtilmektedir. Burada belirtilen değer
*toplam proseslerin ve thread'lerin* sayısıdır. (Linux sistemlerinde aslında thread'ler de prosesler gibi 
kaynak kullanmaktadır. Çekirdek alanında proseslerle thread'ler aynı veri yapısıyla temsil edilmektedir.) 
Örneğin kursun yapıldığı sistemdeki aynı anda yaratılabilecek proseslerin ve thread'lerin maksimum sayısı 
şöyledir:

.. code-block:: console

    $ cat /proc/sys/kernel/threads-max
    30231

Bu değer ``sysctl`` komutu ile de elde edilebilir:

.. code-block:: console

    $ sysctl kernel.threads-max
    kernel.threads-max = 30231

Bu değer çalışmakta olan makinedeki fiziksel bellek miktarına bağlı olarak da değişebilmektedir. Bu değer
``proc`` dosya sistemi yoluyla ya da ``sysctl`` komutu ile değiştirilebilmektedir.

Yine UNIX/Linux sistemlerinde belli bir kullanıcının yaratabileceği maksimum proses ve thread sayısı da
sınırlandırılmaktadır. Eğer böyle bir sınırlandırma yapılmasaydı sıradan bir kullanıcı sistemdeki tüm
proses ve thread kapasitesini kendi başına kullanıp diğer kullanıcıları zor durumda bırakabilirdi.
Kullanıcının yaratabileceği maksimum proses ve thread sayısı ``getrlimit`` POSIX fonksiyonuyla ya da
``ulimit -u`` kabuk komutuyla elde edilebilir. Tabii uygun önceliğe sahip olan prosesler (proses ID'si ``0`` olan prosesler 
ve Linux sistemlerinde bu *yetenekliliğe (capability)* sahip olan prosesler) bu sınırlamaya tabi değildir. (Bu konu
ileride "process kaynak limitlerinin" anlatıldığı bölümde ayrıntılarıyla ele alınacaktır.) Uygun önceliğie sahip 
prosesler genel olarak kaynakların *hard limitlerini* yükseltebilmektedir. Aşağıda kursun yapıldığı sistemde sıradan
bir kullanıcının limiti gösterilmiştir:

.. code-block:: console

    $ ulimit -u
    15115

Ancak bu limitler de makineden makineye de değişebilmektedir. Çünkü sistemdeki fiziksel RAM miktarıyla da
ilişkilidir.

Bu değerler aslında o anda ya da kalıcı olarak değiştirilebilmektedir. Bu değerlerin boot edilene kadar
değiştirilmesi *proc* girişlerine yeni değerlerin yazılmasıyla ya da ``sysctl`` komutu ile de yapılabilir.
Kalıcı değişiklikler için sistem boot edilirken başvurulan bazı konfigürasyon dosyalarından
faydalanılmaktadır. Örneğin ``/etc/sysctl.conf`` dosyasına yeni limitler girilirse sistem bu limitlerle
açılacaktır.

Prosesleirn Yaratılması: fork Fonksiyonu
========================================

UNIX/Linux sistemlerinde prosesler ``fork`` isimli POSIX fonksiyonu ile yaratılmaktadır. ``fork``
fonksiyonu pek çok UNIX türevi sistemde doğrudan işletim sisteminin bu işi yapan sistem fonksiyonunu
çağırmaktadır. Linux sistemlerinde ``sys_fork`` ve bunun daha genel biçimi olan `
``sys_clone`` sistem fonksiyonları bu işi yapmaktadır. ``fork`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    pid_t fork(void);

.. note::

    İngilizce ``fork`` sözcüğü Türkçe *çatal* anlamına gelmektedir. *Akışın çatallanması* gibi bir benzetmeye
    dayanılarak bu isim uydurulmuştur.

``fork`` bir prosesin tamamen özdeş bir kopyasını oluşturmaktadır. ``fork`` fonksiyonunun çalışmasını şöyle 
bir analoji ile daha iyi anlayabiliriz. Diyelim ki bir klonlama makinesi var. Bu makine insandaki tüm atomları 
bire bir kopyalayarak yeni bir klon oluşturuyor olsun. Bu durumda klonlama makinesine bir kişi girdiğinde makineden 
iki kişi çıkmaktadır. Makineden çıkan iki kişinin tüm geçmiş yaşantıları, her şeyi aynı olacaktır. Makineden 
çıktığında her iki kişi de kendini gerçek kopya sanabilir. Çünkü klonlama sırasında tüm atomlar (bellek 
mekanizması nöral düzeyde işlev görmektedir, nöronlar da neticede atomlardan oluşmaktadır) kopyalanmıştır. 
Makineden çıkan iki kişinin her şeyi aynıdır. Ancak makineden çıktıktan sonra artık bunlar farklı olaylarla 
karşılaşacağı için bellek ve deneyim olarak farklılaşacaklardır. Burada her ne kadar makineden çıkan iki 
kişi de kendisinin orijinal kopya olduğunu sanıyorsa da klonlamayı yapan operatör kimin orijinal kopya kimin 
onun kopyası olduğunu bilmektedir. 

İşte ``fork`` fonksiyonunun çalışması yukarıdaki analojiye çok benzemektedir. ``fork`` fonksiyonuna giren akış 
orada yeni ve özdeş bir prosesin yaratılmasına yol açmaktadır. Yeni yaratılan prosesin bellek alanı ve proses 
kontrol bloğu büyük ölçüde üst prosesten kopyalanmaktadır. Yeni yaratılan prosesin akışı ``fork`` içerisinden 
başlatılmaktadır. Böylece asıl proses ile yeni yaratılan prosesin her ikisi de ``fork`` fonksiyonundan çıkar. 
Asıl proses ile yeni yaratılan proses aynı bellek alanına (yani kabaca aynı *code*, *data*, *stack* ve *heap* 
alanlarına) sahiptir. ``fork`` çağrısını yapan asıl prosese *üst proses (parent process)*, ``fork`` ile 
yaratılan yeni prosese ise *alt proses (child process)* denilmektedir. Tabii üst proses de aslında ``fork`` 
çağrısı ile başka bir proses tarafından yaratılmıştır.

``fork`` fonksiyonunda kabaca şunlar yapılmaktadır:

| **1.** Alt proses için yeni bir proses kontrol bloğu yaratılır. ``fork`` işlemini yapan prosesin (üst proses)
   proses kontrol bloğunun içeriği yeni yaratılan prosesin (alt proses) proses kontrol bloğuna kopyalanır.
   Böylece üst proses ile yeni yaratılan alt proses tamamen aynı özelliklere sahip olmaktadır. Örneğin bu
   iki prosesin "etkin ve gerçek kullanıcı ve grup ID'leri", "çalışma dizinleri (current working
   directories)", "açmış olduğu dosyalara ilişkin bilgiler" aynı olur.

| **2.** Çekirdek alt proses için üst prosesin bellek alanının özdeş kopyasını oluşturur. Böylece her iki proses
   de içerik olarak aynı *kod* alanına, aynı *data* alanına, aynı *stack* alanına ve aynı *heap* alanına sahip olacaktır.
   Ancak bunlar birbirlerinden ayrıdır.

| **3.** Yeni yaratılan alt proses çalışmaya ``fork`` fonksiyonunun içinden başlar. Her iki akış da (yani
   ``fork`` uygulayan prosesin akışı ve yeni yaratılan alt prosesin akışı da) ``fork`` içerisinden çıkar. Üst
   proses ``fork`` fonksiyonundan alt prosesin proses ID'si ile çıkarken alt proses ``0`` ile çıkmaktadır.

Yukarıdaki işlemler ``fork`` fonksiyonunun içinde yapılmaktadır. ``fork`` fonksiyonundan hem bu fonksiyonu
çağıran proses hem de yeni yaratılan proses çıkmaktadır. Ancak bunların bellek alanları ayrı olduğu için
artık birinin yapacağı değişikliği diğeri görmeyecektir. ``fork`` fonksiyonu, bir klonlama yapmaktadır.
Yeni bir prosesi, kendi çağıran prosesle aynı özelliklerle ve aynı bellek alanı ile yaratmaktadır. ``fork``
sırasında prosesin kontrol bloğu yeni yaratılan prosese kopyalandığı için üst proses ile alt proses aynı
gerçek ve etkin kullanıcı ve grup ID'sine sahip olur. 

``fork`` işlemini yapan proses üst proses (parent process) durumundadır. Yeni yaratılan proses ise alt
proses (child process) durumundadır. Tabii alt proses yeni bir proses ID'ye sahip olacaktır. Alt prosesin
üst prosesi, ``fork`` fonksiyonu uygulayan proses olacaktır. 

``fork`` fonksiyonu başarısız olabilir. (Örneğin kaynak yetersizliği durumunda, kullanıcının proses yaratma 
limiti aşıldığı durumda ``fork`` başarısız olabilir.) ``fork`` başarısızlık durumunda ``-1`` değerine geri dönmektedir.

Peki yeni proses hangi noktada yaratılmaktadır? Tabii ``fork`` fonksiyonu içerisinde. Yukarıda da
belirttiğimiz gibi yeni yaratılan prosesin (alt prosesin) akışı da ``fork`` fonksiyonun içerisinden
başlatılacaktır. Bu durumda her iki proses de ``fork`` fonksiyonunun içerisinden çıkacaktır. İşte üst
proses (yani ``fork`` işlemini yapan proses) *alt prosesin ID* değeri ile, alt proses ise *0 değeri ile*
``fork`` fonksiyonundan çıkar. Böylece programcı ``fork`` çıkışında üst proses ile alt prosese farklı
işlemler yaptırabilmektedir. (Alt prosesin ``fork`` içerisinden ``0`` ile çıkması alt prosesin proses ID'sinin
``0`` olduğu anlamına gelmemektedir. Alt prosesin proses ID'si alt proses içerisinden ``getpid`` fonksiyonuyla
elde edilebilmektedir.)

fork Kullanımına İlişkin Tipik Kalıp
------------------------------------

``fork`` kullanımına ilişkin tipik kalıbı şöyledir:

.. code-block:: c

    pid_t pid;
    /* ... */

    if ((pid = fork()) == -1)
        exit_sys("fork");

    if (pid != 0) {        /* parent process */
        /* ... */
    }
    else {                /* child process */
        /* ... */
    }

Aşağıdaki örnekte ``fork`` fonksiyonu ile bir proses yaratılmış ve proses ID'ler üst ve alt proseslerde
yazdırılmıştır. Bu örnekte üst proseste ``fork`` fonksiyonunun alt prosesin proses ID değeri ile geri
döndüğüne dikkat ediniz. Denemenin yapıldığı makinede şöyle bir sonuç elde edilmiştir:

.. code-block:: text

    Parent PID: 549376
    Parent's parent PID: 536568
    fork return value: 549377
    common code...
    Child PID: 549377
    Child's parent PID: 549376
    Common code...

Görüldüğü gibi alt prosesin üst proses ID'si ile üst prosesin proses ID'si aynıdır. Programı kabuk
üzerinden çalıştırdığımız için üst prosesin üst proses ID'si kabuk prosesinin (tipik olarak bash) proses
ID'si olacaktır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <stdint.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        pid_t pid;

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid != 0) {        /* parent process */
            printf("Parent PID: %jd\n", (intmax_t)getpid());
            printf("Parent's parent PID: %jdd\n", (intmax_t)getppid());
            printf("fork return value: %jd\n", (intmax_t)pid);
        }
        else {                /* child process */
            printf("Child PID: %jd\n", (intmax_t)getpid());
            printf("Child's parent PID: %jd\n", (intmax_t)getppid());
        }

        printf("Common code...\n");

        sleep(1);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

fork Sonrası Bellek Ayrışması ve Ortak Kod
------------------------------------------

``fork`` işleminde en fazla kafa karıştıran noktalardan biri ``fork`` fonksiyonundan iki akışın da çıkması
durumudur. Burada genellikle yeni öğrenenlerin gözden kaçırdığı birkaç nokta vardır:

| **1.** ``fork`` sırasında ``fork`` işlemini yapan prosesin (yani üst prosesin) tüm bellek alanının, yani onun
   kod, data, stack ve heap alanlarının özdeş bir kopyası oluşturulmaktadır. Yani ``fork`` işlemini yapan
   prosesin kod, data, stack ve heap alanlarının hepsi alt proseste de bulunmaktadır. Örneğin:

   .. code-block:: c

       if ((pid = fork()) == -1)
           exit_sys("fork");

       if (pid != 0) {
           /* ... */
       }
       else {
           /* ... */
       }

Programcının yazdığı bu kod hem üst proses hem de alt proses tarafından çalıştırılmaktadır. Yani bu
kod hem üst proseste hem de alt proseste bulunacaktır. Bizim buradaki temel amacımız ``fork`` çıkışında
kodu aynı olan iki farklı prosese farklı şeyleri yaptırmaktır. İşte bunu ``fork`` fonksiyonunun geri
dönüş değerini kontrol ederek sağlayabilmekteyiz.

| **2.** Yeni öğrenen kişilere iki prosesin de ``fork`` fonksiyonundan çıkması tuhaf gelebilmektedir. Aslında
   burada bir tuhaflık yoktur. Şöyle ki: Prosesin yaratılması ve bellek alanlarının kopyalanması zaten
   ``fork`` içerisinde yapılmaktadır. ``fork`` fonksiyonunu çağıran proses (üst proses) ``fork``'tan
   çıkacaktır. Kopyası çıkartılan alt prosesin çalışması da ``fork`` içerisinden başlatılmaktadır. Bu
   durumda alt proses de ``fork`` fonksiyonundan çıkacaktır. Çatallanmanın ``fork`` içerisinde
   yapıldığına dikkat ediniz.

Tabii ``fork`` fonksiyonundan çıkınca artık üst proses ile alt prosesin yaşamları farklı olabilmektedir.
Örneğin üst proses bir global değişkenin değerini değiştirse alt proses bunu değişmiş olarak görmez. Çünkü
o global değişkenin üst proseste ve alt proseste artık farklı kopyaları vardır. Üst proses kendi global
değişkenini değiştirmektedir. Yani ``fork`` işleminden çıkıldığında üst ve alt prosesin her şeyi aynı olsa
da artık bunlar kendi yollarına gideceklerdir. (Klon makinesinden çıkar çıkmaz iki kişinin her şeyi aynıdır. 
Ancak bundan sonra bu kişiler bağımsız kişiler oldukları için başlarına farklı olaylar gelecektir. Birisinin 
maruz kaldığı bir duruma diğeri maruz kalmayacaktır.)

Aşağıdaki örnekte ``fork`` işlemi sonrasında üst proses ``g_x`` global değişkenine yeni bir değer
atamıştır. Sonra alt proseste bu global değişkenin değeri yazdırılmıştır. Tabii alt proses üst prosesin
yaptığı bu değişikliği görmeyecektir. Çünkü aslında iki prosesin de bellek alanları tamamen ``fork``
içerisinde klonlama yöntemiyle ayrıştırılmıştır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int g_x = 10;

    int main(void)
    {
        pid_t pid;

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid != 0) {        /* parent process */
            g_x = 100;
        }
        else {                /* child process */
            sleep(1);
            printf("%d\n", g_x);    /* 10 */
        }

        printf("Common code...\n");

        sleep(1);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

``fork`` işleminde yeni proses yaratıldığında hangi proses akışının ``fork`` fonksiyonundan önce
çıkacağının bir garantisi yoktur. Bu, işletim sisteminin çizelgeleme algoritmalarına bağlı olarak
değişebilmektedir.

Aşağıdaki örnekte *Common Code* yazısı ``8`` defa ekranda görünecektir:

.. code-block:: c

    fork();
    fork();
    fork();

    printf("Common code...\n");

Çünkü ilk ``fork`` işleminden sonra ikinci ``fork`` işlemini ``2`` proses yapacaktır. Böylece ikinci ``fork`` 
işleminden sonra aynı koda sahip ``4`` proses oluşacaktır. Sonra bu ``4`` proses de üçüncü ``fork`` işlemini yapacaktır. 
O halde üçüncü ``fork`` işleminden toplam ``8`` proses çıkacaktır. Buradaki ``sleep`` fonksiyonunu neden çağırdığımıza 
takılmayınız. Amacımız tüm yazma işlemleri bittikten sonra kabuğun prompt'una düşülmesini sağlamaktır. Kod karmaşık
olmasın diye ``fork`` çağrılarının geri dönüş değerlerini kontrol etmedik. Ancak uygulamada kontrol
etmelisiniz. Programı çalıştırdığımızda şöyle bir çıktı elde edeceğiz:

.. code-block:: text

    Common code...
    Common code...
    Common code...
    Common code...
    Common code...
    Common code...
    Common code...
    Common code...

.. code-block:: c

    #include <stdio.h>
    #include <unistd.h>

    int main(void)
    {
        fork();
        fork();
        fork();

        printf("Common code...\n");
        sleep(1);

        return 0;
    }

Benzer biçimde yine aşağıdaki kodda ekrana ``8`` tane ``3`` sayısı basılacaktır:

.. code-block:: c

    int a = 0;

    fork();
    ++a;
    fork();
    ++a;
    fork();
    ++a;

    printf("%d\n", a);

Burada prosesler aynı ``a`` değişkenini artırmamaktadır. ``fork`` işlemi ile proseslerin bellek alanları
kopyalandığı için her alt prosesin kendi ``a`` değişkeni vardır. Örneği bütün olarak aşağıda veriyoruz:

.. code-block:: c

    #include <stdio.h>
    #include <unistd.h>

    int main(void)
    {
        int a = 0;

        fork();
        ++a;
        fork();
        ++a;
        fork();
        ++a;

        printf("Common code: %d\n", a);
        sleep(1);

        return 0;
    }

fork İşleminde Dosya Betimleyici Tablosunun Durumu
--------------------------------------------------

``fork`` işlemi sırasında üst prosesin (``fork`` işlemini yapan prosesin) proses kontrol bloğunun yeni
yaratılan alt prosesin proses kontrol bloğuna kopyalandığını belirttik. Bu nedenle alt prosesin kullanıcı
ID'si, grup ID'si, çalışma dizini ve daha pek çok özellikleri üst prosesle aynı olacaktır. Peki alt
proseste dosya betimleyici tablosunun durumu ne olacaktır? Örneğin biz bir dosya açtıktan sonra ``fork``
yapmış olsak alt proseste bu dosyanın durumu ne olacaktır?

``fork`` işlemi sırasında işletim sistemi üst prosesin dosya betimleyici tablosu içerisindeki dosya
nesnelerinin adreslerini de alt prosesin dosya betimleyici tablosuna kopyalamaktadır. Ancak dosya
nesnelerinin kopyalarını çıkartmamaktadır. Böylece ``fork`` işleminin sonunda üst prosesin dosya
betimleyici tablosunun slotları ile alt prosesin dosya betimleyici tablosunun slotları aynı dosya
nesnesini gösteriyor durumda olur. Bu tür kopyalamalara *sığ kopyalama (shallow copy)* denildiğini
anımsayınız:

.. figure:: _static/fork-fd-tables.png
    :align: center
    :width: 75%

Mademki açık dosyaya ilişkin tüm bilgiler dosya nesnesinde tutulmaktadır, o halde örneğin ``fork`` işleminden
sonra örneğin proseslerden biri bir dosyanın dosya göstericisinin konumunu değiştirirse diğer proses de
bunu değişmiş olarak görecektir. Tabii ``fork`` işlemi sırasında dosya nesnelerinin referans sayaçları da
bir artırılmaktadır. Benzer biçimde aslında işin başında açık olan ``0``, ``1`` ve ``2`` numaralı betimleyiciler 
login işlemi öncesinde yaratılmış durumdadır. Her ``fork`` işleminde bu betimleyicilere ilişkin dosya
nesnelerinin kopyaları çıkartılmamaktadır. Prosesler aslında özel bir durum olmadıktan sonra hep aynı ``0``, ``1`` ve ``2``
numaralı dosya nesnelerini kullanmaktadır.

Aşağıdaki örnekte önce bir dosya açılmış sonra üst proses dosya göstericisini ``50``'nci offset'e
konumlandırmıştır. Üst prosesle alt proses aynı dosya nesnelerini gördüğü için bu durumdan alt proses
etkilenecektir. Alt proseste yapılan okuma ``50``'nci offset'ten itibaren yapılacaktır. Bu örnekte önce
``open`` fonksiyonuyla ``test.txt`` dosyası açılmıştır:

.. code-block:: c

    if ((fd = open("test.c", O_RDONLY)) == -1)
        exit_sys("open");

Sonra ``fork`` işlemi uygulanmıştır:

.. code-block:: c

    if ((pid = fork()) == -1)
        exit_sys("fork");

``fork`` işleminden sonra üst proseste dosya göstericisi konumlandırılmış ve alt proseste ``read``
fonksiyonu ile okuma yapılmıştır. Alt proses ``50``'nci offset'ten itibaren okumayı yapacaktır. Örneği
denerken ``test.txt`` dosyasının en az ``60`` karakter uzunluğunda bir text dosya olmasını sağlamalısınız.

Örneğimizde ``close`` işleminin hem üst proseste hem de alt proseste yapıldığına dikkat ediniz.
Dolayısıyla üst ve alt prosesler dosyayı kapattığında dosya nesnesinin referans sayacı azaltılacak, referans 
sayacı ``0``'a düştüğünde dosya nesnesi de silinecektir. 

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <fcntl.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        int fd;
        pid_t pid;
        char buf[10 + 1];
        ssize_t result;

        if ((fd = open("test.c", O_RDONLY)) == -1)
            exit_sys("open");

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid != 0) {
            lseek(fd, 50, SEEK_SET);
        }
        else {
            sleep(1);
            if ((result = read(fd, buf, 10)) == -1)
                exit_sys("read");
            buf[result] = '\0';
            puts(buf);
        }

        close(fd);

        sleep(1);

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

POSIX'in 2024 versiyonunda (henüz çok yeni) dosya açış bayraklarına ``O_CLOFORK`` isimli bir bayrak da eklenmiştir.
Bu bayrak kullanıldığında ``fork`` işlemi sırasında alt proseste betimleyici otomatik olarak kapatılmaktadır.
Örneğin:

.. code-block:: c

    if ((fd = open("test.c", O_RDONLY|O_CLOFORK)) == -1)
        exit_sys("open");

Bu bayrak ileride göreceğimiz gibi daha sonra ``fcntl`` fonksiyonu ile de set edilebilmektedir:

.. code-block:: c

    if (fcntl(fd, F_SETFD, fcntl(fd, F_GETFD)|FD_CLOFORK) == -1)
        exit_sys("fcntl");

Benzer biçimde bu bayrak şöyle de reset edilebilir:

.. code-block:: c

    if (fcntl(fd, F_SETFD, fcntl(fd, F_GETFD) & ~FD_CLOFORK) == -1)
        exit_sys("fcntl");

Bu bayrak Linux çekirdeği tarafından desteklenmemektedir.

fork İşleminde Dosya Tamponların Durumu
---------------------------------------

C'nin standart dosya fonksiyonlarının tamponlama mekanizmasıyla çalıştığını görmüştük. Bu durumda
``fopen`` fonksiyonu ile açtığımız bir dosyaya bir şeyler yazıp henüz tampon flush edilmeden ``fork``
yaparsak üst prosesin tüm bellek alanının kopyası çıkartılacağı için bu tamponun da flush edilmemiş bir
kopyası oluşacaktır.

Örneğin ``fopen`` fonksiyonuyla bir dosyayı ``"r+"`` modunda açıp içerisinden okuma yapmış olalım. Tampon
okuma işlemi ile doldurulacaktır. Bu işlemden sonra ``fork`` yapmış olalım. Artık bu tamponun içeriği hem
üst proseste hem alt proseste bulunacaktır. Üst prosesin dosyanın tampondaki kısmına yeni yazmalar yapıp
dosyayı kapattığını düşünelim. Alt proses de daha sonra hiçbir işlem yapmadan dosyayı kapatmış olsun
(``exit`` işlemiyle zaten ``stdio`` dosyaları otomatik kapatılmaktadır.) Şimdi alt prosesin tamponu flush
edileceğinden üst prosesin yazdıkları ezilecektir. Eğer böyle bir durum programınızda oluşuyorsa ``fork``
işleminden önce dosyayı flush edebilirsiniz. Bu yukarıdaki sorunu engelleyecektir. Örneğin:

.. code-block:: c

    f = fopen("test.txt", "r+b");
    ch = fgetc(f);
    fflush(f);

    pid = fork();
    if (pid != 0) {
        fprinf(f, "test\n");
        /* ... */
    }
    else {
        /* ... */
        exit(EXIT_SUCCESS);
    }

Bu temsili kodda kontrolleri yapmadık. Burada üst proses tampona yazmış olsa da alt prosesteki tampon
temiz durumda olduğu için alt proseste dosya kapatıldığında flush işlemi yapılmayacaktır. Dolayısıyla
yukarıdaki anomali oluşmayacaktır. C standartlarına göre yalnızca tampona yazılan kısım (*unwritten data*)
flush işlemi sırasında asıl hedefe aktarılmaktadır.

Aşağıdaki örnekte ``printf`` fonksiyonu Linux sistemlerinde default durumda *satır tamponlamalı* olan
``stdout`` dosyasının tamponuna bilgileri yazmıştır. Ancak ``''\n'`` karakteri tampona yazılmadığı için flush
işlemi de yapılmamıştır. ``fork`` işlemi ile birlikte bu tamponun da kopyası çıkarılacağından dolayı
ekranda iki tane *Ok* yazısı görünecektir:

.. code-block:: c

    pid_t pid;

    printf("Ok");

    if ((pid = fork()) == -1)
        exit_sys("fork");

    printf("\n");

Burada tampondaki *Ok* yazısı henüz flush edilmemiştir. ``fork`` işlemi sonrasında ``'\n'`` dolayısıyla flush
yapıldığında hem üst prosesin hem de alt prosesin ``stdio`` tamponu flush edilecektir. Dolayısıyla ekrana iki
kez *Ok* yazısı basılacaktır. Örneği bir bütün olarak aşağıda veriyoruz:

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        pid_t pid;

        printf("Ok");

        if ((pid = fork()) == -1)
            exit_sys("fork");

        printf("\n");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Proseslerin Sonlandırılması: _exit, exit ve abort Fonksiyonları
===============================================================

Şimdi de proseslerin nasıl sonlandırıldığını görelim. UNIX/Linux sistemlerinde prosesi sonlandırmak için
``_exit`` isimli POSIX fonksiyonu kullanılmaktadır. Bu fonksiyonun kullanımı C'nin standart ``exit`` fonksiyonuna
benzemektedir.

.. code-block:: c

    #include <unistd.h>

    void _exit(int status);

Fonksiyon, parametre olarak prosesin *çıkış kodunu (exit code)* almaktadır. Tabii bir proses sonlanmadan önce
prosesin sistem genelinde tahsis etmiş olduğu kaynaklar boşaltılmaktadır. Yani örneğin biz ``open``
fonksiyonu ile birtakım dosyalar açmışsak ``_exit`` işlemi sırasında bütün bu dosyalar kapatılacaktır, bu
dosyaların tuttuğu kaynaklar duruma göre serbest bırakılacaktır. Tabii ``_exit`` fonksiyonu yalnızca açık
dosyaları kapatmakla kalmaz, henüz görmediğimiz tahsis edilmiş olan başka kaynakları da boşaltmaktadır.
Linux sistemlerinde ``_exit`` fonksiyonu doğrudan işletim sisteminin ``sys_exit_group`` ya da ``sys_exit``
isimli sistem fonksiyonunu çağırmaktadır. Tabii asıl prosesin sonlandırılması bu sistem fonksiyonu
tarafından yapılmaktadır. Yine geleneksel olarak başarılı sonlanmalar için ``0`` değeri, başarısız sonlanmalar
için sıfır dışı değerler kullanılmaktadır. Peki bir program çökerek sonlandığında ne olmaktadır? Aslında
programın çökerek sonlanması kontrolsüz bir biçimde sonlanması anlamına gelmemektedir. UNIX/Linux
sistemlerinde çökme tabir edilen durumlar aslında bir sinyal oluşturmaktadır. Program çökse de işletim
sistemi tarafından onun tuttuğu kaynaklar yine boşaltılmaktadır. Yani çökme (crash olma) kullanıcının
algıladığı bir durumdur. İşletim sistemi çöken programlarda yine devreye girip gereken boşaltmaları
yapmaktadır.

C'nin standart ``exit`` fonksiyonunun da prototipi şöyledir:

.. code-block:: c

    #include <stdlib.h>

    void exit(int status);

C'nin ``exit`` fonksiyonu prosesin sonlandırılması için UNIX/Linux sistemlerinde aslında ``_exit`` POSIX
fonksiyonunu çağırmaktadır:

.. figure:: _static/exit-call-chain.png
    :align: center
    :width: 65%

``exit`` standart C fonksiyonu, standart C kütüphanesi için yapılan bazı işlemleri de geri almaktadır.
Örneğin ``exit`` fonksiyonu önce ``atexit`` fonksiyonu ile kaydettirilmiş olan fonksiyonları ters sırada
çağırır, sonra ``tmpfile`` fonksiyonu ile yaratılmış geçici dosyaları siler ve dosya bilgi göstericilerine
(streams) ilişkin tamponları flush eder, sonra da bunları kapatır. Eğer prosesinizi ``exit`` standart C
fonksiyonu yerine ``_exit`` POSIX fonksiyonu ile sonlandırırsanız proses sonlanırken bu işlemlerin
hiçbiri yapılmayacaktır.

C Programının Gerçek Başlangıç Noktası (start-up code)
------------------------------------------------------

C'de aslında ilk çalışan kod ``main`` fonksiyonunun kodu değildir. C derleyicileri ismine *başlangıç kodu
(start-up code)* denilen bir kodu da programa eklemektedir. Programın gerçek başlangıç noktası
derleyicinin yerleştirdiği bu başlangıç kodundan yapılmaktadır. Bu başlangıç kodu ``main`` fonksiyonunu
çağırmaktadır. C'de programcı, program içerisinde ``exit`` fonksiyonunu hiç çağırmamışsa akış ``main``
fonksiyonunu bitirdiğinde ``main`` fonksiyonunun geri dönüş değeri ile ``exit`` fonksiyonu çağrılmaktadır.
Yani C'de ``main`` fonksiyonu derleyici tarafından adeta ``exit(main())`` gibi çağrılmaktadır.
Derleyicinin ürettiği kodu şöyle temsil edebiliriz:

.. figure:: _static/startup-code.png
    :align: center
    :width: 60%

Yani C'de aslında tüm program sonlandırmaları her zaman ``exit`` (ya da ``abort``) fonksiyonu ile
yapılmaktadır. ``exit`` fonksiyonu yukarıda da belirttiğimiz gibi kütüphaneye ilişkin bazı son işlemleri
yaptıktan sonra UNIX/Linux sistemlerinde ``_exit`` POSIX fonksiyonunu çağırmaktadır. 

C standartlarına göre ``main`` fonksiyonunun sonunda ``return`` deyimi bulundurulmamışsa ``return 0`` yapılmış 
gibi işlem uygulanmaktadır. Örneğin:

.. code-block:: c

    int main(void)
    {
        /* ... */
    }

Burada ``main`` fonksiyonunun sonunda ``return`` uygulanmamıştır. Yani akış eğer buraya ulaşırsa ``main``
fonksiyonu ``0`` geri dönüş değeri ile sonlanacaktır. ile geri döndürüldüğü kabul edilmektedir. ``main`` dışındaki herhangi 
bir fonksiyonda ``return`` uygulanmazsa geri dönüş değeri çöp değer olarak elde edilmektedir.

Derleyicilerin başlangıç kodları genellikle açık olmayan derleyicilerde bile kaynak kod olarak
verilmektedir. Başlangıç kodları ayrı bir ya da birden fazla amaç dosya biçiminde derlenmiştir ve bağlama
aşamasında bu amaç dosyalar da bağlama işlemine sokulmaktadır. Örneğin:

.. code-block:: console

    $ gcc -o sample sample.c

Burada aslında ``gcc`` ``sample.c`` dosyasını derleyip ``sample.o`` dosyasını elde ettikten sonra ``ld``
bağlayıcısını yalnızca bu dosyayla değil bir grup başlangıç amaç dosyasıyla birlikte çağırmaktadır.

C'nin ``abort`` fonksiyonu ise *normal olmayan (abnormal)* sonlandırmalar için kullanılmaktadır.
UNIX/Linux sistemlerinde ``abort`` standart C fonksiyonu ``SIGABRT`` sinyali oluşturarak programı
sonlandırmaktadır. Programın sonlanması bu sinyal dolayısıyla gerçekleşmektedir. Sinyaller konusu ileride
ayrı bölümde ele alınacaktır.

Yukarıda da belirttiğimiz gibi C'de programlar, standart ``exit`` fonksiyonu ile sonlandırılmalıdır. Çünkü
``exit`` fonksiyonu bazı gerekli son işlemleri de yapmaktadır. Ancak yine de bazen programın doğrudan
``_exit`` POSIX fonksiyonu ile sonlandırılması da gerekebilmektedir. Bu gerekliliğe ilişkin örneklerle
sonraki konularımızda karşılaşacağız.

UNIX/Linux sistemlerinde prosesler ``_exit``, ``exit`` ve ``abort`` çağrılarının dışında sinyaller (signals) yoluyla da 
sonlanmaktadır. Örneğin abort aslında sonlandırmayı böyle yapmaktadır. Sinyaller konusu kitabımızda 
ayrı bir bölümde ele alınmaktadır. 

Aşağıdaki örnekte program ``exit`` fonksiyonu ile değil ``_exit`` fonksiyonu ile sonlandırılmıştır. Bu
nedenle ``atexit`` ile kaydedilen ``foo`` ve ``bar`` fonksiyonları program sonlanırken çağrılmayacaktır.
Aynı zamanda dosya tamponları da flush edilmeyeceğinden dolayı ``printf`` fonksiyonu ile ekrana yazılmak
istenen ancak satır tamponlaması nedeniyle henüz yazılamayan *ok* yazısı da ekranda görülmeyecektir.
Burada ``_exit`` çağrısını kaldırırsanız, akış ``main`` fonksiyonunu bitirince ``exit`` standart C
fonksiyonu çağrılacağı için bir sorun kalmayacaktır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    void foo(void)
    {
        fprintf(stderr, "foo\n");
    }

    void bar(void)
    {
        fprintf(stderr, "bar\n");
    }

    int main(void)
    {
        atexit(foo);
        atexit(bar);

        printf("ok");

        _exit(0);

        return 0;
    }

Linux'ta aslında ``_exit`` fonksiyonu işletim sisteminin ``sys_exit`` sistem fonksiyonunu değil
``sys_exit_group`` sistem fonksiyonunu çağırmaktadır. Linux'ta ``sys_exit`` sistem fonksiyonu yalnızca
fonksiyonu çağıran thread'i, ``sys_exit_group`` fonksiyonu ise tüm thread'leri, dolayısıyla da prosesi
sonlandırmaktadır.

wait Fonksiyonları
==================

``exit`` ve ``_exit`` fonksiyonunun parametresi olan prosesin çıkış kodu işletim sistemi tarafından kullanılmamaktadır.
ilgilendirmemektedir. Bunu prosesi yaratan üst proses isterse ona verir.  prosesin hangi çıkış koduyla sonlandığıyla 
işletim sistemi ilgilenmez. Çıkış kodunun değeri üst prosesle alt prosesin arasındaki bir anlaşma ile anlam kazanmaktadır.

Üst proses ``fork`` fonksiyonu ile alt prosesi yarattıktan sonra onun sonlanmasını bekleyebilir ve alt
proses sonlandığında onun çıkış kodunu alabilir. Bunun için ``wait`` ve ``waitpid`` isimli POSIX
fonksiyonları kullanılmaktadır. ``waitpid`` fonksiyonu ``wait`` fonksiyonunu işlevsel olarak
kapsamaktadır. (Zaten önce ``wait`` fonksiyonu vardı, onun yetersizlikleri görülünce ``waitpid``
fonksiyonu tasarlandı.)

wait Fonksiyonu
---------------

``wait`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <sys/wait.h>

    pid_t wait(int *status);

``wait`` fonksiyonu herhangi bir alt proses sonlanana kadar kendisini çağıran thread'i blokede 
bekletir. Burada blokede bekleme terimi CPU zamanı harcamadan uykuda kalmayı belirtmektedir. Tabii
``wait`` fonksiyonu çağrıldığında alt proseslerden biri zaten sonlanmış da olabilir. Bu durumda ``wait``
fonksiyonu blokeye (yani beklemeye) yol açmaz. ``wait`` fonksiyonu başarı durumunda çıkış kodunu aldığı
prosesin ID değeri ile geri döner. Böylece programcı çok sayıda alt prosesin söz konusu olduğu
durumda hangi alt prosesin çıkış kodunu aldığını geri dönüş değerinden hareketle anlayabilmektedir. Fonksiyon
parametresiyle aldığı ``int`` nesnesinin içerisine sonlanan prosesin çıkış kodunu ve sonlanma nedenine
ilişkin bazı bilgileri yerleştirmektedir.

Normal biçimde sonlanmamış (yani bir sinyal ile sonlanmış) proseslerde çıkış kodu oluşmamaktadır. O halde
programcının prosesin çıkış kodunu alabilmesi için onun normal bir biçimde sonlanmış olduğunu belirlemesi
gerekir. İşte ``<sys/wait.h>`` içerisindeki ``WIFEXITED`` makrosu ile bu belirleme yapılabilmektedir. Bu
makroya ``wait`` fonksiyonuna geçirilmiş olan ``int`` nesne verilir. Makro bu nesnenin bazı bitlerinden
alt prosesin normal sonlanıp sonlanmadığını anlar ve eğer alt proses normal bir biçimde sonlanmışsa sıfır
dışı herhangi bir değere, normal bir biçimde sonlanmamışsa sıfır değerine geri döner. Benzer biçimde biz
prosesin anormal bir biçimde bir sinyal dolayısıyla sonlanıp sonlanmadığını da ``WIFSIGNALED`` makrosuyla
tespit edebiliriz. Proses ``SIGSTOP`` sinyali ile geçici süre durdurulmuş da olabilir. Bu durum da
``WIFSTOPPED`` makrosu ile tespit edilebilmektedir. Prosesin çıkış kodu ise ``WEXITSTATUS`` makrosuyla
elde edilmektedir. Yine bu makroya ``wait`` fonksiyonuna adresi geçirilen ``int`` nesne argüman olarak
verilmektedir. Güncel POSIX standartlarında artık ``wait`` fonksiyonunda iletilen durum (status)
bilgisinin en düşük anlamlı 1 byte'ının çıkış kodunu içerdiği açıkça belirtilmektedir. (Her ne kadar
``exit`` standart C fonksiyonunun çıkış kodunu belirten parametresi ``int`` türden olsa da POSIX
standartlarında çıkış kodu için belirtilen değer ``[0, 255]`` arasında olmak zorundadır.) Programcı ``wait``
fonksiyonuna argüman olarak ``NULL`` adres de geçebilir. Bu durumda fonksiyon çıkış koduyla ilgili bir
yerleştirme yapmaz. Ancak yine ilk alt prosesin bitmesini bekler.

Eğer ``wait`` fonksiyonu çağrıldığında zaten üst prosesin yarattığı herhangi bir alt proses yoksa ya da 
fonksiyona geçersiz bir adres geçilmişse fonksiyon başarısız olacaktır. 

``wait`` fonksiyonunun tasarımında şu problemler vardır:

- ``wait`` fonksiyonu ile biz belli bir alt prosesi bekleyememekteyiz. ``wait`` fonksiyonu çağrıldığında 
    henüz hiçbir alt proses sonlanmamışsa ``wait`` ilk sonlanan alt prosesin çıkış kodunu alır.

- ``wait`` fonksiyonu çağrıldığında eğer zaten birden fazla alt proses sonlanmış durumdaysa POSIX 
    standartları hangi alt prosesin ``exit`` kodunun elde edileceği konusunda bir garanti vermemektedir. 
    Yani bu durumda ``wait`` fonksiyonunun ilk sonlanan alt prosesin çıkış kodunu alması garanti 
    edilmemiştir.

Aşağıdaki örnekte üst proses ``fork`` fonksiyonu ile alt prosesi yaratmıştır ve ``wait`` fonksiyonu 
ile onu beklemiştir. Alt proses normal bir biçimde sonlanmışsa onun çıkış kodunu alıp ekrana 
yazdırmıştır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>

    void exit_sys(const char *msg);

    void child_proc(void)
    {
        for (int i = 0; i < 10; ++i) {
            printf("child running: %d\n", i);
            sleep(1);
        }

        exit(100);
    }

    int main(void)
    {
        pid_t pid;
        int status;

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid == 0)
            child_proc();

        printf("parent waiting for the child to exit...\n");

        if (wait(&status) == -1)
            exit_sys("wait");

        if (WIFEXITED(status))
            printf("child exited with exit code %d\n", WEXITSTATUS(status));

        printf("Ok, parent continues running...\n");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }


Aşağıdaki örnekte ise üst proses ``wait`` fonksiyonu çağırmadan alt proses sonlanmıştır. Tabii bu durumda
üst proses hiç beklemeden alt prosesin çıkış kodunu alıp yoluna devam edecektir.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>

    void exit_sys(const char *msg);

    void child_proc(void)
    {
        printf("child terminates...\n");

        exit(100);
    }

    int main(void)
    {
        pid_t pid;
        int status;

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid == 0)
            child_proc();

        for (int i = 0; i < 10; ++i) {
            printf("parent running: %d\n", i);
            sleep(1);
        }

        if (wait(&status) == -1)
            exit_sys("wait");

        if (WIFEXITED(status))
            printf("child exited with exit code %d\n", WEXITSTATUS(status));

        printf("Ok, parent continues running...\n");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Tabii üst proses ne kadar ``fork`` yapmışsa o kadar sayıda ``wait`` yapmalıdır. Çünkü her ``wait``
fonksiyonu yalnızca tek bir alt prosesin sonlanma bilgilerini alacaktır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <stdint.h>
    #include <unistd.h>
    #include <sys/wait.h>

    #define NCHILDS     10

    void exit_sys(const char *msg);

    void child_proc(int ecode)
    {
        sleep(1);
        for (int i = 0; i < 10; ++i) {
            printf("child running: %d\n", i);
            sleep(1);
        }

        exit(ecode);
    }

    int main(void)
    {
        pid_t pid, pid_wait;
        int status;

        for (int i = 0; i < NCHILDS; ++i) {
            if ((pid = fork()) == -1)
                exit_sys("fork");
            if (pid == 0)
                child_proc(100 + i);
            printf("process created with pid %jd\n", (intmax_t)pid);
        }

        for (int i = 0; i < NCHILDS; ++i) {
            if ((pid_wait = wait(&status)) == -1)
                exit_sys("wait");
            if (WIFEXITED(status))
                printf("child pid %jd exited with exit code %d\n", (intmax_t)pid_wait, WEXITSTATUS(status));
        }

        printf("Ok, parent continues running...\n");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

waitpid Fonksiyonu
------------------

``waitpid`` fonksiyonu ``wait`` fonksiyonunun daha gelişmiş bir biçimidir. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <sys/wait.h>

    pid_t waitpid(pid_t pid, int *status, int options);

Fonksiyonun birinci parametresi beklenecek alt prosesin proses ID değerini belirtir. Bu sayede programcı
belli bir alt prosesi bekleyebilmektedir. Bu birinci parametre aslında birkaç biçimde geçilebilmektedir.
Eğer bu parametreye programcı pozitif değerde bir proses ID geçerse fonksiyon proses id'si geçirilen
değer olan o alt prosesi bekler. (Tabii bu parametreye geçirilen proses ID, o prosesin bir alt
prosesi değilse fonksiyon hemen başarısız olacaktır.) Eğer bu parametreye negatif bir proses ID değeri
geçilirse bu durumda fonksiyon proses grup ID'si bu değerin pozitifi olan herhangi bir alt prosesi
beklemektedir. Eğer bu parametre ``-1`` olarak geçilirse bu durumda fonksiyon tamamen ``wait``
fonksiyonundaki gibi davranmaktadır. Yani herhangi bir alt prosesi beklemektedir. Eğer bu parametreye ``0``
değeri geçilirse fonksiyon, proses grup ID'si kendisini çağıran prosesin ID'si ile aynı olan herhangi bir
alt prosesi beklemektedir.

Bu parametrenin alabileceği değerleri aşağıda bir tablo biçiminde de veriyoruz:

.. list-table::
   :header-rows: 1

   * - pid Değeri
     - Beklenen Alt Proses
   * - ``pid > 0``
     - Proses ID'si pid olan alt proses
   * - ``pid == 0``
     - Proses grup ID'si çağıranla aynı olan herhangi bir alt proses
   * - ``pid == -1``
     - Herhangi bir alt proses (wait ile eşdeğer)
   * - ``pid < -1``
     - Proses grup ID'si \|pid\| olan herhangi bir alt proses

Fonksiyonun ikinci parametresi sonlanma nedeninin ve çıkış kodunun yerleştirileceği ``int`` türden
nesnenin adresini almaktadır. Üçüncü parametre bazı özel değerlerin bit düzeyinde ``OR``'lanmasıyla
oluşturulabilmektedir:

- ``WNOHANG``: Bu durumda ``waitpid`` eğer alt proses henüz sonlanmamışsa bekleme yapmaz, fonksiyon ``0`` ile
  geri döner.
- ``WUNTRACED``, ``WCONTINUED``: Prosesin durdurulması ve devam ettirilmesi ile ilgili bilginin elde
  edilmesinde kullanılmaktadır.

Aşağıdaki tabloda son parametre için kullanılabilecek bayrakların listesini veriyoruz:

.. list-table::
   :header-rows: 1

   * - Seçenek
     - Anlamı
     - Standart
   * - ``0``
     - Bir alt proses sonlanana kadar bloke olunur
     - POSIX
   * - ``WNOHANG``
     - Durumu değişen alt proses yoksa bloke olmadan 0 ile dönülür
     - POSIX
   * - ``WUNTRACED``
     - Durdurulmuş alt prosesler için de bilgi elde edilir
     - POSIX
   * - ``WCONTINUED``
     - SIGCONT ile devam ettirilen alt prosesler için de kullanılır
     - POSIX
   * - ``__WCLONE``
     - Yalnızca sonlanınca SIGCHLD göndermeyen alt prosesler
     - Linux
   * - ``__WALL``
     - Türüne bakılmaksızın tüm alt prosesler beklenir
     - Linux
   * - ``__WNOTHREAD``
     - Aynı thread grubundaki diğer thread'lerin yarattığı alt prosesler beklenmez
     - Linux

Tabii bu üçüncü parametre genellikle ``0`` geçilmektedir. ``0`` geçilmesi bu bayraklardan hiçbirinin
kullanılmadığı anlamına gelmektedir. O halde aslında ``wait(&status)`` çağrısı ile
``waitpid(-1, &status, 0)`` çağrısı eşdeğerdir. Gördüğünüz gibi ``waitpid`` fonksiyonu tamamen ``wait``
fonksiyonunu işlevsel olarak kapsamaktadır.

``waitpid`` fonksiyonunda da ikinci parametre için ``NULL`` adres geçilebilir. Bu durumda proses beklenir
ama çıkış bilgileri elde edilmez.

``waitpid`` fonksiyonu da tıpkı ``wait`` fonksiyonunda olduğu gibi başarı durumunda beklenen proses ID
değeri ile, başarısızlık durumunda ``-1`` değeriyle geri dönmektedir.

Örneğin:

.. code-block:: c

    if ((pid = fork()) == -1)
        exit_sys("fork");

    /* ... */

    if (waitpid(pid, &status, 0) == -1)
        exit_sys("waitpid");

Burada ``waitpid`` fonksiyonu ile alt prosesin PID değeri belirtilerek bekleme yapılmıştır. Örneğin:

.. code-block:: c

    if ((child_pid = waitpid(pid, &status, WNOHANG)) == -1)
        exit_sys("waitpid");

    if (child_pid == 0)
        printf("no child exited...\n");

Burada ``WNOHANG`` bayrağının kullanıldığına dikkat ediniz. Bu bayrak, ``waitpid`` fonksiyonu ile beklenen
alt proses henüz sonlanmamışsa fonksiyonun ``0`` ile geri dönmesine yol açmaktadır.

Aşağıdaki örnekte ``5`` tane alt proses yaratılmış, ancak bunlar herhangi bir sırada değil yaratım sırasına
göre ``waitpid`` fonksiyonu ile beklenmiştir. Bu örneği ``waitpid`` yerine ``wait`` fonksiyonunu
kullanarak da çalıştırıp iki çıktı arasındaki farka bakabilirsiniz.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>
    #include <sys/wait.h>

    #define NCHILDS        5

    void exit_sys(const char *msg);

    void child_proc(int val)
    {
        srand(val);

        sleep(rand() % 5 + 1);

        exit(val);
    }

    int main(void)
    {
        pid_t pids[NCHILDS];
        int status;

        printf("parent is waiting for childs to exit...\n");

        for (int i = 0; i < NCHILDS; ++i) {
            if ((pids[i] = fork()) == -1)
                exit_sys("fork");
            if (pids[i] == 0)
                child_proc(100 + i);
        }

        for (int i = 0; i < NCHILDS; ++i) {
            if (waitpid(pids[i], &status, 0) == -1)
                exit_sys("wait");
            if (WIFEXITED(status))
                printf("child exited with exit code %d\n", WEXITSTATUS(status));
        }

        printf("Ok, parent continues running...\n");

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Programcının ``fork`` fonksiyonu ile her yarattığı alt prosesi ``wait`` fonksiyonları ile beklemesi iyi
bir tekniktir. Aksi halde sonraki paragrafta ele alacağımız gibi *zombie* proses problemi
oluşabilmektedir.

wait3 ve wair4 Fonksiyonları
----------------------------

POSIX standartlarında olmayan ancak *glibc* kütüphanesi tarafından desteklenen ``wait3`` ve ``wait4`` isimli iki
fonksiyon daha vardır. Bu fonksiyonlar Linux sistemlerinde ``sys_wait4`` isimli sistem fonksiyonu çağrılarak
gerçekleştirilmiştir:

.. code-block:: c

    #include <sys/wait.h>

    pid_t wait3(int *status, int options, struct rusage *rusage);
    pid_t wait4(pid_t pid, int *status, int options, struct rusage *rusage);

``wait3`` ve ``wait4`` fonksiyonlarının ``waitpid`` fonksiyonundan tek farkı ``rusage`` isimli bir yapı türünden
nesnenin adresini almasıdır. Bu fonksiyonlar ek olarak prosesin kullanmış olduğu kaynaklara ilişkin bilgileri de
vermektedir. Aslında çağrı sırasında bu ``rusage`` parametreleri ``NULL`` da geçilebilir. Bu durumda:

.. code-block:: c

    wait3(status, options, rusage);

çağrısı ile:

.. code-block:: c

    waitpid(-1, status, options);

çağrısı eşdeğerdir. Benzer biçimde:

.. code-block:: c

    wait4(pid, status, options, rusage);

çağrısı ile de:

.. code-block:: c

    waitpid(pid, status, options);

çağrısı eşdeğerdir. ``wait3`` fonksiyonunun herhangi bir alt prosesi, ``wait4`` fonksiyonunun ise belli bir alt
prosesi beklediğine dikkat ediniz. Fonksiyonlara geçirilen ``rusage`` yapısı ``<sys/resource.h>`` dosyasında
tanımlanmıştır:

.. code-block:: c

    struct rusage {
        struct timeval ru_utime; /* user CPU time used */
        struct timeval ru_stime; /* system CPU time used */
    };

Yapının ``ru_utime`` elemanı prosesin kullanıcı modunda geçirdiği zamanı, ``ru_stime`` elemanı ise çekirdek modunda
geçirdiği zamanı vermektedir.

Linux gerçekleştiriminde bu yapı çok daha fazla elemana sahiptir:

.. code-block:: c

    struct rusage {
        struct timeval ru_utime; /* user CPU time used */
        struct timeval ru_stime; /* system CPU time used */
        long   ru_maxrss;        /* maximum resident set size */
        long   ru_ixrss;         /* integral shared memory size */
        long   ru_idrss;         /* integral unshared data size */
        long   ru_isrss;         /* integral unshared stack size */
        long   ru_minflt;        /* page reclaims (soft page faults) */
        long   ru_majflt;        /* page faults (hard page faults) */
        long   ru_nswap;         /* swaps */
        long   ru_inblock;       /* block input operations */
        long   ru_oublock;       /* block output operations */
        long   ru_msgsnd;        /* IPC messages sent */
        long   ru_msgrcv;        /* IPC messages received */
        long   ru_nsignals;      /* signals received */
        long   ru_nvcsw;         /* voluntary context switches */
        long   ru_nivcsw;        /* involuntary context switches */
    };

Biz proseslerin kullandığı kaynak bilgilerinin elde edilmesini ayrı bir bölümde ele alacağız.

Kabuk Üzerinden Son Çalıştırılan Programın Çıkış Kodunun Elde Edilmesi 
----------------------------------------------------------------------

Biz kabuk üzerinden program çalıştırdığımızda ``fork`` işlemini kabuk uygulamaktadır. Dolayısıyla
çalıştırılan programın çıkış kodunu da üst proses olan kabuk almaktadır. İşte biz ``$?`` ile kabuk
üzerinde son çalıştırılan programın çıkış kodunu elde edebiliriz. Örneğin:

.. code-block:: console

    $ ./sample
    $ echo $?
    100

Zombie Proses Kavramı
======================

UNIX/Linux sistemlerinde prosesler konusunda çokça karşılaşılan *zombie proses* denilen bir
kavram vardır. (İngilizce "zombie" sözcüğü "hortlak" gelmektedir. Hortlak sözcük anlamı olarak "tam ölememiş 
ya da öldükten sonra dirilmiş olan canlılar" için kullanılmaktadır. Tabii gerçek dünyada böyle bir durumla 
karşılaşılmamıştır. Ancak öldü sanılarak defnedilmeye çalışılırken tabuttan çıkan kişiler bulunmaktadır.) 

Bir alt proses sonlandığında işletim sistemi onun kaynaklarını boşaltmaktadır. Örneğin prosesin bellek alanı tamamen 
sisteme iade edilmektedir. Prosesin açmış olduğu dosyalar kapatılmaktadır. Ancak işletim sistemi, alt prosesin çıkış 
kodunu üst prosese iletebilmek için proses kontrol bloğunu proses bittiğinde hemen serbest bırakmamaktadır. (Prosesin 
çıkış kodu proses kontrol bloğunda saklanmaktadır.) İşletim sistemi "üst proses bu çıkış kodunu herhangi bir zaman 
isteyebilir" diye sonlanmış olan prosesin proses kontrol bloğunu (Linux'taki ``task_struct`` yapısı) sisteme iade etmez. 
Böylece bir alt proses bittiğinde eğer üst proses ``wait`` fonksiyonlarıyla alt prosesin çıkış kodunu henüz almamışsa 
"kendisi bitmiş ama proses kontrol bloğu sisteme iade edilmemiş proses" durumu oluşmaktadır. İşte bu duruma UNIX/Linux 
dünyasında *zombie proses* denilmektedir. Zombie proseslerde prosesin ID değeri de "üst proses ``wait`` ya da ``waitpid`` 
fonksiyonunu kullanabilir" diye sisteme iade edilmemektedir. Yani zombie proses oluşması, bitmiş olan prosesin proses 
kontrol bloğunun yer kaplamasına hem de bitmiş olan prosesin ID değerinin yeniden kullanılamamasına yol açmaktadır.

Alt ve üst proseslerin sonlanması şu biçimlerde olabilmektedir:

| **1.** Üst proses alt prosesten önce sonlanmış olabilir. Bu durumda alt proses *öksüz (orphan)* duruma düşer.
   Sistem de ``1`` numaralı ID'ye sahip olan *init* prosesini öksüz prosesin üst prosesi olarak atar. Daha
   sonra alt proses sonlandığında *init* prosesi alt prosesin çıkış kodunu alarak onun zombie duruma
   düşmesini engeller.

| **2.** Alt proses üst prosesten daha önce sonlanmıştır. İşte bu durumda eğer üst proses ``wait``
   fonksiyonlarını henüz uygulamamışsa alt proses zombie durumda kalır. Tabii üst proses ``wait``
   fonksiyonlarını uyguladığı anda alt proses zombie olmaktan  kurtulur.

| **3.** Alt proses üst prosesten önce sonlanmıştır. Ancak üst proses de ``wait`` fonksiyonlarını uygulamadan
   sonlanmıştır. Bu durumda yine işletim sistemi artık çıkış kodunu alacak bir üst proses kalmadığı için
   alt prosesi zombie olmaktan çıkartır. Yani onun proses kontrol bloğunu ve ID değerini serbest bırakır.

O halde zombie proses yalnızca şu süreçte ortaya çıkmaktadır: "Alt proses sonlanmıştır ancak üst proses
wait fonksiyonlarını uygulamadan çalışmasına devam etmektedir."

Şimdi bir zombie proses durumu oluşturalım. Yapacağımız şey alt prosesi sonlandırıp üst prosesin
``wait`` fonksiyonlarını uygulamadan yoluna devam etmesini sağlamaktır. Zombie prosesler *"ps -l"*
komutunda *defunct* olarak gösterilmektedir. Bunların *proses durumları da (process state)* *Z* harfi ile
belirtilmektedir. Örneğin:

.. code-block:: console

    $ ps -la
    F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
    0 S  1000   10612    1868  0  80   0 -   622 hrtime pts/1    00:00:00 sample
    1 Z  1000   10613   10612  0  80   0 -     0 -      pts/1    00:00:00 sample <defunct>
    4 R  1000   10621    1618  0  80   0 -  3540 -      pts/0    00:00:00 ps

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    void exit_sys(const char *msg);

    int main(void)
    {
        pid_t pid;

        if ((pid = fork()) == -1)
            exit_sys("fork");

        if (pid != 0) {            /* üst proses */
            for (int i = 0; i < 60; ++i) {
                printf("parent process continues running: %d\n", i);
                sleep(1);
            }
        }
        else {                    /* alt proses */
            printf("child terminates...\n");

            exit(EXIT_SUCCESS);
        }

        return 0;
    }

    void exit_sys(const char *msg)
    {
        perror(msg);
        exit(EXIT_FAILURE);
    }

Zombie Proseslerin Yarattığı Sorunlar
-------------------------------------

Yukarıda da belirttiğimiz gibi zombie proses oluşmasının yarattığı sorunlar şunlardır:

1. Üst prosesin ömrü fazla değilse genellikle üst prosesin zombie proses oluşturması ciddi bir soruna
   yol açmaz. Ancak üst proses uzun süre çalışıyorsa (günlerce, aylarca) zombie prosesler sistem
   kaynağının boşa harcanmasına yol açabilmektedir.

2. Zombie proseslere ilişkin proses ID değerleri o proses zombie'likten kurtulana kadar sistem tarafından
   kullanılamamaktadır. Sürekli zombie proses üreten bir program proses ID'lerinin tükenmesine bile yol
   açabilmektedir.

3. Daha önce bir kullanıcının yaratabileceği maksimum proses sayısının sınırlandırıldığını belirtmiştik.
   İşletim sistemi bunun için proses yaratıldıkça bir sayacı artırır, proses sonlandığında bu sayacı
   eksiltir. İşte zombie prosesler henüz tam sonlanmadığı için bu sayacı eksiltememektedir.

Zombie Proses Oluşumunun Otomatik Engellenmesi
----------------------------------------------

Peki zombie proses oluşmasının engellenmesinin tek yolu ``wait`` fonksiyonlarını uygulamak mıdır? Çünkü
``wait`` fonksiyonları uygulandığında üst proses alt proses bitene kadar blokede bekleyecektir. Halbuki
bazı uygulamalarda üst prosesin yoluna devam etmesi ve bloke olmaması istenir. İşte zombie oluşmasının
otomatik engellenmesi için iki yöntem kullanılmaktadır:

1. Alt proses bittiğinde ``SIGCHLD`` sinyaline üst proses ``wait`` fonksiyonlarını uygularsa üst proses
   blokede kalmadan zombie durumunu engelleyebilir.

2. Biz alt prosesin çıkış kodunu almak istemediğimizi işletim sistemine söylersek işletim sistemi alt
   proses bittiğinde onu zombie duruma sokmadan onun kaynaklarını boşaltabilmektedir.

Bu iki zombie engelleme yöntemi de *sinyaller (signals)* konusuyla ilgilidir. Bu konu ileride ele
alınacaktır.


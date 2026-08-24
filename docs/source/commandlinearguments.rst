============================================
**Komut Satırı Argümanlarının Ele Alınması**
============================================

Bir programı çalıştırırken program isminin yanına yazdığımız argümanlara *komut satırı argümanları (command line
arguments)* denilmektedir. Örneğin:

.. code-block:: text

    $ ls -l -i /usr/include

Burada *ls* program dosyasını, ``-l``, ``-i`` ve ``/usr/include`` ise komut satırı argümanlarını belirtmektedir. Bu bölümde
UNIX/Linux sistemlerinde komut satırı argümanlarının ele alınması üzerinde duracağız.

POSIX Stili Satırı Argümanları
==============================

UNIX/Linux dünyasında komut satırı argümanlarının oluşturulması için geniş bir kesim tarafından kullanılan geleneksel
bir biçim vardır. Bu biçime *POSIX yada UNIX biçimi* de denilmektedir. Biz de kursumuzda UNIX/Linux dünyasında yazacağımız
programlarda bu geleneği kullanacağız. POSIX stilinde komut satırı argümanları üçe ayrılmaktadır:

| **1.** Argümansız seçenekler
| **2.** Argümanlı seçenekler
| **3.** Seçeneksiz argümanlar

Argümansız seçenekler ``-`` karakterine yapışık tek bir harften oluşmaktadır. Harflerde büyük harf-küçük harf
duyarlılığı (case sensitivity) dikkate alınmaktadır. Örneğin:

.. code-block:: bash

    $ ls -l -i /usr/include

Burada ``-l`` ve ``-i`` argümansız seçeneklerdir. ``/usr/include`` argümanının bu seçeneklerle hiçbir ilgisi yoktur.
Argümansız seçenekler tek bir karakterden oluşturulduğu için birleştirilebilmektedir. Örneğin:

.. code-block:: bash

    $ ls -li

Buradaki ``-li`` aslında ``-l -i`` ile tamamen aynı anlamdadır. Genel olarak POSIX stilinde seçenekler arasındaki sıranın
bir önemi yoktur. Yani örneğin ``ls -l -i`` ile ``ls -i -l`` arasında bir farklılık yoktur. Seçenekler istenilen sırada
belirtilebilmektedir.

Argümanlı seçeneklerde bir seçeneğin yanında o seçenekle ilişkili bir argüman da bulunur. Örneğin:

.. code-block:: bash

    $ gcc -o sample sample.c

Burada ``-o`` seçeneği tek başına kullanılmaz. Hedef dosyanın ismi seçeneğin argümanını oluşturmaktadır. O halde
buradaki ``-o`` seçeneği tipik olarak argümanlı seçeneğe bir örnektir. Argümanlı seçeneklerin birleştirilmesini 
tavsiye etmiyoruz. Ancak birleştirme yapılabilmektedir. Örneğin:

.. code-block:: bash

    $ gcc -co sample.o sample.c

Buradaki argümanların aşağıdaki gibi belirtilmesini tavsiye ediyoruz:

.. code-block:: bash

    $ gcc -c -o sample.o sample.c

Argümanlı seçenekleri argümansız seçeneklerle birleştirecekseniz argümanlı seçeneği sonda bulundurmanız gerekir. Örneğin:

.. code-block:: bash

    $ tar -xvzf test.tar.gz

Burada ``test.tar.gz`` argümanı aslında ``-f`` seçeneğinin argümanıdır. Biz bu ``-f`` seçeneğini sonda bulundurmak
zorundayız. Aşağıdaki kullanım hatalıdır:

.. code-block:: bash

    $ tar -xvfz test.tar.gz

Burada ``-f`` seçeneğindeki argüman sorunu nedeniyle program hata mesajı vererek sonlanacaktır.

Argümanlı seçeneklerde seçeneğin argümanı hiç boşluk karakterleriyle ayrılmasa bile bu durum geçerlidir. Örneğin:

.. code-block:: bash

    $ gcc -osample sample.c

Burada ``-o`` argümanlı seçenek olduğu için onu başka bir seçenek izleyemeyeceğinden dolayı ``sample``, ``-o``
seçeneğinin argümanı olarak ele alınmaktadır. Tabii seçenekli argümanlarda argümandan sonra bir boşluk bırakmak komutun
daha iyi algılanmasını sağlamaktadır.

Seçeneklerle ilgisi olmayan argümanlara *seçeneksiz argümalarn* denilmektedir. Örneğin:

.. code-block:: bash

    $ gcc -o sample sample.c

Burada ``sample.c`` argümanı herhangi bir seçenekle ilgili değildir. Örneğin:

.. code-block:: bash

    $ cp x.txt y.txt

Buradaki ``x.txt`` ve ``y.txt`` argümanları da seçeneklerle ilgili değildir. Seçeneksiz argümanların sonda bulunması
gerekmez. Örneğin:

.. code-block:: bash

    $ gcc sample.c -o sample

Eskiden yalnızca tek karakterden oluşan kısa seçenekler kullanılıyordu. Ancak daha sonraları bu kısa seçeneklerin
yetersiz kaldığı ve okunabilirliği bozduğu gerekçesiyle uzun seçenekler de kullanılmaya başlanmıştır. POSIX standartları
uzun seçenekleri desteklememektedir. Ancak UNIX/Linux dünyasında yaygın biçimde kullanılmaktadır.

Uzun seçenekler ``--`` öneki ile başlatılır. Örneğin:

.. code-block:: bash

    prog --count -a -b --length 100

Uzun seçenekler de argümanlı ve argümansız olabilmektedir. Yukarıdaki örnekte ``--count`` argümansız uzun seçenek,
``-a`` ve ``-b`` argümansız seçenekler ve ``--length 100`` ise argümanlı uzun seçenektir.

Uzun seçeneklerde *isteğe bağlı argüman (optional argument)* denilen özel bir argüman da kullanılmaktadır. İsteğe bağlı
argümanlar uzun seçeneklerin yanında verilip verilmemesi isteğe bağlı olan argümanlardır. Uzun seçeneklerin isteğe bağlı
argümanları ``=`` sentaksı ile yapışık bir biçimde belirtilmektedir. Örneğin:

.. code-block:: bash

    prog --size=512

Burada ``--size`` uzun seçeneğinin argümanı isteğe bağlıdır. Yani bu uzun seçenek argümansız da aşağıdaki gibi
kullanılabilirdi:

.. code-block:: bash

    prog --size

Günümüzde genel olarak programlar kısa seçenekleri de uzun seçenekleri de bir arada kullanmaktadır. Programcılar bazı
kısa seçeneklerin alternatif uzun seçeneklerini oluşturabilmektedir. Örneğin Linux'un ``ls`` komutunun seçenek listesi
şöyledir:

.. list-table:: ``ls`` Komutunun Seçenekleri
   :header-rows: 1
   :widths: 38 62

   * - Seçenek
     - Açıklama
   * - ``-a``, ``--all``
     - Nokta ile başlayanlar dahil tüm dosyaları listeler
   * - ``-A``, ``--almost-all``
     - ``.`` ve ``..`` hariç gizli dosyaları listeler
   * - ``-b``, ``--escape``
     - Yazdırılamaz karakterleri sekizlik gösterimle kaçırır
   * - ``--block-size=BOYUT``
     - Boyutları BOYUT katlarında gösterir (örn: ``--block-size=M``)
   * - ``-B``, ``--ignore-backups``
     - ``~`` ile biten yedek dosyaları listelemeye dahil etmez
   * - ``-c``
     - ``-lt`` ile durum değişikliği zamanını gösterir, ``-l`` ile sıralar
   * - ``-C``
     - Çıktıyı sütunlar halinde listeler (varsayılan)
   * - ``--color[=NE ZAMAN]``
     - Renklendirmeyi etkinleştirir (always/auto/never)
   * - ``-d``, ``--directory``
     - Dizin içeriği yerine dizinin kendisini listeler
   * - ``-D``, ``--dired``
     - Emacs dired modu için çıktı üretir
   * - ``-f``
     - Sıralama yapmaz; ``-aU`` etkinleştirir, ``-ls`` devre dışı bırakır
   * - ``-F``, ``--classify[=NE ZAMAN]``
     - Dosya türünü belirten sembol ekler (``/``, ``*``, ``@``, ``|``, ``=``)
   * - ``--file-type``
     - ``-F`` gibi ancak ``*`` eklemez
   * - ``--format=SÖZCÜK``
     - Listeleme biçimini belirler (across/commas/long/vb.)
   * - ``--full-time``
     - Tam tarih ve saat bilgisini gösterir
   * - ``-g``
     - Kullanıcı adı sütunu olmaksızın uzun listeleme yapar
   * - ``--group-directories-first``
     - Dizinleri dosyalardan önce listeler
   * - ``-G``, ``--no-group``
     - Uzun listede grup adını göstermez
   * - ``-h``, ``--human-readable``
     - Boyutları okunabilir birimlerle gösterir (K, M, G)
   * - ``--si``
     - Boyutları SI (1000'lik) birimleriyle gösterir
   * - ``-H``, ``--dereference-command-line``
     - Sembolik bağlantıları komut satırında takip eder
   * - ``--dereference-command-line-symlink-to-dir``
     - Yalnızca dizine işaret eden sembolik bağlantıları takip eder
   * - ``--hide=KALIP``
     - Belirtilen glob kalıbıyla eşleşen girişleri gizler
   * - ``--hyperlink[=NE ZAMAN]``
     - Dosya adlarına terminal hiper bağlantısı ekler
   * - ``--indicator-style=SÖZCÜK``
     - Tür gösterge stilini belirler (none/slash/file-type/classify)
   * - ``-i``, ``--inode``
     - Her dosyanın inode numarasını gösterir
   * - ``-I``, ``--ignore=KALIP``
     - Belirtilen glob kalıbıyla eşleşen girişleri gizler
   * - ``-k``, ``--kibibytes``
     - Blok boyutlarını 1 KiB olarak gösterir
   * - ``-l``
     - Uzun biçimde listeler (izin, sahip, boyut, tarih)
   * - ``-L``, ``--dereference``
     - Sembolik bağlantılar için bağlantıyı takip eder
   * - ``-m``
     - Girişleri virgülle ayrılmış liste olarak gösterir
   * - ``-n``, ``--numeric-uid-gid``
     - Kullanıcı ve grup adı yerine UID/GID numarası gösterir
   * - ``-N``, ``--literal``
     - Dosya adlarını tırnak içine almaz
   * - ``-o``
     - Grup sütunu olmaksızın uzun listeleme yapar
   * - ``-p``, ``--indicator-style=slash``
     - Dizinlerin sonuna ``/`` ekler
   * - ``-q``, ``--hide-control-chars``
     - Yazdırılamaz karakterleri ``?`` ile değiştirir
   * - ``--show-control-chars``
     - Yazdırılamaz karakterleri olduğu gibi gösterir
   * - ``-Q``, ``--quote-name``
     - Dosya adlarını çift tırnak içinde gösterir
   * - ``--quoting-style=SÖZCÜK``
     - Tırnak stilini belirler (literal/shell/c/escape/vb.)
   * - ``-r``, ``--reverse``
     - Sıralama düzenini tersine çevirir
   * - ``-R``, ``--recursive``
     - Dizin ağacını özyinelemeli olarak listeler
   * - ``-s``, ``--size``
     - Her dosyanın blok cinsinden disk kullanımını gösterir
   * - ``-S``
     - Dosya boyutuna göre sıralar (büyükten küçüğe)
   * - ``--sort=SÖZCÜK``
     - Sıralama ölçütünü belirler (none/size/time/version/ext)
   * - ``--time=SÖZCÜK``
     - Zaman ölçütünü belirler (atime/ctime/mtime/birth)
   * - ``--time-style=BİÇİM``
     - Tarih/saat gösterim biçimini belirler
   * - ``-t``
     - Değişiklik zamanına göre sıralar (yeniden eskiye)
   * - ``-T``, ``--tabsize=SÜTUN``
     - Sekme durakları için sütun genişliğini ayarlar
   * - ``-u``
     - ``-lt`` ile erişim zamanını gösterir, ``-l`` ile erişime göre sıralar
   * - ``-U``
     - Dizin sırasıyla listeler, sıralama yapmaz
   * - ``-v``
     - Dosya adlarındaki sayıları doğal sırada sıralar
   * - ``-w``, ``--width=SÜTUN``
     - Çıktı genişliğini n karakter olarak belirler
   * - ``-x``
     - Sütunlar yerine satırlar halinde sıralar
   * - ``-X``
     - Uzantıya göre alfabetik sıralar
   * - ``-Z``, ``--context``
     - SELinux güvenlik bağlamını gösterir
   * - ``-1``
     - Her dosyayı ayrı satırda listeler
   * - ``--help``
     - Yardım bilgisini gösterip çıkar
   * - ``--version``
     - Sürüm bilgisini gösterip çıkar

Burada da gördüğünüz gibi bazı kısa seçeneklerin alternatif uzun seçenekleri de bulunmaktadır. Ancak yalnızca uzun
seçenekler de vardır. Yukarıda da belirttiğimiz gibi POSIX standartları uzun seçenekleri desteklememektedir.

----

``getopt`` ve ``getopt_long`` Fonksiyonları
--------------------------------------------

Peki biz programımızda GNU stilinde seçenek kullanmak istersek komut satırı argümanlarını nasıl parse edebiliriz? İşte
UNIX/Linux dünyasında komut satırı argümanlarını parse etmek için ``getopt`` ve ``getopt_long`` isimli iki fonksiyon
bulundurulmuştur. ``getopt`` bir POSIX fonksiyonudur. Ancak bu fonksiyon uzun seçenekleri parse etmemektedir.
``getopt_long`` ise uzun seçenekleri de parse eden ``getopt`` fonksiyonunun daha gelişmiş bir biçimidir. Ancak
``getopt_long`` bir POSIX fonksiyonu değildir; ``libc`` kütüphanesinde bulunmaktadır. ``getopt`` ve ``getopt_long``
fonksiyonları Windows sistemlerinde hazır bir biçimde herhangi bir kütüphanede bulunmamaktadır. Zaten yukarıda da
belirttiğimiz gibi Windows sistemlerindeki komut satırı argüman stili UNIX/Linux sistemlerindekinden farklıdır.

``getopt`` Fonksiyonunun Kullanımı
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``getopt`` fonksiyonunun prototipi şöyledir:

.. code-block:: c

    #include <unistd.h>

    int getopt(int argc, char * const argv[], const char *optstring);

``getopt`` fonksiyonunun ilk iki parametresi ``main`` fonksiyonunun ``argc`` ve ``argv`` parametreleri gibidir. Yani
programcı ``main`` fonksiyonunun bu parametrelerini ``getopt`` fonksiyonuna geçirir. Fonksiyonun üçüncü parametresinde
kısa seçenekler belirtilmektedir. Bu parametre bir yazı biçiminde girilir. Bu yazıdaki her bir karakter bir kısa seçeneği
belirtmektedir. Bir karakterin sağında ``:`` karakteri varsa bu ``:`` karakterinin solundaki seçeneğin argümanlı bir
seçenek olduğunu belirtmektedir. Örneğin ``"ab:c"`` yazısında ``-a``, ``-b`` ve ``-c`` seçenekleri belirtilmiştir.
Ancak ``-b`` seçeneğinin argümanı da vardır.

``getopt`` fonksiyonu bir kez çağrılmaz. Bir döngü içerisinde çağrılmalıdır. Çünkü fonksiyon her çağrıldığında bir
kısa seçeneği bulmaktadır. Fonksiyon bütün kısa seçenekleri bulduktan sonra artık bulacak bir seçenek kalmadığında
``-1`` değerine geri dönmektedir. Fonksiyonun tipik çağrılma kalıbı şöyledir:

.. code-block:: c

    int result;

    while ((result = getopt(argc, argv, "ab:c")) != -1) {
        /* ... */
    }

``getopt``, her kısa seçeneği bulduğunda o kısa seçeneğe ilişkin karakterle (yani o karakterin sayısal karşılığı ile)
geri dönmektedir. O halde bizim ``getopt`` fonksiyonunun geri dönüş değerini ``switch`` içerisinde ele almamız gerekir:

.. code-block:: c

    while ((result = getopt(argc, argv, "ab:c")) != -1) {
        switch (result) {
            case 'a':
                /* ... */
                break;
            case 'b':
                /* ... */
                break;
            case 'c':
                /* ... */
                break;
        }
    }

``getopt`` fonksiyonu, olmayan (yani üçüncü parametresinde belirtilmeyen) bir kısa seçenekle karşılaştığında ya da
argümanı olması gerektiği halde girilmemiş bir kısa seçenekle karşılaştığında ``'?'`` özel değerine geri dönmektedir.
Programcının ``switch`` deyimine bu ``case`` bölümünü ekleyerek bu durumu da ele alması uygun olur. Örneğin:

.. code-block:: c

    while ((result = getopt(argc, argv, "ab:c")) != -1) {
        switch (result) {
            case 'a':
                /* ... */
                break;
            case 'b':
                /* ... */
                break;
            case 'c':
                /* ... */
                break;
            case '?':
                /* ... */
                break;
        }
    }

``getopt`` Fonksiyonunun Global Değişkenleri
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``getopt`` fonksiyonunun kullandığı dört global değişken vardır. Bu global değişkenler kütüphanenin içerisinde
tanımlanmıştır. Bunları biz ``extern`` bildirimi ile kullanabiliriz. Ancak bunların ``extern`` bildirimleri zaten
``<unistd.h>`` dosyası içerisinde yapılmış durumdadır:

.. code-block:: c

    extern int opterr;
    extern int optopt;
    extern int optind;
    extern char *optarg;

Varsayılan durumda, ``getopt`` fonksiyonu geçersiz bir seçenekle (yani üçüncü parametresinde belirtilmeyen bir
seçenekle) karşılaştığında ya da seçenek argümana sahip olduğu halde argümanın belirtilmemesi durumunda ``stderr``
dosyasına (ekranda çıkacaktır) kendisi hata mesajını yazdırmaktadır. Programcılar genellikle bunu istemezler.
``getopt`` fonksiyonunun geçersiz seçenekler için hata mesajını yazdırması ``opterr`` değişkenine ``0`` değeri
atanarak engellenebilmektedir. Yani ``opterr`` değişkeni sıfır dışı bir değerdeyse (varsayılan durum) fonksiyon mesajı
``stderr`` dosyasına kendisi de yazar; sıfır değerindeyse fonksiyon hata mesajını ``stderr`` dosyasına yazmaz.

``getopt`` fonksiyonu geçersiz bir seçenekle ya da argümanı girilmemiş argümanlı bir seçenekle karşılaştığında ``'?'``
karakteri ile geri dönmekle birlikte aynı zamanda ``optopt`` global değişkenine geçersiz seçeneğin karakter karşılığını
da yerleştirmektedir. Böylece programcı daha yeterli bir mesaj verebilmektedir. Örneğin:

.. code-block:: c

    opterr = 0;
    while ((result = getopt(argc, argv, "ab:c")) != -1) {
        switch (result) {
            case 'a':
                printf("-a given...\n");
                break;
            case 'b':
                printf("-b given...\n");
                break;
            case 'c':
                printf("-c given...\n");
                break;
            case '?':
                if (optopt == 'b')
                    fprintf(stderr, "-b option given without argument!...\n");
                else
                    fprintf(stderr, "invalid option: -%c\n", optopt);
                break;
        }
    }

Argümanlı bir kısa seçenek bulunduğunda ``getopt`` fonksiyonu, ``optarg`` global değişkenini o kısa seçeneğin
argümanını gösterecek biçimde set eder. Ancak ``optarg`` her argümanlı seçenekte yeni bulunan argümanlı seçeneğin
argümanını gösterecek biçimde ayarlanmaktadır. Dolayısıyla programcı argümanlı kısa seçeneği bulduğu anda ``optarg``
değişkenine başvurmalı, gerekirse onu başka bir göstericede saklamalıdır.

Seçeneksiz Argümanların Elde Edilmesi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Peki seçeneksiz argümanları nasıl elde edebiliriz? Seçeneksiz argümanlar ``argv`` dizisinin herhangi bir yerine
bulunuyor olabilir. İşte ``getopt`` fonksiyonu her zaman seçeneksiz argümanları girildiği sırada ``argv`` dizisinin
sonuna taşır ve onların başladığı indeksi de ``optind`` global değişkeninin göstermesini sağlar. O halde programcı
``getopt`` ile işini bitirdikten sonra (yani ``while`` döngüsünden çıktıktan sonra) ``optind`` indeksinden ``argc``
indeksine kadar ilerleyerek tüm seçeneksiz argümanları elde edebilmektedir. Örneğin:

.. code-block:: bash

    $ ./sample -a ali -b veli selami -c

Burada ``ali`` ve ``selami`` seçeneksiz argümanlardır. ``getopt`` bu ``argv`` dizisini şu hale getirmektedir:

.. code-block:: bash

    ./sample -a -b veli -c ali selami

Şimdi burada ``optind`` indeksi artık ``ali`` argümanının başladığı indeksi belirtecektir. Onun ötesindeki tüm argümanlar
seçeneksiz argümanlardır. Bu argümanları ``while`` döngüsünün dışında şöyle yazdırabiliriz:

.. code-block:: c

    for (int i = optind; i < argc; ++i)
        puts(argv[i]);

Burada bir noktaya dikkatinizi çekmek istiyoruz. Argümanlı seçeneklerde argüman girilmemişse fakat bu seçenekten sonra
başka bir komut satırı seçeneği ya da argümanı varsa ``getopt`` bu argümanlı seçeneğin argümanının ondan sonra gelen
seçenek ya da argüman olduğunu sanmaktadır. Örneğin ``"ab:c"`` seçeneklerinin söz konusu olduğu durumda kullanıcı
programı şöyle çalıştırmış olsun:

.. code-block:: bash

    $ ./sample -a -b -c

Burada kullanıcı ``-b`` için bir argüman girmeyi unutmuştur. ``getopt`` bunu anlayamaz. Bu durumda ``getopt`` sanki
``-c`` seçeneğini ``-b`` seçeneğinin argümanıymış gibi ele almaktadır. Dolayısıyla ``getopt`` bu durumda ``-b``
seçeneği için ``'?'`` karakteriyle geri dönmeyecektir.

Bayrak Değişkenleriyle Seçenek Yönetimi
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Peki komut satırı seçeneklerini program içerisinde nasıl kullanabiliriz? İşte bunun için en klasik yöntem her komut
satırı seçeneği için bir bayrak bulundurmak, bayrakları da ``getopt`` döngüsünde set etmektir. ``getopt`` döngüsünden
çıkıldıktan sonra bayraklara bakılarak hangi seçeneklerin belirtildiği tespit edilebilir. Uygulamaların çoğunda bazı
seçenekler bazı seçeneklerle birlikte kullanılamamaktadır. Programcının ``getopt`` döngüsünden çıktıktan sonra
seçeneklerin doğru kullanılmış olduğunu kontrol etmesi gerekir. Bunun için kullanabileceğiniz bir kalıp şöyle olabilir:

- Her seçenek için bir bayrak değişkeni tutulur. Bu bayrak değişkenlerine başlangıçta ``0`` atanır.
- Her argümanlı seçenek için bir gösterici bulundurulur.
- ``getopt`` döngüsünde her seçenekle karşılaşıldığında bayrak değişkenine ``1`` atanarak o seçeneğin verilmiş olduğu
  kaydedilir.
- Argümanlı seçeneklerle karşılaşıldığında ``optarg`` global değişkeninden faydalanılarak ilgili göstericinin seçeneğin
  argümanını göstermesi sağlanır.

Genellikle programlarda aynı seçeneğin birden fazla kez belirtilmesine yönelik kontroller yapılmamaktadır. Bu tür
durumlarda aynı bayrak değişkeni birden fazla kez set edilir ancak programda bir davranış farklılığı oluşmaz. Örneğin
``ls`` programında da böyle bir kontrol yapılmamıştır:

.. code-block:: bash

    $ ls -lllllllllll



getopt Kullanım Kalıbı
===============================================

``getopt`` fonksiyonunun kullanımına ilişkin tipik bir kalıp aşağıda verilmiştir. Bu örnekte ``-a``, ``-b``, ``-d``
argümansız seçenekler; ``-c`` ve ``-e`` ise argümanlı seçeneklerdir. Bu kalıbı kendi programlarınızda da
kullanabilirsiniz. Bu örnekte ayrıştırma işleminde bir hata oluştuğunda programın devam etmemesini isteriz. Ancak tüm
hataların rapor edilmesi de gerekmektedir. Bunun için bir bayrak değişkeninden faydalanılabilir. Bu bayrak değişkeni
hata durumunda set edilir. Çıkışta kontrol edilip duruma göre program sonlandırılır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    int main(int argc, char *argv[])
    {
        int result;
        int a_flag, b_flag, c_flag, d_flag, e_flag, err_flag;
        char *c_arg, *e_arg;

        a_flag = b_flag = c_flag = d_flag = e_flag = err_flag = 0;

        opterr = 0;
        while ((result = getopt(argc, argv, "abc:de:")) != -1) {
            switch (result) {
                case 'a':
                    a_flag = 1;
                    break;
                case 'b':
                    b_flag = 1;
                    break;
                case 'c':
                    c_flag = 1;
                    c_arg = optarg;
                    break;
                case 'd':
                    d_flag = 1;
                    break;
                case 'e':
                    e_flag = 1;
                    e_arg = optarg;
                    break;
                case '?':
                    if (optopt == 'c' || optopt == 'e')
                        fprintf(stderr, "-%c option must have an argument!\n", optopt);
                    else
                        fprintf(stderr, "-%c invalid option!\n", optopt);
                    err_flag = 1;
            }
        }

        if (err_flag)
            exit(EXIT_FAILURE);

        if (a_flag)
            printf("-a option given\n");
        if (b_flag)
            printf("-b option given\n");
        if (c_flag)
            printf("-c option given with argument \"%s\"\n", c_arg);
        if (d_flag)
            printf("-d option given\n");
        if (e_flag)
            printf("-e option given with argument \"%s\"\n", e_arg);

        if (optind != argc)
            printf("Arguments without option:\n");
        for (int i = optind; i < argc; ++i)
            puts(argv[i]);

        return 0;
    }

----

Örnek: disp Programı
-------------------------

``getopt`` fonksiyonunun kullanımına bir örnek. Bu örnekte ``disp`` isimli bir program yazılmıştır. Program şu komut
satırı seçeneklerine sahiptir:

- ``-x``: Onaltılık (hex) görüntüleme
- ``-o``: Sekizlik (octal) görüntüleme
- ``-t``: Metin olarak görüntüleme
- ``-n <arg>``: Satır başına karakter sayısı

Burada ``-x``, ``-o`` ve ``-t`` seçeneklerinden yalnızca bir tanesi kullanılabilmektedir. Eğer hiçbir seçenek
kullanılmazsa varsayılan durum ``-t`` biçimindedir. ``-n`` seçeneği yalnızca hex ve octal görüntülemede
kullanılabilmektedir. Bu seçenek de belirtilmezse ``-n 16`` gibi bir belirleme yapıldığı varsayılmaktadır.

.. code-block:: c

    /* disp.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <ctype.h>
    #include <stdbool.h>
    #include <unistd.h>

    #define DEFAULT_LINE_CHAR        16

    bool disp_text(FILE *f);
    bool disp_hex(FILE *f, int n_arg);
    bool disp_octal(FILE *f, int n_arg);
    int check_number(const char *str);

    int main(int argc, char *argv[])
    {
        int result;
        int t_flag, o_flag, x_flag, n_flag, err_flag;
        int n_arg;
        FILE *f;

        t_flag = o_flag = x_flag = n_flag = err_flag = 0;
        n_arg = DEFAULT_LINE_CHAR;
        opterr = 0;

        while ((result = getopt(argc, argv, "toxn:")) != -1) {
            switch (result) {
                case 't':
                    t_flag = 1;
                    break;
                case 'o':
                    o_flag = 1;
                    break;
                case 'x':
                    x_flag = 1;
                    break;
                case 'n':
                    n_flag = 1;
                    if ((n_arg = check_number(optarg)) < 0) {
                        fprintf(stderr, "-n argument is invalid!...\n");
                        err_flag = 1;
                    }
                    break;
                case '?':
                    if (optopt == 'n')
                        fprintf(stderr, "-%c option given without argument!...\n", optopt);
                    else
                        fprintf(stderr, "invalid option: -%c\n", optopt);
                    err_flag = 1;
                    break;
            }
        }

        if (err_flag)
            exit(EXIT_FAILURE);

        if (t_flag + o_flag + x_flag > 1) {
            fprintf(stderr, "only one of -[tox] option may be specified!...\n");
            exit(EXIT_FAILURE);
        }

        if (t_flag + o_flag + x_flag == 0)
            t_flag = 1;

        if (t_flag && n_flag) {
            fprintf(stderr, "-n option cannot be used with -t option!...\n");
            exit(EXIT_FAILURE);
        }

        if (argc - optind == 0) {
            fprintf(stderr, "file must be specified!...\n");
            exit(EXIT_FAILURE);
        }
        if (argc - optind > 1) {
            fprintf(stderr, "too many files specified!...\n");
            exit(EXIT_FAILURE);
        }

        if ((f = fopen(argv[optind], t_flag ? "r" : "rb")) == NULL) {
            fprintf(stderr, "cannot open file: %s\n", argv[optind]);
            exit(EXIT_FAILURE);
        }
        if (t_flag)
            result = disp_text(f);
        else if (x_flag)
            result = disp_hex(f, n_arg);
        else if (o_flag)
            result = disp_octal(f, n_arg);

        if (!result) {
            fprintf(stderr, "cannot read file: %s\n", argv[optind]);
            exit(EXIT_FAILURE);
        }

        fclose(f);

        return 0;
    }

    bool disp_text(FILE *f)
    {
        int ch;

        while ((ch = fgetc(f)) != EOF)
            putchar(ch);

        return feof(f);
    }

    bool disp_hex(FILE *f, int n_arg)
    {
        size_t i;
        int ch;

        for (i = 0; (ch = fgetc(f)) != EOF; ++i) {
            if (i % n_arg == 0) {
                if (i != 0)
                    putchar('\n');
                printf("%08zX ", i);
            }
            printf("%02X ", ch);
        }
        putchar('\n');

        return feof(f);
    }

    bool disp_octal(FILE *f, int n_arg)
    {
        size_t i;
        int ch;

        for (i = 0; (ch = fgetc(f)) != EOF; ++i) {
            if (i % n_arg == 0)
                printf("%08zo ", i);

            printf("%03o ", ch);
            if (i % n_arg == n_arg - 1)
                putchar('\n');
        }
        putchar('\n');

        return feof(f);
    }

    int check_number(const char *str)
    {
        const char *temp;
        int result;

        while (isspace(*str))
            ++str;

        temp = str;

        while (isdigit(*str))
            ++str;

        if (*str != '\0')
            return -1;

        result = atoi(temp);
        if (!result)
            return -1;

        return result;
    }

----

Komut Satırı Argümanlarını Fonksiyona Devretme
===============================================

Komut satırı argümanlarını parse etmek uzun bir kod bloğu gerektirmektedir. Bu kısım bir fonksiyona da devredilebilir.
Ancak bu durumda fonksiyondan elde edilen bilgilerin dışarıya iletilmesi gerekir. Bu tipik olarak bir yapı yoluyla
sağlanabilir. Örneğin:

.. code-block:: c

    void check_args(int argc, char *const argv[], struct arginfo *arginfo)
    {
        /* ... */
    }

``check_args`` fonksiyonu ``main`` fonksiyonunun ``argc`` ve ``argv`` parametrelerinin yanı sıra ``arginfo`` isimli bir
yapı nesnesinin de adresini almaktadır. Bu yapı nesnesinin elemanları fonksiyon tarafından doldurulacaktır. Yukarıdaki
``disp.c`` örneği için bu yapı şöyle olabilir:

.. code-block:: c

    struct arginfo {
        int x_flag, o_flag, t_flag, n_flag;
        int bpl;
        char *path;
    };

Aşağıdaki örnekte komut satırı argümanları böyle bir fonksiyon tarafından parse edilmiştir. Bu örnek aynı zamanda
yukarıdaki ``disp.c`` örneğinin daha gelişmiş bir biçimidir. Burada hex ve octal görüntüleme yapılırken her satırın
yanında o satırdaki byte'ların ASCII karşılıkları da yazdırılmıştır. Örneğin hex görüntüleme şuna benzer yapılmaktadır:

.. code-block:: text

    00000000  74 6F 70 6C 61 6D 20 36 30 0A 2D 72 77 78 72 77   |toplam.60.-rwxrw|
    00000010  78 72 2D 78 20 31 20 6B 61 61 6E 20 6B 61 61 6E   |xr-x.1.kaan.kaan|
    00000020  20 31 36 33 38 34 20 4D 61 79 20 31 34 20 31 32   |.16384.May.14.12|
    00000030  3A 32 39 20 64 69 73 70 0A 2D 72 77 2D 72 77 2D   |:29.disp.-rw-rw-|
    00000040  72 2D 2D 20 31 20 6B 61 61 6E 20 6B 61 61 6E 20   |r--.1.kaan.kaan.|
    00000050  20 31 33 31 37 20 4D 61 79 20 31 34 20 31 32 3A   |.1317.May.14.12:|
    00000060  32 38 20 64 69 73 70 2E 63 0A 2D 72 77 78 72 77   |28.disp.c.-rwxrw|
    00000070  78 72 2D 78 20 31 20 6B 61 61 6E 20 6B 61 61 6E   |xr-x.1.kaan.kaan|
    00000080  20 31 36 33 36 38 20 4D 61 79 20 31 34 20 31 32   |.16368.May.14.12|
    .....

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    #define DEF_BYTE_PER_LINE       16
    #define MAX_BPL                 64
    #define MIN_BPL                 4

    struct arginfo {
        int x_flag, o_flag, t_flag, n_flag;
        int bpl;
        char *path;
    };

    void check_args(int argc, char *const argv[], struct arginfo *arginfo);
    void disp_text(FILE *f);
    void disp_hex_octal(FILE *f, int bpl, int dtype);

    int main(int argc, char *argv[])
    {
        struct arginfo arginfo;
        FILE *f;

        check_args(argc, argv, &arginfo);

        if ((f = fopen(arginfo.path, "r")) == NULL) {
            fprintf(stderr, "cannot open file: %s\n", argv[optind]);
            exit(EXIT_FAILURE);
        }

        if (arginfo.t_flag)
            disp_text(f);
        else if (arginfo.x_flag || arginfo.o_flag)
            disp_hex_octal(f, arginfo.bpl, arginfo.x_flag);

        fclose(f);

        return 0;
    }

    void check_args(int argc, char *const argv[], struct arginfo *arginfo)
    {
        int result;
        int x_flag, o_flag, t_flag, n_flag, err_flag;
        const char *n_arg;
        int bpl;

        x_flag = o_flag = t_flag = n_flag = err_flag = 0;

        opterr = 0;
        while ((result = getopt(argc, argv, "xotn:")) != -1) {
            switch (result) {
                case 'x':
                    x_flag = 1;
                    break;
                case 'o':
                    o_flag = 1;
                    break;
                case 't':
                    t_flag = 1;
                    break;
                case 'n':
                    n_flag = 1;
                    n_arg = optarg;
                    break;
                case '?':
                    if (optopt == 'n')
                        fprintf(stderr, "-%c option must have an argument!\n", optopt);
                    else
                        fprintf(stderr, "-%c invalid option!\n", optopt);
                    err_flag = 1;
            }
        }

        if (err_flag)
            exit(EXIT_FAILURE);

        if (argc - optind != 1) {
            fprintf(stderr, "path must be specified!..\n");
            exit(EXIT_FAILURE);
        }

        if (x_flag + o_flag + t_flag > 1) {
            fprintf(stderr, "only one option may be specified!..\n");
            exit(EXIT_FAILURE);
        }

        if (x_flag + o_flag == 0)
            t_flag = 1;

        if (n_flag) {
            if (x_flag + o_flag != 1) {
                fprintf(stderr, "-n option must be used with either -x or -o\n");
                exit(EXIT_FAILURE);
            }
            bpl = atoi(n_arg);
            if (bpl > MAX_BPL) {
                fprintf(stderr, "-n argument too big!..\n");
                exit(EXIT_FAILURE);
            }
            else if (bpl < MIN_BPL) {
                fprintf(stderr, "-n argument too small!..\n");
                exit(EXIT_FAILURE);
            }
        }
        else
            bpl = 16;

        arginfo->x_flag = x_flag;
        arginfo->o_flag = o_flag;
        arginfo->t_flag = t_flag;
        arginfo->bpl = bpl;
        arginfo->path = argv[optind];
    }

    void disp_text(FILE *f)
    {
        int ch;

        while ((ch = fgetc(f)) != EOF)
            putchar(ch);

        if (ferror(f)) {
            fprintf(stderr, "cannot read file!..\n");
            fclose(f);
            exit(EXIT_FAILURE);
        }
    }

    void disp_hex_octal(FILE *f, int bpl, int dtype)
    {
        int ch;
        int i;
        unsigned char line_buf[MAX_BPL];

        for (i = 0; (ch = fgetc(f)) != EOF; ++i) {
            if (i % bpl == 0)
                printf("%08X  ", i);
            printf(dtype ? "%02X" : "%03o", ch);
            line_buf[i % bpl] = ch;
            if (i % bpl == bpl - 1) {
                printf("   |");
                for (int k = 0; k < bpl; ++k)
                    if (line_buf[k] > 32 && line_buf[k] < 128)
                        putchar(line_buf[k]);
                    else
                        putchar('.');
                printf("|\n");
            }
            else
                putchar(' ');
        }

        if (ferror(f)) {
            fprintf(stderr, "cannot read file!..\n");
            fclose(f);
            exit(EXIT_FAILURE);
        }

        if (i % bpl != 0) {
            int space_len = (bpl - i % bpl) * (dtype ? 3 : 4) - 1;

            for (int k = 0; k < space_len; ++k)
                putchar(' ');
            printf("   |");
            for (int k = 0; k < i % bpl; ++k)
                if (line_buf[k] > 32 && line_buf[k] < 128)
                    putchar(line_buf[k]);
                else
                    putchar('.');
            printf("|\n");
        }
    }

----

Örnek: mycalc Programı
---------------------------

Aşağıdaki örnekte ``mycalc`` isimli bir program yazılmıştır. Program iki komut satırı argümanı ile aldığı değerler
üzerinde dört işlem yapmaktadır. Aşağıdaki seçeneklere sahiptir:

- ``-a``: Toplama işlemi
- ``-m``: Çarpma işlemi
- ``-d``: Bölme işlemi
- ``-s``: Çıkartma işlemi
- ``-M <msg>``: Çıktının başında ``msg:`` kısmını ekler

.. code-block:: c

    /* mycalc.c */

    #include <stdio.h>
    #include <stdlib.h>
    #include <unistd.h>

    int main(int argc, char *argv[])
    {
        int result;
        int a_flag, m_flag, M_flag, d_flag, s_flag, err_flag;
        char *M_arg;
        double arg1, arg2, calc_result;

        a_flag = m_flag = M_flag = d_flag = s_flag = err_flag = 0;

        opterr = 0;

        while ((result = getopt(argc, argv, "amM:ds")) != -1) {
            switch (result) {
                case 'a':
                    a_flag = 1;
                    break;
                case 'm':
                    m_flag = 1;
                    break;
                case 'M':
                    M_flag = 1;
                    M_arg = optarg;
                    break;
                case 'd':
                    d_flag = 1;
                    break;
                case 's':
                    s_flag = 1;
                    break;
                case '?':
                    if (optopt == 'M')
                        fprintf(stderr, "-M option must have an argument!\n");
                    else
                        fprintf(stderr, "invalid option: -%c\n", optopt);
                    err_flag = 1;
            }
        }

        if (err_flag)
            exit(EXIT_FAILURE);

        if (a_flag + m_flag + d_flag + s_flag > 1) {
            fprintf(stderr, "only one option must be specified!\n");
            exit(EXIT_FAILURE);
        }
        if (a_flag + m_flag + d_flag + s_flag == 0) {
            fprintf(stderr, "at least one of -amds options must be specified\n");
            exit(EXIT_FAILURE);
        }

        if (argc - optind != 2) {
            fprintf(stderr, "two number must be specified!\n");
            exit(EXIT_FAILURE);
        }

        arg1 = atof(argv[optind]);
        arg2 = atof(argv[optind + 1]);

        if (a_flag)
            calc_result = arg1 + arg2;
        else if (m_flag)
            calc_result = arg1 * arg2;
        else if (d_flag)
            calc_result = arg1 / arg2;
        else
            calc_result = arg1 - arg2;

        if (M_flag)
            printf("%s: %f\n", M_arg, calc_result);
        else
            printf("%f\n", calc_result);

        return 0;
    }

----

getopt_long Fonksiyonu
=============================

Daha önceden de belirttiğimiz gibi komut satırında uzun seçenek kullanımı POSIX standartlarında yoktur. Ancak Linux gibi
pek çok sistemdeki çeşitli yardımcı programlar uzun seçenekleri desteklemektedir. Bu programlarda bazı kısa seçeneklerin
eşdeğer uzun seçenekleri bulunmaktadır. Bazı uzun seçeneklerin ise kısa seçenek eşdeğeri bulunmamaktadır. Bazı kısa
seçeneklerin de uzun seçenek eşdeğerleri yoktur.

Uzun seçenekleri parse etmek için ``getopt_long`` isimli fonksiyon kullanılmaktadır. Uzun seçenekler POSIX
standartlarında olmadığına göre ``getopt_long`` fonksiyonu da bir POSIX fonksiyonu değildir. Ancak GNU'nun ``glibc``
kütüphanesinde bir eklenti biçiminde bulunmaktadır. ``getopt_long`` fonksiyonu işlevsel olarak ``getopt`` fonksiyonunu
kapsamaktadır. Fonksiyonun prototipi şöyledir:

.. code-block:: c

    #include <getopt.h>

    int getopt_long(int argc, char * const argv[], const char *optstring,
                    const struct option *longopts, int *longindex);

Fonksiyonun birinci ve ikinci parametrelerine ``main`` fonksiyonundan alınan ``argc`` ve ``argv`` parametreleri
geçirilir. Fonksiyonun üçüncü parametresi yine kısa seçeneklerin belirtildiği yazının adresini almaktadır. Yani
fonksiyonun ilk üç parametresi tamamen ``getopt`` fonksiyonu ile aynıdır. Fonksiyonun dördüncü parametresi uzun
seçeneklerin belirtildiği ``struct option`` türünden bir yapı dizisinin adresini almaktadır. Her uzun seçenek
``struct option`` türünden bir nesneyle ifade edilmektedir. ``struct option`` yapısı şöyle bildirilmiştir:

.. code-block:: c

    struct option {
        const char *name;
        int         has_arg;
        int        *flag;
        int         val;
    };

Fonksiyon bu yapı dizisinin bittiğini nasıl anlayacaktır? İşte yapı dizisinin son elemanına ilişkin yapı nesnesinin
tüm elemanları 0'larla doldurulmalıdır. (0 sabitinin göstericiler söz konusu olduğunda NULL adres anlamına geldiğini
de anımsayınız.)

``struct option`` yapısının ``name`` elemanı uzun seçeneğin ismini belirtmektedir (isimde ``--`` kullanılmaz). Yapının
``has_arg`` elemanı üç değerden birini alabilir:

.. code-block:: c

    #define no_argument             0
    #define required_argument       1
    #define optional_argument       2

Bu eleman uzun seçeneğin argüman alıp almadığını belirtmektedir. Yapının ``flag`` ve ``val`` elemanları birbirleriyle
ilişkilidir. Yapının ``val`` elemanı uzun seçenek bulunduğunda bunun hangi sayısal değerle ifade edileceğini belirtir.
İşte bu ``flag`` elemanına ``int`` bir nesnenin adresi geçilirse bu durumda uzun seçenek bulunduğunda bu ``val`` değeri
bu ``int`` nesneye yerleştirilir ve ``getopt_long`` bu durumda ``0`` değeri ile geri döner. Ancak bu ``flag``
göstericisine NULL adres de geçilebilir. Bu durumda ``getopt_long`` uzun seçenek bulunduğunda ``val`` elemanındaki
değeri geri dönüş değeri olarak verir. Örneğin:

.. code-block:: c

    struct option options[] = {
        {"count", required_argument, NULL, 'c'},
        {0, 0, 0, 0}
    };

Burada uzun seçenek ``--count`` biçimindedir. Bir argümanla kullanılmak zorundadır. Bu uzun seçenek bulunduğunda
``flag`` parametresi NULL adres geçildiği için ``getopt_long`` fonksiyonu ``'c'`` değeri ile geri dönecektir. Örneğin:

.. code-block:: c

    int count_flag;

    struct option options[] = {
        {"count", required_argument, &count_flag, 1},
        {0, 0, 0, 0}
    };

Burada artık uzun seçenek bulunduğunda ``getopt_long`` fonksiyonu ``0`` ile geri dönecek ancak ``1`` değeri
``count_flag`` nesnesine yerleştirilecektir.

``getopt_long`` fonksiyonunun son parametresi uzun seçenek bulunduğunda o uzun seçeneğin ``option`` dizisindeki kaçıncı
indeksli uzun seçenek olduğunu anlamak için kullanılmaktadır. Burada belirtilen adresteki nesneye uzun seçeneğin
``option`` dizisi içerisindeki indeks numarası yerleştirilmektedir. Ancak bu bilgiye genellikle gereksinim
duyulmamaktadır. Bu parametre NULL geçilebilir. Bu durumda böyle bir yerleştirme yapılmaz.

getopt_long Fonksiyonunun Geri Dönüş Değerleri
----------------------------------------------------

``getopt_long`` fonksiyonunun geri dönüş değeri beş biçimden biri olabilir:

1. Fonksiyon bir kısa seçenek bulmuştur. Kısa seçeneğin karakter koduyla geri döner.
2. Fonksiyon bir uzun seçenek bulmuştur ve ``option`` yapısının ``flag`` elemanında NULL adres vardır. Bu durumda
   fonksiyon ``option`` yapısının ``val`` elemanındaki değerle geri döner.
3. Fonksiyon bir uzun seçenek bulmuştur ve ``option`` yapısının ``flag`` elemanında NULL adres yoktur. Bu durumda
   fonksiyon ``val`` değerini bu adrese yerleştirir ve ``0`` değeri ile geri döner. Biz bu sayede bir uzun seçenek
   bulunduğunda doğrudan o uzun seçenek için tanımladığımız bayrak değişkenini set edebiliriz.
4. Fonksiyon geçersiz (yani olmayan) bir kısa ya da uzun seçenekle karşılaşmıştır ya da argümanlı bir kısa seçeneğin
   ya da uzun seçeneğin argümanı girilmemiştir. Bu durumda fonksiyon ``'?'`` karakterinin değeriyle geri döner.
5. Parse edecek argüman kalmamıştır; fonksiyon ``-1`` ile geri döner.

``getopt`` fonksiyonundaki yardımcı global değişkenlerin aynısı burada da kullanılmaktadır:

- ``opterr``: Hata mesajının fonksiyon tarafından ``stderr`` dosyasına basılıp basılmayacağını belirtir.
- ``optarg``: Argümanlı bir kısa ya da uzun seçenekte argümanı belirtmektedir. Eğer *isteğe bağlı argümanlı* bir uzun
  seçenek bulunmuşsa ve bu uzun seçenek için argüman girilmemişse ``optarg`` nesnesine NULL adres yerleştirilmektedir.
- ``optind``: Bu değişken yine seçeneksiz argümanların başladığı indeksi belirtmektedir.
- ``optopt``: Bu değişken geçersiz bir uzun ya da kısa seçenek girildiğinde hatanın nedenini belirtmektedir.

``getopt_long`` geçersiz bir seçenekle karşılaştığında ``'?'`` karakteri ile geri dönmekle birlikte ``optopt``
değişkenini şu biçimlerde set etmektedir:

1. Eğer fonksiyon geçersiz bir kısa seçenekle karşılaşmışsa ``optopt`` geçersiz kısa seçeneğin karakter karşılığına
   set edilir.
2. Eğer fonksiyon argümanlı bir kısa seçenek bulduğu halde argüman girilmemişse o argümanlı kısa seçeneğin karakter
   karşılığını ``optopt`` değişkenine yerleştirir.
3. Eğer fonksiyon argümanlı bir uzun seçenek bulduğu halde argüman girilmemişse o argümanlı uzun seçeneğin ``option``
   yapısındaki ``val`` değerini ``optopt`` değişkenine yerleştirmektedir.
4. Eğer fonksiyon geçersiz bir uzun seçenekle karşılaşmışsa ``optopt`` değişkenine ``0`` değeri yerleştirilmektedir.

Maalesef ``getopt_long`` olmayan bir uzun seçenek girildiğinde bu uzun seçeneği bize vermemektedir. Ancak GNU'nun
``getopt_long`` gerçekleştirimine bakıldığında bu geçersiz uzun seçeneğin ``argv`` dizisinin ``optind - 1`` indeksinde
olduğu görülmektedir. Yani bu geçersiz uzun seçeneğe ``argv[optind - 1]`` ifadesi ile erişilebilmektedir. Ancak bu
durum ``glibc`` dokümanlarında belirtilmemiştir. Bu nedenle bu özelliğin kullanılması uygun değildir.

----

getopt_long Örnekleri
--------------------------

Örnek 1: --count ve --verbose
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Aşağıdaki örnekteki komut satırı argümanları şunlardır:

- ``-a``
- ``-b``
- ``-c <arg>`` ya da ``--count <arg>``
- ``--verbose``

Burada ``option`` yapısı şöyle oluşturulmuştur:

.. code-block:: c

    struct option options[] = {
        {"count",   required_argument, NULL,           'c'},
        {"verbose", no_argument,       &verbose_flag,  1},
        {0, 0, 0, 0}
    };

``--count`` seçeneği için ``getopt_long``, ``'c'`` ile geri döndürülmüştür. ``-c`` kısa seçeneğinde de
``getopt_long`` fonksiyonunun ``'c'`` geri döndürdüğüne dikkat ediniz. ``--verbose`` seçeneğinde yapının ``flag``
elemanına doğrudan ``verbose_flag`` değişkeninin adresi girilmiştir. Böylece ``verbose`` bayrağı ``switch`` içerisinde
değil doğrudan set edilmiştir. Uzun seçenekte bayrakların bu biçimde doğrudan set edilmesi pratiklik sağlamaktadır.
Programda ``getopt_long`` döngüsü şöyledir:

.. code-block:: c

    while ((result = getopt_long(argc, argv, "abc:", options, NULL)) != -1) {
        /* ... */
    }

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <getopt.h>

    int main(int argc, char *argv[])
    {
        int a_flag, b_flag, c_flag, verbose_flag;
        int err_flag;
        char *c_arg;
        int result;

        struct option options[] = {
            {"count",   required_argument, NULL,          'c'},
            {"verbose", no_argument,       &verbose_flag,  1},
            {0, 0, 0, 0}
        };

        a_flag = b_flag = c_flag = verbose_flag = err_flag = 0;

        opterr = 0;
        while ((result = getopt_long(argc, argv, "abc:", options, NULL)) != -1) {
            switch (result) {
                case 'a':
                    a_flag = 1;
                    break;
                case 'b':
                    b_flag = 1;
                    break;
                case 'c':
                    c_flag = 1;
                    c_arg = optarg;
                    break;
                case '?':
                    if (optopt == 'c')
                        fprintf(stderr, "option -c or --count without argument!...\n");
                    else if (optopt != 0)
                        fprintf(stderr, "invalid option: -%c\n", optopt);
                    else
                        fprintf(stderr, "invalid long option!...\n");
                    err_flag = 1;
                    break;
            }
        }

        if (err_flag)
            exit(EXIT_FAILURE);

        if (a_flag)
            printf("-a option given\n");
        if (b_flag)
            printf("-b option given\n");
        if (c_flag)
            printf("-c or --count option given with argument \"%s\"\n", c_arg);
        if (verbose_flag)
            printf("--verbose given\n");

        if (optind != argc) {
            printf("Arguments without options");
            for (int i = optind; i < argc; ++i)
                printf("%s\n", argv[i]);
        }

        return 0;
    }

Örnek 2: --help, --count ve --line
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Aşağıdaki programın komut satırı argümanları şunlardır:

- ``-a``
- ``-b <arg>``
- ``-c``
- ``-h`` ya da ``--help``
- ``--count <arg>``
- ``--line[=<arg>]``

Bu programdaki ``option`` yapı dizisi şöyle oluşturulmuştur:

.. code-block:: c

    struct option options[] = {
        {"help",  no_argument,       &h_flag, 1},
        {"count", required_argument, NULL,    2},
        {"line",  optional_argument, NULL,    3},
        {0, 0, 0, 0},
    };

Tabii biz bu örnekte de aslında ``--count`` ve ``--line`` için ``--help`` seçeneğinde yaptığımız gibi doğrudan bayrak
değişkenini set edebilirdik. Ancak örnek alıştırma amacıyla oluşturulmuştur. Siz bu tür durumlarda doğrudan bayrak
değişkenini set edebilirsiniz. Örneğin:

.. code-block:: c

    struct option options[] = {
        {"help",  no_argument,       &h_flag,     1},
        {"count", required_argument, &count_flag, 1},
        {"line",  optional_argument, &line_flag,  1},
        {0, 0, 0, 0},
    };

Programdaki ``getopt_long`` döngüsü de şöyle oluşturulmuştur:

.. code-block:: c

    while ((result = getopt_long(argc, argv, "abc:", long_options, NULL)) != -1) {
        /* ... */
    }

Burada ``--line`` isteğe bağlı bir argüman almaktadır. İsteğe bağlı uzun seçeneklerde argümanla seçenek yapışık
biçimde ``=`` karakteri ile belirtilmektedir. Örneğin:

.. code-block:: bash

    $ ./sample -a -b --line=1024

Eğer isteğe bağlı argümanlı uzun seçeneklerde ``=`` karakterini kullanmazsanız artık girdiğiniz argüman uzun seçeneğe
ilişkin olmaz. Örneğin:

.. code-block:: bash

    $ ./sample -a -b --line 1024

Burada ``getopt_long``, ``--line`` için seçenek belirtilmediğini, ``1024``'ün ise seçeneksiz argüman olduğunu
düşünecektir. İsteğe bağlı argümanlı uzun seçeneklerde eğer argüman belirtilirse bu argümanı ``optarg`` global
değişkeni ile elde edebiliriz. Argüman belirtilmezse ``optarg`` global değişkeninde o sırada NULL adres bulunacaktır.

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <getopt.h>

    int main(int argc, char *argv[])
    {
        int result;
        int a_flag, b_flag, c_flag, h_flag, count_flag, line_flag;
        char *b_arg, *count_arg, *line_arg;
        int err_flag;
        int i;

        struct option options[] = {
            {"help",  no_argument,       &h_flag, 1},
            {"count", required_argument, NULL,    2},
            {"line",  optional_argument, NULL,    3},
            {0, 0, 0, 0},
        };

        a_flag = b_flag = c_flag = h_flag = count_flag = line_flag = 0;
        err_flag = 0;

        opterr = 0;
        while ((result = getopt_long(argc, argv, "ab:ch", options, NULL)) != -1) {
            switch (result) {
                case 'a':
                    a_flag = 1;
                    break;
                case 'b':
                    b_flag = 1;
                    b_arg = optarg;
                    break;
                case 'c':
                    c_flag = 1;
                    break;
                case 'h':
                    h_flag = 1;
                    break;
                case 2:            /* --count */
                    count_flag = 1;
                    count_arg = optarg;
                    break;
                case 3:            /* --line */
                    line_flag = 1;
                    line_arg = optarg;
                    break;
                case '?':
                    if (optopt == 'b')
                        fprintf(stderr, "-b option must have an argument!...\n");
                    else if (optopt == 2)
                        fprintf(stderr, "argument must be specified with --count option\n");
                    else if (optopt != 0)
                        fprintf(stderr, "invalid option: -%c\n", optopt);
                    else
                        fprintf(stderr, "invalid long option!...\n");
                    err_flag = 1;
                    break;
            }
        }

        if (err_flag)
            exit(EXIT_FAILURE);

        if (a_flag)
            printf("-a option given...\n");
        if (b_flag)
            printf("-b option given with argument \"%s\"...\n", b_arg);
        if (c_flag)
            printf("-c option given...\n");
        if (h_flag)
            printf("-h or --help option given...\n");
        if (count_flag)
            printf("--count option specified with \"%s\"...\n", count_arg);
        if (line_flag) {
            if (line_arg != NULL)
                printf("--line option given with optional argument \"%s\"\n", line_arg);
            else
                printf("--line option given without optional argument...\n");
        }

        if (optind != argc) {
            printf("Arguments without options:\n");
            for (i = optind; i < argc; ++i)
                printf("%s\n", argv[i]);
        }

        return 0;
    }

Örnek 3: --display, --vertical ve --count
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Bu örnekteki seçenekler şöyledir:

- ``-a``: Argümansız kısa seçenek
- ``-b``: Argümansız kısa seçenek
- ``-c <arg>``: Argümanlı kısa seçenek, aynı zamanda ``--count <arg>`` uzun seçeneğiyle eşdeğerdir
- ``--display``: Argümansız uzun seçenek
- ``--vertical[=<arg>]``: İsteğe bağlı argümanlı uzun seçenek

Burada kullanılan ``option`` yapı dizisi şöyle oluşturulmuştur:

.. code-block:: c

    struct option long_options[] = {
        {"display",  no_argument,       NULL, 100},
        {"vertical", optional_argument, NULL, 101},
        {"count",    required_argument, NULL, 'c'},
        {0, 0, 0, 0}
    };

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <getopt.h>

    int main(int argc, char *argv[])
    {
        int result;
        const char *c_arg, *vertical_arg;
        int a_flag, b_flag, c_flag, display_flag, vertical_flag, err_flag;

        struct option long_options[] = {
            {"display",  no_argument,       NULL, 100},
            {"vertical", optional_argument, NULL, 101},
            {"count",    required_argument, NULL, 'c'},
            {0, 0, 0, 0}
        };

        a_flag = b_flag = c_flag = display_flag = vertical_flag = err_flag = 0;

        opterr = 0;
        while ((result = getopt_long(argc, argv, "abc:", long_options, NULL)) != -1) {
            switch (result) {
                case 'a':
                    a_flag = 1;
                    break;
                case 'b':
                    b_flag = 1;
                    break;
                case 'c':
                    c_flag = 1;
                    c_arg = optarg;
                    break;
                case 100:
                    display_flag = 1;
                    break;
                case 101:
                    vertical_flag = 1;
                    vertical_arg = optarg;
                    break;
                case '?':
                    if (optopt == 'c')
                        printf("-%c must have an argument\n", optopt);
                    else if (optopt != 0)
                        printf("invalid option: -%c\n", optopt);
                    else
                        printf("%s invalid long option\n", argv[optind - 1]);
                    err_flag = 1;
                    break;
            }
        }

        if (err_flag)
            exit(EXIT_FAILURE);

        printf("Arguments without option:\n");
        for (int i = optind; i < argc; ++i)
            printf("%s\n", argv[i]);

        if (a_flag)
            printf("-a option specified\n");
        if (b_flag)
            printf("-b option specified\n");
        if (c_flag)
            printf("-c option specified with argument \"%s\"\n", c_arg);
        if (vertical_flag) {
            if (vertical_arg != NULL)
                printf("--vertical option specified with argument \"%s\"\n", vertical_arg);
            else
                printf("--vertical option specified without argument\n");
        }
        if (display_flag)
            printf("--display option specified\n");

        return 0;
    }

Örnek 4: flag Elemanına Adres Geçme ve Kısa/Uzun Seçenek Eşdeğerleri
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

``getopt_long`` fonksiyonunda ``struct option`` yapısındaki ``flag`` elemanına NULL adres yerine ``int`` bir nesnenin
adresi geçirilirse bu durumda ``getopt_long`` bu uzun seçenek girildiğinde doğrudan yapının ``val`` elemanındaki değeri
bu nesneye yerleştirir ve ``0`` ile geri döner. Böylece programcı isterse argümansız uzun seçenekleri ``switch``
içerisinde işlemeden doğrudan onun bayrağına set işlemi yapabilir. Ayrıca programlarda kısa seçeneklerin uzun seçenek
eşdeğerleri de bulunabilmektedir. Bunu sağlamanın en kolay yolu uzun seçeneğe ilişkin ``struct option`` yapısındaki
``val`` elemanına kısa seçeneğe ilişkin karakter kodunu girmektir.

Bu örnekteki seçenekler şöyledir:

- ``-a``: Argümansız kısa seçenek
- ``-b <arg>`` ya da ``--length <arg>``: Kısa ve uzun seçenek eşdeğeri; ``val`` elemanına ``'l'`` yazılmıştır
- ``--all``: Argümansız uzun seçenek; bayrak doğrudan ``all_flag`` değişkenine set edilmektedir
- ``--number[=<arg>]``: İsteğe bağlı argümanlı uzun seçenek

.. code-block:: c

    #include <stdio.h>
    #include <stdlib.h>
    #include <getopt.h>

    int main(int argc, char *argv[])
    {
        int result;
        int a_flag, b_flag, all_flag, length_flag, number_flag, err_flag;
        char *b_arg, *length_arg, *number_arg;

        struct option options[] = {
            {"all",    no_argument,       &all_flag, 1},
            {"length", required_argument, NULL,      'l'},
            {"number", optional_argument, NULL,       3},
            {0, 0, 0, 0},
        };

        a_flag = b_flag = all_flag = length_flag = number_flag = err_flag = 0;
        opterr = 0;
        while ((result = getopt_long(argc, argv, "ab:l:", options, NULL)) != -1) {
            switch (result) {
                case 'b':
                    b_flag = 1;
                    b_arg = optarg;
                    break;
                case 1:
                    all_flag = 1;
                    break;
                case 'l':
                    length_flag = 1;
                    length_arg = optarg;
                    break;
                case 3:
                    number_flag = 1;
                    number_arg = optarg;
                    break;
                case '?':
                    if (optopt == 'b')
                        fprintf(stderr, "-b option without argument!\n");
                    else if (optopt == 2)
                        fprintf(stderr, "--length option without argument!\n");
                    else if (optopt != 0)
                        fprintf(stderr, "invalid option: -%c\n", optopt);
                    else
                        fprintf(stderr, "invalid long option!\n");
                    err_flag = 1;
            }
        }

        if (err_flag)
            exit(EXIT_FAILURE);

        if (a_flag)
            printf("-a option given\n");
        if (b_flag)
            printf("-b option given with argument \"%s\"\n", b_arg);
        if (all_flag)
            printf("--all option given\n");
        if (length_flag)
            printf("--length option given with argument \"%s\"\n", length_arg);
        if (number_flag) {
            if (number_arg != NULL)
                printf("--number option given with argument \"%s\"\n", number_arg);
            else
                printf("--number option given without argument\n");
        }

        if (optind != argc)
            printf("Arguments without options:\n");
        for (int i = optind; i < argc; ++i)
            puts(argv[i]);

        return 0;
    }

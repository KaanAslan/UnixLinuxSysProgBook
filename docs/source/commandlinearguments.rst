===============================================
UNIX/Linux Dünyasında Komut Satırı Argümanları
===============================================



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

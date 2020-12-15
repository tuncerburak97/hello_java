# Fibabanka Java Bootcamp - 2. Hafta

## Exception handling
Exception runtime da yani uygulama çalışırken meydana gelen hatalardır. Bu hataların bir kısmı tolere edilebilirken bir kısmı 
ise uygulamanın tamamen durmasına neden olur. Developer olarak bizim amacımız bu hataları yakalamak ve mümkünse tolere edip 
uygulamanın çalışmaya devam etmesini sağlamaktır, tabi hatanın meydana gelmesini önlemek çok daha öncelikli hedefimiz.

Java bizlere kullanılması kolay ve esnek bir hata yakalama mekanizması sunuyor. Detaylarına geçmeden önce hata hierarşisini
anlamak önemli.

Javada tüm hatalar sınıflar tarafından temsil edilir, yani bir hata oluştuğunda bir hata objesi oluşturulur. Java dilinde
tüm hataların atası **Throwable** sınıfıdır. Throwable sınıfının iki tane doğrudan alt sınıfı bulunur, **Exception** ve **Error**.
Error sınıfı doğrudan bizim kodumuzla ilgili olmayan JVM ile ilgili hatalar için kullanılır ve bu hatalar kontrolümüz dışındadır.
O nedenle bunlardan çok bahsetmeyeceğiz. Exception sınıfı ise doğrudan yazdığımız kodla ilgili bizim kontrolümüzde olan
hatalar için kullanılır.

### Exception handling temelleri
Javada hata yakalama beş anahtar kelime ile yönetilir; **try**, **catch**, **throw**, **throws** ve **finally**.
Hata açısından izlemek istediğimiz kodları try bloğuna yazmalıyız. try bloğu içerisinde bir hata oluşursa bu hata fırlatırlır.
Fırlatılan hatayı catch bloğu yakalar. Sistem tarafından oluşturulan hatalar otomatik olarak fırlatılır fakat bazı durumlarda 
bizim de manuel olarak hata fırlatmamız gerekir, bu durumda throw ifadesi kullanılır. Bazı durumlarda yazdığımız metodun 
hangi hataları fırlatabileceğini metod imzasında belirtmemiz gerekir. Bu durumda throws ifadesi kullanılır. Bir try bloğundan
sonra hata olsun olmasın mutlaka yapılması gereken bir iş varsa o da finally bloğunda yapılır.

try ve catch bloğunun genel yapısı aşağıdaki gibidir. try tek başına kullanılabilir ama catch ifadesi sadece try ile birlikte
kullanılabilir.
```java
try {
    // block of code to monitor for errors
} catch(ExceptionType1 e) {
    // handler for ExceptionType1
} catch(ExceptionType2 e) {
    // handler for ExceptionType2
}
```
try bloğu içerisinde bir hata oluştuğu zaman hatadan sonraki try bloğu içinde kalan kod işletilmez. O noktadan itibaren 
kodun akışı oluşan hata ile en çok uyumlu olan catch bloğuna geçer. O nedenle catch bloğunu oluştururken hata tipi seçimi
önemlidir. catch bloğuna oluşan hatanın objesi verilir ve blok içerisinde hata işlenir.

Eğer hiçbir hata oluşmaz ise kod akışı en son catch ifadesinden sonraki kod parçasıyla devam eder. catch blokları yalnızca 
ilgili hata oluştuğu zaman çalıştırılır.

Kod içerisinde şöyle bir akış olduğunu düşünelim; a() metodu içerisinde bir noktada b() metodunu çağırsın ve b() metodu 
içinde bir noktada c() metodunu çağırsın ve c() metodu çalıştırılırken bir hata alalım. Eğer hata c() metodu içerisinde 
yakalanmıyorsa hata hierarşide bir üst katmana aktarılır yani b() metodunda c() metodunu çağırdığımız noktaya. Eğer o noktada
da hatayı yakalayan bir catch bloğu yoksa hata bir üst katmana yani a() metodu içinde b() metodunun çağrıldığı yere aktarılır.
Eğer bu noktada bu hatayı yakalayacak bir catch bloğu varsa hata orada yakalınır ve işlenir. Yok ise bu hata main metoduna
kadar aktarılır ve uygulamanın durmasına neden olabilir. 

Şimdi bütün bu öğrendiklerimizi bir örnekle kod üzerinde görelim;

```java
public class ArrayOperations {

    public void putValue(int[] array, int index, int value) {
        System.out.println("    putValue method started");
        array[index] = value;
        System.out.println("    putValue method ended successfully");
    }
}

public class Main {

    public static void main(String[] args) {

        ArrayOperations operations = new ArrayOperations();
        int[] numbers = new int[7];
        try {
            System.out.println("try block is started");
            operations.putValue(numbers, 10, 27);
            System.out.println("try block is ended");
        } catch (ArrayIndexOutOfBoundsException ex) {
            System.out.println("Exception catched " + ex.toString());
        }

        System.out.println("Program ends gracefully");
    }
}
```
Output :
```
try block is started
    putValue method started
Exception catched java.lang.ArrayIndexOutOfBoundsException: 10
Program ends gracefully
```

Bu örnek bize hata yakalamanın nasıl işlediğine dair pek çok bilgi veriyor. Örnekte görüldüğü gibi putValue() metodunda 
arraye bir int değer koymaya çalışırken ArrayIndexOutOfBoundsException alıyoruz ve bu hatadan sonraki satır işletilmeden 
hata bu metodun çağırıldığı bir üst katmana iletiliyor, yani main() metoduna. main() metodu içinde putValue() metoduna 
çağrı yaptığımız kod parçası try bloğu içinde yer alıyor. Ve catch bloğunda yakaladığımız hata türü runtime da aldığımız 
hata türüyle eşleştiği için program akışı catch bloğuna geçiyor. catch bloğu içinde hata işleniyor ve buradan yeni bir hata 
fırlatılmadığı için akış catch bloğu sonrasından devam ediyor. 

Bu örnekte hatanın yakalanmadığını düşünelim;

```java
public class Main {

    public static void main(String[] args) {

        ArrayOperations operations = new ArrayOperations();
        int[] numbers = new int[7];

        System.out.println("before calling put value");
        operations.putValue(numbers, 10, 27);
        System.out.println("after calling put value");
    }
}

```
Output :
```
before calling put value
    putValue method started
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 10
	at com.hkarabakla.inheritance.ArrayOperations.putValue(ArrayOperations.java:7)
	at com.hkarabakla.inheritance.Main.main(Main.java:11)

Process finished with exit code 1
```
Bu çıktı bize uygulamayı debug ederken hatanın yerini bulmak için yardımcı olurken başkalarının pek işine yaramayacaktır.
O nedenle hatanın uygun şekilde işlenip uygun hata mesajlarının loga basılması önemli.

Şimdi catch bloğunun bulunması fakat fırlatılan hata ile eşleşmemesi durumunda neler olur ona bakalım;
```java
public class Main {

    public static void main(String[] args) {

        ArrayOperations operations = new ArrayOperations();
        int[] numbers = new int[7];
        try {
            System.out.println("try block is started");
            operations.putValue(numbers, 10, 27);
            System.out.println("try block is ended");
        } catch (ArithmeticException ex) {
            System.out.println("ArithmeticException catched " + ex.toString());
        }

        System.out.println("Program ends gracefully");
    }
}
```
Output : 
```
try block is started
    putValue method started
Exception in thread "main" java.lang.ArrayIndexOutOfBoundsException: 10
	at com.hkarabakla.inheritance.ArrayOperations.putValue(ArrayOperations.java:7)
	at com.hkarabakla.inheritance.Main.main(Main.java:11)

Process finished with exit code 1
```
Sonuç aynı, doğru hata türünü catch bloğunda kullanmak o nedenle önemli.

Hatalar uygun bir şekilde yakalanırsa uygulamanın çalışmaya devam edeceğini söylemiştik, şimdi bunu bir [örnekle](../../examples/src/com/hkarabakla/exception/ExceptionDemo3.java) görelim;
```java
public class ExceptionDemo3 {

    public static void main(String[] args) {
        int[] numbers = new int[] {1, 34, 56, 23, 78};
        int[] dividers = new int[] {5, 0, 12, 0, 34};

        for (int i = 0; i < numbers.length; i++) {
            try {
                System.out.println(numbers[i] + "/" + dividers[i] + "=" + numbers[i]/dividers[i]);
            } catch (ArithmeticException ex) {
                System.out.println("Division by 0, " + numbers[i] + "/" + dividers[i]);
            }
        }

        System.out.println("Program ended successfully");
    }
}
```
Output :
```
1/5=0
Division by 0, 34/0
56/12=4
Division by 0, 23/0
78/34=2
Program ended successfully
```

### Exception yakalamada super class - subclass ilişkisi
Bir try ifadesinin birden fazla catch bloğu ile ilişkilendirilebileceğini söylemiştik. Böyle bir durumda aralarında üst 
sınıf alt sınıf ilişkisi bulunan hatalardan önce alt sınıf hatayı sonra üst sınıf hatayı catch blokları ile yakalamalıyız.
Tersi durumda compiler hata verecektir. Şimdi bunu bir [örnekle](../../examples/src/com/hkarabakla/exception/ExceptionDemo4.java) görelim;
```java
public class ExceptionDemo4 {

    public static void main(String[] args) {

        int[] numbers = new int[] {1, 34, 56, 23, 78, 123, 49};
        int[] dividers = new int[] {5, 0, 12, 0, 34};

        for (int i = 0; i < numbers.length; i++) {
            try {
                System.out.println(numbers[i] + "/" + dividers[i] + "=" + numbers[i]/dividers[i]);
            } catch (ArithmeticException ex) {
                System.out.println("Division by 0, " + numbers[i] + "/" + dividers[i]);
            } catch (RuntimeException ex) {
                System.out.println("Another exception catched : " + ex);
            }
        }

        System.out.println("Program ended successfully");
    }
}
```
Output :
```
1/5=0
Division by 0, 34/0
56/12=4
Division by 0, 23/0
78/34=2
Another exception catched : java.lang.ArrayIndexOutOfBoundsException: 5
Another exception catched : java.lang.ArrayIndexOutOfBoundsException: 6
Program ended successfully
```
### throw ile hata fırlatma
Daha önce fırlatılan hataların nasıl yakalanacağını gördük. Bazı durumlarda kod içerisinden manuel olarak bizim de hata fırlatmamız
gerekebilir. Bu durumda bir hata objesi oluşturmamız ve onu throw ifadesi ile fırlatmamız gerekir. 
Yada catch bloğu içinde yakaladığımız bir hatayı işledikten sonra onu bir üst katmana tekrar fırlatmamız gerekebilir. 
Bu durumda tekrar hata objesi oluşturmamız gerekmez catch bloğuna gelen hatayı aynen fırlatabiliriz. Şimdi bunu 
bir [örnekle](../../examples/src/com/hkarabakla/exception/ExceptionDemo5.java) görelim;

```java
public class ExceptionDemo5 {

    public static void main(String[] args) {
        try {
            multipleDivision();
        } catch (RuntimeException ex) {
            System.out.println("Fatal error, program terminated");
        }
    }

    private static void multipleDivision() {
        int[] numbers = new int[] {1, 34, 56, 23, 78, 123, 49};
        int[] dividers = new int[] {5, 0, 12, 0, 34};

        for (int i = 0; i < numbers.length; i++) {
            try {
                System.out.println(numbers[i] + "/" + dividers[i] + "=" + numbers[i]/dividers[i]);
            } catch (ArithmeticException ex) {
                System.out.println("Division by 0, " + numbers[i] + "/" + dividers[i]);
            } catch (ArrayIndexOutOfBoundsException ex) {
                System.out.println("Array sizes are not equal");
                throw new ArrayIndexOutOfBoundsException("Array sizes must be equal");
            }
        }

        System.out.println("Program ended successfully");
    }
}
```
Output :
```
1/5=0
Division by 0, 34/0
56/12=4
Division by 0, 23/0
78/34=2
Array sizes are not equal
Fatal error, program terminated
```

### finally ifadesi
Bazı durumlarda try-catch ifadesini terk etmeden hemen önce çalıştırmamız gerekn kodlar olabilir. Bu durumlar genelde 
try bloğu içinde bir dosya açtıysak onu kapatmak için yada bir network bağlantısı kurduysak o bağlantıyı kapatmak için
kullanılan kodlardır. Bu tarz durumlar programlamada oldukça yaygındır ve java bize bu durumu yönetmek için finally ifadesini
sunuyor. Genel yapısı şu şekildedir;

```java
try {
    // code to be monitored for error
} catch(ExceptionType1 ex) {
    // handling ExceptionType1
} catch(ExceptionType2 ex) {
    // handling ExceptionType2
} finally {
    // finally code to clean up all we created in try
}
```
Bu yapıda finally bloğu catch bloğundan sonra gösterilmiştir ama teoride finally bloğu doğrudan try bloğunun ardından da
gelebilir, bu durumda exception yakalanmaz. try bloğu içinden fırlatılan hata daha üst katmanlarda yakalanmaz ise uygulama 
son bulur. Bu durum teoride mümkün olsa da pratikte hiç tercih edilmez.

Şimdi finally bloğunu bir [örnekle](../../examples/src/com/hkarabakla/exception/ExceptionDemo6.java) görelim;

```java
public class ExceptionDemo6 {

    public static void main(String[] args) {
        FileInputStream input = null;
        try {
            ClassLoader classLoader = ExceptionDemo6.class.getClassLoader();
            input = new FileInputStream(classLoader.getResource("file.txt").getFile());

            int data = input.read();
            while (data != -1) {
                System.out.print((char) data);
                data = input.read();
            }
        } catch (IOException e) {
            System.out.println("Exception occurred while reading file");
            e.printStackTrace();
        } finally {
            if (input != null) {
                try {
                    input.close();
                } catch (IOException e) {
                    System.out.println("Exception occurred while closing file");
                }
            }
        }
    }
}
```

### throws ifadesi
Bazı durumlarda bir metod bir yada birkaç hata yarattığı halde bu hataları kendi içinde yakalamak yerine hata yakalama işini
kendisini çağıran metodlara bırakır. Bu durumda yakalanmayan hatalar metod imzasına throws ifadesi ile eklenir. Genel yapı şu
şekildedir;

```java
return_type methodName(parameter_list) throws exception_list {
}
```
Burada exception_list virgül ile ayrılmış hata listesini içerir.
Şimdi bir önceki örneği biraz değiştirerek throws ifadesinin nasıl kullanıldığını bir [örnekle](../../examples/src/com/hkarabakla/exception/ExceptionDemo7.java) görelim;

```java
import java.io.FileInputStream;
import java.io.IOException;

public class ExceptionDemo7 {

    public static void main(String[] args) throws IOException {
        FileInputStream input = null;
        try {
            ClassLoader classLoader = ExceptionDemo7.class.getClassLoader();
            input = new FileInputStream(classLoader.getResource("file.txt").getFile());

            int data = input.read();
            while (data != -1) {
                System.out.print((char) data);
                data = input.read();
            }
        } finally {
            if (input != null) {
                try {
                    input.close();
                } catch (IOException e) {
                    System.out.println("Exception occurred while closing file");
                }
            }
        }
    }
}
```
catch bloğunun ortadan kalktığına ve throws IOException ifadesinin metod imzasına eklendiğine dikkat edelim.

### try-with-resource
Önceki örneklerde gördüğümüz gibi eğer try bloğu içeerisinde bir dosya yada bir network bağlantısı açıyorsak o kaynağın
finally bloğu içinde kapatılması gerekiyor. Java 7 ile birlikte bu tarz durumlar için try-catch-finally blokları yeerine
try-with-resource kavramı geldi. try-with-resource içerisinde açılan kaynaklar try bloğu sonrasında otomatik olarak kapatılır.
Bu kaynakların otomatik kapatılabilmeleri için AutoClosable interface ini implemente etmeleri gerekir.

Daha önceki örneklerdee gördüğümüz dosya okuma işlemini şimdi try-with-resource ile [tekrar](../../examples/src/com/hkarabakla/exception/ExceptionDemo8.java) yapalım;

```java
import java.io.FileInputStream;
import java.io.IOException;

public class ExceptionDemo8 {

    public static void main(String[] args) {
        ClassLoader classLoader = ExceptionDemo8.class.getClassLoader();

        try (FileInputStream input = new FileInputStream(classLoader.getResource("file.txt").getFile())) {

            int data = input.read();
            while (data != -1) {
                System.out.print((char) data);
                data = input.read();
            }
        } catch (IOException e) {
            System.out.println("Exception occurred while reading file");
            e.printStackTrace();
        }
    }
}
```
Görüldüğü gibi aynı işi yapan kod çok daha kısa ve sade.

Eğer birden fazla kaynağa erişmemiz gerekirse try bloğu içinde bu drumda aşağıdaki örnekte gösterildiği gibi eerişilebilir.

## Multithreaded programlama

TODO

## Generics (Jenaerikler)
Java dilinde pek çok özellik çoğunlukla 1.0 versiyonunda eklenmiştir. Eklenen diğer tüm özellikler dilin kapsamını genişletmiştir 
ki bunlardan biri olan Jenerikler dili en çok şekillendirenlerdendir.

Jenerikler temelde parametrik tip demektir. Yani parametrik bir sınıf, interface yada metod yaratırken aslında o kod parçasının
hangi tip data üzerinde işlem yapacağını bildiriyoruz. Bu ne demek ? Biraz daha açalım konuyu.

Diyelim ki bir sıralama algoritması yazdınız ve bu algoritmanın int, String, Thread, Object gibi farklı tipler üzerinde çalışmasını 
istiyorsunuz. Jenerikler javaya eklenmeden önce bunu yapmanın iki yolu vardı, ya algoritmayı kullandığımız her obje tipi
için yeniden yazmak, ki çok miktarda kod tekrarı demektir yada algoritmayı Object sınıfını baz alarak yazmaktı. Yani yaratacağınız sınıf, metodlar hepsi Object 
tipinde input alacak ve Object tipinde değer dönecek. Ancak bu şekilde genel bir algoritma implementasyonu mümkün oluyordu.
Tabi çağırdığımız her metoddan sonra bize geri dönen Object tipindeki değeri de istediğimiz tipe cast etmemiz gerekiyordu.
Bu beraberinde hem performans kaybı hemde runtime da hatalar almamıza neden oluyordu.

İşte bu sorunlara çözüm olarak Java dilini tasarlayanlar Jenerikleri dile kazandırdı. Jenerikler sayesinde algoritmalarda 
veri tipinden bağımsız genelleme, kodun tekrar kullanılabilirliği ve güvenlik arttı.

Şimdi basit bir jenerik implementasyonu görelim :

```java
public class GenericType<T> {

    private T obj;

    public GenericType(T obj) {
        this.obj = obj;
    }

    public T getObj() {
        return obj;
    }

    void showType() {
        System.out.println("Type of the class is " + this.obj.getClass().getName());
    }
}

public class Main {
    public static void main(String[] args) {

        GenericType<Integer> integerGenericType = new GenericType<>(7);
        integerGenericType.showType();
        Integer value1 = integerGenericType.getObj();
        System.out.println("Value of generic obj " + value1));

        GenericType<String> stringGenericType = new GenericType<>("--Generics--");
        stringGenericType.showType();
        String value2 = stringGenericType.getObj();
        System.out.println("Value of generic type " + value2);
    }
}
```    
Output :
```
Type of the class is java.lang.Integer
Value of generic obj 7
Type of the class is java.lang.String
Value of generic type --Generics--
```

Bu örnekte dikkat etmemiz gereken bir kaç nokta var. Birincisi jenerik sınıfımıza parametre tipini nasıl geçtiğimiz. Sınıf
isminden hemen sonra gelen <> sembolleri arasına tip ismini veriyoruz. Burada tek harflik bir isim seçmek ve T, V, E gibi
çok kullanılan tip isimlerini seçmek iyi olacaktır. Daha sonra bu tip ismini sınıf içindefarklı yerlerde kullanacağız.

Aynı T tipinde bir değişken tuttuğumuza ve sınıfın constructor unda bu değişkeni initialize ettiğimize dikkat edelim.

Daha sonra bu değişkene erişebilmek için bir getter metodu ekledik, bu metod daha sonra yarattığımız jenerik değerine 
ulaşmak için kullanılacak.

Jenerik sınıfın sonunda da bu parametre tipinin gerçek ismini ekrana basan bir metod ekledik.

Bu jenerik sınıftan obje yaratmak için Integer ve String tip parametrelerini ve bu tiplerden değerler kullandık. Bu jenerik 
sınaftan türettiğimiz objelerin değerlerine ulaşmak için getObj metodunu çağırdık ve type casting yapmadık.
 
Jenerik sınıflar birden fazla tip parametresi ile çalışabilir :

```java
public class KeyValuePair<K, V> {
    private K key;
    private V value;

    public KeyValuePair(K key, V value) {
        this.key = key;
        this.value = value;
    }

    public K getKey() {
        return key;
    }

    public V getValue() {
        return value;
    }

    void displayInfo() {
        System.out.println("Type of K " + key.getClass().getName());
        System.out.println("Type of V " + value.getClass().getName());
    }
}

public class Main {
    public static void main(String[] args) {

        KeyValuePair<String, String> strKeyValuePair = new KeyValuePair<>("name", "Java");
        strKeyValuePair.displayInfo();

        KeyValuePair<String, Integer> mixKeyValuePair = new KeyValuePair<>("age", 25);
        mixKeyValuePair.displayInfo();
    }
}
```
Output :
```
Type of K java.lang.String
Type of V java.lang.String
Type of K java.lang.String
Type of V java.lang.Integer
```

### Sınırlandırılmış jenerikler
Daha önceki örneklerde herhangi bir referans tipin bir jenerik sınıf için tip parametresi olabildiğini gördük. Bazı durumlarda
bu bir avantaj olurken bazen de sadece belli tiplerde tip parametresi kabul etmek isteriz. Örneğin matematiksel işlemler yapan 
bir jenerik sınıf yaratacaksak sadece nümerik değerlerin parametre olmasını isteriz.

```java
public class NumericOperation<T extends Number> {
    private T obj;

    public NumericOperation(T obj) {
        this.obj = obj;
    }

    public T getObj() {
        return obj;
    }

    boolean isDividableByTen() {
        return obj.doubleValue() - obj.intValue() == 0 && obj.intValue() % 10 == 0;
    }
}

public class Main {
    public static void main(String[] args) {

        NumericOperation<Integer> intValue = new NumericOperation<>(23);
        System.out.println("Is " + intValue.getObj() + " dividable by ten : " + intValue.isDividableByTen());

        NumericOperation<Double> doubleValue = new NumericOperation<>(23.12);
        System.out.println("Is " + doubleValue.getObj() + " dividable by ten : " + doubleValue.isDividableByTen());

        NumericOperation<Double> doubleValue2 = new NumericOperation<>(30.0);
        System.out.println("Is " + doubleValue2.getObj() + " dividable by ten : " + doubleValue2.isDividableByTen());

        NumericOperation<String> stringNumericOperation = new NumericOperation<String>("invalid parameter type"); // HATA
    }
}
```
Output :
```
Is 23 dividable by ten : false
Is 23.12 dividable by ten : false
Is 30.0 dividable by ten : true
```

### Wildcard argüman
Daha önceki konuda gördüğümüz tek tipte tip argümanı kullanmak type safety için oldukça faydalı. Fakat özellik bizi tek tipte
veri üzerinde işlem yapmaya zorluyor. Bazen farklı veri tipleri üzerinde işlem yapmak da gerekebilir, örneğin bir double değer
ile bir float değeri karşılaştırmak gibi. Bu gibi durumlarda wildcard **?** ifadesini kullanabiliriz.

Daha önce gördüğümüz NumericOperation sınıfına iki sayısal değişkenin mutlak değerlerini karşılaştıran ve sonucunu dönen
bir metod ekleyelim. Bu metod tüm Number sınıfından türeyen değerleri karşılaştırabilsin.

```java
public class NumericOperation<T extends Number> {
    private T obj;

    public NumericOperation(T obj) {
        this.obj = obj;
    }

    public T getObj() {
        return obj;
    }

    boolean isDividableByTen() {
        return obj.doubleValue() - obj.intValue() == 0 && obj.intValue() % 10 == 0;
    }

    boolean absEqual(NumericOperation<?> another) {
        return Math.abs(this.obj.doubleValue()) == Math.abs(another.obj.doubleValue());
    }
}

public class Main {
    public static void main(String[] args) {

        NumericOperation<Double> doubleNumeric = new NumericOperation<>(2.5);
        NumericOperation<Float> floatNumeric = new NumericOperation<>(-2.5f);

        System.out.println(doubleNumeric.absEqual(floatNumeric));
    }
}
```
Bu örnekte görüldüğü gibi wildcard ifadesini metod argümanı olarak kullanabildiğimiz gibi sınıf seviyesinde sınırlandırılmış
jenerik yapımında da kullanabiliriz. Bunun daha önce yaptığımız sınırlandırılmış jenerik kullanımından hiç bir farkı yok, T yerine 
wildcard ifadesini kullanıyoruz.

```java
access_modifier class class_name<? extends super_class> {
    
}
```
Böylece jenerik sınıfımızda süper sınıftan türemiş tüm veri tipleri üzerinde işlem yapabiliriz.

### Jenerik metodlar
Jenerik tip parametresinin sınıflara nasıl uygulandığını görmüştük. Bir sınıf jenerik parametre alırsa otomatikman sınıf içinde 
yeralan tüm metodlar bu jenerik tip üzerinde işlem yapabilme imkanına sahip oluyor. Bazen jenerik tipleri sadece belli 
metodlar üzerinde kullanmak gerekebilir. Bunun için metodun bulunduğu sınıfın tamamen jenerik olması gerekmez.

Örneğin jenerik olmayan bir sınıfta jenerik bir metodun nasıl yer alabileceğine bir bakalım. Bunun için iki arrayi karşılaştıran
bir örnek yapalım;

```java
public class GenericMethodDemo {

    public <T extends Comparable<T>, V extends T> boolean arraysEqual(T[] arr1, V[] arr2) {
        if (arr1.length != arr2.length) {
            return false;
        }

        for (int i = 0; i < arr1.length; i++) {
            if (!arr1[i].equals(arr2[i])) {
                return false;
            }
        }

        return true;
    }
}

public class Main {
    public static void main(String[] args) {

        GenericMethodDemo genericMethodDemo = new GenericMethodDemo();

        Integer[] integers1 = new Integer[]{1, 2, 3, 4, 5};
        Integer[] integers2 = new Integer[]{1, 2, 3, 4, 5};
        Integer[] integers3 = new Integer[]{1, 2, 3, 4, 5, 7};

        Double[] doubles = new Double[]{1.0, 2.0, 3.0, 4.0, 5.0};

        System.out.println("integers1 is equal to integers1 : " + genericMethodDemo.arraysEqual(integers1, integers1));
        System.out.println("integers1 is equal to integers2 : " + genericMethodDemo.arraysEqual(integers1, integers2));
        System.out.println("integers1 is equal to integers3 : " + genericMethodDemo.arraysEqual(integers1, integers3));

        //System.out.println("integers1 is equal to doubles : " + genericMethodDemo.arraysEqual(integers1, doubles));
    }
}
```
Output :
```
integers1 is equal to integers1 : true
integers1 is equal to integers2 : true
integers1 is equal to integers3 : false
```

Bu örnekte jenerik parametre tanımının metodun dönüş tipinden önce yapıldığına dikkat edelim. Jenerik metodlarda metod imzasındaki
bu sıralama zorunludur. Yani jenerik metod tanımını erişim belirleyiciden önce yada metodun dönüş tipinden sonra yapamayız.

<T extends Comparable<T>, V extends T> bu tanımda ilk tipimizin(T) Comparable interface nden türemesi gerektiğini ve ikinci tipin
(V) birinci tipten türemesi ya o tip ile aynı olması gerektiği belirttik. Bu durumda örneğin bir Integer arrayini bir Double
array i ile karşılaştıramayız. Bu nedenle main metodu içinde yer alan son satırı yorum satırına çevirdik, aksi durumda compiler
hata verecektir.

### Jenerik interface
Jeneriklerin sınıflara ve metodlara uygulanabilir olduğunu gördük. Şimdi sıra geldi interfacelere. Jenerikliğin uygulanışı 
bakımından sınıf ve interface in hiçbir farkı yoktur. 

```java
public interface ContainChecker<T> {
    boolean contains(T obj);
}

public class GenericInterfaceDemo<T> implements ContainChecker<T> {

    T[] values;

    public GenericInterfaceDemo(T[] values) {
        this.values = values;
    }

    public boolean contains(T obj) {
        for (T value : values) {
            if (value.equals(obj)) {
                return true;
            }
        }
        return false;
    }
}

public class Main {
    public static void main(String[] args) {
        Integer[] integers = new Integer[]{1, 2, 3, 6, 7, 9, 10};
        GenericInterfaceDemo<Integer> interfaceDemo = new GenericInterfaceDemo<>(integers);
        System.out.println("Contains 5 : " + interfaceDemo.contains(5));
        System.out.println("Contains 7 : " + interfaceDemo.contains(7));
    }
}
``` 
Output :
```
Contains 5 : false
Contains 7 : true
```

Jenerik interfacelerde dikkat edilmesi gerek birkaç nokta var.

Birincisi jenerik interface i implemente eden sınıf belli bir tip kullanmadığı sürece jenerik olmalıdır ve tip parametresini
aynen interface e de geçmelidir.
```java
class GenericInterfaceDemo<T> implements ContainChecker<T> // Doğru
class GenericInterfaceDemo implements ContainChecker<T> // Hata
class GenericInterfaceDemo implements ContainChecker<Double> // Doğruu
```

Bir diğer önemli nokta sınırlamadır, eğer jenerik bir interface tip parametresinde sınırlamaya gitmiş ise onu implemente eden 
sınıf da gitmek zorundadır.
```java
interface ContainChecker<T extends Number>
class GenericInterfaceDemo<T extends Number> implements ContainChecker<T> // Doğru
class GenericInterfaceDemo<T extends Number> implements ContainChecker<T extends Number> // Hata
```

## Lambda ifadeleri ve fonksiyonel interface
Daha önce interface tanımlamayı ve interfacelerin sınıflar tarafından nasıl implemente edildiğini gördük.
Şimdi interface metodlarını implemente etmek için başka bir yol öğreneceğiz, Lambda ifadeleri.
Lambda ifadeleri Java 8 ile birlikte geldi ve tıpkı jenerikler gibi Java dilini yeniden şekillendirdi.

Lambda ifadesi denilince akla iki şey gelmeli; birincisi lambda fonksiyonunun kendisi ikincisi ise fonksiyonel interface
kavramı.

Lambda ifadelerini tanımlamak için arrow operatörünü **->** kullanıyoruz. Arrow operatörünün sol tarafında lambda ifadesinin 
parametre listesi sağ tarafında ise implementasyonu yer alır.  
```java
() -> 3.4
```
Yukardaki ifade aşağıdaki ifadeye dengtir
```java
boolean getValue() {
    return 3.4;
}
```

Lambda ifadeleri tek başlarına bir anlam ifade etmezler. Çünkü anonim metodlardır, yani metod ismi ve ait oldukları bir sınıfları yoktur.
Sadece özünde bir anonim metod implementasyonudur. O nedenle lambda ifadeleri bir fonksiyonel interface ile ilişkilendirilmelidir.

Fonksiyonel interface özünde sadece bir abstract metod bulunduran interface lere denir. Bu interfaceler sadece bir işe odaklanmıştır 
o nedenle sadece bir abstract metod içerirler. Bu abstract metodun yanında private yada default yada static metodlarda
içerebilirler fakat önemli olan sadece tek bir abstract metodunun olmasıdır. 

```java
interface MyValue {
    double getValue();
}
```

Lambda ifadelerini aşağıdaki şekilde gösterildiği gibi fonksiyonel interfaceler ile aşağıdaki gibi ilişkilendiriyoruz.

```java
interface MyValue {
    double getValue();
}

MyValue = () -> 3.4;
```
Burada önemli olan lambda ifadesinin ve abstract metodun ımzalarının tamamen birbirine uymasıdır. Aksi durumda compiler
hata verir.

Fonksiyonel interfacelerin önemli özelliklerinden biri de içerdikleri abstract metodun kendisiyle uyumlu birden fazla 
lambda metodu ile ilişkilendirilebiliyor olmasıdır. Bunu daha iyi anlamak için aşağıdaki örneğe bakalım;

```java
public interface NumericTest {

    boolean test(int a, int b);
}

public class Main {
    public static void main(String[] args) {

        NumericTest isFactor = (m, n) -> m % n == 0;

        System.out.println("5 and 3 is factor " + isFactor.test(5, 3));
        System.out.println("27 and 3 is factor " + isFactor.test(27, 3));

        NumericTest lessThan = (x, y) -> x < y;
        System.out.println("5 is less than 3 " + lessThan.test(5, 3));
        System.out.println("11 is less than 17 " + lessThan.test(11, 17));

        NumericTest absEqual = (x, y) -> (x < 0 ? -x : x) == (y < 0 ? -y : y);
        System.out.println("absolute value of -9 is equal to -4 " + absEqual.test(-9, -4));
        System.out.println("absolute value of 17 is equal to -17 " + absEqual.test(17, -17));
    }
}
```
Output :
```
5 and 3 is factor false
27 and 3 is factor true
5 is less than 3 false
11 is less than 17 true
absolute value of -9 is equal to -4 false
absolute value of 17 is equal to -17 true
```

Buraya kadarki örneklerde tek bir işlemden oluşan lambda ifadelerini gördük. Lambda ifadeleri herzaman bukadar basit olmayabilir
ve birden fazla işlem içerebilir. Bu drumda block lambda dediğimiz ifadeleri kullanırız.

```java
public interface NumericFunc {
    int func(int a);
}

public class Main {
    public static void main(String[] args) {

        NumericFunc biggestFactorBesidesItSelf = (x) -> {
            int result = 1;

            x = x < 0 ? -x : x;

            for (int i = 2; i < x/2; i++) {
                if (x % i == 0) {
                    result = i;
                }
            }

            return result;
        };

        System.out.println("Biggest positive factor of 20 is " + biggestFactorBesidesItSelf.func(20));
        System.out.println("Biggest positive factor of 37 is " + biggestFactorBesidesItSelf.func(37));
    }
}
```
Output :
```
Biggest positive factor of 20 is 5
Biggest positive factor of 37 is 1
```

Burada block içerisinde normal bir sınıf metodu gibi yapıyoruz fakat sonucu return etmemiz gerekiyor, önceki örneklerde bu 
return işlemi JVM tarafından otomatik olarak yapılıyordu.

Lambda ifadelerin bir diğer güzel özelliği ise metodlara argüman olarak gönderilebiliyor olmalarıdır. Yani lambda ifadeleri
yardımıyla bir metodu farklı implemente edilmiş lambda ifadeleri ile çağırmak mümkün.

```java
public interface StringFunc {
    String func(String str);
}

public class Main {

    static String changeStr(StringFunc stringChanger, String value) {
        return stringChanger.func(value);
    }

    public static void main(String[] args) {

        StringFunc reverse = (str -> {
            String result = "";

            for (int i = str.length() - 1; i >= 0; i--) {
                result += str.charAt(i);
            }
            return result;
        });

        System.out.println("Reverse of 'madam' is " + changeStr(reverse, "madam"));

        System.out.println("Cleaned word of '1,45,23,78,99' is " + changeStr(str -> str.replaceAll(",", " "), "1,45,23,78,99"));

    }
}
```
Output :
```
Reverse of 'madam' is madam
Cleaned word of '1,45,23,78,99' is 1 45 23 78 99
```

Örnekte hem bir referans değere atanmış lambda ifadenin hemde embedded bir lambda ifadenin nasıl metodlara parametre 
olarak gönderildiğini gördük. Bu kullanım şekliyle çok daha kompleks implementasyonlar yapmak mümkün.

### Jenerik lambda ifadeleri
Daha önceki örneklerde bir fonksiyonel interface içerisinde iki tane integer input alıp boolean değer dönen bir abstract metodu
lambda ifadeleri ile implemente etmiştik. 

```java
public interface NumericTest {

    boolean test(int a, int b);
}
public class Main {
    public static void main(String[] args) {
        NumericTest lessThan = (x, y) -> x < y;
        System.out.println("5 is less than 3 " + lessThan.test(5, 3));
        System.out.println("11 is less than 17 " + lessThan.test(11, 17));
    }
}
```

Bu implementasyonun aynısının birde double değerleri karşılaştırmak için yapılması gerektiğini düşünelim. Bu drumda 
aynı fonskiyonel interfaceden bir tane daha yaratmamız gerekecekti, fakat özünde implementasyon veri tipinden bağımsız 
olarak aynı olacaktı. İşte bu kod tekrarını önlemek için jeneriklerin kullanılabileceğini daha önce öğrenmiştik. 
Lambda ifadeleri doğaları gereği, bir sınıfa ait olmadıkları için, jenerik olamazlar fakat fonksiyonel interfaceler jenerik
olabilir.

```java
public interface NumericTest<T> {

    boolean test(T a, T b);
}

public class Main {
    public static void main(String[] args) {

        NumericTest<Integer> lessThan = (x, y) -> x < y;
        System.out.println("5 is less than 3 " + lessThan.test(5, 3));
        System.out.println("11 is less than 17 " + lessThan.test(11, 17));

        NumericTest<Double> lessThanDouble = (x, y) -> x < y;
        System.out.println("5.4 is less than 3.2 " + lessThanDouble.test(5.4, 3.2));
        System.out.println("11.9 is less than 17.1 " + lessThanDouble.test(11.9, 17.1));
    }
}
```
Output :
```
5 is less than 3 false
11 is less than 17 true
5.4 is less than 3.2 false
11.9 is less than 17.1 true
```

### Lambda ifadeler ve variable capture
Lambda ifadelerin kendi içinde birer metod olduklarını ve normal metodlar gibi kendi içinde local değişken 
tanımlayabildiklerini gördük. Lambda ifadeler içinde tanımlanan local değişkenler sadece lambda bloğu içinde erişilebilirdir,
yani lambda dışından erişilemez.

Peki tam tersi mümkün müdür ? Evet lambda ifadeler yaratıldıkları sınıfın instance değişkenlerine, static değişkenlerine ve 
metodlarına erişebilirler. Burada instance değişkenlerine ve static değişkenlerine değer de atayabilirler (özellikle final 
olmadıkları sürece). Ayrıca tanımlandıkları sınıfa da **this** ifadesi ile erişebilirler.

Ayrıca lambda ifadeler yaratıldıkları kod bloğu içinde tanımlanan local değişkenlere de sınırlı olarak erişebilirler, sadece
değerini kullanabilir yeni değer atayamazlar. Bu olaya **variable capture** denir. Bu durumda local değişkenin **final** yada **effectively 
final** olması gerekir. Effectively final demek bir değişkenin değerinin ilk atamadan sonra değiştirilememesi demektir, 
özellikle final olarak belirtilmemesine rağmen.

```java
public interface NumericFunc {
    int func(int a);
}

public class Main {

    static int value;

    public static void main(String[] args) {

        value = 10;

        NumericFunc func = (a) -> {
            value = 20;
            return value * a;
        };

        System.out.println("Value before calling lambda " + value);
        int valueFromFunc = func.func(3);
        System.out.println("Value after calling lambda " + value);
    }
}
```
Output :
```
Value before calling lambda 10
Value after calling lambda 20
```

Bu örnekte lambda ifadelerin static sınıf değişkenlerini nasıl değiştirebildiğini ve kullanabildiğini gördük.

```java
public interface NumericFunc {
    int func(int a);
}

public class Main {

    public static void main(String[] args) {

        int value = 10;

        NumericFunc func = (a) -> {
            //value = 20;
            return value * a;
        };
        
        //value = 20;
    }
}
```
Bu comment out edilen iki satıra dikkat edelim. Lambda tarafından erişilen dış dünyanın local değişkenleri effectively
final demiştik. Yani bu drumda ilk yorum satırını açtığımız zaman compiler bize hata verecektir, çünkü effectively final
olan local değişkenler lambda ifadeler tarafından değiştirilemez. Fakat lambda ifade tarafından değerlerine erişilebilir.
Bu nedenle de ikinci yorum satırını comment out etmek zorunda kaldık, lambda tarafından erişilen local değişkenler lambda
dışında dahi değiştirilemez, çünkü effectively final.

Peki illa böyle birşey yapmamız gerekirse bu hatalardan kurtulmanın bir yolu yok mu ? Neyseki var, örneği inceleyelim;

```java
public class Main {

    public static void main(String[] args) {

        final int[] value = {10};
        AtomicReference<String> strValue = new AtomicReference<>("sasa");

        NumericFunc func = (a) -> {
            value[0] = 20;
            strValue.set("");
            return value[0] * a;
        };

        System.out.println("Value before calling lambda " + value[0]);
        int valueFromFunc = func.func(3);
        System.out.println("Value after calling lambda " + value[0]);
        value[0] = 30;
        System.out.println("Value in the end " + value[0]);
    }
}
```
Output :
```
Value before calling lambda 10
Value after calling lambda 20
Value in the end 30
```

Örnekte görüldüğü gibi değiştirmek istediğimiz local değişkenleri final bir array içine atarak yada atomic referans tanımlayarak
bunu yapmak mümkün.

### Lambda ifadeden fırlatılan exceptionlar
Checked ve unchecked olmak üzere iki tip exception olduğunu daha önce görmüştük. Lambda ifadeler de normal metodlar gibi
exception fırlatabilir. Eğer bir lambda ifade checked bir exception fırlatıyorsa bu durumda lambda ifadenin implemente ettiği
fonksiyonel interface in abstract metodu da imzasında bu checked exceptionı bulundurmalı, aksi durumda compiler hata verir.

```java
public interface NumericTest<T extends Number> {

    boolean test(T a, T b);
}

public interface StringFunc {
    String func() throws IOException;
}

public class Main {

    public static void main(String[] args) {

        StringFunc strReader = () -> {
            String result = "";
            InputStreamReader reader = new InputStreamReader(System.in);
            result = String.valueOf(reader.read());

            return result;
        };


        NumericTest<Integer> isFactor = (a, b) -> {
            if (b == 0) {
                throw new IllegalArgumentException("Divider cannot be zero");
            }
            return a % b == 0;
        };

        isFactor.test(20, 5);

        isFactor.test(20, 0);
    }
}
```

Örnekte görüldüğü gibi InputStreamReader sınıfının read metodu bir checked exception fırlattığı için StringFunc interfaceinin 
func metodu throws IOException ifadesini bulundurmak zorunda.

### Metod referanslar
Daha önceki bölümlerde fonksiyonale interfaceler tarafından bize sunulan abstract metodların anonim lambda ifadeleri
tarafından nasıl implemente edildiğini gördük. Şimdi varolan metodlarımızı nasıl lambda ifadeleri ile ilişkilendirebileceğimizi
göreceğiz. Burada önemli olan referans edeceğimiz metodun imzasının fonksiyonel interface tarafından sunulan abstract 
metodun imzası ile eşleşiyor olması, aksi durumda compiler hatası oluşur.

#### static metodlara metod referans verme
Daha önce oluşturulmuş bir static metodu aşağıda gösterilen şekilde metod referans olarak kullanabiliriz;

```java
ClassName::methodName
```

Sınıf ismi ve metod ismi arasında yeralan çift iki nokta **::** ifadesi Java'ya metod referanslar için JDK8 versiyonuyla 
birlikte eklenmiştir.

```java
public interface IntPredicate {

    boolean test(int a);
}

public class MyIntPredicates {

    static boolean isPrime(int value) {

        if (value < 2) {
            return false;
        }

        for (int i = 2; i < value / i; i++) {
            if (value % i == 0) {
                return false;
            }
        }

        return true;
    }

    static boolean isEven(int value) {
        return value % 2 == 0;
    }

    static boolean isNegative(int value) {
        return value < 0;
    }

    static boolean isPositive(int value) {
        return value > 0;
    }
}

public class Main {

    static boolean numberTest(IntPredicate predicate, int value) {
        return predicate.test(value);
    }

    public static void main(String[] args) {

        boolean result = numberTest(MyIntPredicates::isPrime, 17);
        System.out.println("17 is prime : " + result);

        result = numberTest(MyIntPredicates::isEven, 22);
        System.out.println("22 is even : " + result);

        result = numberTest(MyIntPredicates::isNegative, 13);
        System.out.println("13 is negative : " + result);

        result = numberTest(MyIntPredicates::isPositive, -1);
        System.out.println("-1 is positive : " + result);
    }
}
```
Output :
```
17 is prime : true
22 is even : true
13 is negative : false
-1 is positive : false
```
Aynı anonim lambda ifadelerinde olduğu gibi test metodunu çağırana kadar lambda ifadesi çalıştırılmaz.

#### objeler üzerinden metod referans verme
Aşağıda gösterilen şekilde objeler üzerinden metod referansları verilebilir.

```java
objectName::methodName
```
Daha önce static metodlar ile oluşturduğumuz örneği şimdi biraz değiştirelim,

```java
public interface IntPredicate {

    boolean test(int a);
}

public class MyIntPredicates {

    boolean isPrime(int value) {

        if (value < 2) {
            return false;
        }

        for (int i = 2; i < value / i; i++) {
            if (value % i == 0) {
                return false;
            }
        }

        return true;
    }

    boolean isEven(int value) {
        return value % 2 == 0;
    }

    boolean isNegative(int value) {
        return value < 0;
    }

    boolean isPositive(int value) {
        return value > 0;
    }
}

public class Main {

    public static void main(String[] args) {

        MyIntPredicates predicates = new MyIntPredicates();

        IntPredicate intPredicate = predicates::isPrime;
        System.out.println("17 is prime : " + intPredicate.test(17));

        intPredicate = predicates::isEven;
        System.out.println("22 is even : " + intPredicate.test(22));

        intPredicate = predicates::isNegative;
        System.out.println("13 is negative : " + intPredicate.test(13));

        intPredicate = predicates::isPositive;
        System.out.println("-1 is positive : " + intPredicate.test(-1));
    }
}
```

#### Constructor referans
Lambda ifadeleri aracılığı ile obje yaratmak da mümkün, bunun için yaratmak istediğimiz objenin constructor ına referans 
vermek yeterli.

```java
ClassName::new
```

```java
public interface MyStringFunc {

    String build(char[] chars);
}

public class Main {

    public static void main(String[] args) {
        MyStringFunc stringFunc = String::new;
        char[] chars = new char[]{'J', 'a', 'v', 'a', ' ', 'i', 's', ' ', 'a', 'w', 'e', 's', 'o', 'm', 'e', '!'};
        String newString = stringFunc.build(chars);
        System.out.println(newString);
    }
}
```
Output :
```
Java is awesome!
```
### Öntanımlı fonksiyonel interfaceler
Daha önceki örneklerimizde lambda ifadeleri kullanmak için fonksiyonel interfaceleri kendimiz tanımlamıştık. Fakat Java 
tafından bize sunulan öntanımlı interfaceler pek çok durumda işimizi görecek ve kendi interface imizi tanımlama gereğini
ortadan kaldıracktır. Bu interfaceler java.util.function paketi altına yeralır.


| Interface        | Amacı           |
| ---------------- |:-------------|
| Consumer&lt;T&gt;      | T tipinde bir obje gönderip üzerinde işlem yapmak için kullanılır. Metodu **accept()** |
| Supplier&lt;T&gt;      | T tipinde bir obje dönmek için kullanılır. Metodu **get()**      |
| Function&lt;T,R&gt;    | T tipinde bir obje gönderip üzerinde işlem yapıp R tipinde bir obje dönmek için kullanılır. Metodu **apply()**      |
| Predicate&lt;T&gt;     | T tipinde bir obje gönderip üzerinde işlem yapıp boolean değer dönmek için kullanılır. Metodu **test()**      |

```java
import java.util.function.Predicate;

public class Main {

    public static void main(String[] args) {

        Predicate<Integer> isEven = n -> n % 2 == 0;

        System.out.println("5 is even : " + isEven.test(5));
        System.out.println("12 is even : " + isEven.test(12));
    }
}
```
Output:
```
5 is even : false
12 is even : true
```
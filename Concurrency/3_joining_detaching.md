# Join ve Detach Etme
- C++'ta eşzamanlı programlama söz konusu olduğunda, thread oluşturmak ve yönetmek işin sadece bir kısmıdır.
- Bir diğer konu ise bir thread görevini tamamladıktan sonra ne yapılacak ? 
- Threadin bitmesini beklemeli miyiz ? Yoksa bağımsız şekilde çalışmasına izin mi vermeliyiz ?
- İşte burada **joining** ve **detaching** devreye giriyor. Bu iki mekanizma, threadlerin çalışmasının tamamlandıktan sonra nasıl davranacağını kontrol ederler.

## Join Etmenin Temelleri

- Bir thread üzerinde `join()` çağırdığımızda, programa aslında şunu söylemiş oluyoruz:  
  *"Devam etmeden önce bu thread'in çalışmasını bitirmesini bekle."*

- Bu, thread tarafından hala kullanılıyor olabilecek kaynaklara erişim gibi sorunları önleyecektir.

### Ne Zaman `join()` Kullanılır ?
- Ana program devam etmeden önce bir threadin görevini tamamladığından emin olmamız gerekiyorsa `join()` kullanmalıyız.
- Bu özellikle şu durumlarda önemlidir:
   1) Thread paylaşılan veriyi güncelliyorsa
   2) Hesaplama yapıyorsa
   3) Kritik bir işi yürütüyorsa

```cpp
#include <iostream>
#include <thread>

void printNumbers() {
   for (int i = 0; i < 5; ++i) {
       std::cout << "Number: " << i << std::endl;
   }
}

int main() {
   std::thread t(printNumbers); // Yeni thread başlatıyoruz.
   t.join();                    // Threadin bitmesini bekliyoruz.

   std::cout << "Thread has finished execution." << std::endl;
   return 0;
}
```
Çıktı

```text
Number: 0
Number: 1
Number: 2
Number: 3
Number: 4
Thread has finished execution.
```
- Bu örnekte `main thread`, son mesajı yazdırmadan önce `t` threadinin çalışmasını bitirmesini bekler.
- Bu kritik bir noktadır. Çünkü eğer `t.join()` çağırmasaydık, ana program thread tamamlanmadan önce sonlanabilirdi.
- Yukarıdaki örnekte `t.join()` kısmını yorum satırına alıp kodu çalıştırdığımızda elde edilen çıktı (Her çalıştırmada farklı bir çıktı):

```text
Thread has finished execution.
terminate called without an active exception
Number: 0
```
## Detach Etmenin Temelleri

- Bir threadi detach ettiğimizde, onun *main threadden bağımsız şekilde çalışmasına* izin vermiş oluruz.
- Thread detach edildikten sonra arka planda çalışmaya devam eder ve artık daha sonra ona `join()` yapamayız ya da durumunu kontrol edemeyiz.

### Ne Zaman `detach()` Kullanılır ?

- Bir thread ile senkronize olmamız veya onun tamamlanmasını beklememiz gerekmiyorsa `detach()` kullanmak faydalı olabilir.
- Bu yöntem şu senaryolarda uygulanabilir:
   1) Loglama işlemleri
   2) Arka plan veri işleme işleri
   3) Sonucuna hemen ihtiyaç duyulmayan görevler
 
 ```cpp
#include <iostream>
#include <thread>
#include <chrono>

using namespace std;

void logMessage() {
    this_thread::sleep_for(chrono::seconds(2));
    cout << "Log message printed after 2 seconds." << endl;
}

int main() {
    thread logger(logMessage);
    logger.detach();

     cout << "Main thread is continuing its work..." << endl;

     this_thread::sleep_for(chrono::seconds(1));

     cout << "Main thread finished." << endl;

     this_thread::sleep_for(chrono::seconds(3));

     return 0;
}
```
Çıktı:

```text
Main thread is continuing its work...
Main thread finished.
Log message printed after 2 seconds.
```
- Bu örnekte loglama threadi, main thread ile paralel şekilde çalışır.
- Main thread, logger threadinin tamamlanmasını beklemeden çalışmasına devam eder.
- Ancak şuna dikkat edilmeli burada: Eğer main thread, detached threadden önce çalışmasını bitirirse, zamanlamaya bağlı olarak log mesajını hiç göremeyebiliriz.

## Join vs. Detach
- Hem `join` hem de `detach`, thread çalışmasını yönetmek için kullanılır.
- Ancak programımızın davranışını etkileyebilecek ölnemli farklar vardır. Dikkate almamız gereken bazı noktalar şunlardır:
  1) **Kaynak Yönetimi** : `join()`, main threadin ilgili threadin bitmesini beklemesini gerektirir. `detach()` ise threadin senkronizasyon olmadan çalışmasına izin verir.
  2) **Thread Sonucuna Erişim** : `join()` ile threadin dönüş değerine, varsa tabii, erişebiliriz. Ancak detached threadler daha sonra join edilemez. Bu yüzden sonuçlarını almak mümkün değildir.
  3) **Yaşam Döngüsü Kontrolü** : Join edilen threadlerin yaşam döngüsü main threade bağlıdır. Detached thread ise bağımsız çalışır ve main threadden daha uzun süre yaşayabilir.
 
```cpp
#include <iostream>
#include <thread>
#include <chrono>

using namespace std;

void compute() {
    this_thread::sleep_for(chrono::seconds(2)); // Uzun hesaplama simulasyonu
    cout << "Computation finished." << endl;
}

int main() {
    thread t1(compute);
    thread t2(compute);

    // İlk threadi join edelim.
    t1.join();

    // İkinci threadi detach edelim.
    t1.detach();

    cout << "First thread joined, seconds thread detached." << endl;

    // Detached threadin bitmesine izin vermek için bekleyelim
    this_thread::sleep_for(chrono::seconds(3)); // t2 çıktısının görüldüğünden emin olmak için.

    return 0; 
}
```
Çıktı:

```text
Computation finished.
Computation finished.
First thread joined, second thread detached.
```

- Bu örnekte ilk threadi `join` ediyoruz, ikinci threadi ise `detach` ediyoruz.
- Bu, programın ilk threadin hesaplamasını bitmesini beklemesini sağlar. Fakat ikinci thread bağımsız olarak çalışmaya devam eder.
- Ancak yine de ikinci threadin çıktısını görebilmek için main threadin yeterince uzun süre canlı kalmasını sağlamamız gerekir. 

















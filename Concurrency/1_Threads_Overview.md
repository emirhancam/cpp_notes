# Thread'lere Genel Bakış
- Daha hızlı uygulamalara olan talebin artmasıyla birlikte thread'ler, aynı anda birden fazla işi yapmamıza olanak sağlar.
- Bu da verimliliği ve uygulamanın tepki verme süresini artırır.

## 1. Thread Nedir
- Temel olarak bir thread, işletim sistemi tarafından zamanlanabilen en küçük işlem birimidir.
- Bir uygulama çalıştığında genellikle **main thread** olarak bilinen tek bir thread ile başlar.
- Bu thread, eşzamanlı olarak çalışacak ek thread'ler oluşturabilir.
- Böylede uygulamamız aynı anda birden fazla işlemi gerçekleştirebilir.
- Bir thread'i, programın izlediği bir **çalışma yolu** olarak düşünebiliriz.
- Her thread'in kendine ait bir **program counter'ı, stack'i ve yerel değişkenleri** vardır.
- Bu, thread'lerin aynı bellek alanını paylaştığı ancak bağımsız şekilde çalıştığı anlamına gelir.
- Bu durum güçlü avantajlar sağlar ancak bazı riskleri de vardır.

## 2. Neden Thread Kullanılır ?
- **Daha İyi Tepki Verebilirlik** : Bir GUI uygulamasında bir thread kullanıcı girişlerini işlerken, başka bir thread arka plandaki işleri yapabilir. Böylece arayüz donmadan çalışmaya devam edilir.
- **Paralellik** : Birden fazla thread farklı CPU çekirdeklerinde çalışabilir .Bu sayede çok çekirdekli işlemciler daha verimli kullanılır.
- **Kaynak Paylaşımı** : Aynı process içindeki threadler kaynakları paylaşır. Bu da processlere kıyasla daha düşük maliyetli bir yapı sağlar.
- Ancak bu kadar güç, büyük sorumluluk getirir. Threadler ile çalışmak, **senkronizasyon** ve **veri tutarlılığı** gibi bazı karmaşıklıkları da beraberinde getirir.

## 3. C++ Thread Kütüphanesi
- C++11, thread desteğide dahil olmak üzere concurrency için standart kütüphane getirdi.
- Thread oluşturmak ve yönetmek için kullanılan temel sınıf **std::thread** sınıfıdır.
### Temel Thread Oluşturma
- C++ta bir thread'in nasıl oluşturulduğuna bakalım:
  
```cpp
#include <iostream>
#include <thread>

void printHello() {
    std::cout << "Hello from thread!" << std::endl;
}

int main() {
    // printHello fonksiyonunu çalıştıran thread oluşturalım
    std::thread t(printHello);

    // Thread'in bitmesini bekleyelim
    t.join();

    return 0;
}
```
- Bu örnekte, sadece ekrana bir mesaj yazdıran `printHello` adlı fonksiyonu tanımladık.
- Ardından çalıştırılacak fonksiyon olarak `printHello` fonksiyonunu vererek `t` adında bir thread nesnesi oluşturduk.
- `join()` metodu, program sonlanmadan önce ana thread'in `t` thread'inin bitmesini beklemesini sağlamak için çağrılır.

### Thread Parametreleri
- Thread'ler parametre de alabilir. Bir thread fonksiyonuna argümanları nasıl geçirdiğimize bakalım:
  ```cpp
  #include <iostream>
  #include <thread>

  void printMessage(int id, const std::string& message) {
      std::cout << "Thread " << id << ": " << message << std::endl;
  }

  int main() {
      std::thread t1(printMessage, 1, "Hello from thread 1");
      std::thread t1(printMessage, 2, "Hello from thread 2");

      t1.join();
      t2.join();

      return 0;
  }
  ```

- Burada thread fonksiyonuna integer ve string gönderiyoruz.
- Her thread kendine ait mesajı ekrana yazdırır.
- Bu  örnek, aynı kodu paylaşan ancak farklı verilerle çalışan birden fazla thread'in nasıl oluşturulabileceğini gösterir.

## 4. Thread Yaşam Döngüsü
- Bir thread birkaç farklı durumda olabilir:
  
**1. New:** Thread oluşturulmuştur ancak henüz başlatılmamıştır.

**2. Runnable:** Thread çalışmaya hazırdır ve CPU zamanı bekliyordur.

**3. Blocked:** Thread, başka bir thread tarafından tutulan bir kaynağı bekliyordur. Örneğin bir `mutex` bekliyor olabilir.

**4. Terminated:** Thread çalışmasını tamamlamıştır.

- Bu yaşam döngüsünü akılda tutmak özellikle kaynak yönetimi yaparken önemlidir.
- Örneğin, bir thread süresiz olarak blocked durumda kalırsa, bu darboğazlara yol açabilir.

### C++'ta Thread Durumları
- C++, thread durumlarını yönetmemize yardımcı olabilecek fonksiyonlar sağlar.
- Bir thread'in `joinable` olup oolmadığını kontrol edebiliriz.
- Bu, thread'in `join` veya `detach` edilebilir durumda olduğu anlamına gelir.

  
```cpp
if (t.joinable()) {
    t.join();
}
```

- Bu kontrol önemlidir, çünkü **joinable olmayan** bir thread'e `join()` çağırmaya çalışmak tanımsız davranışa (UB) yol açar.

## 5. Thread Güvenliği ve Zorluklar
- Threadler uygulama performansını önemli ölçüde artırabilir.
- Ancak bunun yanında özellikle thread safety konusunda bazı zorlukları da getirir.
- Birden fazla thread aynı anda paylaşılan kaynaklara eriştiğinde, **data race** gibi problemlerle karşılaşma riski oluşur.
- Data race durumunda programın sonucu, threadlerin hangi sırada ve hangi anda çalıştığına bağlı hale gelir.

### Data Race
- Bir data race, 2 veya daha fazla threadin aynı anda paylaşılan veriye erişmesi ve bu threadlerden en az birinin o veriyi değiştirmesi durumunda oluşur.
- Örnek ile inceleyelim:

  ```cpp
  #include <iostream>
  #include <thread>

  int counter = 0;

  void increment() {
      for(int i = 0; i < 100000; ++i) {
          ++counter; //Potansiyel data race
      }
  }

  int main() {
      std::thread t1(increment);
      std::thread t2(increment);

      t1.join();
      t2.join();

      std::cout << "Final counter value: " << counter << std::endl; // UB
      return 0;
  }  
  ```

- Bu örnekte iki thread, herhangi bir senkronizasyon olmadan aynı `counter` değişkenini artırıyor. Bu durum **race condition** oluşmasına neden olur.
- Sonuç olarak `counter` değişkeninin son değeri tahmin edilemez hale gelir ve programın farklı çalıştırmalarında farklı sonuçlar görülebilir.

### Çözümler
- Bu sorunları azaltmak için `mutex` gibi senkronizasyon mekanizmaları kullanmamız gerekir.

- Şimdilik önemli olan nokta, C++'ta eşzamanlı programlama yaparken **thread safety**, tasarım aşamasında düşünülmesi gereken bir konudur.

## 6. Pratikte Dikkat Edilmesi Gerekenler
- Thread kullanan sistemler tasarlarken aşağıdaki uygulamaları göz önünde bulundurmalıyız:
- **Paylaşılan Durumu Azalt**: Mümkün oldukça threadlerin ortak veri paylaşmak yerine kendi verileri üzerinde çalışmasını sağlamalıyız. 
- **Thread Pool Kullan** : Sık sık thread oluşturmayı gereken uygulamalarda worker threadleri verimli şekilde yönetmek için thread pool kullanılmalıdır.

## 7. Gerçek Dünya Uygulamaları
- Threadler birçok alanda kullanılır:
- ** Web Sunucuları **: Aynı anda birden fazla client isteğini işler.
- ** Oyun Geliştirme **: Render işlemlerini, kullanıcı girişlerini ve oyun mantığını paralel şekilde yönetir.
- ** Veri İşleme **: Ağır hesaplamaları veya veri dönüşümlerini eşzamanlı olarak gerçekleştirir. 

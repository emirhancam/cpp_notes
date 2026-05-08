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
**Daha İyi Tepki Verebilirlik** : Bir GUI uygulamasında bir thread kullanıcı girişlerini işlerken, başka bir thread arka plandaki işleri yapabilir. Böylece arayüz donmadan çalışmaya devam edilir.
**Paralellik** : Birden fazla thread farklı CPU çekirdeklerinde çalışabilir .Bu sayede çok çekirdekli işlemciler daha verimli kullanılır.
**Kaynak Paylaşımı** : Aynı process içindeki threadler kaynakları paylaşır. Bu da processlere kıyasla daha düşük maliyetli bir yapı sağlar.
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
- Bu örnekte, sadece ekrana bir mesaj yazdıran **printHello** adlı fonksiyonu tanımladık.
- Ardından çalıştırılacak fonksiyon olarak **printHello** fonksiyonunu vererek **t** adında bir thread nesnesi oluşturduk.
- **join()** metodu, program sonlanmadan önce ana thread'in **t** thread'inin bitmesini beklemesini sağlamak için çağrılır.


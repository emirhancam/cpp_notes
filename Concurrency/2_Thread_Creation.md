# Thread Oluşturma
- Threadlerin nasıl oluşturulup yönetileceğini anlamak, verimli ve tepki verebilir uygulamalar geliştirmek için önemlidir.
- Bu bölümde thread oluşturmanın farklı yollarını göreceğiz.

## Thread Oluşturmak için `std::thread` Kullanımı
- C++'ta thread oluşturmanın en temel yolu, C++ standart kütüphanesindeki `std::thread` sınıfını kullanmaktır.
- Bu sınıf; bir fonksiyonu, lambda ifadesini veya bir sınıf metodunu çalıştıran yeni bir thread oluşturmamızı sağlar.

```cpp
#include <iostream>
#include <thread>

void hello() {
    std::cout << "Hello from a thread!" << std::endl;
}

int main() {
    std::thread t(hello);  // hello fonksiyonunu çalıştıran yeni bir thread oluştur.
    t.join();              // Thread'in bitmesini bekle.
    return 0;
}
```
Çıktı:
```text
Hello from a thread!
```

- Yukarıdaki örnekte, yeni bir thread içinde çalıştırılacak `hello` adlı bir fonksiyon tanımlarız.
- `std::thread` contructor'ı, argüman olarak fonksiyon adını alır.
- Thread oluşturulduktan sonra, çalışmasının bitmesini beklemek için `t.join()` kullanırız.
- Eğer bu çağrıyı yapmazsak, `main` fonksiyonu thread çalışmasını tamamlamadan önce sonlanabilir. Bu da tanımsız davranışa yol açar.

### Thread'lere Argüman Geçirme
- Threadin çalıştırdığı fonksiyona argümanlar da geçirebiliriz:

```cpp
#include <iostream>
#include <thread>

void greet(const std::string& name) {
    std::cout < "Hello, " << name << " from a thread!" << std::endl;
}

int main() {
    std::thread t(greet, "Alice"); // "Alice" argümanını greet fonksiyonuna geçir.
    t.join();
    return 0;
}
```
Çıktı:

```text
Hello, Alice from a thread!
```

- Bu örnekte, `greet` fonksiyonunu bir string argümanı alacak şekilde değiştirdik.
- Thread oluştururken, argümanı doğrudan fonksiyon adından veriyoruz.
- `std::thread` sınıfı, argüman geçirme işlemini bizim için kendisi halleder.

## Lambda İfadeleri ve Thread Oluşturma
- Lambda ifadeleri, küçük ve isimsiz fonksiyonlar tanımlamak için şık bir yoldur.
- Thread oluşturma konusunda özellikle kullanışlıdırlar.
- Çünkü çalıştırılacak işlemleri doğrudan thread'in oluşturulduğu yerde tanımlamamıza olanak sağlarlar. Bu da kodun okunabilirliğini artırır.

```cpp
#include <iostream>
#include <thread>

int main() {
    std::thread t( []() { std::cout << "Hello from a lambda thread!" << std::endl; } );
    t.join();
    return 0;
}
```
Çıktı:
```text
Hello from a lambda thread!
```

- Bu kod parçasında, bir lambda fonksiyonu çalıştıran yeni bir thread oluşturuyoruz. Bu lambda fonksiyonu ekrana basit bir mesaj yazdırıyor.
- Bu yöntem, ayrı bir fonksiyon tanımlamak zorunda kalmadan thread mantığını kapsüllemek için temiz bir yol sağlar.

### Lambda'larda Değişken Yakalama
- Thread işlemleri için lambda kullandığımızda, bazen lambda'nın bulunduğu dış kapsamda değişkenleri yakalamamız gerekebilir.
- Bunu, **capture clause** denilen yakalama ifadeleriyle yapabiliriz:

```cpp
#include <iostream>
#include <thread>

void increment(int& counter) {
    for (int i = 0; i < 5; ++i) {
        ++counter;
    }
}

int main() {
    int counter = 0;

    std::thread t( [&counter]() { increment(counter); } );
    t.join();

    std::cout << "Counter: " << counter << std::endl;
    return 0;
}
```
Çıktı:
```text
Counter: 5
```

- Burada lambda, `counter` değişkenini **referans olarak** yakalar.
- Bu, thread içinde `counter` için yapılan değişikliklerin `main` fonksiyonundaki `counter` değişkenine de yansıyacağı anlamına gelir.
- Bu yaklaşım kullanılırken dikkatli olunmalı! Çünkü referans olarak yakalama, referans verilen nesnenin ömrü thread bitmeden önce sona ererse problemlere yol açar. 

## Üye Fonksiyonlarla Thread Oluşturma
- Sınıfların üye fonksiyonlarını çalıştıran threadler oluşturmak biraz daha dikkat gerektirir.
- Özellikle `this` pointer'ı konusunda dikkatli olunmalıdır.

```cpp
#include <iostream>
#include <thread>

class Worker {
public:
    void doWork() {
        std::cout << "Working in a thread!" << std::endl;
    }
};

int main() {
    Worker worker;
    std::thread t(&Worker::doWork, &worker); // Üye fonksiyonu ve sınıf örneğini geçirdik.
    t.join();
    return 0;
}
```
Çıktı:
```text
Working in a thread!
```

- Bu örnekte, `doWork` adlı bir üye fonksiyona sahip `Worker` sınıfını oluşturuyoruz.
- Thread'i oluştururken, üye fonksiyonun adresini kullanıyor ve ayrıca sınıfın bir örneğini geçiriyoruz. Bu sayede thread, doğru nesneye erişebiliyor.

### Daha Karmaşık Durumlar için `std::bind` Kullanımı
- Bazen birden fazla argümanımız olabilir ya da bir üye fonksiyona belirli değerleri bağlamamız gerekebilir.

```cpp
#include <iostream>
#include <thread>
#include <functional> // std::bind için

class Worker {
public:
    void doWork(int id) {
        std::cout << "Working in thread " << id << "!" << std::endl;
    }
};

int main() {
    Worker worker;
    std::thread t( std::bind(&Worker::doWork, &worker, 1) ); // id değerini 1'e bağla.
    t.join();
    return 0;

}
```
Çıktı:
```text
Working in thread 1!
```
- Bu örnekte, `doWork` fonksiyonunun ilk argümanını `1` değerine bağlamak için `std::bind` kullanıyoruz.
- Bu sayede, birden fazla parametre geçirmemiz ya da belirli değerleri önceden sabitlememiz gereken daha karmaşık senaryolar oluşturabiliriz.





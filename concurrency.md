# C++ OOP Notları

## 1. Class Nedir?

Class, C++ dilinde veri ve fonksiyonları bir arada tutan yapıdır.

Kullanım amacı:
- Gerçek dünyadaki nesneleri modellemek
- Kod tekrarını azaltmak
- Veriyi ve davranışı aynı yerde toplamak

Örnek:

```cpp
#include <iostream>
using namespace std;

class Car {
public:
    string brand;

    void start() {
        cout << brand << " started" << endl;
    }
};

int main() {
    Car car;
    car.brand = "Toyota";
    car.start();

    return 0;
}
```

Çıktı:

```text
Toyota started
```

Açıklama:

```cpp
class Car
```

Burada `Car` adında bir sınıf tanımlanır.

```cpp
string brand;
```

Bu sınıfın bir değişkenidir. Her `Car` nesnesinin kendi `brand` değeri olabilir.

```cpp
void start()
```

Bu sınıfa ait bir fonksiyondur.

## Nerede Kullanılır?

Class yapısı şu durumlarda kullanılır:

- Birden fazla bilgiyi tek yapı altında toplamak istediğinde
- Aynı davranışa sahip nesneler oluşturmak istediğinde
- Büyük projelerde kodu düzenli tutmak istediğinde

## Kısa Özet

Class bir şablondur. Nesne ise bu şablondan oluşturulan gerçek değişkendir.

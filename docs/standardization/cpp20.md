# C++20 の新機能

## 言語機能

### ビットフィールドにデフォルトの初期値を設定可能に [(P0683R1)](http://wg21.link/p0683r1)
```C++
#include <iostream>

enum class Terrain
{
    Open, Forest, Hill, Mountain, Desert, Tundra, River, Ocean
};

struct Tile
{
    unsigned int height : 4 = 1; // デフォルト値を 1 に
    Terrain terrain : 3 = Terrain::Open; // デフォルト値を明示的に Terrain::Open に
    bool passable : 1 = true; // デフォルト値を true に
};

int main()
{
    std::cout << std::boolalpha;
    
    Tile tile1;
    std::cout << tile1.height << ", " << static_cast<int>(tile1.terrain) << ", " << tile1.passable << '\n';
    
    Tile tile2{ 15, Terrain::Mountain, false };
    std::cout << tile2.height << ", " << static_cast<int>(tile2.terrain) << ", " << tile2.passable << '\n';
}
```
```
1, 0, true
15, 3, false
```


## 標準ライブラリ

### 文字列の先頭や末尾が、ある文字列と一致するか判定 [(P0457R2)](https://wg21.link/P0457R2)
`std::basic_string` と `std::basic_string_view` に、`starts_with()` と `ends_with()` メンバ関数が追加されます。
```C++
#include <iostream>
#include <string_view>

constexpr bool HasPNGExtension(std::string_view filePath)
{
    // 文字列が ".png" で終わるなら true, それ以外は false を返す
    return filePath.ends_with(".png");
}

int main()
{
    std::cout << std::boolalpha;
    std::cout << HasPNGExtension("picture.png") << '\n';
    std::cout << HasPNGExtension("photo.jpg") << '\n';
    std::cout << HasPNGExtension("music.mp3") << '\n';
}
```
```
true
false
false
```


### `operator>>(basic_istream&, charT*)` の第二引数を `charT(&)[N]` に変更して安全に [(P0487R1)](https://wg21.link/P0487R1)

C++17 までの `operator>>(basic_istream&, charT*)` は、関数にバッファのサイズが渡されないため、次のようなプログラムでバッファオーバーフローへの対策が必要でした。
```C++
#include <iostream>
#include <iomanip>

int main()
{
	char buffer[4];
	
	// std::cin >> buffer; // 危険: バッファオーバーフローの可能性
	
	std::cin >> std::setw(4) >> buffer; // OK: バッファオーバーフロー対策
	
	std::cout << buffer;
}
```
C++20 では引数を次のように変更し、関数がバッファオーバーフローの対策を実装するようになりました。
```C++
// C++17 まで
template<class charT, class traits>
basic_istream<charT, traits>& operator>>(basic_istream<charT, traits>& in, charT* s);

// C++20 から
template<class charT, class traits, size_t N>
basic_istream<charT, traits>& operator>>(basic_istream<charT, traits>& in, charT (&s)[N]);
```
```C++
#include <iostream>
#include <iomanip>

int main()
{
	char buffer[4];
	
	std::cin >> buffer; // OK: C++20 ではバッファオーバーフローを防げる
	
	std::cout << buffer;
}
```
この変更に伴い、C++17 までは有効だった次のようなプログラムが、C++20 からコンパイルエラーになります。
```C++
#include <iostream>
#include <iomanip>

int main()
{
    char* p = new char[100];
    std::cin >> std::setw(100) >> p; // C++20 からはコンパイルエラー
    std::cout << p;
}
```


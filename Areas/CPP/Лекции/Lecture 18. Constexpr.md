<h1 align="center">CONSTEXPR</h1>

---
<p align="center">Программы времени компиляции и метапрограммы.</p>
## Метапрограммирование
#### Идея "рекурсивного" раскрытия
• Вспомним функцию `print_all`, которая была написана нами ранее.
```cpp
void print_all() { return 0; } // опечатка по словам лектора

template<typename T, typename... Args>
void print_all(T first, Args... args) {
	std::cout << first << " ";
	print_all(args...);
}
```
• Здесь порождается цепочка экземпляров шаблонной функции.
```cpp
print_all(1, 1.0, 1u);
// → print_all<double, unsigned>(1.0, 1u)
// → print_all<unsigned>(1u) → print_all()
```
#### Обсуждение
• На самом деле, никакой рекурсии здесь нет: цепочка инстанцирований порождает разные инстанцирования.
• Но сама схожесть процессов наводит на мысли.
• Первым, кого она навела на мысли был Эрвин Арнух в 1994-м году.
• И эти мысли без малого <span style="color: blue;">обусловили успех C++</span>.
#### Открытие метапрограммирования
```cpp
template<int i> struct D {
	D(void *); // 0 is ok, but not 1
	operator int();
};

template<int p, int i> struct is_prime {
	enum { prim = (p % i) && is_prime<(i > 2 ? p : 0), i - 1>::prim };
};

template<int i> struct Prime_print {
	Prime_print<i - 1> a;
	enum { prim = is_prime<i, i - 1>::prim };
	void f() { D<i> d = prim; } // error: Type enum can't be converted to type D<3>
};

struct is_prime<0, 0> { enum { prim = 1 }; };
struct is_prime<0, 1> { enum { prim = 1 }; };
struct Prime_print<2> { enum { prim = 1 }; }; void f() { D <2> d = prim; } };

int main() { Prime_print<30> a; }
```
Пример с гита:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Primes by Erwin Unruh, modern variant, by D. Vandervoorde
//
// compile with clang++ only
// clang++ unruh.cc 2> >(grep error)
//
//-------------------------------------------------------------------------------

template<int p, int i> struct is_prime {
	enum {
		prim = (p == 2) || ((p % i) && is_prime)
	};
};
```
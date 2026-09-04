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
		prim = (p == 2) || ((p % i) && is_prime<(i > 2 ? p : 0), i - 1>::prim)
	};
};

template<> struct is_prime<0, 0> {
	enum { prim = 1 };
};

template<> struct is_prime<0, 1> {
	enum { prim = 1 };
};

template<int i> struct D { D(void *); };

template<int i> struct CondNull { static int const value = i; };
template<> struct CondNull<0> { static void* value; };
void* CondNull<0>::value = 0;

template<int i> struct Prime_print {
	Prime_print<i - 1> a;
	enum { prim = is_prime<i, i - 1>::prim };
	void f() {
		D<i> d = CondNull < prim ? 1 : 0 > ::value;
		a.f();
	}
};

template<> struct Prime_print<1> {
	enum { prim = 0 };
	void f() { D<1> d = 0; };
};

int main() {
	Prime_print<70> a;
	a.f();
}
```
![[../../../_Meta/attachments/18.1.png]]
#### Факториал
• Идея лежит на поверхности: что если развернуть систематическое sfinae от типов на целые числа?
```cpp
template<size_t N> struct fact : integral_constant<size_t, N * fact<N - 1>{}> {};
template<> struct fact<0> : integral_constant<size_t, 1> {};
std::cout << fact<5>::value << std::endl;
```
• Вычисления итогового значения выполняются на этапе компиляции.
• Наследование играет роль рекурсивного вызова.
Далее лектор показывает ассемблер этой программы и в файле видно, что 120 уже посчитано.
#### Числа Фибоначчи
• С той же лёгкостью можно вычислять на этапе компиляции числа Фибоначчи.
```cpp
template<int N> struct fibonacci :
	std::integral_constant<int, fibonacci<N - 1>{} + fibonacci<N - 2>{}> {};
template<> struct fibonacci<1> : std::integral_constant<int, 1> {};
template<> struct fibonacci<0> : std::integral_constant<int, 0> {};
```
• Не смущает ли нас здесь двойная "рекурсия"?
#### Две модели вычислений
• "Императивная":
```cpp
int fact(int x) {
	int i = 2, res = 1;
	for(; i <= x; ++i)
		res *= i;
	return res;
}
```
• Временные переменные
• Циклы
• Изменяемая память
• "Функциональная":
```cpp
int fact(const int x) {
	if(x <= 2)
		return x;
	else
		return x * fact(x - 1);
}
```
• Вызовы функций
• Рекурсия
• "Чистые" вычисления
#### Целочисленный квадратный корень
• Чтобы делать такие сложные вещи на шаблонах, полезно сначала просто написать программу в функциональном стиле.
```cpp
int isqrt(int N, int lo = 1, int hi = N) {
	int mid = (lo + hi + 1) / 2;
	if(lo == hi) // это похоже на специализацию
		return lo;
	else {
		if(N < mid * mid) // как организовать if?
			return isqrt(N, lo, mid - 1);
		else
			return isqrt(N, mid, hi);
	}
}
```
#### Условный тип
• Вспомним уже известный нам условный тип:
```cpp
template<bool B, typename T, typename F>
struct conditional { using type = T; }

template<typename T, typename F>
struct conditional<false, T, F> { using type = F; }

template<bool B, typename T, typename F>
using conditional_t = typename conditional<B, T, F>::type;
```
• Это отображение `{true, false}` на `{T, F}`.
#### Целочисленный квадратный корень
• Здесь `std::conditional_t` вполне сработает в качестве `meta-if`.
```cpp
template<int N, int L = 1, int H = N, int mid = (L + H + 1) / 2>
struct Sqrt : std::integral_constant<int,
								std::conditional_t<(N < mid * mid),
																	 Sqrt<N, L, mid - 1>,
																	 Sqrt<N, mid, H>>{}> {};
	
template<int N, int S> struct Sqrt <N, S, S, S> :
	std::integral_constant<int, S> {};
```
• Домашняя наработка: попробуйте найти N-е простое число на этапе компиляции.
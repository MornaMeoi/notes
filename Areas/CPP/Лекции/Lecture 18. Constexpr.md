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
#### Квадранты вычислений
• Runtime computations
• Compile-time computations
• Type-level computations
```cpp
template<typename T> struct add_const_pointer {
	using type = const T*;
};

using types = mpl::vector<int, char, float, void>;
using pointers = mpl::transform<types, add_const_pointer<mpl::_1>>::type;
```
• Heterogenious computations
```cpp
auto to_string = [](auto t) {
	std::stringstream ss;
	ss << t;
	return ss.str();
};

fusion::vector<int, std::string, float> seq{1, "abc", 3.4f};

auto strings = fusion::transform(seq, to_string);
```
#### Обсуждение
• Поговорим о вычислениях времени компиляции.
• Допустим, я хочу предвычислить не этапе компиляции первые двадцать числе Фибоначчи и использовать их на этапе исполнения.
## Constexpr функции
#### Константность
• В чём смысл следующей конструкции, и где она может быть применима?
```cpp
uint8_t const volatile* const p_latch_reg = (uint8_t*)0x42;
```
• Это проводок с заданным адресом, с которого можно считать данные, но не изменить их.
• При этом, сами данные могу непредсказуемо измениться, так что доступ к ним нельзя оптимизировать.
```cpp
data = *p_latch_reg; // считали значение
// .....
data = *p_latch_reg; // снова считали значение
```
• Этот пример показывает, что `const` означает `readonly`.
#### Что известно на этапе компиляции
• Литералы `(1, "hello", 'c', 1.0, 1ull)` и члены `enum`.
• Параметры шаблонов и результаты `sizeof` над типами.
• <span style="color: blue;">constexpr переменные</span>
```cpp
template<typename T> struct my_numeric_limits;

template<> struct my_numeric_limits<char> {
	static constexpr size_t max() { returnm CHAR_MAX; }
};

constexpr size_t arrsz = my_numeric_limits<char>::max();
int arr[arrsz]; // OK
```
#### Ограничение на constexpr переменные
• `constexpr` переменная должна иметь литеральный тип.
• Использовать `constexprs` с плавающей точкой можно, но не рекомендуется.
```cpp
constexpr float ct = 1.0f / 3.0f;

assert(x == 1.0f && y == 3.0f);
float rt = x / y;

assert(rt == ct); // ORLY?
```
#### CONSTEXPR означает CONST?
• Следующий случай может быть несколько неочевиден:
```cpp
constexpr int arr[] = {2, 3, 5, 7, 11};
constexpr int* x = &arr[3]; // всё хорошо?
```
• Тут зависит от того, к чему относится `constexpr` во второй строчке. Варианта, собственно, два:
1. `constexpr int* x → const int* x`
2. `constexpr int* x → int* const x`
• Обсуждение: давайте проголосуем?
• Второй вариант семантически консистентен: мы объявили constexpr pointer.
```cpp
constexpr const int* x = &arr[3]; // теперь всё хорошо
```
#### C++17: constexpr control flow
• Возможность использования выражений времени компиляции делает интересным вопрос переключения по ним.
```cpp
if constexpr (b) {
	// тут много кода
} else {
	// эта ветка не участвует в инстанцировании
}
```
• Начиная с C++17 такое ленивое поведение предоставляет `if constexpr`.
• Обратите внимание: основное использование этой конструкции - это выбрасывание веток инстанцирований.
Пример:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Constexpr if application
//
//-------------------------------------------------------------------------------

#include <iostream>
#include <string>
#include <utility>

template<typename T> auto length(T value) {
	if constexpr(std::is_integral<T>::value)
		return sizeof(T);
	else
		return value.length();
}

int main() {
	int a = 5;
	std::string b = "foo";
	std::cout << length(a) << ' ' << length(b) << std::endl;
}
```
#### Некоторые альтернативы SFINAE
```cpp
template<typename T> enable_if_t<(sizeof(T) > 4)>
foo(T x) { /* сделать что-то с x */ }

template<typename T> enable_if_t<(sizeof(T) <= 4)>
foo(T x) { /* сделать что-то ещё с x */ }
```
• Кажется, теперь появился иной вариант.
```cpp
template<typename T>
void foo(T x) {
	if constexpr(sizeof(T) > 4) { /* сделать что-то с x */ }
	else { /* сделать что-то ещё с x */ }
}
```
• Но это выглядит немного интрузивно. Скоро мы увидим ещё лучшие опции.
#### If constexpr для вариабельных шаблонов
• В случае вариабельных шаблонов тоже можно избежать специализаций.
```cpp
template<typename Head, typename... Tail>
void print(Head head, Tail... tail) {
	std::cout << head;
	if constexpr(sizeof...(tail) > 0) {
		std::cout << ", ";
		print(tail...);
	}
}
```
• Вы понимаете, почему это работает?
Далее пример от лектора:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// print all, constexpr if edition
//
//-------------------------------------------------------------------------------

#include <iostream>

template<typename T, typename... Args> void print_all(T first, Args... args) {
	std::cout << first;
	if /* constexpr */ (sizeof...(args) > 0) {
		std::cout << ", ";
		print_all(args...);
	}
}

int main() {
	print_all(1, 1.0, 1u);
	std::cout << std::endl;
}
```
Компилятор выдаёт
```
error: no matching function for call to 'print_all()'
```
если не использовать constexpr.
#### Снова о метапрограммах
• Простая задача: возведение в квадрат времени компиляции.
```cpp
template<size_t n> struct square : integral_constant<size_t, n * n>;
int arr[square<5>{}]; // arr[25]
```
• Тут угадать, что `square` на самом деле функтор - довольно сложно.
```cpp
constexpr int square(int x) { return x * x; }
int arr[square(5)]; // ok, arr[25]
```
• Теперь очевидно, что мы вызываем функцию времени компиляции.
• Стандарт накладывает некоторые ограничения на тела таких функций.
#### Ограничения в C++14
• `new` и `delete`
• Генерация исключений через `throw`
• Вызов не-`constexpr` функций
• Использование goto
• Лямбда выражения
• Преобразования `const_cast` и `reinterpret_cast`
• Преобразования `void*` в `object*`
• Модификация нелокальных объектов
• Неинициализированные данные
• Сравнения с `unspecified` результатом
• Вызов `type_id` для полиморфных классов и `dynamic_cast`
• Блоки `try` для обработки исключений
• Операции с `undefined behaviour`
• Инлайн ассемблер во всех разновидностях
• Большая часть операций с `this`
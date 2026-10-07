<h1 align="center">ШАБЛОНЫ ФУНКЦИЙ</h1>

---
<p align="center">Строительные блоки обобщения и тонкости перегрузки.</p>
## Обобщённое программирование
#### Возводим число в степень
• "The first step is to get the algorithm right. The second step is to figure out which sorts of things (types) it works for." - Alex Stepanov
• Начнём с первого
```cpp
unsigned nth_power(unsigned x, unsigned n); // returns x^n
```
• Как написать тело этой функции?
#### Выбираем правильный алгоритм
• "The first step is to <span style="color: blue;">get the algorithm right</span>. The second step is to figure out which sorts of things (types) it works for." - Alex Stepanov
```cpp
unsigned nth_power(unsigned x, unsigned n) {
	unsigned acc = 1;
	if((x < 2) || (n == 1)) return x;
	while(n > 0)
		if((n & 0x1) == 0x1) { acc *= x; n -= 1; }
		else { x *= x; n /= 2; }
	return acc;
}
```
• Разумеется, вариант перемножить x ровно n раз в цикле не рассматривается.
#### Ищем возможности обобщения
• "The first step is to get the algorithm right. The second step is to <span style="color: red;">figure out which sorts of things (types) it works for</span>." - Alex Stepanov
• Как обобщить этот алгоритм?
#### Наивное обобщение
```cpp
template<typename T> T nth_power(T x, unsigned n) {
	T acc = 1;
	if((x < 2) || (n == 1)) return x;
	while(n > 0)
		if((n & 0x1) == 0x1) { acc *= x; n -= 1; }
		else { x *= x; n /= 2; }
	return acc;
}
```
• Тут всё хорошо?
#### Проблемы наивного обобщения
• Присвоение единицы сомнительно (вдруг T - это матрица?), а сравнение просто неверно (вдруг T - знаковый тип?).
#### Менее наивное обобщение
```cpp
template<typename T> T nth_power(T x, unsigned n) {
	T acc = id<T>();
	if((x == acc) || (n == 1)) return x;
	while(n > 0)
		if((n & 0x1) == 0x1) { acc *= x; n -= 1; }
		else { x *= x; n /= 2; }
	return acc;
}
```
• Этот вариант предполагает, что где-то есть функция `id<T>`, и она правильно работает. Это сильное требование. Ещё варианты?
#### Заводим traits
```cpp
template<typename T, typename Trait = default_id_trait<T>>
T nth_power(T x, unsigned n) {
	T acc = Traitd::id();
	if((x == acc) || (n == 1)) return x;
	while(n > 0)
		if((n & 0x1) == 0x1) { acc *= x; n -= 1; }
		else { x *= x; n /= 2; }
	return acc;
}
```
• Немного лучше, хотя для алгоритмов так делать не принято.
#### Отделяем чистую часть
```cpp
template<typename T> T do_nth_power(T x, T acc, unsigned n) {
	while(n > 0)
		if((n & 0x1) == 0x1) { acc *= x; n -= 1; }
		else { x *= x; n /= 2; }
	return acc;
}

unsigned nth_power(unsigned x, unsigned n) {
	if(x < 2u || n == 1u) return x;
	return do_nth_power<unsigned>(x, 1u, n);
}
```
#### Определяем требования
• Какие требования мы предъявляем к типу T?
```cpp
template<typename T>
T do_nth_power(T x, T acc, unsigned n) {
	while(n > 0) {
		if((n & 0x1) == 0x1) {
			acc *= x; n -= 1;
		}
		x *= x; n /= 2;
	}
	return acc;
}
```
• Можем ли мы формализовать перечень?
#### Добавляем концепт
• Концепт - это список требований к типу.
```cpp
template<typename T> concept multiplicative = requires(T t) {
	{ t *= t } -> std::convertible_to<T>;
};
```
• Его можно использовать через ключевое слово `requires`.
```cpp
template<typename T>
T do_nth_power(T x, T acc, unsigned n)
requires multiplicative<T> && std::copyable<T>
```
• Начинайте пользоваться стандартными концептами.
#### Интермедия: class vs typename
• Во многих местах шаблонный параметр написан как `typename`.
```cpp
template<typename T> int foo(T x);
```
• Во многих других как `class`.
```cpp
template<class T> int foo(T x);
```
• Особой разницы нет. Раньше `class` использовался, чтобы подчеркнуть, что там ожидается нетривиальный объект, но это уже никому не нужно.
• Предпочтительно (если мы знает, что ожидать) писать концепт.
```cpp
template<std::integral T> int foo(T x);
```
## Специализация
#### Инстанцирование
• Инстанцирование - это процесс порождения экземпляра специализации.
```cpp
template<typename T>
T max(T x, T y) { return x > y ? x : y; }
// ....
max<int>(2, 3); // порождает template<> int max(int, int)
```
• Мы называем этот процесс неявным (`implicit`) инстанцированием.
• Оно порождает код через подстановку параметра в шаблон и осуществляется <span style="color: blue;">по требованию</span> (то есть, лениво).
#### Порождение специализаций
```cpp
template<typename T> T do_nth_power(T x, T acc, unsigned n) {
	while(n > 0)
		if((n & 0x1) == 0x1) { acc *= x; n -= 1; }
		else { x *= x; n /= 2; }
	return acc;
}

// unsigned do_nth_power<unsigned>(unsigned x, unsigned acc, unsigned n);

unsigned nth_power(unsigned x, unsigned n) {
	if(x < 2u || n == 1u) return x;
	return do_nth_power<unsigned>(x, 1u, n);
}

/*
unsigned do_nth_power(unsigned x, unsigned acc, unsigned n) {
	while(n > 0)
		if((n & 0x1) == 0x1) { acc *= x; n -= 1; }
		else { x *= x; n /= 2; }
	return acc;
}
*/
```
#### Управление инстанцированием
• Инстанцирование может быть явно запрещено в этой единице трансляции.
```cpp
extern template int max<int, int>;
```
• Инстанцирование может быть явно вызвано.
```cpp
template int max<int>(int, int);
```
• Эта техника может использоваться для уменьшения размера объектных файлов при инстанцировании тяжёлых функций.
Пример max.hpp:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Demo of instantiations, max function
//
//-------------------------------------------------------------------------------

#pragma once

template<typename T>[[gnu::noinline]] T max(T x, T y) {
	return (x > y) ? x : y;
}

extern int foo(int x, int y);
extern int bar(int x, int y);
```
main.cc:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Demo of instantiations, main function
//
//-------------------------------------------------------------------------------

#include <iostream>

#include "max.hpp"

int main() {
	int foores = foo(0, 0);
	int barres = bar(0, 0);
	std::cout << "foo: " << foores << "; bar: " << barres << std::endl;
	return 0;
}
```
maxuser1.cc:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Demo of instantiations, function foo
//
//-------------------------------------------------------------------------------

#include "max.hpp"

int foo(int x, int y) { return max<int>(x, y + 1); }
```
maxuser2.cc:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Demo of instantiations, function bar
//
//-------------------------------------------------------------------------------

#include "max.hpp"

int bar(int x, int y) { return max<int>(x + 1, y); }
```
Далее лектор делает
```shell
g++ maxuser1.cc -c
objdump -d -M intel maxuser1.o > maxuser1.dis
g++ -O1 maxuser2.cc -c
objdump -d -M intel maxuser2.o > maxuser2.dis
ar cr libmax.a maxuser1.o maxuser2.o
g++ main.cc -L. -lmax -o result
objdump -d -M intel libmax.a > libmax.dis
```
И показывает, что у нас есть разные экземпляры функции max в maxuser1.dis и в maxuser2.dis. Они же **обе** попадают в либу, что видно в libmax.dis. Что же вызовется, если пользоваться этой библиотекой? Лектор говорит, что implementation defined.
Далее лектор в качестве решения предлагает заблокировать инстанцирование в одном из файлов:
maxuser2.cc:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Demo of instantiations, function bar
//
//-------------------------------------------------------------------------------

#include "max.hpp"

// block instantiation here
extern template int max<int>(int, int);

int bar(int x, int y) { return max<int>(x + 1, y); }
```
Спрашивает, нравится ли это решение?) Говорит, что хуета (конечно, не прямая цитата). И предлагает инстанцировать функцию в другом месте. Для этого создаёт
max.cc:
```cpp
// шапки нет в оригинале
#include "max.hpp"

// POI just here
template int max<int>(int, int);
```
А в max.hpp:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Demo of instantiations, max function
//
//-------------------------------------------------------------------------------

#pragma once

template<typename T>[[gnu::noinline]] T max(T x, T y) {
	return (x > y) ? x : y;
}

// block instancing everywhere
extern template int max<int>(int, int);

extern int foo(int x, int y);
extern int bar(int x, int y);
```
Ну и далее демонстрирует, что теперь в дизассемблере либы всего 1 экземпляр функции max.
#### Явная специализация
• Кроме инстанцирования (явного или неявного), шаблонная функция или структура может быть явно специализирована.
```cpp
template<typename T> T max(T x, T y) { /*....*/ }
template<> int max<int>(int x, int y) { /*....*/ }
template<> float max<float>(float x, float y) { /*....*/ }
```
• Специализация обязана физически следовать за основным шаблоном.
```cpp
template<> int min<int>(int x, int y) { /*....*/ } // ошибка
template<typename T> T min(T x, T y) { /*....*/ } // primary тут
```
#### Пример явной специализации: OpenCL
• Общий случай:
```cpp
template<typename T> struct ReferenceHandler { };
```
• Конкретные случаи:
```cpp
template<> struct ReferenceHandler<cl_mem> {
	static cl_int retain(cl_mem memory) { return ::clRetainMemObject(memory); }
	static cl_int release(cl_mem memory) { return ::clReleaseMemObject(memory); }
};
```
• Теперь `ReferenceHandler<X>::release()` - это либо `release X`, либо ошибка.
#### Правила игры
• Общее правило для функций и не только `[temp.spec.general]`
	• Явное инстанцирование единожды в программе.
	• Явная специализация единожды в программе.
	• Явное инстанцирование должно **следовать за** явной специализацией.
```cpp
template<typename T> max(T x, T y) { /*....*/ }
template<> int max<int>(int x, int y) { /*....*/ }
template int max<int>(int, int); // вынудили инстанцировать
```
• Нарушение влечёт за собой `IFNDR`.
• Как вы думаете, как играет запрет инстанцирования со специализацией?
Пример на "поиграться с явным инстанцированием":
```cpp
#include "gtest/gtest.h"

#define EARLY 0
#define SPECIALIZE 1

template<typename T> T max(T x, T y) {
	return x > y ? x : y;
}

#if (EARLY == 1)
extern template int max<int>(int, int);
#endif

#if (SPECIALIZE == 1)
template<> int max(int x, int y) {
	return x > y ? x : y;
}
#endif 

#if (EARLY == 0)
extern template int max<int>(int, int);
#endif 

TEST(max_test_case, testWillPass) {
	EXPECT_EQ(max(2, 11), 11);
}
```
При EARLY 1 и SPECIALIZE 1:
```
error: explicit specialization of 'max<int>' after instantiation
```
Лектор рекомендует посмотреть стандарт. Гарантирует смех.
#### Инстанцирование и специализация
• Явная специализация может войти в конфликт с инстанцированием.
```cpp
template<typename T> T max(T x, T y);

// OK, указываем явную специализацию
template<> double max(double x, double y) { return 42.0; }

// никакой implicit instantiation не нужно
int foo() { return max<double>(2.0, 3.0); }

// процесс implicit instantiation нужен, и он произошёл
int bar() { return max<int>(2, 3); }

// ошибка: мы уже породили эту специализацию
template<> int max(int x, int y) { return 42; }
```
#### Удаление специализаций
• Частым случаем явной специализации является её запрет.
```cpp
// для всех указателей
template<typename T> void foo(T*);

// но не для char* и не для void*
template<> void foo<char>(char*) = delete;
template<> void foo<void>(void*) = delete;
```
• Как вы думаете, что произойдёт, если мы сначала сгенерируем специализацию, а потом запретим её?
#### Non-type параметры
• Параметры, не являющиеся типами могут быть **структурными типами**.
• Структурные типа - это:
	• Скалярные типы (кроме плавающей точки).
	• Левые ссылки.
	• Структуры, у которых все поля и базовые классы `public` и не `mutable`. И, при этом, все поля и поля всех базовых классов тоже структурные типы или массивы.
```cpp
struct Pair { int x, int y};
template<int N, int* PN, int& RN, Pair P> int foo();
```
• Базовая интуиция: всё должно быть compile-time known.
#### Специализация по nontype параметрам
• Нет никаких проблем в том, чтобы специализировать класс по нетиповым параметрам.
```cpp
template<typename T, int N> foo(T(&Arr)[N]);
template<> foo<int, 3>(int(&Arr[3])) {
	// тут более эффективная реализация для трёх целых
}
```
• Обратите внимание: при явной специализации функций вы обязаны указать все параметры.
• Как вы видите себе специализацию по указателям, ссылкам и структурным типам?
• Массив в шаблоннов параметре редуцируется до указателя (как в функции).
#### Шаблонные шаблонные параметры
• Параметрами могут быть шаблоны классов.
```cpp
template<template<typename> typename Cont, typename Elt>
void print_size(const Cont<Elt>& a);
```
• Разумеется, специализация по ним тоже возможна.
• Пока что кажется, что это переусложнение.
```cpp
template<typename Container>
void print_size(const Container& a);
```
• Это работает не хуже (а, собственно, лучше, например, для `vector<int>`).
• Мы вернёмся к этому при разговоре о шаблонах классов.
## Вывод типов в функциях
#### Вывод типов до подстановки
• Для параметров, являющихся типами, работает вывод типов.
```cpp
int x = max(1, 2); // -> int max<int>(int, int);
```
• При выводе режутся ссылки и внешние cv-квалификаторы.
```cpp
const int& a = 1;
const int& b = 2;
int x = max(a, b); // -> int max<int>(int, int);
```
• Вывод не работает, если он не однозначен.
```cpp
unsigned x = 5; do_nth_power(x, 2, n); // FAIL
int a = 1; float b = 1.0; max(a, b); // FAIL
```
#### Вывод типов после подстановки
• Вывод типов внутри шаблонной функции даёт точки вывода, где разрешить тип можно только после подстановки:
```cpp
template<typename T> T max(T x, T y) { /*....*/ }

template<typename T> T min(T x, T y) { /*....*/ }

template<typename T> bool
test_minmax(const T& x, const T& y) {
	if(x > y) return test_minmax(y, x);
	return min(x, y) == x && max(x, y) == y;
}
```
• Таким образом, вывод и подстановка включаются попеременно.
#### Вывод уточнённых типов
• Иногда шаблонный тип аргумента  может быть уточнён ссылкой или указателем и cv-квалификатором.
```cpp
template<typename T> T max(const T& x, const T& y);
```
• В этом случае, выведенный тип тоже будет уточнён.
```cpp
int a = max(1, 3); // -> int max<int>(const int&, const int&);
```
• Уточнённый вывод иначе работает с типами: он сохраняет cv-квалификаторы.
```cpp
template<typename T> void foo(T& x);

const int& a = 3;
int b = foo(a); // void foo<const int>(const int& x);
```
#### Вывод ещё более уточнённых типов
• Вывод типов работает шире, чем люди обычно думают.
```cpp
template<typename T> int foo(T(*p)(T));
int bar(int);
foo(bar); // -> int foo<int>(int(*)(int));
```
• Могут быть выведены даже параметры, являющиеся константными.
```cpp
template<typename T, int N> void buz(T const(&)[N]);
buz({1, 2, 3}); // -> void buz<int, 3>(int const(&)[3]);
```
• Общее правило: вывод типов матчит сложные композитные типы.
#### Частичный вывод типов
• В некоторых случаях у нас просто нет **контекста вывода**.
```cpp
template<typename DstT, typename SrcT>
DstT implicit_cast(SrcT const& x) {
	return x;
}

double value = implicit_cast(-1); // fail!
```
• Тогда мы можем указать необходимое и положиться на вывод остального.
```cpp
double value = implicit_cast<double, int>(-1); // ok
double value = implicit_cast<double>(-1); // ok
```
#### Обсуждение
• Иногда возникает контекст, где при выводе хочется убрать параметр-другой.
```cpp
template<typename T> int foo(T x) {
	return 42;
}

int x = foo()l // -> int foo<void>()
```
• Представьте, что вы в комитете. Вы бы разрешили такое?
#### Параметры по умолчанию
• Допустим, у вас есть функция, берущая по умолчанию плавающее число.
```cpp
template<typename T> void foo(T x = 1.0);
```
• Увы, если его не указать, вывод типов работать не будет.
```cpp
foo(1); // ok, foo<int>(1);
foo<int>(); // ok, foo<int>(1.0 narrowed to int);
foo(); // fail
```
• Тем не менее, ситуацию можно исправить. Трюк не так уж и сложен. Догадки?
```cpp
template<typename T = double> void foo(T x = 1.0);
```
• Заметьте: во втором случае ниже вывода типов всё ещё нет.
```cpp
foo(1); // ok, foo<int>(1);
foo<int>(); // ok, foo<int>(1.0 narrowed to int);
foo(); // ok
```
• Параметр по умолчанию шаблона в данном случае подсказывает компилятору, что делать.
#### Вывод типов non-type параметров
• Что делать компилятору, если в шаблоне вывод типов требуется в списке параметров?
```cpp
template<auto n> int foo() { /*....*/ }

foo<1>();
foo<1.5>();
```
• В этом случае компилятор выводит тип построением `invented expression`.
```cpp
auto n = 1;
auto n = 1.5;
```
• Кстати, clang до сих пор не поддержал FP NTTP из C++20.
#### Вывод специализирующего типа
• Очень интересной техникой является оставить специализирующий тип выводу типов.
```cpp
template<typename T> T foo(T x) { /* code for all */ }
template<> int foo(int x) { /* code for int */ } // -> foo<int>
```
• Это удобно, и это часто применяется. Но иногда сложно понять, по чему специализируем.
```cpp
template<>
inline cl_int cl::Program::getInfo(cl_program_info name,
																	 vector<vector<unsigned char>>* param) const {
}
```
## Перегрузка
#### Разбор примера с прошлой лекции
• В конце прошлой лекции была поставлена задача написать `operator==` для `basic_string`.
• Один из простых вариантов решения.
```cpp
template<typename CharT, typename Traits, typename Alloc>
bool operator==(const basic_string<CharT, Traits, Alloc>& lhs,
								const basic_string<CharT, Traits, Alloc>& rhs) {
	return lhs.compare(rhs) == 0;
}
```
• Чем он плох?
• Он неэффективен. Подумайте про ("hello" == str), тут явно создаётся лишняя копия. Мы бы хотели его перегрузить, как обучную функцию.
#### Лучший вариант
• Принятый (в т.ч. в libstdc++) вариант решения использует перегрузки.
```cpp
template<typename CharT, typename Traits, typename Alloc>
bool operator==(const basic_string<CharT, Traits, Alloc>& lhs,
								const basic_string<CharT, Traits, Alloc>& rhs) {
	return lhs.compare(rhs) == 0;
}

template<typename CharT, typename Traits, typename Alloc>
bool operator==(const CharT* lhs, const basic_string<CharT, Traits, Alloc>& rhs){
	return rhs.compare(lhs) == 0;
}

template<typename CharT, typename Traits, typename Alloc>
bool operator==(const basic_string<CharT, Traits, Alloc>& lhs, const CharT* rhs){
	return lhs.compare(rhs) == 0;
}
```
#### Обсуждение
• Что является единицей перегрузки в языке C++?
```cpp
foo(s);
```
• Например, как может быть истолкована строчка выше?
Мозговыносящий Владимиров be like:
```cpp
#include <concepts>

#include "gtest/gtest.h"

struct foo {
	int x = 1;
	foo() = default;
	static int s;
	foo(int x) { s = x; }
};

int foo::s;

TEST(ovrnames, structtest) {
	foo(s);
	EXPECT_EQ(s.x, 1);
}

TEST(ovrnames, structctor) {
	int x = 2;
	delete new foo(x);
	EXPECT_EQ(foo::s, 2);
}

int foo(int) { return 3; }

TEST(ovrnames, fn) {
	int s = 0;
	foo(s);
	EXPECT_EQ(foo(s), 3);
}
```
#### Общий обзор правил перегрузки
• Выбирается множество **перегруженных имён**.
• Выбирается множество **кандидатов**.
• Из множества кандидатов выбираются **жизнеспособные** (viable) кандидаты для данной перегрузки.
• Лучший из жизнеспособных кандидатов выбирается на основании **цепочек неявных преобразований** для каждого параметра.
• Если лучший кандидат **существует и является единственным**, то перегрузка разрешена успешно, иначе программа ill-formed.
#### Спрятанные имена
• Вопрос для новичка: что на экране?
```cpp
struct B {
	void f(int) { std::cout << "B" << std::endl; }
};

struct D : B {
	void f(const char*) { std::cout << "D" << std::endl; }
};

int main() {
	D d;
	d.f(0);
}
```
• Прошлый пример может показаться странным, но давайте его упростим.
```cpp
void f(int) { std::cout << "B" << std::endl; }
void f(const char*) { std::cout << "D" << std::endl; }

int main() {
	extern void f(const char*);
	f(0); // тут всё очевидно
}
```
• Опеределения ищутся внутри общего scope, что упрощает реализацию компилятора.
• Но тут может возникнуть вопрос насчёт пространства имён.
#### Проблема: операторы
• Обычно оператор может находиться в любом пространстве имён.
```cpp
std::cout << "Hello!\n";
```
• Это вполне может быть эквивалентно следующему.
```cpp
operator<<(std::cout, "Hello!\n");
```
• Чтобы это работало, это должен быть оператор из пространства имён `std`.
```cpp
std::operator<<(std::cout, "Hello!\n");
```
• Но компилятор не может об этом догадаться из записи `std::a << b`.
#### Решение: поиск Кёнига
• Эндрю Кёниг предложил решение в начале 90-х.
	1. Компилятор ищет имя функции из текущего и всех охватывающих пространств имён.
	2. Если оно не найдено, компилятор ищет имя функции в пространствах имён её аргументов.
```cpp
namespace N { struct A; int f(A*); }
int g(N::A* a) { int i = f(a); return i; }
```
Может найти вот так:
```cpp
typedef int f;
namespace N { struct A; int f(A*); }
int g(N::A* a) { int i = f(a); return i; }
```
и это будет каст к инту.)
#### Поиск Кёнига и шаблоны
• Следующий пример не работает.
```cpp
namespace N {
	struct A;
	template<typename T> int f(A*);
}

int g(N::A* a) {
	int i = f<int>(a); // FAIL
	return i;
}
```
• Кто-нибудь может угадать причину?
• Можно заставить это работать, введя f как имя шаблонной функции.
```cpp
namespace N {
	struct A;
	template<typename T> int f(A*);
}

template<typename T> void foo(int); // неважно, какой параметр

int g(N::A* a) {
	int i = f<int>(a); // теперь всё ok
	return i;
}
```
#### Идея построения цепочки
• С наивной точки зрения, в цепочку преобразований входят:
	• С высшим приоритетом: стандартные преобразования.
	• Немного ниже: пользовательские преобразования.
	• С низшим приоритетом: троеточия.
• Сложности начинаются, когда их комбинируется много разных.
```cpp
struct S { S(int){} };

void foo(int); // 1
void foo(S);   // 2
void foo(...); // 3

foo(1); // -> 1
```
#### Стандартные преобразования
• Трансформация объектов (ранг точного совпадения).
```cpp
int arr[10]; int* p = arr; // [conv.array]
```
• <span style="color: blue;">Коррекции квалификаторов</span> (ранг точного совпадения).
```cpp
int x; const int* px = &x; // [conv.qual]
```
• Продвижения (ранг продвижения).
```cpp
int res = true; // [conv.prom]
```
• Конверсии (ранг конверсии).
```cpp
float f = 1; // [conf.fpint]
```
<table>
<caption>Table 16: Conversions [tab:over.ics.scs]</caption>
<thead>
<tr><th>Conversion</th><th>Category</th><th>Rank</th><th>Subclause</th></tr>
</thead>
<tbody>
<tr><td>No conversions required</td><td>Identity</td><td rowspan="6">Exact Match</td><td></td></tr>
<tr><td>Lvalue-to-rvalue conversion</td><td rowspan="3">Lvalue Transformation</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.lval">7.3.1</a></td></tr>
<tr><td>Array-to-pointer conversion</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.array">7.3.2</a></td></tr>
<tr><td>Function-to-pointer conversion</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.func">7.3.3</a></td></tr>
<tr><td>Qualification conversions</td><td rowspan="2">Qualification Adjustment</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.qual">7.3.5</a></td></tr>
<tr><td>Function pointer conversion</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.fctptr">7.3.13</a></td></tr>
<tr><td>Integral promotions</td><td rowspan="2">Promotion</td><td rowspan="2">Promotion</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.prom">7.3.6</a></td></tr>
<tr><td>Floating-point promotion</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.fpprom">7.3.7</a></td></tr>
<tr><td>Integral conversions</td><td rowspan="6">Conversion</td><td rowspan="6">Conversion</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.integral">7.3.8</a></td></tr>
<tr><td>Floating-point conversions</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.double">7.3.9</a></td></tr>
<tr><td>Floating-integral conversions</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.fpint">7.3.10</a></td></tr>
<tr><td>Pointer conversions</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.ptr">7.3.11</a></td></tr>
<tr><td>Pointer-to-member conversions</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.mem">7.3.12</a></td></tr>
<tr><td>Boolean conversions</td><td><a href="https://timsong-cpp.github.io/cppwp/n4861/conv.bool">7.3.14</a></td></tr>
</tbody>
</table>
#### Пользовательские преобразования
• Задаются `implicit` конструктором, либо оператором преобразования.
```cpp
struct A {
	operator int(); // 1
	operator double(); // 2
};

int i = A{}; // calls (1)
```
• При этом, (1) лучше чем (2), потому что для него нужно меньше стандартных преобразований.
• Интуитивно: у цепочки короче хвост, значит она лучше.
#### Тонкости построения цепочек
```cpp
struct S{ S(long){} };
void foo(S) {}
int x = 42;
foo(x); // int -> long -> S
```
```cpp
struct T { T(int){} };
struct S { S(T){} };
void foo(S) {}
int x = 42;
foo(x); // int -> T -> S
```
Далее лектор отмечает, что второе, как раз, не работает, потому что в цепочке не может быть более одного пользовательского преобразования.
#### Перегрузка и шаблоны
• Шаблон может выиграть перегрузку. При этом, запускается вывод типов.
```cpp
void foo(double x);                 // 1
template<typename T> void foo(T x); // 2
foo(1); // несомненно, 2
```
• Для выигравшего перегрузку шаблона запускается инстанцирование или ищется специализация.
```cpp
template<> void foo<int>(int x);    // 3
foo(1); // -> что вы думаете?
```
#### Что если вывод удался дважды?
• Рассмотрим более сложный пример.
```cpp
template<typename T> void f(T);  // 1
template<typename T> void f(T*); // 2
```
• В точке вызова у нас нечто вроде
```cpp
int*** a;
foo(a); // -> ?
```
• Здесь вывод работает, но шаблоны сами по себе перегружены.
• Тогда, внезапно, вывод будет повторён дважды.
#### Частичный порядок шаблонов функций
• В случае шаблонов функций, они частично упорядочены на более и менее специализированные.
• Для установления этого порядка мы должны трансформировать параметры.
```cpp
template<typename T> void f(T); // -> f(T1)
template<typename T> void f(T*); // -> f(T2*)
```
• И далее запустить вывод типов.

| \#template             | Parameter type                    | Argument type                                       |
| ---------------------- | --------------------------------- | --------------------------------------------------- |
| `template`<sub>1</sub> | `template<typename T> void f(T)`  | <span style="color: blue;">f(T<sub>1</sub>)</span>  |
| `template`<sub>2</sub> | `template<typename T> void f(T*)` | <span style="color: blue;">f(T<sub>2</sub>*)</span> |
#### Результирующее отношение
• Мы видим, что нам нужно рассмотреть 2 случая
```cpp
// 1. compare (1) not less specialized than (2)
template<typename T> void f(T);
T2* a; f(a); // OK, (1) >= (2)

// 2. compare (2) not less specialized than (1)
template<typename T> void f(T*);
T1 b; f(b); // FAIL, (2) < (1)
```
• Откуда очевидно (1) > (2).
#### Некая двусмысленность в выводе
1. `template<typename T> void foo(T);`
2. `template<typename T> void foo(T*);`
3. `template<> void foo(int*);`
 ![[../../../_Meta/attachments/Магистерский курс/2.1.png]]
```cpp
int x;
foo(&x); // вызовет [3], и это, в целом, ok, но....
```
• Вопрос: является (3) специализацией для (2) или для (1) не имеет смысла. Она одинаково хорошо подходит для обоих, поэтому специализирует <span style="color: blue;">выигравший перегрузку</span> шаблон.
• В связи с этим могут возникать неприятные сюрпризы.
#### Контрпример Димова-Абрамса
```cpp
template<typename T> void foo(T);  // 1
template<> void foo(int*);         // 2
template<typename T> void foo(T*); // 3

int x;
foo(&x); // вызовет [3], хотя [2] подходит лучше
```
• Важно помнить: <span style="color: blue;">специализации не участвуют в перегрузке</span>. Сначала разрешается перегрузка, потом ищется наименее общая специализация.
• Но в данном случае (2) не специализирует (3), так как встречается раньше.
• В целом, это аргумент против специализации.
#### Контрольный вопрос
1. `template<typename T, typename U> void foo(T, U);`
2. `template<typename T, typename U> void foo(T*, U*);`
3. `template<> void foo<int*, int*>(int*, int*);`
```cpp
int x;
foo(&x, &x); // ???
```
 ![[../../../_Meta/attachments/Магистерский курс/2.1.png]]
#### Обсуждение
• Перегрузка и специализация для функций выглядят дублирующими механизмами. Но, как рассмотрено выше, это очень разные вещи.
• Ситуация становится ещё интереснее, когда в дело включаются классы.
#### Литература
• ISO/IEC, Information technology - Programming languages - C++, 14882:2020
• Bjarne Stroustrup, The C++ Programming Language (4th Edition)
• Alexander A. Stepanov, Paul McJones - Elements of programming, Addison-Wesley, 2009
• Alexander A. Stepanov, Daniel E. Rose - From mathematics to generic programming, Addison-Wesley, 2014
• David Vandevoorde, Nicolai M. Josuttis, Douglas Gregor - C++ Templates - The Complete Guide, 2nd Edition, Addison-Wesley, 2017
• Stephan T. Lavavej - Core C++, lectures 1, 2 and 3
• Andrei Alexandrescu, Generic: Min and Max Redivivus, Dr. Dobb's, 2001
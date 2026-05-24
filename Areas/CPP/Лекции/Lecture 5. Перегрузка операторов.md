<h1 align="center">ОПЕРАТОРЫ В C++</h1>

---
<p align="center">Приведение типов. Операторы и цепочки. Бинарные операторы. Экзотика.</p>
## Приведение типов
#### Типы гораздо важнее в C++ чем в C
• В заголовок этого слайда вынесено неоспоримое утверждение.
	• Типы участвуют в разрешении имён.
	• Типы могут иметь ассоциированное поведение.
	• За счёт шаблонной параметризации, типов может быть куда больше, их куда проще порождать из обобщённого кода.
• Но, при всём этом, любой объект - это просто кусок памяти:
```cpp
float f = 1.0;

char x = *((char *)&f + 2); // Это легально. Что в x?
```
#### Обсуждение
• Не имеет ли приведение в стиле C (реинтерпретация памяти) тёмных сторон?
• Конечно, имеет. Она слишком разрешающая. Есть некая разница между
	• Приведением int к double
	• Приведением const int* к int*
	• Приведением int* к long
• Первое - это обычное дело, второе - это опасное снятие внутренней константности, третье - за гранью добра и зла.
```c
x = (T) y;
```
• Но в языке C всё это пишется одинаково.
#### Приведение в стиле C++
• static_cast - обычные безопасные преобразования.
```cpp
int x;
double y = 1.0;
x = static_cast<int>(y);
```
• const_cast - снятие константности или волатильности.
```cpp
const int *p = &x;
int *q = const_cast<int *>(p);
```
• reinterpret_cast - слабоумие и отвага.
```cpp
long long uq = reinterpret_cast<long long>(q);
```
• <span style="color: blue;">Но лучше, чем C-style cast</span>.
```cpp
char c;
std::cout << "char # " << static_cast<int>(c) << std::endl;

int i;
const int* p = &i;
std::cout << "int: " << *(const_cast<int*>(p)) << std::endl;
```
• В обоих эти случаях reinterpret_cast будет ошибкой компиляции.
#### Избегаем reinterpret_cast в C++20
• Побитовая реинтерпретация значения очень коварна.
```cpp
float p = 1.0;
int n = *reinterpret_cast<int*>(&p); // [basic.lval/11] UB
```
• Чтобы вы так не делали, в C++ появилась функция std::bit_cast.
```cpp
int m = std::bit_cast<int>(p);
```
• Она делает примерно следующее:
```cpp
std::memcpy(&m, &p, sizeof(int));
```
• И не вовлекает вас в грех перед строгим алиасингом.
#### Static cast - это явное преобразование
• Уже рассмотренные нами explicit конструкторы регламентируют необходимость static_cast.
```cpp
struct T{};
struct S { explicit S(T) {} };

void foo(S s) {}

foo(T); // FAIL
foo(static_cast<S>(T)); // OK
```
• То же самое касается синтаксиса копирующей инициализации.
```cpp
T x; S x = static_cast<S>(y); // OK
```
#### Functional style cast в C++
• Функциональный каст - это C-style cast вывернутый наизнанку.
```cpp
int a = (int) y; // C-style
int b = int(y); // functional-style C-style cast
```
• Разницы между ними нет, но заметьте
```cpp
int c = int{y}; // ctor, блокирует сужающее преобразование
int d = S(x, y); // ctor, два аргумента
```
• Неприятно иногда вместо честного конструирования влипнуть в C-style cast.
• Итак, почти всегда наш выбор - это static_cast или нечто похожее.
• В частности, он является нашим выбором для явных преобразований типов.
#### Обсуждение
• Кроме того, что C++ style casts позволяют чётко указать, что вы хотите, они еще и лучше видны в коде.
• По ним проще искать, чтобы их удалить, потом что вообще-то в статически типизированном языке преобразование типов - это сигнал о проблемах в проектировании.
• Самый "безопасный" static_cast на самом деле самый сложный, т.к. у него нет чётких правил, что на входе, и что на выходе.
• static_cast определяет явные преобразования. Но как типы преобразуются неявными преобразованиями?
#### Особенности неявного поведения
• В наследство от языка C нам достались неявные арифметические преобразования.
```c
int a = 2; double b = 2.8;
short c = a * b; // c = ?
```
• Со своими странностями и засадами.
```c
unsigned short x = 0xFFFE, y = 0xEEEE; // x * y = 0xEEEC2224
unsigned short v = x * y;     // v = ?
unsigned w = x * y;           // w = ?
unsigned long long z = x * y; // z = ?
```
• Может ли кто-нибудь исчерпывающе изложить сишную часть правил?
Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Example for integral promotions
//
//---------------------------------------------------------------------------

#include <iostream>

int main() {
	int a = 2;
	double b = 2.8;
	short c = a * b;
	std::cout << std::dec << a << " * " << b << " = " << c << std::endl;
	
	unsigned short x = 0xFFFE, y = 0xEEEE;
	unsigned short v = x * y;
	std::cout << std::hex << x << " * " << y << " = " << v << std::endl;
	
	unsigned w = x * y;
	std::cout << std::hex << x << " * " << y << " = " << w << std::endl;
	
	// in unsigned long long shall be represented value of signed int
	unsigned long long z = x * y;
	std::cout << std::hex << x << " * " << y << " = " << z << std::endl;
}
```
• Сишные правила (применять сверху-вниз):
```c
type 'op' fptype => fptype 'op' fptype
```
• Порядок: long double, double, float.
```c
type 'op' unsigned itype => unsigned itype 'op' unsigned itype
type 'op' itype => itype 'op' itype
```
• Порядок: long long, long, int.
```c
(itype less then int) 'op' (itype less then int) => int 'op' int
```
• Любые комбинации (unsigned) short  и (unsigned) char.
• Неявные касты на инициализации:
```c
widetype x, narrowtype y;

[decayed] widetype z = y; // ok
[decayed] narrowtype v = x; // ok, если v вмещает значение x
```
• Понятно, что параметры функции - это тоже инициализация.
```c
void foo(double);
foo(5); // ok, int implicitly promoted
```
#### Унарный плюс (positive hack)
• Оператор унарного плюса интересен тем, что почти для всех строенных типов он не значит ничего. Например, 2 == +2.
• Но, при этом, он, даже если не перегружен, предоставляет легальный способ вызвать приведение к встроенному типу.
```cpp
struct Foo { operator long() { return 42; }};

void foo(int x);

void foo(Foo x);

Foo f;
foo(f); // вызовет foo(Foo)
foo(+f); // вызовет foo(int)
```
## Операторы и цепочки
#### Ваши типы как встроенные
• Собственный класс кватернионов
```cpp
template<typename T> struct Quat {
	T x, y, z, w;
};
```
• У нас уже есть бесплатное копирование и присваивание. Хотелось бы, чтобы работало всё остальное: сложение, умножение на число и так далее.
• Начнём с чего-нибудь простого:
```cpp
Quat q {1, 2, 3, 4};

Quat p = -q; // унарный минус: {-1, -2, -3, -4}
```
#### Общий синтаксис операторов
• Обычно используется запись operator и далее - какой это оператор.
```cpp
template<typename T> struct Quart {
	T x, y, z, w;
};

template<typename T> Quat<T> operator-(Quat<T> arg) {
	return Quat<T>{-arg.x, -arg.y, -arg.z, -arg.w};
}
```
• Теперь всё как надо.
• Альтернатива: метод в классе:
```cpp
template<typename T> struct Quat {
	T x, y, z, w;
	
	Quat operator-() const {
		return Quat{-x, -y, -z, -w};
	}
};
```
• И снова всё как надо.
#### Обсуждение
• Обычно есть два варианта (исключение: присваивание и пара-тройка других).
• -a означает a.operator-()
• -a означает operator-(a)
• Как вы думаете, что будет, если определить оба?
Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Both op- redefined inside and outside class
//
//---------------------------------------------------------------------------

#include <iostream>

template <typename T> struct Quat {
	T x, y, z, w;
	
	Quat operator-() const {
		std::cout << "in class\n";
		return Quat{-x, -y, -z, -w};
	}
};

template<typename T> Quat<T> operator-(Quat<T> arg) {
	std::cout << "out class\n";
	return Quat<T>{-arg.x, -arg.y, -arg.z, -arg.w};
}

int main() {
	Quat<float> q{1, 2, 3, 4};
	Quat<float> p = -q;
	Quat<float> r = operator-(q);
	Quat<float> s = q.operator-();
}
```
Вывод:
```bash
in class
out class
in class
```
#### Обсуждение
• Как вы думаете, чем закончится попытка:
• перегрузить operator- для int
```cpp
int operator-(int x) {
	std::cout << "MINUS!" << std::endl;
	return x;
}
```
• перегрузить operator- для всего подряд. В том числе, и для int.
```cpp
template <typename T> T operator-(T x) {
	std::cout << "MINUS!" << std::endl;
	return x;
}
```
В первом случае будет ошибка компиляции, во втором - ошибка инстанцирования.
#### Обсуждение
• Унарный минус всё-таки немного сомнительный оператор для перегрузки.
• Давайте, прежде чем двигаться дальше, мотивируем перегрузку операторов. То есть, покажем, как она даёт нам производительность и возможности.
#### Функторы: постановка проблемы
• Эффективность std::sort резко проседает, если для его объектов нет operator< и нужен кастомный предикат
```cpp
bool gtf(int x, int y) { return x > y; }

// неэффективно: вызовы по указателю
std::sort(myarr.begin(), myarr.end(), &gtf);
```
• Можно ли с этим что-то сделать?
#### Функторы: первый вариант решения
• Функтором называется класс, который ведёт себя как функция.
• Простейший способ - это неявное приведение к указателю на фукнкцию.
```cpp
struct gt {
	static bool gtf(int x, int y) { return x > y; }
	using gtfptr_t = bool (*)(int, int);
	operator gtfptr_t() const { return gtf(x, y); }
};

// гораздое лучше: теперь возможна подстановка
std::sort(myarr.begin(), myarr.end(), gt{});
```
• Увы, это жутковато выглядит и плохо расширяется.
Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Effective quick sort with cast operator
//
//---------------------------------------------------------------------------

#include <algorithm>
#include <cassert>
#include <cstdio>
#include <cstring>
#include <ctime>

struct myless {
	static bool less(const int &lhs, const int &rhs) { return lhs > rhs; }
	using lessptr_t = bool (*)(const int &lhs, const int &rhs);
	operator lessptr_t() const { return less; }
};

int main(int argc, char **argv) {
	size_t nelts, i;
	int *narr;
	clock_t startm fin;
	
	printf("Hello from C++ qsort bench cast overload\n");
	
	if(argc != 2) {
		printf("usage: %s <number-of-elements>\n", argv[0]);
		abort();
	}
	
	nelts = strtol(argv[1], NULL, 10);
	assert(nelts > 0);
	
	narr = (int *)calloc(nelts, sizeof(int));
	assert(narr);
	
	for(i = 0; i < nelts; ++i)
		narr[i] = rand() % nelts;
	
	start = clock();
	
	std::sort(narr, narr + nelts, myless{});
	
	fin = clock();
	printf("Elapsed: %lg seconds\n", (double)(fin - start) / CLOCKS_PER_SEC);
}
```
Вывод:
```bash
Hello from C++ qsort bench cast overload
Elapsed: 2.76562 seconds
```
#### Функторы: перегрузка ()
• Более правильный способ сделать функтор - это перегрузка вызова
```cpp
struct gt {
	bool operator() (int x, int y) { return x > y; }
};

// всё так же хорошо
std::sort(myarr.begin(), myarr.end(), gt{});
```
• Почти всегда это лучше, чем указатель на функцию.
• Кроме того, в классе можно хранить состояние.
• Функторы с состоянием получат второе дыхание, когда мы дойдём до так называемых <span style="color: blue;">лямбда-функций</span>.
Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Effective quick sort with "operator ()"
//
//---------------------------------------------------------------------------

#include <algorithm>
#include <cassert>
#include <cstdio>
#include <cstring>
#include <ctime>

struct myless {
	bool operator() (int x, int y) { return x > y; }
};

int main(int argc, char **argv) {
	size_t nelts, i;
	int *narr;
	clock_t startm fin;
	
	printf("Hello from C++ qsort bench op overload\n");
	
	if(argc != 2) {
		printf("usage: %s <number-of-elements>\n", argv[0]);
		abort();
	}
	
	nelts = strtol(argv[1], NULL, 10);
	assert(nelts > 0);
	
	narr = (int *)calloc(nelts, sizeof(int));
	assert(narr);
	
	for(i = 0; i < nelts; ++i)
		narr[i] = rand() % nelts;
	
	start = clock();
	
	// myless comp
	std::sort(narr, narr + nelts, myless{});
	
	fin = clock();
	printf("Elapsed: %lg seconds\n", (double)(fin - start) / CLOCKS_PER_SEC);
}
```
Вывод:
```bash
Hello from C++ qsort bench cast overload
Elapsed: 2.84375 seconds
```
Пример с лямбдой:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Effective quick sort with "operator ()"
//
//---------------------------------------------------------------------------

#include <algorithm>
#include <cassert>
#include <cstdio>
#include <cstring>
#include <ctime>

int main(int argc, char **argv) {
	size_t nelts, i;
	int *narr;
	clock_t startm fin;
	
	printf("Hello from C++ qsort bench op overload\n");
	
	if(argc != 2) {
		printf("usage: %s <number-of-elements>\n", argv[0]);
		abort();
	}
	
	nelts = strtol(argv[1], NULL, 10);
	assert(nelts > 0);
	
	narr = (int *)calloc(nelts, sizeof(int));
	assert(narr);
	
	for(i = 0; i < nelts; ++i)
		narr[i] = rand() % nelts;
	
	start = clock();
	
	std::sort(narr, narr + nelts, [](auto lhs, auto rhs) { return lhs > rhs; });
	
	fin = clock();
	printf("Elapsed: %lg seconds\n", (double)(fin - start) / CLOCKS_PER_SEC);
}
```
#### Идиома PImpl
• Идиома  PImpl предполагает единичное владение
```cpp
class Ifacade {
	CImpl *impl_;
public:
	Ifacade() : impl_(new CImpl) {}
	// методы
};
```
• Эта идиома очень полезна: в частности, она позволяет всегда иметь обект класса одного и того же размера, что может быть очень важно в ABI.
• Хорошей ли идеей является здесь заменить CImpl * на unique_ptr?
#### Проблема неполного типа
• Попробуем использовать unique pointers в PImpl.
```cpp
class MyClass; // предварительное объявление

struct MyWrapper {
	MyClass *c; // это ок
	MyWrapper() : c(nullptr) {};
};

struct MySafeWrapper {
	unique_ptr<MyClass> c; // увы, не компилируется
	MySafeWrapper() : c(nullptr) {};
};
```
#### Как реально выглядит unique_ptr?
• <span style="color: blue;">Стратегия</span> удаления у него вынесена в параметр шаблона.
```cpp
template <typename T, typename Deleter = default_delete<T>>
class unique_ptr {
	T *ptr_; Deleter del_;
	
public:
	unique_ptr(T *ptr = nullptr, Deleter del = Deleter()) : ptr_(ptr), del_(del) {}
	
	~unique_ptr() { del_(ptr_); }
	// и так далее
}
```
• Как мог бы выглядеть default_delete?
#### Дефолтный удалитель
• Разумеется, по дефолту - это пустой класс с перегруженными круглыми скобками.
```cpp
template <typename T> struct default_delete {
	void operator() (T *ptr) { delete ptr; }
};
```
• Теперь давайте вернёмся к исходной проблеме:
```cpp
class MyClass; // предварительное объявление

struct MySafeWrapper {
	unique_ptr<MyClass> c; // увы, не компилируется
	MySafeWrapper() : c(nullptr) {};
};
```
#### Решение: пользовательский делетер
• Внезапно нам помогает пользовательский удалитель.
```cpp
class MyClass; // предварительное объявление

struct MyClassDeleter {
	void operator()(MyClass *); // определён где-то ещё
};

struct MySafeWrapper {
	unique_ptr<MyClass, MyClassDeleter> c;
	MySafeWrapper() : c(nullptr) {}; // ok
};
```
• В данном случае проблема была в том, что delete проникает в хедер при использовании стандартного удалителя.
#### Обсуждение: unique void pointer
• Вспомним про рассмотренный ранее unique pointer.
• Может ли он работать как умный void pointer для стирания типов?
• В чистом виде это невозможно даже скомпилировать.
```cpp
unique_ptr<void> u;
```
• Можно ли разумно модифицировать это определение?
#### Тизер: влияние на размеры
• Как вы думаете, влияет ли на размер необходимость хранить удалитель?
#### Обсуждение
• Вернёмся к базовой арифметике.
• Итак, мы умеем определять унарные плюс/минус.
• Какие ещё арфиметические операторы в языке вы можете вспомнить?
#### Источник названия языка
• Язык C++ получил название от операции ++ (постинкремента).
• Бывает также преинкремент:
```cpp
int x = 42, y, z;
y = ++x; // y = 43, x = 43
z = y++; // z = 43, y = 44
```
• Для их переопределения используется один и тот же operator++.
```cpp
Quat<T>& Quat<T>::operator++(); // это pre-increment
Quat<T> Quat<T>::operator++(int); // это post-increment
```
• Дополнительный аргумент в постинкременте липовый.
• Обычно постинкремент делается в термина преинкремента.
```cpp
template<typename T> struct Quat {
	T x_, y_, z_, w_;
	
	Quat<T>& Quat<T>::operator++() { x_ += 1; return *this; }
	
	Quat<T> Quat<T>::operator++(int) {
		Quat<T> tmp {*this};
		++(*this);
		return tmp;
	}
};
```
• Разумеется точно так же работает декремент и постдекремент.
#### Обсуждение: немного джигитовки
• Признак новичка - это "неэффективный" обход контейнера.
```cpp
using itt = typename my_container<int>::iterator;

for(itt it = cont.begin(); it != cont.end(); it++) {
	// do something
}
```
• Профессионал использует преинкремент и не будет делать вызовов в проверке условия.
```cpp
for(itt it = cont.begin(), ite = cont.end(); it != ite; ++it) {
	// do something
}
```
#### Цепочечные операторы
• Операторы, образующие цепочки имеют вид op=.
```cpp
int a = 3, b = 4, c = 5;
a += b *= c -= 1; // чему теперь равны a, b, c?
```
• Все они правоассоциативны.
• Исключение составляют очевидные бинарные >= и <=.
• Все они модифицируют свою правую часть, и их место внутри класса в качестве методов.
• Например, для кватернионов.
```cpp
struct Quat {
	int x, y, z, w'
	
	Quat& operator+=(const Quat& rhs) {
		x += rhs.x; y += rhs.y; z += rhs.z; w += rhs.w;
		return *this;
	}
};
```
• Здесь возврат ссылки на себя нужен, чтобы организовать цепочку
```cpp
a += b *= c; // a.operator+=(b.operator*=(c));
```
#### Определение через цепочки
• Чем плоха идея теперь определить в классе и оператор +?
```cpp
struct Quat {
	int x, y, z, w'
	
	Quat& operator+=(const Quat& rhs);
	
	Quat operator+(const Quat& rhs) {
		Quat tmp(*this); tmp += rhs; return tmp;
	}
};
```
• Казалось бы, всё хорошо:
```cpp
Quat x, y; Quat t = x + y; // ok?
```
## Бинарные операторы
#### Неявные преобразования
• Часто мы хотим, чтобы работали неявные преобразования.
```cpp
Quat::Quat(int x);
Quat Quat::operator+(const Quat& rhs);

Quat t = x + 2; // ok, int -> Quat
Quat t = 2 + x; // FAIL
```
• Увы, метод класса не преобразует свой неявный аргумент.
• Единственный вариант делать настоящие бинарные операторы - это делать их вне класса.
Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Binary operator: symmetric for non-template class
//
//---------------------------------------------------------------------------

struct Quat {
	int x_, y_, z_, w_;
	
	// implicit cast int -> Quat
	Quat(int x = 0, int y = 0, int z = 0, int w = 0) : x_(x), y_(y), z_(z), w_(w)      {}
	
	// operator += in class
	Quat& operator+=(const Quat& rhs) {
		x_ += rhs.x_; y_ += rhs.y_; z_ += rhs.z_; w_ += rhs.w_;
		return *this;
	}
	
#ifndef NOFAIL
	// operator+ in class
	Quat operator+(const Quat& rhs) { Quat tmp(*this); tmp += rhs; return tmp; }
#endif
};

#ifdef NOFAIL
Quat operator+(Quat lhs, Quat rhs) { Quat tmp(lhs); tmp += rhs; return tmp; }
#endif

int main() {
	Quat q;
	Quat p = q + 1;
	Quat t = 1 + q; // FAIL
}
```
Если без NOFAIL, то вывод:
```bash
error: no match for 'operator+' (operand types are 'int' and 'Quat')
```
Под NOFAIL всё, очевидно, работает.
```cpp
Quat::Quat(int x);
Quat operator+(const Quat& lhs, const Quat& rhs);

Quat t = x + 2; // ok, int -> Quat rhs
Quat t = 2 + x; // ok, int -> Quat lhs
```
#### Определение через цепочки
• Это не мешает использовать для определения бинарных операторов цепочечные с соответствующими аргументами.
```cpp
Quat operator+(const Quat& x, const Quat& y) {
	Quat tmp {x};
	tmp += y;
	return tmp;
}
```
• Это логично и позволяет переиспользовать код.
• Кроме того, такой оператор может не быть friend и действовать в терминах открытого интерфейса.
#### Призыв к осторожности
• Одновременное наличие implicit ctors и внешних операторов может вызывать странные эффекты.
```cpp
struct S {
	S(std::string) {}
	S(std::wstring) {}
};

bool operator==(S lhs, S rhs) { return true; }

assert (std::string{"foo"} == std::wstring{L"bar"}); // WAT?
```
• В таких случаях стоит рассмотреть возможность занести сравнение внутрь и сделать его friend.
Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Strange implicit cast behaviour
//
//---------------------------------------------------------------------------

#include <iostream>
#include <string>

struct S {
	S(std::string) {}
	S(std::wstring) {}
};

bool operator==(S lhs, S rhs) { return true; }

int main() {
	std::string s = "foo";
	std::wstring t = L"bar";
	if(s == t)
		std::cout << "WAT?\n";
}
```
Вывод, очевидно:
```bash
WAT?
```
#### Одна небольшая проблема
• Увы, это не работает для шаблонов.
```cpp
template<typename T>
Quat<T> operator+ (const Quat<T>& x, const Quat<T>& y) {
	Quat<T> tmp {x};
	tmp += y;
	return tmp;
}
```
• Такой оператор будет скорее всего иметь проблемы с подстановкой типов.
• Потому что преобразование не работает через шаблонную подстановку.
Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Binary operator for templated class: not so easy
//
//---------------------------------------------------------------------------

template<typename T> struct Quat {
	T x_, y_, z_, w_;
	
	// implicit cast T -> Quat
	Quat(T x = 0, T y = 0, T z = 0, T w = 0) : x_(x), y_(y), z_(z), w_(w) {}
	
	// operator += in class
	Quat& operator+=(const Quat& rhs) {
		x_ += rhs.x_; y_ += rhs.y_; z_ += rhs.z_; w_ += rhs.w_;
		return *this;
	}
	
#ifndef NOFAIL
	// operator+ in class
	Quat operator+(const Quat& rhs) { Quat tmp(*this); tmp += rhs; return tmp; }
#endif
};

#ifdef NOFAIL
template<typename T>
Quat<T> operator+ (const Quat<T>& lhs, const Quat<T>& rhs) {
	Quat<T> tmp(lhs);
	tmp += rhs;
	return tmp;
}
#endif

int main() {
	Quat<int> q;
	Quat<int> p = q + 1;
	Quat<int> t = 1 + q; // FAIL anyway
}
```
Вывод без NOFAIL:
```bash
error: no match for 'operator+' (operand types are 'int' and 'Quat<int>')
```
Вывод с NOFAIL - огромная портянка SFINAE:
```bash
error: no match for 'operator+' (operand types are 'Quat<int>' and 'int')
...
error: no match for 'operator+' (operand types are 'int' and 'Quat<int>')
```
Решение. Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Binary operator for templated class: something like a solution
//
//---------------------------------------------------------------------------

template<typename T> struct Quat {
	T x_, y_, z_, w_;
	
	// implicit cast T -> Quat
	Quat(T x = 0, T y = 0, T z = 0, T w = 0) : x_(x), y_(y), z_(z), w_(w) {}
	
	// operator += in class
	Quat& operator+=(const Quat& rhs) {
		x_ += rhs.x_; y_ += rhs.y_; z_ += rhs.z_; w_ += rhs.w_;
		return *this;
	}
	
#ifndef NOFAIL
	// operator+ in class
	Quat operator+(const Quat& rhs) { Quat tmp(*this); tmp += rhs; return tmp; }
#endif
};

#ifdef NOFAIL
template<typename T>
Quat<T> operator+(const Quat<T>& lhs, const Quat<T>& rhs) {
	Quat<T> tmp(lhs);
	tmp += rhs;
	return tmp;
}

template<typename T>
Quat<T> operator+(T lhs, const Quat<T>& rhs) {
	Quat<T> tmp(lhs);
	tmp += rhs;
	return tmp;
}

template<typename T>
Quat<T> operator+(const Quat<T>& lhs, T rhs) {
	Quat<T> tmp(lhs);
	tmp += rhs;
	return tmp;
}
#endif

int main() {
	Quat<int> q;
	Quat<int> p = q + 1;
	Quat<int> t = 1 + q; // FAIL anyway
}
```
#### Сравнение для basic_string
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
• <span style="color: blue;">Должен ли</span> оператор сложения действительно складывать?
• <span style="color: blue;">Должен ли</span> он быть согласован с цепочечным +=?
• Увы, на оба вопроса правильный ответ - нет.
• Хорошим тоном является поддерживать консистентную семантику, но никто не заставляет делать это.
• В языках с перегрузкой операторов вы никогда не можете быть уверены, что делает сложение сегодня утром.
• Поэтому во многих языках этой опции сознательно нет.
#### Интермедия: невезучий сдвиг
• Меньше всего повезло достойному бинарному оператору сдвига.
```cpp
int x = 0x50;
int y = x << 4; // y = 0x500
x >>= 4; // x = 0x5
```
• У него, как видите, даже есть цепочечный эквивалент.
• Но сейчас де-факто принято в языке использовать его для ввода и вывода на потому и именно в бинарной форме.
```cpp
std::cout << x << " " << y << std::endl;
std::cin >> z;
```
• Обычно сдвиг делают всё-таки вне класса, используя внутренний дамп.
```cpp
template<typename T> struct Quat {
	T x, y, z, w;
	void dump(std::ostream& os) const {
		os << x << " " << y << " " << z << " " << w;
	}
}
```
• И далее, собственно, оператор (тут не лучшая его версия).
```cpp
template<typename T>
std::ostream& operator<<(std::ostream& os, const Quat<T>& q) {
	q.dump(os); return os;
}
```
#### Обсуждение
• А что насчёт сигнатуры?
• Она, хотя бы, должна быть правильной?
• С точностью до количества аргументов. У бинарного оператора - это
• (a).operatorX (b)
• operatorX (a, b)
• У оператора присваивания и некоторых других есть только первая форма.
• С точки зрения языка и operator=, и operator+, и operator+= - это независимые бинарные операторы. По сути, просто разные методы.
#### Проблемы определения через цепочки
• Для матриц не всё так красиво:
```cpp
template<typename T> class Matrix {
	// .....
	Matrix &operator+=(const Matrix& rhs);
};

Matrix operator+(const Matrix& lhs, const Matrix& rhs) {
	Matrix tmp{lhs}; tmp += rhs; return tmp;
}
```
• Здесь создаётся довольно дорогой временный объект.
```cpp
Matrix x = a + b + c + d; // а здесь трижды
```
#### Обсуждение
• Должны ли мы сохранять основные математические свойства операций?
• Например, умножение для всех встроенных типов коммутативно.
• Имеет ли смысл тогда переопределять operator* для матриц?
• Или оставить его только для умножения матрицы на число?
#### Сравнения как бинарные операторы
• В чём отличие следующих двух способов сравнить кватернионы?
```cpp
// 1
template<typename T>
bool operator== (const Quat<T>& lhs, const Quat<T>& rhs) {
	return (&lhs == &rhs);
}

// 2
template<typename T>
bool operator== (const Quat<T>& lhs, const Quat<T>& rhs) {
	return (lhs.x == rhs.x) && (lhs.y == rhs.y) &&
	       (lhs.z == rhs.z) && (lhs.w == rhs.w);
}
```
#### Равенство и эквивалентность
• Базовая эквивалентность объектов означает, что их адреса равны (то есть, это **один и тот же объект**).
• Равенство через operator== может работать сколько угодно сложно.
```cpp
bool operator== (const Foo& lhs, const Foo& rhs) {
	bool res;
	std::cout << lhs << " vs " << rhs << "?" << std::endl;
	std::cin >> std::boolalpha >> resl
	return res;
}
```
• Это, конечно, крайний случай, но почему нет.
• Считается, что хороший оператор равенства удовлетворяет трём основным соотношениям.
```cpp
assert(a == a);
assert((a == b) == (b == a));
assert((a != b) || ((a == b) && (b == c)) == (a == c));
```
• Первое - это рефлексивность, второе - симметричность, третье - транзитивность.
• Говорят, что обладающие такими свойствами соотношения являются <span style="color: blue;">отношениями эквивалентности</span>.
#### Дву и три валентные сравнения
• В языке C приняты тривалентные сравнения:
```c
strcmp(p, q); // returns -1, 0, 1
```
• В языке C++ приняты двувалентные сравнения:
```cpp
if(p > q) // if(strcmp(p, q) == 1)
if(p >= q) // if(strcmp(p, q) != -1)
```
• Кажется, из одного тривалентного сравнения <=> можно соорудить все двухвалентные.
Пример на godbolt:
```cpp
#include <compare>
#include <iostream>

int main() {
	int a = 4, b = 4, c = 5;
	auto ac = (a <=> c);
	std::cout << std::boolalpha;
	std::cout << (ac < 0) << std::endl;
	std::cout << ((a <=> b) == 0) << std::endl;
	std::cout << ((c <=> a) <= 0) << std::endl;
}
```
Компилится. Вывод:
```bash
true
true
false
```
#### Spaceship operator
• В 2020 году в C++ появился перегружаемый "оператор летающая тарелка".
```cpp
struct MyInt {
	int x_;
	MyInt(int x = 0) : x_(x) {}
	std::strong_ordering operator<=>(const MyInt &rhs) {
		return x_ <=> rhs.x_;
	}
};
```
• Такое определение MyInt сгенерирует все сравнения кроме равенства и неравенства (потому что он не сможет решить, какое вы хотите равенство).
Ещё пример на godbolt:
```cpp
#include <compare>
#include <iostream>

struct MyInt {
	int x_;
	MyInt(int x = 0) : x_(x) {}
	std::strong_ordering operator<=>(const MyInt &rhs) {
		return x_ <=> rhs.x_;
	}
};

int main() {
	MyInt a = 5, b = 6;
	std::cout << (b > a) << std::endl;
	std::cout << (b <= a) << std::endl;
}
```
• Самое важное - это концепция упорядочения.
```cpp
struct S {
	ordering type operator<=>(const S& that) const;
};
```
• Всего доступны три вида упорядочения:

| Тип упорядочения      | Равные значения | Несравнимые значения |
| --------------------- | --------------- | -------------------- |
| std::strong_ordering  | Неразличимы     | Невозможны           |
| std::weak_ordering    | Различимы       | Невозможны           |
| std::partial_ordering | Различимы       | Возможны             |
#### Defaulted spaceship operator
• Летающая тарелка - это один из немногих примеров осмысленного умолчания.
```cpp
struct MyInt {
	int x_;
	MyInt(int x = 0) : x_(x) {}
	auto operator<=>(const MyInt &rhs) = default;
};
```
• Сгенерированный по умолчанию (изо всех полей класса), он сам определяет упорядочение и, как бонус, определяет также равенство и неравенство.
• Логика тут такая: если вы генерируете всё по умолчанию, то вы <span style="color: blue;">точно не хотите от равенства ничего необычного</span>.
## Экзотика
#### Взятие адреса
• Может быть перегружено так же, как разыменование.
```cpp
template <typename T> class scoped_ptr {
	T *ptr_;
public:
	scoped_ptr(T *ptr) : ptr_{ptr} {}
	~scoped_ptr() { delete ptr_; }
	T** operator&() { return &ptr_; }
	T operator*() { return *ptr; }
	T* operator->() { return ptr; }
}
```
• В реальности перегружается редко.
#### Обсуждение
• А что если мне и правда нужен именно адрес объекта, а у него, как назло, перегружен оператор взятия адреса?
Пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// example of explicit adressoff
//
//---------------------------------------------------------------------------

#include <iostream>

struct MyInt {
	int x_;
	MyIn(int x = 0) : x_(x) {}
	int operator&() { return 42; } // because I can
};

int main() {
	MyInt a = 5;
	std::cout << "Seemingly " << &a << " bur really " << std::adressoff(a)
            << std::endl;
}
```
Вывод:
```bash
Seemingly 42 but really 0x7fffe11cb364
```
#### Ограничения
• Операторы разыменования (\*) и разыменования с обращением (->) обязаны быть методами. Они не могут быть свободными функциями, как и operator=.
• Какие последствия могло бы иметь разрешение перегружать их как не-методы?
• Очень интересно, что это не относится к разыменованию с обращеним по указателю на метод (->\*).
• Кстати, а что это такое?
#### Указатели на методы классов
• Имеет ли смысл выражение "указатель на нестатический метод"?
```cpp
struct MyClass { int DoIt(float a, int b) const; };
```
• Казалось бы, нет.
• Как мы уже говорили, метод <span style="color: blue;">частично</span> ведёт себя <span style="color: blue;">как будто</span> это функция вроде
```cpp
int DoIt(MyClass const *this, float a, int b);
```
• И на такую функцию возможен указатель. Но метод класса <span style="color: blue;">не является</span> этой функцией.
• Например, в точке вызова на него должны распространяться соображения времени жизни и контроля доступа. <span style="color: red;">Вызов через подобный указатель на функцию кажется возможностью нарушить инкапсуляцию</span>.
• Однако, смысл это выражение имеет:
```cpp
using constif_t = int (MyClass::*)(float, int) const;
```
• Поддерживается два синтаксиса вызова.
```cpp
constif_t ptr = &MyClass::DoIt;
MyClass c; (c.*ptr)(1.0, 1);
MyClass *pc = &c; (pc->ptr)(1.0, 1);
```
• И второй из них даже перегружается.
#### Волшебные свойства ->*
• Оператора ->* примечателен свои никаких приоритетом и никакими требованиями к перегрузке.
• Как следсвтие, его где только не используют (привдённый ниже пример чуточку безумный).
```cpp
template <typename T> T& operator->*(pair<T,T> &l, bool r) {
	return r ? l.second : l.first;
}

pair<int, int> y {5, 6};
y ->* false = 7;
```
#### Оператор запятая
• Малоизвестен, но встречается оператор запятая.
```cpp
for (int i = 0, j = 0; (i + j) < 10; i++, j++) { use(i, j); }
```
• Например, он работает в приведённом цикле.
• Оператор имеет общий вид
```cpp
result = foo(), bar();
```
• Здесь <span style="color: blue;">выполняется foo, потом bar</span>, потом в result записывается результат bar.
```cpp
buz(1, (2, 3), 4); // вызовет buz(1, 3, 4)
```
• Удивительно, но этот оператор тоже перегружается. Этого никогда не следует делать, потому что вы потеряет sequencing.
#### Интермедия: sequencing
• Выражения, разделённые точкой с запятой состоят в <span style="color: blue;">отношениях последования</span> sequenced-after и sequenced-before.
```cpp
foo(); bar(); // foo sequenced before bar
```
• Но увы, вызов функции не определяет sequencing.
```cpp
buz(foo(), bar()); // no sequencing between foo and bar
```
• Почему это так важно? Потому что unsequenced modification - это UB-case.
```cpp
y = x++ + x++; // operator++ and operator++ unsequenced
```
• В этом примере компилятор имеет право отформатировать жёсткий диск. Он вряд ли это сделает, но ситуация неприятная.
#### Что нельзя перегрузить
• Доступ через точку a.b
• Доступ к члену класса через точку a.\*b
• Доступ к пространству имён a::b
• Последовательный доступ a ; b
• <span style="color: red;">Почти все</span> специальные операторы. В том числе, sizeof, alignof, typeid.
	• Правило такое: если вы видите специальный оператор, <span style="color: red;">скорее всего</span>, его нельзя перегрузить.
	• Сюда же относятся: static_cast и его друзья.
• Тернарный оператор a ? b : c
#### Что не следует перегружать
• Данные логические операции && и ||, потому что они теряют сокращённое поведение.
```cpp
if(p && p->x) // может взорваться, если && перегружено
```
• Запятую, чтобы не потерять sequencing (допустим, в примере ниже foo инициализирует данные, которые использует bar).
```cpp
x = foo(), bar(); // может взорваться, если , перегружена
```
• Унарный плюс, чтобы не потерять positive hack.
#### И это ещё не все
• Фундаментальную роль в языке играют операторы работы с памятью и их перегрузка: мы по ряду причин пока ничего не сказали про operator new, operator delete и прочие прекрасные вещи.
• Также по ряду причин на будущее отложено обсуждение оператора "" нужного для <span style="color: blue;">пользовательских литералов</span>.
• Начиная с C++20, можно также перегрузить оператор co_await.
#### Литература
• [CC11] ISO/IEC 14882 - "Information technology - Programming languages - C++", 2020
• [BS] Bjarne Stroustrup - The C++ Programming Language (4th Edition), 2013
• [GB] Grady Booch - Object-Oriented Analysis and Design with Applications, 2007
• [JG] Joshua Gerrard - The dangers of C-style casts, CppCon, 2015
• [BD] Ben Deane - Operator Overloading: History, Principles and Practice, CppCon, 2018
• [JM] Jonathan Muller - Using C++20's Three-way Comparison <=>, CppCon, 2019
• [TW] Titus Winters - Modern C++ Design, CppCon, 2018
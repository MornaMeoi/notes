<h1 align="center">ИСКЛЮЧЕНИЯ</h1>

---
<p align="center">Механизмы нелокальной обработки ошибок. Гарантии безопасности. Проектирование с учётом исключений.</p>
## Ошибки и исключения
#### Обработка ошибок в стиле C
• Определяется область целочисленных кодов ошибок:
```c
enum error_t { E_OK = 0, E_NO_MEM, E_UNEXPECTED };
```
• Как функция сигнализирует, что результат её исполнения - это E_OK?
• Вернёт код ошибки
```c
error_t open_file(const char *name, FILE **handle);
```
• Использует thread-local facility. Например, errno/GetLastError.
```c
FILE *open_file(const char *name);
```
• Вернёт error_t* в списке параметров
```c
FILE *open_file(const char *name, error_t *errcode);
```
#### Проблемы уже в C
• Замечательно стандартная функция
```c
int atoi(const char *nptr);
```
• В случае, если конвертировать невозможно, возвращает 0.
	• <span style="color: blue;">Действительно ли возвращать ноль - хорошая идея?</span>
• В случае, если число слишком большое, возвращает HUGE_VAL и устанавливает errno = ERANGE.
	• <span style="color: blue;">Часто ли вы проверяли возврат на HUGE_VAL?</span>
#### Проблема в C++
```cpp
template <typename T> class MyVector {
	T *arr_ = nullptr;
	size_t size_, used_ = 0;

public:
	MyVector(size_t sz) : size_(sz) {
		arr_ = static_cast<T*>(malloc(sizeof(T) * sz_));
	}
// .... тут всё остальное ....
};
```
• Вы видите, в чём проблема в этом коде?
```cpp
MyVector(size_t sz) : size_(sz) {
	arr_ = static_cast<T*>(malloc(sizeof(T) * sz_));
	// тут должна быть обработка случая arr_ == nullptr
}
```
• Не обработана ситуация, когда malloc возвращает nullptr.
#### Чем нам грозит эта ситуация?
```cpp
MyVector v(100);
// тут объект v может оказаться в несогласованном состоянии
// v.arr_ = 0 т.к. память кончилась
// v.size_ = 100 т.к. конструктор никак не обработал ошибку
```
• Хуже всего то, что объект в несогласованном сотоянии никак не отличается от нормального объекта.
• Несогласованность может проявиться через тысячи строк кода.
• Это даже не UB. Несогласованное состояние вполне корректно.
#### Попытка решения: iostream style
```cpp
template <typename T> class MyVector {
	T *arr_ = nullptr;
	size_t size_, used = 0;
	bool valid_ = true;

public:
	MyVector(size_t sz) : size_(sz) {
		arr_ = static_cast<T*>(malloc(sizeof(T) * sz));
		if(!arr_) valid_ = false;
	}
	
	bool is_valid() const { return valid_; }
// .... и так далее ....
};
```
#### Обсуждение
• Покритикуйте решение в стиле потоков ввода-вывода.
```cpp
MyVector v(1000);

if(!v.is_valid())
	return -1;

// здесь используем v
```
• Кому оно нравится?
#### Копирование и присваивание
• Похоже, такой вектор тяжело использовать.
```cpp
MyVector v(1000); assert(v.is_valid());
MyVector v2(v); assert(v2.is_valid());
v2.push_back(3); assert(v2.is_valid());
v = v2; assert(v.is_valid());
```
• Есть ли идеи получше?
#### Перегрузка операторов
• Делаеть вещи ещё хуже.
```cpp
Matrix operator+(Matrix a, Matrix b);
```
• Здесь неоткуда вернуть код возврата.
• И поскольку это отдельная функция, здесь негде сохранить  goodbit.
• Конечно, мы всё ещё можем вернуть errno. Кому нравится идея его проверять в таких случаях?
#### Основная идея решения
• Выйти из вызванной функции в вызывающий код в обход обычных механизмов возврата управления.
• Аннотировать этот <span style="color: blue;">нелокальный</span> выход информацией о случившемся.
• Но что вообще мы знаем о нелокальных переходах?
#### Типы передачи управления
• Локальная передача управления
	• условные операторы
	• циклы
	• локальный goto
	• прямой вызов функций и возврат из них
• Нелокальная передача управления
	• косвенный вызов функций (напр. по указателю)
	• возобновление/приостановка сопрограммы
	• <span style="color: blue;">исключения</span>
	• переключение контекста потоков
	• нелокальный longjmp и вычисляемый goto
#### Исключения
• Исключительные ситуации уровня аппаратуры (например, undefined instruction exception).
• Исключительные ситуации уровня операционной системы (например, data page fault).
• Исключения C++ (только они и будут нас далее интересовать).
#### Исключительные ситуации
• Ошибки (исключительными ситуациями не являются)
	• рантайм ошибки, после которых состояние не восстановимо (например, segmentation fault)
	• ошибки контракта функции (assertion failure из-за неверных аргументов, невыполненные предусловия вызова)
• Исключительные ситуации
	• Состояние программы должно быть восстановимо (например: исчерпание памяти или отсутствие файла на диске)
	• Исключительная ситуация не может быть обработана на том уровне, на котором возникла (программа сортировки не обязана знать, что делать при нехватке памятина временный буфер)
#### Порождение ошибки
```cpp
struct UnwShow {
	UnwShow() { cout << "ctor\n"; }
	~UnwShow() { cout << "dtor\n"; }
};

int foo(int n) {
	UnwShow s;
	if(n == 0) abort(); // abort - это убийство
	foo(n - 1);
}

foo(4); // что на экране?
```
На экране 5 выводов конструктора.
#### Порождение исключения
```cpp
struct UnwShow {
	UnwShow() { cout << "ctor\n"; }
	~UnwShow() { cout << "dtor\n"; }
};

int foo(int n) {
	UnwShow s;
	if(n == 0) throw 1;
	foo(n - 1);
}

// вызов внутри try-блока
foo(4); // что на экране?
```
Теперь на экране 5 конструкторов, 5 деструкторов.
Полный пример:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Demonstration of stack unwinding
//
//---------------------------------------------------------------------------

#include <iostream>

struct UnwShow {
	int n_;
	long t = 0xDEADBEEF;
	UnwShow(int n) : n_(n) { std::cout << "ctor: " << n_ << "\n"; }
	~UnwShow() { std::cerr << "dtor: " << n_ << "\n"; }
};

void foo(int n) {
	UnwShow s{n};
	
	// odr-use to materialize
	std::cout << "Addr of " << n << ": " << &s << std::endl;
	if(n == 0) {
		std::cout << "throw\n";
		throw 1;
	}
	foo(n - 1);
}

int main() {
	try {
		foo(5);
	} catch(int n) {
		std::cout << "catch\n";
	}
}
```
Компиляция показала то же самое. Помимо этого, видно ещё, что адреса стека растут вниз. А ещё см. ниже:
#### Раскрутка стека
![[8.1.png]]
#### Больше про throw
• Конструкция <span style="color: blue;">throw &ltexpression&gt</span> означает следующее:
	• Создать объект исключения
	• Начать размотку стека
• Примеры:
```cpp
throw 1;
throw new int(1);
throw MyClass(1, 1);
```
• Исключения отличаются от ошибок тем, что их нужно <span style="color: blue;">ловить</span>.
#### Ловля исключений
• Производится внутри <span style="color: blue;">try</span>. блока
```cpp
int divide(int x, int y) {
	if(y == 0) throw OVF_ERROR; // это так себе идея
	return x / y;
}

// где-то далее:

try {
	c = divide(a, b);
} catch (int x) {
	if(x == OVF_ERROR) std::cout << "Overflow" << std::endl;
}
```
#### Некоторые правила
• Ловля происходит по точному типу:
```cpp
try { throw 1; } catch(long l) {} // не поймали
```
• Или по ссылке на точный тип:
```cpp
try { throw 1; } catch(const int &ci) {} // поймали
```
• Или по указателю на точный тип:
```cpp
try { throw new int(1); } catch(int *pi) {} // поймали
```
• Или по ссылке или указателю на базовый класс:
```cpp
try { throw Derived(); } catch(Base &b) {} // поймали
```
Пример:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Demonstration of different throws
//
//---------------------------------------------------------------------------

#include <iostream>

struct Base {
	virtual ~Base() {}
};

struct Derived : Base {};

enum MyErrs { MY_OK, MYFAL };

enum class MyErrsC : int { MY_OK, MYFAL };

int main() {
	try {
		MyErrs m = MY_OK;
		throw m;
	} catch (int) {
		std::cout << "int\n";
	} catch (MyErrs) {
		std::cout << "MyErrs\n"; // ловим тут
	}
	
	try {
		MyErrsC m = MyErrsC::MY_OK;
		throw m;
	} catch (int) {
		std::cout << "int\n";
	} catch (MyErrsC) {
		std::cout << "MyErrsC\n"; // ловим тут
	}
	
	try {
		throw 1;
	} catch (long) {
		std::cout << "long\n";
	} catch (int) {
		std::cout << "int\n"; // ловим тут
	}
	
	try {
		throw 1;
	} catch (const double &) {
		std::cout << "const double &\n";
	} catch (const int &) {
		std::cout << "const int&\n"; // ловим тут
	}
	
	try {
		throw 1;
	} catch (const int &) {
		std::cout << "first\n"; // ловим тут
	} catch (int) {
		std::cout << "second\n";
	}
	
	try {
		throw 1;
	} catch (int) {
		std::cout << "new first\n"; // ловим тут
	} catch (const int&) {
		std::cout << "new second\n";
	}
	
	try {
		throw Derived();
	} catch (Base) {
		std::cout << "Base sliced\n"; // ловим тут
	}
	
	try {
		throw Derived();
	} catch (Base &) {
		std::cout << "Base ref\n"; // ловим тут
	}
	
	try {
		throw new Derived();
	} catch (Base *b) {
		std::cout << "Base ptr\n"; // ловим тут
		delete b;
	}
}
```
• Catch-блоки пробуются в порядке перечисления:
```cpp
try { throw 1; }
catch(long l) {} // не поймали
catch(const int &ci) {} // поймали
```
• Пойманную переменную можно менять или удалять.
```cpp
try { throw new Derived(); } catch(Base *b) { delete b; } // ok
```
• Пойманное исключение можно перевыбросить.
```cpp
try { throw Derived(); } catch(Base &b) { throw; } // ok
```
#### Обсуждение
• Чуть раньше был приведён следующий код для обработки ошибки переполнения:
```cpp
enum class errs_t { OVF_ERROR, UDF_ERROR, /*и так далее*/ };

int divide(int x, int y) {
	if(y == 0) throw errs_t::OVF_ERROR; // это так себе идея
	return x / y;
}
```
• Покритикуйте, что тут плохо?
• Как можно улучшить этот код?
• Очевидное улучшение: переход к классам исключений.
```cpp
class MathErr { /*информация об ошибке*/ };

class DivByZero : public MathErr { /*расширение*/ };

int divide(int x, int y) {
	if(y == 0) throw DivByZero("Division by zero occured");
	return x / y;
}

// где-то дальше
catch (MathErr &e) { std::cout << e.what() << std::endl; }
```
#### Некоторые неприятности
• Какие проблемы вы видите в этом коде?
```cpp
class MathErr { /*информация об ошибке*/ };
class Overflow : public MathErr { /*расширение*/ };

// где-то дальше
try {
	// тут много опасного кода
}
catch(MathErr e) { /* обработка всех ошибок */ } // slicing!
catch(Overflow o) { /* обработка переполнения */ }
```
Пример:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Demonstration of slicing inside exception handling
//
//---------------------------------------------------------------------------

#include <iostream>

using std::cout;
using std::endl;

struct Base {
	virtual ~Base() { cout << "~Base" << endl; }
};

struct Derived : Base {
	~Derived() { cout << "~Derived" << endl; }
};

int main() {
	try {
		throw Derived();
	}
#if defined(CORR)
	catch (Base &b) {
	}
#else
	catch (Base b) {
	}
#endif
}
```
Вывод без CORR:
```bash
~Base
~Derived
~Base
```
Вывод с CORR:
```
~Base
~Derived
```
#### Избегаем неприятностей
• Обсужденеи: какие <span style="color: red;">ещё</span> проблемы вы видите в этом коде?
```cpp
class MathErr { /*информация об ошибке*/ };
class Overflow : public MathErr { /*расширение*/ };

// где-то дальше
try {
	// тут много опасного кода
}
// 1. Правильный порядок: от частных к общим
// 2. Ловим строго по косвенности
catch(Overflow& o) { /* обработка переполнения */ }
catch(MathErr& e) { /* обработка всех ошибок */ }
```
#### Но как избежать самобытности?
• Теперь всё неплохо, но хм... неужели я первый, кто наткнулся на такие ошибки?
```cpp
class MathErr { /*информация об ошибке*/ };
class Overflow : public MathErr { /*расширение*/ };

// где-то дальше
try {
	// тут много опасного кода
}
// 1. Правильный порядок: от частных к общим
// 2. Ловим строго по косвенности
catch(Overflow& o) { /* обработка переполнения */ }
catch(MathErr& e) { /* обработка всех ошибок */ }
```
#### Стандартные классы исключений
![[8.2.png]]
![[8.3.png]]
#### Обсуждение
• Какой интерфейс вы бы сделали у std::exception?
```cpp
struct exception {
	exception() noexcept;
	exception(const exception&) noexcept;
	exception& operator=(const exception&) noexcept;
	virtual ~exception();
	virtual const char *what() const noexcept;
};
```
• Аннотация noexcept означает обещание, что эта функция не выбросил исключений.
• Она распространяется на определения виртуальных функций.
#### Используем стандартные классы
• Наследование от стандартного класса вводит расширение в иерархию
```cpp
class MathErr : public std::runtime_error { /* информация */ };
class Overflow : public MathErr { /* расширение */ };

// где-то дальше
try {
	// тут много опасного кода
}
catch (Overflow& o) { /* обработка переполнения */ }
catch (MathErr& e) { /* обработка всех ошибок */ }
```
• Впрочем, у наследования есть и тёмные стороны...
#### Множественное наследование
Пример:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Demonstration of problems with multiple inheritance
//
//---------------------------------------------------------------------------

#include <iostream>
#include <stdexcept>

struct my_exc1 : std::exception {
	char const* what() const noexcept override { return "exc1"; }
};

struct my_exc2 : std::exception {
	char const* what() const noexcept override { return "exc2"; }
};

struct your_exc3 : my_exc1, my_exc2 {};

int main() {
	try {
		throw your_exc3();
	} catch(std::exception const& e) {
		std::cout << e.what() << "\n";
	} catch(...) {
		std::cerr << "whoops!\n";
	}
}
```
Вывод:
```bash
whoops!
```
#### Перехват всех исключений
• Используется троеточие (как в printf).
```cpp
try {
	// тут много опасного кода
} catch (...) {
	// тут обрабатываются все исключения
}
```
• Сама идея, что можно как-то осмысленно обработать любое исключение очень сомнительна.
#### Нейтральность
• Функция называется нейтрально относительно исключений, если она не ловит чужих исключений.
• Хорошо написанная функций в хорошо спроектированном коде как минимум нейтральна.
![[../../../_Meta/attachments/8.4.png]]
#### Перевыброс
• Единственное разумное применение catch-all - это очистка критического ресурса и перевыброс исключения.
• На самом деле, даже разумность этого варианта под сомнением.
```cpp
int *critical = new int[10000]();
try {
	// тут много опасного кода
}
catch (...) {
	delete [] critical;
	throw;
}
```
• Кто-нибудь предложит лучше?
#### Обсуждение
• Кажется, есть одно место, где мы не можем поймать исключение.
```cpp
template <typename T> struct Foo {
	T x_, y_;
	Foo(int x, int y) : x_(x), y_(y) { // <-- exception in x_(x)
		try {
			// some actions
		}
		catch(std::exception& e) {
			// some processing
		}
	}
};
```
• С одной стороны, вроде и не нужно ловить. Или, может быть, нужно?
#### Try-блоки уровня функций
• Мы можем завернуть всю функцию в try-block.
```cpp
int foo() try { bar(); }
catch(std::exception& e) { throw; }
```
• В том числе и конструктор.
```cpp
Foo::Foo(int x, int y) try : x_(x), y_(y) {
	// some actions
}
catch(std::exception& e) {
	// some processing
}
```
• Техника скорее экзотическая, но лучше знать чем не знать.
#### Catch уровня функций
• На уровне функций, catch входит в scope функции.
```cpp
int foo(int x) try {
	bar();
}
catch(std::exception& e) {
	std::cout << x << ': ' << e.what() << std::endl; // ok
}
```
• Увы, но try-block на main не ловит исключения в конструкторах глобальных объектов.
#### Исключения для лучшего кода?
• <span style="color: blue;">Преимущества</span>:
	• Текст не замусоривается обработкой кодов возврата или errno, вся обработка ошибок отделена от логики приложения.
	• Ошибки не игнорируются по умолчанию. Собственно, они не могут быть проигнорированы.
• <span style="color: red;">Недостатки</span>:
	• Code path disruption - появление в коде неожиданных выходных дуг.
	• Некоторый оверхед на исключения.
## Гарантии безопасности
#### Вернёмся к исходной проблеме
```cpp
template <typename T> class MyVector {
	T *arr_ = nullptr;
	size_t size_, used_ = 0;
public:
	explicit MyVector(size_t sz) : size_(sz) {
		arr_ = static_cast<T*>(malloc(sizeof(T) * sz));
		if(!arr_) {
			// и что здесь делать?
		}
	}
// .... тут всё остальное ....
};
```
• Теперь <span style="color: blue;">вполне</span> ясно, как эта ошибка вообще может быть обработана.
```cpp
template <typename T> class MyVector {
	T *arr_ = nullptr;
	size_t size_, used_ = 0;
public:
	explicit MyVector(size_t sz) : size_(sz) {
		arr_ = static_cast<T*>(malloc(sizeof(T) * sz));
		if(!arr_) {
			throw std::bad_alloc();
		}
	}
// .... тут всё остальное ....
};
```
• Этот код можно упростить, так как по сути тут написан оператор new.
```cpp
template <typename T> class MyVector {
	T *arr_ = nullptr;
	size_t size_, used_ = 0;
public:
	// бросает bad_alloc
	explicit MyVector(size_t sz) : arr_(new T[sz]), size_(sz) {}
// .... тут всё остальное ....
};
```
• Задача: написать копирующий конструктор.
#### Пример Каргилла
• Все ли понимают, что тут плохо?
```cpp
template <typename T> class MyVector {
	T *arr_ = nullptr;
	size_t size_, used_ = 0;
	
public:
	MyVector(const MyVector &rhs) {
		arr_ = new T[rhs.size_]; // здесь утечка памяти
		size_ = rhs.size_; used_ = rhs.used_;
		for(size_t i = 0; i != rhs.size_; ++i) {
			arr_[i] = rhs.arr_[i]; // если здесь исключение
		}
	}
};
```
#### Безопасность относительно исключений
• Код, в котором при исключении могут утечь ресурсы, оказаться в несогласованном состоянии объекты и прочее, называется <span style="color: blue;">небезопасным</span> относительно исключений.
• Каргилл писал: "<span style="color: brown;">I suspect that most members of the C++ community vastly underestimate the skills needed to program with exceptions and therefore underestimate the true costs of their use.</span>" \[3]
• И, в общем, это до сих пор так, хотя прекрасные книги Саттера \[5] и \[6] сильно улучшили общую грамотность.
#### Гарантии безопасности
• Базовая гарантия: исключение при выполнении операции может изменить состояние программы, но не вызывает утечек и оставляет все объекты в согласованном (<span style="color: blue;">но не обязательно предсказуемом</span>) состоянии.
• Строгая гарантия: при исключении гарантируется <span style="color: blue;">неизменность состояния</span> программы относительно задействованных в операции объеков (commit/rollback).
• Гарантия бессбойности: функция не генерирует исключений (noexcept).
#### Безопасное копирование
```cpp
template <typename T>
T *safe_copy(const T* src, size_t srcsize) {
	T *dest = new T[srcsize];
	try {
		for(size_t idx = 0; idx != srcsize; ++idx)
			dest[idx] = src[idx];
	}
	catch (...) {
		delete [] dest;
		throw;
	}
	return dest;
}
```
#### Теперь конструктор копирования
```cpp
template <typename T> class MyVector {
	T *arr_ = nullptr;
	size_t size_, used_ = 0;
	
public:
	MyVector(const MyVector &rhs) {
		arr_ = (safe_copy(rhs.arr_, rhs.size_)),
		size_(rhs.size), used_(rhs.used_)
	}
};
```
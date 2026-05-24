<h1 align="center">КОНКРЕТНЫЕ КЛАССЫ</h1>

---
<p align="center">Конструкторы и деструкторы. Копирование и присваивание.</p>
## Имена и сущности
#### Одна забавная странность в языке C
• Функция strstr(haystack, needle) ищет подстроку needle в строке haystack.
• Она определена, мягко скажем, странно:
```c
char *strstr(const char* str, const char* substr);
```
• Почему аргументы const?
	• <span style="color: red;">Потому что иначе не будет работать передача const строк.</span>
• Почему если оба аргумента const, результат non-const?
	• <span style="color: red;">Потому иначе не будет работать возврат non-const строк.</span>
• При этом, мы сознательно жертвуем возвратом const строк. Омерзительно.
#### Обсуждение
• Как решить эту проблему?
• Пункт первый: разрешить в языке перегрузку функций.
• Пункт второй: перегрузить функции.
```cpp
char const * strstr(char const * str, char const * target);
char * strstr(char * str, char const * target);
```
• Теперь константность первого аргумента правильно согласована с константностью результата.
• Так и сделано в C++. Увы, этого нельзя сделать в C.
#### Гарантии по именам
• Язык C предоставляет строгие гарантии по именам.
```c
double sqrt(double); // метка не будет зависеть от сигнатуры
```
• Язык C++ не даёт гаранти по именам.
```cpp
double sqrt(double); // метка может зависеть от сигнатуры
```
• Кроме случая extern "C"
```cpp
extern "C" double sqrt(double); // то же, что и в C
```
• Последний случай введён, чтобы согласовать API.
• Процесс искажения имён называется <span style="color: blue;">манглированием.</span>.
Пример с гита:
```c
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Example of mangling for C
// compile with: g++ -g0 -O1 -masm=intel -S mangle.c
//
//---------------------------------------------------------------------------

extern int myfunc(const char *format, ...);

int foo(int x) {
	myfunc("%d\n", x); // will become myfunc@PLT
	return x;
}
```
Другой пример с гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Example of mangling for C++
// compile with: g++ -g0 -O1 -masm=intel -S mangle.cc
//
//---------------------------------------------------------------------------

struct S {
	int foo(int) __attribute__((noinline));
};

int foo(S *p, int x) { return p->foo(x); } // _Z3fooP1Si

int foo(int x) { return x; } // _Z3fooi

int S::foo(int x) { return x; } // _ZN1S3fooEi

extern "C" int bar(int x) { return x; } // bar (can not overload foo!)
```
#### Обсуждение
• Догадайтесь, можно ли делать вот так:
```cpp
extern "C" template <typename T> void foo(T x); // 1
struct S { extern "C" void foo(); };            // 2
```
• Обоснуйте свои догадки.
• Оба ответа: нельзя.
• Оба механизма с первой лекции: (1) обобщение данных и (2) объединение данных с методами невозможны без манглирования имён.
• Точно так же перегрузка функций невозможна без манглирования имён.
#### Разрешение перегрузки
• Наличие перегрузки вносит некоторые сложности.
```c
float sqrtf(float x);  // 1
double sqrt(double x); // 2
sqrtf(42); // вызовет 1, неявно преобразует int -> float
```

• В языке C нет перегрузки и нет проблем, программист всегда <span style="color: blue;">явно указывает</span>, какую функцию нужно вызвать.
• В языке C есть строгие гарантии по именам. Нельзя сказать, что в C нет манглирования. Оно может и быть. Чего в C нет, так это <span style="color: blue;">манглирования типом</span>.
```cpp
float sqrt(float x);  // 1
double sqrt(double x); // 2
sqrt(42); // неясно, что вызвать, оба варианта подходят
```
• В языке C++ есть перегрузка и компилятор должен <span style="color: blue;">разрешить имя</span>. То есть, связать упомянутое в коде имя с обозначаемой им сущностью.
• В коде выше как по вашему будет сделан вызов и почему?
• Разумеется, будет ошибка компиляции. Оба варианта одинаково хороши.
#### Правила разрешения перегрузки
• Первое приближение (здесь много чего не хватает):
1. Точное совпадение (int -> int, int -> const int&, etc)
2. Точное совпадение с шаблоном (int -> T)
3. Стандартные преобразования (int -> char, float -> unsigned short, etc)
4. Переменное число аргументов
5. Неправильно связанные ссылки (literal -> int&, etc)
• Мы вернёмся к перегрузке, когда подробнее поговорим о шаблонах функций.
Пример overload.cc
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Example of overload
// compile with: g++ overload.cc -DN0 ... -DN15
//
//---------------------------------------------------------------------------

#include <iostream>

#ifndef N0
int foo(int x) { return 0; }
#endif

#ifndef N1
int foo(const int &x) { return 1; }
#endif

#ifndef N2
template <typename T> T foo(T x) { return 2; }
#endif

#ifndef N10
int foo(char x) { return 10; }
#endif

#ifndef N11
int foo(short x) { return 11; }
#endif

#ifndef N12
int foo(const char &x) { return 12; }
#endif

#ifndef N13
int foo(double x) { return 13; }
#endif

#ifndef N14
int foo(...) { return 21; }
#endif

#ifndef N15
int foo(int &x) { return 31; }
#endif

int main() {
	std::cout << "result: " << foo(10) << std::endl;
	return 0;
}
```
#### Перегрузка конструкторов
• Методы класса, разумеется, тоже можно перегружать и наиболее полезно это для конструкторов.
```cpp
class line_t {
	float a_ = -1.0f, b_ = 1.0f, c_ = 0.0f;
	
public:
	// по умолчанию
	line_t() {}
	
	// из двух точек
	line_t(const point_t &p1, const point_t &p2);
	
	// явные параметры линии
	line_t(float a, float b, float c);
};
```
#### Коротко о пространствах имён
• Любое имя принадлежит к какому-то пространству имён.
```cpp
// no namespace here

int x;

int foo() {
	return ::x;
}
```
• Здесь кажется, что x не принадлежит ни к какому пространству имён.
• Но на самом деле x принадлежит к <span style="color: blue;">глобальному пространству имён</span>.
#### Пространство имён std
• Вся стандартная библиотека принадлежит к пространству имён std.
```cpp
std::vector, std::string, std::sort, ...
```
• Исключение - это старые хедера, наследованные от C, такие, как \<stdlib.h>.
• Чтобы завернуть atoi в std, сделаны новые хедера вида \<cstdlib>.
• Вы не имеете права добавлять в стандартное пространство имён свои имена.
• Точно по той же причине, по какой вы не можете начинать свои имена с подчёркивания и большой буквы.
#### Ваши пространства имён
• Вы можете вводить свои пространства имён и неограниченно вкладывать их друг в друга.
• При том, структуры тоже вводят пространства имён.
```cpp
namespace Containers {
	struct List {
		struct Node {
			// .... whatever ....
		};
	};
}

Containers::List::Node n;
```
#### Преоткрытие пространств имён
• В отличие от структур, пространства имён могут быть переоткрыты
```cpp
namespace X {
	int foo();
}
// теперь переоткроем и добавим туда bar
namespace X {
	int bar();
}
```
• Структура вводит **тип данных**. Тип не должен существовать, если в проограмме не будет его объектов.
• Для пространств имён куда удобнее (сюрприз) пространства имён.
#### Директива using, второй смысл
• Мы можем вводить отдельные имена и даже целые пространства имён:
```cpp
namespace X {
	int foo();
}

using std::vector;
using namespace X;

vector<int> v; v.push_back(foo());
```
• Использовать эти механизмы следует осторожно, так как пространства имён придуманы не просто так.
#### Анонимные пространства имён
• Это распространённый механизм для замены статических функций.
```cpp
namespace /* IdFgghbjhbklbkuU6*/ {

int foo() {
	return 42;
}

}
// using namespace IdFgghbjhbklbkuU6;

int bar() { return foor(); } // ok!
```
• Означает сделать пространство имён со сложным уникальным именем и тут же сделать его using namespace.
• Поскольку имена из него не видны снаружи, они, как бы, статические.
#### Правила хорошего тона
• Не засорять глобальное пространство имён.
• Никогда не писать using namespace в заголовочных файлах.
• Использовать анонимные пространства имён вместо статических функций.
• Не использовать анонимные пространства имён в заголовочных файлах.
#### Правильный hello world
```cpp
#include <iostream>

namespace {
	const char * const helloworld = "Hello, world";
}

int main() {
	std::cout << helloworld << std::endl;
}
```
• Обратите внимание: функция main обязана быть в глобальном пространстве имён.
## Сбалансированные деревья
#### Поисковые деревья
• Поисковость - это свойство дерева, заключающееся в том, что любой элемент в правом поддереве больше любого элемента в левом.м
• Любой ключ может быть найден, начиная от верхушки дерева за время, пропорциональное <span style="color: blue;">высоте</span> дерева.
• В лучшем случае, у нас дерево из N элементов будет иметь высоту logN.
• Важное наблюдение: над одним и тем же множеством элементов все возможные поисковые деревья сохраняют его inorder обход сортированным.
![[3.1.png]]
#### Range queries
• К данным, хранящимся в дереве, удобно применять range queries.
• Пусть на вход поступают ключи (каждый ключ - это целое число, **все ключи разные**) и запросы (каждый запрос - это пара из двух целых чисел, **второе больше первого**).
• Нужно для каждого запроса подсчитать в дереве количество ключей, таких, что все они лежат строго между его левой и правой границами включительно.
• Вход: <span style="color: red;">k</span> 10 <span style="color: red;">k</span> 20 <span style="color: red;">q</span> <span style="color: blue;">8</span> <span style="color: blue;">31</span> <span style="color: red;">q</span> 6 9 <span style="color: red;">k</span> 30 <span style="color: red;">k</span> 40 <span style="color: red;">q</span> 15 40
• Результат: <span style="color: blue;">2</span> 0 3
#### Решение через std::set
```cpp
template <typename C, typename T>
int range_query(const C& s, T fst, T snd) {
	using itt = typename C::iterator;
	itt start = s.lower_bound(fst);   // first not less than fst
	itt fin = s.upper_bound(snd);     // first greater than snd
	return mydistance(s, start, fin); // std::distance для set
}
```
• Мы хотим, чтобы наше поисковое дерево поддерживало тот же интерфейс (кроме distance, т.к. нам нужны переопределённые операторы).
• Кроме того, нужен метод insert для вставки ключа.
#### Проектирование поискового дерева
```cpp
namespace Trees {
	template <typename KeyT, typename Comp>
	class SearchTree {
		struct Node;             // внутренний узел
		using iterator = Node *; // положение внутри дерева
		Node *top_;
	
	public: // селекторы
		iterator lower_bound(KeyT key) const;
		iterator upper_bound(KeyT key) const;
		int distance(iterator fst, iterator snd) const;
	
	public: // модификаторы
		void insert(KeyT key);	
	};
}
```
#### Проблема дисбаланса
• В лучшем случае, поисковое дерево из N элементов будет иметь высоту logN.
• Но дерево может быть поисковым и, при этом, довольно бесполезным.
• В худшем случае, оно вырождается в список, что делает RBQ довольно неэффективными.
• Но мы видим, что std::set работает довольно быстро. То есть, как-то решает эту проблему.
![[3.2.png]]
#### Балансировка поворотами
• Два базовых преобразования, сохраняющих инвариант поисковости - это левый и правый поворот.
![[3.3.png]]
#### Хранение инварианта в узле
• Надлежащим количеством поворотов можно сделать любое дерево полезным, но это нетривиальная задача.
• Гораздо проще при каждой вставке поддерживать поворотами какой-нибудь инвариант, который гарантирует нам полезность дерева.
• **Красно-чёрный** инвариант:
	• Корень чёрный
	• Все нулевые потомки чёрные
	• У каждого красного узла все потомки чёрные
	• На любом пути от данного узла до каждого из нижних листьем одинаковое количество чёрных узлов
![[3.4.png]]
• Инвариант AVL:
	• Высота пустого узла нулевая
	• Высота дерева - это длина наибольшего пути от корня до пустого узла
	• Для каждой вершины высота обоих поддеревьем различается не более чем на 1
![[3.5.png]]
#### Проектирование узла
```cpp
struct Node {
	KeyT ket_;
	Node *parent_, *left_, *right_;
	int height_; // AVL инвариант
};
```
• Чем плох так спроектированный узел?
• Он может быть инициализирован только старой **агрегатной** инициализацией:
```cpp
Node n = { key, nullptr, nullptr, nullptr, 0 };
Node n = { key }; // остальные нули

Node n { key }; // остальные нули, новшество в C++11
```
• Агрегатная инициализация ломается при появляении приватного состояния.
```cpp
struct Node {
	KeyT ket_;
	Node *parent_, *left_, *right_;
	int balance_factor() const;
	
private:
	int height_; // AVL инвариант
};

Node n { key }; // ошибка, это не агрегат
```
• Кроме того, она не даёт **уверенности**, что поле key инициализировано.
## Конструкторы и деструкторы
#### Проектирование узла
```cpp
struct Node {
	KeyT key_;
	Node *parent_ = nullptr, *left_ = nullptr, *right_ = nullptr;
	int height_ = 0;
	
	Node(KeyT key) { key_ = key; } // конструктор
};
```
• Он может быть инициализирован либо **direct**, либо **copy** инициализацией.
```cpp
Node n(key);  // прямая инициализация, старый синтаксис
Node n{key};  // прямая инициализация, новый синтаксис
Node k = key; // копирующая инициализация
```
#### Отступление: старая инициализация
• До 2011 года вызов конструктора предполагал круглые скобки.
```cpp
Triangle2D<double> t(p1, p2, p3); // вызов конструктора
Triamgle2D<double> t{p1, p2, p3}; // вызов конструктора
```
• До сих пор это означает одно и то же. <span style="color: red;">Но есть одно но</span>.
```cpp
myclass_t m(list_t(), list_t()); // вызов конструктора?
myclass_t m{list_t(), list_t()}; // вызов конструктора?
```
• Одна из этих строчек значит не то, что вы думаете.
Пример из гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Example of vex parsing
//
//---------------------------------------------------------------------------

#include <iostream>

struct list_t {};
struct myclass_t {
	int x = 42;
	myclass_t(list_t, list_t) {}
};

int main() {
	myclass_t m1(list_t(), list_t());
	myclass_t m2{list_t(), list_t()};
	
	std::cout << m1.x << std::endl; // WAT?
	std::cout << m2.x << std::endl;
}
```
Ошибка:
```bash
error: request for member 'x' in 'm1', which is of non-class type 'myclass_t(list_t (*)(), list_t (*)())'
25 | std::cout << m1.x << std::endl; // WAT?
```
Пояснение лектора: myclass_t m1(list_t(), list_t()); компилятором воспринимается как объявление функции (function declaration).
#### Двойная инициализция
• Присваивая <span style="color: red;">в теле</span> конструктора, мы инициализируем дважды (второй раз временный объект для присваивания):
```cpp
struct S {
	S() { std::cout << "default" << std::endl; }
	S(KeyT key) { std::cout << "direct" << std::endl; }
};

struct Node {
	S key_; int val_;
	
	Node(KeyT key, int val) { key_ = key; val_ = val; }
};
```
#### Списки инициализации
• Чтобы уйти от двойной инициализации, до тела конструктора предусмотрены <span style="color: blue;">списки инициализации</span>.
```cpp
struct S {
	S() { std::cout << "default" << std::endl; }
	S(KeyT key) { std::cout << "direct" << std::endl; }
};

struct Node {
	S key_; int val_;
	
	Node(KeyT key, int val) : key_(key), val_(val) {}
};
```
#### Два правила для инциализации
• Список инициализации выполняется строго в том порядке, в каком поля определены в классе (не в том, в каком они записаны в списке).
```cpp
struct Node {
	S key_; T key2_;
	Node(KeyT key) : key2_(key), key_(key) {} // S, T
};
```
• Инициализация в теле класса незримо входит в список инициализации.
```cpp
struct Node {
	S key_ = 1; T key2_;
	Node(KeyT key) : key2_(key) {} // S, T
};
```
#### Параметры по умолчанию
• Если что-то уже есть в списке инициализации, то инициализатор в теле класса игнорируется.
```cpp
struct Node {
	S key_ = 1;
	Node() {} // key_(1)
	Node(KeyT key) : key_(key) {} // key_(key)
};
```
• Такое лучше переписать с **параметром по умолчанию**.
```cpp
struct Node {
	S key_;
	Node(KeyT key = 1) : key_(key) {} // key_(key)
};
```
#### Обсуждение: делегация конструкторов
• Если конструктор делает нетривиальные вещи, его можно <span style="color: blue;">делегировать</span>.
```cpp
struct slass_c {
	int max = 0, min = 0;
	
	class_c(int my_max) : max(my_max > 0 ? my_max : DEFAULT_MAX) {}
	
	class_c(int my_max, int my_min) : class_c(my_max),
		min(my_min > 0 && my_min < max ? my_min : DEFAULT_MIN) {}
};
```
• Место делегированного конструктора первое в списке инициализации.
• Далее делегирующий конструктор можно тоже делегировать и т.д.
#### Проектирование узла
• Кроме создания нам нужно освобождать память.
```cpp
struct Node {
	KeyT key_;
	Node *parent_ = nullptr, *left_ = nullptr, *right_ = nullptr;
	int height_ = 0;
	
	Node(KeyT key) : key_(key) {} // конструктор
	~Node() { delete left_; delete right_; }
};
```
• Здесь деструктор через delete рекурсивно вызывает деструкторы подузлов.
• Чем это решение плохо?
#### Мнимые и реальные проблемы
```cpp
template <typename KeyT, typename Comp>
SearchTree::~SearchTree() { delete top_; }

template <typename KeyT, typename Comp>
SearchTree::Node::~Node() { delete left_; delete right; }
```
• Пример некачественной критики: нет проверки на nullptr.
• Пример качественной критики: возможно переполнение стека.
• Как бы вы сделали без рекурси?
#### Частые ненужные приседания
• Люди часто пытаются делать в деструкторе лишние обнуления состояния.
```cpp
public:
	~MyVector() {
		delete[] buf_;
		buf_ = nullptr; // oops
		size_ = 0;      // oops
		capacity_ = 0;  // oops
	}
};
```
• После того, как деструктор отработал, время жизни окончено.
• Технически компилятор имеет право **выбросить** выделенные строчки.
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
// Unnecessary dto code demo
// compile with: g++ -S -O1 -g0 -masm=intel unnecessary-2.cc
//
//---------------------------------------------------------------------------

struct Ptr {
	int x;
	int *p;
	
	Ptr(int x) : x(x), p(new int[x]) {}
	~Ptr();
};

Ptr::~Ptr() {
	delete[] p;
	x = 0;
	p = nullptr;
}

void Ptr::use(Ptr &p) {}
```
Далее лектор в .s файле показывает, что эти зануления оптимизируются компилятором и не генерируют ассемблерные инструкции.
#### Ассимметрия инициализации
• Для класса с конструктором без аргументов, нет разницы между:
```cpp
SearchTree s;   // default-init, SearchTree()
SearchTree t{}; // default-init, SearchTree()
```
• Но для примитивных типов и агрегатов разница гигантская.
```cpp
int n;   // default-init, n = garbage
int m{}; // value-init, m = 0

int *p = new int[5]{} // calloc
```
• То же самое для полей классов и т.д. рекурсивно.
## Специальные конструкторы
#### Волшебные очки
• Что вы видите здесь?
```cpp
class Empty {
};
```
• Программист видит возможность скопировать и присвоить:
```cpp
{
	Empty x; Empty y(x); x = y;
} // x, y destroyed
```
#### Отличия копирования от присваивания
• Копирование - это, в основном, способ инициализации.
```cpp
Copyable a;
Copyable b(a), c{a}; // прямое конструирование via copy ctor
Copyable d = a; // копирующее конструирование
```
• Присваивание - это переписывание готового объекта.
```cpp
a = b; // присваивание
d = c = a = b; // присваивание цепочкой (правоассоциативно)
```
• Ergo: копирование <span style="color: blue;">похоже на</span> конструктор. Присваивание совсем не похоже.
#### Волшебные очки
• Посмотрим на пустой класс через волшебные очки.
```cpp
class Empty {
	// Empty(); // ctor
	// ~Empty(); // dtor
	// Empty(const Empty&); // copy ctor
	// Empty& operator=(const Empty&); // assignment
};
```
• Все эти (и пару других) методов для вас сгенерировал компилятор.
```cpp
{
	Empty x; Empty y(x); x = y;
} // x, y destroyed
```
#### Семантика копирования
• По умолчанию конструктор копирования и оператор присваивания реализуют:
	• побитовое копирование и присваивание встроенных типов и агрегатов
	• вызов конструктора копирования, если есть
```cpp
template <typename T> struct Point2D {
	T x_, y_;
	// Point2D() : default-init x_, default-init y_ {}
	// ~Point2D() {}
	// Point2D(const Point2D& rhs) : x_(rhs.x_), y_(rhs.y_) {}
	// Point2D& operator=(const Point2D& rhs) {
	//	 x_ = rhs.x_; y = rhs.y_, return *this;
	// }
};
```
#### Обсуждение
• Должны ли мы делать неявное явным?
```cpp
template <typename T, typename KeyT> class Cache {
	std::list<T> cache_;
	std::unordered_map<KeyT, T> locations_;
};
```
• Здесь не нужны конструктор копирования и оператор присваивания.
```cpp
Cache c1{c2}; // или Cache c1 = c2;
c2 = c1;
```
• По умолчанию копирование и присваивание тут отлично работают.
• В таких случаях мы не должны определять копирование/присваивание.
#### Случай, когда умолчание опасно
• Казалось бы, всё просто:
```cpp
class Buffer {
	int *p_;
public:
	Buffer(int n) : p_(new int[n]) {}
	~Buffer() { delete [] p_; }
	// Buffer(const Buffer& rhs) : p_(rhs.p_) {}
	// Buffer& operator= (const Buffer& rhs) { p_ = rhs.p_; .... }
};
```
• Что может пойти не так?
• Увы, в волшебных очках мы видим проблему:
```cpp
{ Buffer x; Buffer y = x; } // double deletion
```
#### Default и delete
• Мы можем явно попросить дефолтное поведение, прописав default, и явно его заблокировать, написав delete.
```cpp
class Buffer {
	int *p_;
public:
	Buffer(int n) : p_(new int[n]) {}
	~Buffer() { delete [] p_; }
	Buffer(const Buffer& rhs) = delete;
	Buffer& operator= (const Buffer& rhs) = delete;
};

{ Buffer x; Buffer y = x; } // compilation error
```
#### Обсуждение
• Хорошая ли идея иметь некопируемый буфер?
#### Реализуем копирование
```cpp
class Buffer {
	int n_; int *p_;
public:
	Buffer(int n) : n_(n), p_(new int[n]) {}
	~Buffer() { delete [] p_; }
	
	// думайте о "Buffer rhs; Buffer b{rhs};"
	Buffer(const Buffer& rhs) : n_(rhs.n_), p_(new int[n_]) {
		std::copy(p_, p_ + n_, rhs.p_);
	}
	
	Buffer& operator= (const Buffer& rhs);
};
```
#### Реализуем присваивание
```cpp
Buffer& Buffer::operation= (const Buffer& rhs) {
	n_ = rhs.n_;
	delete [] p_;
	p_ = new int[n_];
	std::copy(p_, p_ + n_, rhs.p_);
	return *this;
}
```
• Тут можно визуализировать это как:
```cpp
Buffer a, b; a = b;
```
• Видите ли вы ошибку в коде?
#### Не забываем о себе
```cpp
Buffer& operator= (const BUffer& rhs) {
	if(this == &rhs) return *this;
	n_ = rhs.n_;
	delete [] p_;
	p_ = new int[n_];
	std::copy(p_, p_ + n_, rhs.p_);
	return *this;
}
```
• Первая проблема - это присваивание вида a = a. Её довольно просто решить.
• Вторая проблема сложнее. Её мы пока отложим и поговорим о специально семантике копирования и присваивания.
#### Спецсемантика копирования: RVO
```cpp
struct foo {
	foo() { cout << "foo::foo()" << endl; }
	foo(const foo&) { cout << "foo::foo( const foo& )" << endl; }
	~foo() { cout << "foo::~foo()" << endl; }
};

foo bar() { foo local_foo; return local_foo; }

int main() {
	foo f = bar();
	use(f); // void use(foo &);
}
```
• Что здесь должно быть на экране? А что реально будет?
```bash
foo::foo()
foo::~foo()
```
#### Допустимые формы
• Поскольку конструктор копирования подвержен RVO, это непросто функция. У неё есть специальное значение, которое компилятор должен соблюдать.
• Но чтобы он распознал конструктор копирования, у него должна быть одна из форм, предусмотренных стандартом. Основная форма - это константная ссылка:
```cpp
struct Copyable {
	Copyable(const Copyable &c);
};
```
• Допустимо также принимать неконстантную ссылку, <span style="color: blue;">как угодно cv-квалифицированную</span> ссылку или значение.
#### Отступление: cv-квалификация
• В языке C++ есть два очень специальных квалификатора const и volatile.
• Что означает const для объекта?
```cpp
const int c = 34;
```
- это объект, в который вы не можете записать другое значение.
• Что означает volatile для объекта?
```cpp
volatile int v;
```
- это объект, который может непредсказуемо измениться.
• Что означает const volatile для объекта?
```cpp
const volatile int cv = 42;
```
- это объекты, в который вы не можете ничего записать, но он может непредсказуемо измениться.
• Что означает const для метода?
```cpp
int S::foo() const {  return 42; }
```
- этот метод не может изменить значения класса.
- этот метод **может быть вызван**, если объект класса - const.
• Что означает volatile для метода?
```cpp
int S::bar() volatile { return 42; }
```
- этот метод может быть вызван для volatile объекта.
• Что означает const volatile для метода?
```cpp
int S::buz() const volatile { return 42; }
```
- этот метод существует и для обычных объектов, и для const-объектов, и для volatile-объектов, и для const-volatile-объектов.
#### Исторический анекдот
• Что вы сможете сделать с volatile объектом std::vector?
```cpp
volatile std::vector v;
```
• Посмотрите в предусмотренную стандартом реализацию.
• Потом поэкспериментируйте.
Пример из гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Volatile string demo: string class have no volatile methods
// compile with: g++ volstr.cc
//
//---------------------------------------------------------------------------

#include <iostream>
#include <string>

int main() {
	volatile std::string s{"Hello"};
	
	char c = s[2];
	s += "world";
	std::cout << s << std::endl;
	std::cout << s.c_str() << std::endl;
}
```
При попытке компиляции выдаётся лютейшая портянка из предупреждений и ошибок.
#### Недопустимые формы
• Шаблонный конструктор - это никогда не конструктор копирования.
```cpp
template <typename T> struct Copyable {
	Copyable(const Copyable &c) {
		std::cout << "Hello!" << std::endl;
	}
};

Copyable<void> a;
Copyable<void> b{a}; // на экране Hello
```
• Здесь всё нормально, класс шаблонный, конструктор не шаблонный.
```cpp
template <typename T> struct Coercible {
	template <typename U> Coercible(const Coercible<U> &c) {
		std::cout << "Hello!" << std::endl;
	}
};

Coercible<void> a;
Coercible<void> b{a}; // на экране ничего

Coercible,int> c{a};
```
• Здесь компилятор сгенерирует копирующий конструктор по умолчанию.
#### Спецсемантика инициализации
• Обычные конструкторы определяют <span style="color: blue;">неявное преобразование типа</span>.
```cpp
struct MyString {
	char *buf_; size_t len_;
	MyString(size_t len) : buf_{new char[len]{}}, len_{len} {}
};

void foo(MyString);

foo(42); // ok, MyString implicitly constructed
```
• Почти всегда это очень полезно.
• Но это <span style="color: red;">не всегда</span> хорошо. Например, в ситуации со строкой, мы ничего такого не имели в виду.
#### Требуем ясности
• Ключевое слово explicit указывается, когда мы хотим заблокировать пользовательское преобразование.
```cpp
struct MyString {
	char *buf_; size_t len_;
	explicit MyString(size_t len) : buf_{new char[len]{}}, len_{len} {}
};
```
• Теперь здесь будет ошибка компиляции:
```cpp
void foo(MyString);

foo(42); // error: could not convert '42' from 'int' to 'MyString'
```
#### Снова direct vs copy
• Важно понимать, что explicit-конструкторы рассматриваются для прямой инициализации:
```cpp
struct Foo {
	explicit Foo(int x) {} // блокирует неявные преобразования
};

Foo f{2}; // прямая инициализация

Foo f = 2; // инициализация копирование, FAIL
```
• В этом смысле инициализация копирование похожа на вызов функции.
#### Пользовательские преобразования
• В некоторых случаях мы не можем сделать конструктор. Скажем, что если мы хотим неявно преобразовать Quat\<int> в int?
• Тогда мы пишем <span style="color: red;">operator type</span>.
```cpp
struct MyString {
	char *buf_; size_t len_;
	/* explicit? */ operator const char*() { return buf_; }
};
```
• Можно operator int, operator double, operator S и так далее.
• На такие операторы можно навешивать explicit. Тогда возможно только явное преобразование.
• Таким образом, есть некая избыточность: два способа перегнать туда и два способа прегнать обратно.
• Конечно, хороший тон - это использовать конструкторы, где возможно.
• Как вы думаете, что будет при конфликте?
![[3.6.png]]
Пример из гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Example of both ctor and operator conversion
//
//---------------------------------------------------------------------------

#include <iostream>

struct Foo;
struct Bar;

struct Foo {
	Foo() {}
	Foo(const Bar &) { std::cout << "Ctor Bar -> Foo" << std::endl; }
};

struct Bar {
	Bar() {}
	Bar(const Foo &) { std::cout << "Ctor Foo -> Bar" << std::endl; }
	operator Foo() {
		std::cout << "Op Bar -> Foo" << std::endl;
		return Foo{};
	}
};

int main() {
	Bar b;
	Foo f1{b};  // direct-init, ctor
	Foo f2 = b; // copy-init, ctor, vs op, op wins
}
```
#### Перегрузка
• Пользовательские преобразование участвуют в перегрузке.
• Они проигрывают стандартным, но выигрывают у троеточий.
```cpp
struct Foo { Foo(long x = 0) {} };

void foo(int x);
void foo(Foo x);

void bar(Foo x);
void bar(...);

long l; foo(l); // вызовет foo(int)

bar(1); // вызовет bar(Foo)
```
#### Такие разные операторы
• Перегрузка операторов присваивания и приведения выглядит непохоже.
```cpp
struct Point2D {
	int x_, y_;
	
	Point2D& operator=(const Point2D& rhs) = default;
	
	operator int() { return x; }
};
```
• В мире конструкторов спецсемантика есть только у копирования и приведения.
• В мире переопределённых операторов она есть везде, и она нас ждёт уже на следующей лекции.
#### Домашня работа HWT
• Со стандартного ввода приходят ключи (каждый ключ - это целое число, **все ключи разные**) и запросы двух видов.
• Запрос (m) на поиск k-го наименьшего элемента.
• Запрос (n) на поиск количества элементов, меньших, чем заданный.
• Вход: <span style="color: red;">k</span> 8 <span style="color: red;">k</span> 2 <span style="color: red;">k</span> -1 <span style="color: red;">m</span> <span style="color: blue;">1</span> <span style="color: red;">m</span> 2 <span style="color: red;">n</span> 3
• Результат: <span style="color: blue;">-1</span> 2 2
• Ключи могут быть как угодно перемешаны с запросами. Чтобы успешно пройти тесты, вы должны продумать такую балансировку дерева, чтобы оба вида запросов работали с логарифмической сложностью.
<h1 align="center">ВЕКТОРА И SFINAE</h1>

---
<p align="center">На долгом пути к std::vector - проектирование реалистичного шаблонного контейнера.</p>
## Вектора
#### От ручного выделения к векторам
```cpp
int* n = new int[10];
n[5] = 5;
// тут много кода
// какой сейчас размер у n?
// стереть крайний элемент?
// пуст ли теперь n?
// не забыть delete[]
```
```cpp
vector<int> v(10);
v[5] = 5;
// тут много кода
size_t vsize = v.size();
v.pop_back();
if(v.empty()) { /* что-то */ }
// ресурсы буду освобождены
```
#### Два непростых вопроса
• Допустим, я хочу завести в своей программе вектор из константных ссылок:
```cpp
std::vector<const int&> v;
```
• Что скажете?
• Допустим, я не знаю тип переменной x, но знаю, что это контейнер.
```cpp
template<typename Cont> void foo(const Cont& x) {
	if(x.empty()) return;
	// do something
}
```
• Могу ли я быть уверенным, что это будет работать для всех контейнеров?
#### Требования к элементам контейнеров
• Общие для всех контейнеров методы <span style="color: brown;">[container.requirements.general]</span>
	• empty - проверка пустоты контейнера
	• swap - обмен конетйнерных переменных содержимым
	• size (кроме array) - действительный размер контейнера
	• clear (кроме array) - очистка контейнера
	• begin, end, cbegin, cend - получение итераторов (см. далее)
• Требования к элементам зависят от конкретной операции, но чаще всего
	• DefaultConstructible - требование к наличию конструктора по умолчанию
	• MoveConstructible - требование к наличию конструктора перемещения или копирования
#### Гарантии непрерывности памяти
```cpp
// функция init написана в старом стиле
template<typename T> void init(T* arr, size_t size);

// но её можно использовать с векторами
vector<T> t(n);
T* start = &t[0];
init_t(start, n);
assert(t[1] == start[1]);
```
*When choosing a container, remember vector is best. Leave a comment to explain if you choose from the rest. (c) Tony van Eerd*
![[../../../_Meta/attachments/13.1.png]]
#### Неприятное исключение: vector\<bool\>
```cpp
vector<bool> t(n);
bool* start = &t[0]; // это не скомпилируется, но представим
assert(t[1] == start[1]); // oops!
```
Важно запомнить две вещи:
• <span style="color: red;">vector&ltbool&gt не удовлетворяет соглашениям контейнера vector.</span>
• <span style="color: red;">vector&ltbool&gt не содержит элементов типа bool.</span>
• Не используйте vector\<bool\> для обобщённого программирования.
```cpp
using vector_bool = vector<bool>;
vector_bool x(10); // условно ок, но тут лучше std::bitset
```
#### Задача: что можно здесь улучшить?
```cpp
vector<int> v;

for(int i = 0; i != N; ++i)
	v.push_back(i);
```
#### Ответ: вектор не терпит халатности
```cpp
vector<int> v;
v.reserve(N);

for(int i = 0; i != N; ++i)
	v.push_back(i); // теперь здесь не будет перевыделений
```
• При вставке в конец вектору могут потребовать реаллокации памяти.
• Это означает, что всегда полезно думать о памяти вектора не меньше, чем о памяти динамического массива.
#### Ещё про size и capacity
• size - это сколько элементов у вектора уже есть.
• capacity - это сколько элементов в нём может быть до первого перевыделения.
```cpp
vector<int> v(10000);

assert(v.size() == 10000);
assert(v.capacity() >= 10000);
```
Размер - это что-то, чем можно в явном виде управлять, в отличие от ёмкости.
```cpp
v.resize(100);
assert(v.size() == 100);
assert(v.capactity >= 10000);
```
#### Амортизация
• При написании метода push, вам предлагалось оценить его алгоритмическую сложность.
• Проблема в том, что она очевидно O(1), если не надо реаллоцировать и O(n), если надо.
• То есть, мы платим иногда. Это примерно как купить машину и платить только за бензин, пока машина не износится, а потом купить новую.
• В экономике распределение стоимости товара по стоимости его периода эксплуатации называется амортизацией товара.
• Амортизированное O(n) обозначается O(n)+.
#### Амортизированная стоимость
• По определению амортизированная стоимость операции - это стоимость N операций, отнесённая к N.
• Для динамического массива c<sub>i</sub> = 1 + \[*realloc*\] * (*i* - 1)
• Амортизированная стоимость одной вставки будет $\frac{\sum_i c_i}{N}$ для N вставок.
• Допустим, мы, если реаллокация нужна, растим массив на 10 элементов. ${\sum_i c_i}$ = ?
Ответ: $N + \sum_{k=1}^{N/10} 10 \cdot k = O(N^2)$
• Заметим, что это очень плохая стратегия. Амортизированная сложность push будет $\frac{O(N^2)}{N} = O(N)+$. Можем ли мы придумать и доказать нечто лучшее?
#### Лучшая стратегия
• Прирост вдвое
$$\frac{\sum_i c_i}{N} = \frac{N + \sum_{k=1}^{lg(N)} 2^j}{N} = \frac{O(N)}{N} = O(1)$$
• Видно, что разница есть: при одной стратегии у нас в среднем линейное, а при другой в среднем постоянное время вставки.
• Увы, взять сумму $\sum_{k=1}^{lg(N)} 2^j$ в общем уже не так просто, а при более сложных стратегиях, это становится мучительно.
• Можем ли мы упростить себе жизнь?
#### Дополнение: метод потенциала
• Выберем функцию потенциала Ф(n) так, чтобы Ф(0) = 0, Ф(n) $\geq$ 0.
• Здесь n - это номер шага.
• Амортизированная стомость - это стомость плюс изменение потенциальной функции $c_i + Ф(i) - Ф(i-1)$
• Выбор потенциальной функции облегчает вычисления, потому что
$$\sum_i(c_i + Ф(i) - Ф(i - 1)) = Ф(n) - Ф(0) + \sum_i c_i \geq \sum_i c_i$$
• Удачный выбор сделает выражение $\sum_i(c_i + Ф(i) - Ф(i - 1))$ проще, чем $\sum_i c_i$
• Обсуждение: как выбрать для массива?
• Для массива, поскольку при реаллокации вдвое $2 * s_n \geq c_n$
$$Ф(n) = 2 * s_n - c_n$$
• Без реаллокации
$$c_i + Ф(i) - Ф(i-1) = 1 + (2 * s_i - C) - (2 * s_{i-1} - C) = 1 + 2(s_i - s_{i-1}) = 3$$
• С реаллокацией $Ф(i - 1) = 2k - k = k$, $Ф(i) = 2(k + 1) - 2k = 2$
$$c_i + Ф(t_i) - Ф(t_{i-1}) = (k + 1) + 2 - k = 3$$
• В итоге, в любом случае, $\sum c_i \leq 3N$, и мы доказали асимптотику O(1).
• В качестве упражнения проанализируйте стратегию роста в log(N) раз.
#### Обсуждение
• Выбор простого роста вдвое не всегда лучшая стратегия.
• Реальная стратегия из libstdc++ несколько сложнее и обладает рядом приятных теоретических свойстсв.
```cpp
const size_type __len = size() + std::max(size(), __n);
```
• Попробуйте дома проанализировать эту стратегию и обосновать, почему она выбрана в качестве основной.
#### Два механизма инициализации
• Расширенный синтаксис.
• Явный конструктор из списка инициализации.
```cpp
class B {
	int a_;
public:
	B(int a) : a_(a) {}
	B(std::initializer_list<int> il);
};

B b(1), c{1}; // теперь они вызывают разные конструкторы
```
#### Списочная инициализация: вектора
```cpp
// это вектор [14, 14, 14]
vector<int> v1(3, 14);

// а это вектор [3, 14]
vector<int> v2{3, 14};
```
• Это связано с наличием у вектора **нескольких** конструкторов.
```cpp
v(10); // размер 10, инициализация по умолчанию
v(10, 1); // размер 10, инициализировать единицами
v{10, 1}; // размер = размеру списка, инициализация списком
```
#### То же для ваших контейнеров
• Хорошая новость: initializer_list - это тоже разновидность последовательного контейнера, и его можно обходить итераторами.
```cpp
template<typename T> class Tree {
	// тут какая-то специфика дерева
	bool add_node(T& data);
	
public:
	Tree(std::initializer_list<T> il) {
		for(auto ili = il.begin(); ili != il.end(); ++ili)
			add_node(*ili);
	}
};
```
• Плохая новость: вам теперь надо следить, есть ли он в классе.
#### Просто правило для {}
• Если в классе совсем нет конструкторов, это агрегат как в C.
```cpp
struct S { int x, y; }; S s = {1, 2}; // aggregate
```
• Иначе, если есть конструктор из initializer_list, возьмётся он.
• Иначе, если есть любой другой конструктор, возьмётся он.
```cpp
struct S {
	int x, y;
	S(int n) : x(n), y(n) {}
};

S s = {3}; // ctor
```
#### Первое представление об итераторах
```cpp
vector<int> v(10);

// pi - это указатель
auto pi = &v[0];
pi += 3;
assert(*pi == v[4]);

// как узнать, что pi в конце?
```
```cpp
vector<int> v(10);

// vi - это итератор
auto vi = v.begin();
vi += 3;
assert(*vi == v[4]);

if(vi == v.end()) { что-то }
```
![[../../../_Meta/attachments/13.2.png]]
#### Абстракция указателя
• Важно, что итераторы не являются указателями, они абстрагируют их.
• В итоге, любой контейнер может быть сконструирован из любого диапазона.
```cpp
std::list<int> l{1, 2, 3};
std::vector<int> v(l.begin(), l.end());
```
• Это потрясающе удобно, чтобы перекидывать один контейнер в другой.
• Как бы вы написали конструктор из пары итераторов?
#### Конструирование из итераторов
• Наивная попытка вызывает у нас небольшую проблему.
```cpp
template<typename T> class MyVector {
	// ....
public:
	MyVector(size_t nelts, T value); // 1
	template<typename Iter> MyVector(Iter fst, Iter lst); // 2
	// ....
};

MyVector<int> mvec(2, 2); // ошибка, выбран 2
```
## SFINAE
#### Обсуждение: провал подстановки
• Что если подстановка в некотором контексте не может быть выполнена?
```cpp
template<typename T>
typename T::ElementT at(T const& a, int i);

int* p = new int[30];

auto a = at<int>(p, i); // Substitution failure
```
• Что если вывод типов в некотором контексте провален?
```cpp
template<typename T> T max(T a, T b);
int g = max(1, 1.0); // Deduction failure
```
#### SFINAE
• **Substitution Failure Is Not An Error** (провал подстановки не является ошибкой).
```cpp
template<typename T> T max(T a, T b);
template<typename T, typename U> auto max(T a, U b);

int g = max(1, 1.0); // подстановка в 1 провалена
										 // подстановка в 2 успешна
```
• Если в результате подстановки в <span style="color: blue;">непосредственном контексте</span> класса (функции, алиаса, переменной) возникает <span style="color: blue;">невалидная конструкция</span>, эта подстановка неуспешна, но не ошибочна.
• В этом случае второй фазы поиска имён просто не выполняется.
#### SFINAE и ошибки
• Не любая ошибочная конструкция - это SFINAE. Важен контекст подстановки.
```cpp
int negate(int i) { return -i; }

template<typename T> T negate(const T& t) {
	typename T::value_type n = -t();
	// тут используем n
}

negate(2.0); // ошибка второй фазы
```
• Здесь в контексте сигнатуры и шаблонных параметров нет никакой невалидности.
```cpp
int negate(int i) { return -i; }

template<typename T> T::value_type negate(const T& t) {
	typename T::value_type n = -t();
	// тут используем n
}

negate(2.0); // substitution failure
```
• Здесь в контексте сигнатуры и шаблонных параметров выводится T -> double и, разумеется, T::value_type невалидно.
#### Обсуждение
• Техника SFINAE кажется очень простой, но вообще-то её приложения многочисленны и часто очень нетривиальны.
• Рассмотрим задачу: у вас есть два типа и вам нужно определить равны ли они.
```cpp
template<typename T, typename U> int foo() {
	// как вернуть 1, если T == U и 0, если нет?
}
```
• Обратим внимание, что это задача отображения из типов на числа.
• Прежде чем её решать, решим обратную задачу.
#### Интегральные константы
• Отображение из целых чисел на типы называется интегральной константой.
```cpp
template<typename T, T v> struct integral_constant {
	static const T value = v;
	typedef T value_type;
	typedef integral_constant type;
	operator value_type() const { return value; }
};
```
• Возможна даже арифметика.
```cpp
using ic6 = integral_constant<int, 6>
auto n = 7 * ic6{};
```
#### Истина и ложь для типов
• Самые полезные из интегральных констант - самые простые.
```cpp
using true_type = integral_constant<bool, true>;
using false_type = integral_constant<bool, false>;
```
• Всё это есть в стандарте: std::integral_constant и т.д.
• Попробуем написать простой определитель, чтобы проверить одинаковые ли два типа.
```cpp
template<typename T, typename U>
struct is_same : std::false_type {};
```
• По умолчанию разные. Что дальше?
#### Равенство типов
• Теперь можно решить задачу определения равенства типов.
```cpp
template<typename T, typename U>
struct is_same : std::false_type {};

template<typename T>
struct is_same<T, T> : std::true_type {}; // для T == T

template<typename T, typename U>
using is_same_t = typename is_same<T, U>::type;
```
• Благодаря SFINAE, будет работать.
```cpp
assert(is_same<int, int>::value && !is_same<char, int>::value);
```
#### Определители и модификаторы
Определитель: является ли тип ссылкой.
```cpp
template<typename T> struct is_reference : false_type {};
template<typename T> struct is_reference<T&> : true_type {};
template<typename T> struct is_reference<T&&> : true_type {};
```
Модификатор: убираем ссылку с типа. Если ссылки не было, то оставляем тип.
```cpp
template<typename T> struct remove_reference { using type = T; };
template<typename T> struct remove_reference<T&> { using type = T; };
template<typename T> struct remove_reference<T&&> { using type = T; };
```
Для модификатора полезен алиас
```cpp
template<typename T>
using remove_reference_t = typename remove_reference<T>::type;
```
#### Четырнадцать категорий
• Любой тип в языке C++ попадает хотя бы под одну из перечисленных ниже категорий.
```cpp
is_void;
is_null_pointer;
is_integral, is_floating_point; // для T и для cv T& транзитивно
is_array; // только встроенные, не std::array
is_pointer; // включая указатели на обычные функции
is_lvalue_reference, is_rvalue_reference;
is_member_object_pointer, is_member_function_pointer;
is_enum, is_union, is_class;
is_function; // обычные функции
```
• Использование довольно тривиально.
```cpp
std::cout << std::boolalpha << std::is_void<T>::value << '\n';
```
#### Свойства типов
• Также очень полезны определители свойств типов.
```cpp
is_trivially_copyable; // побайтово копируемый, memcpy
is_standard_layout; // можно адресовать поля указателем
is_aggregate; // доступна агрегатная инициализация как в C
is_default_constructible; // есть default ctor
is_copy_constructible, is_copy_assignable;
is_move_constructible, is_nothrow_move_assignable, is_move_assignable;
is_base_of; // B является базой (транзитивно, включая сам тип)
is_convertible; // есть преобразование из A к B
```
• И многие другие (их реально десятки).
#### Обсуждение: std::copy
• Рассмотрим наивное копирование, чем-то похожее на алгоритм std::copy:
```cpp
template<typename InpIter, typename OutIter>
OutIter cross_copy(InpIter fst, InpIter lst, OutIter dst) {
	while(fst != lst) { *dst = *fst; ++fst; ++dst; }
	return dst;
}
```
• Увы, по сравнению с настоящим std::copy, у него есть проблемы.
• Можем ли мы их решить с помощью SFINAE?
Пример бенчмарка с гита с таким наивным копированием:
```cpp
//-----------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-----------------------------------------------------------------------------
//
// Benchmark for naive copy vs memcpy
//
// Homebrew std::copy
// > g++ -O2 benchcopy.cc -o bench
// > cl /std:c++20 /O2 /EHsc benchcopy.cc
// > bench 300000000
//
// Real std::copy
// > g++ -O2 -DSTANDARD benchcopy.cc -o bench
// > cl /std:c++20 /O2 /DSTANDARD /EHsc benchcopy.cc
// > bench 300000000
//
//-----------------------------------------------------------------------------

#include <algorithm>
#include <chrono>
#include <cstring>
#include <iostream>
#include <list>
#include <random>
#include <vector>

#ifndef STANDARD
#define NAIVE
#endif

using std::chrono::duration_cast;
using std::chrono::high_resolution_clock;
using std::chrono::milliseconds;

const int IMAX = 9;

template <typename InputIt, typename OutputIt>
OutputIt naive_copy(InputIt first, InputIt last, OutputIt d_first) {
  while (first != last)
    *d_first++ = *first++;
  return d_first;
}

int main(int argc, char **argv) {
  if (argc < 2) {
    std::cerr << "usage: " << argv[0] << " nelts" << std::endl;
    return -1;
  }

  auto nelts = std::stoi(argv[1]);
  std::vector<int> arr(nelts);
  std::vector<int> arrcopy(nelts);
  std::vector<int> arrcopy2(nelts);

  std::random_device rd;
  std::default_random_engine reng(rd());
  std::uniform_int_distribution<int> dist(0, IMAX);

  std::generate(arr.begin(), arr.end(), [&dist, &reng] { return dist(reng); });

  auto tstart = high_resolution_clock::now();
  std::memcpy(arrcopy.data(), arr.data(), nelts * sizeof(int));
  auto tfin = high_resolution_clock::now();

  std::cout << "memcpy: " << duration_cast<milliseconds>(tfin - tstart).count()
            << std::endl;

  tstart = high_resolution_clock::now();
#ifdef NAIVE
  naive_copy(arr.begin(), arr.end(), arrcopy2.begin());
#else
  std::copy(arr.begin(), arr.end(), arrcopy2.begin());
#endif
  tfin = high_resolution_clock::now();

#ifdef NAIVE
  std::cout << "naive copy: "
#else
  std::cout << "std copy: "
#endif
            << duration_cast<milliseconds>(tfin - tstart).count() << std::endl;

  // sanity: do we have mismatch (we shall not)
  auto mism = std::mismatch(arrcopy.begin(), arrcopy.end(), arrcopy2.begin());
  if (mism.first != arrcopy.end() || mism.second != arrcopy2.end()) {
    std::cout << "mismatch: " << *mism.first << " vs " << *mism.second
              << std::endl;
    std::cout << "at: " << std::distance(arrcopy.begin(), mism.first)
              << std::endl;
  }
}
```
Вывод на 500000000 чисел:
```
memcpy: 698
naive copy: 1609
```
При запуске с -DSTANDARD (std\:\:copy) на тех же 500000000 чисел:
```
memcpy: 684
std copy: 7443
```
Там был лаг в связи со стримом. Так как std\:\:copy ест слишком много памяти. На меньшем количестве чисел std\:\:copy выигрывает у memcpy. Например на 300000000 вывод:
```
memcpy: 189
std copy: 139
```
#### Решение проблемы std::copy
• Заведём хелпер и его специализацию для true:
```cpp
template<bool Triv, typename In, typename Out> struct CpSel {
	static Out select(In begin, In end, Out out)
		{ return CopyNormal(begin, end, out); }
};

template<typename In, typename Out>
struct CpSel<true, In, Out> {
	static Out select(In begin, In end, Out out)
		{ return CopyFast(begin, end, out); } // для простых типов
};
```
• Теперь сам алгоритм копирования будет просто решать, кого он вызывает.
• Также тривиально мы решаем проблему с копированием:
```cpp
template<typename In, typename Out>
Out realistic_copy(In begin, In end, Out out) {
	using in_type = /* pointee type (In); */ // как это написать?
	using out_type = /* pointee type (Out); */
	
	enum { Sel = std::is_trivially_copyable<in_type>::value &&
							 std::is_trivially_copyable<out_type>::value &&
							 std::is_same<in_type, out_type>::value };
	
	return CpSel<Sel, In, Out,>::select(begin, end, out);
}
```
Дальше лектор объясняет, что пишется это через std::iterator_traits\<In\>::value_type на примере с гита:
```cpp
//-----------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-----------------------------------------------------------------------------
//
// Benchmark for naive copy vs memcpy
//
// Homebrew std::copy
// > g++ -O2 benchcopy.cc -o bench
// > cl /std:c++20 /O2 /EHsc benchcopy.cc
// > bench 300000000
//
// Real std::copy
// > g++ -O2 -DSTANDARD benchcopy.cc -o bench
// > cl /std:c++20 /O2 /DSTANDARD /EHsc benchcopy.cc
// > bench 300000000
//
// godbolt link for simplicity: https://godbolt.org/z/GanKEGddo
//
//-----------------------------------------------------------------------------

#include <algorithm>
#include <chrono>
#include <cstring>
#include <iostream>
#include <list>
#include <random>
#include <vector>

#ifndef STANDARD
#define NAIVE
#endif

using std::chrono::duration_cast;
using std::chrono::high_resolution_clock;
using std::chrono::milliseconds;

const int IMAX = 9;

template <typename InputIt, typename OutputIt>
OutputIt long_copy(InputIt first, InputIt last, OutputIt d_first) {
  while (first != last)
    *d_first++ = *first++;
  return d_first;
}

template <bool Type, typename In, typename Out> struct CpSel {
  static Out select(In begin, In end, Out out) {
    return long_copy(begin, end, out);
  }
};

template <typename In, typename Out> struct CpSel<true, In, Out> {
  static Out select(In begin, In end, Out out) {
    using in_type = typename std::iterator_traits<In>::value_type;
    auto sz = (end - begin) * sizeof(in_type);
    memcpy(&*out, &*begin, sz);
    return out;
  }
};

template <typename In, typename Out>
Out nonnaive_copy(In begin, In end, Out out) {
  using in_type = typename std::iterator_traits<In>::value_type;
  using out_type = typename std::iterator_traits<Out>::value_type;
  enum {
    Sel = std::is_trivially_copyable<in_type>::value &&
          std::is_trivially_copyable<out_type>::value &&
          std::is_same<in_type, out_type>::value
  };
  return CpSel<Sel, In, Out>::select(begin, end, out);
}

int main(int argc, char **argv) {
  if (argc < 2) {
    std::cerr << "usage: " << argv[0] << " nelts" << std::endl;
    return -1;
  }

  auto nelts = std::stoi(argv[1]);
  std::vector<int> arr(nelts);
  std::vector<int> arrcopy(nelts);
  std::vector<int> arrcopy2(nelts);

  std::random_device rd;
  std::default_random_engine reng(rd());
  std::uniform_int_distribution<int> dist(0, IMAX);

  std::generate(arr.begin(), arr.end(), [&dist, &reng] { return dist(reng); });

  auto tstart = high_resolution_clock::now();
  std::memcpy(arrcopy.data(), arr.data(), nelts * sizeof(int));
  auto tfin = high_resolution_clock::now();

  std::cout << "memcpy: " << duration_cast<milliseconds>(tfin - tstart).count()
            << std::endl;

  tstart = high_resolution_clock::now();
#ifdef NAIVE
  nonnaive_copy(arr.begin(), arr.end(), arrcopy2.begin());
#else
  std::copy(arr.begin(), arr.end(), arrcopy2.begin());
#endif
  tfin = high_resolution_clock::now();

#ifdef NAIVE
  std::cout << "custom copy: "
#else
  std::cout << "std copy: "
#endif
            << duration_cast<milliseconds>(tfin - tstart).count() << std::endl;

  // sanity: do we have mismatch (we shall not)
  auto mism = std::mismatch(arrcopy.begin(), arrcopy.end(), arrcopy2.begin());
  if (mism.first != arrcopy.end() || mism.second != arrcopy2.end()) {
    std::cout << "mismatch: " << *mism.first << " vs " << *mism.second
              << std::endl;
    std::cout << "at: " << std::distance(arrcopy.begin(), mism.first)
              << std::endl;
  }
}
```
Вывод на 400000000 чисел:
```
memcpy: 261
custom copy: 212
```
#### Обсуждение
• Теперь единственным облачком на горизонте остался emplace.
```cpp
struct S {
	S();
	S(int, double, int);
};

std::vector<S> v;

v.emplace_back(1, 1.0, 2); // создали на месте
```
• Но как это может работать для любого типа, если мы в общем случае не знаем количество аргументов конструктора?
#### Представление графа
• У Кнута в ТАОСР приведено следующее представление графа

<h1 align="center">АЛГОРИТМЫ</h1>

---
<p align="center">Введение в алгоритмы стандартной библиотеки и обобщённое программирование.</p>
## Функторы и состояние
#### Обсуждение: for each
• У нас есть метод `equal_range` в `unordered_multimap`, который возвращает диапазон пар ключ-значение с одинаковым ключом.
```cpp
std::unordered_multimap<int, int> m;
// ....
auto [begin, end] = m.equal_range(i);
```
• Вы можете как-то проитерировать и что-то сделать с каждым значением.
```cpp
for(auto it = begin; it != end; ++it) f(*it); // 1
std::for_each(begin, end, f);                 // 2
```
• Какой из двух вариантов вы выберете и почему?
#### Обсуждение
• Выбирать всегда следует алгоритм стандартной библиотеки.
• Аргумент от тела цикла:
```cpp
for(auto elt : cont) {
	// позволяет неконтролируемо вставить массу кода
}
```
• Аргумент от распараллеливания (C++17)
```cpp
for_each(std::execution::par, cont.begin(), cont.end(), func);
```
• В общем случае абстракция циклов повышает и читаемость и эффективность.
• Такие абстракции называются <span style="color: blue;">абстракциями с отрицательной стоимостью</span>.
#### Обсуждение
• Представим, что нам надо как-то использовать внешнее значение.
• Например, путь нам надо умножить `value` на некое число.
```cpp
int mult = get_my_mult(); // очень долгая функция
for(auto it = begin; it != end; ++it) {
	it->second = it->second * mult;
}
```
• Кажется, тут нет способа сделать вызов `std::for_each`?
```cpp
std::for_each(std::execution_par, begin, end, [](auto&& elt) {
	elt.second = elt.second * mult; // ошибка
});
```
#### Идея функтора
• Допустим, мы написали класс с состояние и оператором вызова.
```cpp
struct Multer {
	int m;
	Multer(mult) : m(mult) {}
	void operator()(auto&& elt) { elt.second = elt.second * m; }
};
```
• Тогда это становится возможным.
```cpp
std::for_each(std::execution::par, begin, end, Multer{mult});
```
• Но на каждый чих не наздравствуешься. Неудобно каждый раз писать функтор. 
#### Hello, lambda world
• Давайте посмотрим на лямбды в их блеске и славе.
```cpp
int main(int argc, char** argv) {
	return [argv]() -> int {
		std::cout << "Hello from " << argv[0] << std::endl;
		return 0;
	} ();
}
```
• Мы можем это упростить, выкинув пустое тело и выводимый тип.
```cpp
auto hi = [argv] { std::cout << argv[0] << "\n"; return 0; };
hi();
```
<h1 align="center">([](){})();</h1>
<p align="right">is now legal C++</p>
#### Обсуждение
• λ-выражения - это не функции.
```cpp
auto t = [z](auto x, auto y) { return x < y * z; };
```
• Это, скорее, класс с перегруженным `operator()`.
```cpp
struct __closure_type_for_t {
	int __k;
	auto operator()(auto x, auto y) const {
		return x < y * __k;
	}
} t{z};
```
#### Убираем const
• Если мы хотим изменять захваченный по значению контекст, мы должны сделать нашу лямбду в явном виде `mutable`.
```cpp
auto t = [z](auto x, auto y) mutable { z += 1; return x < y * z; };
```
• Обратите внимание, что z изменяется в пределах замыкания.
```cpp
auto s = t;
t(1, 2); // z изменилось внутри t, но не внутри s
```
• Замыкания по уполчанию копируемые (если не захвачено ничего со стёртым copy ctor).
• Глобальные и статические переменные захватывать не надо, они доступны и так.
#### Виды захвата
• Захват по значению (по ссылке).
```cpp
auto fval = [a, b](int x) { return a + b * x; };
auto fvalm = [a, b](int x) mutable { a += b * x; return a; };
auto fref = [&a, &b](int x) { a += b * x; return a; };
```
• Захват по ссылке всегда mutable и отслеживает состояние переменной.
```cpp
a = 42;

fval(x); // тот же
fref(x); // использует новое a
```
• Разумеется, можно смешивать: `[&a, b, &c, d]`.
• Захват всего контекста по значению (по ссылке).
```cpp
auto faval = [=](int x) { return a + b * x; };
auto faref = [&](int x) { a += b * x; return a; };
```
• Захват всего по значению и частично по ссылке и наоборот.
```cpp
auto favalb = [=, &b](int x) { return a + b * x; };
auto farefa = [&, a](int x) { b += a * x; return b; };
```
• Захват с переименованием.
```cpp
auto freval = [la = a](int x) { return la + x; };
auto freref = [&la = a](int x) { return la += x; return la; };
```
• Переименование позволяет захватить правую ссылку.
```cpp
std::ostream_iterator<int> os{std::cout, " "};
std::vector v = {1, 2, 3};

auto out = [w = std::move(v), os] {
	std::copy(w.begin(), w.end(), os);
};
```
• Теперь вектор передан в состояние замыкания.
• Передача осуществляется в конструкторе замыкания, т.е. в момент создания функции, а не в момент её вызова.
#### Захват в теле класса
```cpp
struct Foo {
	int x;
	void func() {
		[x] mutable { x += 3; } (); // FAIL
		[&x] { x += 3; } (); // FAIL
		
		[=] { x_ += 3; } (); // OK
		[&] { x_ += 3; } (); // OK
		
		[this] { x_ += 3; } (); // OK
	}
};
```
• Это работает, поскольку полный захват захватывает `this`.
#### Задача: локальный контекст
```cpp
auto factory(int parameter) {
	static int a = 0;
	return [=](int argument) {
		static int b = 0;
		a += parameter; b += argument;
		return a + b;
	}
}

auto func1 = factory(1); auto func2 = factory(2);

cout << func1(20) << " " << func1(30) << " "
		 << func2(20) << " " << func2(30) << endl;
```
На экране:
```
21 52 74 106
```
#### Решение: локальный контекст
![[../../../_Meta/attachments/17.1.png]]
#### Единая типизация замыканий
```cpp
auto t = [&x, &y] { return x + y; };
```
• Тут не вполне ясно, что такое `auto`.
• Мало того, так нельзя делать в теле класса.
```cpp
struct Foo {
	auto t = [&x, &y] { return x + y; }; // ошибка
};
```
• Мы понимаем, что это closure type, но не можем записать его явно.
• В этом случае, мы можем частично стереть тип.
```cpp
auto t = [&x, &y] { return x + y; }; // closure
function<int()> f = [&x, &y] { return x + y; }; // function
```
• `std::function<сигнатура>` - это единый тип, к которому <span style="color: blue;">приводятся</span> все замыкания с данной сигнатурой.
• Ключевое слово "приводятся".
```cpp
t = [] { return 42; } // FAIL
f = [] { return 42; } // ok
```
• Тип функции теряет информацию о захвате контекста. Значение имеет только сигнатура.
![[../../../_Meta/attachments/17.2.png]]
#### Снова захват в теле класса
```cpp
using VVTy = std::function<void(void)>;

struct Foo {
	int x;
	
	VVTy xplus1 = [&] { x += 3; }; // OK, but hmmm....
	VVTy xplus2 = [this] { x += 3; }; // OK
};
```
• Это вдвойне интересно, так как такие лямбды-члены ведут себя как методы.
```cpp
struct Foo f;
f.xplus1(); // OK
```
#### Информация о конкретном типе
• Механизм std::function унифицирует типы замыканий.
• Информация о реальном типе возвращается через `target_type`.
```cpp
int f(int);

function<int(int)> fn1 = f,
									 fn2 = [](int a) { return -a; },
									 fn3 = [x](int a) { return x - a; };
cout << fn1.target_type().name() << endl
		 << fn2.target_type().name() << endl;
		 << fn3.target_type().name() << endl;
```
Вывод:
```
PFiiE
Z4mainEUliE_
Z4mainEUliE0_
```
А через c++filt:
```
int (*)(int)
main::{lambda(int)#1}
main::{lambda(int)#2}
```
#### Case study: finally
```cpp
struct Finally {
	std::function<void()> action_;
	explicit Finally(std::function<void()> action) : action_(std::move(action)) {}
	~Finally() { action_(); }
};
```
• Теперь мы можем вот такие фокусы:
```cpp
FILE* f = fopen("myfile.dat", "r"); assert(f);
Finally close_f([&f]{ fclose(f); });
```
#### Обсуждение: heap indirection
• Есть некоторая проблема с таким подходом к `finally`, а именно производительность. Гораздо эффективней иметь `closure`.
```cpp
template<typename ActTy> inline auto Finally(ActTy fn) {
	struct Finally_impl {
		ActTy act;
		explicit Finally_impl(ActTy action) : act(std::move(action)) {}
		~Finally() { act(); }
	};
	
	return Finally_impl(std::move(fn)); // bingo
}
```
• Функция нужна для вывода типов (можно ли сделать deduction hint?).
Далее лектор приводит пример, где он пытался сделать это с deduction hint, но понял, что даже deduction hint не нужен.
```cpp
#include <iostream>
#include <functional>

namespace V1 {
struct Finally {
	std::function<void()> action_;
	explicit Finally(std::function<void()> action) : action_(std::move(action)) {}
	~Finally() { action_(); }
};
}

namespace V2 {
template<typename ActTy>
inline auto Finally(ActTy fn) {
	struct Finally_impl {
		ActT act;
		explicit Finally_impl(ActTy action) : act(std::move(action)) {}
		~Finally() { act(); }
	};
	return Finally_impl(std::move(fn)); // bingo
}
}

namespace V3 {
	template<typename ActTy> struct Finally {
		ActTy act;
		explicit Finally(ActTy action) : act(std::move(action)) {}
		~Finally() { act(); }
	};
}

int main() {
	int x = 0;
	V1::Finally close_x1{[&] {std::cout << x << std::endl;}};
	auto close_x2 = V2::Finally([&] {std::cout << x << std::endl;});
	V3::Finally close_x3{[&] { std::cout << x << std::endl;}};
	x = 42;
}
```
#### Предостережение
• У вас может возникнуть соблазн сделать функтор с изменяемым состоянием.
```cpp
auto [begin, end] = m.equal_range(i);

int max = 0;
std::for_each(begin, end, [&max](auto&& elt)) {
	int n = f(elt.second);
	if(n > max) max = n;
};

std::cout << "Answer is: " << max << std::endl;
```
• Это не ошибка, но это дурной тон. Чем опасен такой  подход?
## Алгоритмы
#### Алгоритмы
• Алгоритм стандартной библиотеки - это <span style="color: blue;">функция</span>, выполняющая действие над <span style="color: blue;">интервалами</span>, заданными с помощью итераторов.
```cpp
template<class InputIt, class OutputIt>
OutputIt copy(InputIt fst, InputIt last, OutputIt res);
```
• Имя алгоритма может иметь суффиксы и префиксы.
	• if (например, copy_if) - выполнить действие при выполнении предиката.
	• n (например, copy_n) - выполнить действие ограниченное количество раз.
	• copy (например, reverse_copy) - разместить результат в новом контейнере.
	• stable (например, stable_partition) - алгоритм работает стабильно.
• Вопрос - должен ли copy_n существовать в языке? - дискуссионный.
#### Несколько примеров copy_x
```cpp
int arr[] = {2, 3, 5, 7, 11, 13, 17};
std::vector<int> v(7);
std::copy_n(arr, 7, v.begin());
std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, "\n"));
std::fill(v.begin(), v.end(), 0);
std::copy_if(arr, arr + 7, v.begin(), [](int i){ return (i % 3) == 1; });
```
• Контрольный вопрос: что на экране, что в векторе?
#### Обсуждение: специализации циклов
• Есть довольно серьёзное заблуждение, что все алгоритмы - это `for_each`.
• Общее правило: более специализированный алгоритм лучше менее специализированного.
• Алгоритмы в этом смысле можно рассматривать как <span style="color: blue;">специализацию цикла</span>.
	• Обычный `for` - это что угодно, компилятор и библиотека ничего не знают и соптимизируют как угодно.
	• Но тот же `copy` - это именно копирование и ничто иное.
• Важнейший навык - выбирать правильные алгоритмы.
• Для этого надо видеть <span style="color: blue;">паттерны в коде</span>.
#### Паттерны в коде: find_if
• Классический паттерн: `for`, внутри `if`, внутри `break`.
```cpp
for(auto&& elt : v)
	if(check(elt)) {
		action(elt);
		break;
	}
```
• Разумеется, это `std::find_if`.
```cpp
auto it = std::find_if(v.begin(), v.end(), [](auto&& elt) {return check(elt);});
if(it != v.end()) action(elt);
```
#### Иногда и find_if много
• Допустим, вам принесли код, который использует `find_if`.
```cpp
if(std::find_if(v.begin(), v.end(), p) != v.end())
	action();
```
• Его можно и даже нужно упростить далее.
```cpp
if(std::any_of(v.begin(), v.end(), p)) // есть ли элемент?
	action();
```
• Аналогично `std::all_of` и `std::none_of`.
• Мы всегда должны стремиться к выбору самого специализированного алгоритма из существующих.
#### Обсуждение: и это частая проблема
• На самом деле, это частая ситуация. Если вы видите, как кто-то пишет:
```cpp
auto [ita, itb] = std::mismatch(a.begin(), a.end(), b.begin());
if(ita != a.end() && itb != b.end())
	action();
```
• То такому человеку надо вежливо объяснить, что он просто забыл про то, что существует более специализированный способ.
```cpp
if(!std::equal(a.begin(), a.end(), b.begin()))
	action();
```
#### Паттерны в коде: копирование
• Допустим, мы хотим скопировать диапазон `[1,6)`, начиная с позиции перед элементом 4.
• Все ли понимают, почему обычный `std::copy` не подходит.
• Как бы вы выкрутились?
![[../../../_Meta/attachments/17.3.png]]
• В примере по [ссылке](https://godbolt.org/z/8GnbqsP8c) обратите внимание, что мы указываем конец. Как вы думаете, что будет, если мы укажем начало?
#### Паттерны в коде: transform
• То, что в функциональных языках называется `map`, в C++ - это `transform`.
```cpp
vector<int> v = {2, 3, 5, 7, 11, 13};
```
• Неправильно: `for_each` тут пытается что-то делать над текущим элементом.
```cpp
std::for_each(v.begin(), v.end(), [](auto& i) { i = -i; } );
```
• Обратите внимание: при использовании правильного алгоритма легко заменить выходной итератор.
```cpp
auto negate = [](auto&& i) { return -i; }
std::transform(v.begin(), v.end(), v.begin(), negate);
```
![[../../../_Meta/attachments/17.4.png]]
#### Обусуждение: функция как предикат
• Предположим, хочется использовать функцию `std::toupper`. Как написать вызов алгоритма с этим предикатом?
```cpp
std::string s = "hello";

// увы, это не будет работать
std::transform(s.begin(), s.end(), s.begin(), std::toupper);
```
• Причина банальная: `std::toupper` - это множество перегрузки. Компилятор не понимает, какую из перегруженных функций взять.
• Как бы вы выкрутились?
[Решение](https://godbolt.org/z/r4zf9hMd9) на golbolt.
#### Второй смысл transform
• У этого алгоритма есть форма, где он работает как своего рода `zip`.
![[../../../_Meta/attachments/17.5.png]]
• Дополнение к нему `std::transform_reduce` позволяет аккумулировать бинарную операцию над ними.
• Интересно, что то же самое делает `std::inner_product`, но делает это **иначе**.
#### Отличия в гарантиях
• Есть такие пары функций, которые существуют с разными гарантиями на исполнение.
• Например, `reduce` и `accumulate`
	• Обе берут бинарную операцию и считают результат, но:
	• `accumulate` гарантированно сделает это `in-order`.
	• `reduce` сделает это как угодно.
• Ещё пример - это `inclusive_scan` и `partial_sum`.
• Также `transform_reduce` и `inner_product`.
• Мы должны внимательно читать документацию, чтобы осознавать такие вещи.
#### Задача: кстати о чтении документации
• Как вы думаете, что такое `all_of`, `any_of` и `none_of` на пустых диапазонах?
```cpp
std::vector<int> v; // пустой вектора

if(std::all_of(v.begin(), v.end(), p)) // ?
	action_all();
	
if(std::any_of(v.begin(), v.end(), p)) // ?
	action_any();
	
if(std::none_of(v.begin(), v.end(), p)) // ?
	action_none();
```
Результат там true, false, true.
#### Паттерны в коде: remove
• Как бы вы написали функцию `remove`?
• Идея функции: удалить из диапазона все значения `val`.
```cpp
template<typename Iter, typename T>
Iter remove(Iter first, Iter last, const T& val) {
	// что здесь?
}
```
#### Некоторая засада с remove
• Правильный ответ: <span style="color: red;">никак</span>. По итератору нечто можно удалить из контейнера толкьо используя `Cont.erase(it)`.
#### Идиома erase-remove
• Как же по-настоящему работает `remove`?
```cpp
std::vector<int> v = {1, 42, 2, 42, 3, 42, 4};

auto result = std::remove(v.begin(), v.end(), 42);
v.erase(result, v.end());
```
• Или, группируя это в одну фразу:
```cpp
v.erase(std::remove(v.begin(), v.end(), 42), v.end());
```
• Эта техника называется <span style="color: cyan;">"идиома erase-remove"</span>.
![[../../../_Meta/attachments/17.6.png]]
#### Обсуждение: не только remove
• Среди алгоритмов не только `remove` "удаляет" элементы.
• Например, `unique`.
```cpp
vector v = {1, 1, 2, 2, 3, 3, 4};
std::sort(v.begin(), v.end());
v.erase(std::unique(v.begin(), v.end()), v.end());
```
• Это тоже идиома `erase-remove` только без `remove`.
• К счастью, пока что в стандарте C++ только три таких алгоритма: `remove`, `remove_if` и `unique`. Но в пользовательском коде может попасться всякое.
#### Паттерны в коде: перемещение группы
![[../../../_Meta/attachments/17.7.png]]
• Три параметра:
	• начало группы **f, \*f == 3**
	• конец группы **l, \*l == 6**
	• позиция **p**, куда группа должна быть перемещена, **\*p == 8**
• Как бы вы написали такое перемещение?
#### Внезапно rotate
• `rotate` работает следующим образом:
```cpp
void rotate(FwIt first, FwIt n_first, FwIt last);
```
• Диапазон от `first` до `last` проворачивается так, чтобы первым элементом стал `n_first`.
• Ментальная модель `rotate(f, l, p)` - это перенос группы `[f, l)` в позицию перед `p`.
![[../../../_Meta/attachments/17.8.png]]
#### Групповое перемещение - это rotate
• `rotate(f, l, p)` - это перенос группы `[f, l)` в позицию перед `p`.
• Итак, вы бы написали `rotate`?
#### Небольшая проблема
• `rotate` работает следующим образом:
```cpp
void rotate(FwIt first, FwIt n_first, FwIt last);
```
• Но что, если позиция, куда нужно переместить лежит выше f?
• Тогда `rotate(f, l, p')` не будет работать, так как `[f, p')` не образует диапазон.
![[../../../_Meta/attachments/17.9.png]]
#### Решение
• Зато будет работать `rotate(p', f, l)`.
![[../../../_Meta/attachments/17.10.png]]
#### Групповое перемещение элементов
• Три параметра:
	• начало группы f, где угодно
	• конец группы l, где угодно
	• позиция p, куда группа должна быть перемещена, где угодно
• Итого мы хотим сделать так
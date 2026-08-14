<h1 align="center">КОНТЕЙНЕРЫ</h1>

---
<p align="center">Последовательные и ассоциативные контейнеры, адаптеры и вспомогательные классы.</p>
## Последовательные контейнеры
#### Последовательные контейнеры
• Контейнеры
	◦ **vector** - массив с переменным размером и гарантией непрерывности памяти*
	◦ **array** - массив с фиксированным размером, известным в момент компиляции
	• **deque** - массив с переменным размером без гарантий по памяти
	• **list** - двусвязный список
	• **forward_list** - односвязный список
• Адаптеры
	• stack - FIFO-контейнер, чаще всего на базе deque
	• queue - LIFO-контейнер, чаще всего на базе deque
	• priority_queue - очередь с приоритетами, чаще всего на базе vector
<span style="color: gray;">* When choosing a container, remember vector is best. Leave a comment to explain if you choose from the rest. (c) Tony van Eerd</span>
#### Что может смущать в этом коде?
```cpp
std::deque<int> d; // подумайте, если бы это был vector?

for(int i = 0; i != N; ++i) {
	d.push_front(i);
	d.push_back(i);
}
```
• **deque** - массив с переменным размером без гарантий по памяти.
• Поэтому ответ: всё хорошо.
• Вставка в начало и в конец дека имеет всегда честную константную сложность *O*(1).
#### Рассмотрим deque вместо vector*
• Эффективно растёт в обоих направлениях.
• Не требует больших реаллокаций с перемещениями, так как разбит на блоки.
• Гораздо меньше фрагментирует кучу.
![[../../../_Meta/attachments/16.1.png]]
\**Но оставьте комментарий в коде, если вы его действительно* ***выберете***.
#### Деки против векторов
#### Вектора
• Доступ к элементу O(1)
• Вставка в конец аморт. O(1)+
• Вставка в начало O(N)
• Вставка в середину O(N)
• Вычисление размера O(1)
• <span style="color: cyan;">Есть гарантии по памяти</span>
• Есть reserve/capacity
#### Деки
• Доступ к элементу O(1)
• <span style="color: cyan;">Вставка в конец O(1)</span>
• <span style="color: cyan;">Вставка в начало O(1)</span>
• Вставка в середину O(N)
• Вычисление размера O(1)
• Нет гарантий по памяти
• <span style="color: cyan;">Нет необходимости в reserve/capacity</span>
#### Обсуждение
• <span style="color: brown;">"deque is the data structure of choice when most insertions and deletions take place at the beginning or at the end of the sequence"</span>
• А как бы вы реализовали deque?
#### Узловые контейнеры
• deque - произвольный доступ, быстрая вставка в начало и в конец.
• forward_list - последовательный доступ, быстрая вставка в любое место.
• list - последовательный доступ, быстрая вставка в любое место, итерация в обе стороны.
![[../../../_Meta/attachments/16.2.png]]
#### Особая возможность списков: сплайс
![[../../../_Meta/attachments/16.3.png]]
#### Сплайс для списков: простая форма
```cpp
forward_list<int> fst = { 1, 2, 3 };
forward_list<int> snd = { 10, 20, 30 };
auto it = fst.begin(); // указывает на 1

// перемещаем second в начало first, it указывает на 1
fst.splice_after(fst.before_begin(), snd);
```
![[../../../_Meta/attachments/16.4.png]]
#### Сплайс для списков: сложная форма
```cpp
// forward_list<int> fst = {10, 20, 30, 1, 2, 3};
// forward_list<int> snd = {};
// it указывает на 1

// перекидываем элементы со второго по it в список second
snd.splice_after(snd.before_begin(), fst, fst.begin(), it);
```
![[../../../_Meta/attachments/16.5.png]]
#### Сплайс для списков: средняя форма
```cpp
// forward_list<int> fst = { 10, 1, 2, 3 };
// forward_list<int> snd = { 20, 30 };
// it указывает на 1

// все элементы второго списка, начиная со второго, в первый
fst.splice_after(fst.before_begin(), snd, snd.begin());
```
![[../../../_Meta/attachments/16.6.png]]
#### Обсуждение
• Какие вы видите применения спискам?
#### Идея контейнерных адаптеров
![[../../../_Meta/attachments/16.7.png]]
#### Виды адаптеров
• stack - LIFO стек над последовательным контейнером.
```cpp
template<class T, class Container = deque<T>> class stack;
```
• queue - FIFO очередь над последовательным контейнером.
```cpp
template<class T, class Container = deque<T>> class queue;
```
• priority_queue - очередь с приоритетами (как binary heap) над последовательным контейнером.
```cpp
template<class T,
				 class Container = vector<T>,
				 class Compare = less<typename Container::value_type>>
class priority_queue;
```
#### Case study: алгоритм Прима
```cpp
pq.push(std::make_pair(first(G), src)); // first(G)

while(!pq.empty()) {
	auto elt = pq.top().second; pq.pop();
	if(mst[elt]) continue;                // mst[v]
	for(auto e : adjacent(G, elt)) {      // adjacent(G, v)
		w = wight(G, e); v = tip(G, e);     // weight(G,e); tip(G,e)
		if(!mst[elt] && key[v] < w) {       // key[v]
			key[v] = w; parent[v] = u;        // parent[v]
			pq.push(std::make_pair(w, v));
		}
	}
}
```
#### Защита от ортогональности
```cpp
std::stack<int> s; // ok, это stack<int, deque<int>>
std::stack<int, std::vector<long>> s1; // сомнительно
std::stack<int, std::vector<char>> s2; // совсем плохо
s2.push(1000);
// Что вернёт s2.top()?
```
• К счастью, всё это безобразие перекрыто static asserts.
#### Недостаточная ортогональность
```cpp
std::stack<int, std::forward_list<int>> s; // ok
s.push(100); // ошибка: нет push_back
s.pop(); // ошибка: нет pop_back
s.top(); // ошибка: нет back
```
• Эти ошибки неочевидны.
• Стек вполне может быть сделан на односвязном списке.
• Но адаптер `std::stack` требует (неявно требует) вполне определённый интерфейс.
#### Обсуждение
• Почему стек, очередь и очередь с приорететами не отдельные контейнеры?
• И почему двухголовая очередь deque не адаптер?
## Контейнеро-подобные классы
#### Коротко о битовых массивах
• `bitset` - это альтернатива `array<bool>`. То есть, у него фиксированный размер, являющий параметром контейнера.
• При этом, он хранит данные более компактно (как `vector<bool>`).
```cpp
// 24-bit number
bitset<24> s1 = 0x7ff000;
bitset<24> s2 = 0xff00;

s1[0] = 1; // или s1.set(0) или s1.set(0, 1)

auto s3 = s1 & s2; // s3 = 0xf000
```
• По сути, он делает `array<bool>` ненужным.
#### Обсуждение: поговорим о строках
• Почему специальный `std::string`, а не `vector<char>`?
• Важная ремарка: формально `std::string` - это непрерывный контейнер, имеющий с вектором много общего.
#### Строки: базовая функциональность
\#include \<cstring\>
\#include \<cassert\>

char astr\[\] = "hello";
char bstr\[15\];
int alen = <span style="color: blue;">std::strlen</span>(astr);
assert(alen == 5);
<span style="color: blue;">std::strcpy</span>(bstr, astr);
<span style="color: blue;">std::strcat</span>(bstr, ", world!");
res = <span style="color: blue;">std::strcmp</span>(astr, bstr);
assert(res < 0);
foo(bstr);

\#include \<string\>
using std::string;

string astr = "hello";
string bstr;
int alen = <span style="color: blue;">astr.length();</span>
assert(alen == 5);
<span style="color: blue;">bstr = astr;</span>
<span style="color: blue;">bstr += ", world!";</span>
res = <span style="color: blue;">astr.compare(bstr);</span>
assert(res < 0);
foo(bstr<span style="color: blue;">.c_str()</span>);
#### Шаблон класса строки
• Представим (это не так), что строка была бы устроена вот так:
```cpp
template<typename CharT> class basic_string { /*....*/ };
```
• Определения для удобства:
```cpp
typedef basic_string<char> string;
typedef basic_string<u16char_t> u16string;
typedef basic_string<u32char_t> u32string;
typedef basic_string<wchar_t> wstring;
```
• Что бросается в глаза?
#### Характеристики типов
• Есть много вопросов, ответы на которые разные для разных строк с разными типами символов. Разумно свести всё это в класс
```cpp
template<typename CharT> class char_traits;
```
• Основные методы:
• `assign`, `eq`, `lt`, `move`, `compare`, `find`, `eof`, ....
```cpp
template<typename CharT, typename Traits = std::char_traits<CharT>>
class basic_string {
```
• К слову, а является ли способ выделения памяти характеристикой символа?
#### Аллокаторы
• Выделение памяти абстрагирует аллокатор. Стандартный аллокатор сводится к malloc.
```cpp
template<typename CharT,
				 typename Traits = std::char_traits<CharT>,
				 typename Allocator = std::allocator<CharT>>
class basic_string { /*....*/ }
```
• К слову, полный шаблон вектора тоже выглядит не вполне очевидно:
```cpp
template<typename T, typename Allocator = std::allocator<T>>
class vector { /*....*/ }
```
#### Обсуждение
• Следующие вопросы не слишком логически связаны.
• Как, по вашему, выглядит аллокатор для `std::list`?
• Как вы думаете, строка должна иметь методы вроде `reserve` и `capacity`?
• Ну и, раз уж мы вынесли строку в отдельный класс, что вы думаете о специальных интерфейсах для неё?
#### Поиск в строках
• Строки предлагают эффективные специальные возможности поиска в них.
```cpp
string s = "Hello";

unsigned long notfound = s.find("bye");
assert(notfound == std::string::npos);

unsigned long ellp = s.find("ell");
unsigned long hpos = s.find("H", ellp);
assert(hpos == std::string::npos);
```
![[../../../_Meta/attachments/16.8.png]]
• Кто видит возможную проблему в этом коде?
• Но использование этих возможностей таит сюрпризы:
```cpp
using szt = std::string::size_type;

string s = "Hello";

szt notfound = s.find("bye");
assert(notfound == std::string::npos);

szt ellp = s.find("ell");
szt hpos = s.find("H", ellp);
assert(hpos == std::string::npos);
```
![[../../../_Meta/attachments/16.9.png]]
#### Проблема статических строк
• Что вы думаете об использовании константных статических строк?
```cpp
static const std::string kName = "oh literal, my literal";
// .....
int foo(const std::string& arg);
// .....
foo(kName);
```
#### Решение: string_view(C++17)
• `string_view` - это невладеющий указатель на строку.
```cpp
static std::string_view kName = "oh literal, my literal";
// .....
int foo(std::string_view arg);
// .....
foo(kName);
```
• Здесь нет ни heap indirection, ни создания временного объекта.
#### Базовые операции над string_view
• remove_prefix
• remove_suffix
• copy
• substr
• compare
• find
• data
```cpp
std::string str = "    trim me  ";
str::string_view sv = str;

auto trimfst = sv.find_first_not_of(" ");;
auto minsz = std::min(trimfst, sv.size());

sv.remove_prefix(minsz);

auto trimlst = sv.find_last_not_of(" ");
auto sz = sv.size() - 1;
minsz = std::min(trimlst, sz);

sv.remove_suffix(sz - minsz);
```
#### Views: идея для span (C++20)
• `std::span` для одномерных массивов то же, что `string_view` для строк.
```cpp
int arr[4] = {1, 2, 3, 4}; // просто данные
std::array<int, 4> = {1, 2, 3, 4}; // копирование до main
```
• span решает эту проблему.
```cpp
std::span<int, 4> arr = {1, 2, 3, 4}; // просто данные
```
• По умолчанию, второй параметр N - это `std::dynamic_extent`/
```cpp
std::span<int> dynarr(arr); // неизвестный размер
```
• Разумеется, у него куда более простой интерфейс, чем у string view.
#### Обсуждение
• Хватит ли нам последовательных контейнеров?
## Ассоциативные контейнеры
#### Смысл ассоциативности
• Вектора индексированы целыми числами и позволяют сопоставить целое число хранимому значению.
```cpp
vector<T> v; // int -> T
```
• Как сделать произвольное отображение `T -> U`?
#### Ассоциативный массив
• Основная идея ассоциативного массива - это контейнер unordered map.
```cpp
template<
	typename Key, typename T,
	typename Hash = std::hash<Key>,
	typename KeyEqual = std::equal_to<Key>,
	typename Allocator = std::allocator<std::pair<const Key, T>>
> class unordered_map;
```
• Здесь важными являются два отношения: отношение equals и, собственно, hash-функция.
• При этом, ключи уникальны, и мы можем менять значения, но не ключи.
#### Обсуждение: собственный ключ
• Допустим, у нас есть пользовательская структура из двух строк
```cpp
struct S { std::string first_name, last_name; };
std::unordered_map<S, std::string> Ump; // error
```
• Для неё нужно сделать две вещи:
	• Определить равенство (все ли помнят как?)
	• Определить хеш. Есть тут у вас идеи, как именно? Хорош ли вариант по [ссылке](https://godbolt.org/z/4E4W38WjM)?
• Обратите внимание: мы можем добавлять в стандартную библиотеку специализации.
#### Собственный hash
• Простейший способ - это сделать что-нибудь, исходя из фантазии.
```cpp
size_t operator()(const S& s) const noexcept {
	std::hash<std::string> h;
	auto h1 = h(s.first_name), h2 = h(s.last_name);
	return h1 ^ (h2 << 1);
}
```
• Этот способ привлекателен, так как мы же программисты.
• Часто (например, в этом случае) он даже работает.
• Но, в общем, это всегда угадайка.
• Если угадайка не привлекает, есть boost.
```cpp
size_t operator()(const S& s) const noexcept {
	std::hash<std::string> h;
	auto h1 = h(s.first_name), h2 = h(s.last_name);
	size_t seed = 0;
	boost::hash_combine(seed, h1);
	boost::hash_combine(seed, h2);
	return seed;
}
```
• Это работает всегда. Но это boost, его надо затаскивать в проект.
#### Требования к предикату сравнения
• Общая концепция называется strict weak ordering.
• Она включает:
	• Антисимметричность: $pred(x, y) \Rightarrow \neg pred(y, x)$
	• Транзитивность: $pred(x, y) \land pred(y, z) \Rightarrow pred(x, z)$
	• Иррефлексивность: $\neg pred(x, x)$
	• Транзитивность эквивалентности:
	$eq(x, y) \equiv \neg pred(x, y) \land \neg pred(y, x) \vdash eq(x, y) \land eq(y, z) \Rightarrow eq(x, z)$
• Она же распространяется на предикаты в алгоритмах сортировки и т.д.
• Математическая разминка: пусть $(a + ib < c + id) \Leftrightarrow (a < c) \land (b > d)$ является ли это strict weak ordering для комплексных чисел?
#### Представление в памяти*
• О хеш-таблицах можно думать как о массиве корзин (buckets), каждая из которых содержит элементы с одинаковым хешом.
• Это даёт асимптотически быстрый поиск (индексация по массиву), если load factor хорош.
• load factor = size / bucket count.
• На картинке снизу это 0.75 и, в общем, это уже довольно плохо.
![[../../../_Meta/attachments/16.10.png]]
<span style="color: gray;">* это не настоящее представление в памяти</span>
#### Низкоуровневая информация
• Дополнительно каждый неупорядоченный контейнер даёт возможность смотреть его статистику.
• `bucket_count()` - количество бакетов
• `max_bucket_count()` - максимальное количество бакетов без реаллокаций
• `bucket_size(n)` - размер бакета с номером n
• `bucket(Key)` - номер бакета для ключа Key
• `load_factor()` - среднее количество ключей в бакете
• `max_load_factor()` - максимальное количество ключей в бакете
#### Обсуждение
• По сути, неупорядоченный контейнер - это что-то вроде гибрида непрерывного и узлового последовательного контейнера.
• Что это означает в практическом смысле в плане управления памятью?
• Напомню: в узловых контейнерах (list) управлять памятью не нужно кроме случаев особых аллокаторов. А в последовательных (vector) об этом нельзя забывать.
#### Рехэш
• Особая функция `rehash(count)` служит для того, чтобы изменить количество бакетов (установить в count) и перераспределить по ним элементы.
• `reserve(count)` делает то же самое, что `rehash(ceil(count / max_load_factor()))`
• Особый случай `rehash(0)` позволяет безусловно (в автоматическом режиме) перехешировать контейнер.
#### Резервирование памяти
• Следующий эксперимент показывает эффект резервирования.
```cpp
std::unordered_map<int, Foo> mapNoReserve, mapReserve;
// контрольная точка 1

mapReserve.reserve(1000);
// контрольная точка 2

for(int i = 0; i < 1000; ++i) {
	mapNoReserve.emplace(i, Foo());
	mapReserve.emplace(i, Foo());
}
// контрольная точка 3
```
#### Два вида итерации
• По хеш-таблице можно итерировать как по единому целому.
```cpp
for(aito it = m.begin(); it != m.end(); ++it)
```
• Можно итерироваться внутри бакета, указав его номер.
```cpp
for(int i = 0; i < m.bucket_count(); ++i) {
	for(auto it = m.begin(); it != m.end(i); ++it)
```
• В обоих случаях вам доступен только forward iterator.
• Как бы вы написали адаптор, чтобы позволить второй вариант через range based for?
Пример с godbolt:
```cpp
#include <iostream>
#include <unordered_map>

int main() {
	std::unordered_map<int, int> m;
	m.reserve(10);
	m.max_load_factor(10.0);
	for(int i = 0; i < 100; ++i)
		m.insert(std::make_pair(i, i));
	
	for(auto it = m.begin(); it != m.end(); ++it)
		std::cout << it->first << " ";
	std::cout << std::endl;
	
	for(int i = 0; i < m.bucket_count(); ++i) {
		std::cout << "Bucket #" << i << std::endl;
		for(auto it = m.begin(i); it != m.end(i); ++it)
			std::cout << it->first; << " ";
		std::cout << std::endl;
	}
}
```
Ну и далее в первом выводе видно, что все числа перемешаны, а в дальнейших видно, как они раскиданы по бакетам.
#### Представление в памяти
• На самом деле, в растространённых реализациях (libstdc++, etc) таблица представлена списком элементов, каждый из которых хранит свой хеш и вектором указателей на начало блока.
• Стандарт устроен так, что это практически единственный способ выполнить все его ограничения.
![[../../../_Meta/attachments/16.11.png]]
#### Обсуждение: отказ от хранения
• Идея для оптимизации - это отказ от хранения.
• Вместо того, чтобы хранить хеш, мы вычисляем хеш каждый раз, когда смотрим бакет.
• Что вы думаете про эту оптимизацию?
![[../../../_Meta/attachments/16.12.png]]
• Так как unordered map - это, по сути, список, гарантии по итераторам для него **как для списка**. И даже для рехеша.
• Не можем ли мы улучшить наше отображение, убрав строгие гарантии по итераторам?
#### Первая идея: node_map
• Мы можем отказаться от хранения указателя в списке бакетов.
• Это лишает нас гарантий по итераторам при рехеше и ставит нас перед лицом внезапных реаллокаций.
• Кроме того, мы усложняем (фактически теряем) итерацию по бакетам.
• Кстати, как бы вы организовали быстрый переход к началу бакета при таком подходе?
• Этот контейнер довольно популярен в библиотеке Abseil от Google.
![[../../../_Meta/attachments/16.13.png]]
#### Интермедия: алгоритмический базис
• Таблицы, в которых мы точно не знаем по хешу номер бакета, называются таблицами с открытой адресацией (в противоположность прямой адресации).
• При открытой адресации используется probing (исследование) ячеек.
$$h(x) = (h'(x) + i) \bmod m$$
• Здесь функция может быть линейной по i, квадратичной или даже более сложной (см. двойное хеширование).
$$h(x) = \left(h'(x) + ih''(x)\right) \bmod m$$
• В принципе, именно открытая адресация подсказывает нам следующую идею.
Далее лектор рекомендует ознакомиться с [этой](https://www.sebastiansylvan.com/post/robin-hood-hashing-should-be-your-default-hash-table-implementation/#:~:text=Robin%20Hood%20Hashing%20should%20be%20your%20default%20Hash%20Table%20implementation,-8%2FMay%202013&text=There's%20a%20neat%20variation%20on,tables%20called%20Robin%20Hood%20hashing.) статьёй о Robin Hood Hashing.
#### Вторая идея: flat map
• Мы можем, в принципе, хранить всё как один вектор.
![[../../../_Meta/attachments/16.14.png]]
• Да, мы теряем все гарантии по итераторам, и всё такое.
• Но мы приобретаем потрясающую локальность кешей, и работать с этим практически также приятно, как с векторами.
#### Обсуждение
• Многие критикуют unordered-контейнеры за то, что стандарт заперт ограничениями, позволяющими только неэффективную реализацию. Максимум, с пробингом.
• С другой стороны, в **стандартной** библиотеке должно быть нечто, удобное всем. Для прочего есть abseil и folly.
![[../../../_Meta/attachments/16.15.png]]
#### Загадочные квадратные скобки
• Поскольку ассоциативный массив - это массив, для него сделали удобное массиво-подобное обращение:
```cpp
std::unordered_map<int, int> m = {{1, 20}, {100, 30}};
auto& x = m[100];
```
• Это эквивилентно вот чему:
```cpp
auto p = m.emplace(100, int{});
auto it = p.first; auto b = p.second;
if(!b) it = m.find(100);
auto& x = it->second;
```
• Тут сразу видно два ограничения: оператор квадратные скобки не константный, и у ключа должен быть конструктор по умолчанию.
#### Кстати о квадратных скобках
• Также можно использовать особый синтаксис auto, развязывающий пару:
```cpp
auto [it, b] = m.emplace(100, int{});
if(!b) it = m.find(100);
auto& x = it->second;
```
• Он называется structured binding.
#### Неупорядоченные множества
• Особый вид `unordered_map`, который хранит только ключи называется `unordered_set`.
• Вы можете рассматривать `unordered_set` как массив с дешёвым поиском из уникальных элементов.
```cpp
std::unordered_set s = {1, 2, 2, 2, 1}; // = {1, 2}
```
• Поддержка инварианта уникальности и поиска (в случае вектора нужна сортированность) дешевле, чем для вектора.
#### Case study: орбита в группе
• Группой называется множество элементов с групповой операцией над ними.
• Например, группа $\{Z_7, \times\}$ это числа $\{1 \dots 6\}$ с операцией умножения $\bmod 7$.
• Зададимся генерирующими элементами группы, например $\{3, 5\}$.
• Тогда у любого элемента будет **орбита**: все элементы, которые можно получить, умножая его на генераторы, умножая поличшиеся результаты на генераторы и т.д.
• Естественный контейнер для хранения орбиты - это `unordered_set`, т.к. вектор при вставке придётся пересортировывать и удалять дубликаты.
Пример из гита:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Simplest example: blank square on the screen
// This example also show how to use GLFW library which is rather standard
// Note really short time to first triangle (quad in this case)
//
// cl /EHsc ogl-simplest.cc /link glfw3dll.lib opengl32.lib
//
//-------------------------------------------------------------------------------

#include <algorithm>
#include <iostream>
#include <iterator>
#include <unordered_set>
#include <vector>

struct Z7 {
	int val;
	Z7(int v = 0) : val(v % 7) {}
	operator int() const { return val; }
	auto operator <=>(const Z7&) const = default;
};

namespace std {
template<> struct hash<Z7>  {
	std::size_t operator()(Z7 K) const {
		std::hash<int> h;
		return h(K.val);
	}
};
} // namespace std

Z7 operator*(Z7 lhs, Z7 rhs) { return {(lhs.val * rhs.val) % 7}; }

template<typename T, typename RandIt>
auto orbit(T num, RandIt gensbeg, RandIt gensend) {
	std::unordered_set<T> orbit;
	std::vector<T> next = {num};
	while(!next.empty()) {
		std::vector<T> tmp{};
		orbit.insert(next.begin(), next.end());
		for(const auto& elem : next)
			for(auto igen = gensbeg; igen != gensend; ++igen)
				if(auto newelem = (*igen) * elem; orbit.count(newelem) == 0) {
					std::cout << elem << " * " << *igen << " = " << newelem << std::endl;
					tmp.push_back(newelem);
				}
		next.swap(tmp);
	}
	return orbit;
}

int main() {
	std::vector gens = {Z7{3}, Z7{5}};
	// ....
}
```
#### Обсуждение
• Чем `unordered_set` **хуже**, чем сортированный массив?
	• Оно не позволяет range-based queries.
	• Оно не хранит повторные элементы.
• Второе решается с помощью мультиконтейнера `unordered_multiset`.
• К слову, видите ли вы применения для `unordered_multimap`?
## Упорядоченные контейнеры
#### Уникальность элементов
• Упорядоченное множество также хранит уникальные элементы:
```cpp
std::set<int> s = {67, 42, 141, 23, 42, 106, 15, 50};
for(auto elt : s) cout << elt << endl;
```
• Ничего не сломается, но на экране будет:
```
15, 23, 42, 50, 67, 106, 141
```
• Главное отличие от `unordered_set`: оно хранит их именно что упорядоченно.
• Это позволяет `range-based queries` через `upper` и `lower` `bound`.
#### Порядок сравнения
• Множество создаёт упорядочение своих элементов.
```cpp
std::set<int> s = {67, 42, 141, 23, 42, 106, 15, 50};

auto itb = s.lower_bound(30);
auto ite = s.upper_bound(100);
```
• Теперь можно итерировать в интервале `[30, 100)` независимо от того, есть ли во множестве в точности такие элементы.
```cpp
for(auto it = itb; it != ite; ++it)
	std::cout << *it << std::endl;
```
• Что на экране?
• Можно задать любой предикат упорядочения.
```cpp
std::set<int, std::greater<int>> s = {67, 42, 141, 23, 42, 106, 15, 50};

auto itb = s.lower_bound(30);
auto ite = s.upper_bound(100);
```
• Задают ли итераторы itb и ite валидный интервал для итерирования?
• Что будет, например, при таком цикле?
```cpp
for(auto it = itb; it != ite; ++it)
	std::cout << *it << std::endl;
```
• Прошлый  интервал был невалиден. Вот исправления:
```cpp
auto itb = s.lower_bound(100);
auto ite = s.upper_bound(30);
```
• Теперь всё хорошо, но это крайне контринтуитивно.
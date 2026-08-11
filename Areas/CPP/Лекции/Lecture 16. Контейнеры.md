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

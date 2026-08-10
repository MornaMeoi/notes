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

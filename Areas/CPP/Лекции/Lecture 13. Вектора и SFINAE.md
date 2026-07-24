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
• vector<bool> не удовлетворя
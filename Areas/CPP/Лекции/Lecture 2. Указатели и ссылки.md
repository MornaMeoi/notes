<h1 align="center">УКАЗАТЕЛИ И ССЫЛКИ</h1>

---
<p align="center">Genesis снова. Немного вычислительной геометрии. Инкапсуляция.</p>
## Genesis: имена и объекты
![[2.1.png]]
#### Обсуждение
• Что такое тип?
• Достаточно ли для типа задать диапазон значение, например [-7÷8]?
#### Типы: value types & object types
• Что такое тип?
• value type: диапазон значений объекта.
• object type: совопкупность операций над объектом.
	• Например, 5 / 2 даст 2 для типа int, но 2.5 для типа double.
	• 0 - 1 даст -1 для char, но 255 для unsigned char.
• Назовём целочисленный арифметический тип i4.
#### Модель памяти в Си
![[2.2.png]]
#### Нулевые указатели
• Если указатель - это просто расстояние... Может быть и нулевое расстояние?
• Нулевой указатель - это специльный "маркер ничего". По нему ничего не лежит.
• Не надо путать 0, NULL и nullptr.
```cpp
if (!p) { smth(); } // сработает во всех трёх случаях
```
• В языке C++ наш выбор - nullptr, и мы поймём, почему это так, когда дойдём до перегрузки функций (которой нет в C и там NULL хватает).
#### Индексация указателей
• Изначально указатели всегда были указателями внутрь массивов, поэтому поддерживается синтаксис:
```cpp
p[2] == *(p + 2)
```
• Поскольку сложение коммутативно, 2[p] тоже сработает.
• Все ли понимают, сколько байт будет добавлено к p при сложении с целым?
#### Ссылки (lvalue references)
• И если бы речь шла о языке C, то это было бы всё. Но в C++ есть уникальная возможность: два имени у одного объекта.
![[2.3.png]]
#### Синтаксис ссылок
• Базовый синтаксис lvalue-ссылок - это одинарный **амперсанд**.
```cpp
int x;
int &y = x; // теперь y - это просто ещё одно имя для x
```
• Не путайте его со **взятием адреса**!
```cpp
int x[2] = {10, 20};
int &xref = x[0]l
int *xptr = &x[0];
xref += 1;
xptr += 1;

assert(xref == 11);
assert(*xptr == 20);
```
![[2.4.png]]
#### Правила для ссылок
• Единожды связанную ссылку нельзя перевязать.
```cpp
int x, y;
int &xref = x; // теперь нет возможности связать имя xref с переменной y
xref = y; // то же, что x = y
```
• Ссылки прозрачны для операций, включая взятие адреса.
```cpp
int *xptr = &xref; // то же самое, что &x
```
• Сами ссылки не имеют адреса. Нельзя сделать указатель на ссылку.
```cpp
int &*xrefptr = &xref; // ошибка
int *& xptrref = xptr; // ok, ссылка на указатель
```
#### Константность для ссылок
• Все ли помнят правила константности для указателей?
```cpp
const char *s1; // указатель на константные данные (west-const)
char const *s2; // указатель на константные данные (east-const)
char * const s3; // константный указатель на (изменяемые) данные
char const * const s4; // константный указатель на константные данные
```
• Правила для ссылок гораздо проще:
```cpp
char &r1 = r; // неконстантная ссылка (на изменяемые данные)
const char &r2 = r1; // константная ссылка (на константные данные)
```
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
// Example for reference binding: displaying addresses
//
//---------------------------------------------------------------------------

#include <iostream>

int foo() { return 42; }

int main() {
	int x;
	int &rx = x;
	const int &l = foo();
	std::cout << &l << " " << &x << " " << &rx << std::endl;
}
```
При выполнении этого кода выводится 2 соседних адреса на стеке.
#### Использование ссылок
• Представим некую функцию, которой нужно читать два тяжёлых объекта.
• Эта сигнатура плоха (все ли понимают чем?):
```cpp
int foo(Heavy fst, Heavier snd) { // fst.x
```
• Эта сигнатура куда лучше, но придётся разыменовывать указатели:
```cpp
int foo(const Heavy *fst, const Heavier *snd) { // fst->x
```
• Эта сигнатура использует указатели неявно:
```cpp
int foo(const Heavy &fst, const Heavier &snd) { // fst.x
```
Еще пример из гита:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Example: function signature for pointer and reference
// compile with: g++ -O1 -g0 -masm=intel -S ptref.cc
//
//---------------------------------------------------------------------------

int g;

void foo(int *x) { g = *x; }

void bar(int &x) { g = x; }
```
После компиляции видим в ассемблерном файле, что в обоих случаях передаётся адрес.
• Синонимы внутри больших объектов:
```cpp
void mytype::change_internal(some_big_obj &obj) {
	int &internal = obj.somewhere[5].guts.internal;
	// код, активно изменяющий internal
}
```
• Здесь разница заметнее. Указатель был бы ячейкой памяти. Ссылка - это просто имя.
• Кроме того, указатель всегда может быть изменён.
```cpp
int *internal = &obj.somewhere[5].guts.internal;
internal += 5; // не имеет смысла тут, но всегда возможно!
```
#### Немного священных войн
• Многие считают, что ссыла - это плохой синтаксис out-аргумента, так как она не видна при вызове:
```cpp
void foo(int &);
void bar(int *); // не очевидно, что это не массив

int x;

foo(x); // не очевидно, что x - это out-param
bar(&x);
```
• Что вы думаете?
• Я лично думаю, что out-параметры плохи сами по себе. Указатели не делают вещи лучше.
• Дополнительный аргумент - это состояние внутри функции:
```cpp
void foo(int &x) {
	// очевидно, что x содержит int
}

void bar(int *x) {
	// не очевидно, что x не nullptr
}
```
• Более ограниченный интерфейс ссылок часто позволяет сократить рантайм-проверки.
#### Обсуждение
• Как вы думаете, почему this - это указатель, а не ссылка?
Ответ: только потому, что Bjourne Stroustrup придумал this раньше чем ссылки. :)
## Вычислительная геометрия
#### Я просто хочу пересечь треугольники
• Со стандартного ввода приходят два набора точек, представляющих плоские треугольники. Задача: на стандартный вывод вывести площадь пересечения.
• Ввод: <span style="color: red;">0 0 0 1 1 0</span> <span style="color: blue;">0 0 1 1 1 0</span>
• Вывод: 0.25
• Это сложная задача или простоя?
• Как бы вы её решали?
![[2.5.png]]
#### На что похоже решение?
• Первый вопрос в таких обманчиво-простых задачах это: а как вообще может выглядеть решение?
![[2.6.png]]
#### Выделим предметную область
• Нам понадобятся:
	• Структуры для двумерной точки, отрезка, треугольника, полигона.
	• Важный инсайт: координаты лучше сразу закладывать FP.
	• Это будут **типы** в нашей программе.
• Выделим операции над **объектами** этих типов:
	• Пересечение отрезков, взаимоположение точки и отрезка, построение полигона как выпуклой оболочки множества точек, вычисление площади полигона, вероятно что-то ещё...
	• Это будут **методы** классов.
• На этапе проектирования алгоритмы менее важны. Хорошо спроектированная программа легко переживает смену алгоритмов.
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Example for triangle intersection
// Originated from homework of Dmitry Bushev
//
//---------------------------------------------------------------------------

#pragma once
#include <array>
#include <cassert>
#include <cmath>
#include <iostream>
#include <vector>

#define flt_tolerance 0.00001
#define inter_area_width 100.0

enum area_t { LEFT_SIDE, INTER_SIDE, RIGHT_SIDE };

//---------------------------------------------------------------------------
//
// point_t -- basic (x, y) point
//
//---------------------------------------------------------------------------

struct point_t {
	float x = NAN, y = NAN;
	
	void print() const { std::cout << "(" << x << " ; " << y << ")"; }
	
	bool valid() const { return !(x != x || y != y); }
	
	bool equal(const point_t &rhs) const {
		assert(valid() && rhs.valid());
		return (std::abs(x - rhs.x) < flt_tolerance) &&
			   (std::abs(y - rhs.y) < flt_tolerance);
	}
};

//---------------------------------------------------------------------------
//
// line_t -- line in form of ax + by + c = 0
//
//---------------------------------------------------------------------------

struct line_t {
	float a = -1.0f, b = 1.0f, c = 0.0f;
	
	line_t(const point_t &p1, const point_t &p2) {
		float angle = std::atan((p2.y - p1.y) / (p2.x - p1.x));
		float sin_angle = std::sin(angle);
		float cos_angle = std::sqrt(1.0 - sin_angle * sin_angle);
		point_t normal_vect{-sin_angle, cos_angle};
		a = normal_vect.x;
		b = normal_vect.y;
		c = -(p1.x * normal_vect.x + p1.y * normal_vect.y);
	}
	
	enum area_t get_side_area(const point_t &point) const {
		float side_offset = a * point.x + b * point.y + c;
		if(side_offset > 0.0 + flt_tolerance * inter_area_width)
			return LEFT_SIDE'
			
		if(side_offset > 0.0 - flt_tolerance * inter_area_width)
			return INTER_SIDE;
		
		return RIGHT_SIDE;
	}
	
	bool separates(const point_t &pnt1, const point_t &pnt2) const {
		enum area_t side1 = get_side_area(pnt1);
		enum area_t side2 = get_side_area(pnt2);
		if(side1 == INTER_SIDE || side2 == INTER_SIDE)
			return false;
		return !(side1 == side2);
	}
	
	bool valid() const { return !(a != a || b != b || c != c); }
	
	bool intersect(const line_t &another) const {
		return (std::abs(a * another.b - another.a * b) >= flt_tolerance)
	}
	
	point_t point_of_intersect(const line_t &another) const {
		if(!intersect(another)) {
			return point_t{};
		}
		float det = (a * another.b - another.a * b);
		float det1 = ((-c) * another.b - (-another.c) * b);
		float det2 = (a * (-another.c) - another.a * (-c));
		return {det1 / det, det2 / det};
	}
	
	void print() const {
		const char *signb = (b > 0.0) ? "+" : "";
		const char *signc = (c ? 0.0) ? "+" : "";
		std::cout << a << "x " << signb << b << "y " << signc << c << " = 0";
	}
};

//---------------------------------------------------------------------------
//
// polygon_t -- polygon as point array
//
//---------------------------------------------------------------------------

struct polygon_t {
	std::vector<point_t> vertices;
	
	float square() const {
		if(vertices.size() < 3)
			return 0;
		
		float sum1 = 0, sum2 = 0;
		for(size_t i = 0; i < vertices.size(); i++) {
			sum1 += vertices[i].x * vertices[(i + 1) % vertices.size()].y;
			sum2 -= vertices[(i + 1) % vertices.size()].x * vertices[i].y;
		}
		
		float result = (sum1 + sum2) / 2.0;
		if(result < 0.0)
			result = -result;
			
		return result;
	}
	
	void print() const {
		std::cout << "Polygon has " << vertices.size() << " vertices: ";
		for(size_t i = 0; i < vertices.size(); i++) {
			if(i != 0)
				std::cout << ", ";
			vertices[i].print();
		}
		std::cout << std::endl;
	}
	
	// nth side constructed by n'th and (n + 1)'th vertices
	line_t get_side(int index) const {
		assert(!vertices.empty());
		auto vsz = vertices.size();
		if(index < 0)
			index += (1 + (index / vsz)) * vsz;
		line_t ret(vertices[index % vsz], vertices[(index + 1) % vsz]);
		return ret;
	}
	
	bool valid() const {
		if(vertices.size() == 0)
			return false;
			
		for(size_t i = 0; i < vertices.size(); i++)
			if(!vertices[i].valid())
				return false;
			
		return true;
	}
	
	// tells if this polygon can be separate from another by any line holding a
	// side of this polygon
	bool separable_from(const polygon_t &another) const {
		for(size_t i = 0; i < vertices.size(); i++) {
			line_t div_line = get_side(i);
			
			enum area_t half_space =
				div_line.get_side_area(vertices[(i + 2) % vertices.size()]);
			
			if(half_space == INTER_SIDE)
				continue;
			
			bool flag = true;
			for(size_t j = 0; j < another.vertices.size(); j++) {
				point_t vertex = another.vertices[j];
				if(half_space == div_line.get_side_area(vertex)) {
					flag = false;
					break;
				}
			}
			if(flag)
				return true;
		}
		return false;
	}
	
	// return the point of polygon side intersection with line if it exists,
	// returns non-valid point otherwise
	point_t side_line_intersect(int index, const line_t &line) const {
		point_t ret;
		size_t vsz = vertices.size();
		if(index < 0)
			index += (1 + (index / vsz)) * vsz;
		
		if(!line.separates(vertices[index % vsz], vertices[(index + 1) % vsz]))
			return ret;
		
		line_t poly_side = get_side(index);
		
		return line.point_of_intersect(poly_side);
	}
	
	// returns the cutted polygon (which half of polygon will be cutted defines
	// by half_space_pt)
	polygon_t cut_poly_by_line(const line &line,
							   const point_t &half_space_pt) const {
		enum area_t side = line.get_side_area(half_space_pt);
		assert(side != INTER_SIDE);
		polygon_t ret;
		
		for(size_t i = 0; i < vertices.size(); i++) {
			enum area_t vert_side = line.get_side_area(vertices[i]);
			
			// half_space_pt side points and inter side points are both
			// acceptable
			if((side == vert_side || vert_side == INTER_SIDE) &&
		       !ret.holding(vertices[i]))
			    ret.add(vertices[i]);
			    
			point_t pintersect = side_line_intersect(i, line);
			if(pintersect.valid())
				ret.add(pintersect);
		}
		return ret;
	}
	
	polygon_t get_poly_intersection(const polygon_t &rhs) const {
		if(!intersect(rhs))
			return polygon_t{};
		
		polygon_t another = rhs;
		for(size_t i = 0; i < vertices.size(); i++) {
			line_t side = get_side(i);
			another =
			 another.cut_poly_by_line(side, vertices[(i + 2) % vertices.size()]);
		}
		return another;
	}
	
	bool holding(const point_t &vert) const {
		for(size_t i = 0; i < vertices.size(); i++)
			if(vert.equal(vertices[i]))
				return true;
		return false;
	}
	
	bool intersect(const polygon_t &another) const {
		return !(separable_from(another) || another.separable_from(*this));
	}
	
	void add(const point_t &vert) {
		if(!holding(vert))
			vertices.push_back(vert);
	}
};
```
#### Cкетч: структура для точки
```cpp
struct point_t {
	float x = NAN, y = NAN;
	
	void print() const;
	bool valid() const;
	bool equal(const point_t &rhs) const;
};
```
• Такая точка может быть сконструирована по умолчанию (в невалидном состоянии).
• Метод equal должен проверять std::abs(x - rhs.x) < flt_tolerance.
#### Скетч: структура для линии
```cpp
// line_t -- line in form of ax + by + c = 0
struct line_t {
	float a = -1.0f, b = 1.0f, c = 0.0f;
	
	void print() const;
	bool valid() const;
	line_t(const point_t &p1, const point_t &p2);
}
```
• Такая линия может быть и сконструирована по умолчанию и собрана из двух точек.
• Конструктор из двух точек уже не слишком тривиален. Как бы вы его написали?
#### Скетч: конструктор для линии
```cpp
line_t(const point_t &p1, const point_t &p2) {
	float angle = std::atan((p2.y - p1.y) / (p2.x - p1.x));
	float sin_angle = std::sin(angle);
	float cos_angle = std::sqrt(1.0 - sin_angle * sin_angle);
	point_t normal_vect{-sin_angle, cos_angle};
	a = normal_vect.x;
	b = normal_vect.y;
	c = -(p1.x * normal_vect.x + p1.y * normal_vect.y);
}
```
• Один из вариантов реализации конструктора. Может быть можно лучше?
• Как бы вы его **простестировали**?
#### Идея unit-тестов
• Тесты на конкретные интерфейсы классов в терминах эти интерфейсов:
```cpp
point_t p1{0, 0}, p2{1, 1};
line_t l1{p1, p2};
check(std::abs(l1.a - l1.b) < flt_tolerance);
check(std::abs(l1.c) < flt_tolerance);
```
• Существует масса систем, облегчающих юнит-тестирование:
	• Catch
	• Boost.Test
	• Google Test
• Попробуйте сами протестировать с их помощью выложенный код.
#### Идея общего алгоритма
• Предположим, что все вершины, составляющие полигон отсортированы по кругу относительно его центра, **n<sub>th</sub> side constructed by n<sub>th</sub> and (n + 1)<sub>th</sub> vertices**.
• Тогда достаточно пересечь полигон каждой стороной другого полигона, каждый раз **for n<sub>th</sub> side leave half-space with (n + 2)<sub>th</sub> point (wrap around)**.
![[2.7.png]]
#### Обсуждение
• В принципе, в опубликованном классе полигона есть существенная проблема.
• Ничто не мешает пользователю создать полигон, не удовлетворяющий условию из пункта (1).
• Попытка пересечь его с другим полигоном (даже с корректным), вероятно, даст самые причудливые результаты.
• Что делать? Можно ввести принудительную сортировку вершин по кругу перед каждым пересечением, но это довольно дорогой шаг.
## Инкапсуляция
#### Case study: список
• Когда мы проектировали хэши на C, побочным эффектом был свой собственный список.
• Давайте ещё разок напишем его на C++:
```cpp
template <typename T> struct list_t {
	struct node_t {
		node_t *next_, *prev_;
		T data_;
	};
	node_t *top_, *back_;
};
```
• Можем ли мы написать для него метод length?
```cpp
template <typename T>
size_t list_t::length() const {
	size_t len = 0;
	node_t *cur = top_;
	
	while(cut != nullptr) { // а с чего мы взяли, что нет петли?
		len += 1;
		cur = cur->next_;
	}
	
	return len;
};
```
• Что не так с этим методом?
```cpp
list_t<int> l;
// тут как-то заполняем
l.top_->next_ = l.top_; // oops
size_t len = l.length();
```
• Можем ли мы проверить, что в списке нет петли?
#### Алгоритм Флойда
• Начинают два указателя: заяц и черепаха.
• Заяц за каждый ход продвигается вперёд на два элемента, а черепаха на один.
• Если они встретились, значит петля есть.
![[2.8.png]]
![[2.9.png]]
#### Case stude: список
• Ok, допустим алгоритм Флойда работает для определения длины.
• Но что, если мы хотим теперь написать метод reverse?
• Надо ли в начале reverse **опять** вызывать алгоритм Флойда, проверяя, нет ли петли и удваивая общее время работы?
#### Обсуждение
• У нас уже две очень похожие ситуации.
• Методы для полигона закладываются на то, что вершины отсортированы по кругу.
• Методы для списка закладываются на то, что он не содержит петли.
• Интуитивно "то, на что рассчитывают методы конкретного типа" - это нечто довольно важное.
#### Инварианты
• **Предусловиями** эффективного метода reverse является тот факт, что список является корректным двусвязным списком, начинается нулём, завершается нулём, не сломан нигде внутри.
• Проверять всё это каждый раз просто не хочется.
• Утверждение, которое должно быть верно всё время жизни объекта некоего типа называется **инвариантом** этого типа.
• Все методы списка существенно упростятся, если он сможет **сохранять** корректность, отсутствие петель и ноль-терминированность как свои инварианты.
• Что для этого нужно?
• Есть методы типа, которые пишем мы как разработчики типа. **Сохранять инварианты в методах** - обязанность разработчика, и он обычно с ней справляется.
• Но есть внешние функции, работающие с объектами этого типа. И вот они, как раз, являются источниками проблем.
• Есть ли у нас языковые средства, чтобы **запретить всем, кроме методов класса, работать с его состоянием**?
#### Инкапсуляция в язке C
• Мы можем использовать механизмы области видимости. Например, сделать тип непрозрачным (opaque):
```c
struct list_t;
struct list_t *list_create();
int list_length(struct list_t *list);
```
• Теперь пользователь не имеет доступа к состоянию list и может работать только с указателем на объект только методами этого типа.
• Что не так с **этим** подходом?
#### Инкапсуляция в языке C++
• В языке C++ для **инкапсуляции** (скрытия состояния объекта) используется специальный механизм, позволяющий сохранить видимость состояния.
```cpp
template <typename T> class list_t {
	struct node_t;
	node_t *top_, *back_;
	
public:
	int length() const;
};
```
• В структуре по умолчанию все поля public.
• Новое ключевое слово class определяет по умолчанию закрытые поля.
#### Обсуждение: инварианты и линейность
• У нас есть линейная модель памяти.
• Разве это не значит, что просто приведя указатель на объект к char* мы можем нарушить все инварианты?
#### Неконсистентное состояние
• Да, можем (по крайней мере для standard-layout и для trivially copyable). Идея в том, что мы **не хотим** этого делать.
• Объект, у которого нарушены инварианты - это объект в **неконсистентном состоянии**. Операции над ним опасны и непредсказуемы.
• Никакой программист, будучи в своём уме, не приведёт свой или чужой объект в неконсистентное состояние по доброй воле.
#### Обсуждение: ссылки
• Ссылки тоже сохраняют инварианты:
```cpp
int foo(const int *p) { int t = *p; delete p; return t; }

int bar(const int &p) { return p; }

foo(nullptr); // это невозможно проделать с bar

double d = 1.0;
int *q = *reinterpret_cast<int **>(&d);

foo(q); // это невозможно проделать с bar
```
• Инвариант const int reference: правильное **и не вам принадлежащее** целое число под ней. Именно поэтому побитовое представление ссылки **скрыто**.
#### Важное замечание
• Инкапсуляция - это свойство типа, а не его объектов:
```cpp
template <typename T> class list_t {
	node_t<T> *top_, *back_;
	
public:
	void concat_with(list_t<T> other) {
		for(auto cur = other.top_); // всё нормально, мы можем
			cur != other.back_;     // работать не только с this
			cur = cur->next_)       // а с любым объектом list_t<T>
			push(cur->data_);
	}
};
```
• Но, при этом, разные шаблонные параметры делают разные типы:
```cpp
template <typename T> class list_t {
	node_t<T> *top_, *back_;
	
public:
	template <typename U>
	void concat_with(list_t<U> other) {
		for(node_t<U> *cur = other.top_); // нарушение инкапсуляции
			cur != other.back_;           // тут тоже
			cur = cur->next_)
			push(cur->data_);
	}
};
```
#### Конструкторы и деструкторы
• Инкапсуляция делает критически важными конструкторы.
• Теперь состояние объектов просто нельзя установить извне:
```cpp
template <typename T> class list_t {
	node_t<T> *top_ = nullptr, *back_ = nullptr;
public:
	list_t(size_t initial_len); // ctor
	~list_t(); // dtor
}
```
• Но в случае со списком, его нельзя и очистить извне. Поэтому важными становятся также деструкторы. Синтаксис показан выше.
#### Обсуждение
• Увы, старые добрые malloc и free ничего не знают о конструкторах и деструкторах.
• Созданный с их помощью в динамической памяти объект не будет корректно инициализирован, и будет создан в **невалидном состоянии**.
• Что делать?
#### Аллокация динамической памяти
• В языке C++ аллокация делается через new и delete. Они вызывают конструкторы и деструкторы создаваемых объектов.
• Важно запомнить парность операторов:
```cpp
int *t = new Widget;    // выделяем скалярный объект
delete t;               // освобождаем скалярный объект

int *p = new Widget[5]; // выделяем массив
delete [] p;            // освобождаем массив
```
• Вы не должны пытаться освободить через delete выделенное через new[] и наоборот.
• Вы не должны смешивать new/delete с механизмом malloc/free.
#### Семантика new и delete
• Парность вызовов крайне важна:
```cpp
using mvi = MyVector<int>;
mvi *pv = new mvi();             // ctor
mvi *pvs = new MyVector<int>[5]; // 5 ctors
mvi *vpv = static_cast<mvi *> malloc(sizeof(mvi));
delete pv;                       // dtor
delete [] pvs;                   // 5 dtors
free(vpv); // no dtors
```
• По типу pv и pvs очень похожи. Как в точке удаления pvs понять, что нужно пять деструкторов?
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
// Example: wrong pairing of new/delete and consequences
// compile with: g++ -g newdelete.cc
// try x/20x P-1 in gdb
//
//---------------------------------------------------------------------------

#include <iostream>

struct MySmallClass {
	int t = 1;
	MySmallClass() { std::cout << "small ctor" << std::endl; }
	~MySmallClass() { std::cout << "small dtor" << std::endl; }
};

struct MyBigClass {
	int t = 1, p = 2, q = 3;
	MyBigClass() { std::cout << "big ctor" << std::endl; }
	~MyBigClass() { std::cout << "big dtor" << std::endl; }
};

int main() {
	MyBigClass *S = new MyBigClass;
	MySmallClass *T = new MySmallClass;
	MyBigClass *P = new MyBigClass[5];
	MySmallClass *Q = new MySmallClass[7];
	delete[] T; // terribly wrong
}
```
На выходе куча строчек "small dtor" и free(): invalid pointer;  Aborted (core dumped). :)
Причина в том, что delete[] смотри адрес перед началом T, чтобы узнать количество объектов. А потом делает delete количество раз, которое там в неинициализированной памяти было записано. Это UB.
#### Обсуждение
• Что вы думаете о ссылке на выделенную память?
```cpp
int *p = new int[5];
int &x = p[3];
```
 • Вроде всё хорошо, если бы не червячок сомнения. А что будет после delete?
## Область видимости и время жизни
• У любого имени есть область видимости (scope): совокупность всех **мест** в программе, откуда к нему можно обратиться.
```cpp
 int a = 2;
 
 void foo() {
	 int b = a + 3; // ok, we are in scope of a
	 
	 if(b > 5) {
		 int c = (a + b) / 2; // ok, we are in scope of a and b
	 }
	 
	 b += c; // compilation fail
 }
```
#### Время жизни
• У любой переменной есть время жизни (lifetime): совокупность всех **моментов времени** в программе, когда её состояние **валидно**.
• Первый такой момент случается после окончания инициализации.
```cpp
int main() {
	int a = a; // a declared, but lifetime of "a" not started (UB)
}
```
• Это довольно редкий пример, когда мы пытаемся использовать нечто до его рождения.
• Куда более часто мы будем пытаться использовать нечто после его смерти.
#### Провисшие указатели
• Указатель, ссылающийся на переменную с истёкшим временем жизни называется провисшим (dangling).
```cpp
int a = 2;

void foo() {
	int b = a + 3; int *pc;
	
	if(b > 5) {
		int c = (a + b) / 2; pc = &c;
	} // c scope end; c lifetime end; pc dangles
	
	b += *pc; // this is parrot no more
} // b scope end; b lifetime end;
```
![[2.10.png]]
#### Провисшие ссылки
• Сделать висячую ссылку чуть сложнее, чем указатель, но можно.
• Классика: ссылка внутрь удалённой памяти:
```cpp
int *p = new int[5];
int &x = p[3];
delete [] p; // x dangles
```
• Сама по себе провисшая ссылка ничего не значит. Проблемы будут только если по ней куда-то обратиться:
```cpp
x += 1; // it ceased to be
```
• Ещё классика: вернуть ссылку на временное значение:
```cpp
int& foo() {
	int x = 42;
	return x;
}

int x = foo(); // it expired and gone
```
• Компиляторы довольно плохи в диагностике провисших ссылок и указателей.
#### Продление жизни
• Константные (и только они) lvalue-ссылки продлевают жизнь временных объектов.
```cpp
const int &lx = 0;
int x = lx; // ok

int foo();

const int &ly = 42 + foo();
int y = ly; // ok
```
• Но не стоит соблазняться. Ссылка связывается со значением, а не со ссылкой, так что константная ссылка тоже может провиснуть при возврате из функции.
#### Жизнь временных объектов
• Временный объект живёт до конца полного выражения.
```cpp
struct S {
	int x;
	const int &y;
};

S x{1, 2}; // ok, lifetime extended
S *p = new S{1, 2}; // this is a late parrot
```
• На первой строчке у нас не временный, а постоянный объект.
• На второй будет висячая ссылка, потому что временный объект, продлявший жизнь константе закончился в конце выражения.
#### Иногда временный объект не создаётся
• Неконстантные левые ссылки не создают временных объектов и просто отказываются связываться с литералами.
```cpp
int foo(int &x);

foo(1); // ошибка компиляции
```
• И даже проще:
```cpp
int &x = 1; // ошибка компиляции
```
• И это одна из лучших новостей в этой части лекции.
• Попробуйте догадаться, отчего так сделано.
#### Decaying
```cpp
int foo(const int& t) {
	return t;
}
```
• Ссылка на объект в выражениях ведёт себя как сам объект.
• Мы это где-то встречали.
• Массив **деградирует** (decays) к указателю на свой первый элемент, когда он использован как rvalue.
```cpp
void foo(int *);

int arr[5];
int *t = arr + 3; // ok
foo(arr); // ok
arr = t; // fail
```
![[2.11.png]]
• Все ли помнят, чем отличается lvalue от rvalue?
#### Lvalue & rvalue
• В языке C концепция lvalue означала "left-hand-side value".
```c
y = x;
```
• Здесь y - это lvalue, а x - это rvalue.
• В языке C можно отделить синтаксически: вызов функции, имя массива, выражение сложения - всё это никогда не lvalue и технически не может встретиться в присваивании слева.
• Так ли это в C++?
• Увы, C++ усложняет вещи:
```cpp
int& foo();

foo() = x; // ok
```
• В языке C++ lvalue - это, скорее, "location value" - в смысле, что-то, у чего есть положение (location) в памяти.
• В языке C++11 также есть более точный термин glvalue, объединяющий положения с временными положениями. Мы поговорим о нём на лекции по rvalue-ссылкам.
• Ссылки, рассматриваемые здесь - это lvalue-ссылки.
• Технически может существовать lvalue-ссылка на массив. Это происходит именно потому, что, хотя массив и не может быть слева в присваивании, но он всегда lvalue в C++, потому что у него всегда есть локация (сам массив это локация по определению).
#### Ссылки и указатели на массивы
• Сначала: все ли помнят разницу между этими двумя строчками?
```cpp
int *x[20]; // ?
int (*y)[20]; // ?
```
• В языке возможны массивы указателей, но не массивы ссылок:
```cpp
int *x[20]; // массив указателей
int (*y)[20]; // указатель на массив
int (&z)[20] = *y; // ссылка на массив
```
• Все ли помнят, чем отличается указатель на массив?
Пример на godbolt:
```cpp
int main() {
	int ai[20] = {0};
	int *api[20] = {nullptr};
	int (*pai)[20] = &ai;
	int (&rai)[20] = ai;
	
	std::cout << api << " + 1 = " << api + 1 << std::endl;
	std::cout << pai << " + 1 = " << pai + 1 << std::endl;
	
	rai[2] = 40;
	(*pai)[2] += 2;
	
	std::cout << ai[2] << std::endl;
}
```
#### Ссылки и cdecl
• Прочитайте определения
```cpp
char *(*(&c)[10])(int *&p);
void (*bar(int x, void (*func)(int&))) (int&);
```
• Ссылки отлично ложатся в схему cdecl.
• Все ли помнят, как мы боролись с милыми особенностями cdecl в языке C?
#### Борьба против cdecl: typedef
• Рассмотрим запутанное объявление:
```cpp
void (*bar(int x, void (*func)(int&))) (int&);
```
• Используем typedef:
```cpp
typedef void (*ptr_to_fref) (int&);
ptr_to_fref bar(int x, ptr_to_fref func);
```
• Стало куда лучше, но, увы, typedef не позволяет создавать шаблонные алиасы.
#### Альтернатива typedef: using
• Используем using:
```cpp
using ptr_to_fref = void (*) (int&);
ptr_to_fref bar(int x, ptr_to_fref func);
```
• Техника using позволяет создавать не только синонимы типов. Можно даже создавать параметризованные синонимы:
```cpp
template <typename T>
using ptr_to_fref = void (*) (T&);
```
• В будущем параметризованные синонимы типов на неоднократно пригодятся.
#### Домашняя работа HW3D
• Со стандартного ввода приходит число 0 < N < 1000000, а потом N наборов точек, представляющих трёхмерные треугольники. Задача: вывести номера всех треугольников, которые пересекаются с каким-либо другим.
• Можно воспользовать [GCT], если вам не хватает базы в таких вопросах.
• Как вы будете тестировать ваш алгоритм?
#### Литература
• [CC11] ISO/IEC 14882 - "Information technology - Programming languages - C++", 2011
• [BS] Bjarne Stroustrup - The C++ Programming Language (4th Edition), 2013
• [GB] Grady Booch - Object-Oriented Analysis and Design with Applications, 2007
• [GCT] Eberly, Schneider - Geometric Tools for Computer Graphics, 2002
• [GS] Gilbert Strang - Introduction to Linear Algebra, Fifth Edition, 2016
• [BB] Ben Saks - Back to Basics: Pointers and Memory, CppCon, 2020
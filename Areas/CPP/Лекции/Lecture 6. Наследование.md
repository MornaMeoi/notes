<h1 align="center">НАСЛЕДОВАНИЕ</h1>

---
<p align="center">Динамический полиморфизм. Интерфейсы и реализация. Виртуальное наследование.</p>
## Наследование
#### Язык ParaCL
• Базовый синтаксис: арифметика, while, if, print, ?, объявления переменных.
```
fst = 0; // тип не требуется, все типы int
snd = 1;
iters = ?; // считать со stdin число, определить переменную

while(iters > 0) { // синтаксис для if такой же
	tmp = fst;
	fst = snd;
	snd = snd + tmp;
	iters = iters - 1;
}

print snd;
```
Пример с гита (fibs.pcl):
```
n = 0;
a = 0;
b = 1;
x = ?;

while(n < x) {
	n = n + 1;
	
	if(n == 1)
		print a;
	
	if(n == 2)
		print b;
	
	if(n > 2) {
		tmp = b;
		b = a + b;
		a = tmp;
		print b;
	}
}
```
#### Синтаксические деревья
• Грамматика представляется синтаксическим деревом.
```
while (iters > 0) {
	iters = iters - 1;
}
```
![[6.1.png]]
• И тут есть проблема: допустим, мы хотим представить узел такого дерева.
• Но все узлы очень разные.
#### Обсуждение
• Как нам сложить разные узлы внутрь одного дерева?
#### Первая попытка: супер-узел
```cpp
struct Node {
	Node *parent_;
	Node_t type_; // enum Node_t
	union Data {
		struct Decl { std::string declname_; } decl_;
		struct Binop { BinOp_t op_; } binop_; // enum BinOp_t
		// .... все остальные варианты ....
	} u;
	std::vector<Node *> childs_;
};
```
• Покритикуйте этот подход.
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
// Example of union with ctors and why you shall not do this
//
//---------------------------------------------------------------------------

#include <cassert>
#include <iostream>
#include <string>
#include <vector>

#ifdef WRONG
union Wrong {
	std::string s_;
	std::vector<int> v_;
};
#endif

union U {
	std::string s_;
	std::vector<int> v_;
	U(std::string s) { new (&s_) std::string{s}; }
	U(std::vector<int> v) { new (&v_) std::vector<int>{v}; }
	~U() {} // here =default will not work
}

struct S {
	bool str_;
	U u_;
	S(std::string s) : str_(true), u_(s) {}
	S(std::vector<int> v) : str_(false), u_(v) {}
	
	int &operator[](int n) {
		assert(!str_);
		return u_.v_[n];
	}
	operator const char *() {
		assert(str_);
		return u_.s_.c_str();
	}
	
	~S() {
		if(str_)
			u_.s_.~basic_string<char>();
		else
			u_.v_.~vector();
	}
};

int main() {
#ifdef WRONG
	Wrong w;
#endif
	std::string s = "Hello";
	std::vector<int> v = {0, 1, 2, 3, 4};
	S t1{s};
	S t2{v};
	std::cout << t1 < " " << t2[1] << std::endl;
}
```
Под -DWRONG будет ошибка компиляции. Основное:
```bash
error: use of deleted function 'Wrong::Wrong()'
```
Без -DWRONG прекрасно работает. Но выглядит так себе.
#### Вторая попытка: void pointers
• Что если мы заведём структуру, которая знает свой тип?
```cpp
struct Node {
	Node *parent_;
	Node_t type_;
	void *data_;
};
```
• Конкретные узлы хранят базовую часть.
```cpp
struct BinOp {
	Node basepart_;
	BinOp_t op_;
	Node *lhs_, *rhs_;
};
```
![[6.2.png]]
• Теперь можно написать функцию-конструктор для бинарной операции.
```cpp
Node* create_binop(Node *parent, BinOp_t opcode) {
	Node base = {parent, Node_t::BINOP, nullptr};
	BinOp *pbop = new BinOp {base, opcode, nullptr, nullptr};
	pbop->basepart_.data_ = static_cast<void *>(pbop);
	return &pbop->basepart_;
}
```
• Мне кажется, даже не надо просить это покритиковать.
• Этот код является худшей критикой самого себя.
Но оно будет работать. Пример на godbolt:
```cpp
enum Node_t {
	SCOPE, UNOP, BINOP
};

enum BinOp_t {
	ADD, SUB, MULT, DIV
};

struct Node {
	Node *parent_;
	Node_t type_;
	void *data_; // holds pointer to any type, for BINOP to BinOp_t, etc...
}

struct BinOp {
	Node basepart_;
	BinOp_t op_;
	Node *lhs_, *rhs_;
};

Node* create_binop(Node *parent, BinOp_t opcode) {
	Node base = {parent, Node_t::BINOP, nullptr};
	BinOp *pbop = new BinOp {base, opcode, nullptr, nullptr};
	pbop->basepart_.data_ = static_cast<void *>(pbop);
	return &pbop->basepart_;
}
```
#### Лучшее решение: поддержка в языке
• Кажется, для идеи "B является A" (также называется "отношение is-a") в языке нужна непосредственная поддержка.
• Это называется <span style="color: blue;">наследование</span>, и его <span style="color: blue;">открытая</span> форма записывается через двоеточкие и ключевое слово public.
```cpp
class A {};

class B : public A {}; // B is a A
```
• Это отношение открытого наследования позволяет нам переписать отношения более явно.
#### Открытое наследование
• Мы сэкономили сколько-то данных.
```cpp
struct Node {
	Node *parent_;
	Node_t type_;
};
```
• Но главное мы получили куда лучшую запись.
```cpp
struct BinOp : public Node {
	BinOp_t op_;
	Node *lhs_, *rhs_;
};
```
![[6.3.png]]
• Теперь фунция-конструктор станет и впрямь конструктором.
```cpp
struct Node {
	Node *parent_;
	Node_t type_;
};

struct BinOp : public Node {
	BinOp_t op_;
	Node *lhs_, *rhs_;
	
	BinOp(Node *parent, BinOp_t opcode) :
		Node{parent, Node_t::BINOP}, op_(opcode) {}
};
```
• Поскольку объект производного класса <span style="color: blue;">является</span> объектом базового класса, указатели и ссылки приводятся неявным приведением.
• Обратно можно привести через static_cast.
```cpp
struct Node;
struct BinOp : public Node;

void foo(const Node &pn);

BinOp *b = new BinOp(p, op);
foo(*b); // ok
Node *pn = b; // ok
b = static_cast<BinOp*>(pn); // ok
```
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
// Different nodes in the same vector
//
//---------------------------------------------------------------------------

#include <vector>

enum Node_t {
	SCOPE,
	UNOP,
	BINOP
};

enum UnOp_t { INC, DEC };

enum BinOp_t { ADD, SUB, MULT, DIV };

struct Node {
	Node *parent_;
	Node_t type_;
};

struct BinOp : public Node {
	BinOp_t op_;
	Node *lhs_, *rhs_;
	BinOp(Node *parent, BinOp_t opcode) :
		Node{parent, Node_t::BINOP}, op_(opcode) {}
};

struct UnOp : public Node {
	UnOp_t op_;
	Node *rhs_;
	UnOp(Node *parent, UnOp_t opcode) : Node{parent, Node_t::UNOP}, op_(opcode) {}
};

int main() {
	// type-erased container
	std::vector<Node *> nodes;
	nodes.push_back(new BinOp(nullptr, ADD));
	nodes.push_back(new UnOp(nullptr, INC));
	using ItTy = typename std::vector<Node *>::iterator;
	
	for(ItTy it = nodes.begin(), ite = nodes.end(); it != ite; ++it) {
		Node *elt = *it;
		if(elt->type_ == Node_t::BINOP) {
			BinOp *p = static_cast<BinOp *>(elt);
			delete p;
		} else if(elt->type_ == Node_t::UNOP) {
			UnOp *p = static_cast<UnOp *>(elt);
			delete p;
		}
	}
}
```
#### Обсуждение: квадрат и прямоугольник
• У открытого наследования есть два несвязанных смысла:
	• B **расширяет** A
	• B **является частным случаем** A
```cpp
struct Square {
	double x; // x*x square
	void double_square(Square &s) { x *= sqrt(2.0); }
};

struct Rectangle : public Square {
	double y; // x*y rectangle
};

Rectangle r{2, 3}; r.double_square(); // ??
```
В этом случае мы выбираем расширение. И нарушается "is-a".
```cpp
struct Rectangle {
	double x, y; // x*y rectangle
	void double_square() { s.x *= 2.0; }
};

sturct Square : public Rectangle {
	// в нашем квадрате кажется есть лишнее поле
};

Square s{2}; s.double_square(); // всё стало хуже
```
В этом случае мы выбираем "is-a". Тогда ломается расширение.
#### Принцип подстановки Лисков
• Типы Base и Derived связаны отношениями is-a (Derived является Base), если <span style="color: blue;">любой истинный предикат*</span> (\*любой интересующий нас истинный предикат) относительно Base остаётся истинным при подстановке Derived.
• Именно этот принципе даёт нам возможность завести в языке неявное приведение из Derived в Base.
• Для C++ этот принцип обычно выполняется с точностью до декорирования.
• При правильном проектировании, вы всегда можете подставить Derived* вместо Base* и Derived& вместо Base&.
• Подстановка значений в C++ сопряжена с некоторыми проблемами.
#### Проблема срезки: первое приближение
```cpp
struct A {
	int a_;
	A(int a) : a_(a) {}
};

struct B : public A {
	int b_;
	B(int b) : A(b / 2), b_(b) {}
};

B b1(10);
B b2(8);
A& a_ref = b2;
a_ref = b1; // b2 == ???
```
![[6.4.png]]
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
// Example of slicing problem
//
//---------------------------------------------------------------------------

#include <iostream>

struct A {
	int a_;
	A(int a) : a_(a) {}
};

struct B : public A {
	int b_;
	B(int b) : A(b / 2), b_(b) {}
};

std::ostream &operator<<(std::ostream &os, const B &b) {
	os << b.a_ << " " << b.b_;
	return os;
}

int main() {
	B b1(10); // a_ = 5, b_ = 10
	B b2(8);  // a_ = 4, b_ = 8
	A& a_ref = b2;
	a_ref = b1; // a_ = 5, b_ = 8, WAT??
	std::cout << b2 << std::endl;
}
```
Вывод:
```bash
5 8
```
#### Обсуждение
• Базовая срезка возникает из-за того, что присваивание не полиморфно.
```cpp
struct A {
	int a_;
	A(int a) : a_(a) {}
	A& operator=(const A& rhs) { a_ = rhs.a_; }
};

a_ref = b1; // a_ref.operator=(b1); b1 приводится к const A&
```
• Было бы здорово, если бы функция во время выполнения вела себя по разному, в зависимости от <span style="color: blue;">настоящего типа</span> своего первого аргумента.
• Мы поговорим о полиморфизме, но сначала домашнее задание.
#### HWP: ParaCL FE + симулятор
• Разработавайте фронтенд и симулятор языка ParaCL (далее - парасил) в объёме арифметика + if + while.
• Симулируемая программа должна считывать со стандартного ввода всё, что считывается и печатать на стандартный вывод всё, что нужно распечатать.
• Скрафтить хорошие тесты - важная часть задачи.
• Язык парасил чуть сложнее, чем приведено на этих слайдах. <span style="color: blue;">По мере продвижения вглудь реализации парасила будут открываться нюансы синтаксиса парасила</span>.
• Эта домашняя работа рекомендована для группового выполнения, поскольку среди высоких уровней будут и задачи на бэкенд и оптимизации.
## Полиморфизм
#### Общий интерфейс
• Мы можем спроектировать классы Triangle и Polygon так, чтобы они имели общий метод square(), вычисляющий их площадь.
• Можем ли мы сохранить массив из неважно каких объектов, лишь бы они имели этот метод?
• Ответ - да. Для этого мы должны сделать для них общий интерфейс от которого они оба наследуют.
```cpp
struct ISquare { void square(); }
struct Triangle : public ISquare; // реализует square()
struct Polygon : public ISquare; // реализует square()

std::vector<ISquare*> v; // хранит и Triangle* и Polygon*
``` 
• Проблемы возникают с тем, как здесь <span style="color: red;">реализовать</span> этот метод в ISquare.
#### Первая попытка: указатель на метод
```cpp
class ISquare {
	sometype *sqptr_;
public:
	ISquare(sometype *sqptr) : sqptr_(sqptr) {}
	double square() const { reutrn sqptr_->square(); }
};

template <typename T> struct Triangle : public ISquare {
	Point<T> x, y, z;
	Triangle() : ISquare(this) {}
	double square() const; // вычисление площади треугольника
};
```
• Покритикуйте этот способ. Начните с того, чем может являться sometype.
#### Языкова поддержка: virtual
```cpp
struct ISquare {
	virtual double square() const;
};

template <typename T> struct Triangle : public ISquare {
	Point<T> x, y, z;
	double square() const;
};
```
• Это всё ещё <span style="color: red;">очень плохой код (здесь три ошибки в семи строчках)</span>, мы скоро его улучшим.
• Но он иллюстрирует концепцию. Просто совпадение имени означает <span style="color: blue;">переопределение (overriding)</span> виртуальной функции.
#### Таблица виртуальных функций
• При создании класса с хотя бы одним виртуальным методом в него добавляется vptr.
• Конструктор базового класса динамически выделяет память для таблицы виртуальных функций.
• Конструктор каждого потомка производит инициализацию её своими методами. В итоге, там всегда оказываются нужные указатели.
![[6.5.png]]
#### Порядок конструирования
• При наследовании он имеет ключевое значение.
```cpp
template <typename T> struct Triangle : public ISquare {
	Point<T> x, y, z;
	double square() const;
	Triangle() : ISquare, x{}, y{}, z{} {}
};
```
• Сначала конструируется подобъект базового класса, который невидимо конструирует себе таблицу виртуальных функций.
• Потом конструктор подобъекта производного класса невидимо заполняет её адресами своих методов.
#### Статический и динамический тип
• Рассмотрим функцию
```cpp
double sum_square(const ISquare &lhs, const ISquare &rhs) {
	return lhs.square() + rhs.square();
}

Triangle t; Polygon p;
sum_square(t, p);
```
• <span style="color: blue;">Статическим типом</span> для lhs и rhs является известный на этапе компиляции тип const ISquare&.
• При этом, в конкретном вызове у них могут быть разные <span style="color: blue;">динамические типы</span>.
![[6.6.png]]
#### Языковая поддержка: virtual
```cpp
struct ISquare {
	virtual double square() const;
};

template <typename T> struct Triangle : public ISquare {
	Point<T> x, y, z'
	double square() const;
};
```
• Это всё ещё <span style="color: red;">очень плохой код</span>, мы скоро его улучшим.
• Но он иллюцистрирует концепцию. Просто совпадение имени означает <span style="color: blue;">переопределение (overriding)</span> виртуальной функции.
• Увы, имена могут быть ещё и <span style="color: red;">перегружены (overloaded)</span>.
#### Проблемы с overloading
• Здесь допущена обычная человеческая ошибка с типами int vs long.
```cpp
struct Matrix {
	virtual void pow(int x); // возведение в степень любой матрицы
};

struct SparseMatrix : Matrix {
	void pow(long x); // возведение в степень разреженной матрицы
	                  // крайне эффективный алгоритм
};

Matrix *m = new SparseMatrix;

m->pow(3); // увы, вызовется Matrix::pow
```
#### Обсуждение: overload vs override
• Переопределение функции (overriding) - это замещение в классе наследнике виртуальной функции на функцию наследника.
• Перегрузка функции (overloading) - это введение того же имени с другими типами аргументов.
```cpp
struct Matrix {
	virtual void pow(int x);
};

struct SparseMatrix : Matrix {
	void pow(int x) override; // никогда не overload
};
```
• Аннотация override сообщает, что мы имели в виду переопределение.
#### Языковая поддержка: override
```cpp
struct ISquare {
	virtual double square() const;
};

template <typename T> struct Triangle : public ISquare {
	Point<T> x, y, z;
	double square() override const;
};
```
• Это всё ещё <span style="color: red;">очень плохой код</span>, мы скоро его улучшим.
• Следующая проблема - это как нам написать тело самой общей функции? Тела наследников понятны. Но что должно быть в самой ISquare::square? Может быть, аборт?
#### Языковая поддержка: pure virtual
```cpp
struct ISquare {
	virtual double square() const = 0;
};

template <typename T> struct Triangle : public ISquare {
	Point<T> x, y, z;
	double square() override const;
};
```
• Это всё ещё <span style="color: red;">очень плохой код</span>, мы скоро его улучшим.
• Проблема решается чисто виртуальными методами, которые не требуют определния и только делегируют наследникам.
• Объект класса с чисто виртуальными методами не может быть создан.
#### Внезапная утечка памяти
```cpp
struct ISquare {
	virtual double square() const = 0;
};

template <typename T> struct Triangle : public ISquare {
	Point<T> x, y, z;
	double square() override const;
};
```
• Это всё ещё <span style="color: red;">очень плохой код</span>, мы скоро его улучшим.
• Следующая проблема: удаление по указателю на базовый класс.
```cpp
ISquare *sq = new Triangle<int>; delete sq; // утечка
```
#### Обсуждение
• Мы хотим, чтобы удаление по указателю на базовый класс вызывало правильный деструктор производного класса.
• Это означает, что нам нужен <span style="color: red;">виртуальный деструктор</span>.
```cpp
struct ISquare {
	virtual double square() const = 0;
	virtual ISquare() {}
};

template <typename T> struct Triangle : public ISquare {};

ISquare *sq = new Triangle<int>;
delete sq; // Ok, вызван Triangle::~Triangle()
```
#### Интерфейсные классы
• Класс, в котором все методы чисто виртуальные, служит своего рода общим интерфейсом.
```cpp
struct ISquare {
	virtual double square() const = 0;
	virtual ~ISquare() {}
};
```
• Такой класс называется абстрактным базовым классом.
• К сожалению, виртуальный конструктор (в том числе, копирующий) невозможен.
• Тогда непонятно, как нам скопировать по базовому классу.
#### Виртуальное копирование
• Обычно используется виртуальный метод clone.
```cpp
struct ISquare {
	// всё остальное
	virtual ISquare *clone() const = 0;
};

template <typename T> struct Triangle : public ISquare {
	std::array<Point<T>, 3> pts_;
	Triangle *clone() const override { return new Triangle{pts_}; }
};
```
• Обратите внимание: override здесь законный, поскольку Triangle\<T\>* открыто наследует и, значит, является ISquare*.
#### Срезка возвращается
• Из-за невозможности виртуальных конструкторов, срезка возможна при передаче по значению.
```cpp
void foo(A a) { std::cout << a << std::endl; }

B b(10); foo(b1); // на экране "5"
```
• <span style="color: red;">Поэтому никогда не передавайте объекты базовых классов по значению</span>.
• Используйте указатель или ссылку.
```cpp
void foo(A& a) { std::cout << a << std::endl; }

B b(10); foo(b1); // на экране "5 10"
```
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
// Example of slicing with arguments
//
//---------------------------------------------------------------------------

#include <iostream>

struct A {
	int a_;
	A(int a) : a_(a) {}
	virtual void dump(std::ostream &os) const { os << a_; }
	virtual ~A() {}
};

struct B : public A {
	int b_;
	B(int b) : A(b / 2), b_(b) {}
	void dump(std::ostream &os) const override { os << a_ << " " << b_; }
};

std::ostream &operator<<(std::ostream &os, const A &a) {
	a.dump(os);
	return os;
}

void foo(A a) { std::cout << a << std::endl; }

void bar(A &a) { std::cout << a << std::endl; }

int main() {
	B b1(10);
	foo(b1);
	bar(b1);
}
```
Вывод:
```bash
5
5 10
```
#### Необходимость: virtual dtor
```cpp
struct ISquare {
	virtual double square() const = 0;
	virtual ~ISquare() {}
};

template <typename T> struct Triangle : public ISquare {
	Point<T> x, y, z;
	double square() override const;
};
```
• Вот это уже неплохо.
• Но хотя этот код стал неплохим, концептуально у нас проблемы.
#### Обсуждение: как теперь жить?
• Допустим, мы написали некий класс Foo.
• Писать ли у него виртуальный деструктор?
#### Языкова поддержка: final
• Если мы хотим, чтобы от него наследовались, то да, писать.
• Если не хотим и не хотим оверхеда на vtable, то можно объявить его final.
```cpp
struct Foo final {
	// content
};
```
• Теперь наследование будет ошибкой компиляции.
#### Пишем правильно: четыре способа
• Класс в C++ написан правильно, если и только если любое из условий выполнено:
1. Класс содержит виртуальный деструктор.
2. Класс объявлен как final.
3. Класс является stateless и подвержен EBCO.
4. Класс не может быть уничтожен извне, но может быть уничтожен потомком.
• Первые два варианта мы уже обсудили.
• Давайте поговорим о третьем и четвёртом.
#### Empty Base Class Optimizations
• Оптимизации пустого базового класса (EBCO) применяются, когда базовый класс хм... пустой.
```cpp
class A{};

class B : public A {};

A a; assert(sizeof(a) == 1);

B b; assert(sizeof(b) == 1);
```
• Заметьте, класс с хотя бы одним виртуальным методом точно не пустой.
• Пока неясно, зачем нам вообще такие ребята. Они сыграют позже, так как нужны для так называемых <span style="color: blue;">миксинов</span>.
#### EBCO и unique_pointer
• Мы говорили, что unique_ptr выглядит как-то так:
```cpp
template <typename T, typename Deleter = default_delete<T>>
class unique_ptr {
	T *ptr_; Deleter del_;
public:
	unique_ptr(T *ptr = nullptr, Deleter del = Deleter()) : ptr_(ptr), del_(del) {}
	~unique_ptr() { del_(ptr_); }
	// и так далее
};
```
• Но можем ли мы сэкономить, если Deleter - это stateless class?
• Если делетер в unique_pointer - это класс, то:
```cpp
template <typename T, typename Deleter = default_delete<T>>
class unique_ptr : public Deleter {
	T *ptr_;
public:
	unique_ptr(T *ptr = nullptr, Deleter del = Deleter()) :
		Deleter(del), ptr_(ptr), del_(del) {}
	~unique_ptr() { Deleter::operator()(ptr_); }
	// и так далее
};
```
• Увы, это невозможно, если делетер функция (см. пример).
• Оставим в качестве тизера **как же** unique_pointer отличает класс от функции.
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
// Example of unique_ptr sizes
//
//---------------------------------------------------------------------------

#include <iostream>
#include <memory>

enum { SZ = 1000 };

struct CDeleterTy {
	void operator()(int *t) { delete[] t; }
};

auto LDeleter = [](int *t) { delete[] t; };
using LDeleterTy = decltype(LDeleter);

void FDeleter(int *t) { delete[] t; }
using FDeleterTy = decltype(&FDeleter);

int main() {
	int *Uip = new int[SZ]();
	std::unique_ptr<int, CDeleterTy> Uic{new int[SZ]()};
	std::unique_ptr<int, LDeleterTy> Uil{new int[SZ](), LDeleter};
	std::unique_ptr<int, FDeleterTy> Uif{new int[SZ](), FDeleter};
	
	std::cout << "pi:" << sizeof(Uip) << "\tuic:" << sizeof(Uic);
	std::cout << "\tuil:" << sizeof(Uil) << "\tuif:" << sizeof(Uif) << std::endl;
	
	delete[] Uip;
}
```
Вывод:
```bash
pi:8	uic:8  uil:8	uif:16
```
#### Обсуждение
• Разумеется, при использовании таких миксинов никто не будет стирать класс по указателю на его делетер.
```cpp
struct CDeleterTy {
	void operator()(int *t) { delete[] t; }
};

CDeleterTy *pDel = new std::unique_ptr<int, CDeleterTy> { new int[SZ]() };
delete pDel; // к счастью, это не скомпилируется
```
• Писать виртуальный деструктор в миксин не хочется. Потому что он резко станет statefull.
#### Языкова поддержка: protected
• Модификатор protected служит для защиты от всех, кроме наследников.
• Он позволяект писать чисто-базовые классы.
```cpp
class PureBase {
	// что угодно
protected:
	~PureBase() {}
};
```
• Теперь объект класса-наследника просто нельзя удалить по указателю на базовый класс и проблема снимается.
• Если не удалять изнутри класса и тогда всё по прежнему.
#### Пишем правильно: <span style="color: blue;">два</span> способа
• Класс в C++ написано правильно, если и только если любое из условий выполнено:
1. <span style="color: blue;">Класс содержит виртуальный деструктор</span>.
2. <span style="color: blue;">Класс объявлен как final</span>.
3. Класс является stateless и подвержен EBCO.
4. Класс не может быть уничтожен извне, но может быть уничтожен потомком.
• Первые два варианта мы уже обсудили.
• Третий и четвёртый, скорее, культурно приемлимы, чем надёжны.
• Кроме того, ключевое слово final помогает <span style="color: blue;">девиртуализации</span>.
#### Обсуждение
• Как вы вообще считаете: как виртуальные функции влияют на производительность? А на стабильность?
• Сугубо мрачно. Виртуальная функция вызывается как минимум по указателю (в случае множественного наследования всё еще хуже).
• Мало того, этот указатель должен быть правильно заполнен в конструкторе.
• На практике это значит целый новый класс ошибок.
#### Обсуждение: PVC
• Распространённой ошибкой является вызов чисто виртуального метода.
```cpp
struct Base {
	Base() { doIt(); } // PVC invocation
	virtual void doIt() = 0;
};

strict Derived : public Base { void doIt() override; };

int main() {
	Derived d; // PVC appears
}
```
• Заметьте, вызов чисто виртуальной функции - это ошбика не только в ctor/dtor, но и в любой функции, которая из них вызывается.
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
// Example of PVC invocation
// Compile: g++ pvc.cc -DPVCDIAG
// Compile: g++ pvc.cc -DPVC
// Then run and you will have PVC error
//
//---------------------------------------------------------------------------

#include <iostream>

struct Base {
#if defined(PVC)
	Base() { unsafe(); }
#elif defined(PVCDIAG)
	Base() { doIt(); }
#else
	Base() { safe(); }
#endif
	void basesafe() { safe(); }
	virtual void safe() { std::cout << "Base\n"; } // ok invocation
	void unsafe() { doIt(); }
	virtual void doIt() = 0;
	virtual ~Base() = 0;
};

Base::~Base() {}

struct Derived : public Base {
	void safe() override { std::cout << "Derived" << std::endl; }
	void doIt() override { std::cout << "Derived::doit" << std::endl; }
};

int main() {
	Derived d; // PVC appears
	d.basesafe();
}
```
С -DPVCDIAG ошибка:
```bash
warning: pure virtual 'virtual void Base::doIt()' called from constructor
...: undefined reference to 'Base::doIt()'
```
С -DPVC скомпилировалось. При запуске бинаря runtime ошибка:
```bash
pure virtual method called
terminate called without an active exception
Aborted (core dumped)
```
#### Виртуальные функции в конструкторах
• Даже если они не приводят к PVC, они работают как невиртуальные.
```cpp
struct Base {
	Base() { doIt(); }
	virtual void doIt();
};

struct Derived : public Base {
	void doIt() override;
};

Derived d; // Base::doIt
```
• Поэтому многие вообще скептически относятся к вызовам функций в ctor/dtor.
#### Статическое и динамическое связывание
• Говорят, что виртуальные функции <span style="color: blue;">связываются динамически</span> (так называется процесс разрешения адреса функции через vtbl во время выполнения).
• Обычные функции <span style="color: blue;">связываются статически</span>.
• Даже если физически они приходят из динамических библиотек или являются позиционно независимыми и адресуются через PLT, это неважно.
• <span style="color: blue;">На уровне модели языка</span> они считаются связывающимися статически.
• Увы, но многие другие вещи имеют статическое связывание. Например, аргументы по умолчанию.
#### Аргументы по умолчанию
• Как уже было сказано, они связываются статически. То есть, <span style="color: blue;">зависят только от статического типа</span>.
```cpp
struct Base {
	virtual int foo(int a = 14) { return a; }
};

struct Derived : public Base {
	int foo(int a = 42) override { return a; }
};

Base *pb = new Derived{};
std::cout << pb->foo() << std::endl; // на экране 14
```
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
// Example of non-virtual default args
//
//---------------------------------------------------------------------------

#include <iostream>

struct Base {
	virtual int foo(int a = 14) { return a; }
};

struct Derived : public Base {
	int foo(int a = 42) override { return 2 * a; }
};

int main() {
	Base *pb = new Derived{};
	std::cout << pb->foo() << std::endl;
	delete pb;
}
```
Вывод:
```bash
28
```
#### Выход из положения: NVI (Non-Virtual Interface)
• Если хочется интерфейс с аргументами по умолчанию, его можно сделать невиртуальным, чтобы никто не смог их переопределить.
```cpp
struct BaseNVI {
	int foo(int x = 14) { return foo_impl(x); }

private:
	virtual int foo_impl(int a) { return a; }
};

struct Derived : public Base {
	int foo_impl(int a) override { return a; }
};
```
• Закрытая виртуальная функция <span style="color: blue;">открыто переопределена</span>. Это нормально.
#### Два полиморфизма
• Полиморфной (по данному аргументу) называется функция, которая ведёт себя по разному в зависимости от <span style="color: blue;">типа</span> этого аргумента.
• <span style="color: blue;">Полиморфизм</span> бывает <span style="color: blue;">статический</span>, когда функция управляется известными на этапе компиляции типами и <span style="color: blue;">динамический</span>, когда тип известен только на этапе выполнения.
• Примеры:
	• Множество перегрузки можно рассматривать как одну статически полиморфную функцию (по любому аргументу).
	• Шаблон функции - это статически полиморфная функция (по любому аргументу).
	• Виртуальная функция - это динамически полиморфная функция (по первому неявному аргументу this).
#### Обсуждение: ограничения
• Давайте посмотрим, насколько можно смешивать статический и динамический полиморфизм.
• Два простых вопроса:
1. Как вы думаете, может ли существовать шаблон виртуального метода?
	- <span style="color: blue;">К счастью, не может (какие последствия это вызвало бы для таблиц виртуальных функций?)</span>
2. Как вы думаете, можно ли перегружать виртуальные функции?
	- <span style="color: red;">К сожалению, можно. И это вызывает крайне мрачные последствия из-за скрытия имён.</span>
#### Перегрузка виртуальных функций
• Предположим, что мы умеем эффективно возводить разреженные матрицы в целые степени и хотим просто переиспользовать возведение в дробные.
```cpp
struct Matrix {
	virtual void pow(double x); // обычный алгоритм
	virtual void pow(int x); // эффективный алгоритм
};

struct SparseMatrix : public Matrix {
	void pow(int x) override; // крайне эффективный алгоритм
};

SparseMatrix d;

d.pow(1.5); // какой метод будет вызван?
```
#### Сокрытие имён
• Увы, в коде ниже будет вызван SparseMatrix::pow.
```cpp
struct Matrix {
	virtual void pow(double x); // Matrix::pow(int)
	virtual void pow(int x);    // Matrix::pow(double)
};

struct SparseMatrix : public Matrix {
	void pow(int x) override; // имя pow скрывает Matrix::pow
};

SparseMatrix d;

d.pow(1.5); // SparseMatrix::pow(1);
```
#### Введение имён в область видимости
• Для введения имён в область видимости, используем using.
```cpp
struct Matrix {
	virtual void pow(double x); // Matrix::pow(int)
	virtual void pow(int x);    // Matrix::pow(double)
};

struct SparseMatrix : public Matrix {
	using Matrix::pow;
	void pow(int x) override; // имя pow скрывает Matrix::pow
};

SparseMatrix d;

d.pow(1.5); // Matrix::pow(1.5);
```
#### Обсуждение: контроль доступа
• К этому времени мы знаем три модификатора доступа.
	• public - доступно всем
	• protected - доступно только потомкам
	• private - доступно только самому себе
• Но мы также знаем, что public означает открытое наследование и воодит отношение is-a.
```cpp
class Derived : public Base {} // Derived is a Base
```
• Можем ли мы представить себе иные отношения общее-частное?
#### Разновидности наследования
• При любом наследовании private поля недоступны классам наследникам.
• Остальные поля изменяют в наследниках уровень доступа в соответствии с типом наследования.

|                        | public inheritance | protected inheritance | private inheritance |
| ---------------------- | ------------------ | --------------------- | ------------------- |
| **public becomes:**    | public             | protected             | private             |
| **protected becomes:** | protected          | protected             | private             |
• Приватное наследование эквивалентно композиции в закрытой части.
• Говорят, что оно моделирует отношение part-of.
• Неявного приведения типа, при этом, не происходит.
#### Наследование по умолчанию
• Второе отличие class от struct: у class по умолчанию private, у struct public.
```cpp
struct S : /*public*/ D {
//public:
	int n;
};

class S : /*private*/ D {
//private:
	int n;
};
```
• Разумеется, крайне хороший тон - это писать явные модификаторы, <span style="color: blue;">если их больше одного</span>.
#### Отношение part-of
• Закрытое наследование
```cpp
class Whole : private Part {
	// everythin else
};
```
• Композиция
```cpp
class Whole {
	// everything else
private:
	Part p_;
};
```
• Ключевое отличие наследования - это:
	• возможность переопределять виртуальные функции из базового класса
	• доступ к защищённым (protected) полям базового класса
	• возможность использовать using и вводить имена из базового класса в свой scope
• Композиция должна быть выбором по умолчанию.
#### EBCO и unique_pointer: private inh
• Логично, что мы хотим private. На него EBCO тоже работает.
```cpp
template <typename T, typename Deleter = default_deleter<T>>
class unique_ptr : private Deleter {
	T *ptr_;
public:
	unique_ptr(T *ptr = nullptr, Deleter del = Deleter()) :
		Deleter(del), ptr_(ptr), del_(del) {}
	~unique_ptr() { Deleter::operator()(ptr_); }
}
```
• Теперь нет опасности приведения к базовому классу:
```cpp
DeleterTy *pD = new unique_ptr<int, DeleterTy>{}; // FAIL
```
#### Case study: MyArray
• Допустим, у вас есть интерфейс <span style="color: blue;">IBuffer</span>, использованный в <span style="color: blue;">Array</span>.
```cpp
class Array {
protected:
	IBuffer *buf_;
public:
	explicit Array(IBuffer *buf) : buf_(buf) {}
	// something interesting
};
```
• Вы реализовали ваш собстенный превосходный класс MyBuffer, наследующий от IBuffer.
• Как написать класс MyArray, наследующий от Array и использующий MyBuffer?
#### Первая попытка: двойное включение
• Мы можем просто сохранить MyBuffer внутри.
```cpp
class MyArray : public Array {
protected:
	MyBuffer mbuf_;
public:
	explicit MyArray(int size) : mbuf_(size), Array(&mbuf_) {}
	// something MORE interesting
};
```
• Это не будет работать, так как буфер нельзя инициализировать раньше базового класса.
• Но и переставить инициализаторы местами мы не можем.
#### Обсуждение
```cpp
class MyArray : public Array {
protected:
	MyBuffer mbuf_;
};
```
• Чтобы здесь в порядке инициализации MyBuffer шёл раньше, чем Array, он должен быть включён в список базовых класов.
• Но это означает, что нам нужно унаследовать сразу от двух классов.
• Разве это возможно?
## Множественное наследование
#### Множественное наследование
```cpp
class MyArray : protected MyBuffer, public Array {
public:
	explicit MyArray(int size) : MyBuffer(size), Array(???) {}
	// something MORE interesting
};
```
• Синтаксис наследования: все базовые классы с модификаторами через запятую.
• Здесь наследование защищённое, потому что:
	• мы не хотим прятать защищённую часть MyBuffer и не можем сделать его приватным
	• мы не хотим показывать MyBuffer наружу и не можем сделать его публичным
• Но есть небольшая проблема: что написать вместо знаковы вопроса?
#### Решение: прокси-класс
```cpp
struct ProxyBuf {
	MyBuffer buf;
	explicit ProxyBuf(int size) : buf(size) {}
};

class MyArray : protected ProxyBuf, public Array {
public:
	explicit MyArray(int size) : ProxyBuf(size), Array(&ProxyBuf::buf) {}
	// something MORE interesting
};
```
• Теперь всё срастается.
#### Обсуждение: сама идея сомнительна
• Множественное наследование интерфейса не вызывает вопросов.
```cpp
class Man : public ITwoLegs, public INoFeather {
public:
	// методы для двуногих
public:
	// методы для лишённых перьев
// всё остальное
};
```
• Но в довольно большом количестве языков запрещено множественное наследование реализации. И сделано это неспроста.
#### Ромбовидные схемы
```cpp
struct File { int a; };

struct InputFile : public File { int b; };

struct OutputFile : public File { int c; };

struct IOFile : public InputFile, public OutputFile { int d; };
```
![[6.7.png]]
#### Проблемы ромбовидных схем
• Поскольку в объект нижнего класса входят два верхних подобъекта, доступ к переменным неочевиден.
```cpp
IOFile f{11};
int x = f.a; // ошибка
int y = f.InputFile::a; // ok, но это боль
```
• Кроме того, в принципе f.InputFile::a и f.OutpuFile::a могут и разойтись в процессе работы.
• В качестве решения хотелось бы иметь один экземпляр базового класса, сколькими бы путями он не пришёл в производный.
• Такие базовые классы называются <span style="color: blue;">виртуальными</span>.
#### Виртуальные базовые классы
• Виртуальное наследование - это поддержка в языке
```cpp
struct File { .... };
struct InputFile : virtual public File { .... };
struct OutputFile : virtual public File { .... };
struct IOFile : public InputFile, public OutputFile { .... };

IOFile f{11};
int x = f.a; // ok
int y = f.InputFile::a; // ok, тоже работает
```
• Конечно, тут сразу возникает масса вопросов....
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
// Romb scheme -- virtual base classes
//
//---------------------------------------------------------------------------

#include <iostream>

struct File {
	int a;
	File(int a) : a{a} { std::cout << "File ctor" << std::endl; }
	virtual ~File() {}
};

struct InputFile : virtual public File {
	int b;
	InputFile(int b) : File(b * 2), b{b} {
		std::cout << "IFile ctor" << std::endl;
	}
};

struct OutputFile : virtual public File {
	int c;
	OutputFile(int c) : File(c * 3), c{c} {
		std::cout << "OFile ctor" << std::endl;
	}
};

struct IOFile : public InputFile, public OutputFile {
	int d;
	IOFile(int d) : File(d), InputFile(d * 5), OutputFile(d * 7), d{d} {
		std::cout << "IOFile ctor" << std::endl;
	}
};

#if 0
struct IOFile2 : public IOFile {
	int e;
	IOFile2(int e) : IOFile(e), e(e) {}
};
#endif

int main() {
	IOFile f(11);
	std::cout << f.InputFile::a << std::endl;
	std::cout << f.OutputFile::a << std::endl;
	std::cout << f.a << std::endl;
	
#if 1
	InputFile g(11);
	OutputFile h(11);
	std::cout << g.a << std::endl;
	std::cout << h.a << std::endl;
#endif

#if 0
	IOFile2 io2(11);
#endif
}
```
Без -D0 вывод:
```bash
File ctor
IFile ctor
OFile ctor
IOFile ctor
11
11
11
```
Если закомментировать File(d) в IOFile, то будет ошибка компиляции.
```bash
error: no matching function for call to 'File::File()'
```
Если с -D0, то снова ошибка компиляции, потому что в IOFile2 также нет вызова конструктора виртуального базового класса.
Если с -D1, то вывод:
```bash
File ctor
IFile ctor
File ctor
OFile ctor
22
33
```
#### Виртуальные базовые классы
• Что если базовый класс виртуальный не по всем путям?
-  <span style="color: blue;">никаких проблем, вниз попадёт только один со всех виртуальных путей и по одному с каждого невиртуального</span>
• Что если базовый класс виртуальный, но в нижнем подобъекте всего один?
- <span style="color: blue;">никаких проблем, можно хоть все базовые классы всегда делать виртуальными, будет работать как обычно наследование*</span> (<span style="color: red;">*только хуже</span>)
• В каком порядке и когда конструируются обычные и виртуальные подобъекты?
- <span style="color: blue;">такое чувство, что сначала должны конструировать все виртуальные, а потом все остальные</span>
#### Виртуальные базовые классы
• Вызов виртуальной функции при множественном наследовании должен пройти через дополнительный уровень диспетчеризации.
• А при виртуальном наследовании через <span style="color: red;">ещё один дополнительный уровень</span> из-за того, что таблицы для виртуальных подобъектов должны быть отдельно смёржены.
![[6.8.png]]
#### Списки инициализации
• Виртуальный базовый класс <span style="color: blue;">обязан появиться</span> в списке инициализации самого нижнего подобъекта.
```cpp
struct InputFile : virtual public File {
	InputFile() : File(smths1) {} // этот File() не вызовется до IOFile
};

struct OutputFile : virtual public File {
	OutputFile() : File(smths2) {} // этот File() не вызовется до IOFile
};

struct IOFile : public InputFile, public OutputFile {
	IOFile() : File(smths3), InputFile(), OutputFile() {}
};

IOFile f; // вызовет File(smths3)
```
#### Case study: сложная диаграмма
Порядок иницализации? (Литерой V помечены виртуальные классы.)
![[6.9.png]]
Ответ:
• V1: B1, V1
• V2: B4, B2, V2
• D1: D1
• D2: B3, D2
• X: M1, M2, X
Итого:
B1, V1, B4, B2, V2, D1, B3, D2, M1, M2, X
#### Обсуждение
• Множественное наследование уже кажется мрачным?
• Это мы ещё не дошли до по-настоящему мрачных вещей.
• Дело в том, что проблемы возможны не только с ромбовидными схемами.
#### Проблема преобразований
• Для того, чтобы при одиночном наследовании преобразовать вверх или вниз по указателю или ссылке достаточно static cast.
![[6.10.png]]
```cpp
struct Base {};

struct Derived : public Base {};

Derived *pd = new Derived{};

Base *pb = static_cast<Base*>(pd); // ok

pd = static_cast<Derived*>(pb); // ok
```
• Сработает ли такой подход при множественном наследовании?
• Как ни странно, всё магическим образом прекрасно работает при касте вверх.
![[6.11.png]]
```cpp
struct B1 {};
struct B2 {};
struct D : B1, B2 {};

D *pd = new D{};

B1 *pb1 = static_cast<B1*>(pd); // ok

B2 *pb2 = static_cast<B2*>(pd); // ok
```
• Мало того, всё магическим образом работает и вниз (см. пример):
```cpp
#include <iostream>

struct Filler {
	int x, y;
	virtual void filler() = 0;
	virtual ~Filler() {}
};

struct InputFile {
	int b;
	InputFile(int b) : b{b} {}
	virtual ~InputFile() {}
};

struct OutputFile {
	int c;
	OutputFile(int c) : c{c} {}
	virtual ~OutputFile() {}
};

struct IOFile : public Filler, public InputFile, public OutputFile {
	int d;
	void filler() override { std::cout << " " << std::endl; }
	IOFile(int d) : InputFile(d * 2), OutputFile(d * 3), d{d} {}
};

int main() {
	IOFile *piof = new IOFile{5};
	std::cout << std::hex << "piof = " << piof << ": " << std::dec;
	std::cout << piof->d << std::endl;
	
	OutputFile *pof = static_cast<OutputFile *>(piof);
	std::cout << std::hex << "pof = " << pof << ": " << std::dec;
	std::cout << pof->c << std::endl;
	
	InputFile *pif = static_cast<InputFile *>(piof);
	std::cout << std::hex << "pif = " << pif << ": " << std::dec;
	std::cout << pif->b << std::endl;
	
	piof = static_cast<IOFile *>(pif);
	std::cout << std::hex << "piof = " << piof << ": " << std::dec;
	std::cout << piof->d << std::endl;
	
		piof = static_cast<IOFile *>(pof);
	std::cout << std::hex << "piof = " << piof << ": " << std::dec;
	std::cout << piof->d << std::endl;
}
```
Вывод:
```bash
piof = 0x7fffd5fbceb0: 5
pof = 0x7fffd5fbced0: 15
pif = 0x7fffd5fbcec0: 10
piof = 0x7fffd5fbceb0: 5
piof = 0x7fffd5fbceb0: 5
```
#### Обсуждение
• Такое чувство, что при виртуальном наследовании из-за смёрженных таблиц не должен работать каст вниз?
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
// Romb scheme -- sibling casts
//
//---------------------------------------------------------------------------

#include <iostream>

struct File {
	int a;
	File(int a) : a{a} { std::cout << "File ctor" << std::endl; }
	virtual ~File() {}
};

struct InputFile : virtual public File {
	int b;
	InputFile(int b) : File(b * 2), b{b} {
		std::cout << "IFile ctor" << std::endl;
	}
};

struct OutputFile : virtual public File {
	int c;
	OutputFile(int b) : File(c * 3), c{c} {
		std::cout << "OFile ctor" << std::endl;
	}
};

struct IOFile : public InputFile, public OutputFile {
	int d;
	IOFile(int d) : File(d), InputFile(d * 5), OutputFile(d * 7) {
		std::cout << "IOFile ctor" << std::endl;
	}
};

int main() {
	IOFile *pe = new IOFile{11};
	std::cout << pe->a << std::endl;
	
	InputFile *pg = static_cast<InputFile *>(pe);
	std::cout << pg->a << std::endl;
	
	File *pf = static_cast<File *>(pg);
	std::cout << pf->a << std::endl;
	
	pe = static_cast<IOFile *>(pf);
	std::cout << pe->a << std::endl;
}
```
Ошибка компиляции:
```bash
error: cannot convert from pointer to base class 'File' to pointer to derived class 'IOFile' because the base is virtual
```
## RTTI
#### Runtime Type Information
• Для разрешения насущных вопросов (например, "какой у меня динамический тип") и свободного хождения вниз-вверх по иерархиям классов, программа на C++ должна вов время исполнения поддерживать особые невидимые программисту структуры данных.
• Это очень странное решение для C++, потому что оно противоречит идеологии языка.
• В языке ровно два таких сомнительных механизма: RTTI и исключения.
• Много раз делались попытки завести к ним какой-нибудь третий, но других ошибок с 1998 года комитет ни разу не делал.
• И, конечно, основа RTTI - это typeinfo.
#### Возможности typeid
• Оператор typeid возвращает объект std::typeinfo, который можно сравнивать и можно выводить на экран.
• Этот объект представляет собой динамический или статический тип.
```cpp
OutputFile *pof = new IOFile{5}'
assert(typeid(*pof) == typeid(IOFile)); // динамический тип
```
• typeid может брать type или expression. Если он берё expression, то динамический тип выводится только если это lvalue expression объекта с хотя бы одной виртуальной функцией.
```cpp
assert(typeid(pof) != typeid(IOFile*)); // статический тип
```
#### Возможности dynamic_cast
• Самым распространённым (и самым накладным) механизмом RTTI является dynamic_cast. Он может приводить типы внутри иерархий.
```cpp
IOFile *piof = new IOFile{}; // File - это виртуальная база

File *pf = static_cast<File *>(piof); // ok

InputFile *pif = dynamic_cast<InputFile *>(pf); // ok

OutputFile *pof = dynamic_cast<OutputFile *>(pf); // ok

pif = dynamic_cast<InputFile *>(pof); // ok!
```
• Обратите внимание, возможно приведение к сестринскому типу.
#### Ограничения
• dynamic_cast ходит по всем путям. В том числе, по виртуальным. Время его работы может превышать время работы static_cast <span style="color: red;">на порядки</span>.
• К тому же, затраты на dynamic_cast могут изменять при изменении иерархий наследования.
• При отсутствии таблиц виртуальных функций, dynamic_cast ведёт себя как static_cast, и это наиболее безумное его использование.
• dynamic_cast работает только для указателей и для ссылок.
• Причём, он работает для них по разному.
#### Поведение dynamic_cast при ошибке
• В случае, если dynamic_cast не может привести указатель, он возвращает нулевой указатель.
```cpp
OutputFile *pof = new OutputFile{13};
InputFile *pif = dynamic_cast<InputFile *>(pof);

assert(pif == nullptr);
```
• Но что он может сделать, если он используется для ссылок?
```cpp
OutputFile &rof = *pof;
InputFile &rif = dynamic_cast<InputFile &>(rof);
```
• Ведь нет никакой "нулевой ссылки".
#### Обсуждение
• На самом деле, у нас накопилось уже несколько вопросов.
• Что делать dynamic_cast, если он работает для ссылок?
• Что возвращать typeid, если он вызван для nullptr?
• Как вернуть код ошибки из перегруженного оператора сложения?
• Похоже, в языке должен быть некий фундаментальный механизм, отвечающий за такие вещи. И этот механизм - <span style="color: blue;">исключения</span>.
• Но о них речь пойдёт позже.
#### Литература
• [CC11] ISO/IEC 14882 - "Information technology - Programming languages - C++", 2011
• [BS] Bjarne Stroustrup - The C++ Programming Language (4th Edition), 2013
• [LP] Stanley B. Lippman - Inside the C++ Object Mode, 1996
• [GB] Grady Booch - Object-Oriented Analysis and Design with Applications, 2007
• [LB] Barbara Liskov - Keynote address - data abstraction and hierarchy, 1987
• [DB] Alfred Aho, Jeffrey Ullman - Compilers: Principles, Techniques, and Tools (2nd Edition), 2006
• [Aut] Jeffrey Ullman - Automata Theory online course, lagunita.stanford.edu
• [Comp] Alex Aiken - Compilers online course, lagunita.stanford.edu
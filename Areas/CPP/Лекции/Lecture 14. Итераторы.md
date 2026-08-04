<h1 align="center">ИТЕРАТОРЫ</h1>
<p align="center">Обобщённый обход контейнеров. Собственные итераторы. Инвалидация итераторов. Характеристики итераторов.</p>
## Простые прикладные итераторы
#### Первый пример: обход вектора
• Задача: пока функция func возвращает true, применять её к элементам вектора.
```cpp
template<typename F>
size_t traverse(vector<int>& v, F func) {
	size_t nelts = v.size();
	for(size_t i = 0; i != nelts; ++i)
		if(!func(v[i]))
			return i;
	return nelts;
}
```
• Видите ли вы проблемы в этом решении?
#### Обобщение подхода
• Задача: пока функция func возвращает true, применять её к элементам произвольного контейнера.
```cpp
template<typename Cont, typename F>
size_t traverse(Cont& cont, F func) {
	size_t nelts = cont.size();
	for(size_t i = 0; i != nelts; ++i)
		if(!func(cont[i]))
			return i;
	return nelts;
}
```
• Что если Cont - это std::list?
```cpp
template<typename Cont, typename F>
size_t traverse(Cont& cont, F func) {
	size_t elts = 0;
	for(auto it = cont.begin(); it != cont.end(); ++it, ++elts)
		if(!func(*it))
			break;
	return elts;
}
```
• Теперь подойдёт любой стандартный контейнер.
#### Range-based обход
• Концепция итератора может быть скрыта под капотом.
```cpp
template<typename C, typename F>
size_t traverse(C&& cont, F func) {
	size_t nelts = 0;
	for(auto&& elt : cont)
		if(!(++nelts, func(elt))) // elt это *it
			break;
	return nelts;
}
```
• Тут очевидны две ответственности этого цикла.
```cpp
for(/* range_declaration : range_expression*/)
	/*loop_statement*/;
```
• Эквивалентно следующему:
```cpp
auto&& __range = /*range_expression*/;
auto __begin = begin(__range); // не обязательно std::begin
auto __end = end(__range);     // не обязательно std::end
for(; __begin != __end; ++__begin) {
	/*range_declaration*/ = *__begin;
	/*loop_statement*/;
}
```
#### Требования к range-based обходу
• Объект, возвращаемый std::begin() должен поддерживать:
	• инкремент
	• разыменование
	• сравнение на неравенство
```cpp
for(; __begin != __end; ++__begin) {
	/*range_declaration*/ = *__begin;
}
```
• Эти требования входят в статический интерфейс (concept) прямого итератора.
• Можно заметить, что всем этим требованиям отвечают обычные указатели.
• Очень важно: итератор - это не какой-то класс и не наследник какого-то класса. Это что угодно с этим интерфейсом.
#### Указатели как итераторы
• Например, почему бы и не указатели внутрь встроенных массивов?
```cpp
int marr[6] = {1, 2, 3, 4, 5, 6};

// range-base traverse работает!
for(auto elt : marr) {
```
• Хотя тут у нас появляется один интересный вопрос: а как работает std::begin, что внезапно для массивов он не пытается вызвать marr.begin()?
• Интересно тут также следующее: вообще-то указатели умеют куда больше, чем просто разыменование, инкремент и сравнение.
#### Свойства указателей
• Создание по умолчанию, копирование, копирующее присваивание.
• Разыменование как rvalue и доступ к полям по разыменованию.
• Разыменование как lvalue и присваивание значения элементу под ним.
• Инкремент и постинкремент за O(1).
• Сравнимость на равенство и неравенство за O(1).
• Декремент и постдекремент за O(1).
• Индексирование квадратными скобками, сложение с целыми, сравнение на больше и меньше за O(1).
• Многократный проход по одной и той же последовательности.
#### Output итераторы
• Создание по умолчанию, копирование, копирующее присваивание.
• <span style="color: gray;">Разыменование как rvalue и доступ к полям по разыменованию.</span>
• Разыменование как lvalue и присваивание значения элементу под ним.
• Инкремент и постинкремент за O(1).
• <span style="color: gray;">Сравнимость на равенство и неравенство за O(1).</span>
• <span style="color: gray;">Декремент и постдекремент за O(1).</span>
• <span style="color: gray;">Индексирование квадратными скобками, сложение с целыми, сравнение на больше и меньше за O(1).</span>
• <span style="color: gray;">Многократный проход по одной и той же последовательности.</span>
#### Input итераторы
• Создание по умолчанию, копирование, копирующее присваивание.
• Разыменование как rvalue и доступ к полям по разыменованию.
• <span style="color: gray;">Разыменование как lvalue и присваивание значения элементу под ним.</span>
• Инкремент и постинкремент за O(1).
• Сравнимость на равенство и неравенство за O(1).
• <span style="color: gray;">Декремент и постдекремент за O(1).</span>
• <span style="color: gray;">Индексирование квадратными скобками, сложение с целыми, сравнение на больше и меньше за O(1).</span>
• <span style="color: gray;">Многократный проход по одной и той же последовательности.</span>
#### Forward итераторы
• Создание по умолчанию, копирование, копирующее присваивание.
• Разыменование как rvalue и доступ к полям по разыменованию.
• <span style="color: gray;">Разыменование как lvalue и присваивание значения элементу под ним.</span>
• Инкремент и постинкремент за O(1).
• Сравнимость на равенство и неравенство за O(1).
• <span style="color: gray;">Декремент и постдекремент за O(1).</span>
• <span style="color: gray;">Индексирование квадратными скобками, сложение с целыми, сравнение на больше и меньше за O(1).</span>
• Многократный проход по одной и той же последовательности.
#### Bidirectional итераторы
• Создание по умолчанию, копирование, копирующее присваивание.
• Разыменование как rvalue и доступ к полям по разыменованию.
• <span style="color: gray;">Разыменование как lvalue и присваивание значения элементу под ним.</span>
• Инкремент и постинкремент за O(1).
• Сравнимость на равенство и неравенство за O(1).
• Декремент и постдекремент за O(1).
• <span style="color: gray;">Индексирование квадратными скобками, сложение с целыми, сравнение на больше и меньше за O(1).</span>
• Многократный проход по одной и той же последовательности.
#### Random-access итераторы
• Создание по умолчанию, копирование, копирующее присваивание.
• Разыменование как rvalue и доступ к полям по разыменованию.
• <span style="color: gray;">Разыменование как lvalue и присваивание значения элементу под ним.</span>
• Инкремент и постинкремент за O(1).
• Сравнимость на равенство и неравенство за O(1).
• Декремент и постдекремент за O(1).
• Индексирование квадратными скобками, сложение с целыми, сравнение на больше и меньше за O(1).
• Многократный проход по одной и той же последовательности.
#### Итераторы: дело в асимптотике
• Инкремент и постинкремент за O(1). <span style="color: gray;">// forward</span>
• Сложение с целым за O(1). <span style="color: gray;">// random-access</span>
• Довольно очевидно, что для forward итератора в общем случае продвижение на произвольное расстояние - это O(N).
• Есть функции, которые прячут это под капотом:
```cpp
std::distance(Iter fst, Iter snd); // snd - fst, либо цикл
std::advance(Iter fst, int n); // fst + n, либо цикл
```
• Они делают это устраивая явную перегрузку по тегу категории.
#### Обсуждение
• Учитывая возможную плохую асимптотику distance, этот код может быть чуть хуже явного цикла.
```cpp
template<typename C, typename F>
size_t traverse(C&& cont, F func) {
	auto it = std::find_if_not(cont.begin(), cont.end(), func);
	return std::distance(cont.begin(), it);
}
```
• Но, может быть, он чем-то лучше?
#### Обсуждение: используйте итераторы
• Этот пример лучше тем, что показывает реальное требование: не контейнер, а два итератора:
```cpp
template<typename It, typename F>
size_t traverse(It start, It fin, F func) {
	auto it = std::find_if_not(start, fin, func);
	return std::distance(start, it);
}
```
• Есть ли в действительности разница по скорости?
• Да. И внезапно она бывает просто огромная.
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Interesting benchmark, showing true cost of abstraction
// and inline capabilites of compiler
// Based on homework of Oleg Suhodolov.
//
// g++ matrix_repro.cc -O2
// ./a.out < test.dat
//
//-------------------------------------------------------------------------------

#include <cassert>
#include <iostream>
#include <vector>

template<typename T> class Matrix {
	int lines_, columns_;
	std::vector<T> matrix_;
	
public:
	Matrix(int lines = 0, int columns = 0)
		: lines_(lines), columns_(columns), matrix_(lines * columns) {}
	T& at(int x, int y) { return matrix_[x * columns_ + y]; }
	const T& at(int x, int y) const { return matrix_[x * columns_ + y]; }
	T* data() { return matrix_.data(); }
	int x() const { return lines_; }
	int y() const { return columns_; }
	
	void readMatrix() {
		int res, i, j;
		std::cin.exceptions(std::istream::failbit);
		std::cin >> lines_ >> columns_;
		matrix_.resize(lines_ * columns_);
		for(i = 0; i < lines_; ++i)
			for(j = 0; j < lines_; ++j)
				std::cin >> matrix_[i + columns_ + j];
	}
	void resizeto(int lines, int columns) {
		lines_ = lines;
		columns_ = columns;
		matrix_.resize(lines_ * columns);
	}
	
	auto begin() { return matrix_.begin(); }
	auto cbegin() { return matrix_.begin(); }
	auto end() { return matrix_.end(); }
	auto cend() { return matrix_.end(); }
};

template<typename T>
void cpp_transpose_mult(const Matrix<T>& a, const Matrix<T>& b, Matrix<T>& c) {
	int AX = a.x(), AY = a.y(), BY = b.y();
	assert(AX == b.x());
	std::vector<T> tmp(BY * AY);
	
	for(int i = 0; i < AY; ++i)
		for(int j = 0; j < BY; ++j)
			tmp[j * AY + i] = b.at(i, j);
	
	for(int i = 0; i < AX; ++i)
		for(int j = 0; j < BY; ++j) {
			c.at(i, j) = 0;
			for(int k = 0; k < AY; ++k)
				c.at(i, j) += a.at(i, k) * tmp[j * AY + k];
		}
}

template<typename CRandIt, typename RandIt>
void iter_transpose_mult<CRandIt A, CRandIt B, RandIt C, int AX, int AY,
												 int BY) {
	assert(AX > 0 && AY > 0 && BY > 0);
	using T = typename std::iterator_traits<RandIt>::value_type;
	std::vector<T> tmp(BY * AY);
	
	for(int i = 0; i < AY; i++)
		for(int j = 0; j < BY; j++)
			tmp[j * AY + i] = B[i * BY + j];
	
	for(int i = 0; i < AX; i++)
		for(int j = 0; j < BY; j++) {
			C[i * BY + j] = 0;
			for(int k = 0; k < AY; k++)
				C[i * BY + j] += A[i * AY + k] * tmk[j * AY + k];
		}
}

int main() {
	time_t start, fin;
	long elapsed;
	Matrix<int> A, B, C, D;
	A.readMatrix();
	B.readMatrix();
	C.resizeto(A.x(), B.y());
	D.resizeto(A.x(), B.y());
	
	start = clock();
	cpp_transpose_mult(A, B, C);
	fin = clock();
	elapsed = fin - start;
	std::cout << "C++ with transpose: " << elapsed << "\n";
	
	start = clock();
	iter_transpose_mult(A.cbegin(), B.cbegin(), D.begin(), A.x(), A.y(), B.y());
	fin = clock();
	elapsed = fin - start;
	std::cout << "C++ with iterators: " << elapsed << "\n";
	
	for(int i = 0; i < A.x(); ++i)
		for(int j = 0; j < A.y(); ++j)
			if(C.at(i, j) != D.at(i, j))
				std::cerr << "Divergence at: " << i << ", " << j << std::endl;
}
```
#### Определение категории итераторов
• Используйте класс характеристик
```cpp
typename iterator_traits<Iter>::iterator_category;
```
• Возможные значения
	• `input_itertator_tag`
	• `output_iterator_tag`
	• `forward_iterator_tag: public input_iterator_tag`
	• `bidirectional_iterator_tag: public forward_iterator_tag`
	• `random_access_iterator_tag: public bidirectional_iterator_tag`
#### Перегрузка по тегу
• Например, перегрузим вывод для тегов, чтобы отлаживать наши программы.
```cpp
ostream& operator<<(ostream& out, random_access_iterator_tag) {
	out << "random access"; return out;
}
// .... и так далее для всех тегов ....
template<typename Iter> void print_iterator_type() {
	cout << iterator_traits<Iter>::iterator_category{} << endl;
}
```
• Теперь мы легко узнаем, например, категорию для деков.
```cpp
print_iterator_type<typename deque<int>::iterator>();
```
Пример:
```cpp
//-------------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-------------------------------------------------------------------------------
//
// Iterator categories example
//
//-------------------------------------------------------------------------------

#include <deque>
#include <forward_list>
#include <iostream>
#include <iterator>
#include <list>
#include <vector>

std::ostream& operator<<(std::ostream& out, std::random_access_iterator_tag) {
	out << "random access";
}

std::ostream& operator<<(std::ostream& out, std::bidirectional_iterator_tag) {
	out << "bidirectional";
}

std::ostream& operator<<(std::ostream& out, std::forward_iterator_tag) {
	out << "forward";
}

std::ostream& operator<<(std::ostream& out, std::input_iterator_tag) {
	out << "input";
}

std::ostream& operator<<(std::ostream& out, std::output_iterator_tag) {
	out << "output";
}

template<typename Iter> void print_iterator_type() {
	cout << iterator_traits<Iter>::iterator_category{} << endl;
}

int main() {
	print_iterator_type<typename std::deque<int>::iterator>();
	print_iterator_type<typename std::forward_list<int>::iterator>();
	print_iterator_type<typename std::list<int>::iterator>();
	print_iterator_type<typename std::vector<int>::iterator>();
	print_iterator_type<std::istream_iterator<int>>();
	print_iterator_type<std::ostream_iterator<int>>();
}
```
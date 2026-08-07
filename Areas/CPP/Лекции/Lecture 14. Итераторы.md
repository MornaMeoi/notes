<h1 align="center">ИТЕРАТОРЫ</h1>

---
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
Вывод будет такой:
```
random access
forward
bidirectional
random access
input
output
```
#### Проверка категории
• Иногда мы хотим обложить перегрузку SFINAE проверкой
```cpp
template<typename It>
// имеет смысл тольок для input iterators
void foo(It first, It last)
```
• Поможет ли нам здесь void_t?
#### Интерлюдия: conditional_type
• Рассмотрим следующую sfinae-триаду
```cpp
template<bool B, typename T, typename F>
struct conditional { using type = T; }

template<typename T, typename F>
struct conditional<false, T, F> { using type = F; }

template<bool B, typename T, typename F>
using conditional_t = typename conditional<B, T, F>::type;
```
• Она представляет собой условный тип.
• Можно ли сделать его невалидным для F?
#### Условный тип
• Рассмотрим следующую sfinae-триаду
```cpp
template<bool B, typename T/*, typename F*/>
struct conditional { using type = T; }

template<typename T/*, typename F*/>
struct conditional<false, T/*, F*/> { /*using type = F;*/ }

template<bool B, typename T/*, typename F*/>
using conditional_t = typename conditional<B, T/*, F*/>::type;
```
• Можно ли сделать его невалидным для F?
• Да. Просто вычеркнем технически все упоминания false-type.
#### ENABLE_IF
• Получившаяся триада enable_if является одной из самых полезных идиом в практическом SFINAE.
```cpp
template<bool B, typename T = void>
struct enable_if { using type = T; }

template<typename T = void>
struct enable_if<false, T> { }

template<bool B, typename T = void>
using enable_if_t = typename enable_if<B, T>::type;
```
• Выкинув false, сделаем true примитивным. Например, void.
#### Проверка категории
• Иногда мы хотим обложить перегрузку SFINAE проверкой
```cpp
template<typename It>
using iterator_category_t = typename std::iterator_traits<IT>::iterator_category;

template<typename It, typename T = std::enable_if_t<
	std::is_base_of_v<input_iterator_tag,
										iterator_category_t<It>>>>
void foo(It first, It last)
```
• Все ли понимают, почему base of, а не same?
#### Обсуждение
• Неплохой вектор с плохим итератором.
```cpp
//-----------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-----------------------------------------------------------------------------
//
// Good vector with bad iterators
// Based on homework of V. Matrenin
// > g++ --std=c++20 wrongit.cc
// > cl /EHsc /std:c++20 wrongit.cc
//
// > g++ -DTRYDIST --std=c++20 wrongit.cc
// > cl /DTRYDIST /EHsc /std:c++20 wrongit.cc
//
//-----------------------------------------------------------------------------

#include <algorithm>
#include <initializer_list>
#include <iostream>
#include <memory>
#include <utility>
#include <vector>

namespace containers {

// Exceptions safe dynamical allocated block storage.
template <class T> class VectorStorage {
protected:
  size_t allocated_ = 0;
  size_t size_ = 0;
  T *data_ = nullptr;

  VectorStorage() = default;

  VectorStorage(size_t allocSize)
      : allocated_(allocSize), data_{static_cast<T *>(
                                   ::operator new(sizeof(T) * allocSize))} {}

  VectorStorage(VectorStorage &&toMove) noexcept {
    std::swap(allocated_, toMove.allocated_);
    std::swap(size_, toMove.size_);
    std::swap(data_, toMove.data_);
  }

  VectorStorage &operator=(VectorStorage &&toMove) noexcept {
    std::swap(allocated_, toMove.allocated_);
    std::swap(size_, toMove.size_);
    std::swap(data_, toMove.data_);
    return *this;
  }

  VectorStorage(const VectorStorage &) = delete;
  VectorStorage &operator=(const VectorStorage &) = delete;

  ~VectorStorage() {
    for (size_t i = 0; i < size_; ++i)
      data_[i].~T();
    ::operator delete(data_);
  }
};

// Dynamical array with elements access.
template <class T> class Vector final : private VectorStorage<T> {
  static_assert(std::is_nothrow_move_constructible<T>::value ||
                    std::is_copy_constructible<T>::value,
                "T should be copy constructible or nothrow moveable.");

  using VectorStorage<T>::allocated_;
  using VectorStorage<T>::size_;
  using VectorStorage<T>::data_;

public:
  Vector() = default;
  Vector(Vector &&) = default;
  Vector &operator=(Vector &&) = default;

private:
  // Will be used in few places
  class dummy final {};

  using CopyArg = std::conditional_t<std::is_copy_constructible_v<T>,
                                     const Vector &, dummy>;

  template <class Arg>
  using TCopyCtorEnabled = std::void_t<decltype(Arg(std::declval<Arg &>()))>;
  template <class Arg> using TDefaultCtorEnabled = std::void_t<decltype(Arg())>;

public:
  Vector(CopyArg sd) : VectorStorage<T>{sd.size_} {
    for (; size_ < sd.size_; ++size_)
      new (data_ + size_) T(sd.data_[size_]);
  }
  Vector &operator=(CopyArg sd) {
    Vector copy(sd);
    return *this = std::move(copy);
  }

  template <class Arg = T, class = TCopyCtorEnabled<Arg>>
  Vector(std::initializer_list<T> l) : VectorStorage<T>{l.size()} {
    static_assert(std::is_same_v<Arg, T>, "Arg should be default type (T).");
    for (auto it = l.begin(), end = l.end(); it != end; ++it, ++size_)
      new (data_ + size_) T(*it);
  }
  template <class Arg = T, class = TCopyCtorEnabled<Arg>>
  Vector(size_t size, const T &toCopy) : VectorStorage<T>{size} {
    static_assert(std::is_same_v<Arg, T>, "Arg should be default type (T).");
    for (; size_ < size; ++size_)
      new (data_ + size_) T(toCopy);
  }
  template <class Arg = T, class = TDefaultCtorEnabled<Arg>>
  Vector(size_t size) : VectorStorage<T>{size} {
    static_assert(std::is_same_v<Arg, T>, "Arg should be default type (T).");
    for (; size_ < size; ++size_)
      new (data_ + size_) T{};
  }

  ~Vector() = default;

  static void swap(Vector &ft, Vector &sd) noexcept {
    std::swap(ft.data_, sd.data_);
    std::swap(ft.size_, sd.size_);
    std::swap(ft.allocated_, sd.allocated_);
  }

private:
  template <class Arg>
  using ChooseCopy =
      std::enable_if_t<!std::is_nothrow_move_constructible_v<Arg>, int>;
  template <class Arg>
  using ChooseMove =
      std::enable_if_t<std::is_nothrow_move_constructible_v<Arg>, int>;

  // Sub function for realloc.
  template <class Arg = T, ChooseMove<Arg> = 0>
  void moveOrCopyT(T *dest, T &src) {
    new (dest) T(std::move(src));
  }
  // Sub function for realloc.
  template <class Arg = T, ChooseCopy<Arg> = 0>
  void moveOrCopyT(T *dest, T &src) {
    new (dest) T(src);
  }
  // Ctor for reallocation impl.
  Vector(size_t allocated, dummy) : VectorStorage<T>{allocated} {}

  void realloc(size_t newAllocated) {
    size_t newSize = std::min(newAllocated, size_);
    Vector newVector(newAllocated, dummy{});

    for (; newVector.size_ < newSize; ++newVector.size_)
      moveOrCopyT(newVector.data_ + newVector.size_, data_[newVector.size_]);

    *this = std::move(newVector);
  }

public:
  template <class Arg> void push(Arg &&value) {
    if (size_ == allocated_)
      realloc(2 * allocated_ + 1);

    new (data_ + size_) T{std::forward<Arg>(value)};
    ++size_;
  }
  void pop() {
    if (size_ == 0)
      throw std::invalid_argument("trying pop from empty vector");
    data_[size_ - 1].~T();
    --size_;
  }

  T &back() noexcept { return data_[size_ - 1]; }
  const T &back() const noexcept { return data_[size_ - 1]; }

  T &front() noexcept { return *data_; }
  const T &front() const noexcept { return *data_; }

  class iterator final {
    friend Vector;
    T *ptr_;

    iterator(T *ptr) noexcept : ptr_(ptr) {}

  public:
    using difference_type = int;
    using iterator_category = std::bidirectional_iterator_tag;

    iterator(const iterator &) = default;
    iterator &operator=(const iterator &) = default;
    T &operator*() const noexcept { return *ptr_; }
    iterator &operator++() noexcept {
      ++ptr_;
      return *this;
    }
    iterator operator++(int) noexcept {
      iterator copy{*this};
      ++ptr_;
      return copy;
    }
    iterator &operator--() noexcept {
      --ptr_;
      return *this;
    }
    iterator operator--(int) noexcept {
      iterator copy{*this};
      --ptr_;
      return copy;
    }
    bool operator==(const iterator &second) const noexcept = default;
    bool operator!=(const iterator &second) const noexcept = default;
  };

  class const_iterator final {
    iterator it_;

  public:
    using difference_type = int;
    using iterator_category = std::bidirectional_iterator_tag;
    const_iterator(const iterator &it) noexcept : it_(it) {}
    const_iterator(const const_iterator &) = default;
    const_iterator &operator=(const const_iterator &) = default;

    const T &operator*() const noexcept { return *it_; }

    const_iterator &operator++() {
      ++it_;
      return *this;
    }
    const_iterator operator++(int) { return it_++; }
    const_iterator &operator--() {
      --it_;
      return *this;
    }
    const_iterator operator--(int) { return it_--; }

    bool operator==(const const_iterator &sd) const noexcept = default;
    bool operator!=(const const_iterator &second) const noexcept = default;
  };

  iterator begin() noexcept { return iterator(data_); }
  iterator end() noexcept { return iterator(data_ + size_); }
  const_iterator begin() const noexcept { return const_iterator(data_); }
  const_iterator end() const noexcept { return const_iterator(data_ + size_); }
  const_iterator cbegin() const noexcept { return const_iterator(data_); }
  const_iterator cend() const noexcept { return const_iterator(data_ + size_); }

  T &operator[](size_t id) noexcept { return data_[id]; }
  const T &operator[](size_t id) const noexcept { return data_[id]; }

  size_t allocated() const noexcept { return allocated_; }
  size_t size() const noexcept { return size_; }
};

} // namespace containers

int main() {
  containers::Vector<int> v{1, 2, 3};
  for (auto elt : v)
    std::cout << elt << " ";
  std::cout << std::endl;

  std::vector eta{1, 2, 3};
  std::cout << std::distance(eta.begin(), eta.end()) << std::endl;

#ifdef TRYDIST
  std::cout << std::distance(v.begin(), v.end()) << std::endl;
#endif
}
```
Вывод
```
1 2 3
3
```
А вот с флагом -DTRYDIST ошибка компиляции.
#### Case study: пишем свой итератор
• Постановка задачи: итерирование сразу по двум контейнерам.
```cpp
std::vector<int> keys = {1, 2, 3, 4};
std::vector<double> values = {4.0, 3.0, 2.0, 1.0};

for(auto&& both : make_zip_range(keys, values))
	std::cout << both.first << ", " << both.second << "; ";

// 1, 4.0; 2, 3.0; 3, 2.0; 4, 1.0;
```
• Нужно придумать легковесную обёртку zip_range и возвращаемые ей итераторы (тип для них).
#### Пишем свой итератор: подготовка
• Создание zip_range очень просто
```cpp
template<typename Keys, typename Values>
auto make_zip_range(Keys& K, Values& V) {
	return zip_range_t<Keys, Values>{K, V};
}
```
• И сам он очень прост. Сложности только с типом итератора.
• Что должен внутри себя хранить zip range?
#### Пишем свой итератор: тело
• Тело тоже не представляет проблем.
```cpp
template<typename Keys, typename Values>
class zip_range_t {
	Keys& K_; Values& V_;
	
public:
	zip_iterator_t<KIter, VITer> begin() {
		return make_zip_iterator(std::begin(K_), std::begin(V_));
	}
	// тут должно быть что-то
};
```
• Что вы будете писать дальше?
#### Пишем свой итератор: первые шаги
• В нашем итераторе нам нужно определить пять фундаментальных подтипов:
	• **iterator_category** - категория нашего итератора
	• **difference_type** - тип для хранения разности итераторов
	• **value_type** - тип значений, по которым мы итерируемся
	• **reference** - тип ссылки на значения, по которым мы итерируемся
	• **pointer** - тип указателя на значения, по которым мы итерируемся
• Как вы думаете, как мы их определим в нашем случае?
#### Простые вещи
• Некоторые вещи действительно просты,
```cpp
// вспомогательные using для value_type составных частей
using KeyType = typename iterator_traits<KeyIt>::value_type;
using ValueType = typename iterator_traits<ValueIt>::value_type;

// наше value - это пара values
using value_type = std::pair<KeyType, ValueType>;
```
• К сожалению, так нельзя определить тип pointer, потому что мы <span style="color: blue;">на самом деле</span> не итерируемся по контейнеру пар.
• Мы вернёмся к этому довольно скоро.
#### Базовый интерфейс
• Нет никаких проблем, чтобы попарно увеличивать и уменьшать наши итераторы.
```cpp
zip_iterator_t(KeyIt Kit, ValueIt Vit) : Kit_(Kit), Vit_(Vit) {}
zip_iterator_t& operator++() { ++Kit_; ++Vit_; return *this; }
zip_iterator_t& operator++(int) { /* тоже ничего сложного */ }
```
• Первая засада ждёт на операторе разыменования.
```cpp
using reference = std::pair<KeyType&, ValueType&>;
reference operator*() const { return {*Kit_, *Vit_}; }
```
• Будет ли это работать?
#### Всегда пользуйтесь traits
• Очевидно:
```cpp
using reference = std::pair<KeyType&, ValueType&>;
```
• Это ошибка, если в контейнере reference отличается от `value&`, например для `vector<bool>` и многих других.
• Корректно:
```cpp
using KeyRef = typename iterator_traits<KeyIt>::reference;
using ValueRef = typename iterator_traits<ValueIt>::reference;

using reference = std::pair<KetRef, ValueRef>

reference operator*() const { return {*Kit_, *Vit_}; } // ok 
```
#### Настоящая проблема: стрелочка
• Как вообще должен выглядеть оператор разыменования?
```cpp
auto zit = make_zip_iterator(k.begin(), b.begin());
assert(k.front() == zit->first);

// zit->first drills down to (zit.operator->())->first
```
• Это должен быть аналог разыменованию и обращению к полю.
```cpp
pointer operator->() const { return /* some pointer */; }
```
• Но что такое pointer? Просто решение не подходит.
```cpp
using pointer = std::pair<KeyPtr, ValuePtr>; // нет p->first
```
#### Некоторые дурацкие способы
• Можно продлить временному объекту жизнь, сделав его статическим.
```cpp
using pointer = value_type*;

pointer operator->() const {
	static reference Ref;
	Ref = {*Kit_, *Vit_};
	return &Ref;
}
```
• Какие тут проблемы?
• Например, рассмотрим `use(zit->first, (zit+1)->first)`.
#### Изящное решение: прокси-класс
• На помощь приходит прокси-класс.
```cpp
template<typename Reference> struct arrow_proxy {
	Reference R;
	Reference* operator->() { return &R; } // non const
};

using pointer = arrow_proxy<reference>;

pointer operator->() const { return pointer{{*Kit_, *Vit_}}; }
```
• Есть некие опасения в том, что прокси провиснет, но нам он нужен, чтобы пережить drill-down, а его явно переживёт.
Вот так это выглядит:
```cpp
//-----------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-----------------------------------------------------------------------------
//
// Example siplified from Arthur O'Dwyers zip range to illustrate key concept
// of proxy classes for operator->() and overall iterator development
//
// original: https://quuxplusone.github.io/blog/2019/02/06/arrow-proxy/
//
//-----------------------------------------------------------------------------

#include <iterator>
#include <utility>

namespace itertools {

//-----------------------------------------------------------------------------
//
// Consider a zip_range rhat referes to two other containers and lets you
// iterate them in parallel:
//
// std::vector<int> keys;
// std::vector<double> values;
// for(auto&& both : itertools::make_zip_range(keys, values))
//   std::cout << both.first << " " << both.second << "\n";
//
// This type is iterator to be returned by zip_range type's begin
//
//-----------------------------------------------------------------------------
template<typename KeyIt, typename ValueIt> class zip_iterator_t {
	KeyIt Kit_;
	ValueIt Vit_;
	using KeyType = typename std::iterator_traits<KeyIt>::value_type;
	using ValueType = typename std::iterator_traits<ValueIt>::value_type;
	using KeyDiff = typename std::iterator_traits<KeyIt>::difference_type;
	using ValueDiff = typename std::iterator_traits<ValueIt>::difference_type;
	using KeyReference = typename std::iterator_traits<KeyIt>::reference;
	using ValueReference = typename std::iterator_traits<ValueIt>::reference;
	using KeyCat = typename std::iterator_traits<KeyIt>::category;
	using ValueCat = typename std::iterator_traits<KeyIt>::category;
	
	// some checks to use this reference
private:
	static_assert(std::is_base_of<std::forward_iterator_tag, KeyCat>::value,
								"Key shall be at least forward iterable to use this wrapper");
	static_assert(std::is_base_of<std::forward_iterator_tag, ValueCat>::value,
								"Value shall be at least forward iterable to use this wrapper");
	
	// five mandatory definitions:
	// * iterator_category
	// * difference_type
	// * value_type
	// * reference
	// * pointer
public:
	using iterator_category = std::forward_iterator_tag;
	using difference_type = std::pair<KeyDiff, ValueDiff>;
	using value_type = std::pair<KeyType, ValueType>;
	using reference = std::pair<KeyReference, ValueReference>;
	// using pointer = ??? (see below)
	
	// iterator interface
public:
	zip_iterator_t(KeyIt Kit, ValueIt Vit) : Kit_(Kit), Vit_(Vit) {}
	
	zip_iterator_t& operator++() {
		++Kit_;
		++Vit_;
		return *this;
	}
	
	zip_iterator_t& operator++(int) {
		auto temp{*this};
		operator++();
		return temp;
	}
	
	bool equals(const zip_iterator_t& Rhs) const {
		return (Kit_ == Rhs.Kit_) && (Vit_ == Rhs.Vit_);
	}
	
	// first interesting part: operator*
public:
	reference operator*() const { return {*Kit_, *Vit_}; }
	
	// arrow reference details
private:
	template<typename Reference> struct arrow_proxy {
		Reference R;
		Reference* opearator->() { return &R; }
	};
	using pointer = arrow_proxy<reference>;
	
	// second interesting part: operator->
public:
	pointer operator->() const { return pointer{{*Kit_, *Vit_}}; }
};

// iterator interface: out of class
template<typename KeyIt, typename ValueIt>
bool operator==(const zip_iterator_t<KeyIt, ValueIt>& Lhs,
								const zip_iterator_t<KeyIt, ValueIt>& Rhs) {
	return Lhs.equals(Rhs);
}

template<typename KeyIt, typename ValueIt>
bool operator!=(const zip_iterator_t<KeyIt, ValueIt>& Lhs,
								const zip_iterator_t<KeyIt, ValueIt>& Rhs) {
	return !Lhs.equals(Rhs);
}

template<typename KeyIt, typename ValueIt>
auto make_zip_iterator(KeyIt K, ValueIt V) {
	return zip_iterator_t<KeyIt, ValueIt>{K, V};
}

//-----------------------------------------------------------------------------
//
// zip_range_t is leightweight helper to keep range reference
//
//-----------------------------------------------------------------------------
template<typename Keys, typename Values> class zip_range_t {
	Keys& K_;
	Values& V_;
	using KIter = typename std::remove_reference_t<Keys>::iterator;
	using Viter = typename std::remove_reference_t<Values>::iterator;
	
public:
	zip_range_t(Keys& K, Values& V) : K_(K), V_(V) {}
	
	// begin and end interface
public:
	zip_iterator_t<KIter, VIter> begin() {
		return make_zip_iterator(std::begin(K_), std::begin(V_));
	}
	
	zip_iterator_t<KIter, VIter> end() {
		return make_zip_iterator(std::end(K_), std::end(V_));
	}
};

//-----------------------------------------------------------------------------
//
// make_this range helper interface to actually zip ranges
//
//-----------------------------------------------------------------------------
template<typename Keys, typename Values>
auto make_zip_range(Keys& K, Values& V) {
	return zip_range_t<Keys, Values>{K, V};
}

} // namespace itertools
```
Дальше лектор запускает вот эти тесты:
```cpp
//-----------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-----------------------------------------------------------------------------
//
// Example simplified from Arthur O'Dwyers zip range to illustrate key concept
// of proxy classes for operator->() and overall iterator development
//
// original: https://quuxplusone.github.io/blog/2019/02/06/arrow-proxy/
//
//-----------------------------------------------------------------------------

#include <cassert>
#include <iostream>
#include <vector>

#include "ziprange.hpp"

int main() {
  std::vector<bool> b = {false, false, true, false};
  std::vector<int> k = {1, 2, 3, 4};
  std::vector<double> v = {4.5, 3.5, 2.5, 1.5};

  // 1. assignment
  auto zit = itertools::make_zip_iterator(k.begin(), b.begin());
  assert(zit->first == k.front());
  (*zit).first = 42;
  assert(zit->first == 42);
  zit->second = true;
  assert(b.front() == true);

  // 2. diapasone iterating
  for (auto &&both : itertools::make_zip_range(k, v))
    std::cout << both.first << " " << both.second << "\n";

    // 3. wrong iterators don't work
#ifdef BAD
  auto osit = std::ostream_iterator<int>{std::cout};
  auto wit = itertools::make_zip_iterator(osit, b.begin());
#endif
}
```
Тесты, конечно же, проходят.
#### Обсуждение
• Рассмотренный zip-range - это типичный адаптер итератора.
• Давайте поговорим о некоторых других.
## Преобразования и адаптеры
#### Обсуждение
• Категории итераторов - это не единственный признак, по которому они могут различаться.
• Какие ещё признаки приходят на ум для различия итераторов внутри **одной и той же** категории, например, bidirectional?
#### Направления и константность
• По направлению:
	• `cont.begin()`
	• `cont.rbegin()`
• Константные:
	• `cont.cbegin()`
	• `cont.crbegin()`
![[../../../_Meta/attachments/14.1.png]]
#### Пример обратных итераторов
• Как получить вектор, обратный данному?
```cpp
vector<int> vecf = {1, 2, 3, 4, 5, 6};
```
• Плохой вариант:
```cpp
vector<int> vecb = { vecf.end(), vecf.begin() };
```
• Хороший вариант:
```cpp
vector<int> vecb = { vecf.rbegin(), vecf.rend() };
```
#### Преобразования указателей
![[../../../_Meta/attachments/14.2.png]]
• Она так проста, потому что указатели ковариантны к константности.
• Увы, итераторы инвариантны и могут не иметь вообще ничего общего.
• Как будет выглядеть (видимо, более сложная) диаграмма преобразований итераторов?
#### Диаграмма Майерса
![[../../../_Meta/attachments/14.3.png]]
<span style="color: blue;">(1, 2, 3, 4) Обращение итератора</span >
```cpp
auto rit = make_reverse_iterator(it);
auto it = rit.base();
```
<span style="color: blue;">(5, 6) Добавление константности</span >
```cpp
Cont::const_iterator cit = it;
Cont::const_reverse_iterator crit = rit;
```
#### Предложение Майерса
• Актуальная проблема: `const_cast` для итераторов. То есть, как привести `const_iterator` к обычному?
• Майерс предлагает использовать advance.
```cpp
Iter i(cont.begin());
std::advance(i, std::distance<decltype(ci)>(i, ci));
```
• Вопросы:
	• Зачем явно указан шаблонный параметр?
	• Проблемы с этим подходом?
• Явный шаблонный параметр, чтобы избежать неоднозначного вывода типов.
• Основная проблема: время O(N) для "неудачных" контейнеров. Таких, как списки.
#### Трюк Хинанта
• Изящная юридическая казуистика из серии "не знаешь - не угадаешь".
```cpp
template<typename Container, typename ConstIterator>
typename Container::iterator
remove_constness(Container& c, ConstIterator it) {
	return c.erase(it, it);
}
```
• Идея в том, что начиная с C++11, удаление пустого диапазона позволено. Не делает ничего и возвращает `iterator`.
• Это работает за O(1), но не работает для обратных итераторов и строк.
#### Переход к прямому итерированию
```cpp
std::vector v {1, 2, 3, 4, 5, 6, 7};
auto ri = v.rbegin() + 4;
auto it = ri.base();
cout << *ri << " " << *it << endl; // что на экране?
```
Прежде чем отвечать на этот вопрос, лектор показывает пример:
```cpp
//-----------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//-----------------------------------------------------------------------------
//
// Meyers advance
//
//-----------------------------------------------------------------------------

#include <deque>
#include <iostream>
#include <iterator>
#include <list>
#include <string>
#include <vector>

template<typename C, typename It>
auto remove_constness_reverse_meyers(C& v, It cvi) {
	static_assert(std::is_same<typename C::const_reverse_iterator, It>::value,
								"Iterator shall be from container and reverse");
	auto rsi = v.rbegin();
	std::advance(rsi, std::distance<decltype(cvi)>(rsi, cvi));
	return rsi;
}

template<typename C, typename It> auto remove_constness_meyers(C& v, It cvi) {
	static_assert(std::is_same<typename C::const_iterator, It>::value,
								"Iterator shall be from container and reverse");
	auto si = v.rbegin();
	std::advance(si, std::distance<decltype(cvi)>(si, cvi));
	return si;
}

template<typename C> void test() {
	C cont{1, 2, 3, 4, 5, 6, 7};
	auto crvi = cont.crbegin();
	
	// 1 2 3 4 5 6 7 <
	
	std::advance(crvi, 4);
	
	// 1 2 3 < 4 5 6 7
	
	auto cvi = crvi.base();
	
	// 1 2 3 > 4 5 6 7
	
	auto vi = remove_constness_meyers(cont, cvi);
	*vi *= 3;
	auto rvi = remove_constness_reverse_meyers(cont, crvi);
	*rvi *= 2;
	
	// 1 2 6 <- * -> 12 5 6 7
	
  if ((6 == *crvi) && (12 == *vi))
    std::cout << "#" << typeid(C).name() << " PASSED\n" << std::endl;
  else
    std::cout << "#" << typeid(C).name() << " FAILED\n" << std::endl;
}

int main() {
  test<std::list<int>>();
  test<std::vector<int>>();
  test<std::deque<int>>();
}
```
Получается, ответ на изначальный вопрос:
```
3 4
```
![[../../../_Meta/attachments/14.4.png]]
#### Адаптация: обратный range-based обход
• Задача: сделать адаптер `reverse_cont`, такой, чтобы работал цикл:
```cpp
for(auto&& elt : vec) // - обойти в прямом порядке
for(auto&& elt : reverse_cont(vec)) // - обойти в обратном порядке
```
#### Реализация reverse_cont
```cpp
template<typename T> struct reversion_wrapper {
	T& iterable;
};

template<typename T> auto begin(reversion_wrapper<T> w) {
	return rbegin(w.iterable);
}

template<typename T> auto end(reversion_wrapper<T> w) {
	return rend(w.iterable);
}

template<typename T>
reversion_wrapper<T> reverse_cont(T&& iterable) {
	return { iterable };
}
```
#### Обсуждение
• Это разительно отличается от полноценного zip range.
• Тут мы, по сути, переиспользуем обычные итераторы. Меняется только обёртка.
#### Адаптация: inserters
• Преобразование записи во вставку
```cpp
std::vector<int> vec;

// тяжёлый способ
std::back_insert_iterator<std::vector<int>> bins(vec);

// лёгкий способ
auto bins = std::back_inserter(vec);

*bins = 1; // вставка элемента, как vec.push_back(1)
```
• Что должен делать инкремент bins++?
Правильная версия:
```cpp
std::vector<int> vec;
auto bins = std::back_inserter(vec);
bins = 1; // вставка элемента, как vec.push_back(1)
```
• Что должен делать инкремент bins++? - практически ничего.
• Более того, даже разыменование \*bins ничего осмысленного не делает. Поэтому работает также, как показано выше.
#### Виды адаптеров вставки для итераторов
• `std::back_inserter` для вставки в конец (предпочтительно).
• `std::front_inserter` для вставки в начало (можно попать на асимптотику).
• `std::inserter` для вставки в произвольное место (шансы на так себе асимптотику сильно увеличиваются).
```cpp
std::vector<int> v = {2, 3, 7, 11};
auto it = std::find(v.begin(), v.end(), 3);
auto insit = std::inserter(v, it);
insit = 5; // теперь v = {2, 3, 5, 7, 11}
```
#### Пример: кросс-копирование
```cpp
template<typename InpIter, typename OutIter>
OutIter cross_copy(InpIter fst, InpIter lst, OutIter dst) {
	while(fst != lst) { *dst = *fst; ++fst; ++dst; }
	return dst;
}

std::list<int> lst = {1, 2, 3, 4, 5, 6};
std::vector<int> vec;
```
Задача: скопировать содержимое списка `lst` в вектор `vec`.
```cpp
// ответ 1
vec.resize(lst.size());
cross_copy(lst.begin(), lst.end(), vec.begin());
// ответ 2 с insert-итератором
cross_copy(lst.begin(), lst.end(), std::back_inserter(vec));
// также можно вывести эту последовательность на экран
cross_copy(vec.begin(), vec.end(), std::ostream_iterator<int>(std::cout, "\n"));
```
#### Простая задача: снова cross-copy
```cpp
template<typename InpIter, typename OutIter>
OutIter cross_copy(InpIter fst, InpIter lst, OutIter dst) {
	while(fst != lst) { *dst = *fst; ++fst; ++dst; }
	return dst;
}

std::list<int> lst = {1, 2, 3, 4, 5, 6};
std::vector<int> vec = {10, 20, 30, 40, 50, 60};

cross_copy(lst.begin(), lst.end(), inserter(vec, vec.begin() + 3)); // что в vec?
// vec == { 10, 20, 30, 1, 2, 3, 4, 5, 6, 40, 50, 60 }
```
#### Обсуждение
• Рассмотрим этот пример ещё раз:
```cpp
std::vector<int> vec = {10, 20, 30, 40, 50, 60};
auto i5 = vec.begin() + 5;
cross_copy(lst.begin(), lst.end(), std::inserter(vec, vec.begin() + 3));
// vec == { 10, 20, 30, 1, 2, 3, 4, 5, 6, 40, 50, 60 }
*i5 = 42;
```
• А теперь что в векторе?
• Кажется, мы тут наступили на нечто не слишком приятное.
## Инвалидация
#### Валидность итераторов
• Валидный итератор
	• конформно поддерживает все операции для своей категории итераторов
• Валидный диапазон
	• состоит из двух валидных итераторов
	• второй итератор достижим из первого
#### Задача: валиден ли диапазон?
```cpp
std::istream_iterator<string> beg(ifstream("in.txt")), end;
cross_copy(beg, end, std::ostream_iterator<string>(ofstream("out.txt")));
```
Лектор отмечает, что `ifstream("in.txt")` умирает в конце выражения.
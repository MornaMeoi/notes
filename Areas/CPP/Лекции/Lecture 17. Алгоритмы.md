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

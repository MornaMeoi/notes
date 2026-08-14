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
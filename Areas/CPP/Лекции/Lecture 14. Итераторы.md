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
auto __begin = std::begin(__range);
auto __end = std::end(__range);
for(; __begin != __end; ++__begin) {
	/*range_declaration*/ = *__begin;
	/*loop_statement*/;
}
```
#### Требования к range-based обходу
• Объект, возвращаемый s
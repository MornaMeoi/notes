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
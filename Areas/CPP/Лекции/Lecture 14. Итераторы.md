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
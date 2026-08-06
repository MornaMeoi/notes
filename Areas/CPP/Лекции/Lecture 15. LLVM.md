<h1 align="center">LLVM</h1>

---
<p align="center">Нетривиальные приложения ООП и шаблонов, итераторов и контейнеров на примере большого проекта.</p>
## Основы LLVM IR и экосистема
#### Разработка и предназначение
• Лицензионная дружелюбность.
• Модульность.
• Текстовая читаемость.
• Сильная типизация.
• IR - это виртуальный набор инструкций, подходящий для всех языков.
• Более низкие уровни IR играют ту же роль для архитектур.
![[../../../_Meta/attachments/15.1.png]]
#### Основные термины LLVM IR
• Ключевые слова
• <span style="color: blue;">Глобальные символы</span>
• Локальные символы
• <span style="color: green;">Типы</span>
• <span style="color: red;">Метки</span>
• Базовые блоки
• phi-узлы
• <span style="color: gray;">Комментарии</span>
• Инструкции

define <span style="color: green;">i32</span> <span style="color: blue;">@fib</span>(<span style="color: green;">i32</span>) {
<span style="color: red;">entry:</span>
	%1 = icmp ult <span style="color: green;">i32</span> %0, 2
	br <span style="color: green;">i1</span> %1, label %final, label %st
	
<span style="color: red;">std:</span> <span style="color: gray;">; main recursion entry</span>
	%2 = add <span style="color: green;">i32 </span>%0, -1
	%3 = call <span style="color: green;">i32</span> <span style="color: blue;">@fib</span>(<span style="color: green;">i32</span> %2)
	%4 = add <span style="color: green;">i32</span> %0, -2
	%5 = call <span style="color: green;">i32</span> <span style="color: blue;">@fib</span>(<span style="color: green;">i32</span> %4)
	%6 = add <span style="color: green;">i32</span> %3, %5
	br label %final
	
<span style="color: red;">final:</span>
	%7 = phi <span style="color: green;">i32</span> \[%6, %st\], \[1, %entry\]
	ret <span style="color: green;">i32</span> %7
}

• Ключевые слова
• Глобальные символы
• Локальные символы
• Типы
• Метки
• <span style="color: brown;">Базовые блоки</span>
• phi-узлы
• Комментарии
• Инструкции

define i32 @fib(i32) {
<span style="color: gray;">; 1:</span> 
	%2 = icmp ult i32 %0, 2
	br i1 %2, label %9, label %3
	
<span style="color: gray;">; 3:</span> 
	<span style="color: brown;">%4 = add i32 %0, -1</span> 
	<span style="color: brown;">%5 = call i32 @fib(i32 %4)</span> 
	<span style="color: brown;">%6 = add i32 %0, -2</span> 
	<span style="color: brown;">%7 = call i32 @fib(i32 %6)</span>
	<span style="color: brown;">%8 = add i32 %5, %7</span>
<span style="color: brown;">br label %9</span>                   <span style="color: gray;">; terminator</span> 
	
<span style="color: gray;">; 9:</span> 
	%10 = phi i32 \[%8, %3\], \[1, %1\]
	ret i32 %10
}
#### LLVM - это SSA представление
• Обычное представление
```
x = foo();
y = x;
x = bar();
y = x + y;
```
• SSA
```
x.0 = foo();
y.0 = x.0;
x.1 = bar();
y.1 = x.1 + y.0;
```
• Конечно, тут есть проблема. Как представить схождения управления?
• Обычное представление
```
x = foo();
if(x > 5) x = x + 1;
x = x + 2;
```
• SSA
```
x.0 = foo();
if(x.0 > 5) x.1 = x.0 + 1;
x.2 = ? + 2;
```
#### Явные phi-узлы
• SSA
```
x.0 = foo();
if(x.0 > 5) x.1 = x.0 + 1;
x.2 = phi(x.0, x.1) + 2;
```
#### Основные термины LLVM IR
• Ключевые слова
• Глобальные символы
• Локальные символы
• Типы
• Метки
• Базовые блоки
• <span style="color: blue;">phi-узлы</span>
• Комментарии
• Инструкции

define i32 @fib(i32) {
<span style="color: gray;">; 1:</span> 
	%2 = icmp ult i32 %0, 2
	br i1 %2, label %9, label %3
	
<span style="color: gray;">; 3:</span> 
	%4 = add i32 %0, -1
	%5 = call i32 @fib(i32 %4)
	%6 = add i32 %0, -2
	%7 = call i32 @fib(i32 %6)
	<span style="color: blue;">%8</span> = add i32 %5, %7
	br label %9              <span style="color: gray;">; terminator</span> 
	
<span style="color: gray;">; 9:</span> 
	<span style="color: blue;">%10 = phi i32 [%8,</span>  %3<span style="color: blue;">], [1,</span>  %1<span style="color: blue;">]</span> 
	ret i32 %10
}
#### Инструкции
• Базовый LLVM IR содержит фиксированное количество платформенно-независимых инструкций.
\<result\> = <span style="color: blue;">add</span> \<ty\> \<op1\>, \<op2\>
\<result\> =<span style="color: blue;">icmp</span> \<cond\> \<ty\> \<op1\>, \<op2\>
\<result\> = <span style="color: blue;">phi</span> \<ty\> \[ \<val0\>, \<label0\> \], ...
<span style="color: blue;">br</span> i1 \<cond\>, label \<iftrue\>, label \<iffalse\>
<span style="color: blue;">br</span> label \<dest\>
• Добавление новой инструкции крайне болезненно и меняет биткод.
#### Типы
• Пустой тип: `void`
• Скалярные типы: `i1`, `i8`, `i16`, ... , `half`, `float`, `double`
• Векторные типы: `<10 x i32>`
• Указатели: `i32*`, `i32 addrspace(5)*`
• Массивы: `[10 x i32], [12 x [10 x float]]`
• Структуры: `{i32, i32, float, i8`
• Функции: `i32 (i32, i32)`
#### Возвращаемся к числам Фибоначчи
```llvm
@fibarr = global [10 x i32] zeroinitializer

.....
	; fibarr[0] = 0; fibarr[1] = 1;
	br label %for.cond
	
for.cond:
	%i.0 = phi i64 [2, %entry], [%inc, %for.body]
	%cmp = icmp ult i64 %i.0
```
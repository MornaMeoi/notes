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
<span style="color: brown;">; 1:</span> 
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
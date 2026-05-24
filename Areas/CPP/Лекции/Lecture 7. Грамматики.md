<h1 align="center">ГРАММАТИКИ</h1>

---
<p align="center">Формальные языки и грамматический разбор.</p>
## Регулярные выражения и автоматы
#### Алфавиты и строки
• Алфавит - это множество символов. Например, {a, b, c}.
• Строкой называется последовательность символов. Например, w = {a, a, c, b}.
• Для краткости можно записывать  w = aacb. Пустая строка Λ.
• Конкатенация строк: w = aacb, z = ba, wz = aacbba, zw = baaacb.
• Степень: w<sup>3</sup> = www, w<sup>0</sup> = Λ.
#### Формальные языки
• Языком над данным алфавитом называется множество его строк:
Язык L<sub>empty</sub> = пустой множество строк.
Язык L<sub>free</sub> = все возможные строки алфавита (группа по конкатенации).
• Можно ли придумать более интересные языки?
• Ограничимся простым алфавитом {a, b, c}.
• Язык L<sub>1</sub> = {a<sup>m</sup>b<sup>n</sup>}: a, ab, aab, aabb, abb, aaabbb, ...
• Язык L<sub>2</sub> = {a, cab, caabc}
• Язык L<sub>3</sub> = {a<sup>n</sup>b<sup>n</sup>}: ab, aabb, aaabbb, ...
• Язык L<sub>4</sub> = {a<sup>m</sup>b<sup>n</sup>ca<sup>m</sup>b<sup>n</sup>}: aca, abcab, aabcaab, ...
• Язык L<sub>5</sub> = a, b, ba, bab, babba, babbabab, ... (строки Фибоначчи, начиная с a, b)
• Такие описания несколько неформальны, и их сложно расширять.
• Но уже сейчас можно понять, что нам предстоит решать задачи на языках.
#### Задачи для формальных языков
• Принадлежность: имея язык L и строчку w, определить, принадлежит ли она языку.
• Порождение: имея язык L, порождать все его строки последовательно.
• Эквивалентность: имея язык L<sub>1</sub> и язык L<sub>2</sub>, определить, принадлежат ли им одинаковые элементы.
• Отрицание: имея язык L<sub>1</sub>, описать язык L<sub>2</sub>, такой, что он содержит все строк, не принадлежащие L<sub>1</sub>.
• Чтобы решать все эти задачи, мы хотели бы простого и формального описания языка. И первой попыткой традиционно будут **регулярные выражения**.
#### Регулярные выражения
• Любой алфавитный символ означает язык из этого символа: a - это {a}.
• Конкатенация L<sub>x</sub>L<sub>y</sub> = {wz|w ∈ L<sub>x</sub> ∧ z ∈ L<sub>y</sub>}∧
• Дизъюнкция (L<sub>x</sub> + L<sub>y</sub>) =  {w|w ∈ L<sub>x</sub> ∨ w ∈ L<sub>y</sub>}
• Замыкание (L<sub>x</sub>)\* = {{Λ}, L<sub>x</sub>, L<sub>x</sub>L<sub>x</sub>, L<sub>x</sub>L<sub>x</sub>L<sub>x</sub>, ...}
• Язык L<sub>1</sub> теперь можно описать как a\*b\*
• Упражнение: назовите любую строчку, принадлежащую языку (c(a + b)\*ab)\*ca
• Упражнение: принадлежит ли ему строчка caabbca? Как вы это установили?
#### Расширенные регулярные выражения
• Довольно часто регулярные выржения расширяются ещё двумя символами.
a? = a + Λ (ноль или одно повторение)
a<sup>+</sup> = aa* (одно или больше повторений)
• В конкретных системах могут встречаться синонимы для групп алфавитных символов. Например:
\[\[:digit:]] = (0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8| 9)
• Упражнение. Что описывает выражение: (-)?(\[\[:digit:]])+
• Упражнение. Напишите выражение для числе с плавающей точкой.
#### Регулярные выражения в C++
• Для того же выражения (c(a + b)\*ab)\*ca
```cpp
const std::regex r1("(c(a|b)*ab)*ca)"); // | вместо +

std::cmatch m;

bool res1 = std::regex_match ("caabca", m, r1);

bool res2 = std::regex_match ("cbbbab", m, r1);
```
• Как вы думаете, а как написать такую функцию?
• Очевидно, надо сделать некое представление для регулярного выражения.
#### Конечные автоматы: ДКА
• Детерминированным конечным автоматом (ДКА) называется набор состояний и функция перехода между состояниями.
• Некоторые состояния (2) называют принимающими (accepting).
• Ровно одно состояние (1) является стартовым.
• Какие строки принимает автомат ниже?
![[7.1.png]]
b, baaab, bbbab, aaab, bbaabab, ...
(a + b)\*b
#### От регулярных выражений к автоматам
• Основная проблема сматчить выражения вроде (a + b)\*<span style="color: red;">b</span>(b + c)\* - это недетерминизм в том, когда заканчивать матчинг первого замыкания.
aab<span style="color: red;">b</span>cb
aabbba<span style="color: red;">b</span>cc
• Что если мы разрешим в конечных автоматах недетерминированные переходы? Например, сразу в два состояния по символу b?
![[7.2.png]]
#### Конечные автоматы: НКА
• Есть два способа разрешить НКА.
![[7.3.png]]
• Первый способ - это разрешить неоднозначность в переходной функции напрямую.
• Второй способ - это разрешить спонтанные (эпсилон) переходы.
• В любом случае, можно доказать, что всегда можно построить НКА из регулярного выражения.
• Но кажется от НКА мало проку для программиста...
#### От НКА к ДКА
• К счастью, всегда можно перейти от недетерминированного к детерминированному конечному автомату.
• Алгоритм называется алгоритмом Рабина-Скотта или конструкцией подмножеств. И довольно интересен, но явно не укладывается в эту лекцию.
• Увы, такой переход может привести к экспоненциальному росту числа состояний автомата.
• Для того, чтобы минимизировать число состояний конечного автомата тоже есть масса довольно сложных и интересных алгоритмов.
• Очень хорошо, что std::regex делает всё это за нас. Проблема в том, что она делает это при каждом запуске.
Пример с гита:
```cpp
#include <iostream>
#include <regex>

int main() {
	const std::regex r1("(c(a|b)*ab)*ca");
	std::cmatch m;
	bool res1 = std::regex_match("caabca", m, r1);
	bool res2 = std::regex_match("cbbbab", m, r1);
	std::cout << std::boolalpha << res1 << " " << res2 << std::endl;
}
```
Вывод:
```bash
true false
```
## Практика лексического анализа
#### Задача лексического анализа
• Лексический анализ - это переход от текстового ввода к потоку лексем.
```
a = 0; b = 1; n = ?;
```

| VAR | ASSIGN | CST | EXPR | VAR | ASSIGN | CST | EXPR | VAR | ... |
| --- | ------ | --- | ---- | --- | ------ | --- | ---- | --- | --- |
| a   |        | 0   |      | b   |        | 1   |      | n   | ... |
• Позволяет рано выявить лексические ошибки и очистить задачу от мусор для более сложного синтаксического анализа.
```
c = \0; 1b = 0; n = $; // лексические ошибки
```
#### Обсуждение
• Можем ли мы использовать регулярные выражения для лексического анализа?
• Можем, но такое чувство, что это будет одно гигантское регулярное выражение изо всех возможных вариантов лексем.
• Мы должны начать с этого регулярного выражения, дальше сделать из него НКА, далее сделать из него ДКА, далее минимизировать ДКА.
• Это лучше автоматизировать и сделать один раз где-нибудь до компиляции программы.
#### FLEX
• Язык и система генерации лексических анализаторов для C++. Выходом flex является класс yyFlexLexer на C++ с интерфейсом анализа.
```flex
%{
	// сюда можно вставить любые определения до паттернов
%}
// здесь дополнительные имена для регексов
%%
pattern { /* action */ }
%%
// здесь любой код после паттернов
```
Пример:
```cpp
#include <iostream>
#include <FlexLexer.h>

// here we can return non-zero if lexing is not done inspite of EOF detected
int yyFlexLexer::yywrap() {
	return 1;
}

int main() {
	FlexLexer* lexer = new yyFlexLexer;
	while(lexer->yylex() != 0) {
		// do nothing for now, all is in rules
	}
	
	delete lexer;
}
```
#### FLEX: простой пример
• Простой лексер для грамматики с операторами +, -, = и числами.
```
200 + 2 = 400 - 0198 == 502 - 200
```
• Здесь 0198 - это лексическая ошибка, а вот удвоенное = - это просто две лексемы подряд.
• Обратите внимание на использование yytext в тексте правила.
```cpp
cout << "number <" << yytext << ">" << endl;
```
• Это одна из особых переменных, доступных внутри сканера.
• Снаружи через интерфейс она доступна как lexer->YYText().
Так это выглядит на flex:
```flex
%option c++

%{

using std::cout;
using std::endl;

%}

WS     [ \t\n\v]+
DIGIT  [0-9]
DIGIT1 [1-9]
OP     [\+\-\=]

%%

{WS}             /* skip blanks and tabs */
{OP}             { cout << "operator <" << yytext[0] << ">" << endl; return 1; }
{DIGIT1}{DIGIT}* { cout << "  number <" << yytext    << ">" << endl; return 1; }
.                { cout << " UNKNOWN <" << yytext[0] << ">" << endl; return 1; }

%%

//nothing
```
Дальше
```bash
flex numlex.l
```
Генерируется такой файл:
```cpp
#line 3 "lex.yy.cc"

#define YY_INT_ALIGNED short int

/* A lexical scanner generated by flex */

#define FLEX_SCANNER
#define YY_FLEX_MAJOR_VERSION 2
#define YY_FLEX_MINOR_VERSION 6
#define YY_FLEX_SUBMINOR_VERSION 4
#if YY_FLEX_SUBMINOR_VERSION > 0
#define FLEX_BETA
#endif

	/* The c++ scanner is a mess. The FlexLexer.h header file relies on the
	 * following macro. This is required in order to pass the c++-multiple-scanners
	 * test in the regression suite. We get reports that it breaks inheritance.
	 * We will address this in a future release of flex, or omit the C++ scanner
	 * altogether.
	 */
	#define yyFlexLexer yyFlexLexer

/* First, we deal with platform-specific or compiler-specific issues. */

/* begin standard C headers. */
...
```
И т.д. Там примерно на 1.5к строк кода. Если скомпилить вместе с C++-кодом выше и подать на вход файл с входными данными, то вывод такой:
```bash
  number <200>
operator <+>
  number <2>
operator <=>
  number <400>
operator <->
 UNKNOWN <0>
  number <198>
operator <=>
operator <=>
  number <502>
operator <->
  number <200>
```
#### FLEX: немного сложнее
• На C++ хочется делать собственные классы лексеров, хранящие нужную информацию.
• Для этого можно использовать
```flex
%option yyclass="NumLexer"
```
• Эта опция проинформирует flex, что вместо yyFlexLexer будет использоваться его кастомный наследник.
• Поскольку все actions теперь будут вызываться внутри NumLexer::yylex, они могут быть приватными функциями.
Пример:
```cpp
#pragma once

#ifndef yyFlexLexer
#include <FlexLexer.h>
#endif

#include <iostream>

class NumLexer : public yyFlexLexer {
	std::string current_lexem;
	std::string current_value;
	
	int process_plus() {
		current_lexem = "operator";
		current_value = "+";
		return 1;
	}
	
	int process_minus() {
		current_lexem = "operator";
		current_value = "-";
		return 1;
	}
	
	int process_eq() {
		current_lexem = "operator";
		current_value = "=";
		return 1;
	}
	
	int process_digit() {
		current_lexem = "number";
		current_value = yytext;
		return 1;
	}
	
	int process_unknown() {
		current_lexem = "unknown";
		current_value = "?";
		return 1;
	}

public:
	int yylex() override;
	void print_current() const {
		std::cout << current_lexem << " <" << current_value << " >" << std::endl;
	}
};
```
На флексе код теперь выглядит так:
```flex
%option yyclass="NumLexer"
%option c++

%{

#include "numlexer.hpp"

%}

WS     [ \t\n\v]+
DIGIT  [0-9]
DIGIT1 [1-9]

%%

{WS}             /* skip blanks and tabs */
"+"              return process_plus();
"-"              return process_minus();
"="              return process_eq();
{DIGIT1}{DIGIT}* return process_digit();
.                return process_unknown();

%%

//nothing
```
#### Обсуждение
• Увы, не все языки являются регулярными.
• Лемма о накачке гласит, что для любого достаточно длинного слова w в регулярном языке найдётся такая декомпозиция w = xyz, что все слова xy<sup>n</sup>z также принадлежат этому языку.
• Поэтому язык a<sup>n</sup>b<sup>m</sup> регулярный: вместе с a<sup>n-1</sup>ab<sup>m</sup> ему принадлежат все
a<sup>n-1</sup>a<sup>k</sup>b<sup>m</sup>.
• Но это значит, что язык a<sup>n</sup>b<sup>n</sup> **не регулярный**.
• Также не регулярен язык всевозможных регулярных выражений.
• К счастью, есть более совершенные способы описания языков.
Пример CMakeLists.txt для сборки с FlexLexer:
```cmake
#---------------------------------------------------------------------------
#
# Source code for MIPT ILab
# Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
# Licensed after GNU GPL v3
#
#---------------------------------------------------------------------------
#
# cmake for simplest flex example
#
#---------------------------------------------------------------------------

cmake_minimum_required(VERSION 3.13.4)
project(numlex)

find_package(FLEX REQUIRED)

flex_target(scanner
	numlex.l
	${CMAKE_CURRENT_BINARY_DIR}/lexer.cc
)

add_executable(${PROJECT_NAME}
	numlex.cc
	${FLEX_scanner_OUTPUTS}
)

target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_20)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
```
## Грамматики
#### Грамматика регулярных выражений
• Все правильные регулярные выражения над {a, b, c} и только их можно построить по следующему правилу:
```
A → A+A|(A)|A.A|A*|a|b|c
```
• Например, построим (c.(a+b)\*.a.b)\*.c.a
<span style="color: red;">A</span> → <span style="color: red;">A.</span>A → <span style="color: red;">A.</span>A.A → (<span style="color: red;">A</span>)\*A.A → (A.<span style="color: red;">A</span>)\*A.A → (A.A.<span style="color: red;">A</span>)\*A.A → (A.<span style="color: red;">A.</span>A.A)\*A.A →
→ (A.(<span style="color: red;">A</span>)\*.A.A)\*A.A → (A.(A+A)\*.A.A)\*.A.A → ... → (c.(a + b)\*.a.b)\*.c.a
• Мы получили <span style="color: blue;">вывод</span>, использующий на каждом шаге одну <span style="color: blue;">продукцию</span>.
#### Грамматика
• По определению грамматика состоит из продукций α → β.
```
A → A+A|(A)|A.A|A*|a|b|c
```
• <span style="color: blue;">Нетерминальные</span> символы в общем случае могут стоять и слева и справа, <span style="color: blue;">терминальные</span> только слева.
• Для языка регулярных выражений над {a, b, c}.
• Терминалы: a, b, c, +, (, ), \*
• Нетерминалы: пока только A
• Обратим внимание: во всех продукциях языка регулярных выражений у нас слева всего один нетерминал.
#### Контекстная свобода и зависимость
• **Контекстно свободной** грамматикой называется такая, которую можно представить так, чтобы слева в каждой продукции был ровно один нетерминал.
• Интуитивно означает, что "if везде if".
• Любой регулярный язык тривиально является контекстно свободным.
• Язык L<sub>4</sub> = {a<sup>m</sup>b<sup>n</sup>ca<sup>m</sup>b<sup>n</sup>} не является контекстно свободным.
• Контекстно-свободный язык эквивалентен автомату с магазинной памятью (естественному обобщению обычного автомата).
#### Автоматы с магазинной памятью
• Pushdown automata - это обычный НКА, к которому добавлен стек.
• На рисунке снизу стек изображён синим.
![[7.4.png]]
• Переход делается по входному символу и по стековому символу.
• При этом, при переходе можно запушить или извлечь символ.
• Z/AZ означает "если верхушка Z, то перейти и запушить A".
#### Вывод в грамматике
• Мы говорим, что в данной грамматике выражения бывают выводимые и нет.
```
A → A+A|(A)|A.A|A*|a|b|c
```
• Обратим внимание, что у насетьс две естественные стратегии вывода: брать самый левый или самый правый нетерминал на каждом шаге.
A → A + <span style="color: red;">A</span> → A + A + A → A + A + c → A + b + c → a + b + c
A → <span style="color: red;">A</span> + A → A + A + A → A + A + c → A + b + c → a + b + c
• Это нормально: мы можем задаться левым или правым выводом, и если он единственный, у нас всё хорошо.
• А вот если нет, то грамматика <span style="color: blue;">неоднозначна</span>.
#### Грамматика: неоднозначность
• Можно заметить много проблем у такой грамматики
```
A → A+A|(A)|A.A|A*|a|b|c
```
• Строчка <span style="color: blue;">a + b.c</span> имеет два разных левых вывода.
A → <span style="color: red;">A</span>.A → A + A.A → a + <span style="color: red;">A</span>.A → a + b.<span style="color: red;">A</span> → a + b.c
A → <span style="color: red;">A</span> + A → a + <span style="color: red;">A</span> → a + <span style="color: red;">A</span>.A → a + b.<span style="color: red;">A</span> → a + b.c
• В первом случае она будет означать <span style="color: blue;">a + (b.c)</span>, а во втором случае это будет
<span style="color: blue;">(a + b).c</span>
• Эта неоднозначность очень неприятно выглядит, если взглянуть на получившиеся деревья вывода.
![[7.5.png]]
#### Грамматика: приоритеты операций
• Добавив нетерминалов, мы получаем приоритеты у операций.
• Строчка a + bc имеет, по сути, единственный вывод с точностью до порядка выбора продукций.
• Логично выбрать продукцию либо для самого левого нетерминала, либо для самого правого (leftmost/rightmost).
A → B + A | B
B → C.B | C
C → D*| D
D → (E) | E
E → a | b | c
Заметим также, что эта грамматика факторизована слева, у неё нет продукций вида A → Aα.
#### Таксономия L/R
• Первая буква означает направление
	• L означает слева-направо
	• R означает справа-налево
• Вторая буква означает выбранный нетерминал
	• L означает берём самый левый
	• R означает берём самый правый
• Далее могут следовать скобки, в которых стоит сколько символов предпросматриваем.
• Есть также префиксы. Например, LALR (LA = look ahead).
• Мы можем формулировать вопросы про языки в терминах их классов.
#### Нисходящий парсинг LL(k)
• Нисходящий парсинг контекстно-свободных грамматик состоит в построении деревьев сверху-вниз.
![[7.6.png]]
#### Рекурсивный спуск LL(k)
• Для достаточно хороших грамматик для построения дерева можно использовать метод рекурсивного спуска.
• Для каждого нетерминала X<sub>i</sub> делается отдельная функция X<sub>i</sub>.
```
void Xi() {
	// выбрать продукцию Xi → Y1Y2 ... Yk
	for(i = 1 to k) {
		if(Yi nonterminal) Yi();
		else if(Yi equals current input symbol) advance input
		else ???
	}
}
```
• Первая главная проблема нисходящего разбора: может понадобиться откат, если продукция была выбрана неудачно.
• Рассмотрим грамматику A → cBd; B → ab | a
• Как выводить строчку  cad?
• Первая продукция, очевидно, A → cBd.
![[7.7.png]]
• Вторую продукцию можно выбрать как B → ab.
![[7.8.png]]
• Вторую продукцию можно выбрать как B → a.
• Теперь на третьем шаге всё тоже будет успешно.
![[7.9.png]]
• Но сколько откатов потребует нетривиальная грамматика?
#### Обсуждение: LL(1)
• Можем ли мы построить парсер без откатов?
• Да, но не для всех грамматик.
• Рассмотрим следующую грамматику арифметических выражений:
E → E **+** T | T
T → T \* F | F
F → (E) | **id**
• Понятно ли, почему тут LL(1) не получится?
#### Ограничения на LL(1)
• Можно ли доказать, что язык относится к LL(1), если и только если для двух разных продукций A → α и A → β.
• Не существует терминала a<sub>i</sub> такого, чтобы α и β выводили строки, начинающиеся с a.
• Не более чем одна из α и β выводит пустую строку.
• Если β выводит пустую строку, то α не выводит строк из FOLLOW(A).
![[7.10.png]]
#### Построение FIRST / FOLLOW
E → T E'                FIRST(F) = FIRST(T) = FIRST(E) = {(,id)}
E' → + T E' | _ε_       FIRST(E') = {+, _ε_}
T → F T'                 FIRST(T') = {\*, _ε_}
T' → \* F T' | _ε_        FOLLOW(E) = FOLLOW(E') = {),$}
F → (E) | id            FOLLOW(T) = FOLLOW(T') = {+,),$}
                             FOLLOW(F) = {+, \*, ), $}
#### Таблица для LL(1) разбора
• По множествам FIRST и FOLLOW можно построить таблицу выбора продукций.

|     | id       | +           | *           | (         | )         | $         |
| --- | -------- | ----------- | ----------- | --------- | --------- | --------- |
| E   | E → T E' |             |             | E → T E'  |           |           |
| E'  |          | E' → + T E' |             |           | E' →  _ε_ | E' →  _ε_ |
| T   | T → F T' |             |             | T → F T'  |           |           |
| T'  |          | T' → _ε_    | T' → * F T' |           | T' →  _ε_ | T' →  _ε_ |
| F   | F → id   |             |             | F → ( E ) |           |           |
• Попробуйте прогнать по этой таблице a + b \* c.
#### Dangling else
• Классический пример неоднозначности, не позволяющей построить LL(1) парсер - это dangling else.
```
stmt → if expr then stmt
stmt → if expr then stmt else stmt
stmt → Si
expr → Ei
```
• Тогда в следующей строчке будет непонятно, к какому if отнести else.
```
if E1 then if E2 then S1 else S2
```
• Неоднозначность из грамматики можно убрать, сделав из неё нормальную форму Хомского (CNF).
#### Более совершенные методы
• Рекурсивный спуск LL(k) может потребовать откатов.
• Ошибки в грамматике (неоднозначности, циклы) не видны, пока в них не наступишь в рантайм.
• Нисходящий LL(1) эффективен, но не для всех языков применим.
• Более совершенными являются восходящие методы.
#### Восходящий парсинг LR(k)
• Восходящий разбор основан на двух операциях:
	• shift - сдвинуть текущий элемент входного потока в стек
	• reduce - использовать продукцию, чтобы изменить содержимое стека на терминал слева от продукции
• Главна хитрость этого метода, когда сделать shift и когда reduce.
• Для принятия этого решения, сначала строятся множества FIRST/FOLLOW, потом для них строится LR(0) автомат.
• Всё это технически сложные действия, которые проще автоматизировать.
#### Восходящий парсинг LR
• Восходящий парсинг контекстно-свободных грамматик состоит в построении деревьев снизу-вверх. Также известен как shift-reduce подход.
![[7.11.png]]
#### Обсуждение
• К сожалению, написать руками LR-парсер очень нелегко. Там нужно делать LR(0) автомат и использовать его в парсинге.
• Если мы генерируем лексер из регулярных выражений, можно ли сделать также парсер из правил продукции?
## Практика синтаксического анализа
#### Задача синтаксического анализа
• Основная задача - это построить синтаксическое дерево из потока лексем.
• Мы не хотели бы делать LL(k) из-за бэктрекинга.
• Хочется сделать LR или LALR, но делать их руками может быть крайне неудобно.
• Поэтому мы хотели бы точно также сгенерировать парсер.
#### BISON
• Язык и система генерации синтаксических анализаторов для C++
```bison
%code qualifier { /* C++ code */ }
// определения для bison
%%
grammar rule { /* action */ }
%%
// здесь любой код после правил грамматики
```
• qualifier всегда можно посмотреть в грамматике бизона.
#### BISON: простой пример
• Допустим, нам необходимо проверить серию равенств
```
200 + 2 = 400 - 198; 100 + 1 = 100 - 2;
```
• Для простоты всё, что нужно - это вывести на экран результаты.
• Обратите внимание на (увы, неправильную) явную грамматику для expr:
```bison
expr: NUMBER           { $$ = $1; }
    | expr PLUS expr   { $$ = $1 + $3; }
    | expr MINUS expr  { $$ = $1 - $3; };
```
• Здесь использованы бизоновские сокращения для семантических значений.
• Ясно, что если мы можем в правилах складывать, можно там и AST строить.
Пример:
```bison
/* ------------------------------------------------------------------------- **
 *
 * Source code for MIPT ILab
 * Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
 * Licensed after GNU GPL v3
 *
 * -------------------------------------------------------------------------- **
 *
 * Simple, but incorrect grammar
 * EQL -> EQ; EQL | empty
 * EQ -> E = E
 * E -> number | E + E | E - E
 *
 * -------------------------------------------------------------------------- */

%language "c++"

%skeleton "lalr1.cc"
%defines
%define api.value.type variant
%param {yy::NumDriber* driver}

%code requires
{
#include <iostream>
#include <string>

// forward dec1 of argument to parser
namespace yy { class NumDriver; }
}

%code
{
#include "numdriver.hpp"

namespace yy {

parser::token_type yylex(parser::semantic_type* yyval,
                         NumDriver* driver);
}
}

%token
	EQUAL  "="
	MINUS  "-"
	PLUS   "+"
	SCOLON ";"
	ERR
;

%token <int> NUMBER
%nterm <int> equals
%nterm <int> expr

%left '+' '-'

%start program

%%

program: eqlist
;

eqlist: equals SCOLON eqlist
      | %empty
;

equals: expr EQUAL expr {
		                      $$ = ($1 == $3);
		                      std::cout << "Checking: " << $1 << " vs " << $3
		                                << "; Result: " << $$
		                                << std::endl;
                        }
;

expr: expr PLUS expr    { $$ = $1 + $3; }
    | expr MINUS expr   { $$ = $1 - $3; }
    | NUMBER            { $$ = $1; }
;
```
Драйвер:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Driver for simplest grammar of comparisons. Note plex_->YYText() usage.
//
//---------------------------------------------------------------------------

#pragma once

#include "numgrammar.tab.hh"
#include <FlexLexer.h>

namespace yy {

class NumDriver {
	FlexLexer *plex_;

public:
	NumDriver(FlexLexer *plex) : plex_(plex) {}
	
	parser::token_type yylex(parser::semantic_type *yylval) {
		parser::token_type tt = static_cast<parser::token_type>(plex_->yylex());
		if (tt == yy::parser::token_type::NUMBER)
			yylval->as<int>() = std::stoi(plex_->YYText());
		return tt;
	}
	
	bool parse() {
		parser parser(this);
		bool res = parser.parse();
		return !res;
	}
};

} // namespace yy
```
Лексер:
```flex
%option c++

%{

#include "numgrammar.tab.hh"

%}

WS     [ \t\n\v]+
DIGIT  [0-9]
DIGIT1 [1-9]

%%

{WS}             /* skip blanks and tabs */
"+"              return yy::parser::token_type::PLUS;
"-"              return yy::parser::token_type::MINUS;
"="              return yy::parser::token_type::EQUAL;
";"              return yy::parser::token_type::SCOLON;
{DIGIT1}{DIGIT}* return yy::parser::token_type::NUMBER;
.                return yy::parser::token_type::ERR;

%%

//nothing
```
CMakeLists.txt для этого:
```cmake
#---------------------------------------------------------------------------
#
# Source code for MIPT ILab
# Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
# Licensed after GNU GPL v3
#
#---------------------------------------------------------------------------
#
# cmake for simplest grammar example
# default: cmake -DCMAKE_BUILD_TYPE=Release -DGRAMMAR="numgrammar-sr.y" ..
#     try: cmake -DCMAKE_BUILD_TYPE=Release -DGRAMMAR="numgrammar-sr-fixed.y" ..
#     try: cmake -DCMAKE_BUILD_TYPE=Release -DGRAMMAR="numgrammar.y" ..
#
#---------------------------------------------------------------------------

cmake_minimum_required(VERSION 3.13.4)
project(numgrammar)

find_package(BISON REQUIRED)
find_package(FLEX REQUIRED)

if (NOT DEFINED GRAMMAR)
	set(GRAMMAR "numgrammar-sr.y" CACHE STRING "file with grammar" FORCE)
endif()

flex_target(scanner
	numgrammar.l
	${CMAKE_CURRENT_BINARY_DIR}/lexer.cc
)

bison_target(parser
	${GRAMMAR}
	${CMAKE_CURRENT_BINARY_DIR}/parser.cc
	COMPILE_FLAGS "--defines=${CMAKE_CURRENT_BINARY_DIR}/numgrammar.tab.hh"
)

add_flex_bison_dependency(scanner parser)

add_executable(${PROJECT_NAME}
	driver.cc
	${BISON_parser_OUTPUTS}
	${FLEX_scanner_OUTPUTS}
)

target_compile_features(${PROJECT_NAME} PRIVATE cxx_std_20)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_BINARY_DIR})
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_CURRENT_SOURCE_DIR})
```
Ну и driver.cc:
```cpp
//---------------------------------------------------------------------------
//
// Source code for MIPT ILab
// Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
// Licensed after GNU GPL v3
//
//---------------------------------------------------------------------------
//
// Driver program for simplest grammar of comparisons
//
//---------------------------------------------------------------------------

#include <iostream>

#include <numdriver.hpp>

// here we can return non-zero if lexing is not done inspite of EOF detected
int yyFlexLexer::yywrap() { return 1; }

int main() {
	FlexLexer *lexer = new yyFlexLexer;
	yy::NumDriver driver(lexer);
	driver.parse();
}
```
В numgrammar-sr.y дописано на BISON:
```bison
namespace yy {

parser::token_type yylex(parser::semantic_type* yylval,
                         NumDriver* driver)
{
	return driver->yylex(yylval);
}

void parser::error(const std::string&) {}
}
```
В приведённом выше примере грамматики 4 shift/reduce ошибки. Вот fixed-версия:
```bison
/* ------------------------------------------------------------------------- **
 *
 * Source code for MIPT ILab
 * Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
 * Licensed after GNU GPL v3
 *
 * -------------------------------------------------------------------------- **
 *
 * Grammar with shift/reduce, but correct
 * EQL -> EQ; EQL | empty
 * EQ -> E = E
 * E -> number | E + number | E - number
 *
 * -------------------------------------------------------------------------- */
```
Есть ещё такой пример от замечательного лектора:
```bison
/* ------------------------------------------------------------------------- **
 *
 * Source code for MIPT ILab
 * Slides: https://sourceforge.net/projects/cpp-lects-rus/files/cpp-graduate/
 * Licensed after GNU GPL v3
 *
 * -------------------------------------------------------------------------- **
 *
 * Correct grammar withoud shift/reduce
 * EQL -> EQ; EQL | empty
 * EQ -> E = E
 * E -> number A
 * A -> + number A | - number A | empty
 *
 * -------------------------------------------------------------------------- */

/* уже написанные части */

expr: NUMBER arith        {$$ = $1 + $2; }
;

arith: PLUS NUMBER arith  { $$ = $2 + $3; }
     | MINUS NUMBER arith { $$ = -$2 + $3; }
     | %empty             { $$ = 0; }
;

%%
```
Дальше, наконец, показана информация со слайдов.
#### Токены и нетерминалы
```bison
%language "c++"

%token
	EQUAL  "="
	MINUS  "-"
	PLUS   "+"
	SCOLON ";"
	ERR
;

%token <int> NUMBER
%nterm <int> equals
%nterm <int> expr

%left '+' '-'

%start program
```
• Здесь пропущены технические куски кода с заголовочными файлами и всем прочим.
• Токен без типа - это просто токен.
• Токен с типом несёт в себе семантическое значение.
• Операторы сделаны левоассоциативными.
• Задана стартовая точка.
#### Грамматические правила
```bison
program: eqlist;

eqlist: equals SCOLON eqlist
      | equals SCOLON
;

equals: expr EQUAL expr { $$ = ($1 == $3); }
;

expr: NUMBER            { $$ = $1; }
		| expr PLUS expr    { $$ = $1 + $3; }
		| expr MINUS expr   { $$ = $1 - $3; }
;
```
• Видно, что почти точно повторяются продукции грамматики.
```
EqList → Equals;EqList | Equals
Equals → Expr = Expr
Expr → NUM | Expr + Expr | Expr - Expr
```
• Текущее семантическое значение - это \$\$.
• Символами \$i обозначаются значения параметров продукций (нумерация с единицы).
#### Взаимодействие с лексером
• Поскольку лексер должен распознавать те же токены, которые являются основой для парсера, в bison есть возможность сгенерировать заголовочный файл, где все они перечислены.
• Это оказывается очень удобно. Достаточно просто включить его в лексер.
#### Код лексера
```flex
%option c++

%{
#include "numgrammar.tab.hh"
%}

WS     [ \t\n\v]+
DIGIT  [0-9]
DIGIT1 [1-9]

%%
{WS}             /* skip blanks and tabs */
"+"              return yy::parser::token_type::PLUS;
"-"              return yy::parser::token_type::MINUS;
"="              return yy::parser::token_type::EQUAL;
";"              return yy::parser::token_type::SCOLON;
{DIGIT1}{DIGIT}* return yy::parser::token_type::NUMBER;
.                return yy::parser::token_type::ERR;
%%
```
#### Драйвер: связующее звено
• Поскольку и лексер и парсер в данном случае являются классами, драйвер должен соединять их воедино:
```cpp
parser::token_type yylex(parser::semantic_type* yylval) {
	
	// вызвали лексер
	auto tt = static_cast<parser::token_type>(ples_->yylex());
	
	// установили семантическое значение
	if(tt == yy::parser::token_type::NUMBER)
		yylval->as<int>() = std::stoi(plex_->YYText());
		
	// вернули тип токена
	return tt;
}
```
#### Ошибки shift и reduce
• Два вида ошибок
	• менее мрачные shift/reduce
	• более мрачные reduce/reduce
• Бизон предупреждает о тех и о других и лучше их убирать из грамматики.
• Их удобно посмотреть на демо парсера для арифметических выражений.
Пример еще из advgrammar:
```bison
%nterm <std::vector<int>> expr
%nterm <std::pair<std::vector<int>, std::vector<int>>> equals
%nterm <std::vector<std::pair<std::vector<int>, std::vector<int>>>> eqlist
```
Ну и, соответственно, там не считаются на месте, а через \$\$.push_back всё складывается.
#### BISON: семантические значения
• Допустим, мы не хотим в узлах сразу считать, а хотим накапливать вектора.
• Нет ничего проще. Благодаря объявлению
```bison
#define api.value.type varian
```
• Мы можем иметь сколь угодно сложные семантические типы. В частности
```bison
%nterm <vector<int>> expr
%nterm <pair<vector<int>, vector<int>>> equals
%nterm <vector<pair<vector<int>, vector<int>>>> eqlist
```
• И это тоже будет работать.
#### Обсуждение
• Что насчёт обработки ошибок?
• В системе bison она концептуально не слишком сложна, но требует много технической работы (как, впрочем, и в любом компиляторе).
#### Простая архитектура компилятора
![[7.12.png]]
Дальше показывает код INode.hpp
```cpp
//---------------------------------------------------------------------------
//
// INode -- main node interfaces, operations enum and ctor functions for nodes
//
//---------------------------------------------------------------------------

#pragma once

// node interface
struct INode {
	virtual int calc() = 0;
	virtual void dump() const = 0;
	virtual ~INode() {}
};

// scope interface
struct IScope : public INode {
	virtual IScope *push() = 0;
	virtual IScope *resetScope() const = 0;
	virtual void addBranch(INode *branch) = 0;
	virtual INode *access(std::string const &var_name) = 0;
	virtual INode *visible(std::string const &var_name) = 0;
};

// operations
enum class Ops {
	Plus,
	Minus,
	Assign,
	Greater,
	Less,
	GreaterEq,
	LessEq,
	StdOut,
	StdIn,
	Equal,
	NotEqual,
	Div,
	Mul,
	Mod,
	Not,
	And,
	Or
};

// ctor functions
INode *make_value(int);
INode *make_op(INode *l, Ops o, INode *r);
INode *make_while(INode *o, INode *s);
INode *make_if(INode *o, INode *s);
IScope *create_scope();
```
Теперь Node.hpp:
```cpp
//---------------------------------------------------------------------------
//
// Node.hpp -- concrete node types
//
//---------------------------------------------------------------------------

#pragma once

#include <iostream>
#include <map>
#include <string>
#include <typeinfo>
#include <vector>

#include "INode.hpp"

using RType = int;

// Integer value
class Value final : public INode {
	int val;

// INode interface
public:
	RType calc() override;
	void dump() const override;

public:
	Value(int v) : val(v) {}
};

// Declaration
class Decl final : public INode {
	int val;
	
// INode interface
public:
	RType calc() override;
	void dump() const override;
	
public:
	Decl() = default;
	void SetValue(int val);
};

// Scope
class Scope final : public IScope {
	std::vector<INode *> branches;
	IScope *prev_scope;

// INode interface
public:
	IScope *push() { return new Scope(this); }
	IScope *resetScope() const override;
	void addBranch(INode *branch) override;
	INode *access(std::string const &var_name);
	INode *visible(std::string const &var_name);
	
public:
	Scope(Scope *prev) : prev_scope(prev) {}
	~Scope();
};

// Operand
class Op final : public INode {
	INode *right;
	INode *left;
	Ops op;

// INode interface
public:
	RType calc() override;
	void dump() const override;
	
public:
	Op(INode *l, Ops o, INode *r) : left(l), right(r), op(o) {}
	~Op();
};

// Whule loop node
class While final : public INode {
	INode *op = nullptr;
	INode *scope = nullptr;
	
// INode interface
public:
	RType calc() override;
	void dump() const override;
	
public:
	While(INode *o, INode *s) : op(o), scope(s) {}
	~While();
};

// If node
class If final : public INode {
	INode *op;
	INode *scope;
	
// INode interface
public:
	RType calc() override;
	void dump() const override;
	
public:
	If(INode *o, INode *s) : op(o), scope(s) {}
	~If();
};
```
Node.cpp:
```cpp
//---------------------------------------------------------------------------
//
// Node.cpp -- concrete node types implementation
//
//---------------------------------------------------------------------------

#include "Node.hpp"
#include "Symtab.hpp"

Symtab globalTable;

INode *make_value(int v) { return new Value{v}; }
INode *make_op(INode *l, Ops o, INode *r) { return new Op{l, o, r}; }
INode *make_while(INode *o, INode *s) { return new While{o, s}; }
INode *make_if(INode *o, INode *s) { return new If(o, s); }
IScope *create_scope() { return new Scope(nullptr); }

// NUMBER
RType Value::calc() { return val; }
void Value::dump() const { std::cout << "Node Value: " << val << std::endl; }

RType Decl::calc() { return val; }
void Decl::dump() const { std::cout << "Node Decl: " << val << std::endl; }
void Decl::SetValue(int Val) { val = Val; }

// SCOPE
// ...
```
parser.hpp:
```cpp
//---------------------------------------------------------------------------
//
// parser.hpp -- concrete node types implementation
//
//---------------------------------------------------------------------------

#pragma once

#include "INode.hpp"
#include <cstdarg>
#include <cstring>
#include <sstream>
#include <string>

struct yyRet {
	std::string name;
	int value;
	int linePos;
	int inLinePos;
	INode *treeNode;
	Ops op;
};

#define YYSTYPE yyRet

#include "compiler.cpp.h"

extern FILE *yyin;

int yylex();
int yyerror(char const *);
void PrintError(char const *s, ...);
void BeginToken(char *, int *);

// line number diagnostics
extern int yylineno;

// for string position diagnostics
static int yyinlinePos;
```
driver.cpp:
```cpp
//---------------------------------------------------------------------------
//
// driver.cpp -- main entry point
//
//---------------------------------------------------------------------------

#include "parser.hpp"

IScope *currentScope = nullptr;

static int currentinlinePos = 0;

int main(int argc, char *argv[]) {
	FILE *f = fopen(argv[1], "r");
	if(f <= 0) {
		perror("Cannot open file");
		return 1;
	}
	yyin = f;
	currentScope = create_scope();
	yyparse();
	fclose(f);
	delete currentScope;
	
	return 0;
}

void PrintError(char const *errorstring, ...) {
	static cahr errmsg[10000];
	va_list args;
	
	bool isNotNullPar = false;
	for(int i = 0; i < strlen(errorstring) - 1; ++i) {
		if(errorstring[i + 1] == '%' && errorstring[i] != '\\') {
			isNotNullPar = true;
			break;
		}
		// etc
	}
}
// etc
```
#### Литература
• [CC11] ISO/IEC 14882 - "Information technology - Programming languages - C++", 2011
• [BS] Bjarne Stroustrup - The C++ Programming Language (4th Edition), 2013
• [DB] Alfred Aho, Jeffrey Ullman - Compilers: Principles, Techniques, and Tools (2nd Edition), 2006
• [Aut] Jeffrey Ullman - Automata Theory online course, online.stanford.edu.
• [Comp] Alex Aiken - Compilers online course, online.stanford.edu
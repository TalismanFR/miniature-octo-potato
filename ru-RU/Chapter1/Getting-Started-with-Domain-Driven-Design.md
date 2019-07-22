Глава 1. Начало работы с DDD.
==
Так откуда весь шум? Если вы уже читали книги на эту тему за авторствами Vaughn Vernon и Eric Evans,
вы, вероятно, знакомы с тем, что мы собираемся рассказать, поскольку мы в значительной степени заимствуем их определения 
и объяснения. DDD - это подход, который позволяет нам преуспеть в понимании и построении моделей программных продуктов.
Он предоставляет нам инструменты стратегического и тактического моделирования для разработки высококачественного программного
обеспечения, соотвествующего нашим бизнес-целям.
>Основная цель этой книги - показать примеры тактических шаблонов DDD используя PHP. Если вы хотите
больше узнать о стратегических шаблонах и о DDD впринцепе, вам следует прочитать книги за авторством 
Vaughn Vernon и Eric Evans.

Что очень важно, DDD не связан с технологиями. Вместо этого речь идет о развитии знаний о бизнесе и испльзовании технологий
для обеспечения ценности. Только когда вы сможете понять бизнесс, в котором работает ваша компания
, вы сможете участвовать в процессе создания модели программного обеспечения пораждая Единый Язык (Ubiquitous Language).

## Почему DDD так важен
Програмнное обеспечение это не только код. Код редко является конечной целью вашей работы. Код это только средства решения
бизнесс-задач. Так почему код должен быть на языке отличном от языка бизнесса? DDD подчеркивает что код и бизнесс должны 
говорить на одном языке. Когда барьер преодален, нет необходимости в переводе или утомительной синхронизации, информация не
потеряется. Каждый участник влияет на Бизнесс-Домен, не только разработчики.Получающееся программное обеспечение - 
единственная правда для общего языка.

DDD также обеспечивает основу для стратегического и тактического моделирования. Стратегическое проектирование позволяет точно 
определить наиболее важные области разрабоки на основе бизнес-ценностей. Тактическо
моделироване нужно для построения работающей Доменой Модели с использование проверянных в бою
строительных блоков и шаблонов.

## Три столпа DDD
Domain-Driven Design это подход к разработке программного обеспечения, который сфокусирован на трёх основных принципах:
1. **Единный Язык (Ubiquitous Language)**: Эксперты предметной области (Domain Experts) и разработчики программного обеспечения работают вместе,
чтобы создать общий язык для разрабатываемых бизнесс-сфер. Тут не может быть противопоставления, это единная команда. 
Разработка программного обеспечения это инвестиция в бизнес, а не просто статья расходов. Усилия, прилогаемые к созданию 
Единого Языка (Ubiquitous Language), помогают распространить глубокое понимание Домена (Domain) среди всех членов команды.
2. **Стратегическое моделирование (Strategic Design)**: DDD направлен на стратегию управления бизнесом, а не только на технические
аспекты его работы. Это помогает определить внутренние отношения и системы обратной связи раннего предупреждения.
С технической стороны, стратегический дизайн защищает каждую бизнес-услугу, обеспечивая мотивацию для достижения 
сервис-ориентированной архитектуры.
3. **Тактическое моделирование (Tactical Design)**: DDD предоставляет инструменты и строительные блоки для итеративной разработки
программного обеспечения. Инструменты тактического моделирования позволяют создать программное
обеспечение, которое не только предельно корректно, но и является тестируемым и менее подверженым ошибкам.
### Единый язык (Ubiquitous Language)
Наряду с темой главы 12 "Интеграция Ограниченного Контекста (Bounded Contexts)", Единый Язык (Ubiquitous Language) является одной из главных сильных сторон DDD.
>Пока будем рассматривать Ограниченные Контекст как концептуальную границу вокруг системы. Единый язык внутри этой границы
имеет особое концептуальное значение. Концепции и определения вне этой границы могут иметь иные значения.

Итак, как найти, изучить и запечатлеть этот особый язык, следующие подсказки помогут вам в этом:
+ Определить ключевые бизнесс-процессы, их входы и выходы.
+ Создайть глосарий терминов и определений.
+ Определить важные концепции программного обеспечения с хорошей документацией.
+ Обменивайтесь этими знаний со всеми членами команды (Разработчиками и Экспертами Предметной Области)
С распростронением DDD, появились новые методы для улучшающие и упрощающие процесс создания
Единого языка (Ubiquitous Language). Самый важный и наболее часто используемы это Событийный штурм (Event Storming).

#### Событий Штурм (Event Storming)
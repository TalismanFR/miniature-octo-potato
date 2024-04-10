Глава 4. Сущности (Entities)
==

--------

<br>


Мы говорили о преимуществах попытки сначала смоделировать все в Домене как Объект Значения. Но при моделировании Домена, вероятно, возникнут ситуации, когда вы обнаружите, что какая-то концепция во повсеместном языке требует потока идентичности.

<br>

## Введение

<br>
Четкие примеры объектов, требующих идентификатора:

- Человек. У человека всегда есть удостоверение личности, и оно всегда одинаково с точки зрения его имени или удостоверения личности.

- Заказ в системе электронной коммерции. В таком контексте каждый новый созданный порядок имеет свою собственную идентичность, и она одинакова с течением времени.

<br>
У этих понятий есть идентичность, которая неизменна со временем. Независимо от того, сколько раз данные в концепциях меняются, их идентичности остаются прежними. 
Именно это делает их сущностями, а не Объектами Значений. С точки зрения реализации PHP, они были бы простыми классами.
Например, рассмотрим следующее в случае человека:
<br>
<br>

```php
namespace Ddd\Identity\Domain\Model;

class Person
{
    private $identificationNumber;
    private $firstName;
    private $lastName;
    public function __construct(
        $anIdentificationNumber, $aFirstName, $aLastName
    ) {
        $this->identificationNumber = $anIdentificationNumber;
        $this->firstName = $aFirstName;
        $this->lastName = $aLastName;
    }
    public function identificationNumber()
    {
        return $this->identificationNumber;
    }
    public function firstName()
    {
        return $this->firstName;
    }
    public function lastName()
    {
        return $this->lastName;
    }
}
}
```
<br>

Или рассмотрим следующее в случае заказа:

<br>

```php
namespace Ddd\Billing\Domain\Model\Order;

class Order
{
    private $id;
    private $amount;
    private $firstName;
    private $lastName;
    public function __construct(
        $anId, Amount $amount, $aFirstName, $aLastName
    ) {
        $this->id = $anId;
        $this->amount = $amount;
        $this->firstName = $aFirstName;
        $this->lastName = $aLastName;
    }
    public function id()
    {
        return $this->id;
    }
    public function firstName()
    {
        return $this->firstName;
    }
    public function lastName()
    {
        return $this->lastName;
    }
}
```


<br>
<br>


## Объекты Vs. Примитивные типы
<br>
Большую часть времени Идентификатор Сущности представляется как примитивный тип - обычно строка или целое число. Но использование объекта Value для его представления имеет больше преимуществ:

- Объекты Value являются неизменяемыми, поэтому их нельзя изменять.

- Объекты-значения - это сложные типы, которые могут иметь пользовательское поведение, то, что примитивные типы не могут иметь. Возьмем в качестве примера операцию равенства. 
С помощью Value Objects операции равенства могут моделироваться и инкапсулироваться в свои собственные классы, делая концепции переходящими от неявных к явным.

<br>

Рассмотрим возможную реализацию для OrderId, идентификатора заказа, который превратился в объект Value:

<br>

```php
namespace Ddd\Billing\Domain\Model;

class OrderId
{
    private $id;
    public function __construct($anId)
    {
        $this->id = $anId;
    }
    public function id()
    {
        return $this->id;
    }
    public function equalsTo(OrderId $anOrderId)
    {
        return $anOrderId->id === $this->id;
    }
}
```
<br>


Существует несколько реализаций, которые можно использовать для реализации идентификатора заказа. Приведенный выше пример довольно прост. 
Как описано в главе 3, «Объекты значений», можно сделать метод __construct () частным и использовать статические фабричные методы для создания новых экземпляров. 
Поговорите со своей командой, проведите эксперимент и договоритесь. Поскольку идентификаторы сущностей не являются сложными Объектами Значений, мы рекомендуем вам здесь не беспокоиться слишком много.
<br>

Возвращаясь к Order, пришло время обновить ссылки на OrderId:

<br>

```php
class Order
{
    private $id;
    private $amount;
    private $firstName;
    private $lastName;
    public function __construct(
        OrderId $anOrderId, Amount $amount, $aFirstName, $aLastName
    ) {
        $this->id = $anOrderId;
        $this->amount = $amount;
        $this->firstName = $aFirstName;
        $this->lastName = $aLastName;
    }
    public function id()
    {
        return $this->id;
    }
    public function firstName()
    {
        return $this->firstName;
    }
    public function lastName()
    {
        return $this->lastName;
    }
    public function amount()
    {
        return $this->amount;
    }
}
```

- Наша сущность имеет идентификатор, смоделированный с использованием Value Object.

Рассмотрим различные способы создания идентификатора заказа.

<br>
<br>

## Операция идентификации

<br>
Как было указано ранее, идентификатор сущности определяет его. Таким образом, обработка этого является важным аспектом Сущности. 
Обычно существует четыре способа определения Identity of an Entity: механизм персистентности предоставляет Identity, клиент предоставляет Identity,
само приложение предоставляет Identity или другой ограниченный контекст предоставляет Identity.

<br>
<br>

### Механизм хранения (БД) генерирует Идентификатор
<br>
Обычно самый простой способ генерации Identity - делегировать его механизму персистентности, потому что подавляющее большинство механизмов персистентности 
поддерживают некую генерацию Identity - как атрибут `AUTO_INCREMENT` MySQL или последовательности Postgres и Oracle. 
Это, хотя и просто, имеет главный недостаток: мы не будем иметь Идентичность Сущности, пока мы не сохраним ее. Таким образом, в некоторой степени, если мы идем с Mechanism Generated Identity, 
мы свяжем операцию Identity с базовым хранилищем персистентности:
<br>
<br>

```sql
CREATE TABLE `orders` (
 `id` int(11) NOT NULL auto_increment,
 `amount` decimal (10,5) NOT NULL,
 `first_name` varchar(100) NOT NULL,
 `last_name` varchar(100) NOT NULL,
 PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;
```
<br>

И тогда мы могли бы рассмотреть этот код:

<br>

```php
namespace Ddd\Identity\Domain\Model;

class Person
{
    private $identificationNumber;
    private $firstName;
    private $lastName;
    public function __construct(
        $anIdentificationNumber, $aFirstName, $aLastName
    ) {
        $this->identificationNumber = $anIdentificationNumber;
        $this->firstName = $aFirstName;
        $this->lastName = $aLastName;
    }
    public function identificationNumber()
    {
        return $this->identificationNumber;
    }
    public function firstName()
    {
        return $this->firstName;
    }
    public function lastName()
    {
        return $this->lastName;
    }
}
```
<br>

Если вы когда-либо пытались создать собственный ORM, вы уже испытали эту ситуацию.

Каков подход к созданию нового Person? Если база данных собирается создать идентификатор, нужно ли передавать его в конструктор? 
Когда и где магия, которая обновит Person? Что произойдет, если мы в конечном итоге не будем сохранять Сущность?

<br>
<br>

#### Суррогатная идентичность
<br>
Иногда при использовании ORM для отображения сущностей в хранилище сохраняемости накладываются некоторые ограничения - 
например, доктрина требует целочисленное поле, если используется стратегия генератора IDENTITY. Это может привести к конфликту с моделью домена, если для нее требуется другой тип удостоверения.
Простейшим способом обработки такой ситуации является использование супертипа слоя, в котором помещается поле Identity, созданное для хранилища сохраняемости:

<br>
<br>

```php
namespace Ddd\Common\Domain\Model;

abstract class IdentifiableDomainObject
{
    private $id;
    protected function id()
    {
        return $this->id;
    }
    protected function setId($anId)
    {
        $this->id = $anId;
    }
}
```

<br>

```php
namespace Acme\Billing\Domain;

use Acme\Common\Domain\IdentifiableDomainObject;

class Order extends IdentifiableDomainObject
{
    private $orderId;
    public function orderId()
    {
        if (null === $this->orderId) {
            $this->orderId = new OrderId($this->id());
        }
        return $this->orderId;
    }
}
```

<br>
<br>

#### Active Record Vs. Data Mapper для Богатых Доменных Моделей
<br>
Каждый проект всегда сталкивается с решением, какая ORM должна использоваться. Есть много хороших ОRM для PHP: Doctrine, Propel, Eloquent, Paris, и много остальных.

Большинство из них являются реализациями Active Record. Реализация Active Record в основном подходит для приложений CRUD, но она не является идеальным решением для моделей Rich Domain по следующим причинам:

- Шаблон активной записи предполагает связь «один к одному» между сущностью и таблицей базы данных. Таким образом, она связывает конструкцию базы данных с конструкцией объектной системы. 
А в модели Rich Domain иногда Сущности строятся с информацией, которая может поступать из разных источников данных.


- Продвинутые вещи, такие как коллекции и наследование, сложно реализовать.


- Большинство реализаций вынуждают использовать посредством наследования некие конструкции, которые навязывают несколько конвенций.
Это может привести к постоянной утечке в модель домена путем соединения модели домена с ORM. Единственная реализация Active Record, которая не навязывает наследование от базового класса, -Castle ActiveRecord от Castle Project, .NET framework. 
В то время как это приводит к некоторой степени разделения между устойчивостью и проблемами Домена в созданных Сущностях, это не отделяет детали устойчивости низкого уровня от конструкции домена высокого уровня.

<br>

Как упоминалось в предыдущей главе, в настоящее время лучшим ORM для PHP является Doctrine, которая является реализацией шаблона Отображения Данных. Data Mapper отделяет проблемы персистентности от проблем домена, что приводит к появлению сущностей, свободных от персистентности. 
Это делает инструмент лучшим для тех, кто хочет построить модель Rich Domain.

<br>
<br>


### Клиент предоставляет Идентификатор
<br>
Иногда при работе с определенными Доменами Идентификация приходят естественным образом, когда клиент использует Модель Домена. Это, скорее всего, идеальный случай, потому что Идентичность может быть легко смоделирована. Рассмотрим рынок продажи книг:
<br>
<br>

```php
namespace Ddd\Catalog\Domain\Model\Book;

class ISBN
{
    private $isbn;
    private function __construct($anIsbn)
    {
        $this->setIsbn($anIsbn);
    }
    private function setIsbn($anIsbn)
    {
        $this->assertIsbnIsValid($anIsbn, 'The ISBN is invalid.');
        $this->isbn = $anIsbn;
    }
    public static function create($anIsbn)
    {
        return new static($anIsbn);
    }
    private function assertIsbnIsValid($anIsbn, $string)
    {
        // ... Validates an ISBN code
    }
}
```
<br>

Согласно Википедии: Международный стандартный номер книги (ISBN) является уникальным числовым коммерческим идентификатором книги. 
ISBN назначается каждому изданию и варианту (за исключением повторной печати) книги. Например, электронная книга, обратная страница и печатное издание одной и той же книги будут иметь разные ISBN. 
Длина ISBN составляет 13 цифр, если она назначена 1 января 2007 года или позднее, и 10 цифр, если она назначена до 2007 года. Метод присвоения ISBN является национальным и варьируется от страны к стране, 
часто в зависимости от того, насколько велика издательская индустрия в стране.

Самое интересное в ISBN то, что он уже определен в Домене, это действительный идентификатор, потому что он уникален, и его можно легко проверить. 
Это хороший пример идентификатора, предоставленного клиентом:

<br>

```php
namespace Ddd\Catalog\Domain\Model\Book;

class Book
{
    private $isbn;
    private $title;
    public function __construct(ISBN $anIsbn, $aTitle)
    {
        $this->isbn = $anIsbn;
        $this->title = $aTitle;
    }
}
```
<br>

Теперь, это простой вопрос использования его:

<br>

```php
$book = new Book(
    ISBN::create('...'),
    'Domain-Driven Design in PHP'
);
```

<br>
<br>


### Приложение создаёт Идентификатор
<br>
Если клиент не может предоставить идентификатор в целом, предпочтительным способом обработки операции идентификации является разрешение приложению генерировать идентификаторы, обычно через UUID. 
Это наш рекомендуемый подход в случае, если у вас нет сценария, как показано в предыдущем разделе.

**Согласно Википедии:**<br>
Цель UUID - дать возможность распределенным системам однозначно идентифицировать информацию без существенной централизованной координации. В этом контексте слово «уникальный» следует понимать как практически уникальный, а не как гарантированный уникальный. 
Поскольку идентификаторы имеют конечный размер, два различных элемента могут совместно использовать один и тот же идентификатор. Это форма хеш-коллизии. 
Размер идентификатора и процесс генерации должны быть выбраны так, чтобы сделать это достаточно маловероятным на практике. 
Любой может создать UUID и использовать его для идентификации чего-либо с разумной уверенностью, что тот же идентификатор никогда не будет непреднамеренно создан кем-либо для идентификации чего-либо другого. 
Поэтому информация, помеченная UUID, может быть впоследствии объединена в единую базу данных без необходимости разрешения конфликтов идентификаторов (ID).

<br>

> Существует несколько библиотек в PHP, которые генерируют UUID, и они могут быть найдены на Packagist: https ://packagist.org/search/?q=uuid.
> Лучшая рекомендация - разработанная Беном Рэмзи: https ://github.com/ramsey/uuid
> потому что имеет много наблюдателей на GitHub и миллионы установок на Packagist.

<br>

Предпочтительным местом для создания Идентификатора будет Репозиторий (подробнее об этом в Главе 10.

Пример Репозитория:

<br>

```php
namespace Ddd\Billing\Domain\Model\Order;

interface OrderRepository
{
    public function nextIdentity();
    public function add(Order $anOrder);
    public function remove(Order $anOrder);
}
```
<br>

При использовании Доктрины необходимо создать пользовательский Репозиторий, реализующий такой интерфейс. Он в основном создает новый Идентификатор и использует EntityManager для сохранения и удаления сущностей. 

Небольшое изменение состоит в том, чтобы поместить реализацию nextIdentity в интерфейс, который станет абстрактным классом:

<br>

```php
namespace Ddd\Billing\Infrastructure\Domain\Model\Order;

use Ddd\Billing\Domain\Model\Order\Order;
use Ddd\Billing\Domain\Model\Order\OrderId;
use Ddd\Billing\Domain\Model\Order\OrderRepository;
use Doctrine\ORM\EntityRepository;

class DoctrineOrderRepository
    extends EntityRepository
    implements OrderRepository
{
    public function nextIdentity()
    {
        return OrderId::create();
    }
    public function add(Order $anOrder)
    {
        $this->getEntityManager()->persist($anOrder);
    }
    public function remove(Order $anOrder)
    {
        $this->getEntityManager()->remove($anOrder);
    }
}
```
<br>

Давайте быстро рассмотрим конечную реализацию объекта OrderId Value Object:
<br>
<br>


```php
namespace Ddd\Billing\Domain\Model\Order;

use Ramsey\Uuid\Uuid;

class OrderId
{
    private $id;
    private function __construct($anId = null)
    {
        $this->id = $id ? :Uuid::uuid4()->toString();
    }
    public static function create($anId = null )
    {
        return new static($anId);
    }
}
```
<br>

Основная проблема, связанная с этим подходом, как будет показано в следующих разделах, заключается в том, насколько просто сохранять объекты, содержащие объекты значений. 
Однако сопоставление встроенных объектов значений, находящихся внутри сущности, может быть сложным в зависимости от ORM.

<br>
<br>

### Другой ограниченный контекст генерирует идентификатор
<br>
Вероятно, это наиболее сложная стратегия создания Identity, поскольку она вынуждает локальную сущность зависеть не только от локальных событий ограниченного контекста, но и от внешних событий ограниченного контекста. 
Так что с точки зрения технического обслуживания стоимость была бы высокой.

Другой ограниченный контекст предоставляет интерфейс для выбора идентификатора из локальной сущности. Некоторые из экспонируемых свойств могут восприниматься как собственные.

Когда необходима синхронизация между сущностями ограниченных контекстов, она обычно может быть достигнута с помощью управляемой событиями архитектуры в каждом из ограниченных контекстов, которые должны быть уведомлены при изменении исходной сущности.

<br>
<br>


## Сохраняющиеся сущности
<br>
В настоящее время, как обсуждалось ранее в главе, лучшим инструментом для сохранения состояния сущности в постоянном хранилище является доктрина ORM. 
Доктрина имеет несколько способов определения метаданных Сущности: с помощью аннотаций в коде Сущности, XML, YAML или простого PHP. В этой главе мы подробно обсудим, почему аннотации не являются лучшими для использования при сопоставлении сущностей.
<br>
<br>

### Настройка доктрины
<br>
Прежде всего, мы должны установить Доктрину через Composer. В корневой папке проекта должна быть выполнена следующая команда:
<br>
<br>

> php composer.phar require "doctrine/orm=^2.5"

<br>

Затем эти строки позволяют настроить доктрину:
<br>
<br>

```php
require_once '/path/to/vendor/autoload.php';

use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\EntityManager;

$paths = ['/path/to/entity-files'];
$isDevMode = false;

// the connection configuration
$dbParams = [
    'driver' => 'pdo_mysql',
    'user' => 'the_database_username',
    'password' => 'the_database_password',
    'dbname' => 'the_database_name'
];

$config = Setup::createAnnotationMetadataConfiguration($paths, $isDevMode);
$entityManager = EntityManager::create($dbParams, $config);
```

<br>
<br>

### Маппинг Entities
<br>
По умолчанию в документации доктрины представлены примеры кода с использованием аннотаций. Итак, мы начинаем пример кода, используя аннотации и обсуждая, почему их следует избегать, когда это возможно.
Для этого мы вернем класс Order, обсуждавшийся ранее в этой главе.
<br>
<br>
<br>

#### Маппинг Entities с помощью аннотированного кода
<br>
Когда была выпущена доктрина, броским способом отображения объектов в примерах кода было использование аннотаций.

**Что такое аннотация?**<br>
Аннотация - это особая форма метаданных. В PHP он помещен в комментарии к исходному коду. Например, PHPDocumentor использует аннотации для построения информации API, а PHPUnit использует некоторые аннотации для указания поставщиков данных или для предоставления ожиданий относительно исключений, порождаемых фрагментом кода:
<br>
<br>

```php
class SumTest extends PHPUnit_Framework_TestCase
{
    /** @dataProvider aMethodName */
    public function testAddition() {
		//...
    }
}
```
<br>

Чтобы сопоставить объект Order с хранилищем, исходный код Order должен быть изменен для добавления аннотаций доктрины:
<br>
<br>

```php
use Doctrine\ORM\Mapping\Entity;
use Doctrine\ORM\Mapping\Id;
use Doctrine\ORM\Mapping\GeneratedValue;
use Doctrine\ORM\Mapping\Column;

/** @Entity */
class Order {
    
    /** @Id @GeneratedValue(strategy="AUTO") */
    private $id;
    
    /** @Column(type="decimal", precision="10", scale="5") */
    private $amount;
    
    /** @Column(type="string") */
    private $firstName;
    
    /** @Column(type="string") */
    private $lastName;
    
    public function __construct(
        Amount $anAmount,
               $aFirstName,
               $aLastName
    ) {
        $this->amount = $anAmount;
        $this->firstName = $aFirstName;
        $this->lastName = $aLastName;
    }
    public function id()
    {
        return $this->id;
    }
    public function firstName()
    {
        return $this->firstName;
    }
    public function lastName()
    {
        return $this->lastName;
    }
    public function amount()
    {
        return $this->amount;
    }
}

```
<br>

Затем, чтобы сохранить Сущность в постоянном хранилище, так же легко сделать следующее:
<br>
<br>

```php
$order = new Order(
    new Amount(15, Currency::EUR()),
    'AFirstName',
    'ALastName'
);
$entityManager->persist($order);
$entityManager->flush();

```
<br>

На первый взгляд, этот код выглядит просто, и это может быть простым способом указать информацию сопоставления. Но это имеет определенную цену. Что странного в окончательном коде?

Во-первых, проблемы домена смешиваются с проблемами инфраструктуры. Order является концепцией домена, тогда как Table, Column и т. д. являются проблемами инфраструктуры.

В результате эта Сущность тесно связана с информацией отображения, указанной аннотациями в исходном коде. 
Если бы Сущность требовалось сохранить с помощью другого менеджера Сущности и с другими метаданными сопоставления, это было бы невозможно.

Аннотации, как правило, приводят к побочным эффектам и плотной связи, поэтому их лучше не использовать.

Так какой лучший способ указать информацию о сопоставлении? Лучший способ - это способ, позволяющий отделить информацию о сопоставлении от самой сущности. 
Это может быть достигнуто с помощью сопоставления XML, сопоставления YAML или PHP. В этой книге мы рассмотрим сопоставление XML.

<br>
<br>

#### Маппинг Entities с помощью XML
<br>
Чтобы отобразить объект Order с помощью сопоставления XML, код настройки доктрины должен быть слегка изменен:
<br>
<br>

```php

require_once '/path/to/vendor/autoload.php';

use Doctrine\ORM\Tools\Setup;
use Doctrine\ORM\EntityManager;

$paths = ['/path/to/mapping-files'];
$isDevMode = false;

// the connection configuration
$dbParams = [
    'driver' => 'pdo_mysql',
    'user' => 'the_database_username',
    'password' => 'the_database_password',
    'dbname' => 'the_database_name',
];

$config = Setup::createXMLMetadataConfiguration($paths, $isDevMode);
$entityManager = EntityManager::create($dbParams, $config);

```
<br>

Файл сопоставления должен быть создан по пути, по которому доктрина будет выполнять поиск файлов сопоставления, 
а файлы сопоставления должны быть названы в честь полного имени класса, заменяя обратную косую черту \ точками.
<br>

Рассмотрим следующее:<br>
`Acme\Billing\Domain\Model\Order`

файл сопоставления будет иметь имя:
  `Acme.Billing.Domain.Model.Order.dcm.xml`

<br>
Кроме того, удобно, чтобы все файлы сопоставления использовали специальную XML-схему, созданную специально для указания информации сопоставления:<br>
https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd

<br>
<br>
<br>

#### Сопоставление Идентификатора Сущности
<br>
Наш идентификатор, OrderId, является объектом значения. Как видно из предыдущей главы, существуют различные подходы к отображению объекта Value Object с использованием доктрины, встраиваемых объектов и пользовательских типов.
Если в качестве идентификаторов используются объекты значений, лучшим вариантом является использование пользовательских типов.

Интересной новой особенностью Doctrine 2.5 является то, что теперь можно использовать Объекты в качестве идентификаторов для Сущностей, если они реализуют магический метод __toString (). 
Таким образом, мы можем добавить __toString () к нашим объектам Identity Value Object и использовать их в наших сопоставлениях:
<br>
<br>

```php
namespace Ddd\Billing\Domain\Model\Order;

use Ramsey\Uuid\Uuid;

class OrderId
{
// ...
    public function __toString()
    {
        return $this->id;
    }
}

```
<br>

Проверьте реализацию пользовательских типов доктрины. Они наследуют GuidType, поэтому их внутреннее представление будет UUID. 
Необходимо указать собственный перевод базы данных. Затем необходимо зарегистрировать пользовательские типы, прежде чем использовать их. 
Если требуется справка по этим шагам, то [Custom Mapping Types](https://www.doctrine-project.org/projects/doctrine-orm/en/3.1/cookbook/custom-mapping-types.html) являются хорошей ссылкой.
<br>
<br>

```php
use Doctrine\DBAL\Platforms\AbstractPlatform;
use Doctrine\DBAL\Types\GuidType;

class DoctrineOrderId extends GuidType
{
    public function getName()
    {
        return 'OrderId';
    }

    public function convertToDatabaseValue($value, AbstractPlatform $platform)
    {
        return $value->id();
    }

    public function convertToPHPValue($value, AbstractPlatform $platform)
    {
        return new OrderId($value);
    }
}

```
<br>

Наконец, мы настроим регистрацию пользовательских типов. Опять же, мы должны обновить начальную загрузку:
<br>
<br>

```php
require_once '/path/to/vendor/autoload.php';
// ...

\Doctrine\DBAL\Types\Type::addType(
    'OrderId',
    'Ddd\Billing\Infrastructure\Domain\Model\DoctrineOrderId'
);

$config = Setup::createXMLMetadataConfiguration($paths, $isDevMode);
$entityManager = EntityManager::create($dbParams, $config);

```
<br>
<br>
<br>


#### Окончательный файл сопоставления
<br>
Со всеми изменениями, мы наконец-то готовы, так что давайте посмотрим на окончательный файл сопоставления.
Наиболее интересной детализацией является проверка того, как идентификатор сопоставляется с нашим определенным пользовательским типом для OrderId:
<br>
<br>

```xml
<?xml version="1.0" encoding="UTF-8"?>
<doctrine-mapping
        xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
        xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
        xsi:schemaLocation="
http://doctrine-project.org/schemas/orm/doctrine-mapping
https://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">
    <entity
            name="Ddd\Billing\Domain\Model\Order"
            table="orders">
        <id name="id" column="id" type="OrderId" />
        <field
                name="amount"
                type="decimal"
                nullable="false"
                scale="10"
                precision="5"
        />
        <field
                name="firstName"
                type="string"
                nullable="false"
        />
        <field
                name="lastName"
                type="string"
                nullable="false"
        />
    </entity>
</doctrine-mapping>

```
<br>
<br>


## Тестирование Entities
<br>
Относительно легко протестировать Сущности, просто потому, что они являются простыми старыми PHP-классами с действиями, производными от концепции Домена, которую они представляют. 
В центре внимания теста должны быть инварианты, которые защищает Сущность, потому что поведение на Сущностях, вероятно, будет смоделировано вокруг этих инвариантов.

Например, для простоты предположим, что для блога необходима модель домена. Один из возможных вариантов мог бы заключаться в следующем:
<br>
<br>

```php
class Post
{
    private $title;
    private $content;
    private $status;
    private $createdAt;
    private $publishedAt;
    
    public function __construct($aContent, $title)
    {
        $this->setContent($aContent);
        $this->setTitle($title);
        $this->unpublish();
        $this->createdAt(new DateTimeImmutable());
    }
    
    private function setContent($aContent)
    {
        $this->assertNotEmpty($aContent);
        $this->content = $aContent;
    }
    
    private function setTitle($aPostTitle)
    {
        $this->assertNotEmpty($aPostTitle);
        $this->title = $aPostTitle;
    }
    
    private function setStatus(Status $aPostStatus)
    {
        $this->assertIsAValidPostStatus($aPostStatus);
        $this->status = $aPostStatus;
    }
    
    private function createdAt(DateTimeImmutable $aDate)
    {
        $this->assertIsAValidDate($aDate);
        $this->createdAt = $aDate;
    }
    
    private function publishedAt(DateTimeImmutable $aDate)
    {
        $this->assertIsAValidDate($aDate);
        $this->publishedAt = $aDate;
    }
    
    public function publish()
    {
        $this->setStatus(Status::published());
        $this->publishedAt(new DateTimeImmutable());
    }
    
    public function unpublish()
    {
        $this->setStatus(Status::draft());
        $this->publishedAt = null ;
    }
    
    public function isPublished()
    {
        return $this->status->equalsTo(Status::published());
    }

    public function publicationDate()
    {
        return $this->publishedAt;
    }
}

```
<br>

```php
class Status
{
    const PUBLISHED = 10;
    const DRAFT = 20;
    private $status;
    public static function published()
    {
        return new self(self::PUBLISHED);
    }
    public static function draft()
    {
        return new self(self::DRAFT);
    }
    private function __construct($aStatus)
    {
        $this->status = $aStatus;
    }
    public function equalsTo(self $aStatus)
    {
        return $this->status === $aStatus->status;
    }
}

```

<br>

Чтобы проверить эту модель домена, мы должны убедиться, что тест охватывает все Post инварианты:
<br>
<br>

```php
class PostTest extends PHPUnit_Framework_TestCase
{
    /** @test */
    public function aNewPostIsNotPublishedByDefault()
    {
        $aPost = new Post(
            'A Post Content',
            'A Post Title'
        );
        $this->assertFalse(
            $aPost->isPublished()
        );
        $this->assertNull(
            $aPost->publicationDate()
        );
    }
    
    /** @test */
    public function aPostCanBePublishedWithAPublicationDate()
    {
        $aPost = new Post(
            'A Post Content',
            'A Post Title'
        );
        $aPost->publish();
        $this->assertTrue($aPost->isPublished());
        $this->assertInstanceOf(
            'DateTimeImmutable',
            $aPost->publicationDate()
        );
    }
}

```
<br>
<br>
<br>

### Дата Время (DateTimes)
<br>
Поскольку DateTimes широко используется в сущностях, мы считаем важным указать конкретные подходы к единицам тестирования сущностей, которые имеют поля с типами дат. 
Учтите, что Пост является новым, если он был создан в течение последних 15 дней:
<br>
<br>

```php
class Post
{
    const NEW_TIME_INTERVAL_DAYS = 15;
    // ...
    private $createdAt;
    
    public function __construct($aContent, $title)
    {
        // ...
        $this->createdAt(new DateTimeImmutable());
    }
    
    // ...
    public function isNew()
    {
        return
            (new DateTimeImmutable())
                ->diff($this->createdAt)
                ->days <= self::NEW_TIME_INTERVAL_DAYS;
    }
}

```
<br>

Метод isNew () должен сравнивать два значения DateTimes. Это сравнение между датой создания Post и сегодняшней датой. Мы вычисляем разницу и проверяем, меньше ли она указанного количества дней. 
Как выполнить единичное тестирование метода isNew ()? Как мы продемонстрировали в реализации, трудно воспроизвести конкретные потоки в наших тестовых комплектах. Какие у нас есть варианты?
<br>
<br>
<br>

#### Передача все дат в виде параметров
<br>
Одним из возможных вариантов может быть передача всех дат в качестве параметров при необходимости:
<br>
<br>

```php
class Post
{
    // ...
    public function __construct($aContent, $title, $createdAt = null)
    {
        // ...
        $this->createdAt($createdAt ?: new DateTimeImmutable());
    }

    // ...
    public function isNew($today = null)
    {
        return
            ($today ? :new DateTimeImmutable())
                ->diff($this->createdAt)
                ->days <= self::NEW_TIME_INTERVAL_DAYS;
    }
}

```
<br>
Это самый простой подход для целей модульного тестирования. Просто проходите различные пары дат, чтобы проверить все возможные сценарии со 100-процентным охватом. 
Однако, если учесть код клиента, который создает и запрашивает результат метода isNew (), все выглядит не так хорошо. Полученный в результате код может быть немного странным из-за постоянной передачи текущего DateTime:
<br>
<br>

```php
$aPost = new Post(
    'Hello world!',
    'Hi',
    new DateTimeImmutable()
);
$aPost->isNew(
    new DateTimeImmutable()
);

```

<br>
<br>


#### Класс тестирования
<br>
Другой альтернативой является использование шаблона Test Class. Идея состоит в том, чтобы расширить класс Post новым, которым мы можем манипулировать, чтобы форсировать определенные сценарии. 
Этот новый класс будет использоваться только в целях модульного тестирования. Плохая новость в том, что мы должны немного изменить исходный класс Post, извлекая некоторые методы и изменяя некоторые поля и методы с частных на защищенные. 
Некоторые разработчики могут беспокоиться об увеличении видимости свойств класса только из-за причин тестирования. Однако мы считаем, что в большинстве случаев это того стоит:
<br>
<br>

```php
class Post
{
    protected $createdAt;
    public function isNew()
    {
        return
            ($this->today())
                ->diff($this->createdAt)
                ->days <= self::NEW_TIME_INTERVAL_DAYS;
    }
    protected function today()
    {
        return new DateTimeImmutable();
    }
    protected function createdAt(DateTimeImmutable $aDate)
    {
        $this->assertIsAValidDate($aDate);
        $this->createdAt = $aDate;
    }
}

```
<br>

Как вы видите, мы извлекли логику для получения сегодняшней даты в метод today (). Таким образом, применяя шаблон метода, можно изменить его поведение от производного класса. 
Нечто подобное происходит с методом и полем createAt. Теперь они защищены, поэтому их можно использовать и переопределять в производных классах:
<br>
<br>

```php
class PostTestClass extends Post
{
    private $today;
    
    protected function today()
    {
        return $this->today;
    }
    public function setToday($today)
    {
        $this->today = $today;
    }
}

```
<br>

С учетом этих изменений теперь мы можем протестировать наш исходный класс Post с помощью тестирования PostTestClass:
<br>
<br>

```php
class PostTest extends PHPUnit_Framework_TestCase
{
    // ...
    
    /** @test */
    public function aPostIsNewIfIts15DaysOrLess()
    {
        $aPost = new PostTestClass(
            'A Post Content',
            'A Post Title'
        );
        $format = 'Y-m-d';
        $dateString = '2016-01-01';

        $createdAt = DateTimeImmutable::createFromFormat(
            $format,
            $dateString
        );

        $aPost->createdAt($createdAt);

        $aPost->setToday(
            $createdAt->add(
                new DateInterval('P15D')
            )
        );

        $this->assertTrue($aPost->isNew());

        $aPost->setToday(
            $createdAt->add(
                new DateInterval('P16D')
            )
        );
        
        $this->assertFalse($aPost->isNew());
    }
}

```
<br>
Последняя деталь: при таком подходе невозможно достичь 100-процентного покрытия в классе Post, потому что метод today () никогда не будет выполняться. 
Однако он может быть охвачен другими тестами.
<br>
<br>
<br>


#### External Fake
<br>
Другой вариант заключается в переносе вызовов конструктора DateTimeImmutable или именованных конструкторов с использованием нового класса и некоторых статических методов. 
При этом мы можем статически изменить результат этих методов, чтобы вести себя по-разному на основе конкретных сценариев тестирования:
<br>
<br>

```php
class Post
{
    // ...
    private $createdAt;
    public function __construct($aContent, $title)
    {
        // ...
        $this->createdAt(MyCustomDateTimeBuilder::today());
    }
    // ...
    public function isNew()
    {
        return
            (MyCustomDateTimeBuilder::today())
                ->diff($this->createdAt)
                ->days <= self::NEW_TIME_INTERVAL_DAYS;
    }
}

```
<br>

Для получения текущего DateTime теперь используется статический вызов MyCustomDateTimeBuilder:: today (). Этот класс также имеет несколько методов установки, чтобы подделать результат для возврата в следующих вызовах:
<br>
<br>

```php
class PostTest extends PHPUnit_Framework_TestCase
{
    // ...
    /** @test */
    public function aPostIsNewIfIts15DaysOrLess()
    {
        $createdAt = DateTimeImmutable::createFromFormat(
            'Y-m-d',
            '2016-01-01'
        );
        MyCustomDateTimeBuilder::setReturnDates(
            [
                $createdAt,
                $createdAt->add(
                    new DateInterval('P15D')
                ),
                $createdAt->add(
                    new DateInterval('P16D')
                )
            ]
        );
        $aPost = new Post(
            'A Post Content' ,
            'A Post Title'
        );
        $this->assertTrue(
            $aPost->isNew()
        );
        $this->assertFalse(
            $aPost->isNew()
        );
    }
}

```
<br>

Основная проблема этого подхода заключается в том, что он статически связан с объектом.
В зависимости от вашего варианта использования, также будет сложно создать гибкий поддельный объект.

<br>
<br>

#### Рефлексия
<br>
Можно также использовать методы Reflection для построения нового класса Post с пользовательскими датами.
Рассмотрим

[Mimic](https://github.com/keyvanakbary/mimic)

простую функциональную библиотеку для прототипирования объектов, гидратации данных и отображения данных:
<br>
<br>

```php
namespace Domain;

use mimic as m;

class ComputerScientist {
    private $name;
    private $surname;
    
    public function __construct($name, $surname) {
        $this->name = $name;
        $this->surname = $surname;
    }
    
    public function rocks() {
        return $this->name . ' ' . $this->surname . ' rocks!';
    }
}

assert(m\prototype('Domain\ComputerScientist') instanceof Domain\ComputerScientist);

m\hydrate('Domain\ComputerScientist', array(
    'name' => 'John',
    'surname' => 'McCarthy'
))->rocks(); //John McCarthy rocks!

assert(m\expose(new Domain\ComputerScientist('Grace', 'Hopper')) == array(
    'name' => 'Grace',
    'surname' => 'Hopper'
));

```
<br>

> **Поделиться и обсудить**
>Обсудите с коллегами по работе, как правильно протестировать объекты с фиксированнымидатами и придумайте дополнительные альтернативы.

<br>

Если вы хотите узнать больше о тестировании шаблонов и подходов, ознакомьтесь с книгой xUnit Test Patterns: Refactoring Test Code by Gerard Meszaros.

<br>
<br>
<br>

## Валидация
<br>
Валидация является очень важным процессом в нашей модели домена. Она проверяет не только правильность атрибутов, но и правильность целых объектов и их состава. Для поддержания этой модели в допустимом состоянии требуются различные уровни проверки. 
То, что объект состоит из допустимых атрибутов (базовых), не обязательно означает, что объект (в целом) является действительным. И наоборот: действительные объекты необязательно равны действительным составам.
<br>
<br>
<br>

### Валидация атрибутов
<br>
Некоторые люди понимают проверку как процесс, посредством которого служба проверяет состояние данного объекта. В этом случае проверка соответствует подходу Design-by-contract, который состоит из предварительных условий, постусловий и инвариантов. 
Одним из таких способов защиты отдельного атрибута является использование главы 3 «Объекты значений». Чтобы сделать наш дизайн более гибким для изменений, мы концентрируемся только на утверждении предварительных условий Домена, которые должны быть выполнены. 
Здесь мы будем использовать охрану как простой способ проверки предварительных условий:
<br>
<br>

```php
class Username
{
    const MIN_LENGTH = 5;
    const MAX_LENGTH = 10;
    const FORMAT = '/^[a-zA-Z0-9_]+$/';
    private $username;

    public function __construct($username)
    {
        $this->setUsername($username);
    }

    private function setUsername($username)
    {
        $this->assertNotEmpty($username);
        $this->assertNotTooShort($username);
        $this->assertNotTooLong($username);
        $this->assertValidFormat($username);
        $this->username = $username;
    }

    private function assertNotEmpty($username)
    {
        if (empty($username)) {
            throw new InvalidArgumentException('Empty username');
        }
    }

    private function assertNotTooShort($username)
    {
        if (strlen($username) < self::MIN_LENGTH) {
            throw new InvalidArgumentException(sprintf(
                'Username must be %d characters or more',
                self::MIN_LENGTH
            ));
        }
    }

    private function assertNotTooLong($username)
    {
        if (strlen($username) > self::MAX_LENGTH) {
            throw new InvalidArgumentException(sprintf(
                'Username must be %d characters or less',
                self::MAX_LENGTH
            ));
        }
    }

    private function assertValidFormat($username)
    {
        if (preg_match(self:: FORMAT, $username) !== 1) {
            throw new InvalidArgumentException(
                'Invalid username format'
            );
        }
    }
}

```
<br>

Как видно из приведенного выше примера, для создания объекта «Значение имени пользователя» необходимо выполнить четыре предварительных условия. Оно:

- Не должно быть пустым
- Должно содержать не менее 5 символов
- Должно содержать менее 10 символов
- Должен иметь формат буквенно-цифровых символов или знаков подчеркивания

<br>
Если все предварительные условия выполнены, атрибут будет установлен, и объект будет успешно построен. В противном случае будет создано исключение InvalidArgumentException, выполнение будет остановлено, и клиенту будет выдана ошибка.

Некоторые разработчики могут рассмотреть этот вид проверочного защитного программирования. Однако мы не проверяем, является ли ввод строкой или что значения null не разрешены. 
Мы не можем избегать людей, использующих наш код неправильно, но мы можем контролировать правильность нашего состояния Домена. Как видно из главы 3, «Объекты значений», проверка также может помочь нам в обеспечении безопасности.

[Defensive programming](https://en.wikipedia.org/wiki/Defensive_programming) не плохая вещь.<br> 
В общем, это имеет смысл при разработке компонентов или библиотек, которые будут использоваться в качестве третьей стороны в других проектах.
Однако при разработке собственного ограниченного контекста этих дополнительных параноидных проверок (нулей, основных типов, подсказок типа и т. д.) можно избежать, чтобы увеличить скорость разработки, полагаясь на охват набора модульных тестов.
<br>
<br>
<br>

### Проверка всего объекта
<br>
Бывают случаи, когда объект, состоящий из допустимых свойств в целом, все еще может считаться недействительным. Может быть заманчиво добавить этот вид проверки к самому объекту, но обычно это ==антипаттерн==. Проверка на более высоком уровне изменяется в ритме, отличном от ритма самой логики объекта. Кроме того, рекомендуется разделять эти обязанности.
Проверка информирует клиента о любых обнаруженных ошибках или собирает результаты для последующего просмотра, так как иногда мы не хотим останавливать выполнение при первом признаке неисправности.

Абстрактный и повторно используемый валидатор может быть примерно таким:
<br>
<br>

```php
abstract class Validator
{
    private $validationHandler;
    
    public function __construct(ValidationHandler $validationHandler)
    {
        $this->validationHandler = $validationHandler;
    }
    protected function handleError($error)
    {
        $this->validationHandler->handleError($error);
    }
    abstract public function validate();
}

```
<br>
В качестве конкретного примера мы хотим проверить весь объект Location, состоящий из допустимых объектов Country, City и Postcode Value Objects. 
Однако эти отдельные значения могут находиться в невалидном состоянии во время проверки. Может быть, город не является частью страны, или почтовый индекс не соответствует формату города:
<br>
<br>

```php
class Location
{
    private $country;
    private $city;
    private $postcode;
    public function __construct(
        Country $country, City $city, Postcode $postcode
    ) {
        $this->country = $country;
        $this->city = $city;
        $this->postcode = $postcode;
    }
    public function country()
    {
        return $this->country;
    }
    public function city()
    {
        return $this->city;
    }
    public function postcode()
    {
        return $this->postcode;
    }
}
```
<br>

Validator проверяет состояние объекта Location в целом, анализируя значение взаимосвязей между свойствами:
<br>
<br>

```php
class LocationValidator extends Validator
{
    private $location;
    public function __construct(
        Location $location, ValidationHandler $validationHandler
    ) {
        parent:: __construct($validationHandler);
        $this->location = $location;
    }
    public function validate()
    {
        if (!$this->location->country()->hasCity(
            $this->location->city()
        )) {
            $this->handleError('City not found');
        }
        if (!$this->location->city()->isPostcodeValid(
            $this->location->postcode()
        )) {
            $this->handleError('Invalid postcode');
        }
    }
}

```
<br>

После установки всех свойств мы сможем проверить Сущность, скорее всего, после некоторых описанных процессов. На поверхности это выглядит так, как если бы местоположение подтвердило себя. 
Однако это не так. Класс Location делегирует эту проверку конкретному экземпляру валидатора, разделяя эти две четкие обязанности:
<br>
<br>

```php
class Location
{
    // ...
    public function validate(ValidationHandler $validationHandler)
    {
        $validator = new LocationValidator($this, $validationHandler);
        $validator->validate();
    }
}

```
<br>
<br>


#### Decoupling сообщений валидации
<br>
С некоторыми незначительными изменениями в существующей реализации мы можем отделить сообщения проверки от средства проверки:
<br>
<br>

```php
class LocationValidationHandler implements ValidationHandler
{
    public function handleCityNotFoundInCountry();
    public function handleInvalidPostcodeForCity();
}
```

<br>

```php
class LocationValidator
{
    private $location;
    private $validationHandler;
    
    public function __construct(
        Location $location,
        LocationValidationHandler $validationHandler
    ) {
        $this->location = $location;
        $this->validationHandler = $validationHandler;
    }
    public function validate()
    {
        if (!$this->location->country()->hasCity(
            $this->location->city()
        )) {
            $this->validationHandler->handleCityNotFoundInCountry();
        }
        if (! $this->location->city()->isPostcodeValid(
            $this->location->postcode()
        )) {
            $this->validationHandler->handleInvalidPostcodeForCity();
        }
    }
}

```
<br>

Нам также необходимо изменить подпись метода проверки следующим образом:
<br>
<br>

```php
class Location
{
    // ...
    public function validate(
        LocationValidationHandler $validationHandler
    ) {
        $validator = new LocationValidator($this, $validationHandler);
        $validator->validate();
    }
}
```
<br>
<br>
<br>

### Валидация композиции объектов
<br>
Проверка достоверности композиций объектов может быть сложной задачей. Таким образом, предпочтительным способом достижения этого является использование доменной службы. 
Затем служба взаимодействует с репозиториями, чтобы получить действительный агрегат. Из-за возможных сложных графов объектов, которые могут быть созданы, 
агрегат может находиться в промежуточном состоянии, требуя предварительной проверки других агрегатов.
События домена можно использовать для уведомления других частей системы о проверке определенного элемента.

<br>
<br>
<br>

### Entities и Events домена
<br>
Мы рассмотрим главу 6 «Доменные события» в будущем; однако важно подчеркнуть, что операции, выполняемые с сущностями, могут инициировать события домена. 
Этот подход используется для передачи изменения Домена другим частям Приложения или даже другим Приложениям, как показано в Главе 12, Интеграция ограниченных контекстов:
<br>
<br>

```php
class Post
{
    // ...
    public function publish()
    {
        $this->setStatus(
            Status::published()
        );
        $this->publishedAt(new DateTimeImmutable());
        DomainEventPublisher::instance()->publish(
            new PostPublished($this->id)
        );
    }
    public function unpublish()
    {
        $this->setStatus(
            Status::draft()
        );
        $this-> publishedAt = null;
        DomainEventPublisher::instance()->publish(
            new PostUnpublished($this->id)
        );
    }
    // ...
}
```
<br>

События домена могут даже запускаться при создании нового экземпляра нашей Сущности:
<br>
<br>

```php
class User
{
    // ...
    public function __construct(UserId $userId, $email, $password)
    {
        $this->setUserId($userId);
        $this->setEmail($email);
        $this->setPassword($password);
        DomainEventPublisher::instance()->publish(
            new UserRegistered($this->userId)
        );
    }
}
```
<br>
<br>
<br>

### В заключении
<br>
Некоторые понятия в Домене требуют Идентичность - то есть изменения их внутренних состояний не меняют их собственные уникальные идентичности. 
Мы видели, как моделирование Identity как объекта-значения приносит такие преимущества, как неизменяемость, в дополнение к логике для работы с самой Identity. 
Мы также показали несколько способов предоставления Identity, переформулированных в следующих указателях:

- Механизм персистентности: Простой в реализации, но вы не будете иметь Identity до сохранения объекта, что задерживает и затрудняет распространение событий.


- Суррогатный ID: Некоторым ОРМ требуется дополнительное поле в Сущности, чтобы сопоставить Идентификатор с сохраняющимся механизмом.


- Предоставляется клиентом: Иногда Identity подходит под концепцию Домена и вы можете смоделировать его внутри вашего Домена.


- Создано приложением: Для создания идентификаторов можно использовать библиотеку.


- Генерируется ограниченным контекстом: возможно, самая сложная стратегия. Другие ограниченные контексты обеспечивают интерфейс для создания идентификатора.

<br>

Мы видели и обсуждали Doctrine как механизм персистентности, мы смотрели на недостатки использования шаблона Active Record и, наконец, мы проверили различные уровни проверки Сущности:

- Проверка атрибутов: Проверка специфики в состоянии объекта через предварительные условия, постусловия и инварианты.


- Проверка всего объекта: поиск непротиворечивости объекта в целом. Извлечение проверки во внешнюю службу является хорошей практикой.


- Композиции объектов: Сложные композиции объектов можно проверить с помощью доменных служб. Хороший способ донести это до остальной части приложения - через События домена.


































































































































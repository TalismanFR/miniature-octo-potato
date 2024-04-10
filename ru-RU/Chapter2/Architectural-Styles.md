Глава 2. Архитектурные стили
==

Чтобы иметь возможность создавать сложные приложения, одним из ключевых требований является наличие архитектурного дизайна, который соответствует потребностям приложения. Одно из преимущества DDD заключается в том, что он не привязан к какому-либо конкретному стилю архитектуры. Вместо этого мы можем свободно выбирать архитектуру, которая наилучшим образом соответствует потребностям каждого Ограниченного Контекста внутри основного Домена, который предлагает разнообразный набор архитектурных решений для каждого конкретной проблемы Домена.

Например:
- Система Обработки Заказов может использовать подход **Event Sourcing** для отслеживания всех различных операций с заказом.

- Каталог продуктов может использовать **CQRS** для предоставления информации о продуктах различным клиентам.

- Система Управления Контентом может использовать **Гексогональную Архитектуру** (Hexagonal Architecture) для представления требований, таких как блоги, статичные страницы и т.д.

В этой главе представлено введение во все соответствующие архитектурные стили в контексте PHP, следуя эволюции от традиционного (старого) PHP-кода к более сложной архитектуре. Обратите внимание, что, хотя существует много других существующих архитектурных стилей, таких как Data Fabric или SOA, некоторые из них оказались слишком сложными для представления средствами PHP.

<br>

## Старые, добрые времена

До релиза PHP версии 4, язык не охватывал Объектно-Ориентированную Парадигму. В то время обычным способом написания приложения было использование процедур и глобального состояния. Такие понятия, как **Разделение Ответственности** (Separation of Concerns (SoC)) и **Модель-Представление-Контролер** (Model-View-Controller (MVC)) были широко распространены среди сообщества PHP.

Пример ниже представляет собой приложение, написанное традиционным способом, где приложение состоят из множества фронтальных контроллеров, смешанных с кодом HTML. В те времена слой Инфраструктуры, Представления, UI, и Доменные слои были перемешаны.

<br>

```php
<?php
include __DIR__ . '/bootstrap.php';
$link = mysql_connect('localhost', 'a_username', '4_p4ssw0rd');
if (!$link) {    
    die('Could not connect: ' . mysql_error());
}
mysql_set_charset('utf8', $link);
mysql_select_db('my_database', $link);
$errormsg = null;
if (isset($_POST['submit']) && isValid($_POST['post'])) {
    $post = getFrom($_POST['post']);
    mysql_query('START TRANSACTION', $link);
    $sql = sprintf(
        "INSERT INTO posts (title, content) VALUES ('%s','%s')",
        mysql_real_escape_string($post['title']),
        mysql_real_escape_string($post['content']
    ));
    $result = mysql_query($sql, $link);
    if ($result) {
        mysql_query('COMMIT', $link);
    } else {
        mysql_query('ROLLBACK', $link);
        $errormsg = 'Post could not be created! :(';
    }}
$result = mysql_query('SELECT id, title, content FROM posts', $link);?>
<html>
    <head></head>
    <body>
        <?php if (null !== $errormsg) : ?>
            <div class="alert error"><?php echo $errormsg; ?></div>
        <?php else: ?>
            <div class="alert success">
                Bravo! Post was created successfully!
            </div>
        <?php endif; ?>
        <table>
            <thead><tr><th>ID</th><th>TITLE</th>
            <th>ACTIONS</th></tr></thead>
            <tbody>
            <?php while($post = mysql_fetch_assoc($result)) : ?>
                <tr>
                    <td><?php echo $post['id']; ?></td>
                    <td><?php echo $post['title']; ?></td>
                    <td><?php editPostUrl($post['id']); ?></td>
                </tr>
            <?php endwhile; ?>
            </tbody>
        </table>
   </body>
 </html>
 <?php mysql_close($link); ?>
```
![Cool](https://raw.githubusercontent.com/TalismanFR/dddinphp/master/share/cool.jpeg)

<br>

Этот стиль кода часто называют **Большой Ком Грязи** (Big Ball of Mud).
Однако в этом стиле улучшение стало то, что заголовок и нижний колонтитул веб-страницы были заключены в отдельные файлы. Это позвонило избежать дублирования и способствовало повторному использованию:

<br>

```php
<?php
include __DIR__ . '/bootstrap.php';
$link = mysql_connect('localhost', 'a_username', '4_p4ssw0rd');
if (!$link) {
    die('Could not connect: ' . mysql_error());
}
mysql_set_charset('utf8', $link);
mysql_select_db('my_database', $link);
$errormsg = null;
if (isset($_POST['submit']) && isValid($_POST['post'])) {
    $post = getFrom($_POST['post']);
    mysql_query('START TRANSACTION', $link);
    $sql = sprintf(
        "INSERT INTO posts(title, content) VALUES('%s','%s')",
        mysql_real_escape_string($post['title']),
        mysql_real_escape_string($post['content'])
    );
    $result = mysql_query($sql, $link);
    if ($result) {
        mysql_query('COMMIT', $link);
    } else {
        mysql_query('ROLLBACK', $link);
        $errormsg = 'Post could not be created! :(';
    }
}
$result = mysql_query('SELECT id, title, content FROM posts', $link);
?>
<?php 
include __DIR__ . '/header.php';
 ?>
<?php 
if (null !== $errormsg) : ?>
    <div class="alert error">
<?php echo $errormsg; ?>
</div>
<?php else: ?>
    <div class="alert success">
        Bravo! Post was created successfully!
    </div>
<?php endif;
 ?>
<table>
    <thead>
        <tr>
            <th>ID</th>
            <th>TITLE</th>
            <th>ACTIONS</th>
        </tr>
    </thead>
    <tbody>
    <?php while($post = mysql_fetch_assoc($result)): ?>
        <tr>
            <td><?php echo $post['id']; ?></td>
            <td><?php echo $post['title']; ?></td>
            <td><?php editPostUrl($post['id']); ?></td>
        </tr>
    <?php endwhile; ?>
    </tbody>
</table>
<?php include __DIR__ . '/footer.php'; ?>
```

<br>

В настоящее время, хотя это крайне нежелательно, все еще встречаются приложения которые используют подобный процедурный подход (скорее всего legacy код). Основным недостатком этого стиля архитектуры является то, что нет реального **Разделения Ответственности** (Separation of Concerns). Обслуживание и стоимость разработки приложения, разработанного таким образом, резко возрастает по сравнению с другими известными и проверенными архитектурами.

<br>

## Многоуровневая архитектура (Layered Architecture)

С точки зрения удобства поддержки и повторного использования кода, наилучший способ сделать код более простым в обслуживании - это разделить концепции, то есть создать слои для каждой отдельной задачи.
В нашем предыдущем примере легко сформировать разные уровни:
- первый для инкапсуляции доступа и манипулирования данными.
- второй для решения проблем инфраструктуры.
- третий для оркестрирования двух предыдущих.

Основное правило многоуровневой архитектуры - это то, что каждый слой должен иметь связь (возможность использовать) с нижестоящими слоями, как показано на рисунке ниже:

![Layered Architecture](https://raw.githubusercontent.com/TalismanFR/dddinphp/master/share/image--004.jpg)

**Многоуровневая Архитектура** стремится к разделению различных компонентов приложения. Например, с точки зрения предыдущего примера, **Представление** сообщения в блоге должно быть полностью независимым от сообщения в блоге как концептуальной сущности.
Сообщение в блоге как концептуальная сущность может быть отображено одним или несколькими **Представлениями**, вместо того чтобы быть тесно связанным с каким либо конкретным **Представлением**. Это принято называть **Разделением Ответственности** (Separation of Concerns).

Другой парадигмой и шаблоном архитектуры, преследующей ту же цель, является шаблон **Model-View-Controller**. Изначально он задумывался и широко использовался для создания настольных приложений с графическим интерфейсом, а теперь он в основном использвуется в веб-приложениях благодаря популяризации веб-фреймворков, таких как Symfony, Zend, CodeIgniter.

<br>

### Model-View-Controller

**MVC**
Это архитектурный шаблон и парадигма, которая делит приложение на три основных уровня, описанных в следующих пунктах:

**Модель (The Model)**
Содержит все поведения Доменой Модели. Этот уровень управляет всеми данными, логикой, и бизнесс-правилами назависимо от уровня представления данных. Уровень Модели является сердцем и душой каждого приложения MVC.

**Контроллер (The Controller)**
Организует взаимодействия между другими уровнями приложения, запускает действия в слое Модели для обновления ее состояния и обновления слоя Представления связанного с этой моделью.
Кроме того, контроллер может отправлять сообщения на уровень Представления для изменения конкретного отображения Модели.

**Представление (The View)**
Отображает различные Представления слоя Модели и предоставляет способ вызывать изменения в состоянии модели.

![The MVC pattern](https://raw.githubusercontent.com/TalismanFR/dddinphp/master/share/image--006.jpg)

<br>

### Пример Многоуровневой Архитектуры

#### Модель

Продолжая предыдущий пример, мы упомянули, что различные задачи должны быть разделены. Для этого все слои должны быть идентифицированы в нашем оригинальном запутанном коде. На протяжении всего этого процесса мы должны уделять особое внимание коду, соответствующему уровню **Модели**, который будет ядром приложения:

<br>

```php
<?php 
class Post
{
    private $title;
    private $content;

    public static function writeNewFrom($title, $content)
    {
        return new static($title, $content);
    }

    private function __construct($title, $content)
    {
        $this->setTitle($title);
        $this->setContent($content);
    }

    private function setTitle($title)
    {
        if (empty($title)) {
            throw new RuntimeException('Title cannot be empty');
        }
        $this->title = $title;
    }

    private function setContent($content)
    {
        if (empty($content)) {
            throw new RuntimeException('Content cannot be empty');
        }
        $this->content = $content;
    }
}

class PostRepository
{
    private $db;
    public function __construct()
    {
        $this->db = new PDO(
            'mysql:host=localhost;dbname=my_database',
            'a_username',
            '4_p4ssw0rd',
            [
            PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES utf8mb4',
            ]
        );
    }

    public function add(Post $post)
    {
        $this->db->beginTransaction();
        try {
            $stm = $this->db->prepare(
                'INSERT INTO posts (title, content) VALUES (?, ?)'
            );
            $stm->execute([
                $post->title(),
                $post->content(),
            ]);
            $this->db->commit();
        } catch (Exception $e) {
            $this->db->rollback();
            throw new UnableToCreatePostException($e);
        }
    }
}
```

<br>

Слой Модели теперь определяется классом `Post` и классом `PostRepository`.
Класс `Post` представляет сообщение в блоге, а класс `PostRepository` представляет всю коллекцию доступных сообщений в блоге. Кроме того, есть еще один слой - тот, который координирует необходимое поведение Модели Домена.
Рассмотрим **Прикладной** слой (Application layer):

<br>

```php
<?php
class PostService
{
    public function createPost($title, $content)
    {
        $post = Post::writeNewFrom($title, $content);
        (new PostRepository())->add($post);
        return $post;
    }
}
```

<br>

Класс `PostService` - это, так называемая Прикладная служба (Служба Приложения, Application Service), и её целью является организация поведения Домена. Ни один другой тип объекта не может напрямую взаимодействовать с внутренними слоями Модели.

<br>

#### Представление (View)

Представление - это слой, который может отправлять и получать сообщения со слоя модели и/или со слоя контроллера. Его основная цель - отобразить модель пользователю на уровне пользовательского интерфейса, а также обновлять это отображение при обновлении модели.

В общем случае, слой Представления получает объект (часто это Объект Передачи Данных (DataTransferObject)), тем самым собирая всю необходимую информацию для успешного отображения. Для PHP есть несколько шаблонизаторов, которые могут помочь
отделить Представление Модели от самой Модели и от Контроллера. Самым популярным, на текущий момент, является Twig.
Посмотрим как будет выглядеть слой Представления с Twig.

<br>

> **DTO вместо экземпляров Модели?**
>
> Это старый холивар. Зачем создавать DTO вместо того чтобы передать экземпляр класса Модели?
> Основная причина и самый лаконичный ответ, знакомое нам **Разделение Ответственности** (Separation of Concerns). Возможность Представления просматривать и использовать поступающие объекты приводит к тесной связи между слоем Представления и слоем Модели. Фактически, изменение в уровне Модели может нарушить все представления, которые используют измененные экземпляры модели.

<br>

```twig
{% extends "base.html.twig" %}
{% block content %}
    {% if errormsg is defined %}
        <div class="alert error">{{ errormsg }}</div>
    {% else %}
    <div class="alert success">
        Bravo! Post was created successfully!
    </div>
    {% endif %}
    <table>
        <thead>
            <tr>
                <th>ID</th>
                <th>TITLE</th>
                <th>ACTIONS</th>
            </tr>
        </thead>
        <tbody>
            {% for post in posts %}
                <tr>
                    <td>{{ post.id }}</td>
                    <td>{{ post.title }}</td>
                    <td><a href="{{ editPostUrl(post.id) }}">Edit Post</a></td>
                </tr>
            {% endfor %}
        </tbody>
    </table>
{% endblock %}
```

<br>

В большинстве случаев, когда Модель инициирует изменение состояния, она также уведомляет связанные Представления, чтобы обновить пользовательский интерфейс. В типичном веб-сценарии синхронизация между Моделью и её представлениями может быть немного сложной из-за природы клиент-серверной архитектуры. В таких средах, обычно требуется использовать JavaScript, чтобы поддерживать эту синхронизацию.
По этой причине, JavaScript MVC фреймворки, о которых говорится ниже, стали широко популярными в последние годы:
- AngularJS
- Ember.js
- Marionette.js
- React

<br>

#### Контроллер (Controller)

Слой Контроллера отвечает за организацию и взаимодействия слоёв Модели и Представления. Он получает сообщения от слоя Представления и запускает поведение Модели для выполнения желаемого действия.
Кроме того, он отправляет сообщения в Представления для отрисовки отображения Модели. Обе операции выполняются благодаря прикладному уровню, который отвечает за организацию, взаимодействие и инкапсуляцию поведения Домена.

В терминах веб-приложения на PHP, Контроллер, обычно, охватывает набор классов, которые для достижения своих целей "Общаются на HTTP". Другими словами они получают HTTP Request и дают ответ в виде HTTP Response.

<br>

```php
<?php
class PostsController
{
    public function updateAction(Request $request)
    {
        if (
            $request->request->has('submit') &&
            Validator::validate($request->request->post)
        ) {
            $postService = new PostService();
            try {
                $postService->createPost(
                $request->request->get('title'),
                $request->request->get('content')
            );
            $this->addFlash(
                'notice',
                'Post has been created successfully!'
            );
            } catch (Exception $e) {
                $this->addFlash(
                    'error',
                    'Unable to create the post!'
                );
            }
        }

        return $this->render('posts/update-result.html.twig');
    }
}
```

<br>

### Инверсия зависимостей. Гексагональная архитектура.

Следую основному правилу Многоуровневой архитектуры, существует риск инкапсулирования инфраструктурных проблем в реализацию Доменных интерфейсов.
Например, класс PostRepository из предыдущего MVC примера, был помещен в Домен предметной области. Однако размещение инфраструктурных деталей прямо в теле нашего Домена нарушает Разделение Ответственности (Domain Separation).
Это создает проблемы. Трудно избежать нарушения основных правил многоуровневой архитектуры, что приводит  к написанию кода который тяжело поддается тестированию, т.к. уровень Домена осведомлен о технических реализациях.

<br>

#### Принцип Инверсии Зависимостей (The Dependency Inversion Principle (DIP))

Как мы можем это исправить? Поскольку уровень Доменной Модели связан с конкретной реализаций инфраструктуры (PostRepository), можно применить принцип инверсии зависимости (DIP), переместив **Инфраструктурный** слой выше всех остальных слоёв.

<br>

> **Принцип Инверсии Зависимостей**
> Модули более высокого уровня не должны зависеть от нижележащий уровней. Оба должны связываться через абстракции.
> Абстракции не должны зависеть от деталей. Детали должны зависеть от абстракций. _Robert C. Martin_

<br>

При использовании принципа Инверсии Зависимостей, схема архитектуры меняется, и уровень инфраструктуры, который можно назвать модулем низкого уровня, теперь зависит от пользовательского интерфейса, Прикладного слоя, и Доменного слоя, которые являются высокоуровневыми модулями. Зависимость была инвертирована.

Но что такое Гексагональная Архитектура и как она вписывается во все это? Гексагональная Архитектура (так же именуемая как Порты и Адаптеры) была определена Алистером Кокберном в его книге _"Hexagonal Architecture"_.
Он изображает приложение как шестиугольник, где каждая сторона представляет Порт с одним или несколькими Адаптерами.  
**Порт** - это коннектор с подключаемым **Адаптером**, который преобразует внешний вход во что-то, что может понять внутреннее приложение. С точки зрения DIP, **Порт** был бы модулем высокого уровня, а **Адаптер** был бы модулем нижележащего уровня. Кроме того, если приложению необходимо отправить сообщение во внешний сервис, оно также будет использовать **Порт** с **Адаптером** для преобразования сообщения в язык понятный внешнему сервису и последующей отправкой сообщения.
По этой причине **Гексагональная** архитектура воспитывает концепцию симметрии в приложении, а также является основной причиной, по которой меняется схема архитектуры.
Её часто представляют в виде шестиугольника, потому что больше не имеет смысла говорить о верхнем или нижнем слое. Вместо этого, в Гексагональной Архитектуре говорится о внешнем слое и внутреннем.

<br>

#### Применение Гексогональной Архитектуры

Продолжим рассматривать пример с блогом. Первая концепция, которая нам нужна, это **Порт**, через который внешний мир может общаться с приложением. Для этого случая мы будем использовать HTTP-порт и соответствующий ему Адаптер.
Внешним будет порт для отправки сообщений в приложение. В примере блога использовалась база данных для хранения постов блога, поэтому нам так же необходим **Порт** для извлечения постов из базы данных:

<br>

```php
<?php
interface PostRepository
{
    public function byId(PostId $id);
    public function add(Post $post);
}
```

<br>

Этот **интерфейс** представляет собой **Порт**, через который приложение будет получать информацию о постах блога, и он будет расположен на уровне **Доменного** слоя. Теперь нужен **Адаптер** для этого **Порта**.
**Адаптер** отвечает за определение способа извлечения сообщений из блога с использованием определенной технологии.

<br>

```php
<?php
class PDOPostRepository implements PostRepository
{
    private $db;
    public function __construct(PDO $db)
    {
        $this->db = $db;
    }
    public function byId(PostId $id)
    {
        $stm = $this->db->prepare(
            'SELECT * FROM posts WHERE id = ?'
        );
        $stm->execute([$id->id()]);
        return recreateFrom($stm->fetch());
    }
    public function add(Post $post)
    {
        $stm = $this->db->prepare(
            'INSERT INTO posts (title, content) VALUES (?, ?)'
        );
        $stm->execute([
            $post->title(),
            $post->content(),
        ]);
    }
}
```

<br>

После того, как мы определили Порт и его Адаптер, последним шагом будет рефакторинг класса PostService, чтобы он использовал новый механизм. Это может быть легко достигнуто с помощью Иньекции Зависимостей (Dependency Injection):

<br>

```php
<?php
class PostService
{
    private $postRepository;

    public function __construct(PostRepositor $postRepository)
    {
        $this->postRepository = $postRepository;
    }

    public function createPost($title, $content)
    {
        $post = Post::writeNewFrom($title, $content);
        $this->postRepository->add($post);

        return $post;
    }
}
```

<br>

Это простой пример Гегсагональной архитектуры. Это гибкая архитектура, которая способствует Разделению Ответственности, как в Многоуровневой Архитектуре. Это также способствует симметрии, благодаря наличию внутренней части приложения, которая связывается с внешним слоем через Порты. Отныне, это будет основополагающая архитектура, используемая для построения и объяснения CQRS и Event Sourcing.

Более детальный разбор этой архитектуры вы можете найти в главе _"Приложение. Гексагональная архитектура в PHP"_.
Для более подробного примера вам следует перейти к _Главе 11 "Приложение"_, в которой объясняются такие сложные темы, как транзакционность и другие сквозные вопросы.

<br>

### Command Query Responsibility Segregation (CQRS)

Гексагональная архитектура - это хорошая основополагающая архитектура, но она имеет некоторые ограничения. Например, сложные пользовательские интерфейсы могут требовать Агрегированной информации, отображаемой в различных формах (Глава 8, Агрегаты), или они могут требовать данных, полученных из нескольких агрегатов. И в этом сценарии мы могли бы реализовать множество методов поиска в различных Репозиториях (возможно, столько же, сколько есть представлений пользовательского интерфейса в приложении). Или, может быть, мы решим перенести эту сложность в Службу Приложений (Application Service), используя сложные структуры для сбора и объединения данных из различных Агрегатов. Пример:

<br>

```php
<?php
interface PostRepository
{
    public function save(Post $post);
    public function byId(PostId $id);
    public function all();
    public function byCategory(CategoryId $categoryId); 
    public function byTag(TagId $tagId);
    public function withComments(PostId $id);
    public function groupedByMonth();
    // ...
}
```

<br>

Когда этими конструкциями злоупотребляют, построение пользовательских представлений может стать действительно болезненным. Мы должны принять компромисс между тем, чтобы заставлять Службу Приложений возвращать Доменную Модель или возвращать некие DTO`s. В последнем варианте (DTO), мы избегаем тесной связи между
Доменной Моделью и кодом Инфраструктуры (веб-контроллеры, cli контроллеры и т.д.).

К счастью, есть другой подход. Если проблема заключается в множественных и разрозненных представлениях, то мы можем исключить их из Доменной Модели и начать рассматривать их как чисто инфраструктурную проблему. Эта опция базируется на принципе разработки, Разделение Команд и Запросов (CQS, Command Query Separation). Этот принцип был определен Бертраном Мейером (Bertrand Meyer), и, в свою очередь, он породил новый архитектурный паттерн под названием
"Разделение Ответственности на Запросы и Команды" (CQRS, Command Query Responsibility Segregation), как это определенно Грегом Янгом (Greg Young).

<br>

> **Command Query Separation (CQS)**
> Задание вопроса не должно менять ответ - Бертран Мейер.
> Этот принцип разработки гласит, что каждый метод должен быть либо командой, выполняющей действие, либо запросом, возвращающим данные вызывающей стороне, но не обоими сразу. Wikipedia

<br>

**CQRS** стремится к еще более агрессивному Разделению Проблем, деля модель на две части:
- Модель Записи (The Write Model): также известная как Командная модель (Command Model), она выполняет запись и несет ответственность за истинное поведение домена.
- Модель Чтения (The Read Model): она берет на себя ответственность за чтение в приложении и рассматривается как нечто что должно выходить за пределы предметной области.

<br>

Каждый раз, когда кто-то запускает команду для модели записи, выполняется запись в нужное хранилище данных. Кроме того, она запускает обновление Модели Чтения, чтобы в ней отобразились последние изменения.

Это строгое разделение вызывает еще одно проблему - Согласованность в Конечном Итоге (Eventual Consistency).
Согласованность модели чтения теперь зависит от команд, выполняемых Моделью Записи. Другими словами, модель чтения в конечном итоге непротиворечива. То есть каждый раз, когда Модель Записи выполняет команду, она запускает процесс, который будет отвечать за обновление модели чтения в соответствии с последними обновлениями модели записи. Есть некоторый лаг во времени, когда пользовательский интерфейс может представить устаревшую информацию пользователю. В веб-сценарии это происходит часто, поскольку мы несколько ограничены текущими технологиями.

Подумайте о системе кэширования стоящим перед веб-приложением. Каждый раз, когда база данных обновляется новой информацией, данные на уровне кэша потенциально могут быть устаревшими, поэтому каждый раз, когда она обновляется, должен быть процесс, который обновляет систему кэша. Системы кэширования в конечном итоге становятся согласованными.

Такие процессы, в терминалогии CQRS, называются Проекции Модели Записи (Write Model Projections) или просто Проекции. Мы проецируем Модели Записи на Модель Чтения. Этот процесс может быть синхронным или асинхронным, в зависимости от ваших потребностей, и этом может быть сделано благодаря полезному тактическому шаблону проектирования - Глава "События Домена" - который будет подробно объяснен позже в книге.

Основной проекций Модели Записи является сбор всех опубликованных событий домена и обновление модели чтения всей информацией, поступившей из событий.

<br>

#### Модель записи

Модель записи является истинным владельцем поведения Домена. Продолжая
наш пример, интерфейс репозитория будет упрощен до следующего:

<br>

```php
<?php
interface PostRepository
{
    public function save(Post $post);
    public function byId(PostId $id);
}
```

<br>

Теперь `PostRepository` освобожден от всех задач чтения, кроме одной: функция byId, которая отвечает за загрузку Агрегата по его ID, для дальнейшей работы с ним.

Так же будут удалены все методы запросов (query) из модели `Post`, оставив только методы команд. Это приводит к тому, что мы избавляемся от всех методов получения данных и любых других методов, предоставляющих информацию о Агрегате Post. Вместо этого будут опубликованы Доменные События, чтобы запустить проекцию Модели Записи использую подписку на них (события):

<br>

```php
<?php
class AggregateRoot
{
    private $recordedEvents = [];

    protected function recordApplyAndPublishThat(
        DomainEvent $domainEvent
    ) {
        $this->recordThat($domainEvent);
        $this->applyThat($domainEvent);
        $this->publishThat($domainEvent);
    }

    protected function recordThat(DomainEvent $domainEvent)
    {
        $this->recordedEvents[] = $domainEvent;
    }

    protected function applyThat(DomainEvent $domainEvent)
    {
        $modifier = 'apply' . get_class($domainEvent);
        $this->$modifier($domainEvent);
    }

    protected function publishThat(DomainEvent $domainEvent)
    {
        DomainEventPublisher::getInstance()->publish($domainEvent);
    }

    public function recordedEvents()
    {
        return $this->recordedEvents;
    }

    public function clearEvents()
    {
        $this->recordedEvents = [];
    }
}

class Post extends AggregateRoot
{
    private $id;
    private $title;
    private $content;
    private $published = false;
    private $categories;

    private function __construct(PostId $id)
    {
        $this->id = $id;
        $this->categories = new Collection();
    }

    public static function writeNewFrom($title, $content)
    {
        $postId = PostId::create();
        $post = new static($postId);
        $post->recordApplyAndPublishThat(
            new PostWasCreated($postId, $title, $content)
        );
    }

    public function publish()
    {
        $this->recordApplyAndPublishThat(
            new PostWasPublished($this->id)
        );
    }

    public function categorizeIn(CategoryId $categoryId)
    {
        $this->recordApplyAndPublishThat(
            new PostWasCategorized($this->id, $categoryId)
        );
    }

    public function changeContentFor($newContent)
    {
        $this->recordApplyAndPublishThat(
            new PostContentWasChanged($this->id, $newContent)
        );
    }

    public function changeTitleFor($newTitle)
    {
        $this->recordApplyAndPublishThat(
            new PostTitleWasChanged($this->id, $newTitle)
        );
    }
}
```

<br>

Все действия, которые инициируют изменение состояния, реализуются
через события Домена. Для каждого опубликованного Доменного События существует
соотвествующий метод apply, отвечающий за отражение изменения состояния:

<br>

```php
<?php
class Post extends AggregateRoot
{
    // ...
    protected function applyPostWasCreated(
        PostWasCreated $event
    ) {
        $this->id = $event->id();
        $this->title = $event->title();
        $this->content = $event->content();
    }

    protected function applyPostWasPublished(
        PostWasPublished $event
    ) {
        $this->published = true;
    }

    protected function applyPostWasCategorized(
        PostWasCategorized $event
    ) {
        $this->categories->add($event->categoryId());
    }

    protected function applyPostContentWasChanged(
        PostContentWasChanged $event
    ) {
        $this->content = $event->content();
    }

    protected function applyPostTitleWasChanged(
        PostTitleWasChanged $event
    ) {
        $this->title = $event->title();
    }
}
```

<br>

#### Модель чтения

Модель Чтения, так же известная как модель запросов (Query Model), является
денормализованной, в интересах Домена, моделью данных.
Фактически, в CQRS все задачи чтения считаются процессами отчетности в инфраструктурной задаче. Как правило, при использовании CQRS Модель Чтения зависит от потребностей пользовательского интерфейса и сложности представлений, состовляющих пользовательский интерфейс. В ситуации, когда модель чтения определяется в терминах реляционных баз данных, простейшим подходом было бы установить взаимно-однозначные отношения между таблицами базы данных и представлениями пользовательского интерфейса.
Эти таблицы базы данных и представления пользовательского интерфейса будут
обновлены с использованием проекций Модели Записи, инициированных событиями домена, опубликованными стороной записи:

<br>

```sql
-- Определение представляния UI для поста с его комментариями
CREATE TABLE single_post_with_comments (
    id INTEGER NOT NULL,
    post_id INTEGER NOT NULL,
    post_title VARCHAR(100) NOT NULL,
    post_content TEXT NOT NULL,
    post_created_at DATETIME NOT NULL,
    comment_content TEXT NOT NULL
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci;

-- Вставка данных
INSERT INTO single_post_with_comments VALUES
    (1, 1, "Layered" , "Some content", NOW(), "A comment"),
    (2, 1, "Layered" , "Some content", NOW(), "The comment"),
    (3, 2, "Hexagonal" , "Some content", NOW(), "No comment"),
    (4, 2, "Hexagonal", "Some content", NOW(), "All comments"),
    (5, 3, "CQRS", "Some content", NOW(), "This comment"),
    (6, 3, "CQRS", "Some content", NOW(), "That comment");

-- Запрос на получения данных статьи
SELECT * FROM single_post_with_comments WHERE post_id = 1;
```

<br>

Важной особенностью этого архитектурного стиля является то, что Модель Чтения должна быть полностью одноразовой, поскольку истинное состояние приложения определяется Моделью Записи. Это означает что Модель Чтения может быть удалена и пересоздана при необходимости используя проекцию Модели Записи.

Ниже мы можем увидеть некоторые примеры возможных представлений в приложении блога:

<br>

```sql
SELECT * FROM
posts_grouped_by_month_and_year
ORDER BY month DESC,year ASC;

SELECT * FROM
posts_by_tags
WHERE tag = "ddd";

SELECT * FROM
posts_by_author
WHERE author_id = 1;
```

<br>

Важно отметить, что CQRS не ограничивается реализацией модели для реляционной
базы данных. Это зависит исключительно от потребностей создаваемого приложения.
Это может быть реляционная база данных, документно-ориентированная база данных, хранилище типа ключ-значение, или что-либо, что лучше всего соответствует потребностям вашего приложения.
Далее в приложении для публикации постов в блоге мы будет использовать `Elasticsearch` (базу данных, ориентированную на документы) для реализации модели чтения.

<br>

```php
<?php
class PostsController
{
    public function listAction()
    {
        $client = new ElasticsearchClientBuilder::create()->build();
        $response = $client->search([
            'index' => 'blog-engine',
            'type' => 'posts',
            'body' => [
                'sort' => [
                    'created_at' => ['order' => 'desc']
                ]
            ]
        ]);
        return [
            'posts' => $response
        ];
    }
}
```

<br>

Код Модели Чтения был существенно упрощен до одного запроса к индексу Elasticsearch.

Этот код показывает, что Модель Чтения на самом деле не нуждается в объектно-реляционном преобразователе, посколько это может быть излишним.
Однако Модель Записи может выиграть от использования объектно-реляционного
преобразования, поскольку это позволит вам организовать и структурировать Модель Чтения в соответствии с потребностями приложения.

<br>

#### Синхронизация Модели Записи с Моделью Чтения

Здесь начинается сложная часть. Как мы синхронизируем Модель Чтения с
Моделью Записи? Мы уже говорили, что сделаем это с помощью Событий Домена,
захваченных в транзакции Модели Записи. Для каждого типа захваченного События Домена будет выполнена соответствующая проекция. Таким образом, будет установлено
взаимно-однозначное отношение между Событиями Домена и проекциями.

Давайте посмотрим на пример настройки проекций, для лучшего понимания идеи.
Прежде всего, нам нужно определить каркас для проекций:

<br>

```php
<?php
interface Projection
{
    public function listensTo();
    public function project($event);
}
```

<br>

Определение проекции `Elasticsearch` для события `PostWasCreated` достаточно просто:

<br>

```php
<?php
namespace Infrastructure\Projection\Elasticsearch;
use Elasticsearch\Client;
use PostWasCreated;
class PostWasCreatedProjection implements Projection
{
    private $client;
    public function __construct(Client $client)
    {
        $this->client = $client;
    }

    public function listensTo()
    {
        return PostWasCreated::class;
    }

    public function project($event)
    {
        $this->client->index([
        'index' => 'posts',
        'type' => 'post',
        'id' => $event->getPostId(),
        'body' => [
            'content' => $event->getPostContent(),
                // ...
            ]
        ]);
    }
}
```

<br>

Реализация Проекции является своего рода специализированным слушателем
Событий Домена. Основное различие между этой Проекций и слушателем Доменных Событий по умолчанию в том, что Проекция реагирует на группу Доменных Событий, а не только на одно.

<br>

```php
<?php
namespace Infrastructure\Projection;

class Projector
{
    private $projections = [];

    public function register(array $projections)
    {
        foreach ($projections as $projection) {
            $this->projections[$projection->eventType()] = $projection;
        }
    }

    public function project(array $events)
    {
        foreach ($events as $event) {
            if (isset($this->projections[get_class($event)])) {
                $this->projections[get_class($event)]
                    ->project($event);
            }
        }
    }
}
```

<br>

Следующий код показывает, как будет выглядеть поток между проекцией и событиями:

```php
<?php
$client = new ElasticsearchClientBuilder::create()->build();

$projector = new Projector();
$projector->register([
    new Infrastructure\Projection\Elasticsearch\
    PostWasCreatedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostWasPublishedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostWasCategorizedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostContentWasChangedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostTitleWasChangedProjection($client),
]);

$events = [
    new PostWasCreated(/* ... */),
    new PostWasPublished(/* ... */),
    new PostWasCategorized(/* ... */),
    new PostContentWasChanged(/* ... */),
    new PostTitleWasChanged(/* ... */),
];

$projector->project($event);
```

<br>

Этот код является своего рода синхронным, но процесс может быть и асинхронным, если это необходимо. И вы могли бы информировать своих клиентов об этих
не синхронизированных данных, разместив несколько предупреждений в слое представления.

В следующем примере мы будем использовать PHP-расширение amqplib в сочетании с
ReactPHP:

```php
<?php
// Connect to an AMQP broker
$cnn = new AMQPConnection();
$cnn->connect();

// Создание канала
$ch = new AMQPChannel($cnn);

// Declare a new exchange
$ex = new AMQPExchange($ch);
$ex->setName('events');
$ex->declare();

// Create an event loop
$loop = ReactEventLoopFactory::create();

// Создание поставщика, который будет отправлять
// любые ожидающие сообщения каждые полсекунды
$producer = new Gos\Component\React\AMQPProducer($ex, $loop, 0.5);
$serializer = JMS\Serializer\SerializerBuilder::create()->build();
$projector = new AsyncProjector($producer, $serializer);
$events = [
    new PostWasCreated(/* ... */),
    new PostWasPublished(/* ... */),
    new PostWasCategorized(/* ... */),
    new PostContentWasChanged(/* ... */),
    new PostTitleWasChanged(/* ... */),
];
$projector->project($event);
```

<br>

Чтобы это работало, нам нужен асинхронная проекция. Вот наивная реализация этого:

```php
<?php
namespace Infrastructure\Projection;
use Gos\Component\React\AMQPProducer;
use JMS\Serializer\Serializer;
class AsyncProjector
{
    private $producer;
    private $serializer;
    public function __construct(
        Producer $producer,
        Serializer $serializer
    ) {
        $this->producer = $producer;
        $this->serializer = $serializer;
    }
    public function project(array $events)
    {
        foreach ($events as $event) {
            $this->producer->publish(
                $this->serializer->serialize(
                    $event, 'json'
                )
            );
        }
    }
}
```

<br>

И потребитель событий с использованием брокера RabbitMQ будет
выглядеть примерно так:

```php
<?php
// Connect to an AMQP broker
$cnn = new AMQPConnection();
$cnn-> connect();

// Create a channel
$ch = new AMQPChannel($cnn);

// Create a new queue
$queue = new AMQPQueue($ch);
$queue->setName('events');
$queue->declare();

// Create an event loop
$loop = React\EventLoop\Factory::create();
$serializer = JMS\Serializer\SerializerBuilder::create()->build();
$client = new Elasticsearch\ClientBuilder::create()->build();

$projector = new Projector();
$projector->register([
    new Infrastructure\Projection\Elasticsearch\
    PostWasCreatedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostWasPublishedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostWasCategorizedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostContentWasChangedProjection($client),
    new Infrastructure\Projection\Elasticsearch\
    PostTitleWasChangedProjection($client),
]);

// Create a consumer
$consumer = new Gos\Component\ReactAMQP\Consumer($queue, $loop, 0.5, 10);

// Check for messages every half a second and consume up to 10 at a time.
$consumer->on(
    'consume',
    function ($envelope, $queue) use ($projector, $serializer) {
        $event = $serializer->unserialize($envelope->getBody(), 'json');
        $projector->project($event);
    }
);
$loop->run();
```

<br>

Отсюда все становится проще. Мы заставляем все Репозиторий использовать
экземпляр проекции, а затем так же запускаем процесс проецирования:

```php
<?php
class DoctrinePostRepository implements PostRepository
{
    private $em;
    private $projector;

    public function __construct(EntityManager $em, Projector $projector)
    {
        $this->em = $em;
        $this->projector = $projector;
    }

    public function save(Post $post)
    {
        $this->em->transactional(
            function (EntityManager $em) use ($post)
            {
                $em->persist($post);
                foreach ($post->recordedEvents() as $event) {
                    $em->persist($event);
                }
            }
        );
        $this->projector->project($post->recordedEvents());
    }

    public function byId(PostId $id)
    {
        return $this->em->find($id);
    }
}
```

<br>

Экземпляр `Post` и записанные события запускаются и сохраняются в одной транзакции. Это гарантирует, что никакие события не будут потеряны, так как мы спроецируем их на модель чтения, если транзакция прошла успешно.
В результате между Моделью Записи и Моделью чтения не будет никаких несоответствий.

<br>

> **ORM или без ORM**
>
> Один из наиболее распространных вопросов при реализации CQRS - действительно ли нужен объектно-реляционный маппер (ORM)?
> Мы твердо верим, что использование ORM для модели записи прекрасно и дает все преимущества использования инструмента, который поможет нам сэкономить много работы в случае использования реляционной базы данных.
> Но мы не должны забывать, что нам все еще нужно сохранять и извлекать состояние Модели Записи используя реляционную базу данных.

<br>

### Event Sourcing

CQRS - это мощная и гибкая архитектура. Это дает дополнительное преимущество в отношении сбора и сохранения Событий Домена (которые произошли во время выполнения операции Агрегата), предоставляя вам высокую степень детализации того, что происходит в вашем Домене. События в Домене являются одни из ключевых тактических паттернов из-за их значения в Домене, поскольку они описывают прошлые события.

<br>

> **Будьте осторожны с записью слишком большого количества событий**
>
> Постоянно растущее число событий - это звоночек. Это может выявить пристрастие к записи событий Домена, что, скорее всего, стимулируется бизнесом.

<br>

Используя CQRS, мы смогли бы записать все соответствующие события, которые произошли на уровне Домена.
Состояние доменной модели может быть представлено путем воспроизведения событий домена, которые мы записали ранее. Нам просто нужен инструмент для последовательного хранения всех этих событий. Нам нужно хранилище событий.

<br>

> Основная идея, лежащая в основе Event Sourcing, заключается в отображении состояния Агрегатов в виде линейной последовательности событий.

<br>

С помощью CQRS мы частично достигли следующего: сущность Post изменяет свое состояние с помощью событий Домена, но она сохраняется, как уже объяснялось, тем самым сопоставляя объект с строкой в таблицей базы данных.

`Event Sourcing` делает еще один шаг вперед. Сейчас мы использовали одну таблицу базы данных для хранения состояния всех постов блога, другую для хранения состояния всех комментариев постав блога и т.д., использование `Event Sourcing` мы можем использовать одну таблицу базы данных, в которой будут храниться все События Домена, опубликованные всеми агрегатами в Модели Домена. Да, вы прочитали это правильно. Единая таблица базы данных.

Следуя этому подходу, инструменты, подобные объектно-реляционному мапперу, больше не нужны. Единственным необходимым инструментом был бы простой уровень абстракции базы данных, к которому можно добавлять события:

<br>

```php
interface EventSourcedAggregateRoot
{
    public static function reconstitute(EventStream $events);
}

class Post extends AggregateRoot implements EventSourcedAggregateRoot
{
    public static function reconstitute(EventStream $history)
    {
        $post = new static($history->getAggregateId());
        foreach ($events as $event) {
            $post->applyThat($event);
        }

        return $post;
    }
}
```

<br>

Теперь у Агрегата `Post` есть метод, который при заданном наборе событий (другими словами, потоке событий) может пошагово воспроизводить состояние до тех пор, пока оно не достигнет текущего.
Следующим шагом будет создание адаптера порта для `PostRepository`, который будет извлекать все опубликованные события из Агрегата `Post` и добавлять их в хранилище данных, куда добавляются все событий. Это то что мы называем хранилищем событий (event store).

<br>

```php
class EventStorePostRepository implements PostRepository
{
    private $eventStore;
    private $projector;

    public function __construct($eventStore, $projector)
    {
        $this->eventStore = $eventStore;
        $this->projector = $projector;
    }

    public function save(Post $post)
    {
        $events = $post->recordedEvents();
        $this->eventStore->append(new EventStream(
            $post->id(),
            $events)
        );
        $post->clearEvents();
        $this->projector->project($events);
    }
}
```

<br>

Так выглядит реализация `PostRepository`, когда мы используем хранилище событий для сохранения всех событий, опубликованных Агрегатом `Post`. Теперь нам нужен способ восстановить Агрегат из его истории событий. Метод восстановления, реализуется Агрегатом `Post` и используется для восстановления состояния сообщения в блоге из
инициированных событий:

<br>

```php
class EventStorePostRepository implements PostRepository
{
    public function byId(PostId $id)
    {
        return Post::reconstitute(
            $this->eventStore->getEventsFor($id)
        );
    }
}
```

<br>

Хранилище событий это рабочая лошадка, которая несет на себе всю ответственность за сохранение и восстановление потоков событий. Его публичный API состоит из двух простых методов:
`append` и `getEventsFrom`. Первый добавляет поток событий в хранилище событий, а второй получает потоки событий, чтобы запустить построение Агрегата.

Мы могли бы использовать key-value хранилище для реализации хранения всех событий:

<br>

```php
class EventStore
{
    private $redis;
    private $serializer;

    public function __construct($redis, $serializer)
    {
        $this->redis = $redis;
        $this->serializer = $serializer;
    }

    public function append(EventStream $eventstream)
    {
        foreach ($eventstream as $event) {
            $data = $this->serializer->serialize(
                $event, 'json'
            );
            $date = (new DateTimeImmutable())->format('YmdHis');
            $this->redis->rpush(
                'events:' . $event->getAggregateId(),
                $this->serializer->serialize([
                    'type' => get_class($event),
                    'created_on' => $date,
                    'data' => $data
                ],'json')
            );
        }
    }

    public function getEventsFor($id)
    {
        $serializedEvents = $this->redis->lrange('events:' . $id, 0, -1);
        $eventStream = [];
        foreach($serializedEvents as $serializedEvent){
            $eventData = $this->serializer->deserialize(
                $serializedEvent,
                'array',
                'json'
            );
            $eventStream[] = $this->serializer->deserialize(
                $eventData['data'],
                $eventData['type'],
                'json'
            );
        }

        return new EventStream($id, $eventStream);
    }
}
```

<br>

Эта реализация хранилища событий основана на Redis, широко используемом key-value хранилище.
События добавляются в список с использованием префиксных событий: помимо этого, перед сохранением событий мы извлекаем некоторые метаданные, такие как класс события или дата создания, это может пригодиться позже.

Очевидно, что с точки зрения производительности, Агрегату очень затратно обходить всю историю событий, чтобы постоянно находиться в актуальном состоянии. Это особенно заметно, когда поток событий содержи сотни или тысячи событий.

Лучший способ преодолеть эту ситуацию - это сделать снимок Агрегата (snapshot) и воспроизвести только те события в потоке событий, которые произошли после создания снимка.
Снимок - это просто сериализованная версия состояния Агрегата, преимуществено основанный на времени. При одном подходе, снимок делается каждые n запущенный событий. При другом подходе снимок делается каждые n секунд.

Следуя нашему примеру, мы будем использовать первый способ создания снимков. В метаданных события мы храним даполнительное поле, версию, с которой мы начнем воспроизводить историю Агрегата.

<br>

```php
class SnapshotRepository
{
    public function byId($id)
    {
        $key = 'snapshots:' . $id;
        $metadata = $this->serializer->unserialize(
            $this->redis->get($key)
        );
        if (null === $metadata) {
            return;
        }

        return new Snapshot(
            $metadata['version'],
            $this->serializer->unserialize(
                $metadata['snapshot']['data'],
                $metadata['snapshot']['type'],
                'json'
            )
        );
    }

    public function save($id, Snapshot $snapshot)
    {
        $key = 'snapshots:' . $id;
        $aggregate = $snapshot->aggregate();
        $snapshot = [
            'version' => $snapshot->version(),
            'snapshot' => [
                'type' => get_class($aggregate),
                'data' => $this->serializer->serialize(
                    $aggregate, 'json'
                )
            ]
        ];

        $this->redis->set($key, $snapshot);
    }
}
```

<br>

Теперь нам необходимо провести рефакторинг класса `EventStore`, чтобы он начал использовать  `SnapshotRepository` для загрузки Агрегата с допустимыми временными затратами.

<br>

```php
class EventStorePostRepository implements PostRepository
{
    public function byId(PostId $id)
    {
        $snapshot = $this->snapshotRepository->byId($id);
        if (null === $snapshot) {
            return Post::reconstitute(
                $this->eventStore->getEventsFrom($id)
            );
        }

        $post = $snapshot->aggregate();
        $post->replay(
            $this->eventStore->fromVersion($id, $snapshot->version())
        );

        return $post;
    }
}
```

<br>

Нам просто нужно периодически делать снимки Агрегата. Мы можем делать это синхронно или асинхронно с помощью процесса, отвечающего за мониторинг хранилища событий. Следующий код представляет собой простой пример, демонстрирующий реализацию процесса создания снимка Агрегата:

<br>

```php
class EventStorePostRepository implements PostRepository
{
    public function save(Post $post)
    {
        $id = $post->id();
        $events = $post->recordedEvents();
        $post->clearEvents();
        $this->eventStore->append(new EventStream($id, $events));
        $countOfEvents =$this->eventStore->countEventsFor($id);
        $version = $countOfEvents / 100;
    
        if (!$this->snapshotRepository->has($post->id(), $version)) {
            $this->snapshotRepository->save(
                $id,
                new Snapshot(
                    $post, $version
                )
            );
        }

        $this->projector->project($events);
    }
}
```

<br>

> ***ORM или без ORM***
> Из представленного варианта использования архитектурного стиля ясно, что использование ORM только для сохранения/извлечения событий было бы излишним. Даже если мы используем реляционную базу данных для их хранения, нам нужно только сохранять/извлекать события из хранилища данных.

<br>

## Резюмируем

Поскольку существует множество вариантов архитектурных стилей, возможно, вы немного запутались в этой главе. Чтобы сделать выбор вы должны рассмотреть компромиссы для каждого из них. Ясно одно: подход Большой Комок Грязи - не вариант, так как код будет
очень быстро "портиться". Многоуровневая Архитектура является лучшим вариантом, но
она имеет некоторые недостатки, такие как тесная связь между слоями.
Можно утверждать, что наиболее сбалансированным вариантом будет Гексагональная
Архитектура, поскольку она может использоваться в качестве базовой архитектуры и обеспечивает высокую степень развязки и симметрии между внутренней и внешней частью приложения. Это то, что мы рекомендуем для большинства сценариев.

Мы также рассматриваем CQRS и EventSourcing как относительно гибкие архитектуры,
которые помогут вам в борьбе с высокой сложностью проекта.
CQRS и EventSourcing являются мощными подходами, но не позволяйте фактору
крутости отвлекать вас от ценности, которую они предоставляют. Поскольку они оба идут некоторыми накладными расходами, у вас должна быть техническая
причина для оправдания использования этих подходов. Эти архитектурные стили
действительно очень полезны, и эвристически узнать необходимость применения можно посчитав количество заявителей в репозитории CQRS и количеству инициированных событий для EventSourcing. Если число методов поиска начинает расти. а хранилища становятся сложными в обслуживании, то пришло время рассмотреть вопрос об использовании CQRS, чтобы разделить задачи чтения и записи. И после этого, если объем событий в каждом Агрегате имеет тенденции к росту, и бизнес заинтересован в более детальной информации, то можно подумать о том, может ли окупиться переход на EventSourcing.

PrettyForms
===========

Небольшая библиотека, благодаря которой можно легко сделать валидацию формы на клиентской и серверной сторонах.

Зависимости: jQuery.

Клиентская валидация опитимизирована для bootstrap-шаблонов, но вы можете легко изменить шаблоны вывода ошибок на свои собственные, изменив свойства объекта `PrettyForms.templates` после инициализации библиотеки.

![Screenshot](demo.gif)

####Как всё работает?
- Пользователь нажимает кнопку отправки формы. Библиотека проводит валидацию всех данных, и если всё нормально, она собирает все данные формы и отправляет POST-запрос на сервер и ожидает от него JSON-ответ в специальном формате.
- Сервер, получив запрос, проводит дополнительную валидацию данных уже на своей стороне. Если возникли ошибки при серверной валидации, он возвращает клиенту специальным образом сформированный JSON-ответ, содержащий команду для отображения ошибок серверной валидации с информацией о полях и содержащихся в них ошибках.
- Если данные успешно прошли валидацию и на сервере, сервер производит необходимые операции и возвращает JSON-ответ с командами, описывающими действия, которые клиентская машина должна выполнить после успешной обработки операции.

То есть, сервер всегда отвечает определённым набором команд для браузера, а браузер просто исполняет данные команды на клиенской машине. Таков алгоритм работы библиотеки.

####Установка

Через Composer, чтобы подключить к PHP-приложению серверные классы:

```shell
composer install believer-ufa/prettyforms
```

Через bower, чтобы подключить к внешней части сайта JS-библиотеку:

```shell
bower install prettyforms --save
```

####Использование библиотеки

Подключите JS-файл **prettyforms.js** на страницу сайта и добавьте атрибут валидации **data-validation=""** с списком правил валидации ко всем полям вашей формы. После добавления правил, поля формы автоматически станут валидироваться библиотекой PrettyForms. Если данные окажутся невалидными, библиотека не даст форме отправиться на сервер. Это минимальный функционал библиотеки, без связки с сервером.

Для того чтобы подключить библиотеку для работы с сервером, добавьте к вашей стандартной кнопке отправки формы класс **senddata**, благодаря которому клики по кнопке будут перехвачены и обработаны библиотекой. Теперь библиотека не только будет производить клиентскую валидацию, но и станет отвечать за отправку данных на сервер и обработку ответа.

Пример формы для Bootstrap-фреймворка с атрибутами валидации:

```html
<form class="form-horizontal" role="form" method="POST">
    <h1 class="form-signin-heading">Регистрация</h1>
    <div class="form-group">
        <label for="inputEmail3" class="col-sm-2 control-label">Email</label>
        <div class="col-sm-10">
            <input type="email"
                   class="form-control"
                   id="inputEmail3"
                   name="email"
                   data-validation="notempty;isemail"
                   placeholder="Введите ваш email">
        </div>
    </div>
    <div class="form-group">
        <label for="inputPassword3" class="col-sm-2 control-label">Пароль</label>
        <div class="col-sm-10">
            <input
                type="password"
                class="form-control"
                id="inputPassword3"
                name="password"
                data-validation="notempty;minlength:6"
                placeholder="Ваш пароль">
        </div>
    </div>
    <div class="form-group">
        <label for="inputPassword4" class="col-sm-2 control-label">Повторите пароль</label>
        <div class="col-sm-10">
            <input type="password"
                   class="form-control"
                   id="inputPassword4"
                   name="password_retry"
                   data-validation="notempty;passretry"
                   placeholder="Повторите пароль, вдруг ошиблись при вводе? Мы проверим это.">
        </div>
    </div>
    <div class="form-group">
        <div class="col-sm-offset-2 col-sm-10">
            <div type="submit"
                    data-input=".form-horizontal"
                    data-link="<?=URL::to('register')?>"
                    data-clearinputs="true"
                    class="btn btn-default senddata">Зарегистрироваться</div>
        </div>
    </div>
</form>
```


####Валидаторы полей

Правила валидации разделяются знаком **;**, параметры для правил передаются через знак **:**.
Пример корректного списка правил, содержащего два правила, одно из них с параметром: `"notempty;minlength:6"`

| наименование  | Описание      | Параметр|
| ------------- | ------------- |---------|
| notempty  | Поле не может быть пустым  | - |
| minlength  | Не менее {%} символов  | кол-во символов |
| maxlength  | Не более {%} символов  | кол-во символов |
| hasdomain  | Адрес должен начинаться с верного домена ({%})  | домен |
| isnumeric  | Поле может содержать только цифры  | - |
| isemail  | Должен быть введен корректный E-Mail  | - |
| isdate  | Поле должно содержать дату  | - |
| isphone  | Введён не корректный формат телефона  | - |
| minint  | Минимальное вводимое число {%}  | число |
| maxint  | Максимальное вводимое число {%}  | число |
| intonly  | Можно ввести только число  | число |
| passretry  | Должно быть равно полю с паролем  | наименование поля с паролем, по-умолчанию "password" |

####Добавление своих валидаторов
Вы можете легко добавить свои собственные валидаторы, используя подобный пример:
```javascript
PrettyForms.Validator.setValidator('needempty', 'Поле должно быть пустым!', function(element, value, param){
    // needempty - название валидатора
    // второй параметр - сообщение об ошибке
    // третий - это сама функция валидации, в которую передаются три параметра: jQuery-элемент, значение элемента и параметр валидатора, если он был передан в свойствах валидации
    return value === '';
});
```

####Дополнительные атрибуты валидации

Библиотека позволяет добавлять к полям некоторые дополнительные атрибуты, которые регулируют поведение проверки поля.

| атрибут       | Описание      | Обязательно?|
| ------------- | ------------- |-------------|
| data-dontsend="true"  | Не обрабатывать и не отправлять поле | Отключает проверку данного поля и его отправку на сервер |

####Атрибуты кнопки отправки формы

Кнопка, нажатие на которую генерирует отправку формы, должна также иметь несколько дополнительных атрибутов, объясняющих, куда должны быть отправлены данные, из какого DOM-элемента они должны быть собраны, и некоторые другие свойства поведения формы.

| атрибут       | Описание | Обязательно? |
| ------------- | ---------|--------------|
| data-input  | jQuery-селектор элемента, из которого будут собраны данные для отправки на сервер  | Нет, если не указано, то будет отправлен пустой запрос |
| href или data-link  | Адрес, на который будут отправлены данные  | Нет, по умолчанию данные будут отправлены на текущий URL страницы |
| data-clearinputs="true" | Очистить поля формы после успешного выполнения запроса?  | Нет |


####Валидация на сервере

Валидацией на сервере должен заниматься тот фреймворк, с которым вы работаете. Библиотека лишь предоставляет класс `Commands`, упрощающий создание корректного JSON-ответа с командами для клиента.

Но для Laravel были созданы специальные классы, благодаря которым работа библиотеки в данном фреймворке сводится к дописыванию нескольких строк кода, после чего валидация начинает работать.

Чтобы включить валидацию для какого-либо контроллера и модели Laravel, подключите к нужной модели трейт для подключения функционала валидации, а также опишите в ней массив с правилами валидации для полей:

````php
use PrettyForms\LaravelValidatorTrait;
class User extends Eloquent implements UserInterface, RemindableInterface {

	use UserTrait, RemindableTrait, LaravelValidatorTrait;

    //...

    private $rules = [
        'email'    => 'required|email|unique:users,email',
        'password' => 'required|min:6'
    ];

}
````

После этого, в коде обработки запроса, вам остается лишь заменить метод сохранения модели на новый: `validateAndSave()`, который автоматически будет производить валидацию данных перед отправкой, и если что-то пойдёт не так, выполнение будет остановлено, а клиенту будет отправлен ответ с командой отображения ошибок валидации сервера. А также вместо стандартного ответа подготовить и вернуть клиенту специальным образом сформированный JSON-ответ, пример можно увидеть ниже:

```php

    use PrettyForms\LaravelResponse;

    //...

    $user           = new User;
    $user->email    = Input::get('email');
    $user->password = Hash::make(Input::get('password'));
    $user->validateAndSave();

    return LaravelResponse::generate([
        'redirect' => '/login'
    ]);
```

Метод `LaravelResponse::generate` принимает ассоциативный массив из команд и их параметров, которые будут выполнены на клиенской машине. Изначально библиотека поддерживает лишь две команды: `validation_errors` и `redirect`. Первая команда используется библиотекой для отправки ошибок формы, вторая - для редиректа пользователя в другое место. Вы можете легко расширить функционал библиотеки, добавив свои собственные обработчики команд, посмотрев на примеры того, как это сделано в [оригинальном коде](https://github.com/believer-ufa/prettyforms/blob/master/prettyforms.js#L540)

####Формат протокола общения клиента с сервером

Клиентская библиотека ожидает от сервера ответ в следующем формате:
```javascript
[
  [
    "type" : "название команды"
    "data" : "любые данные, которые будут переданы команде"
  ],
  [
    "type" : "название второй команды"
    "data" : "данные"
  ]
  // и так далее
]
```
За генерацию подобного ответа и отвечает метод `Commands::generate`, принимающий ассоциативный массив.

Для отображения ошибок валидации, необходимо послать клиенту следующий ответ:
```javascript
[
  [
    "type" : "validation_errors"
    "data" : [
    	[
    	  "field"  : "название поля, в котором возникла ошибка"
    	  "errors" : ["текст ошибки №1", "текст ошибки №2"]
    	],
    	[
    	  "field"  : "название другого поля"
    	  "errors" : ["текст ошибки №1", "текст ошибки №2", "текст ошибки №2"]
    	]
    	//... и так далее
    ]
  ]
]
```
За генерацию подобных данных (именно данных, а не всего ответа) отвечает команда `Commands::generateValidationErrors`. Она также принимает ассоциативный массив, в котором ключами являются наименования полей, а значения - это массивы с сообщениями об ошибах.

####Проблемы

Библиотека не представляет из себя решение на все случаи жизни и не нацелена на подобное развитие. Это скорее небольшая заготовка, которую вы можете использовать и дорабатывать под свои нужды в своих собственных проектах.

В данный момент, одна из проблем - это невозможность собирать инпуты с теми типами, которые были описаны [в стандартах HTML5](http://www.w3schools.com/html/html_form_input_types.asp), хотя их поддержку можно легко добавить в библиотеку, но в данный момент у меня просто не возникало данной необходимости.

Одна из частых проблем - это трудности с получением содержимого из тех полей, к которым применён какой-то дополнительный плагин, вроде Chosen или CKEDitor. Конкретно для двух этих плагинов в библиотеке уже встроена поддержка, и она корректно получает значения из полей, связанных с данными плагинами, но в мире существуют тысячи других плагинов, с которыми она может работать некорректно. Следует учитывать это при использовании библиотеки.


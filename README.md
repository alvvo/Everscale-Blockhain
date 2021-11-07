# ITGOLD Best practices for writing Smart-Contracts in FreeTON Blockchain.

## Содержание
* [__Смарт-контракты:__](#smart_contracts)
    * [Стиль именования](#write_styles)
    * [Переменные](#states)
    * [Структуры](#struct)
    * [Функции](#functions)
    * [Интерфейсы](#interface) 
* [__Деботы:__](#debots)
    * [Инициализация и начало](#init_and_start)
    * [Геттеры](#getters)
    * [Хранение строк](#string_storage)
    * [Поддержка нескольких языков](#multi_language)
* [__Управление смарт-контрактами (IAO Access):__](#management)
    * [Initial Access](#initial_access)
    * [Admin Access](#admin_access)
    * [Owner Access](#owner_access)

***

<a id="smart_contracts"></a>

### Смарт-контракты

<a id="write_styles"></a>

#### Стиль именования:

* Имена смарт-контрактов, структур: __PascalCase__.
* Имена глобальных полей: ___camelCase__.
* Имена локальных полей, параметры, модификаторы: __camelCase__.
* Имена __constant__ полей записываются только в верхнем регистре с нижним подчеркиванием, например:
```
uint128 constant MINIMAL_MSG_VALUE = 0.3 ton;
```
* Имена интерфейсов начинаются с __I__, например:
```
interface IInterface{
    ...
}
```
<a id="states"></a>

#### Переменные:

* Переменные записываются группами для быстрого поиска и идентификации их в коде контракта, разделенные пустой строкой:
```
contract <имя_контракта>{

    // static поля
    address static <адрес_0>
    address static <адрес_1>

    // сonstant поля
    uint8 constant <адрес_0>
    uint8 constant <адрес_1>

    // Остальные поля
    address <переменная_0>
    address <переменная_1>

    TvmCell <cell_0>
    TvmCell <cell_1>

    constructor() public{
        ...
    }

}
```
<a id="struct"></a>

#### Структуры:

* Структуры позволяют нам пользоваться объектами внутри смарт-контракта, поэтому если объекты используются внутри системы контрактов, то следует вынести публичные структуры в интерфейс самого контракта:
```
interface IFirstContract{

    // Публичная структура:
    // Дает возможность пересылать ее контрактам, которые импортируют этот интерфейс.
    struct PublicObj{
        string name;
    }

    // Метод создания обьекта на основе структуры PublicObj, который реализует контракт FirstContract
    function saveObj(PublicObj obj) external;

}
```
```
import "IFirstInteface.sol";

// Приватная структура, используемая только внутри этого контракта
struct PrivateObj{
    string name;
}

// Контракт, который реализует интерфейс IFirstInteface
contract FirstContract is IFirstInteface{

    // Хэш-таблица, которая содержит как значение структуру PublicObj из интерфейса IFirstInteface
    mapping(uint8=>PublicObj) _publicObjects;
    // Приватная хэш-таблица
    mapping(uint8->PrivateObj) _privateObjects;

    constructor() public{
        tvm.accept();
    }

    // Реализация метода createObj из интерфейса IFirstInteface
    function saveObj(uint8 id, PublicObj obj) external override{
        tvm.accept();
        _publicObjects[id] = obj;
    }

}
```
```
import "IFirstContract.sol";

// Контракт импортирует, но не реализует интерфейс IFirstContract
contract SecondContract{

    // Хэш-таблица, которая содержит как значение структуру PublicObj из интерфейса IFirstInteface
    mapping(uint8=>PublicObj) _publicObjects;

    constructor() public {
        tvm.accept();
    }

    // Метод контракта SecondContract
    function createObj(uint8 id, string name) public {
        tvm.accept();
        _publicObjects[id] = PublicObj(name);
    }

    // Метод отправления созданной на основе интерфейса IFirstInteface структуры в контракт FirstContract.
    function sendObjToFirstContract(uint8 id, address dest) public{
        tvm.accept();
        IFirstContract(dest).saveObj{value: 0.3 ton, flag: 1}(id, _publicObjects[id]);
    }

}
```
<a id="functions"></a>

#### Функции:

* Функции геттеров объединяются в один метод, если отсутсвует необходимость использования отдельно, кроме ```responsible``` функций:
```
uint8 _a;
uint8 _b;

function getInfo() public returs(uint8 a, uint8 b){
    a = _a;
    b = _b;
}

function getC() external override responsible returns(uint){
    c = _a + _b;
    return{value: 0, flag: 64}(c);
}
```
<a id="interface"></a>

#### Интерфейсы:

* Каждый контракт, при наличии в нем функций, вызываемых через ```internal``` сообщения, должен реализовывать свой интерфейс.
* Если вы хотите скрыть некоторые функции интерфейса для другого смарт-контракта, то правильнее использовать локальные интерфейсы:
```
// Интерфейс контракта
interface IInterface{

    function a() external;
    function b() external;
    function c() external;

}
```
```
// Локальный интерфейс, скрывающий функционал другого контракта
interface IInterface{

    function b() external;

}

contract Contracts{

    constructor() public{
        tvm.accept();
    }

    // Отправка сообщения другому контракту по локальному интерфейсу
    function sendMsgToAnotherContract(address dest) public{
        tvm.accept();
        IInterface(dest).b{value: 0.3 ton, flag: 1}();
    }

}
```
***
<a id="debots"></a>

### Деботы

<a id="init_and_start"></a>

#### Инициализация и начало:
* Стартовая функция дебота не должна использоваться в качестве возврата к началу дерева, например:
```
contract debot is Debot{

    // Стартовая функция дебота при старте
    // Допускается инициализация переменных внутри этой функции
    // Вызывает функцию _start(), которую можно использовать в качестве возврата к началу дерева
    function start() public{
        init();
        _start();
    }

    function _start() public{
        ...
        goToMenu();
    }

}
```
* Инициализация переменных может происходить в функции ```init()```, и вызываться в стартовой функции.

<a id="getters"></a>

#### Геттеры:
* Функция, содержщая запрос какого-либо геттера, должна именоваться со слова ```query``` и описывать в названии запрашиваемое поле, например:
```
function queryOwnerAddress(uint32 success, uint32 error) public {

}
```
* Функции получения ответа должны именоваться начиная со слова ```on```, отображать запрос и статус ответа, например:
```
function onOwnerAddressQuerySuccess(address owner) public {
    // Успешный запрос
}

function onOwnerAddressQueryError(uint32 sdkError, uint32 exitCode) public {
    // Запрос с ошибкой, принимает в параметры коды ошибок
}
```
<a id="string_storage"></a>

#### Хранение строк
* Хранение пользовательских строк вывода лучше осуществлять в абстрактном классе или библиотеке, и наследовать их в деботе (строки обязательно должны быть помечены как ```сonstant```):
```
abstract contract strings{

    string constant string0 = "!@#";
    string constant string1 = "!@#";
    string constant string2 = "!@#";

}
```
```
import "res/strings.sol";

contract debot is Debot, strings{
    ...
}
```
<a id="multi_language"></a>

#### Поддержка нескольких языков
* Описаный выше способ не подойдет если дебот будет поддерживать несколько языков, поэтому чтобы сэкономить память при деплое дебота и впоследствии не отвлекаться на реализацию поддержки нескольких языков, можно создать большое количество смарт-контрактов, имеющих полностью одинаковые поля, но разные значения, и провести инициализацию всех переменных при старте дебота через запрос геттера соответствующей локали:
```
// Контракт одной из локалей
// Реализует интерфейс локали
contract EnglishLocale is ILocale{

    ...

    function getLocale() public returns(TvmCell locale){
        TvmBuilder builder;
        builder.store(...);

        TvmCell locale = builder.toCell();
    }

}
```
***
<a id="management"></a>

### Управление смарт-контрактами (IAO Access)

<a id="initial_access"></a>

#### Initial Access:

* Данным уровнем доступа обладают __владельцы__, либо __создатели__ смарт-контракта.
* Особенности данного уровня доступа:
    * Полный контроль над финансами смарт-контракта.
    * Право управления уровнем доступа __Admin Access__.
    * Возможность полного редактирования состояния смарт-контракта.

<a id="admin_access"></a>

#### Admin Access:

* Данным уровнем доступа обладает администратор, назначаемый от имени __initials__.
* Особенности данного уровня доступа:
    * Возможность частичного редактирования состояния, т.е. изменение полей, влияющих на функционирование и поведение смарт-контракта.

<a id="owner_access"></a>

#### Owner Access:

*  Данным уровнем доступа обладает пользователь, хранящий свои средства на некотором пользовательском смарт-контракте.
* Особенности данного уровня доступа:
    * Данный уровень доступа гарантирует владельцу(пользователю) что управлять его средствами может только он сам, либо группа владельцев(пользователей), т.е. не допускается использование уровней доступа __Initial Access__ и __Admin Access__ на пользовательских аккаунтах.
    * Возможность частичного редактирования состояния, т.е. изменение полей, влияющих на функционирование и поведение смарт-контракта.
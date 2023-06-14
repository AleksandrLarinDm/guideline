# Veteran's Legion code style guideline
Даний гайдлайн створено для уніфікації код-стилю та структури проекту.
## Folder structure
В даному проекті пропонується слідувати принципу **папка-для-модуля** і **папка-за-типом**.
Це означає що іменування папки означає ім'я модулю (**функції**). На приклад:
```
|-- common/
|-- authentication/
|-- settings/
|-- utils
```
*WARNING!* common папка використовується **ЛИШЕ** для віджетів, функцій, розширень і т.д., які перевикористовуються в **ЗНАЧНІЙ** частині проекту.
Тобто, якщо базовий віджет тексту ми використовуємо в 10+ місцях, то є сенс додати його у common. Але, якщо це *single-use* віджет / функція, не робіть цього. 
**EXAMPLE**
```
String toDtoString() {
    final formatter = DateFormat(dtoFormatString);
    if (isUtc) {
      return formatter.format(this);
    } else {
      return formatter.format(toUtc());
    }
  }
```

## Internal folder structure
Створивши папку, і назвавши її, наприклад, *authentication*, ми можемо наповнювати її кодом. Структура всередині папки виглядатиме так:

```
|-- ui/
|-- cubits/
|-- business_objects/
|-- exceptions/
|-- use_cases
|-- services/
|-- repositories/
|-- data_sources/
|-- dtos/
|-- authentication.dart
```

*WARNING!* Вище наведено повний приклад, але в деяких випадках щось з наведеного вище може не знадобитись, наприклад *services*.

## User interface (UI)
Код в даній папці відповідає за інтерфейс користувача, такі які: компоненти, екрани, форми і т.д.
### Screen
*Screen* це компонент UI, який містить в собі інтерфейс користувача та його компоненти. Також, screen являється елементом навігації в застосунку.
>**Як описувати screen:**
>>Ім'я файлу та назва класу **повинна** бути з суфіксом Screen.

>>Screen-класи повинні взаємодіяти лише з *cubit*-класами.
```dart
class LoginScreen extends StatelessWidget {
  //...fields and constructor...
  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        LoginView(),
      ],
    );
  }
}
```
### View
*View* є частиною віджета *Screen*, та наповнює його контентом. В основному, користувач взаємодіє з view, а не з screen, тому *View* являє собою основним елементом користувальницького інтерфейсу та надає змогу повної взаємодії з ним.
*View* містить в собі елементи інтерфейсу, такі як *Button, Form, TextField, Container, Text*, тощо.
>**Як описувати view:**
>>Ім'я файлу та назва класу **повинна** бути з суфіксом View.

>>View-класи повинні взаємодіяти лише з *cubit*-класами.

>>В середині view можуть бути лише **ПРИВАТНІ** віджети, якщо вони не використовуються з common.


*Example*

```dart

part 'login_view_components.dart';

class LoginView extends StatelessWidget {
  //...fields and constructor...
  @override
  Widget build(BuildContext context) {
    return Column(
      children: <Widget>[
        _LoginLogo,
        _LoginForm,
        _LoginActions,
        _LoginFooter
      ],
    );
  }
}

```

Непоганою практикою є створення приватних віджетів в окремому файлі, аби код був чистий так лаконічний, без зайвих віджетів.

```dart
part of 'login_view.dart';

class _LoginLogo extends StatelessWidget {}


class _LoginForm extends StatefullWidget {}


class _LoginActions extends StatelesssWidget {}
```

**ДУЖЕ ВАЖЛИВО**

**НЕ ПЕРЕДАВАТИ cubit В КЛАСИ ЧЕРЕЗ КОНСТРУКТОР, ЗАМІСТЬ ЦЬОГО ВИКЛИКАТИ ЙОГО ЧЕРЕЗ *BlocProvider/BlocBuilder/BlocConsumer***
```dart
class LoginScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return BlocBuilder<LoginCubit, LoginCubitState>(
      // Here cubit is not specified either.
      builder: (BuildContext context, LoginCubitState state) {},
    );
  }
}
```

## Cubits

[Cubits](https://pub.dev/packages/flutter_bloc) являють собою класи, які безпосередньо взаємодіятимуть з UI на стороні бізнес-логіки. Вони з'єднані з станом застосунку, та будуть відображати його у UI.

>**Яким повинен бути cubit:**
>>Покривати бізнес-логіку лише для UI.

>>Повертати Future з своїх функцій.

>> Взаємодіяти лише з high-level класами, мається на увазі use_case.

>>Закінчуватись на суфікс Cubit.

### Cubit's state.

Цей клас потрібен для відображення певного стану застосунку на інтерфейсі. Він тісно пов'язаний з UI та cubit.


## Use Cases
Даний клас використовується і створюється для виконання певної операції/задачі/роботи.

>**Рекомендації щодо створення use_case**

>>UseCase повинен закінчуватись на суфікс UseCase, та починатись на префікс з дуже стислим описом своєї задачі, наприклад:
>>```dart
>>GetCurrentAccoutUseCase
>>```
>>```dart
>>SignInUseCase
>>```

>>UseCase завжди містить в собі лише одну публічну функцію call(), повертаємий тип якої є результатом роботи цієї функції.

>>UseCase має доступ лише до ```Repositories``` або ```Services```, тобто ```high-level coordinators```.

>>UseCase не може взаємодіяти з ```DataSource``` або з іншими ```low-level objects```

>>UseCase повинен використовуватись лише в ```Cubit```.

### Example:
```dart
class GetCurrentUserUseCase {
	final UserRepository _repository;
	const GetCurrentUserUseCase(this._repository);
	Future<User?> call() async {
		await _repository.getCurrentUser();
	}
}
```


## Repositories

Класи, що дають доступ до даних, отриманих з локального сховища або іншого постачальника даних (API, Firebase, AWS).

>**Рекомендаціі**

>>Клас повинен містити суфікс Repository в своєму імені

>>Створювати Repository лише в межах модулю та використовувати його лише в ньому


### Example:

```dart
class UserRepository {
  Future<User> getCurrentUser();
}
```

## Data Sources
Класи, що дають нам змогу взаємодіяти з постачальником даних (локальним або на стороні серверу).

**Важливо**

DataSource являє собою інтерфейс або ж абстрактний клас.

>**Рекомендаціі**

>>Клас повинен містити суфікс DataSource в своєму імені

>>Створювати DataSoruce лише в межах модулю та використовувати його лише в ньому

>>DataSource взаємодіє лише з Repository

### Example:

```dart
abstract class UserDataSource {
  Future<User> getCurrentUser();
}
```

```dart
class ApiUserDataSource impements UserDataSource{
  @override
  Future<User> getCurrentUser(){
  // implementation goes here
  }
}

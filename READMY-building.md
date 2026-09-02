## Сборка артефактов для мехонизма репликации.

Для работы механизма репликации необходим артефакт расширения cdc-exe. Расширение реализовано в виде модуля в проекте ignite-extensions (https://github.com/apache/ignite-extensions). Нашей командой сделан форк, доработан и выложен в GitLab App.Farm (https://gitlab.rshbdev.ru/rshbintech/retail/cards/processing/ignite/cdc/ignite-extensions.git). Доработки включают работу с Ignite 2.15, которая по умолчанию не поддерживалась в расширение cdc-ext из-за разных интерфейсов java и версии компиляции (по умолчанию Java 11, Ignite 2.15 на Java 8).

### Сборка из оригинального ignite-extensions

Данное расширение создано для версий выше Ignite 2.15 и компилируется на Java 11.

1. Склонировать проект 

> git clone https://github.com/apache/ignite-extensions.git

2. Перейти в корень проекта ignite-extensions

> cd C:\Users\Ishkhan\IdeaProjects\ignite-extensions\

3. Команда для сборки артефактов

> mvn clean install -DskipTests

4. В процессе сборки будут возникать ошибки. Устранять ошибки в точности по выведенным ошибкам, к примеру, изменить версию библиотеки в pom.xml и т.д. За 3-4 итерации ошибки уйдут.

### Сборка форка ignite-extensions

#### Для Ignite 2.15 и ниже.

1. Склонировать проект

> git clone https://gitlab.rshbdev.ru/rshbintech/retail/cards/processing/ignite/cdc/ignite-extensions.git

2. Перейти в корень проекта ignite-extensions

> cd C:\Users\Ishkhan\IdeaProjects\ignite-extensions\

3. Перейти в ветку

> git checkout 2.15.0

4. Команда для сборки артефактов

> mvn clean install -DskipTests

#### Для версий выше Ignite 2.15.

1. Склонировать проект

> git clone https://gitlab.rshbdev.ru/rshbintech/retail/cards/processing/ignite/cdc/ignite-extensions.git

2. Перейти в корень проекта ignite-extensions

> cd C:\Users\Ishkhan\IdeaProjects\ignite-extensions\

3. Перейти в ветку

> git checkout 2.19.0

4. Команда для сборки артефактов

> mvn clean install -DskipTests




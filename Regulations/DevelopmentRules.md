# Правила разработки 

#### Создание/изменение объекта (таблицы) [EntitySchemaManager]

###### Общие положения

Для объектов, ссылающихся на DB View, а не на физическую таблицу, обязательно наличие флага “Представление в базе данных”. При этом, если в другом объекте создаётся колонка-справочник, ссылающаяся на объект с этим признаком, то для такой колонки необходимо установить признак “Не контролировать целостность”.

###### Встроенный процесс

1.	Без особой необходимости не следует менять автогенерируемые названия событий. В версиях bpm’online  ниже 7.10.3 необходимо к этим названиям добавлять хэш для предотвращения появления warning-сообщений на этапе компиляции.
    ```javascript
    Пример: SlPartnerChecklistSaving_41543abe-f9ca-411f-80e4-3803260a9d84
    ```
2.	Для каждого события в рамках встроенного процесса необходимо создавать Событийный подпроцесс, в который помещаются все элементы, обрабатывающие данное событие. При этом для названия и подписи элемента Событийный подпроцесс следует выбрать сокращённую форму названия обрабатываемого события.
Пример: OnSaving, OnSaved, OnDeleting, OnDeleted, …
    ```javascript
    Пример: OnSaving, OnSaved, OnDeleting, OnDeleted, …
    ```
3.	По возможности, все подпроцессы, элементы и стрелки переходов должны быть выровнены друг относительно друга, как в горизонтальной, так и вертикальной плоскости, с примерно одинаковыми расстояниями между соседними блоками (см. рис. ниже).
 	
    ...
    
4.	При необходимости добавления элементов “Задание-сценарий” каждый логический блок разрабатываемого функционала оформляется в виде отдельного элемента “Задание-сценарий”. Таким образом, выполнение каждого из логических блоков не будет зависеть от остальных.

5.	При написании кода внутри элемента “Задание-сценарий”, а также при создании методов (вкладка “Methods”) необходимо придерживаться стиля, в соответствии с которым ведётся разработка схем исходного кода на данном проекте.


#### Создание/изменение клиентского модуля [ClientUnitSchemaManager]

###### Порядок следования логических блоков в клиентском коде

1.	При создании новой клиентской схемы или изменении уже существующей необходимо соблюдать следующий порядок следования логических блоков возвращаемого модулем объекта:
 
    *	entitySchemaName
    *	mixins
    *	messages
    *	details
    *	attributes
    *	diff
    *	methods
    *	rules

2.	В секции attributes первыми должны идти атрибуты, связанные с колонками таблицы базовой сущности, далее атрибуты, используемые в качестве виртуальных вычисляемых колонок, отображаемых в интерфейсе, далее - вспомогательные атрибуты.
3.	Для атрибутов, не связанных с колонками таблицы базовой сущности, обязательно указание свойств dataValueType (тип атрибута) и value (начальное значение).
4.	При добавлении контролов в секцию diff требуется руководствоваться следующими правилами:
    * В рамках секции diff в новых схемах при помощи комментариев необходимо выделять группы контролов с одинаковой операцией. Порядок следования контролов в зависимости от операции, а также необходимые комментарии приведены в табл. ниже:
 
       ```javascript
       Порядок следования	Операция	Комментарий перед группой
      	 	1				remove		//REMOVE
       	 	2				merge		//MERGE
      	 	3				insert		//INSERT
       ```

    * При добавлении нового контрола в секцию diff необходимо соблюдать порядок, в соответствии с которым контролы будут отображены в интерфейсе. При этом группа дочерних контролов должна следовать непосредственно после родительского контейнера, который в свою очередь должен следовать после своего родительского и т.д., тем самым образуя иерархию.
    
	* Каждый из контролов с propertyName=”tabs” необходимо обозначать следующим комментарием:
	
      ```javascript
      //Tab
      ```
      
5.	Методы необходимо располагать в следующем порядке: первыми идут методы, унаследованные от базового объекта данной клиентской схемы (при этом внутри данной группы желательно соблюдать хронологический порядок, в котором происходит вызов методов), затем методы, унаследованные от модулей, перечисленных в зависимостях данного модуля, затем все остальные. Между соседними методами должен присутствовать отступ в виде пустой строки.

###### Общие правила разработки клиентских схем
1.	При написании кода необходимо придерживаться стиля, в соответствии с которым ведётся разработка клиентских схем на данном проекте.
2.	В рамках клиентских схем реестров, карточек редактирования, деталей необходимо реализовывать только логику, которая касается непосредственно отображения объекта в интерфейсе. Реализацию сложной бизнес-логики (например, операции над другими сущностями) следует выносить на серверную часть: во встроенный процесс объекта, схему исходного кода либо бизнес-процесс.
3.	Перед каждым методом, унаследованным от базового объекта данной клиентской схемы, обязательно наличие комментария вида:

     ```javascript
    /**
    * overridden
    */
     ```

    Если метод унаследован от объекта, стоящего по иерархии выше базового объекта данной схемы, то комментарий должен выглядеть следующим образом:

    ```javascript
    /**
    * overridden <название клиентской схемы, из объекта которой наследуется метод>
    */
    
    Пример: 

    /**
    * overridden BaseDetailV2
    */
    ```

4.	Если в рамках переопределяемого метода осуществляется вызов соответствующего метода из базового объекта (при помощи this.callParent()), а также присутствует какой-либо другой код, то этот код следует располагать после вызова this.callParent(). Исключение: случаи, когда явно необходимо, чтобы добавленный код выполнился перед вызовом метода базового объекта.
5.	В рамках переопределения метода init следует воздерживаться от непосредственного размещения больших блоков кода внутри данного метода. Весь “кастомный” функционал необходимо вынести в отдельные методы, которые уже вызывать внутри init.
6.	При подписке на события внутри методов во избежание множественного срабатывания обработчика следует воздерживаться от размещения этой операции в рамках методов, вызываемых не из метода init.
7.	При наличии большого числа методов, обращающихся к БД, по возможности следует вынести эти методы в отдельный клиентский модуль.
8.	Запрещается использовать константы типа GUID без их предварительного определения в коде. Имена таких констант должны чётко отражать те значения, которые они представляют. 

    ```javascript
    Пример (id записи справочника “Тип контрагента”=”Партнер”):

    var accountType__Partner = "f2c0ce97-53e6-df11-971b-001d60e938c6";
    ```
9.	Для реализации логики по доступности, видимости и обязательности контролов следует использовать бизнес-правила (модуль BusinessRule). Реализация указанной логики другими способами (в рамках свойств самого контрола в секции diff, либо в рамках соответствующего атрибута) выполняется только в том случае, если требуемые условия из-за их сложности невозможно  реализовать при помощи бизнес-правил.
10.	Запрещается использовать строковые константы в заголовках контролов, выводимых сообщениях и т.д. Для этого необходимо определить нужное строковое значение в локализуемых ресурсах схемы и в рамках кода использовать именно его.
11.	Без крайней необходимости не следует использовать sandbox. Для передачи значений атрибутов из карточки редактирования в деталь следует использовать атрибут “DefaultValues” (например, при помощи методов getDefaultValues() и getDefaultValueByName()).
12.	Запрещается оставлять в релизной версии функционала отладочный код вида console.log(“…”), debug и др., кроме случаев, когда это явно предусмотрено в рамках условия соответствующей задачи.
13.	Следует воздерживаться от “комментирования” строк кода, которые предположительно больше не будут использоваться. Такие строки следует сразу удалять.


#### Создание/изменение схемы исходного кода [SourceCodeSchemaManager]
1.	При написании кода необходимо придерживаться стиля, в соответствии с которым ведётся разработка схем исходного кода на данном проекте.
2.	Следует воздерживаться от “комментирования” строк кода, которые предположительно больше не будут использоваться. Такие строки следует сразу удалять.

#### Создание/изменение бизнес-процесса [ProcessSchemaManager]
1.	По возможности, все элементы и стрелки переходов должны быть выровнены друг относительно друга, как в горизонтальной, так и вертикальной плоскости, с примерно одинаковыми расстояниями между соседними блоками (см. рис. ниже).
2.	Из каждого элемента группы “Логические операторы”, наряду со стрелками “Условный поток”, необходимо выводить стрелку “Поток по умолчанию”, по которому продолжится выполнение процесса в случае, если ни одно из условий в остальных условных потоках не выполнится.

#### Настройка интерфейса
Для всех новых реестров и деталей с реестром необходимо выполнить предварительную настройку отображения колонок (Вид – Настроить колонки) в соответствии с постановкой задачи (в случае, когда состав колонок и/или порядок их отображения не указаны, - настроить их по своему усмотрению). После выполнения данной операции необходимо сохранить данные настройки для всех пользователей системы при помощи соответствующей опции (в рамках кнопки “Сохранить”).

#### Подготовка разработанного функционала к релизу
1.	Для всех новых системных настроек должны быть установлены начальные значения.
2.	При добавлении нового элемента типа “Данные” необходимо выбирать опцию “Установка” вместо “Первичная установка”.
3.	В зависимости от того, что конкретно было реализовано в рамках функционала, подготавливаемого к переносу, в поставке обязательно наличие следующих элементов типа “Данные”:
    *	наполнение добавленных/изменённых справочников (необходимо включать только те записи, которые были добавлены/изменены);
    *	регистрация справочников (новые записи из таблицы Lookup);
    *	группы справочников (новые записи из таблицы LookupFolder);
    *	справочники в группе (новые/изменённые записи из таблицы LookupInFolder);
    *	системные настройки, значения системных настроек (новые/изменённые  записи из таблиц SysSettings, SysSettingsValue);
    *	группы системных настроек (новые записи из таблицы SysSettingsFolder);
    *	системные настройки в группе (новые/изменённые  записи из таблицы SysSettingsInFolder);
    *	настройки отображения реестров и деталей для всех пользователей (новые/изменённые  записи из таблицы SysProfileData со значением ContactId = null);
    *	привязки карточек редактирования к объектам (новые/изменённые  записи из таблицы SysModuleEdit);
    *	детали (новые/изменённые  записи из таблицы SysDetail);
    *	разделы (новые/изменённые  записи из таблицы SysModule);
    *	привязки разделов к объектам (новые/изменённые  записи из таблицы SysModuleEntity);
    *	раздел в рабочем месте (новые/изменённые  записи из таблицы SysModuleInWorkplace);
    *	иконки разделов (новые/изменённые  записи из таблицы SysImage).


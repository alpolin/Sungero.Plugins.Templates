# Sungero.Plugins.Templates
В репозитории находятся шаблоны проектов для разработки плагина подписания к системе Directum RX.
Шаблоны проектов созданы в Microsoft Visual Studio 2017.

Базовая информация об использовании электронной подписи приведена в документации к системе.
Плагины подписания поддерживаются, начиная с Directum RX 3.3. Ограничения:
* Плагины подписания не поддерживаются в десктоп-клиенте.
* Клиентский плагин подписания поддерживается в Microsoft Windows 7 и выше, Microsoft Windows Server 2008 R2 и выше.

Совместимость с версией платформы Sungero:
* Ветка master - с Sungero 4.1 и выше. 
* Ветка 4.0 - с Sungero 3.3.8.0019 и до Sungero 4.1. 
* Ветка 3.3.8.0018 - с Sungero версии 3.3 до 3.3.8.0018 включительно.

### Как разработать плагин подписания
1. В проектах `ServerCryptographyPlugin`, `ClientCryptographyPlugin` измените имя сборки на свое (например, на `MyServerCryptographyPlugin`, `MyClientCryptographyPlugin`)
2. Сгенерируйте уникальный идентификатор (GUID) плагина (например на сайте https://www.guidgenerator.com/), пропишите его в свойстве `CryptographyPlugin.Id` класса серверного плагина и в файле `ClientPlugin.targets` клиентского плагина;
3. Реализуйте **серверный плагин**. Для этого: 
    * Реализуйте методы класса `Signer`: `SignData()`, `TryLoadPrivateKey()` и `VerifySignature()`. При необходимости модифицируйте остальные методы класса.
    * При необходимости модифицируйте методы класса `CryptographyPlugin`. Укажите нужный идентификатор алгоритма подписания в данном классе (поле `SignAlgorithmId`).
    * При необходимости создайте свой алгоритм хеширования с помощью класса `HashAlgorithm`.
    * Взаимодействие между классами описано в начале модуля `CryptographyPlugin.cs`.
4. Реализуйте **клиентский плагин** либо **плагин облачного подписания**. Для этого:
    * Для реализации клиентского плагина модифицируйте методы класса `Signer`.
	* Для реализации плагина облачного подписания модифицируйте методы класса `CloudCryptographyPlugin`	
5. Соберите проект. В папке *out* в корне проекта появятся папки *Client* и *Server*, содержащие файлы клиентского и серверного плагинов соответственно.
6. Дополнительные библиотеки, требующие распространения вместе с плагином, включите в соответствующий проект. Они автоматически должны попасть в zip-архив. 
7. Подключите серверный плагин к Directum RX:
    * Создайте папку для хранения плагина, например, *D:\Plugins*. При обновлении системы Directum RX может изменяться содержимое ее папок. Поэтому рекомендуется создать отдельную папку.
    * В конфигурационных файлах *_ConfigSettings.xml* всех серверных компонент в параметре PLUGINS_ZIP_PATH укажите путь к папке с плагинами, например:  
    ```<var name="PLUGINS_ZIP_PATH" value="D:\Plugins" />```
    * Скопируйте архив с серверным плагином в указанную папку (при работе серверных компонент под Windows - архив из папки `\out\Server\netframework`, под Linux - из папки `\out\Server\netstandard`)
    * Если необходимо передать дополнительные настройки в плагин, укажите их в тех же конфигурационных файлах *_ConfigSettings.xml*, где производилась настройка пути к папке с плагинами. Формат секции с настройками: 
      ```XML
      <block name="PLUGINS">
        <plugin id="<ид_плагина>"
          exampleSetting="Example value"
          otherSetting="Other value" />
      </block>
      ```
    Чтение настроек в плагине выполняется в методе `CryptographyPlugin.ApplySettings()`.
	Плагин облачного подписания также является серверным плагином, поэтому подключается и настраивается аналогично.
8. При необходимости подключите клиентский плагин к Directum RX. Клиентский плагин используется в веб-агенте при работе веб-клиента Directum RX. Для подключения:
    * Скопируйте файлы из папки *out\Client* в папку плагинов веб-агента на сервере приложений, например, в  
    ```C:\inetpub\wwwroot\Client\content\WebAgent\plugins\```
    * Запустите утилиту *packages_manifest_updater.exe* из папки *PackagesManifestUpdater* веб-агента на сервере приложений, например, из  
    ```C:\inetpub\wwwroot\Client\content\WebAgent\PackagesManifestUpdater```
	
### Особенности формирования и проверки электронной подписи в Directum RX
* Электронная подпись формируется в формате CAdES-BES.
* При подписании сертификатом в веб-агенте само подписание выполняется на стороне клиента, но данные для подписи (подписываемые атрибуты подписи) формируются на сервере приложений.
* Сервер приложений работает в 64-битном окружении, а веб-агент - в 32-битном. Это необходимо учитывать, если для работы плагинов нужны COM-компоненты и их регистрация.
* При необходимости подписание может быть выполнено на стороне сервера. В этом случае закрытый ключ должен быть доступен на сервере.

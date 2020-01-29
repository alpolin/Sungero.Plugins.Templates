# Sungero.Plugins.Templates
В этом репозитории находятся шаблоны проектов для разработки плагина подписания к системе DirectumRX.

### Как разработать плагин подписания
Шаблоны проектов созданы в Visual Studio 2017. Для разработки нового плагина подписания необходимо:
1. Переименовать проекты `ServerCryptographyPlugin`, `ClientCryptographyPlugin` в соответствии с назначением плагина.
2. Для реализации **серверного плагина** необходимо: 
    * Реализовать методы класса `Signer`: `SignData()`, `TryLoadPrivateKey()` и `VerifySignature()`. При необходимости модифицировать остальные методы класса;
    * Задать идентификатор плагина, сгенерировав уникальный Guid для свойства `CryptographyPlugin.Id`;
    * При необходимости модифицировать методы класса `CryptographyPlugin`. Указать нужный идентификатор алгоритма подписания в данном классе (поле `SignAlgorithmId`);
    * При необходимости создать свой алгоритм хеширования с помощью класса `HashAlhorithmExample`.
3. Для реализации **клиентского плагина** необходимо:
    * Задать идентификатор плагина в `ClientPlugin.targets`, совпадающий с идентификатором серверного плагина;
    * Модифицировать методы класса `Signer`. Указать нужный идентификатор алгоритма подписания в данном классе (поле `AlgorithmId`).
4. Собрать проект. В папке *out* в корне проекта появятся папки *Client* и *Server*, содержащие файлы клиентского и серверного плагинов соответственно.
5. Дополнительные библиотеки, требующие распространения вместе с плагином, должны быть включены в соответствующий проект (они автоматически должны попасть в zip-архив). 
6. Подключение серверного плагина к DirectumRX:
    * Создать папку для хранения плагина, например, *D:\Plugins*. Папку лучше создавать не внутри папок DirectumRX, чтобы избежать возможности ее удаления при обновлении системы.
    * В файлах настроек *_ConfigSettings.xml* всех серверов приложений и всех служб нужно явно задать настройку PLUGINS_ZIP_PATH, где указать путь к папке с плагинами, например:  
    ```<var name="PLUGINS_ZIP_PATH" value="D:\Plugins" />```
    * Скопировать архив с серверным плагином в указанную папку.
    * При необходимости передачи конфигурационных параметров в плагин, настройки нужно прописать в тех же файлах *_ConfigSettings.xml*, где производилась настройка пути к папке с плагинами. Формат блока с настройками следующий: 
      ```
      <block name="PLUGINS">
        <plugin id="<ид_плагина>"
          exampleSetting="Example value"
          otherSetting="Other value" />
      </block>
      ```
	  Чтение настроек выполняется в методе `CryptographyPlugin.ApplySettings()`.
7. Подключение клиентского плагина к DirectumRX. Клиентский плагин используется в веб-агенте при работе веб-клиента DirectumRX:
    * Скопировать файлы из папки *out\Client* в папку плагинов веб-агента на сервере приложений, например, в  
    ```C:\inetpub\wwwroot\Client\content\WebAgent\plugins\```
    * Запустить *packages_manifest_updater.exe* из папки *PackagesManifestUpdater* веб-агента на сервере приложений, например, из  
    ```C:\inetpub\wwwroot\Client\content\WebAgent\PackagesManifestUpdater```

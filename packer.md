## Packer - собираем нужный образ в любом месте (локально, в облаке, vagrant).
Суть `Packer` - подготовить такой образ, на котором есть необходимый минимум, в основном это ОС и рантайм, но так же можно добавлять любые операции, включая `git clone...`, bash-скрипты, ansible-роли и уже на этом образе разворачивать приложения.
### Базовый образ должен содержать:
- Настройки ОС
- Пакеты, с низкой частотой изменения (Nodejs, python, ruby, mongodb...)
- Логи, мониторинг, агенты систем управления конфигурацией
### Подход - Immutable Infrastructure
- Образ VM - артефакт для деплоя
- Не изменяем запущенные инстансы
- Любые изменения: собираем новый образ
- Старый инстанс заменяем на новый
### Packer

[Download](https://www.packer.io/downloads.html)

`$ packer -v`

Для управления сторонней инфраструктурой необходимо в ней авторизоваться

например `Google cloud`:

`$ gcloud auth application-default login`

### Переменные описываются в файле `variables.json`
[Gist с примером файла переменных](https://gist.github.com/mixassio/facbce156f56149f59fdcecd7e24b93d)

### Конфигурация образа описывается в json-файле
Содержит разделы:
- builders (настройки сборки, где - облако или локально, что - имя образа и базовый образ ОС, как - ssh)
- variables (переменные)
- provisioners (различные действия по установке или настройке на ОС)

[Gist на пример конфига с shell](https://gist.github.com/mixassio/9f064fd911be095d43d9076764797801)

[Gist на примере конфига с ansible](https://gist.github.com/mixassio/8aee798ac755d66ad777e4978cdc38da)

### запуск с файлом переменных:
```
$ packer build -var 'project_id=aaaa-123' packer_example.json
$ packer build -var-file variables.json packer_example.json
```
### валидация
```
$ packer validate -var 'project_id=aaaa-123' packer_example.json
Template validation failed. Errors are shown below.
Errors validating build 'googlecompute'. 1 error(s) occurred:
* a source_image or source_image_family must be specified
```
### Inspect
```
$ packer inspect packer_example.json
Required variables:
  project_id
Optional variables and their defaults:
  machine_type = f1-micro
  source_image = ubuntu-1804-bionic-v20181003
  zone         = europe-wes1-b
Builders:
  googlecompute
Provisioners:
  shell
```
### Build
```
packer build packer_example.json
```
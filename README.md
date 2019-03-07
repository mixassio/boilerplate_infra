# boilerplate_infra


Репозиторий с примерами инфраструкторного кода, типовыми решениями.
Список общей литературы:
- Проект "Феникс". Роман о том, как DevOps меняетт бизнес к лучшему (Джин КимБ Кевин Бер, Джордж Спаффорд)
- Открывая организации будующего (Фредерик Лалу)
- Lean Startup
- Lean Enterprise
- Practical Monitoring https://www.practicalmonitoring.com
- Refactoring Databases: Evolutionary Database Design
- Trunk Based Development https://trunkbaseddevelopment.com
- Статьи Мартина Фаулера https://martinfowler.com/design.html
- Топологии http://web.devopstopologies.com


___________
## Git
___________
## Gcloud
___________
## ChatOps
___________
## Packer

### Базовый образ
- Настройки ОС
- Пакеты, с низкой частотой изменения
- Логи, мониторинг, агенты систем управления конфигурацией
### Immutable Infrastructure
- Образ VM - артефакт для деплоя
- Не изменяем запущенные инстансы
- Любые изменения: собираем новый образ
- Старый инстанс заменяем на новый
### Packer

[Download](https://www.packer.io/downloads.html)

`$ packer -v`

Для управления сторонней инфрастуктурой необходимо в ней авторизоваться

`$ gcloud auth application-default login`

### запуск
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
___________
## Terraform
___________
## Docker
___________
## Gitlab CI/CD
___________
## Prometeus
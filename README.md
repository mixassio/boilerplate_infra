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
В дирректории `packer` есть примеры-шаблоны packer-image с использованием скриптов и ансибл.
### Базовый образ должен содержать:
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
### Пример конфигурационного файла main.tf
```
provider "google" {
  project     = "infra-14367"
  region      = "europe-west1"
}
resource "google_compute_instance" "app" {
name = "reddit-app" machine_type = "g1-small"
zone = "europe-west1-b" # определение загрузочного диска boot_disk {
    initialize_params { image = "reddit-base" }
  }
  network_interface {
    network = "default"
    access_config {}
} }
```
### Providers
- Содержат настройки аутентификации и подключения к платформе или сервису
- Предоставляют набор ресурсов для управления
- Могут использоваться в модулях (начиная с версии 0.10.0) - Поддержка большого количества сервисов с API: AWS, Google Cloud, GitHub, PowerDNS, VCloud etc.

Установка провайдера производится командой `terraform init`

[list preferences all providers](https://www.terraform.io/docs/providers/index.html)

### Resources
- Определяются типом провайдера
- Позволяют управлять компонентами платформы или сервиса 
- Могут иметь обязательные и необязательные аргументы Могут ссылаться на другие ресурсы
- Комбинация тип ресурса + имя уникально идентифицирует ресурс в рамках данной конфигурации

### Основные команды
- Для приведения системы в целевое состояние используется команда `terraform -auto-approve=true apply` - идемпотентна!
- Для просмотра, какие изменения будут применены `terraform plan`
- Для обновления конфигурации `terraform refresh`
- Для просмотра выходных переменных `terraform output`
- Для пересоздания ресурса `terraform taint google_compute_instance.app`

### Базовая организация конфигурационных файлов
- `main.tf` - основная конфигурация
- `variables.tf` - объявляение input-переменных 
- `outputs.tf` - определение - выходных переменных
- `terraform.tfvars.example` - пример файла - `terraform.tfvars`

### State файлы
- Terraform хранит информацию об управляемых ресурсах в файле terraform.tfstate
- Файл terraform.tfstate обновляется при каждом запуске apply или refresh
- Файл terraform.tfstate.backup используется для бекапа предыдущего terraform.state
- По умолчанию сохраняются локально в папке с конфигурацией

### Input-переменные
- Позволяют параметризировать конфигурационные файлы Три типа:
  - string
  - map
  - list
  - boolean
- Можно передать из файла, из окружения, из командной строки или интерактивно.

Входные переменные описываются в файле variables.tf:
```
variable project {
  description = "Project ID"
}
variable region {
  description = "Region"
  default = "europe-west1"
}
```
Пример использования переменных в main.tf:
```
provider "google" {
  project = "${var.project}"
  region  = "${var.region}"
}
```
Задание переменных через файл
- Укажем переменные в файле my-vars.tfvars:
```
project = "infra-179015"
public_key_path = "~/.ssh/appuser.pub"
```
- Указываем путь до файла при запуске команд terraform:
```
$ terraform apply -var-file=my-vars.tfvars
```
- Если файл называется terraform.tfvars, то переменные загружаются автоматически.

### Поиск по state-файлу
```
$ terraform show
$ terraform state show google_compute_firewall.firewall_puma
$ terraform show | grep assigned_nat_ip
```
### Output-переменные
- Позволяют сохранить выходные значения после создания ресурсов.
- Облегчают процедуру поиска нужных данных
- Используются в модулях как входные переменные для других модулей

`outputs.tf`
```
output "app_external-ip {
  value="${google_compute_instance.app.network_interface.0.access_config.0.assigned_nat_ip}"
}
// просмотр
$ terraform output
app_external_ip = 104.199.54.241
```

___________
## Docker
___________
## Gitlab CI/CD
___________
## Prometeus
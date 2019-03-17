## Terraform

Terraform позволяет описывать и поднимать удаленно инфраструктуру в облаках (дальнейшие примеры в `Google Cloud Platform`). Простейшая конфигурация состоит из четырех файлов:

- `main.tf` - основной конфиг, описывающий какие инстансы нам нужны
- `variables.tf` - конфиг с описанием переменных и значениями по дефолту, если дефолтных значений нет, то они являются обязательными
- `terraform.tfvars` - конфиг с значением переменных, часто является секретным, нужно с осторожностью пушить в публичные репозитарии
- `outputs.tf` - описание выходных переменных, необязательный файл, но очень удобно вычленять нужные параметры из созданного инстанса, например IP созданного в облаке инстанса

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
### Providers (конкретное облако - GCP, DO, AWS, YANDEX и др.)
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


### Input-переменные `variables.tf`
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

### Output-переменные `outputs.tf`
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
После описания конфигурации всё готово к "поднятию" инфраструктуры в облаке

### Основные команды
- Для приведения системы в целевое состояние используется команда `terraform -auto-approve=true apply` - идемпотентна!
- Для просмотра, какие изменения будут применены `terraform plan`
- Для обновления конфигурации `terraform refresh`
- Для просмотра выходных переменных `terraform output`
- Для пересоздания ресурса `terraform taint google_compute_instance.app`

После создания инфраструктуры в папке появляется `state`-файл со всей созданной инфраструктурой и удаляется после `terraform-destroy`

### State файлы
- Terraform хранит информацию об управляемых ресурсах в файле `terraform.tfstate`
- Файл `terraform.tfstate` обновляется при каждом запуске apply или refresh
- Файл `terraform.tfstate.backup` используется для бекапа предыдущего `terraform.state`
- По умолчанию сохраняются локально в папке с конфигурацией


### Поиск по state-файлу
```
$ terraform show
$ terraform state show google_compute_firewall.firewall_puma
$ terraform show | grep assigned_nat_ip
```
### Пример создания простой инфраструктуры в `GPC`

- [Gist `main.tf`](https://gist.github.com/mixassio/d4934556666a72b6173458dcfe75f2df)
- [Gist `variables.tf`](https://gist.github.com/mixassio/b00b14d69b2b32c4b53a1e5185aa938c)
- [Gist `outputs.tf`](https://gist.github.com/mixassio/1dd7784f7aae7c881f0c9ac0119a942b)
- [Gist `terraform.tfvars`](https://gist.github.com/mixassio/35aac171b4516adc8ab2ee27abcb70c7)

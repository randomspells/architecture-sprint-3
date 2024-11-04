# Установка PlantUML

Диаграммы находятся в папке ./diagrams

Для просмотра необходимо:

- установить VS Code
- установить плагин PlantUML
- проверить, что установлены Java (JDK) и graphviz
- открыть предпросмотр при помощи Alt + D

Либо установить плагин для IntelliJ IDEA аналогичным образом.

# Описание задания 1

В рамках задания описана архитектура AS-IS монолитной системы для управления экосистемой умных домов.

## Функциональность

Пользователь может просматривать информацию о температуре при помощи WEB-приложения, либо посылать команды на включение или выключение отопления. Монолитное приложение на JAVA занимается обработкой запросов и управляет датчиками.

## Недостатки решения

Существующее решение сложно масштабировать. Если в монолитное приложение добавить новую функциональность, например, добавить новые умные устройства и бизнес-логику для работы с ними, то с увеличением числа пользователей нагрузка будет расти и может привести к сбоям в работе системы.

К проблемам существующего решения можно отнести:

- сложность внесения изменений,
- длительные циклы развертывания,
- трудности масштабирования,
- трудности управления командой

Разделение на микросервисы позволит избавиться от этих проблем, поскольку теперь изменения будут затрагивать отдельный бизнес-домен и вероятнось ошибок снизится, можно будет разделить разработчиков на команды, каждая из которых будет заниматься своей частью бизнес-логики. Также каждый микросервис можно будет масштабировать отдельно с учетом нагрузки.

Существующее JAVA-приложение для управления отоплением может быть выделено в отдельный микросервис.

# Описание задания 2



# Базовая настройка

## Запуск minikube

[Инструкция по установке](https://minikube.sigs.k8s.io/docs/start/)

```bash
minikube start
```

## Добавление токена авторизации GitHub

[Получение токена](https://github.com/settings/tokens/new)

```bash
kubectl create secret docker-registry ghcr --docker-server=https://ghcr.io --docker-username=<github_username> --docker-password=<github_token> -n default
```

## Установка API GW kusk

[Install Kusk CLI](https://docs.kusk.io/getting-started/install-kusk-cli)

```bash
kusk cluster install
```

## Смена адреса образа в helm chart

После того как вы сделали форк репозитория и у вас в репозитории отработал GitHub Action. Вам нужно получить адрес образа <https://github.com/><github_username>/architecture-sprint-3/pkgs/container/architecture-sprint-3

Он выглядит таким образом
`ghcr.io/<github_username>/architecture-sprint-3:latest`

Замените адрес образа в файле `helm/smart-home-monolith/values.yaml` на полученный файл:

```yaml
image:
  repository: ghcr.io/<github_username>/architecture-sprint-3
  tag: latest
```

## Настройка terraform

[Установите Terraform](https://yandex.cloud/ru/docs/tutorials/infrastructure-management/terraform-quickstart#install-terraform)

Создайте файл ~/.terraformrc

```hcl
provider_installation {
  network_mirror {
    url = "https://terraform-mirror.yandexcloud.net/"
    include = ["registry.terraform.io/*/*"]
  }
  direct {
    exclude = ["registry.terraform.io/*/*"]
  }
}
```

## Применяем terraform конфигурацию

```bash
cd terraform
terraform init
terraform apply
```

## Настройка API GW

```bash
kusk deploy -i api.yaml
```

## Проверяем работоспособность

```bash
kubectl port-forward svc/kusk-gateway-envoy-fleet -n kusk-system 8080:80
curl localhost:8080/hello
```

## Delete minikube

```bash
minikube delete
```

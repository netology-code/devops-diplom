# Дипломное задание курса DevOps инженер

Цель:
1. Подготовить облачную инфраструктуру на базе облачного провайдера AWS.
2. Запуск кубернетес кластер.
3. Установить и настроить систему мониторинга.
4. Докеризировать и автоматизировать сборку тестового приложения.
5. Настроить CI для автоматической сборки и тестирования.
6. Настроить CD для автоматической выкладки приложения.

Теперь поэтапно:

## Создание облачной инфраструктуры

Для начала необходимо подготовить облачную инфраструктуру в облаке AWS при помощи [терраформа](https://www.terraform.io/).

Особенности выполнения:
1. Активируйте купон для AWS полученный от куратора курса.
1. Имейте в виду, что бюджет купона ограничен.
1. Используем последнюю стабильную версию [терраформа](https://www.terraform.io/).

Теперь надо подготовить инфраструктуру для дальнейшей установки кубернетес кластера.
1. Создайте iam сервис аккаунт из-под которого будет работать терраформ.
1. Подготовьте [бэкэнд](https://www.terraform.io/docs/language/settings/backends/index.html) для этого терраформ приложения:
   1. Рекомендуемый вариант: [Terraform Cloud](https://app.terraform.io/)
   1. Альтернативный вариант: s3 бакет в созданном aws аккаунте
1. Настройки [workspaces](https://www.terraform.io/docs/language/state/workspaces.html)
   1. Рекомендуемый вариант: создайте два workspace: *stage* и *prod*. В случае выбора этого вариант все последующие шаги должны учитывать факт существования нескольких воркспейсов.
   1. Альтернативный вариант: используйте один workspace, назвав его *stage*. Пожалуйста, не используйте workspace создавайемый терраформом по-умолчанию.
1. Создайте VPC при помощи готового [модуля](https://registry.terraform.io/modules/terraform-aws-modules/vpc/aws/latest) от AWS.
1. При помощи терраформа подготовить как минимум 3 ec2 инстантса для запуска кубернетес кластера. Какой тип инстанса выбрать решите самостоятельно. Если в дальнейшем поймете, что необходимо сменить тип инстанса, то благодаря терраформу это будет сделать легко.
1. Во время выполнения также понадобится создать [security groups](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/security_group) и некоторые другие ресурсы.
1. Учтите, что доступ к ec2 должен будет осуществляться по интернету, а не только по локальной сети.
1. Убедитесь, что теперь вы можете выполнить команды `terraform destroy` и `terraform apply` сколько угодно раз без дополнительных ручных действий.
1. В случае использования [Terraform Cloud](https://app.terraform.io/) в виде [бэкэнда](https://www.terraform.io/docs/language/settings/backends/index.html) убедитесь, что применение изменений конфигураций успешно происходит через terraform cloud.


Ожидаемый результат:
1. Терраформ конфигурация позволяющая создать описанную инфраструктуру без дополнительных ручных действий.
1. Если при выполнении следующий пунков задания понадобиться внести изменения в созданную конфигурацию, это нормально.

## Создание кубернетес кластера

На этом этапе необходимо создать [кубернетес](https://kubernetes.io/ru/docs/concepts/overview/what-is-kubernetes/) кластер на базе предварительно созданной инфрастуктуры.

Это можно сделать двумя способами:
1. Рекомендуемый вариант: самостоятельная установка кубернетес кластера.
   1. подготовить [ansible](https://www.ansible.com/) конфигурации, можно воспользоваться, например [Kubespray](https://kubernetes.io/docs/setup/production-environment/tools/kubespray/)
   1. задеплоить кубернетес на подготовленные ранее ec2 инстансы, в случае нехватки каких-либо ресурсов вы всегда можете создать их при помощи терраформа.
1. Альтернативый вариант: воспользуйтесь сервисом [EKS](https://aws.amazon.com/eks/) (Amazon Elastic Kubernetes Service)
   1. воспользуйте модулем [terraform-aws-eks](https://github.com/terraform-aws-modules/terraform-aws-eks)
   1. дополните уже существующую терраформ конфигурацию, новый проект создавать не нужно.

Ожидаемый результат:
1. Работоспособный кубернетес кластер.
2. В файле `~/.kube/config` находятся данные для доступа к кластеру.
3. Команда `kubectl get pods --all-namespaces` отрабатывает без ошибок.

## Создание тестового приложения

Для перехода к следующему этапу необходимо подготовить тестовое приложение, эмулирующее основное приложение разрабатываемое вашей компанией.

Способ подготовки:
1. Рекомендуемый вариант:
   1. Создайте отдельный git репозиторий с простым nginx конфигом, который будет отдавать статические данные.
   2. Подготовьте Dockerfile для создания образа приложения.
2. Альтернативный вариант:
   1. Используйте любой другой код, главное, что бы был самостоятельно создан Dockerfile.

Ожидаемый результат:
1. Git репозиторий с тестовым приложением и Dockerfile.
2. Dockerhub регистр с собранным docker image.

## Подготовка кубернетес конфигурации

Уже должны быть готовы конфигурации для автоматического создания облачной инфрасткруктуры и поднятия кубернетес кластера.  
Теперь необходимо подготовить конфигурационные файлы для настройки нашего кубернетес кластера.

Цель:
1. Задеплоить в кластер [prometheus](https://prometheus.io/), [grafana](https://grafana.com/), [alertmanager](https://github.com/prometheus/alertmanager), [экспортет](https://github.com/prometheus/node_exporter) основных метрик кубернетеса.
1. Задеплоить тестовое приложение, например [nginx](https://www.nginx.com/) сервер отдающий статическую страницу.

Рекомендуемые способ выполнения:
1. Воспользовать пакетом [kube-prometheus](https://github.com/prometheus-operator/kube-prometheus), который уже включает в себя [кубернетес оператор](https://operatorhub.io/) для [grafana](https://grafana.com/), [prometheus](https://prometheus.io/), [alertmanager](https://github.com/prometheus/alertmanager) и [node_exporter](https://github.com/prometheus/node_exporter). При желании можете собрать все эти приложения отдельно.
1. Для организации конфигурации использовать [qbec](https://qbec.io/) основанный на [jsonnet](https://jsonnet.org/). Обратите внимание на имеющиеся функции для интеграции helm конфигов и [helm charts](https://helm.sh/)
1. Если на первом этапе вы не воспользовались [Terraform Cloud](https://app.terraform.io/), то задеплойте в кластер [atlantis](https://www.runatlantis.io/) для отслеживания изменений инфраструктуры.

Ожидаемый результат:
1. Git репозиторий с конфигурационными файлами для настройки кубернетес.
2. Http доступ к web интерфейсу grafana.
3. Дашборды в grafana отображающие состояние кубернет кластера.
4. Http доступ к тестову приложению

##  Установка и настройка CI/CD

Осталось настроить ci/cd систему для автоматической сборки docker image и деплоя приложения при изменении кода.

Цель:
1. Автоматическая сборка docker образа при коммите в репозиторий с тестовым приложением.
2. Автоматический деплой нового docker образа.

Можно использовать [teamcity](https://www.jetbrains.com/ru-ru/teamcity/) или [jenkins](https://www.jenkins.io/)

Ожидаемый результат:
1. Teamcity или jenkins доступны по http.
2. При любом коммите в репозиторий с тестовым приложением происходит сборка и отправка в регистр докер образа.
3. При создании тега в репозитории происходит деплой соответсвующего докер образа.


##  Что необходимо для сдачи задания
1. Репозиторий с конфигурационными файлами терраформа и готовность продемонстировать создание всех рессурсов с нуля.
2. Пример pull request с комментариями созданными atlantis'ом или снимки экрана из Terraform Cloud.
3. Репозиторий с конфигурацией ansible, если был выбран способ создания кубернетес кластера при помощи ansible.
4. Репозиторий с Dockerfile тестового приложения и ссылка на собранный docker image.
5. Репозиторий с конфигурацией кубернетес кластера.
6. Ссылка на тестовое приложение и веб интерфейс графаны с данными доступа.

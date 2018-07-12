# antonritter85_infra
antonritter85 Infra repository

## Homework-6

#### В процессе сделано:

- установлен и настроен Terraform, в том числе и провайдер GCP;
- созданы следующие ресурсы Terraform:
  + инстанс (google_compute_instance) c ssh-ключем для пользователя appuser в метадданных данного инстанса, тегом "reddit-app" провижинерами, (provisioners)  для развертывания приложения reddit app (при этом приложение запускается при старте инстанса (как systemd unit));
  + првило firewall (google_compute_firewall) для инстансов с тегом "reddit-app";
- создан файл "outputs.tf" для выходных переменных (в нашем случае первоначально выводится присвоенный внешний IP-адрес инстанса);
- создан файл "variables.tf" для параметризации конфигурационных файлов Terraform: параметризованы некоторые аттрибуты ресурсов в main.tf;
- (Задание со *) добавлены ssh-ключи в метадданые проекта (в виде ресурса google_compute_project_metadata);
- (Задание с **) создан [HTTP-балансировщик для GCP](https://cloud.google.com/compute/docs/load-balancing/network/example) (в файле "lb.tf"), направляющий траффик на указанный пул инстансов
- (Задание с **) добавлен инстанс reddit-app2, сначала непосредственным копированием, потом с использованием параметра count (параметризованы  адреса инстансов (URI, "${google_compute_instance.app.*.self_link}") в пуле балансировщика (файл "lb.tf")); в "outputs.tf" добавлены IP-адреса балансировщика и параметризован вывод IP-адресов всех инстансов.

#### Задание со * (ответ на вопрос со слайда 50)

Terraform, после выполнения команды **terraform apply** удаляет ssh-ключ пользователя appuser_web, который был добавлен из веб-интерфейса.

#### Задание с ** (ответ на вопрос со слайда 51)

При добавлении идентичных инстанстов код повторяется: теряется гибкость при модификации (создании, изменении, удалении) однотипных ресурсов, нарушается принцип DRY.

## Homework-5

#### В процессе сделано:

 - создан и собран шаблон Packer (ubuntu16.json): Ubuntu 16.04 + Ruby + MongoDB
 - созданный шаблон (ubuntu16.json) параметризован, с использованием пользовательских 
    переменных (в самом шаблоне и в файле variables);
 - на базе шаблона ubuntu16.json создан baked-шаблон (immutable.json): Ubuntu 16.04 + Ruby + 
    MongoDB + Reddit App, при этом приложение запускается при старте инстанса (как systemd unit).
 - создан скрипт create-reddit-vm.sh для создания и запуска машины из подготовленного образа 
    из семейства reddit-full.

## Homework-4

### Дополнительное задание (слайд 20)

Startup скрипт файл, передаваемый команде gcloud:

```
gcloud compute instances create reddit-app\
  --boot-disk-size=10GB \
  --image-family ubuntu-1604-lts \
  --image-project=ubuntu-os-cloud \
  --machine-type=g1-small \
  --tags puma-server \
  --restart-on-failure \
  --metadata-from-file startup-script=scripts/startupscript.sh
```

где в опции startup-script=PATH/TO/FILE - относительный путь до файла скрипта.

Вариант со startup-script-url:

```
gcloud compute instances create reddit-app\
  --boot-disk-size=10GB \
  --image-family ubuntu-1604-lts \
  --image-project=ubuntu-os-cloud \
  --machine-type=g1-small \
  --tags puma-server \
  --restart-on-failure \
  --scopes storage-ro \
  --metadata startup-script-url=gs://antonritter85-bucket/startupscripturl.sh
  ```

### Дополнительное задание (слайд 21)

Правило firewall для работы приложения default-puma-server, созданное через команду gcloud:

```
gcloud compute firewall-rules create default-puma-server \
    --network default \
    --action allow \
    --direction ingress \
    --rules tcp:9292 \
    --source-ranges 0.0.0.0/0 \
    --target-tags puma-server
```

#### Информация для проверки

testapp_IP = 35.205.109.145
testapp_port = 9292

## Homework-3
### Задание со слайда 35:
```
ssh -o ProxyCommand='ssh -W %h:%p appuser@35.206.129.231' appuser@10.132.0.3
```

### Доп. задание со слайда 35:
В файле ~/.ssh/config на локальной машине добавляем следующиие строки:
```
#Host someinternalhost
 HostName 10.132.0.3
 User appuser
 IdentityFile ~/.ssh/appuser
 ProxyCommand ssh -W %h:%p appuser@bastion

Host bastion
 HostName 35.206.129.231
 User appuser
 IdentityFile ~/.ssh/appuser
```
Это позволит подключиться к someinternalhost командой:
```
ssh someinternalhost
```

bastion_IP = 35.206.129.231
someinternalhost_IP = 10.132.0.3

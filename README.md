# Домашнее задание к занятию "`7.3. Подъём инфраструктуры в Yandex Cloud`" - `Краев Александр`


### Задание 1

От заказчика получено задание: при помощи Terraform и Ansible собрать виртуальную инфраструктуру и развернуть на ней веб-ресурс.

В инфраструктуре нужна одна машина с ПО ОС Linux, двумя ядрами и двумя гигабайтами оперативной памяти.

Требуется установить nginx, проверить.

### Решение
Поднимаем через конфигурационный файл terraform два хоста: вебсервер и машину для ansible

```
terraform {
  required_providers {
    yandex = {
      source = "yandex-cloud/yandex"
    }
  }
  required_version = ">= 0.13"
}
provider "yandex" {
  zone = "ru-central1-b"
}

resource "yandex_compute_instance" "vm-1" {
  name = "nginx1"
  platform_id = "standard-v1"
  resources {
    core_fraction = 5
    cores  = 2
    memory = 4
  }
  boot_disk {
    initialize_params {
      image_id = "fd8iea4hktad0qe0nogj"
    }
  }
  network_interface {
    subnet_id = "e2l17d14hi3slv6nnd6v"
    nat       = true
  }
  metadata = {
    user-data = "${file("./meta.txt")}"
  }
  scheduling_policy {
    preemptible = true
  }
}
resource "yandex_compute_instance" "vm-2" {
  name = "ansible"
  platform_id = "standard-v1"
  resources {
    core_fraction = 5
    cores  = 2
    memory = 4
  }
  boot_disk {
    initialize_params {
      image_id = "fd8iea4hktad0qe0nogj"
    }
  }
  network_interface {
    subnet_id = "e2l17d14hi3slv6nnd6v"
    nat       = true
  }
  metadata = {
    user-data = "${file("./meta.txt")}"
  }
  scheduling_policy {
    preemptible = true
  }
}
output "internal_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.ip_address
}
output "external_ip_address_vm_1" {
  value = yandex_compute_instance.vm-1.network_interface.0.nat_ip_address
}
output "internal_ip_address_vm_2" {
  value = yandex_compute_instance.vm-2.network_interface.0.ip_address
}
output "external_ip_address_vm_2" {
  value = yandex_compute_instance.vm-2.network_interface.0.nat_ip_address
}
```

результат

![task](/1.png "Задание 1")
![task](/2.png "Задание 1")

Далее с помощью ansible настраиваем первую гостевую машину

проверяем доступность машины

![task](/3.png "Задание 1")

Описываем два плейбука: для установки nginx и для проверки состояния сервиса

![task](/5.png "Задание 1")

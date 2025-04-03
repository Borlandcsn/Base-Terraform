Домашнее задание к занятию «Основы Terraform. Yandex Cloud»
Задание 1
1 Изучите проект. В файле variables.tf объявлены переменные для Yandex provider.

2 Создайте сервисный аккаунт и ключ. service_account_key_file.

![Снимок](https://github.com/user-attachments/assets/386c7fc0-eb84-4455-b36e-50e94280940c)

3 Сгенерируйте новый или используйте свой текущий ssh-ключ. Запишите его открытую(public) часть в переменную vms_ssh_public_root_key.

4 Инициализируйте проект, выполните код. Исправьте намеренно допущенные синтаксические ошибки. Ищите внимательно, посимвольно. Ответьте, в чём заключается их суть.
  1 ошибка связана с использованием неверного аргумента platform_id = "standart-v4" платформы standart-v4 нет есть standard-v3 - исправляем platform_id = "standard-v3".
  2  ошибка говорит о том что выбрано неправильное значение использования ядра процессора для resource "yandex_compute_instance" исправляем "platform"
     core_fraction = 20
  3    Эта ошибка говорит о том что выбрано неправильное значение ядер исправляем cores = 2

  ![Screenshot_12](https://github.com/user-attachments/assets/ee5f268c-0cb3-4f6a-bd66-eefdbcb3642b)


5 ![подк по ssh](https://github.com/user-attachments/assets/502958e4-cfbc-4c6e-8a95-0f11efad06af)

6 Параметр Core Fraction даёт возможность установить ограничение на долю мощности виртуального процессора, которая будет выделяться ВМ. Если нагрузка на машину небольшая, можно задать значение 20% от одного ядра, чтобы избежать лишних затрат на вычислительные ресурсы

Задание 2

1 resource "yandex_vpc_network" "develop" {
  name = var.vpc_name
}
resource "yandex_vpc_subnet" "develop" {
  name           = var.vpc_name
  zone           = var.default_zone
  network_id     = yandex_vpc_network.develop.id
  v4_cidr_blocks = var.default_cidr
}


data "yandex_compute_image" "ubuntu" {
  family = var.vm_web_image
}
resource "yandex_compute_instance" "platform" {
  name        = var.vm_web_name
  platform_id = var.vm_web_platform_id
  resources {
    cores         = var.vm_web_cores
    memory        = var.vm_web_memory
    core_fraction = var.vm_web_core_fraction
  }
  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.image_id
    }
  }
  scheduling_policy {
    preemptible = var.vm_web_preemptible
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.develop.id
    nat       = var.vm_web_nat
  }

  metadata = {
    serial-port-enable = 1
    ssh-keys           = "ubuntu:${var.vms_ssh_root_key}"
  }

}


2 переменные в файле variables.tf

variable "vm_web_image" {
  type        = string
  default     = "ubuntu-2004-lts"
  description = "image"
}

variable "vm_web_user" {
  type        = string
  default     = "ubuntu"
  description = "user"
}

variable "vm_web_name" {
  type        = string
  description = "Name"
  default     = "netology-develop-platform-web"
}

variable "vm_web_platform_id" {
  type        = string
  description = "platform ID"
  default     = "standard-v3"
}

variable "vm_web_cores" {
  type        = number
  description = "CPU"
  default     = 2
}

variable "vm_web_memory" {
  type        = number
  description = "RAM"
  default     = 1
}

variable "vm_web_core_fraction" {
  type        = number
  description = "CPU%"
  default     = 20
}
variable "vm_web_preemptible" {
  type        = bool
  default     = true
  description = "preemptible"
}

variable "vm_web_nat" {
  type        = bool
  default     = true
  description = "nat"
}

3 ![Screenshot_1](https://github.com/user-attachments/assets/6261e9e9-a376-4b25-8908-fb4179a2f28a)

Задание 3

#сеть для первой машины
resource "yandex_vpc_network" "develop" {
  name = var.vpc_name
}
resource "yandex_vpc_subnet" "develop" {
  name           = var.vpc_name
  zone           = var.default_zone
  network_id     = yandex_vpc_network.develop.id
  v4_cidr_blocks = var.default_cidr
}
#сеть для второй машины в зоне b
resource "yandex_vpc_network" "db" {
  name = var.vpc_db
}

resource "yandex_vpc_subnet" "db" {
  name           = var.vpc_db
  zone           = var.b-zone
  network_id     = yandex_vpc_network.db.id
  v4_cidr_blocks = var.b-zone_cidr
}

data "yandex_compute_image" "ubuntu" {
  family = var.vm_web_image
}
resource "yandex_compute_instance" "platform" {
  name        = var.vm_web_name
  platform_id = var.vm_web_platform_id
  resources {
    cores         = var.vm_web_cores
    memory        = var.vm_web_memory
    core_fraction = var.vm_web_core_fraction
  }
  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.image_id
    }
  }
  scheduling_policy {
    preemptible = var.vm_web_preemptible
  }
  network_interface {
    subnet_id = yandex_vpc_subnet.develop.id
    nat       = var.vm_web_nat
  }

  metadata = {
    serial-port-enable = 1
    ssh-keys           = "ubuntu:${var.vms_ssh_root_key}"
  }

}
resource "yandex_compute_instance" "db" {
  name        = var.vm_db_name
  zone        = var.vm_db_zone
  platform_id = var.vm_db_platform_id
  resources {
    cores         = var.vm_db_cores
    memory        = var.vm_db_memory
    core_fraction = var.vm_db_core_fraction
  }
  boot_disk {
    initialize_params {
      image_id = data.yandex_compute_image.ubuntu.image_id
    }
  }
  scheduling_policy {
    preemptible = var.vm_db_preemptible
  } 
  network_interface {
    subnet_id = yandex_vpc_subnet.db.id
    nat       = var.vm_db_nat
  }
  
  metadata = {
    serial-port-enable = 1
    ssh-keys           = "ubuntu:${var.vms_ssh_root_key}"
  }
}

файл vms_platform.tf

# web
variable "vm_web_image" {
  type        = string
  default     = "ubuntu-2004-lts"
  description = "image"
}

variable "vm_web_user" {
  type        = string
  default     = "ubuntu"
  description = "user"
}

variable "vm_web_name" {
  type        = string
  description = "Name"
  default     = "netology-develop-platform-web"
}

variable "vm_web_platform_id" {
  type        = string
  description = "platform ID"
  default     = "standard-v3"
}

variable "vm_web_cores" {
  type        = number
  description = "CPU"
  default     = 2
}

variable "vm_web_memory" {
  type        = number
  description = "RAM"
  default     = 1
}

variable "vm_web_core_fraction" {
  type        = number
  description = "CPU%"
  default     = 20
}
variable "vm_web_preemptible" {
  type        = bool
  default     = true
  description = "preemptible"
}

variable "vm_web_nat" {
  type        = bool
  default     = true
  description = "nat"
}
variable "vm_db_name" {
  type        = string
  description = "Name"
  default     = "netology-develop-platform-db"
}

variable "vm_db_platform_id" {
  type        = string
  description = "platform ID"
  default     = "standard-v3"
}

variable "vm_db_cores" {
  type        = number
  description = "CPU"
  default     = 2
}

variable "vm_db_memory" {
  type        = number
  description = "RAM"
  default     = 2
}

variable "vm_db_core_fraction" {
  type        = number
  description = "CPU%"
  default     = 20
}
variable "vm_db_preemptible" {
  type        = bool
  default     = true
  description = "preemptible"
}

variable "vm_db_nat" {
  type        = bool
  default     = true
  description = "nat"
}

variable "b-zone" {
  type        = string
  default     = "ru-central1-b"
  description = "https://cloud.yandex.ru/docs/overview/concepts/geo-scope"
}

variable "default_cidr" {
  type        = list(string)
  default     = ["10.0.1.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}

variable "b-zone_cidr" {
  type        = list(string)
  default     = ["10.0.2.0/24"]
  description = "https://cloud.yandex.ru/docs/vpc/operations/subnet-create"
}
variable "vpc_name" {
  type        = string
  default     = "develop"
  description = "VPC network & subnet name"
}

variable "vpc_db" {
  type        = string
  default     = "db"
  description = "VPC network & subnet db"
}

![Снимок](https://github.com/user-attachments/assets/5871b697-6745-42fa-b320-159450813500)

Задание 4

output "instances" {
  value = {
    web = {
      instance_name = yandex_compute_instance.platform.name
      external_ip   = yandex_compute_instance.platform.network_interface[0].nat_ip_address
    }
    db = {
      instance_name = yandex_compute_instance.db.name
      external_ip   = yandex_compute_instance.db.network_interface[0].nat_ip_address
    }
  }
}

![Снимок](https://github.com/user-attachments/assets/f6bb94c3-68d9-46b3-9c33-f21b6b13c6d7)

Задание 5

locals {
  vm_web_f_name = "${var.vpc_name}-${var.vm_web_name}"
  vm_db_f_name  = "${var.vpc_db}-${var.vm_db_name}"
}

![Pasted image 20250204222810](https://github.com/user-attachments/assets/b66f52ee-e8f9-408a-8b57-3422ea7da6cb)

![Pasted image 20250204222739](https://github.com/user-attachments/assets/ce752848-e4d1-4dcb-8a3d-9d03a727a382)

![Снимок](https://github.com/user-attachments/assets/4888b7c6-7047-4082-98be-3a643ccdf8eb)

Задание 6

пример из terraform.tfvars:
vms_resources = {
  web={
    cores=2
    memory=2
    core_fraction=5
    hdd_size=10
    hdd_type="network-hdd"
    ...
  },
  db= {
    cores=2
    memory=4
    core_fraction=20
    hdd_size=10
    hdd_type="network-ssd"
    ...
  }
}

variable "vms_resources" {
  description = "единая map-переменная"
  type = map(object({
    cores         = number
    memory        = number
    core_fraction = number
  }))
  default = {
    web = {
        cores        = 2
        memory       = 1
        core_fraction = 20
    },
    db = {
        cores        = 2
        memory       = 2
        core_fraction = 20
    }
  }
}

пример из terraform.tfvars:
metadata = {
  serial-port-enable = 1
  ssh-keys           = "ubuntu:ssh-ed25519 AAAAC..."
}


variable "metadata" {
  type = map(string)
  description = "Metadata"
}
metadata = {
  serial-port-enable = "1",
  ssh-keys = "ubuntu:ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIHiZjyQiMYyn9ZJrggVSncksTiG+2NMFf2TVZmROmb6O ssilchin-deb"
}

![Pasted image 20250205224840](https://github.com/user-attachments/assets/1ac6e139-f20c-441d-8d3c-8ff9cfd2293a)

![Снимок](https://github.com/user-attachments/assets/15fe4dc0-2f91-4d18-8e21-a5c6fd78806d)



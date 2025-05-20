---
title: Ferramentas OTA - espressif
---

# Algumas ferramentas 
Um aplicativo no "ESP-Dev-Board" pode ser atualizado em tempo de execução baixando uma nova imagem via Wi-Fi ou Ethernet e instalando-a em uma partição OTA. O ESP-IDF oferece dois métodos para realizar atualizações Over The Air (OTA):
- Usando as APIs nativas fornecidas pelo componente `app_update`.
- Usando APIs simplificadas fornecidas pelo componente `esp_https_ota` que fornece funcionalidade para atualização via HTTPS.


## Componente app_update
O componente `app_update` abstrai a complexidade de lidar com operações de partição e camada flash. O arquivo de cabeçalho principal para a API deste componente é esp_ota_ops.h. Através das funções definidas neste cabeçalho, é possível gerenciar as operações de OTA, incluindo:
- Começar uma atualização OTA, alocando o contexto e apagando a partição de destino (esp_ota_begin).
- Escrever os dados do firmware na partição OTA (esp_ota_write ou esp_ota_write_with_offset).
- Finalizar a atualização e validar a nova imagem (esp_ota_end).
- Definir qual partição de aplicação deve ser inicializada no próximo boot (esp_ota_set_boot_partition).
- Obter informações sobre a partição atualmente em execução (esp_ota_get_running_partition) e a próxima partição de atualização (esp_ota_get_next_update_partition).
- Marcar uma aplicação como válida (esp_ota_mark_app_valid_cancel_rollback) ou inválida (esp_ota_mark_app_invalid_rollback_and_reboot) para suportar o recurso de rollback

Ele expõe uma API simplificada   `otatool.py` que pode ser usada diretamente por um host para enviar firmware para flash de um  dispositivo. Além disso, também fornece algumas APIs convenientes para lidar com casos de uso de rollback e anti-rollback.

A ferramenta `otatool.py` pode ser importada e usada de outro script Python ou invocada de um script shell para usuários que desejam executar operações programaticamente. 

###  Interface de linha de comando

```c
# Erase otadata, resetting the device to factory app
otatool.py --port "/dev/ttyUSB1" erase_otadata

# Erase contents of OTA app slot 0
otatool.py --port "/dev/ttyUSB1" erase_ota_partition --slot 0

# Switch boot partition to that of app slot 1
otatool.py --port "/dev/ttyUSB1" switch_ota_partition --slot 1

# Read OTA partition 'ota_3' and save contents to a file named 'ota_3.bin'
otatool.py --port "/dev/ttyUSB1" read_ota_partition --name=ota_3 --output=ota_3.bin
```

### API python

Antes de mais nada, certifique-se de que o otatoolmódulo foi importado. O ponto de partida para usar a API Python da ferramenta é criar um `OtatoolTarget`. 

```
import sys
import os

idf_path = os.environ["IDF_PATH"]  # get value of IDF_PATH from environment
otatool_dir = os.path.join(idf_path, "components", "app_update")  # otatool.py lives in $IDF_PATH/components/app_update

sys.path.append(otatool_dir)  # this enables Python to find otatool module
from otatool import *  # import all names inside otatool module

# Create a parttool.py target device connected on serial port /dev/ttyUSB1
target = OtatoolTarget("/dev/ttyUSB1")
```
O objeto criado agora pode ser usado para executar operações no dispositivo de destino:

```
# Erase otadata, resetting the device to factory app
target.erase_otadata()

# Erase contents of OTA app slot 0
target.erase_ota_partition(0)

# Switch boot partition to that of app slot 1
target.switch_ota_partition(1)

# Read OTA partition 'ota_3' and save contents to a file named 'ota_3.bin'
target.read_ota_partition("ota_3", "ota_3.bin")
```

## API HTTPS OTA
Veja em [API HTTPS OTA - espressif](https-ota.md)
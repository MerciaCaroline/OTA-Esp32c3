---
title: Ferramentas OTA
---

# Algumas ferramentas 
Um aplicativo no "ESP-Dev-Board" pode ser atualizado em tempo de execução baixando uma nova imagem via Wi-Fi ou Ethernet e instalando-a em uma partição OTA. O ESP-IDF oferece dois métodos para realizar atualizações Over The Air (OTA):
- Usando as APIs nativas fornecidas pelo app_updatecomponente.
- Usando APIs simplificadas fornecidas pelo esp_https_otacomponente, que fornece funcionalidade para atualização via HTTPS.


## Componente app_update
O componente `app_update` abstrai a complexidade de lidar com operações de partição e camada flash. Ele expõe uma API simplificada que pode ser usada diretamente por uma aplicação para programar firmware para flash de um  dispositivo. Além disso, também fornece algumas APIs convenientes para lidar com casos de uso de rollback e anti-rollback.
A ferramenta `otatool.py` pode ser importada e usada de outro script Python ou invocada de um script shell para usuários que desejam executar operações programaticamente. 

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

## API HTTPS OTA
É uma camada de abstração sobre as APIs OTA existentes. Este componente de software fornece uma API simplificada para atualizar dispositivos com segurança através do canal TLS. A API aceita dois parâmetros obrigatórios: a URL HTTPS onde a imagem do firmware está hospedada e o certificado do servidor para validar a identidade do servidor.

```c
esp_err_t do_firmware_upgrade()
{
    esp_http_client_config_t config = {
        .url = CONFIG_FIRMWARE_UPGRADE_URL,
        .cert_pem = (char *)server_cert_pem_start,
    };
    esp_https_ota_config_t ota_config = {
        .http_config = &config,
    };
    esp_err_t ret = esp_https_ota(&ota_config);
    if (ret == ESP_OK) {
        esp_restart();
    } else {
        return ESP_FAIL;
    }
    return ESP_OK;
}
```

O workflow simplificado para atualização OTA se parece com o seguinte:
[Fluxo de trabalho de atualização OTA](../img/fluxo-ota.png)
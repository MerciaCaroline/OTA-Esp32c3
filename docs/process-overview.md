---
title: Visão geral do processo OTA
---

# Visão geral
O mecanismo de atualização OTA permite que um dispositivo se atualize com base nos dados recebidos enquanto o firmware normal está em execução (por exemplo, via Wi-Fi, Bluetooth ou Ethernet).
O OTA requer a configuração das tabelas de partição do dispositivo com pelo menos 2 partições OTA do tipo app(`ota_1`e `ota_0`) e uma OTA do tipo Data (`ota`).


## Transição entre partições
O bootloader inicializa o primeiro slot OTA disponível (geralmente `ota_0`). Enquanto isso `ota_1`  está vazio. 
Após a atualização OTA, o novo firmware ficará gravado em `ota_1`. A partição de dados `ota` é então atualizada para especificar qual partição de slot de aplicativo OTA deve ser inicializada em seguida. Assim, a partir do RESET subsequente, o bootloader sempre dará o controle ao firmware na `ota_1` (até a próxima atualização OTA).

![Transição de layout do Flash](../img/transicao-ota-simples.png)

## Rollback
No cenário acima, se houver algum problema (por exemplo, falha na inicialização do firmware) no firmware recém-atualizado da partição `ota_1`, não há como o dispositivo retornar ao firmware anterior que funciona. O recurso de rollback(reversão de firmware) oferece a capacidade de reverter para o firmware funcional anterior conhecido.

-  A primeira atualização OTA grava uma nova imagem de firmware  e marca seu estado como `ESP_OTA_IMG_NEW`
-  Após o RESET, o bootloader vê um novo firmware com estado `ESP_OTA_IMG_NEW`e muda seu estado para `ESP_IMAGE_PENDING_VERIFY`, pois sua funcionalidade ainda não foi validada.
- Assim que o novo firmware inicia a execução, ele pode definir seu estado para `ESP_OTA_IMG_VALID`ou `ESP_OTA_IMG_INVALID`com base na lógica do aplicativo. 

```c
const esp_partition_t *running = esp_ota_get_running_partition();
esp_ota_img_states_t ota_state;
if (esp_ota_get_state_partition(running, &ota_state) == ESP_OK) {
    if (ota_state == ESP_OTA_IMG_PENDING_VERIFY) {
        // run diagnostic function ...
        bool diagnostic_is_ok = diagnostic();
        if (diagnostic_is_ok) {
            ESP_LOGI(TAG, "Diagnostics completed successfully! Continuing execution ...");
            esp_ota_mark_app_valid_cancel_rollback();
        } else {
            ESP_LOGE(TAG, "Diagnostics failed! Start rollback to the previous version ...");
            esp_ota_mark_app_invalid_rollback_and_reboot();
        }
    }
}
```
- Se o aplicativofor marcado como `ESP_OTA_IMG_INVALID` na reinicialização subsequente, o bootloader atualiza seu estado para `ESP_OTA_IMG_ABORTED` e, assim, retorna ao firmware anterior.

![Transição de layout Flash com atualização OTA de rollback](../img/rollback.png)

! A função de diagnóstico pode ser decidida com base em vários pontos, como por exemplo, conexão À rede WIFI, pois garante a possibilidade da próxima atualização OTA.

## Anti-Rollback


## Referencias 

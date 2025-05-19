---
title: OTA pull - arduino
---

# Visão geral
ESP32-OTA-Pull é uma biblioteca arduino para facilitar atualizações OTA simples baseadas em ESP32 " pull " .

Neste cenário, você publica novas imagens de firmware em um servidor web, e seus dispositivos com Wi-Fi encontram as novas imagens e se atualizam. A estratégia é escalável. Você não precisa segmentar cada dispositivo individualmente; desde que seus dispositivos possam se conectar ao Wi-Fi, eles podem se atualizar.

## Configurações 

Um arquivo JSON informa à biblioteca onde o novo firmware pode ser encontrado e qual é sua versão.
No exemplo abaixo, um dispositivo será atualizado somente se tiver sido compilado com uma placa especifica (ESP32_DEV), se seu endereço MAC e as strings de configuração corresponderem exatamente e se estiver executando um firmware com versão inferior a 2.

```json
{
  "Configurations": [
    {
      "Board": "ESP32_DEV",
      "Device": "24:6F:28:AD:FF:04",
      "Config": "4MB",
      "Version": "2.0.0",
      "URL": "https://example.com/myimages/example.esp32_dev.v2.bin"
    }
  ]
}
```

## Como funciona
 O método CheckForOTAUpdate baixa o arquivo JSON especificado e começa a iterar por todas as "Configurações" fornecidas. Ao encontrar uma que corresponda ao esboço em execução no momento, ele compara a string "Version" com o valor passado para o método. Se a versão publicada for maior que a local, uma atualização será realizada, sujeita às restrições do parâmetro "Action". Se a versão publicada for menor, a atualização só será realizada se a propriedade AllowDowngrades estiver definida.

## Veja mais

Mais informações e exemplos disponiveis no [Github] (https://github.com/mikalhart/ESP32-OTA-Pull/tree/main)

# ESP32-CAM Web Server

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Platform](https://img.shields.io/badge/Platform-ESP32--CAM-blue.svg)](https://www.espressif.com/)
[![Arduino](https://img.shields.io/badge/Arduino-Compatible-green.svg)](https://www.arduino.cc/)

Um servidor de streaming de vídeo simples e leve para ESP32-CAM com interface web. Acesse sua câmera de qualquer dispositivo na rede local ou remotamente pela internet usando ngrok.

## Funcionalidades

- Streaming de vídeo MJPEG via HTTP
- Interface web responsiva (funciona em celular e desktop)
- Configuração WiFi fácil
- Suporte a acesso remoto via túnel ngrok
- Compatível com módulo AI-Thinker ESP32-CAM
- Funciona com Arduino IDE (Windows) e Arduino CLI (Linux)

## Hardware Necessário

- ESP32-CAM (modelo AI-Thinker)
- Cabo USB (se a placa tiver USB integrado) ou adaptador FTDI
- Computador com Windows ou Linux

## Início Rápido

### 1. Configurar WiFi

Edite o arquivo `ESP32CAM_WebServer.ino` e defina suas credenciais WiFi:

```cpp
const char* ssid = "NOME_DA_SUA_REDE";
const char* password = "SENHA_DA_SUA_REDE";
```

### 2. Fazer Upload do Código

**Usando Arduino IDE (Windows):**
1. Instale o [Arduino IDE](https://www.arduino.cc/en/software)
2. Adicione suporte à placa ESP32 (veja o [Guia de Instalação](#arduino-ide-windows))
3. Selecione a placa **AI Thinker ESP32-CAM**
4. Clique em Upload

**Usando Arduino CLI (Linux):**
```bash
arduino-cli compile --fqbn esp32:esp32:esp32cam ESP32CAM_WebServer
arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32cam ESP32CAM_WebServer
```

### 3. Acessar a Câmera

Abra o Monitor Serial (115200 baud) para ver o endereço IP:
```
Camera inicializada!
WiFi conectado!
Acesse: http://192.168.x.x
Servidor iniciado!
```

Abra o IP no navegador para visualizar o streaming.

## Limitações

- **Apenas uma conexão por vez**: A ESP32-CAM suporta apenas um cliente conectado ao streaming simultaneamente
- **Apenas WiFi 2.4GHz**: O ESP32 não suporta redes 5GHz
- O streaming pode ficar instável em sessões longas devido a limitações de memória

---

## Guia de Instalação

### Arduino IDE (Windows)

1. **Instalar Arduino IDE**
   - Baixe em: https://www.arduino.cc/en/software

2. **Adicionar Suporte à Placa ESP32**
   - Vá em **Arquivo > Preferências**
   - Adicione em **URLs Adicionais para Gerenciadores de Placas**:
     ```
     https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
     ```
   - Vá em **Ferramentas > Placa > Gerenciador de Placas**
   - Pesquise **esp32** e instale o pacote da Espressif Systems

3. **Instalar Driver USB (se necessário)**
   - **CH340**: Pesquise "CH340 driver download"
   - **CP210x**: Baixe do site da Silicon Labs

4. **Upload**
   - Selecione **Ferramentas > Placa > AI Thinker ESP32-CAM**
   - Selecione **Ferramentas > Porta > COMx**
   - Clique em **Upload**

5. **Monitor Serial**
   - Abra **Ferramentas > Monitor Serial**
   - Configure o baud rate para **115200**

### Arduino CLI (Linux)

1. **Instalar Arduino CLI**
   ```bash
   curl -fsSL https://raw.githubusercontent.com/arduino/arduino-cli/master/install.sh | BINDIR=~/.local/bin sh
   ```

2. **Configurar Suporte ao ESP32**
   ```bash
   arduino-cli config init
   arduino-cli config add board_manager.additional_urls https://raw.githubusercontent.com/espressif/arduino-esp32/gh-pages/package_esp32_index.json
   arduino-cli core update-index
   arduino-cli core install esp32:esp32
   ```

3. **Permissão da Porta Serial**
   ```bash
   sudo usermod -a -G dialout $USER
   # Faça logout e login novamente
   ```

4. **Compilar e Upload**
   ```bash
   arduino-cli compile --fqbn esp32:esp32:esp32cam ESP32CAM_WebServer
   arduino-cli upload -p /dev/ttyUSB0 --fqbn esp32:esp32:esp32cam ESP32CAM_WebServer
   ```

5. **Monitor Serial**
   ```bash
   # IMPORTANTE: use dtr=off e rts=off para evitar reset da placa
   arduino-cli monitor -p /dev/ttyUSB0 -c baudrate=115200 -c dtr=off -c rts=off
   ```

---

## Acesso Remoto (ngrok)

Acesse sua câmera de qualquer lugar usando ngrok:

1. **Crie uma conta** em https://ngrok.com

2. **Instale o ngrok**
   - Windows: Baixe em https://ngrok.com/download
   - Linux:
     ```bash
     curl -s https://ngrok-agent.s3.amazonaws.com/ngrok.asc | sudo tee /etc/apt/trusted.gpg.d/ngrok.asc >/dev/null
     echo "deb https://ngrok-agent.s3.amazonaws.com buster main" | sudo tee /etc/apt/sources.list.d/ngrok.list
     sudo apt update && sudo apt install ngrok
     ```

3. **Configure o token**
   ```bash
   ngrok config add-authtoken SEU_TOKEN_AQUI
   ```

4. **Crie o túnel**
   ```bash
   ngrok http IP_DA_CAMERA:80
   # Exemplo: ngrok http 192.168.0.23:80
   ```

5. **Acesse remotamente**

   Use a URL gerada (ex: `https://xxxxx.ngrok-free.app`) de qualquer dispositivo.

---

## Configuração

### Orientação da Imagem

Se a imagem estiver de ponta cabeça, ajuste estas linhas após `esp_camera_init`:

```cpp
sensor_t *s = esp_camera_sensor_get();
s->set_vflip(s, 1);    // 1 = invertido, 0 = normal
s->set_hmirror(s, 1);  // 1 = espelhado, 0 = normal
```

### Endpoints Disponíveis

| Endpoint | Descrição |
|----------|-----------|
| `/` | Página web com streaming embutido |
| `/stream` | Stream MJPEG direto |

---

## Solução de Problemas

| Problema | Solução |
|----------|---------|
| Caracteres estranhos no Monitor Serial | Configure o baud rate para **115200** |
| Monitor Serial reseta a placa | Use as flags `-c dtr=off -c rts=off` |
| Câmera não inicializa | Verifique se o cabo flat está bem conectado (contatos dourados voltados para a placa) |
| WiFi não conecta | Verifique SSID/senha; ESP32-CAM só suporta redes **2.4GHz** |
| Streaming instável | Apenas 1 dispositivo pode assistir por vez; feche outras conexões |
| ngrok não conecta | Verifique se a câmera está acessível na rede local primeiro |

---

## Estrutura do Projeto

```
esp32cam-webserver/
├── ESP32CAM_WebServer.ino    # Código principal
├── README.md                  # Documentação
├── LICENSE                    # Licença MIT
└── .gitignore                 # Regras de arquivos ignorados
```

---

## Contribuindo

Contribuições são bem-vindas! Sinta-se à vontade para:

1. Fazer um fork do repositório
2. Criar uma branch de feature (`git checkout -b feature/nova-funcionalidade`)
3. Commitar suas mudanças (`git commit -m 'Adiciona nova funcionalidade'`)
4. Fazer push para a branch (`git push origin feature/nova-funcionalidade`)
5. Abrir um Pull Request

---

## Licença

Este projeto está licenciado sob a Licença MIT - veja o arquivo [LICENSE](LICENSE) para detalhes.

---

## Agradecimentos

- [Espressif Systems](https://www.espressif.com/) pelo ESP32 Arduino Core
- [Arduino](https://www.arduino.cc/) pela plataforma de desenvolvimento
- [ngrok](https://ngrok.com/) pelo serviço de túnel

 # 🌱 Sistema Embarcado IoT para Estufa de Precisão em Tempo Real

<p align="center">
  <img src="https://readme-typing-svg.demolab.com?font=Fira+Code&size=20&duration=3000&pause=1000&color=0284C7&center=true&vCenter=true&width=650&lines=ESP32-S3+%2B+FreeRTOS+Multitarefa;Controle+Determin%C3%ADstico+de+Microclima;Efici%C3%AAncia+Energ%C3%A9tica+via+Duty+Cycling;Telemetria+Segura+MQTT+sobre+TLS+1.3" alt="Typing SVG" />
</p>

<p align="center">
  <a href="https://thiago-spba.github.io/estufa-iot-firmware_ucl/">
    <img src="https://img.shields.io/badge/Demo_Ao_Vivo-GitHub_Pages-0284c7?style=for-the-badge&logo=googlechrome&logoColor=white" alt="Demo ao Vivo">
  </a>
  <a href="https://docs.google.com/document/d/1srh7IEBUjqooVZHCZEnmJ8PlO_asVina9pTVnr9Qiek/edit">
    <img src="https://img.shields.io/badge/Relat%C3%B3rio_T%C3%A9cnico-ABNT_PDF-16a34a?style=for-the-badge&logo=googledocs&logoColor=white" alt="Relatório Técnico">
  </a>
  <img src="https://img.shields.io/badge/Plataforma-ESP32--S3-d97706?style=for-the-badge&logo=espressif&logoColor=white" alt="ESP32-S3">
  <img src="https://img.shields.io/badge/Kernel-FreeRTOS_v10.4-15803d?style=for-the-badge" alt="FreeRTOS">
</p>

---

## 📌 Visão Geral do Projeto

Este repositório contém o projeto de engenharia, arquitetura de firmware e prova de conceito (PoC) interativa para o controle de um **nó embarcado inteligente voltado à agricultura de precisão (Agro 4.0)**.

O sistema foi concebido para resolver o desafio de estabilização microclimática em cultivo protegido de alto valor agregado, executando a aquisição simultânea de variáveis ambientais críticas — **temperatura**, **umidade relativa do ar** e **intensidade luminosa (Lux)** — e atuando em malha fechada em tempo real sobre **sistemas de climatização modular (PWM)** e **iluminação de fotoperíodo (Relé)**.

> 🎓 **Contexto Acadêmico:**  
> Projeto desenvolvido para a disciplina de **Programação - Firmware (Fase 1)**  
> **Curso:** Bacharelado em Engenharia da Computação — 6º Semestre  
> **Instituição:** Centro Universitário Celso Lisboa  
> **Autor:** Thiago Fernando

---

## 🚀 Demonstração Interativa ao Vivo

Você pode interagir diretamente com o painel supervisório da estufa através do navegador, sem instalar nada:

👉 **[Acessar Simulador em Tempo Real (GitHub Pages)](https://thiago-spba.github.io/estufa-iot-firmware_ucl/)**

### 🎮 Recursos da Vitrine Interativa:
* **Simulação Climática Dinâmica:** Sliders calibrados para injetar variações de temperatura (15 a 42 °C), umidade (20 a 95%) e iluminância (50 a 2000 Lux).
* **Atuação Mecânica Animada:** Visualização em corte da estufa com hélice de exaustão cuja rotação é proporcional ao ciclo de trabalho do sinal PWM, além de lâmpadas com renderização de feixe luminoso fotoperiódico.
* **Modo Sono Profundo (Deep Sleep):** Demonstração do ciclo de *Duty Cycling*, derrubando a corrente consumida de **28 mA para 15 µA**, acordando por temporizador RTC.
* **Osciloscópio Temporal:** Gráfico contínuo em HTML5 Canvas amostrado a 1 Hz.
* **Interface Autoexplicativa:** Passe o mouse (*hover tooltip*) sobre qualquer indicador para conferir as decisões de engenharia por trás de cada componente.

---

## 🏛️ Arquitetura de Hardware e Software

O projeto emprega uma arquitetura dividida em camadas, garantindo isolamento galvânico entre os circuitos de sinal de 3.3V e os estágios de potência de 12V/110V.

```
                  +-----------------------------------+
                  |        Nuvem / Broker MQTT        |
                  |     (TLS 1.3 - Criptografia)      |
                  +-----------------+-----------------+
                                    ^
                             Wi-Fi 802.11 b/g/n
                                    v
+-----------------------------------+-----------------------------------+
|                           ESP32-S3 SoC                                |
|  [Core 0: Stack Wi-Fi / TLS]             [Core 1: Malhas de Controle] |
|                                                                       |
|  +-----------------------------------------------------------------+  |
|  |                     FreeRTOS Kernel v10.4                       |  |
|  |  * Task_Control (Prioridade 3): Amostragem I2C e PID/PWM        |  |
|  |  * Task_Telemetry (Prioridade 2): Buffer Circular e MQTT        |  |
|  |  * Task_Power_Mgmt (Prioridade 1): Gestão de Sleep e Bateria    |  |
|  |  * Task Watchdog Timer (TWDT - 3000 ms)                         |  |
|  +-----------------------------------------------------------------+  |
+------------------+---------------------------------+------------------+
                   | I2C (400 kHz)                   | PWM / GPIO
                   v                                 v
+------------------+--------------+   +--------------+------------------+
|    Sensores de Precisão         |   |    Estágios de Atuação          |
| * BME280: Temp. (±0.3°C) e Umid.|   | * Driver MOSFET IRF520 (Cooler) |
| * BH1750: Iluminância (1 Lux)   |   | * Módulo Relé Optoacoplado (LED)|
+---------------------------------+   +---------------------------------+
```

---

## 🔌 Pinout e Mapeamento de Periféricos

| Periférico / Atuador | Pino no ESP32-S3 | Interface / Barramento | Parâmetros Técnicos de Operação |
| :--- | :---: | :---: | :--- |
| **Sensor BME280** (Temp/Umid) | GPIO 21 (SDA) / GPIO 22 (SCL) | I2C Master | Endereço `0x76` • Freq. 400 kHz • Resistores pull-up 4,7 kΩ |
| **Sensor BH1750** (Luminosidade) | GPIO 21 (SDA) / GPIO 22 (SCL) | I2C Master | Endereço `0x23` • Modo Contínuo de Alta Resolução (1 Lux) |
| **Climatização** (Cooler DC 12V) | GPIO 18 | Periférico LEDC (PWM) | Frequência 25 kHz • Resolução 10 bits • Modulação linear |
| **Lâmpadas de Suplementação** | GPIO 19 | Saída Digital (Relé) | Isolamento óptico • Lógica invertida (LOW = Relé Fechado) |
| **Sinalizador de Diagnóstico** | GPIO 2 | Saída Digital (LED) | Heartbeat de 1 Hz via timer por software do FreeRTOS |

---

## ⚙️ Malhas de Controle em Tempo Real

1. **Controle Proporcional de Climatização (Ventilação Ativa):**
   * Se a temperatura $T \le 28,0\text{ °C}$, o PWM permanece em **0%** (cooler desligado).
   * Se $T > 28,0\text{ °C}$, o ciclo de trabalho ($D$) é calculado dinamicamente:
     $$D = \min\left(100, (T - 28,0) \times 16,6\right)$$
   * O acionamento em 25 kHz elimina ruído acústico audível e reduz picos de corrente no motor.
   
2. **Controle Fotoperiódico Automatizado:**
   * Caso a iluminância natural captada pelo sensor BH1750 caia para valores **$< 500\text{ Lux}$**, o microcontrolador commuta o relé para acionamento de lâmpadas LED, garantindo o tempo de exposição luminosa essencial à fotossíntese.

3. **Eficiência Energética (*Duty Cycling*):**
   * O sistema opera com transições programadas entre o estado ativo ($28\text{ mA}$) e o estado de suspensão profunda (**$15\text{ µA}$** em *Deep Sleep*), acordando via temporizador RTC de baixíssimo consumo para leitura e despacho de telemetria em lote.

---

## 🔒 Segurança e Resiliência de Firmware

* **Criptografia em Trânsito:** Pacotes MQTT encapsulados sobre **TLS 1.3** com suíte criptográfica **AES-256-GCM**, prevenindo interceptações ou adulteração de comandos remotos.
* **Armazenamento Não-Volátil (Fail-safe):** Buffer circular implementado na partição SPIFFS da memória Flash, garantindo até 72 horas de retenção de dados durante quedas de rede Wi-Fi.
* **Proteção contra Deadlock:** *Task Watchdog Timer (TWDT)* por hardware. Se alguma thread do FreeRTOS for bloqueada por mais de 3 segundos, o sistema executa um reset seguro e restaura os atuadores para o estado seguro padrão.

---

## 📁 Estrutura do Repositório

```bash
├── index.html       # Aplicação web única com simulador, animações e painel supervisório
└── README.md        # Documentação técnica e arquitetural do projeto
```

---

## 👨‍💻 Autor

<table align="center">
  <tr>
    <td align="center">
      <br />
      <b>Thiago Fernando</b><br />
      <sub>Graduando em Engenharia da Computação</sub><br />
      <sub>Centro Universitário Celso Lisboa</sub><br />
      <br />
      <a href="https://github.com/thiago-spba">
        <img src="https://img.shields.io/badge/GitHub-100000?style=flat-square&logo=github&logoColor=white" alt="GitHub Profile" />
      </a>
      <a href="https://docs.google.com/document/d/1srh7IEBUjqooVZHCZEnmJ8PlO_asVina9pTVnr9Qiek/edit">
        <img src="https://img.shields.io/badge/Plano_de_Projeto-Documento_Oficial-0284c7?style=flat-square&logo=googledocs&logoColor=white" alt="Documento Oficial" />
      </a>
    </td>
  </tr>
</table>

---

## 📚 Referências Bibliográficas

* ALENCAR, Robson Lima de. **Estudo sobre a Especificação de Requisitos de Sistemas Embarcados com Metodologias Ágeis**. Recife: UFPE, 2023.
* BARR, Michael; MASSA, Anthony. **Programming Embedded Systems: With C and GNU Development Tools**. 2. ed. O'Reilly Media, 2006.
* CHASE, Otavio. **Sistemas Embarcados**. SBAJovem, 2010.
* COSTA, Isaac Sousa da et al. Monitoramento IoT de planta de bombeamento fotovoltaico utilizando sistema embarcado Linux. **Enciclopédia Biosfera**, v. 18, n. 37, p. 349-361, 2021.
* ESPRESSIF SYSTEMS. **ESP32-S3 Series Datasheet: 2.4 GHz Wi-Fi & Bluetooth 5 (LE) SoC**. v1.4, 2023.
* FRANCO, Eduardo Ferreira. **Um modelo de gerenciamento de projetos baseado nas metodologias ágeis Extreme Programming (XP) e Scrum**. São Paulo: USP, 2007.


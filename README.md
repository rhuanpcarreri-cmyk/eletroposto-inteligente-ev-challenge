# ⚡ Eletroposto Inteligente — EV Challenge 2026

**Sprint 2 — Prova de Conceito Funcional**
Sistema de recarga inteligente para veículos elétricos integrado ao ecossistema **GoodWe**, combinando **energia solar fotovoltaica**, **armazenamento em baterias (BESS)** e **gestão inteligente de consumo**.

> Protótipo funcional desenvolvido em **ESP32 (simulado no Wokwi)** que demonstra, em tempo real, o núcleo da solução proposta na Sprint 1: decidir automaticamente de qual fonte puxar energia, na prioridade **SOLAR → BATERIA → REDE**, reduzindo custo e emissões de carbono.

---

## 📌 Índice
- [O problema](#-o-problema)
- [A solução demonstrada nesta prova de conceito](#-a-solução-demonstrada-nesta-prova-de-conceito)
- [Arquitetura do sistema](#-arquitetura-do-sistema)
- [Lógica de gestão de energia](#-lógica-de-gestão-de-energia)
- [Componentes e justificativa técnica](#-componentes-e-justificativa-técnica)
- [Energias renováveis e sustentabilidade](#-energias-renováveis-e-sustentabilidade)
- [Dados gerados pelo sistema](#-dados-gerados-pelo-sistema)
- [Como executar](#-como-executar)
- [Estrutura do repositório](#-estrutura-do-repositório)
- [Equipe](#-equipe)

---

## 🎯 O problema

O crescimento dos veículos elétricos no Brasil pressiona a infraestrutura energética. Em horários de pico, vários carregamentos simultâneos sobrecarregam a rede local e elevam o custo operacional dos eletropostos. Há ainda uma contradição: boa parte dos EVs é carregada com energia de fontes **não renováveis**, anulando parte do ganho ambiental da mobilidade elétrica.

## 💡 A solução demonstrada nesta prova de conceito

A Sprint 1 propôs um eletroposto que une geração solar, baterias BESS e software de controle. Nesta Sprint 2, implementamos e colocamos para **funcionar o cérebro dessa solução**: o algoritmo de **gestão inteligente de energia** rodando em um microcontrolador ESP32.

A cada hora simulada, o sistema lê três variáveis — geração solar, demanda do carro e estado de carga da bateria — e decide a fonte de energia mais barata e mais limpa disponível, **reservando a rede convencional como último recurso, evitada principalmente no horário de ponta**.

---

## 🏗️ Arquitetura do sistema

```
                    ┌─────────────────────────────┐
   ☀️ Painéis        │     ESP32 (controlador)     │   📟 LCD 16x2
   Solares  ───────▶ │                             │ ─────▶  modo / SoC / R$
                     │   Lê geração, demanda e SoC │
   🔋 Bateria  ◀────▶ │   Decide a fonte de energia │   🟢🟡🔴 LEDs (fonte ativa)
   BESS              │   SOLAR → BATERIA → REDE     │ ─────▶
                     │                             │   🔔 Buzzer (rede na ponta)
   🔌 Rede   ───────▶ │                             │
   elétrica          │   Registra dados (Serial)   │ ─────▶  📊 dataset 24h
                     └─────────────────────────────┘
```

**Camadas:**
1. **Geração** — arranjo fotovoltaico (10 kWp) é a fonte primária.
2. **Armazenamento** — banco de baterias BESS (15 kWh) guarda o **excedente solar**.
3. **Controle (ESP32)** — decide, monitora e registra; é a peça que esta PoC torna funcional.
4. **Interface** — LCD, LEDs e buzzer comunicam o estado ao operador em tempo real.

---

## 🧠 Lógica de gestão de energia

A cada hora, com a demanda do carro `D`, a geração solar `G` e a carga da bateria `SoC`:

```
SE não há carro conectado:
    excedente solar carrega a bateria (até a capacidade máxima)

SENÃO:
    SE  G ≥ D                → usa SOLAR; excedente carrega a bateria      🟢
    SENÃO:
        usa a solar disponível e calcula o déficit
        SE bateria acima da reserva mínima → BATERIA cobre o déficit       🟡
        SE ainda falta energia            → REDE cobre o restante          🔴
                                            (buzzer dispara se for PONTA)
```

> A bateria **só é carregada com excedente solar** — nunca pela rede. Isso garante que a energia armazenada e devolvida ao carro seja sempre limpa, e é o que torna a mobilidade "verdadeiramente sustentável".

**Janela de ponta:** 18h–21h, com tarifa de R$ 1,20/kWh contra R$ 0,60/kWh fora de ponta (valores ilustrativos e parametrizáveis). É justamente nesse intervalo que a bateria — carregada de graça pelo sol ao meio-dia — assume a recarga e evita a rede mais cara.

---

## 🔧 Componentes e justificativa técnica

| Componente | Pino (GPIO) | Papel no sistema | Por que esse componente |
|---|---|---|---|
| **ESP32 DevKit V1** | — | Controlador central | Wi-Fi nativo, várias entradas analógicas e baixo consumo — ideal para Edge Computing em campo |
| **LCD 16x2 I²C** | SDA 21 / SCL 22 | Interface do operador | Usa só 2 fios (I²C) e mostra fonte, SoC e economia em tempo real |
| **Potenciômetro "Solar"** | 34 | Simula o fator climático (nuvens) | Permite demonstrar ao vivo a queda de geração e a resposta do sistema |
| **Potenciômetro "Demanda"** | 35 | Simula a potência pedida pelo carro | Mostra o sistema reagindo a cargas diferentes |
| **Botão CARRO** | 25 | Conecta/desconecta o veículo | Dispara o ciclo de recarga |
| **Botão PAUSA** | 26 | Pausa o relógio simulado | Congela uma cena para explicação no vídeo |
| **LED verde** | 4 | Energia limpa (solar/bateria) | Feedback visual imediato da fonte ativa |
| **LED amarelo** | 5 | Bateria assumindo o déficit | — |
| **LED vermelho** | 2 | Uso da rede elétrica | — |
| **Buzzer** | 15 | Alerta de rede na ponta | Sinaliza o evento que o sistema busca evitar |

> **Eficiência energética como princípio de projeto:** o firmware usa temporização não bloqueante (`millis()`), o LCD e os sensores trabalham em protocolos de baixo consumo (I²C/ADC) e o ESP32 foi escolhido por seu baixo consumo — coerente com um produto que deve operar de forma autônoma e sustentável.

---

## ♻️ Energias renováveis e sustentabilidade

A solução materializa, de forma mensurável, os conceitos trabalhados no semestre:

- **Fonte primária renovável:** a energia solar atende a carga sempre que disponível.
- **Armazenamento de energia limpa:** o excedente solar do meio-dia é guardado na BESS e devolvido no fim do dia, deslocando consumo da rede para fora do horário de pico (*peak shaving*).
- **Redução de emissões:** cada kWh entregue por solar/bateria evita o CO₂ que seria emitido pela rede.
- **Redução de custo e da pressão na rede:** ao evitar a rede na ponta, corta-se o kWh mais caro e alivia-se a infraestrutura urbana.
- **Viabilidade e escalabilidade:** a lógica é replicável em estacionamentos, condomínios, shoppings e rodovias, usando equipamentos GoodWe já consolidados no mercado.

---

## 📊 Dados gerados pelo sistema

A mesma lógica do firmware foi executada para um **dia de referência de 24h** (script [`simulacao_24h.py`](simulacao_24h.py)), gerando o dataset [`dados_simulacao_24h.csv`](dados_simulacao_24h.csv). No Wokwi, esses mesmos dados saem pelo **Serial Monitor** hora a hora.

### Resultado do dia simulado

| Indicador | Valor |
|---|---|
| Energia total entregue ao carro | **46,0 kWh** |
| Atendida por energia renovável (solar + bateria) | **57,8 %** (26,6 kWh) |
| Atendida pela rede elétrica | 42,2 % (19,4 kWh) |
| Custo **sem** o sistema (tudo da rede) | R$ 40,92 |
| Custo **com** o sistema | **R$ 16,87** |
| **Economia no dia** | **R$ 24,05 (59 %)** |
| **CO₂ evitado no dia** | **3,19 kg** |

A bateria, carregada pelo excedente solar do meio-dia, assume a recarga das **18h–19h (ponta)** e evita a rede cara justamente quando ela é mais cara.

> Valores de tarifa e fator de emissão são ilustrativos e parametrizáveis no topo do firmware e do script. A finalidade da PoC é comprovar a **lógica e a viabilidade técnica**, não cravar números regulatórios.

---

## ▶️ Como executar

### No Wokwi (recomendado)
1. Acesse [wokwi.com](https://wokwi.com) e crie um novo projeto **ESP32**.
2. Substitua o conteúdo de `sketch.ino` pelo arquivo [`eletroposto_inteligente.ino`](eletroposto_inteligente.ino).
3. Abra a aba `diagram.json` e cole o conteúdo de [`diagram.json`](diagram.json).
4. Em **Library Manager**, adicione **LiquidCrystal I2C** (ou use o `libraries.txt`).
5. Clique em **▶ Start the simulation**.
6. Operação:
   - Aperte **CARRO** para conectar o veículo.
   - Gire o potenciômetro **Solar** para simular nuvens; o **Demanda** para variar a potência pedida.
   - Acompanhe o **LCD**, os **LEDs** e o **Serial Monitor** (115200 baud).
   - Use **PAUSA** para congelar uma cena.

### Reproduzir os dados de 24h (opcional)
```bash
pip install matplotlib
python3 simulacao_24h.py
```
Gera o dataset `dados_simulacao_24h.csv`.

---

## 📁 Estrutura do repositório

```
.
├── eletroposto_inteligente.ino   # firmware ESP32 (lógica de gestão de energia)
├── diagram.json                  # circuito do Wokwi
├── wokwi.toml                    # configuração do Wokwi
├── libraries.txt                 # dependências (LiquidCrystal I2C)
├── simulacao_24h.py              # simulação de 24h (mesma lógica) -> dados + gráficos
├── dados_simulacao_24h.csv       # dataset gerado (prova de conceito)
└── README.md
```

---

## 👥 Equipe

Mauricio Bertuci Saletti - RM571229 • 
Mateus Eduardo da Cruz Rocha - RM570736 • 
Lucas Caram Bueno - RM570158 • 
Rhuan Pacheco Carreri - RM570129 • 
Leonardo Fortini Marcelo - RM572566 • 
Nicolas Andrade Rodrigues - RM572782 • 

**FIAP — EV Challenge 2026 · Sprint 2 — Prova de Conceito Funcional**

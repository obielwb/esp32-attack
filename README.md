# **Relatório de Análise de Segurança**

ESP32 Web Server \- Controle de GPIOs via WiFi

Grupo 03 \- 09/12/2025

# **1\. Introdução**

Este relatório apresenta uma análise estática de segurança do código de um servidor web embarcado em ESP32. O sistema permite o controle de dois GPIOs (26 e 27\) através de uma interface web acessível via rede WiFi.

O objetivo é identificar vulnerabilidades de segurança, mapear possíveis ataques, avaliar seus riscos e propor recomendações de mitigação baseadas nas melhores práticas de segurança para dispositivos IoT.

# **2\. Descrição do Sistema**

## **2.1 Arquitetura**

O sistema é composto por um microcontrolador ESP32 que hospeda um servidor HTTP na porta 80\. Dois LEDs estão conectados aos pinos GPIO 26 e GPIO 27 através de resistores limitadores de corrente. A comunicação é feita via WiFi e o controle é realizado por requisições HTTP GET.

## **2.2 Funcionalidades**

* Conexão automática à rede WiFi configurada  
* Servidor web na porta 80 servindo página HTML  
* Controle ON/OFF dos GPIOs 26 e 27 via botões web  
* Feedback visual do estado atual de cada GPIO  
* Endpoints: /26/on, /26/off, /27/on, /27/off

# **3\. Análise Estática de Vulnerabilidades**

A análise estática do código-fonte revelou múltiplas vulnerabilidades de segurança, categorizadas conforme a tabela de referência de vulnerabilidades em sistemas IoT.

| Vulnerabilidade | Evidência no Código | Risco Associado |
| ----- | ----- | ----- |
| Comunicação não segura | Uso de HTTP (porta 80\) sem TLS/SSL. Linha: WiFiServer server(80) | Interceptação e modificação de dados em trânsito |
| Autenticação fraca/inexistente | Nenhum mecanismo de autenticação implementado. Qualquer requisição GET é processada | Acesso não autorizado ao sistema e controle dos GPIOs |

# **4\. Análise de Ataques**

## **4.1 Ataque 1: Man-in-the-Middle (MitM) com Interceptação de Tráfego**

### **Descrição**

Este ataque explora a vulnerabilidade de comunicação não segura (HTTP sem criptografia). Um atacante posicionado na mesma rede pode interceptar, visualizar e modificar todo o tráfego entre o cliente e o ESP32.

### **Passo-a-Passo**

1. O atacante conecta-se à mesma rede WiFi que o ESP32  
2. Executa ARP Spoofing usando ferramentas como arpspoof ou Ettercap para se posicionar entre o cliente e o gateway  
3. Utiliza Wireshark ou tcpdump para capturar pacotes HTTP em texto plano  
4. Analisa as requisições GET para identificar os endpoints de controle (/26/on, /27/off, etc.)  
5. Pode modificar requisições em trânsito usando ferramentas como mitmproxy ou Bettercap  
6. Injeta comandos maliciosos alterando requisições legítimas (ex: trocar /26/off por /26/on)

### **Análise de Risco**

| Probabilidade | ALTA (4/5) \- Ferramentas de ataque amplamente disponíveis; requer apenas acesso à mesma rede WiFi; ataques MitM são bem documentados e automatizáveis. |
| :---- | :---- |
| **Impacto** | ALTO (4/5) \- Permite controle não autorizado dos GPIOs; espionagem de padrões de uso; possibilidade de causar danos físicos se os GPIOs controlarem atuadores críticos. |
| **Risco Resultante** | **ALTO (4/5)** |

**Justificativa:** A combinação de alta probabilidade de execução (baixa barreira técnica) com alto impacto potencial resulta em um risco classificado como ALTO. O HTTP em texto plano é um vetor de ataque bem conhecido e facilmente explorável.

## **4.2 Ataque 2: Acesso Não Autorizado Direto (Unauthorized Access)**

### **Descrição**

Este ataque explora a ausência total de autenticação no servidor web. Qualquer dispositivo na rede pode enviar comandos ao ESP32 sem necessidade de credenciais, tokens ou qualquer forma de verificação de identidade.

### **Passo-a-Passo**

1. O atacante obtém acesso à rede WiFi (via credenciais conhecidas, WiFi aberto ou ataque de força bruta)  
2. Realiza varredura de rede com nmap para descobrir hosts ativos: nmap \-sn 192.168.1.0/24  
3. Identifica o ESP32 pela porta 80 aberta: nmap \-p 80 192.168.1.0/24  
4. Acessa o IP do ESP32 via navegador e visualiza a interface de controle  
5. Envia comandos diretamente via URL: curl http://192.168.1.X/26/on  
6. Automatiza controle malicioso com scripts para ligar/desligar repetidamente

### **Análise de Risco**

| Probabilidade | MUITO ALTA (5/5) \- Não requer ferramentas especializadas; qualquer navegador web é suficiente; descoberta trivial via varredura de rede; zero conhecimento técnico avançado necessário. |
| :---- | :---- |
| **Impacto** | ALTO (4/5) \- Controle total e irrestrito dos GPIOs; possibilidade de danos em sistemas conectados; violação de privacidade; potencial para ataques de negação de serviço. |
| **Risco Resultante** | **CRÍTICO (5/5)** |

**Justificativa:** A probabilidade máxima combinada com alto impacto resulta em risco CRÍTICO. A ausência de autenticação é a vulnerabilidade mais grave do sistema, permitindo que qualquer pessoa na rede controle o dispositivo sem qualquer barreira.

**Evidência do Ataque 2:** https://youtu.be/2Se2nLSAY4E

# **5\. Tabela Consolidada de Ataques**

A tabela abaixo apresenta todos os ataques identificados, ordenados do maior risco para o menor risco. O cálculo do risco segue a fórmula: Risco \= Probabilidade × Impacto (escala 1-5 para cada fator).

| Título do Ataque | Probabilidade | Impacto | Risco |
| ----- | :---: | :---: | :---: |
| Acesso Não Autorizado Direto | 5/5 (Muito Alta) | 4/5 (Alto) | **CRÍTICO** |
| Man-in-the-Middle (MitM) | 4/5 (Alta) | 4/5 (Alto) | **ALTO** |

	

# **6\. Recomendações de Mitigação**

## **6.1 Prioridade Crítica**

* **Implementar autenticação:** Adicionar Basic Auth ou Digest Auth no servidor HTTP  
* **Migrar para HTTPS:** Utilizar biblioteca WiFiClientSecure com certificados TLS

## **6.2 Prioridade Alta**

* **Rate Limiting:** Implementar controle de requisições por IP/tempo  
* **Isolamento de rede:** Colocar dispositivo IoT em VLAN separada

# **7\. Conclusão**

A análise estática revelou que o código do ESP32 Web Server apresenta vulnerabilidades críticas de segurança, sendo as mais graves a ausência de autenticação e a comunicação em texto plano. Estas falhas expõem o sistema a ataques triviais que podem resultar em controle não autorizado dos GPIOs.

Recomenda-se fortemente a implementação das mitigações de prioridade crítica antes de utilizar este sistema em qualquer ambiente de produção. O código atual é adequado apenas para fins educacionais em ambientes controlados e isolados.

# **8\. Contribuição Individual**

Antonio Cillio - Revisão do Relatório e Preparação do Pitch <br/>
Carlos Quaglia - Escrita do Relatório <br/>
Caue Taddeo  - Montagem do circuito e Acompanhamento nas análises dos ataques <br/>
Gabriel Bartmanonicz - Montagem do Circuito, Análise do Ataque 1 <br/>
Rafael Santana - Escrita do relatório, Análise do Ataque 2 <br/>
Thulio Bacco - Escrita do Relatório  <br/>

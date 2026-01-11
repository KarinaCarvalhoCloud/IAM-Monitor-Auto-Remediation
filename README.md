#  IAM Monitor: Auto-Remediação de Políticas de Acesso na AWS

![Status: Em Desenvolvimento](https://img.shields.io/badge/Status-Em%20Desenvolvimento-yellow) 
![Foco: Segurança Cloud](https://img.shields.io/badge/Foco-SecOps%20%7C%20IAM%20%7C%20Compliance-red)
![Linguagem: Python](https://img.shields.io/badge/Linguagem-Python%20%7C%20Boto3-blue)

Este projeto implementa uma solução de **auto-remediação serverless** para proteger contas AWS contra o **"Security Drift"** – a degradação gradual das políticas de segurança. O foco é garantir que o **Princípio do Menor Privilégio** seja mantido, detectando e corrigindo automaticamente políticas de IAM excessivamente permissivas.

---

##  O Desafio: Ameaça de Escala de Privilégios

A gestão manual do IAM em ambientes dinâmicos leva inevitavelmente à criação acidental de políticas com permissões excessivas (ex: ações `*` ou `Recurso: *`). Estes são vetores críticos de ataque que permitem a um invasor **escalar privilégios** e acessar dados sensíveis.

### Objetivos Chave
1.  **Monitoramento Contínuo (Real-Time):** Detectar a criação ou modificação de políticas de IAM que violem regras de segurança (**Guardrails**).
2.  **Remediação Imediata:** Reduzir o **tempo de resposta** a vulnerabilidades, aplicando uma política de quarentena ou de negação explícita automaticamente.

---

##  Arquitetura de Auto-Remediação (SecOps)

O sistema opera em um fluxo de trabalho Serverless e orientado a eventos, crucial para a automação de **SecOps**:

1.  **Event Source (Monitoramento):** O **AWS CloudTrail** é monitorado pelo **CloudWatch Events** (ou EventBridge) para eventos críticos de IAM (ex: `AttachUserPolicy`).
2.  **Dispatcher (Gatilho):** O evento aciona uma função **AWS Lambda (Python)**.
3.  **Core Logic (Análise):** A função Lambda utiliza o **SDK Boto3** para inspecionar a política em busca de **padrões de risco** (ex: `Action: "*"`, uso de `Resource: "*"` em contextos sensíveis).
4.  **Action (Remediação):** Se insegura, a política é automaticamente **desanexada** do usuário/role (quarentena) e substituída por uma política de negação.
5.  **Notificação:** Alerta é enviado via **Amazon SNS** para a equipe de segurança, detalhando a política corrigida e o motivo.

#### Diagrama de Arquitetura Proposto

* `![Diagrama de Arquitetura do IAM Monitor](assets/iam_monitor_architecture.png)`)*

---

##  Detalhes da Implementação (Python / Boto3)

O coração deste projeto é o script Python da função Lambda, que garante uma lógica de inspeção rápida e segura.

* **Validação de Políticas:** Uso do Boto3 para obter a política e garantir a validade da sintaxe (JSON).
* **Lógica de Inspeção:** Implementação de funções que iteram sobre os Statements da política (`"Statement": [...]`) e verificam a presença de chaves de alto risco.
* **Função de Remediação Segura:** O método `detach_user_policy()` do Boto3 é utilizado para remover o vínculo de políticas não-conformes, garantindo que o atacante perca imediatamente os privilégios.
* **Princípio de Segurança do Código:** A própria função Lambda possui privilégios mínimos de IAM para interagir apenas com os serviços necessários (Princípio do Menor Privilégio na prática).

---

##  Resultado e Impacto no Negócio

Este projeto é uma prova da capacidade de construir soluções de **Segurança Orientada a Eventos**, fundamentais para a cultura de **SecOps**.

| Métrica de Segurança | Cenário Sem Automação | Cenário Com Automação (IAM Monitor) |
| :--- | :--- | :--- |
| **Risco de Escalada de Privilégios** | Alto e dependente de auditoria | **Baixo** e mitigado em tempo real |
| **Tempo de Resposta à Vulnerabilidade** | Horas a dias (via auditoria manual) | **< 5 minutos** (via auto-remediação Serverless) |
| **Conformidade com Menor Privilégio** | Propenso a erros humanos | Próximo a **100%** (Garantido por código) |

---

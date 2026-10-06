<div align="center">

# ✂️ L&K BARBEARIA — SISTEMA DE GESTÃO INTELIGENTE

**De caderninho para um sistema completo de gestão em tempo real.**  
*Um projeto real, implantado em produção, construído do zero.*

<br>

![Status](https://img.shields.io/badge/Status-Em%20Produção-brightgreen?style=for-the-badge)
![Google Sheets](https://img.shields.io/badge/Google%20Sheets-34A853?style=for-the-badge&logo=googlesheets&logoColor=white)
![Apps Script](https://img.shields.io/badge/Apps%20Script-4285F4?style=for-the-badge&logo=google&logoColor=white)
![Looker Studio](https://img.shields.io/badge/Looker%20Studio-4285F4?style=for-the-badge&logo=looker&logoColor=white)
![HTML](https://img.shields.io/badge/HTML-E34F26?style=for-the-badge&logo=html5&logoColor=white)
![CSS](https://img.shields.io/badge/CSS-663399.svg?style=for-the-badge&logo=CSS&logoColor=white)
![JS](https://img.shields.io/badge/JavaScript-F7DF1E.svg?style=for-the-badge&logo=JavaScript&logoColor=black)
![GitHub Pages](https://img.shields.io/badge/GitHub%20Pages-181717?style=for-the-badge&logo=github&logoColor=white)

<br>

🔗 **[Ver Site ao Vivo](https://gustavo-henrique-santos.github.io/lk-barber-command-center/)** &nbsp;|&nbsp; 📊 **[Acessar Dashboard](#)** &nbsp;|&nbsp; 👤 **[LinkedIn](https://www.linkedin.com/in/santosgustavohenrique)**

</div>

---

## 🎯 O Problema Real

Uma barbearia com 3 barbeiros, atendendo dezenas de clientes por dia, controlava tudo no **caderninho**, sem visibilidade de faturamento, sem controle de comissões, sem rastreio de estoque ou custos operacionais.

**O dono não sabia responder:**
- Quanto faturou essa semana? E esse mês?
- Qual barbeiro performou melhor?
- Quais serviços geram mais margem?
- O estoque de produtos está cobrindo as vendas?
- Os custos fixos estão crescendo?

**A solução:** Um sistema de gestão completo, construído com ferramentas acessíveis, implantado em produção, com custo de manutenção zero.

---

## 🏗️ Arquitetura da Solução

![Arquitetura do Sistema](./architecture.svg)

O sistema foi projetado em **4 camadas independentes e integradas**:

| Camada | Tecnologia | Responsabilidade |
|--------|-----------|-----------------|
| **Entrada** | Google Forms + Site HTML | Coleta de dados pelos barbeiros |
| **Automação** | Google Apps Script | Processamento e roteamento dos dados |
| **Armazenamento** | Google Sheets (6 abas) | Base de dados estruturada |
| **Visualização** | Looker Studio | Dashboard gerencial em tempo real |

---

## ⚙️ Como Funciona

### Fluxo de Atendimentos e Vendas

```
Barbeiro abre o site → Clica no formulário → Preenche o atendimento
      ↓
Google Forms registra na aba Forms_LK_Atendimentos
      ↓
Apps Script (trigger automático) detecta o envio
      ↓
Script busca o preço atual na CONFIG (nunca retroage dados históricos)
      ↓
Linha processada e inserida na aba REGISTROS com 15 colunas:
Timestamp · Tipo · Barbeiro · Serviço · Preço · Pagamento ·
Qtde · ID Único · Mês/Ano · Dia da Semana · Comissão Barbeiro ·
Valor Barbearia · Taxa Máquina · Líquido Barbearia
```

### Fluxo de Compras

```
Compra realizada → Formulário preenchido (produto, fornecedor, qtde, preço custo)
      ↓
Apps Script processa e busca o código do produto na CONFIG automaticamente
      ↓
Linha inserida em COMPRAS com: Data · Produto · Código · Tipo ·
Fornecedor · Preço Custo · Qtde · Total · Forma de Pagamento
      ↓
CONFIG atualiza automaticamente o Preço Custo via QUERY
(sempre o valor mais recente por produto)
```

### Fluxo de Custos Operacionais

```
Custo ocorre (aluguel, água, manutenção, investimento...)
      ↓
Formulário preenchido → Script processa
      ↓
Classifica automaticamente como OPEX ou CAPEX
      ↓
Inserido em CUSTOS com: Data · Categoria · Item · Valor · Observação · Tipo Contábil
```

---

## 📁 Estrutura da Planilha Mestre

```
LK_Barbearia (Google Sheets)
│
├── 📊 REGISTROS          → Consolidação de todos os atendimentos e vendas
├── 🛒 COMPRAS            → Histórico de compras de produtos
├── 💰 CUSTOS             → Lançamentos de custos operacionais
├── ⚙️ CONFIG             → Central de configurações, preços e parâmetros
│
├── 📋 Forms_LK_Atendimentos      → Formulário: serviços realizados
├── 📋 Forms_LK_Vendas_Produtos   → Formulário: produtos vendidos
├── 📋 Forms_LK_Compras           → Formulário: compras de estoque
└── 📋 Forms_LK_Custos_Operacionais → Formulário: custos da barbearia
```

---

## 🧠 Decisões Técnicas Relevantes

### Por que não usar PROCV/VLOOKUP nos REGISTROS?

Uma das decisões mais importantes do projeto: **os preços são capturados no momento do lançamento** e gravados como valor estático, não como fórmula que referencia a CONFIG.

Se usássemos PROCV, uma atualização de preço em dezembro retroagiria todos os registros históricos de janeiro, **corrompendo completamente o histórico financeiro**. A abordagem escolhida garante integridade dos dados históricos.

### Por que o preço de custo usa QUERY na CONFIG?

```
=SEERRO(QUERY(COMPRAS!$A$3:$I$1000;
  "SELECT F WHERE B='"&M5&"' AND D='Para Revenda' ORDER BY A DESC LIMIT 1");0)
```

Essa fórmula sempre traz o **último preço de custo registrado** para cada produto. Isso permite que a margem na CONFIG seja sempre calculada com base no custo mais atual, sem precisar de manutenção manual.

### Por que o OPEX/CAPEX é calculado pelo script e não por fórmula?

Igual ao raciocínio dos preços: valor estático no lançamento garante que, mesmo que a categoria seja editada depois, o tipo contábil original é preservado.

### Tratamento de decimais — locale BR vs. padrão internacional

O Google Sheets no Brasil armazena valores com **vírgula** como separador decimal. O script normaliza isso antes de persistir qualquer cálculo, evitando que valores textuais (`"35,00"`) gerem zeros silenciosos:

```javascript
function toNumber(valor) {
  if (typeof valor === 'number') return valor;
  if (!valor || valor === '') return 0;
  var limpo = valor.toString()
    .replace(/R\$\s*/g, '')
    .replace(/\./g, '')      // remove separador de milhar
    .replace(',', '.')       // converte decimal BR → JS
    .trim();
  var num = parseFloat(limpo);
  return isNaN(num) ? 0 : num;
}
```

Essa função é aplicada em todos os valores numéricos retornados da CONFIG antes de qualquer cálculo.

### Ranges da CONFIG projetados para crescimento

Os ranges de busca na CONFIG foram definidos com folga intencional para suportar a adição de novos serviços e produtos sem necessidade de alterar o script:

```javascript
sheetConfig.getRange("F5:G30")   // serviços — suporta até 26 entradas
sheetConfig.getRange("M5:N20")   // produtos  — suporta até 16 entradas
```

---

## 💈 Catálogo de Serviços

O sistema suporta atualmente **17 serviços**, incluindo serviços individuais, combos e tratamentos:

| Código | Serviço | Preço |
|--------|---------|-------|
| SRV001 | Corte | R$ 35,00 |
| SRV002 | Barba | R$ 30,00 |
| SRV003 | Sobrancelha | R$ 10,00 |
| SRV004 | Combo Corte + Barba + Sobrancelha | R$ 70,00 |
| SRV005 | Combo Corte + Sobrancelha + Bigode | R$ 50,00 |
| SRV006–SRV009 | Relaxamento (4 comprimentos) | R$ 35,00–R$ 50,00 |
| SRV010–SRV013 | Progressiva (4 comprimentos) | R$ 50,00–R$ 80,00 |
| SRV014 | Combo Corte + Barba | R$ 60,00 |
| SRV015 | Combo Corte + Sobrancelha | R$ 40,00 |
| SRV016 | Combo Corte + Bigode | R$ 40,00 |
| SRV017 | Combo Barba + Sobrancelha | R$ 40,00 |

---

## 🖥️ Command Center (Site — Versão Demo)

O site foi construído em HTML, CSS e JS puro, hospedado no **GitHub Pages**, e serve como ponto de acesso central para os barbeiros.

🔗 [gustavo-santos-analytics.github.io/lk-barber-command-center](https://gustavo-santos-analytics.github.io/lk-barber-command-center/)

**Funcionalidades:**
- Botões que abrem uma **prévia visual** de cada formulário em modal
- Aviso fixo de versão demonstrativa no canto inferior esquerdo
- Botão para o Dashboard (Looker Studio)
- **Manual de Uso interativo** — página dedicada com passo a passo para os barbeiros, FAQ accordion e exemplos práticos por situação
- Layout responsivo para uso no celular

> 📸 *Esta é a versão demonstrativa pública. Os botões exibem prints dos formulários reais com aviso de dados fictícios, preservando a confidencialidade do cliente. O sistema em produção opera com acesso direto aos formulários via link.*

**Estrutura de assets:**
```
assets/
├── Logo_LK.png
└── demo/
    ├── form-bancada1.png          ← prévia do formulário de atendimentos
    ├── form-vendas-produtos.png   ← prévia do formulário de vendas
    ├── form-custos.png            ← prévia do formulário de custos
    └── form-compras.png           ← prévia do formulário de compras

manual.html                        ← manual de uso interativo (FAQ accordion,
                                      passo a passo, exemplos por situação)
```

---

## 📊 Dashboard — Looker Studio

> 🚧 **Em desenvolvimento** — será publicado em breve.

**Modelagem planejada:**
- `fRegistros` — fato central com todos os atendimentos e vendas
- `fCompras` — fato de entradas de estoque
- `fCustos` — fato de despesas operacionais
- `dServicos`, `dProdutos`, `dBarbeiros` — dimensões de apoio

**Páginas planejadas:**
1. **Visão Geral** — faturamento bruto, ticket médio, MoM
2. **Desempenho Barbeiros** — ranking, atendimentos, dias mais movimentados
3. **Serviços e Produtos** — top serviços por volume e receita, mix de pagamentos
4. **Custos e Margem** — OPEX vs. CAPEX, margem bruta estimada, evolução mensal
5. **Estoque e Compras** — investimento em produtos, evolução por mês

*O dashboard utilizará dados fictícios (11 meses — Jul/2025 a Mai/2026) para demonstração pública, preservando a confidencialidade do cliente.*

---

## 🚀 Tecnologias Utilizadas

| Tecnologia | Uso no Projeto |
|-----------|---------------|
| **Google Forms** | Interface de entrada de dados para os barbeiros |
| **Google Apps Script (JS)** | Automação, triggers e processamento de dados |
| **Google Sheets** | Base de dados estruturada e cálculos dinâmicos |
| **Looker Studio** | Dashboard gerencial e visualizações |
| **HTML / CSS / JS** | Site Command Center dos barbeiros |
| **GitHub Pages** | Hospedagem gratuita do site |

---

## 👨‍💻 Sobre o Projeto e o Autor

Este projeto foi concebido, arquitetado e desenvolvido por **Gustavo Santos**, Analista de Dados, como solução real para um cliente real.

O objetivo foi ir além de uma planilha comum, entregando um sistema integrado, com automação robusta, controle histórico e inteligência nos dados, usando ferramentas acessíveis e de custo zero para o cliente.

**Habilidades demonstradas neste projeto:**
- Arquitetura de dados para pequenos negócios
- Automação com Google Apps Script
- Modelagem de dados em planilhas
- Integridade de dados históricos
- Desenvolvimento web (HTML/CSS/JS)
- Business Intelligence com Looker Studio
- Entrega de valor para cliente real

<br>

<div align="center">

**[🔗 LinkedIn — Gustavo Santos](https://www.linkedin.com/in/santosgustavohenrique)**

*Se este projeto te inspirou ou foi útil, deixa uma ⭐ no repositório!*

</div>

---

<div align="center">
<sub>Desenvolvido para a L&K Barbearia · 2026</sub>
</div>

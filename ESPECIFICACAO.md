# Especificação do Sistema — Controle de Consumo

## 1. Visão Geral

Aplicativo web PWA para controle de consumo em estabelecimentos. Permite cadastrar itens consumidos, controlar quantidades, visualizar totalizadores em tempo real e gráfico de distribuição por categoria. Funciona offline após o primeiro acesso.

---

## 2. Arquitetura

| Camada | Tecnologia |
|--------|-----------|
| Frontend | HTML5 + CSS3 (Tailwind CSS via CDN) + JavaScript Vanilla |
| Gráficos | Chart.js 4.x (via CDN) |
| Ícones | Font Awesome 6.4.0 (via CDN) |
| Persistência | localStorage do navegador |
| PWA | Service Worker + Web App Manifest |
| Ícones | PNG 192x192 e 512x512 |

### Estrutura de Arquivos

```
index.html          — Aplicação completa (HTML+CSS+JS)
manifest.json       — Manifesto PWA
sw.js               — Service Worker (cache-first)
icon-192.png        — Ícone PWA 192x192
icon-512.png        — Ícone PWA 512x512
Leia-me.md          — Documentação inicial
ESPECIFICACAO.md    — Especificação do sistema
test.html           — Testes automatizados (QUnit)
```

---

## 3. Funcionalidades

### 3.1 Cadastro de Itens
- **Campos**: Nome (texto), Categoria (comida/bebida), Preço unitário (R$), Quantidade
- **Validação**: Nome obrigatório, preço > 0, quantidade >= 1
- **Atalho**: Enter nos campos do formulário confirma a adição
- **Incremento inteligente**: mesmos nome + categoria somam quantidades em vez de criar duplicata

### 3.2 Controles na Tabela
- Botões **+** e **-** em cada linha para ajustar quantidade
- Ao chegar em 0, pergunta se deseja remover o item
- Botão de **excluir** diretamente na linha
- Botão **"Zerar Conta"** no cabeçalho (com confirmação)

### 3.3 Totalizadores em Tempo Real
- **Total de Itens**: soma de todas as quantidades
- **Valor Total da Conta**: em formato monetário BRL
- **Ticket Médio por Item**: valor total / quantidade total

### 3.4 Gráfico de Rosca (Doughnut)
- Distribuição do valor gasto entre Comidas (laranja) e Bebidas (azul)
- Tooltip com valor formatado em R$
- Estado vazio com indicador cinza e mensagem "Adicione itens para ver o gráfico"

### 3.5 Persistência Local
- Dados salvos no `localStorage` sob a chave `consumoApp`
- Estrutura completa: `{ mesas, mesaAtual, historico[], tema }`
- Migração automática do formato legado (`consumoItens`)
- Restaurados automaticamente ao recarregar a página

### 3.6 Tema Escuro (Dark Mode)
- Botão de alternância no cabeçalho (ícone lua/sol)
- Implementado via atributo `data-theme` no `<html>` e CSS overrides das classes Tailwind
- Preferência salva no estado global e persistida

### 3.7 Múltiplas Mesas
- Seletor de mesas no cabeçalho com dropdown
- Botões para **adicionar** e **renomear** mesas
- Cada mesa possui seu próprio array de itens independente
- Troca instantânea entre mesas preservando os dados de cada uma

### 3.8 Histórico de Contas Fechadas
- Ao zerar a conta, os dados atuais são salvos automaticamente no histórico
- Modal com listagem completa de contas fechadas (data, mesa, itens, total)
- Botões para **restaurar** uma conta (substitui a atual) ou **excluir** do histórico
- Limite de 50 entradas no histórico

### 3.9 Edição Inline na Tabela
- Clique no **nome** ou **preço unitário** do item na tabela para editar
- Campo de texto substitui o valor ao clicar
- Enter ou perda de foco confirmam a edição
- Destaque visual (sublinhado + cor) nos campos clicáveis

### 3.10 Máscara Monetária no Input de Preço
- Input do preço formatado como texto com máscara progressiva
- Ao digitar, os números são convertidos automaticamente para valor decimal com 2 casas
- Aceita valores sem separador decimal (ex: "1500" → 15.00)

### 3.11 Exportar PDF
- Gera uma visualização simplificada da conta e aciona a impressão do navegador (`window.print()`)
- Layout otimizado com `@media print` (oculta botões, ajusta margens e cores)
- Exibe nome da mesa, itens, quantidades, subtotais e total geral

### 3.12 Compartilhar via WhatsApp
- Formata a conta como texto simples e abre `wa.me` com o conteúdo
- Inclui nome da mesa, itens, quantidades, valores unitários, subtotais e total
- Abre em nova aba

### 3.13 Exportar / Importar Dados (JSON)
- **Exportar**: baixa um arquivo `.json` com todos os itens da conta (nomeado com mesa + data)
- **Importar**: seleciona um arquivo `.json` válido e substitui os dados atuais
- **Validação na importação**: verifica se o JSON é um array e se cada item possui `nome`, `categoria`, `preco` (number) e `qtd` (number); itens sem `id` recebem um automaticamente
- Botões localizados no cabeçalho

### 3.14 PWA (Progressive Web App)
- Instalável como aplicativo standalone
- Funciona offline (cache-first via Service Worker)
- Meta tags iOS para suporte a Add to Home Screen

---

## 4. Estrutura de Dados

### Estado Global (localStorage key: `consumoApp`)

```json
{
  "mesas": {
    "Mesa 1": [
      { "id": 1685000000000, "nome": "Coca-Cola", "categoria": "bebida", "preco": 6.00, "qtd": 2 }
    ]
  },
  "mesaAtual": "Mesa 1",
  "historico": [
    {
      "data": "2026-06-02T10:30:00.000Z",
      "mesa": "Mesa 1",
      "itens": [ ... ],
      "total": 42.50,
      "qtdItens": 8
    }
  ],
  "tema": "light"
}
```

### Item de Consumo

```json
{
  "id": 1685000000000,
  "nome": "Coca-Cola",
  "categoria": "bebida",
  "preco": 6.00,
  "qtd": 2
}
```

- `id`: gerado via `Date.now()` no momento da criação
- `categoria`: `"comida"` ou `"bebida"`
- `itens` é sempre uma referência a `mesas[mesaAtual]`

---

## 5. Fluxo de Dados

```
Usuário → Formulário → adicionarItem()
                          ↓
          estadoApp.mesas[mesaAtual] (itens[])
                          ↓
              salvarERenderizar()
                ├── sincroniza estadoApp.mesas[mesaAtual] = itens
                ├── localStorage.setItem('consumoApp', estadoApp)
                ├── renderizarTabela() (com suporte a edição inline)
                ├── atualizarTotalizadores()
                ├── atualizarGrafico()
                └── atualizarSelectMesas()

Mudar de mesa:
  mudarMesa(nome)
    ├── salva itens atuais em estadoApp.mesas[mesaAtual]
    ├── atualiza mesaAtual
    └── itens = estadoApp.mesas[novaMesa]

Zerar conta:
  limparTudo()
    ├── salvarHistorico() → estadoApp.historico.unshift(entrada)
    ├── itens = []
    └── salvarERenderizar()
```

---

## 6. Service Worker (sw.js)

| Evento | Estratégia |
|--------|-----------|
| `install` | Pré-cache dos assets (HTML, manifest, CDNs e fontes) |
| `activate` | Limpeza de caches antigos |
| `fetch` | Cache-first com fallback à rede e cache-on-write |

### URLs cacheadas na instalação:
- `./`, `./index.html`, `./manifest.json`
- Tailwind CSS, Chart.js, Font Awesome CSS e webfonts (woff2)

---

## 7. Dependências Externas

| Biblioteca | CDN | Versão |
|-----------|-----|--------|
| Tailwind CSS | `cdn.tailwindcss.com` | mais recente |
| Chart.js | `cdn.jsdelivr.net/npm/chart.js` | 4.x |
| Font Awesome | `cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/` | 6.4.0 |

---

## 8. Layout

### Grid
- **Desktop (lg+)**: 3 colunas — formulário (1) / tabela (2)
- **Tablet (md)**: 3 colunas no topo (totalizadores), conteúdo empilhado
- **Mobile**: layout totalmente empilhado

### Componentes de UI
- Cards com sombra (`shadow-md`, `shadow-sm`)
- Border-left colorido nos totalizadores (indigo, green, purple)
- Botões com hover transitions
- Animações fade-in em novas linhas

---

## 9. Configuração PWA

### Manifest (`manifest.json`)
| Campo | Valor |
|-------|-------|
| `name` | Controle de Consumo |
| `short_name` | Consumo |
| `display` | standalone |
| `start_url` | `.` |
| `theme_color` | `#4f46e5` (indigo-600) |
| `background_color` | `#f3f4f6` (gray-100) |
| `orientation` | portrait-primary |

### iOS Meta Tags
- `apple-mobile-web-app-capable`: yes
- `apple-mobile-web-app-status-bar-style`: default
- `apple-touch-icon`: icon-192.png

---

## 10. Testes

- Framework: **QUnit** via CDN
- Arquivo: `test.html` (abrir no navegador para executar)
- Testes de unidade para funções core (formatação, adição, remoção, alteração de quantidade)
- Verificação de estados (vazio, com itens, importação de dados inválidos)
- Testes de integração para o fluxo de salvamento e carregamento do localStorage

---

## 11. Melhorias Futuras (Ideias)

- Compartilhar conta via Bluetooth/NFC entre dispositivos próximos
- Catálogo de produtos pré-cadastrados (cardápio)
- Sugestão de divisão igualitária da conta entre pessoas
- Tema customizável (cores do estabelecimento)
- Integração com impressoras térmicas

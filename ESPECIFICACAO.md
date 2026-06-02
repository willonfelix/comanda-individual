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
ESPECIFICACAO.md    — Este documento
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
- Dados salvos no `localStorage` sob a chave `consumoItens`
- Restaurados automaticamente ao recarregar a página

### 3.6 Exportar / Importar Dados (JSON)
- **Exportar**: baixa um arquivo `.json` com todos os itens da conta (nomeado com a data atual)
- **Importar**: seleciona um arquivo `.json` válido e substitui os dados atuais
- **Validação na importação**: verifica se o JSON é um array e se cada item possui `nome`, `categoria`, `preco` (number) e `qtd` (number); itens sem `id` recebem um automaticamente
- Botões localizados no cabeçalho ao lado de "Zerar Conta"

### 3.7 PWA (Progressive Web App)
- Instalável como aplicativo standalone
- Funciona offline (cache-first via Service Worker)
- Meta tags iOS para suporte a Add to Home Screen

---

## 4. Estrutura de Dados

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

---

## 5. Fluxo de Dados

```
Usuário → Formulário → adicionarItem()
                          ↓
                itens[] (array em memória)
                          ↓
              salvarERenderizar()
                ├── localStorage.setItem()
                ├── renderizarTabela()
                ├── atualizarTotalizadores()
                └── atualizarGrafico()
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

## 10. Melhorias Futuras (Não Implementadas)

- Edição inline do nome/preço na tabela
- Exportar conta como PDF ou compartilhar via WhatsApp
- Múltiplas contas/mesas simultâneas
- Histórico de contas fechadas
- Input de valor com máscara monetária
- Tema escuro (dark mode)
- Testes automatizados

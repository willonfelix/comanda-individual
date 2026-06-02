Aqui está um aplicativo web completo, funcional e moderno, feito em um único arquivo (HTML + CSS + JavaScript).
Ele atende a todos os seus requisitos: cadastro de itens com nome, categoria (comida/bebida) e preço; controle incremental de quantidade; totalizadores em tempo real; e um gráfico visual separando o consumo de Comidas e Bebidas.

Funcionalidades Principais do Código:
Incremento Inteligente: Se você adicionar "Coca-Cola" por R$ 6,00 e depois adicionar "Coca-Cola" novamente, o sistema não cria uma nova linha. Ele encontra a existente e soma a quantidade, atualizando o subtotal automaticamente.
Controles Rápidos na Tabela: Botões de + e - diretamente na lista para ajustar quantidades rapidamente sem precisar recadastrar.
Totalizadores em Tempo Real:
Total de itens (soma de todas as quantidades).
Valor total da conta (formatado em Reais).
Ticket médio (valor total dividido pela quantidade total de itens).
Gráfico Dinâmico (Chart.js): Um gráfico de rosca (Doughnut) que mostra visualmente a proporção de gasto entre Comidas (Laranja) e Bebidas (Azul). Ele se atualiza instantaneamente a cada alteração.
Persistência de Dados: Usa o localStorage do navegador. Se você fechar a aba e abrir de novo, a conta do seu cliente ainda estará lá.
Design Responsivo: Funciona perfeitamente tanto no computador quanto na tela do celular (graças ao Tailwind CSS).

